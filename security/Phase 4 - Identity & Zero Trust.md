# 🥇 Phase 4 — Identity & Zero Trust  
#### Security Level: 8 / 10

---

✅ **Phase 1** ⇒ Hardened the hosts.  
✅ **Phase 2** ⇒ Gave us visibility.  
✅ **Phase 3** ⇒ Gave us enforcement.

## 🧠 Context: What Changes in Phase 4

Up to Phase 3, security decisions were based on:
- Network location
- IP reputation
- Observed behavior

That model breaks down when:
- Users move between networks
- VPNs flatten trust boundaries
- Internal services are exposed
- Attackers operate *inside* the network

Phase 4 introduces a **deliberate shift**:

> **The network no longer grants trust.  
> Identity does.**

This is the core Zero Trust idea.

---

## 🎯 Phase Goal

Replace **implicit network trust** with **explicit identity‑based access**.

By the end of Phase 4:

- No service is accessible without authentication
- LAN and WAN access behave identically
- Access is evaluated per‑user and per‑service
- MFA protects sensitive access
- CrowdSec shields the identity layer from abuse

If being “on the LAN” still grants access, Phase 4 is **not complete**.

---

## 🔐 Scope

**In scope:**
- Human access to services
- Web UIs and dashboards
- Reverse‑proxied applications
- Administrative tooling

**Out of scope (for now):**
- Kubernetes workload identity
- Service‑to‑service auth
- Mesh‑based identity

Those are addressed in Phase 5.

---

## 🗺️ The Phase 4 Access Model

Under Zero Trust, access follows this path:

```
User
 → Reverse Proxy (Traefik)
 → Identity Provider (Authentik)
 → Policy Decision
 → Service (Allow / Deny)
```

Key change:

> **Services no longer decide who can access them.**

They delegate that responsibility to identity.

---

## 1️⃣ Choosing an Identity Provider (Decision Explained)

Before implementation, we must choose an Identity Provider (IdP).

### What an IdP Must Provide

At minimum:

- Central authentication
- MFA support (TOTP / WebAuthn)
- Policy‑based authorization
- Reverse proxy integration
- Clear audit logging

Anything less is not Zero Trust.

---

### Common Self‑Hosted Options

| IdP | Strengths | Weaknesses |
|---|---|---|
Authentik | Modern, proxy‑native, flexible policies | Learning curve |
Keycloak | Extremely powerful, enterprise‑grade | Heavy and complex |
Authelia | Lightweight and simple | Limited policy depth |
OAuth‑only | Easy | Not Zero Trust by default |

---

### ✅ Selected Option: **Authentik**

We choose **Authentik** because:

- It is designed for self‑hosted environments
- It integrates cleanly with Traefik
- Its policy model matches Zero Trust thinking
- MFA is first‑class
- Auditability is excellent

This guide uses Authentik, but the architecture applies broadly.

---

## 2️⃣ Roles & Responsibilities (Clear Separation)

Each component has a single responsibility:

| Component | Responsibility |
|---|---|
Traefik | Enforces access decisions |
Authentik | Makes identity & policy decisions |
Applications | Provide functionality only |
CrowdSec | Blocks hostile traffic *before* identity |

If any component crosses responsibilities, complexity explodes.

---

## 3️⃣ Where Identity Lives (Placement Matters)

Authentik must:

- Be reachable by Traefik
- Be protected by TLS
- Never be exposed directly

**Recommended placement:**
- Deploy Authentik inside the cluster
- Expose it only through Traefik
- Protect it with CrowdSec

Identity is now **infrastructure‑critical**.

---

## 4️⃣ Forward Authentication (The Core Mechanism)

Forward authentication works as follows:

1. User requests a service
2. Traefik pauses the request
3. Traefik asks Authentik to authenticate the user
4. Authentik enforces login + MFA
5. Authentik returns identity headers
6. Traefik allows or denies the request

