# Phase 5.2 — Vault ↔ Kubernetes Authentication

## Purpose

This phase defines **how External Secrets Operator authenticates to Vault**.

This is the **only authentication path** between Kubernetes and Vault in this platform.

Applications never authenticate to Vault.
Vault Agent Injector is not used.

---

## Trust Model Overview

```
Kubernetes API Server
        ↓
ServiceAccount (ESO)
        ↓
Vault Kubernetes Auth Method
        ↓
Vault Role
        ↓
Vault Policy
        ↓
Read access to secrets
```

Vault treats Kubernetes as an **identity provider**.

---

## Step 1 — ServiceAccount for ESO

ESO authenticates to Vault using a dedicated ServiceAccount.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-secrets
  namespace: external-secrets
```

This ServiceAccount represents ESO’s identity.

---

## Step 2 — Enable Kubernetes Auth in Vault

Enable the auth method (once):

```bash
vault auth enable kubernetes
```

Configure Vault to trust the Kubernetes API:

```bash
vault write auth/kubernetes/config \
  kubernetes_host="https://$KUBERNETES_SERVICE_HOST:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt \
  token_reviewer_jwt=@/var/run/secrets/kubernetes.io/serviceaccount/token
```

Vault can now validate Kubernetes-issued identities.

---

## Step 3 — Define Vault Policies

Policies define **what** ESO can read.

Example (read-only access to app secrets):

```hcl
path "kv/data/apps/*" {
  capabilities = ["read"]
}
```

Apply the policy:

```bash
vault policy write apps-read kv-apps-read.hcl
```

---

## Step 4 — Create Vault Role

Bind the Kubernetes ServiceAccount to the policy:

```bash
vault write auth/kubernetes/role/external-secrets \
  bound_service_account_names=external-secrets \
  bound_service_account_namespaces=external-secrets \
  policies=apps-read \
  ttl=1h
```

---

## Step 5 — ClusterSecretStore (ESO → Vault)

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: http://vault.vault.svc:8200
      path: kv
      version: v2
      auth:
        kubernetes:
          mountPath: kubernetes
          role: external-secrets
          serviceAccountRef:
            name: external-secrets
            namespace: external-secrets
```

---

## Phase Outcome

At the end of Phase 5.2:

- Vault trusts Kubernetes identities
- ESO can authenticate successfully
- ClusterSecretStore is functional
- No applications consume secrets yet
