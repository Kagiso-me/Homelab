# ☸️ Kubernetes Security (Practical Baseline)

This document defines a **practical Kubernetes security baseline** for this environment.

It intentionally focuses on **what materially reduces risk**, with a strong emphasis on:
- external secrets
- identity-based access
- safe workload defaults

It avoids unnecessary theory and excludes Authentik (already handled elsewhere).

---

## 🎯 Goal

By applying this guide, Kubernetes should:
- Never be the source of truth for secrets
- Consume secrets securely from Vault
- Run workloads with safe defaults
- Limit blast radius when a pod is compromised

This is not about perfection — it is about **removing the most common failure modes**.

---

## 🔐 Scope (Intentional)

### Included
- External secrets using Vault
- Safe pod defaults
- Namespace-level isolation (lightweight)
- Minimal but effective controls

### Excluded (on purpose)
- Authentik
- Advanced multi-tenancy
- Service mesh
- Heavy runtime security stacks
- Cloud-specific IAM

---

## 1️⃣ Secrets Management (Primary Focus)

### Principle

**Kubernetes is not a secrets vault.**

Secrets must:
- live outside the cluster
- be injected at runtime
- be rotatable
- never appear in Git or Helm values

---

## 2️⃣ Vault as the Source of Truth

:contentReference[oaicite:0]{index=0} is used as the **single source of truth** for secrets.

Vault responsibilities:
- Store secrets securely
- Control access via policies
- Issue scoped credentials
- Support rotation

Kubernetes responsibilities:
- Authenticate to Vault
- Request secrets
- Cache them briefly
- Consume them as native `Secret` objects

---

## 3️⃣ Vault Authentication Model (Kubernetes)

Vault authenticates Kubernetes workloads using the **Kubernetes auth method**.

High-level flow:
1. Pod runs with a service account
2. Vault validates the service account JWT
3. Vault maps it to a role
4. Vault returns only the secrets that role is allowed to access

No shared credentials.  
No static tokens in pods.

---

## 4️⃣ External Secrets Operator (ESO)

External Secrets Operator is used to **sync secrets from Vault into Kubernetes**.

### What ESO does
- Watches for `ExternalSecret` resources
- Authenticates to Vault
- Fetches secrets
- Creates Kubernetes `Secret` objects
- Refreshes them automatically

Kubernetes becomes a **consumer**, not a vault.

---

## 5️⃣ Installing External Secrets Operator

Example using Helm:

```bash
helm repo add external-secrets https://charts.external-secrets.io
helm repo update

helm install external-secrets external-secrets/external-secrets \
  --namespace external-secrets \
  --create-namespace
```
Example using Helm:
```bash
kubectl get pods -n external-secrets
```
---

## 6️⃣ Configure Vault Access for Kubernetes

Example using Helm:
```hcl
path "kv/data/apps/myapp/*" {
  capabilities = ["read"]
}
```
Create a Vault role bound to a service account

```bash
vault write auth/kubernetes/role/myapp \
  bound_service_account_names=myapp-sa \
  bound_service_account_namespaces=apps \
  policies=myapp \
  ttl=1h
```
This ensures:
 - Only myapp-sa can read these secrets
 - Secrets are scoped
 - Tokens expire automatically

## 7️⃣ Create a SecretStore in Kubernetes

```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
  namespace: apps
spec:
  provider:
    vault:
      server: https://vault.kagiso.me
      path: kv
      version: v2
      auth:
        kubernetes:
          mountPath: kubernetes
          role: myapp
``` 
This tells Kubernetes how to talk to Vault.

## 8️⃣ Create an ExternalSecret

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-secrets
  namespace: apps
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: myapp-secrets
    creationPolicy: Owner
  data:
    - secretKey: DB_PASSWORD
      remoteRef:
        key: apps/myapp/database
        property: password
``` 
Result:
 - Vault secret → synced into Kubernetes
 - Appears as a normal Secret
 - Automatically refreshed
 - Never committed to Git

## 9️⃣ Use the Secret in a Pod

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: myapp-secrets
        key: DB_PASSWORD
``` 
From the application’s perspective:
 - This is just a normal environment variable
 - No Vault logic inside the app

 ## 10️⃣ Pod Security (Minimal but Effective)
 Every workload should, at minimum, enforce:

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  seccompProfile:
    type: RuntimeDefault
  capabilities:
    drop:
      - ALL
``` 
This removes the most common container escape paths.

 ## 11️⃣ Namespace-Level Hygiene
 Keep it simple:
 - One application per namespace (where possible)
 - One service account per app
 - Secrets scoped to that namespace
 - No shared “default” service account

Namespaces are blast-radius boundaries.