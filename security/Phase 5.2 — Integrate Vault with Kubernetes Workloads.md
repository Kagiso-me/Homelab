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

---

## ✅ Phase 5.2 Done When

- Vault trusts Kubernetes
- Roles bound to ServiceAccounts
- Tokens issued with TTLs

---
