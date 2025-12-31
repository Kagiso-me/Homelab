# 🥇 Phase 4 — Identity & Zero Trust
#### Security Level: 8 / 10

---

✅ **Phase 1** ⇒ Hardened the hosts.  
✅ **Phase 2** ⇒ Gave us visibility.  
✅ **Phase 3** ⇒ Gave us enforcement.

**Phase 4 removes implicit trust entirely.**

This is the phase where the network officially stops being a security boundary and **identity becomes the control plane**.

This document is written to be:
- Readable end-to-end
- Referenced later without confusion
- Understood by engineers who are *new* to identity
- Useful to engineers who are *already familiar* with Zero Trust

Nothing is assumed. Everything is explained.

---

## 🧠 Why Phase 4 Exists

Traditional security models assume:

> “If you are on the internal network, you are trusted.”

That assumption no longer holds.

Modern realities:
- Laptops move between networks
- VPNs collapse trust boundaries
- Internal services get exposed accidentally
- Attackers live *inside* networks once breached

**Zero Trust starts with a single, uncomfortable truth:**

> *The network cannot be trusted — even your own.*

Phase 4 is about designing security around that truth.

---

## 🎯 Phase Goal

Replace **network-based trust** with **identity-based access**.

By the end of this phase:

- No service is reachable without authentication
- Internal and external access behave identically
- Every request is evaluated against identity and policy
- MFA protects sensitive access
- CrowdSec and identity reinforce each other

This phase does not add more firewalls.

It changes **what trust means**.

---

## 🔐 Scope

**Applies to:**
- Web applications
- Dashboards and admin panels
- APIs exposed via reverse proxy
- Human access to services

**Out of scope (for now):**
- Kubernetes workload identity
- Service mesh
- Machine-to-machine auth

Those come later.

---

## 🧱 Core Zero Trust Principles

These principles guide every decision in this phase.

1. **Never trust the network**
2. **Always authenticate**
3. **Authorize explicitly**
4. **Assume breach**
5. **Minimize standing access**

If a tool or shortcut violates these principles, it is rejected.

---

## 🗺️ The New Access Model (Conceptual)

Under Zero Trust, access follows this flow:

```
User
  ↓
Reverse Proxy
  ↓
Identity Provider (Authentication + MFA)
  ↓
Policy Decision
  ↓
Service (Allowed or Denied)
```

Key shift:

> **Services no longer decide who can access them.**

They delegate that responsibility to identity.

---

## 1️⃣ Introducing the Identity Provider (IdP)

An **Identity Provider (IdP)** is the system responsible for:

- Verifying who a user is
- Enforcing MFA
- Issuing short-lived identity assertions
- Evaluating access policies

In Phase 4, the IdP becomes the **source of truth for access**.

Common examples:
- Authentik
- Keycloak
- Authelia

This guide uses **Authentik** as the reference implementation, but the concepts apply universally.

---

## 2️⃣ Why Identity Sits in Front of Services

Historically, applications handled:
- Login pages
- Password storage
- Sessions
- Authorization logic

This led to:
- Inconsistent security
- Repeated mistakes
- Weak implementations

Phase 4 removes identity responsibility from applications entirely.

Applications should:
- Trust headers
- Focus on business logic
- Never see passwords

Identity becomes **centralized, audited, and consistent**.

---

## 3️⃣ The Reverse Proxy Becomes an Identity Gateway

Your reverse proxy already controls traffic flow.

In Phase 4, it gains a second role:

> **Identity-aware gateway**

For every incoming request, the proxy asks:

> “Who is this user, and are they allowed here?”

This is achieved using **Forward Authentication**.

---

## 4️⃣ Forward Authentication Explained

Forward authentication works step-by-step:

1. User requests a service
2. Reverse proxy pauses the request
3. Proxy forwards the request to the IdP
4. IdP authenticates the user (and enforces MFA)
5. IdP returns identity headers
6. Proxy allows or denies the request

The protected service:
- Never handles authentication
- Never stores credentials
- Never decides access

This dramatically reduces attack surface.

---

## 5️⃣ Where CrowdSec Fits in Phase 4

CrowdSec does **not disappear** in a Zero Trust model.

Its role becomes more precise.

### CrowdSec acts as:

- A **pre-identity filter**
- A **hostile traffic suppressor**
- A **protective layer in front of identity**

The combined flow looks like this:

```
Hostile IP
 → CrowdSec blocks early (Phase 3)
 → Identity never sees the request

Legitimate user
 → Identity authenticates
 → Access evaluated
 → Service reached
```

This prevents:
- Credential stuffing
- MFA fatigue attacks
- Identity brute-force attempts

> **CrowdSec protects identity. Identity protects services.**

---

## 6️⃣ Designing Human-Centric Access Policies

Zero Trust policies are written in **human terms**, not network terms.

Examples:
- “Admins can access admin dashboards”
- “Only I can access infrastructure tools”
- “Read-only users can view metrics”
- “No service is public by default”

We no longer ask:
> “Where is this user coming from?”

We ask:
> “Who is this user, and what are they allowed to do?”

---

## 7️⃣ MFA — Mandatory, but Intentional

Multi-Factor Authentication is required, but not everywhere blindly.

Apply MFA to:
- Identity login itself
- Administrative access
- Sensitive services

Do **not** apply MFA to:
- Machine-to-machine traffic
- Internal service calls
- Automated processes

The goal is:
> **Strong security without operational exhaustion**.

---

## 8️⃣ Validation — How to Know Phase 4 Is Working

Phase 4 is successful when:

- Accessing any service redirects to identity login
- Unauthenticated requests are denied
- Authenticated users see only allowed services
- LAN and WAN access behave identically
- CrowdSec blocks hostile IPs before identity
- Identity logs show clear audit trails

If being “on the LAN” still grants access — Phase 4 is incomplete.

---

## 🧭 Phase 4 Output

After completing Phase 4, you now have:

- Identity as the security boundary
- Zero Trust access for services
- MFA-backed authentication
- Reduced blast radius for breaches
- A system designed for modern threat models

This is where security stops being about infrastructure and starts being about **people and intent**.

---

**Security Level Achieved:** ⭐⭐⭐⭐⭐⭐⭐⭐☆☆ (8 / 10)

---

## ⏭️ What Comes Next

Phase 5 moves Zero Trust **inside the cluster**:
- Workload identity
- Secret zeroization
- Kubernetes-native trust boundaries

But Phase 4 is the turning point.

> *“Internal” no longer means “trusted”.*

---
⬅️ Previous: [Phase 3 - Active Defense & Enforcement](Phase 3 - Active Defense & Enforcement.md)  
➡️ Next: [Baseline Hardening](02-baseline-hardening.md)
---