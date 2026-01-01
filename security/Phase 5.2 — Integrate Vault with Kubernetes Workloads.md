# 🥇 Phase 5.2 — Vault ↔ Kubernetes Authentication  
**Practical Implementation Guide**

---

## 📖 Purpose

This phase enables **workload authentication**.

Vault learns to trust Kubernetes identities using:
- ServiceAccounts
- Kubernetes Auth Method
- Explicit role bindings

This phase is common to **both Tier 1 and Tier 2**.

---

## 🧠 Identity Flow

```
Pod → ServiceAccount → Vault Auth → Vault Role → Token
```

---

## 1️⃣ Enable Kubernetes Auth

```bash
vault auth enable kubernetes
```

---

## 2️⃣ Allow Vault to Validate Pods

```bash
kubectl create serviceaccount vault-auth -n kube-system
```

Bind RBAC:

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: vault-auth-delegator
roleRef:
  kind: ClusterRole
  name: system:auth-delegator
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: vault-auth
  namespace: kube-system
```

---

## 3️⃣ Configure Vault Trust

```bash
vault write auth/kubernetes/config   kubernetes_host=https://<API_SERVER>   token_reviewer_jwt=@jwt.txt   kubernetes_ca_cert=@ca.crt
```

Vault now trusts Kubernetes identities.

---

## 4️⃣ Define Roles & Policies

Roles bind:
- ServiceAccount
- Namespace
- Policy
- TTL

This identity layer underpins **all secret access**.


## 4.1 Create a Minimal Vault Policy (Proof Only)

This policy grants **no secret access**.  
It exists purely to validate authentication and authorization flow.

```bash
vault policy write k8s-auth-test - <<EOF
path "auth/token/lookup-self" {
  capabilities = ["read"]
}
EOF
```
This allows a workload to inspect its own token metadata and nothing else.

## 4.2 Create a Test Kubernetes Identity

We now create a deliberately unprivileged Kubernetes identity to test authorization.

```bash
kubectl create namespace vault-test
kubectl create serviceaccount vault-test-sa -n vault-test
```

This identity represents:
- a single workload
- in a single namespace
- with no implicit privileges

---

## 4.3 Bind Identity to Policy with a Vault Role

This step enforces **who may authenticate** and **what they receive**.

```bash
vault write auth/kubernetes/role/vault-test \
  bound_service_account_names=vault-test-sa \
  bound_service_account_namespaces=vault-test \
  policies=k8s-auth-test \
  ttl=15m
```

This role enforces:

- Authentication is limited to `vault-test-sa`
- Only in the `vault-test` namespace
- Tokens are short-lived (15 minutes)
- No other workloads are trusted

This is **deny-by-default** authorization.

---

## 4.4 Validate Authentication from a Pod

We now prove that:
- Kubernetes identity works
- Vault authorization works
- Tokens are issued correctly
- No secrets are exposed

### Launch a test pod

```bash
kubectl run vault-auth-test \
  -n vault-test \
  --rm -it \
  --image=alpine \
  --serviceaccount=vault-test-sa \
  -- sh
```

Inside the pod:

```sh
apk add --no-cache curl jq
```

Authenticate to Vault:

```sh
export VAULT_ADDR=https://sentinel.local.kagiso.me:8200
export VAULT_ROLE=vault-test

JWT=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)

curl -s --request POST \
  --data "{\"role\":\"$VAULT_ROLE\",\"jwt\":\"$JWT\"}" \
  $VAULT_ADDR/v1/auth/kubernetes/login | jq
```

### Expected outcome

- A Vault token is returned
- The token has:
  - policy `k8s-auth-test`
  - a short TTL
- No secrets are accessible

Exit the pod:

```sh
exit
```

---

## 4.5 Observe Audit Logs

On the Sentinel node:

```bash
sudo tail -n 10 /var/log/vault_audit.log
```

You should see:
- Kubernetes authentication events
- Role evaluation
- Token issuance

This confirms:
- identity enforcement
- authorization decisions
- observability

---

## ✅ Phase 5.2 Completion Criteria

Phase 5.2 is complete when:

- Vault validates Kubernetes ServiceAccount identities
- Vault roles bind identities to policies
- Policies enforce explicit permissions
- Tokens are short-lived and scoped
- Audit logs show authentication activity
- **No secrets are consumed yet**

At this point:

> **Workload identity is real, enforced, and observable.**

---

## 🧭 Transition to Phase 5.3

With identity and authorization validated, the environment is now safe to introduce secrets.

Phase 5.3 focuses on:
- Tier 1 static secrets
- Tier 2 dynamic credentials
- Data-layer least privilege

This staged approach ensures:
- minimal blast radius
- clean audit trails
- predictable behavior
