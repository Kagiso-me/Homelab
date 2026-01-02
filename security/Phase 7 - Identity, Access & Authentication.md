# Phase 7 — Identity, Access & Authentication

Phase 7 introduces a **central identity plane** for your cluster using **Authentik** as the Identity Provider (IdP), integrated cleanly with:
- Vault (for secrets)
- cert-manager (for TLS)
- Traefik (for ingress + forward authentication)
- Applications (starting with Nextcloud)

> **Goal of Phase 7**
>
> Move authentication and access control *out of individual apps* and into a **single, auditable, policy-driven identity layer**.

---

## 1. Why Phase 7 Exists

Without a central IdP:
- Every app manages its own users and passwords
- No consistent MFA or access policies
- Hard to audit who has access to what
- Revocation is slow and error-prone

With Phase 7:
- One identity source
- One login experience
- Central access rules
- Apps delegate auth instead of owning it

---

## 2. Architecture Overview

High-level flow:

```
User
  ↓
Traefik (Ingress)
  ↓
Authentik (IdP)
  ↓
Application (Nextcloud, Grafana, etc.)
```

Key principles:
- Authentication happens **before** traffic reaches the app
- Apps never store user passwords (when possible)
- Traefik enforces auth consistently
- Secrets are managed via Vault + ESO

---

## 3. Install Authentik (Vault-backed, TLS, Ingress)

### 3.1 Deployment model

Authentik will be deployed:
- Inside Kubernetes
- With ingress via Traefik
- With TLS via cert-manager
- With secrets sourced from Vault (via ESO)

Components:
- authentik-server
- authentik-worker
- PostgreSQL (existing or dedicated)
- Redis (optional but recommended)

---

### 3.2 Secrets strategy (important)

We **do not** use inline Kubernetes Secrets.

Instead:
- All Authentik secrets live in Vault
- ESO syncs them into Kubernetes

Example Vault path:
```
kv/apps/authentik
```

Example keys:
- secret_key
- postgres_password
- redis_password
- email_password (if SMTP enabled)

These are exposed to Authentik via an `ExternalSecret`.

---

### 3.3 TLS & ingress

- Authentik is exposed at:
  ```
  https://auth.kagiso.me
  ```
- TLS certificates are issued by cert-manager (ACME / Let’s Encrypt)
- Traefik handles HTTPS termination

This ensures:
- Secure login flows
- Proper OIDC redirect handling
- Browser trust without warnings

---

## 4. Traefik Forward Authentication Design

Not all apps support SSO natively.  
Forward authentication solves this.

### 4.1 What is forward-auth?

Traefik delegates authentication to Authentik **before** forwarding traffic:

```
Request → Traefik → Authentik → (allowed?) → App
```

If the user is:
- ❌ Not authenticated → redirected to Authentik
- ❌ Not authorized → access denied
- ✅ Authorized → request forwarded

---

### 4.2 Why this matters

Forward-auth allows you to:
- Protect apps that have **no built-in auth**
- Apply consistent access rules
- Avoid app-level auth duplication

Examples:
- Internal dashboards
- Admin UIs
- Self-hosted tools

---

### 4.3 Traefik middleware model

Traefik uses a middleware like:

```yaml
forwardAuth:
  address: https://auth.kagiso.me/outpost.goauthentik.io/auth/traefik
  trustForwardHeader: true
  authResponseHeaders:
    - X-authentik-username
    - X-authentik-email
```

This middleware is attached to IngressRoutes selectively.

---

## 5. SSO for Applications — Start with Nextcloud

Nextcloud is the **first practical win**:
- High user value
- Mature OIDC support
- Clear benefit over local users

---

### 5.1 Authentication model for Nextcloud

We will use:
- OIDC (OpenID Connect)
- Authentik as the IdP
- Nextcloud as the relying party

Flow:

```
User → Nextcloud → Authentik → Nextcloud
```

---

### 5.2 Benefits

- Central user management
- Optional MFA (later)
- One login for multiple apps
- No Nextcloud-managed passwords

---

### 5.3 Integration approach

Steps (high level):
1. Create OIDC provider in Authentik
2. Register Nextcloud as an application
3. Configure redirect URIs
4. Configure Nextcloud OIDC app
5. Test login and user provisioning

Secrets (client_id / client_secret) are stored in Vault and injected via ESO.

---

## 6. Access Control Strategy

Authentik allows:
- User groups
- Policies
- Conditional access (later)

Initial model:
- Admin group
- User group
- Per-app access rules

This keeps things simple while remaining extensible.

---

## 7. What Phase 7 Does NOT Do (Yet)

Phase 7 intentionally does **not**:
- Enforce NetworkPolicies (Phase 6)
- Enable runtime detection (Falco)
- Apply Pod Security restrictions

Those will be layered **after identity is stable**.

---

## 8. End State After Phase 7

By the end of Phase 7, you will have:

- Central identity provider (Authentik)
- TLS-secured login flows
- Traefik enforcing auth at the edge
- Nextcloud using SSO
- Vault-backed secrets throughout
- A foundation for MFA and Zero Trust

---

## 9. Next Execution Steps

Execution will proceed in this order:

1. Deploy Authentik (Helm)
2. Wire Vault secrets via ESO
3. Configure ingress + TLS
4. Add Traefik forward-auth middleware
5. Integrate Nextcloud with OIDC
6. Validate login flows

---

## 10. Closing Notes

Phase 7 is a **major security and usability upgrade**.

It:
- Reduces password sprawl
- Improves auditability
- Simplifies user management
- Sets the stage for Phase 6 enforcement

Once Phase 7 is stable, Phase 6 can be applied with confidence.

---