The protected service:
- Never sees credentials
- Never handles sessions
- Never decides access

This dramatically reduces attack surface.

---

## 5️⃣ Authentik Concepts You Must Understand

Authentik always combines **three objects**:

### Application
Represents *what* is being protected.

### Provider
Defines *how* authentication occurs  
(for Traefik, this is a **Forward Auth Provider**).

### Policy
Defines *who* is allowed (groups, MFA, conditions).

If one is missing, access must fail.

---

## 6️⃣ Designing Identity Groups (Before Config)

Before touching Traefik or Authentik config, define intent.

Recommended baseline groups:
- `admins`
- `users`
- `read-only`
- `infrastructure`

Policies reference **groups**, not individuals.

This keeps access manageable over time.

---

## 7️⃣ Practical Integration: Authentik + Traefik

Now we connect theory to execution.

### 7.1 Create a Forward Auth Provider (Authentik)

In Authentik Admin UI:

- Create **Proxy Provider**
- Type: **Forward Auth (single application)**
- External host: `https://service.example.com`

This defines *how* Traefik will authenticate users.

---

### 7.2 Create the Application (Authentik)

- Name: `Protected Service`
- Provider: select the provider created above

This binds identity logic to a real service.

---

### 7.3 Attach Access Policies

Example policy:
- User must belong to `admins`
- MFA required

Attach policies to:
- The Application or
- The Provider

No policy = no access.

---

## 8️⃣ Traefik Configuration (Minimal, Intentional)

### ForwardAuth Middleware

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: authentik-forward-auth
  namespace: traefik
spec:
  forwardAuth:
    address: https://auth.example.com/outpost.goauthentik.io/auth/traefik
    trustForwardHeader: true
    authResponseHeaders:
      - X-authentik-username
      - X-authentik-groups
      - X-authentik-email
```

Traefik does not authenticate.
It enforces Authentik’s decision.

---

### Protecting a Service

```yaml
middlewares:
  - name: authentik-forward-auth
    namespace: traefik
```

Once applied:
- Unauthenticated users are redirected
- Unauthorized users are denied
- LAN and WAN behave identically

---

## 9️⃣ CrowdSec + Identity (How They Complement)

CrowdSec operates **before** identity.

```
Malicious IP
 → CrowdSec blocks at edge
 → Authentik never sees traffic

Legitimate user
 → Authentik authenticates
 → Traefik enforces
```

This protects:
- Login pages
- MFA endpoints
- Identity infrastructure itself

CrowdSec and identity **layer**, not replace each other.

---

## 🔟 Validation — Declaring Phase 4 Complete

Phase 4 is complete when **all** are true:

- Services redirect to Authentik
- No service is accessible unauthenticated
- LAN access behaves like WAN
- MFA is enforced for sensitive access
- CrowdSec blocks abusive IPs upstream
- Identity audit logs show access events

If even one service bypasses identity, Phase 4 is **not done**.

---

## 🧭 Phase 4 Completion Checklist

- [ ] Authentik deployed securely
- [ ] MFA enabled for admins
- [ ] Identity groups defined
- [ ] Forward auth provider created
- [ ] Traefik middleware applied
- [ ] All services protected
- [ ] LAN trust fully removed
- [ ] CrowdSec protecting identity endpoints

Only when all boxes are checked is Phase 4 complete.

---

## ⏭️ What Comes Next

Phase 5 moves Zero Trust **inside the cluster**:
- Workload identity
- Secrets without plaintext
- Admission‑time policy enforcement

Phase 4 protects **humans**.  
Phase 5 protects **software**.

---

> *Zero Trust is not about distrust.*  
> *It is about being precise about trust.*

---

⬅️ Previous: [Phase 2 - Perimeter Awareness](Phase-2---Perimeter-Awareness.md)  
➡️ Next: [Phase 4 - Identity & Zero Trust](Phase 4 - Identity & Zero Trust.md)