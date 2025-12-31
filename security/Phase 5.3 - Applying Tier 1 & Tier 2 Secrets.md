# 🥇 Phase 5.3 — Applying Tier 1 & Tier 2 Secrets  
**Practical Workload Integration Guide**

---

## 📖 Purpose

This phase applies workload identity to **real applications**.

We intentionally support **two secret models** to balance:
- Security
- Compatibility
- Operational simplicity

---

## 🥇 Tier 1 — Dynamic Secrets (Preferred)

### When to Use
- Databases
- Admin access
- High-impact credentials

### How It Works
- Vault creates credentials on demand
- TTL enforced
- Automatic revocation

Example:
```hcl
path "database/creds/app" {
  capabilities = ["read"]
}
```

Workloads request credentials at runtime.

---

## 🥈 Tier 2 — Identity-Gated KV Secrets

### When to Use
- Third-party API keys
- Legacy apps
- Secrets that cannot rotate dynamically

### How It Works
- Secrets stored in Vault KV v2
- Access controlled by Kubernetes Auth
- No secrets in Git or YAML

Example:
```hcl
path "kv/data/app" {
  capabilities = ["read"]
}
```

Still Zero Trust — just pragmatic.

---

## 🧠 Choosing the Right Tier

| Question | Use Tier |
|-------|---------|
Can credentials rotate automatically? | Tier 1 |
High blast radius if leaked? | Tier 1 |
Legacy app? | Tier 2 |
Low-risk API key? | Tier 2 |

---

## 🧪 Migration Strategy

1. Start with one Tier 1 workload
2. Validate stability
3. Migrate Tier 2 workloads
4. Document exceptions

Never force Vault where it breaks apps.

---

## ✅ Phase 5.3 Done When

- At least one Tier 1 app migrated
- Tier 2 apps gated by identity
- No secrets in manifests
- Vault audit logs show access

---

> *Zero Trust is not dogma. It is deliberate control.*
