# 🥇 Phase 5 — Kubernetes & Workload Identity  
#### Security Level: 9 / 10

---

## 📖 Context: Continuing from Phase 4

Phase 4 removed **implicit trust for humans**.

- Identity (Authentik) decides *who* may access services  
- Traefik enforces access consistently  
- Network location no longer grants trust  
- CrowdSec shields identity infrastructure  

At this point, humans are no longer trusted by default.

However, a critical assumption still exists:

> **Software inside the cluster still trusts too much.**

Phase 5 removes *implicit trust for workloads*.

---

## 🎯 Phase 5 Goal

Introduce **explicit workload identity** and **secret minimisation**.

By the end of Phase 5:

- Workloads authenticate explicitly
- Secrets are never hard-coded into manifests
- Credentials are short-lived *where possible*
- Static secrets are controlled and auditable
- Blast radius of compromise is minimal

Zero Trust now applies to **humans and software**.

---

## 🧠 The Trust Shift

Before:

```
Pod → Kubernetes Secret → Resource
```

After:

```
Pod → Identity → Vault → Credential → Resource
```

This mirrors Phase 4:

```
Human → Identity → Traefik → Service
```

Same philosophy.  
Different actor.

---

## 🧱 Why HashiCorp Vault

Kubernetes Secrets alone cannot:

- Rotate credentials dynamically
- Enforce TTLs
- Revoke access instantly
- Produce strong audit trails

We introduce **HashiCorp Vault** as a **trust broker** — not just a secrets store.

Vault provides:
- Workload authentication
- Dynamic secrets (preferred)
- Identity-gated static secrets (acceptable)
- Centralised policy and audit

---

## 🥇 Two Supported Secret Models

This architecture deliberately supports **two tiers**.

### Tier 1 — Dynamic Secrets (Preferred)
- Issued on demand
- Short-lived
- Automatically revoked
- Used for databases and high-risk access

### Tier 2 — Identity-Gated KV Secrets (Pragmatic)
- Stored in Vault KV v2
- Access controlled by workload identity
- No secrets in Git or YAML
- Used for legacy or incompatible apps

Both tiers:
- Use Kubernetes Auth
- Enforce least privilege
- Are auditable and revocable

---

## 🗺️ Phase 5 Execution Plan

```
Phase 5   — Workload Identity (this document)
Phase 5.1 — Install & Secure Vault on Sentinel
Phase 5.2 — Vault ↔ Kubernetes Integration (Auth + Policies)
Phase 5.3 — Applying Tier 1 & Tier 2 to Real Workloads
```

---

## 🧭 Definition of Done

Phase 5 is complete when:

- Workloads authenticate to Vault
- No secrets live in manifests
- Dynamic secrets used where possible
- Static secrets are identity-gated
- Vault audit logs show access trails

---

> *Zero Trust is complete only when software must prove who it is.*
