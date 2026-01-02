# Phase 6 — Identity, Access & Authentication (Practical Guide)

## Purpose of Phase 6

Phase 6 introduces **human identity and access control** into the platform.

Up to this point:
- Machines are secured (Vault + ESO)
- Traffic is encrypted (TLS)
- Visibility and alerting exist

What is still missing is **who is allowed to access what**, in a consistent way.

> **Goal of Phase 6**
>
> Provide a *central, understandable, and optional* identity layer for humans  
> **without** breaking apps, **without** overengineering, and **without** tight coupling.

This phase is **deliberately pragmatic**.

---

## Security Level

#### **Security Level:** 8.5 / 10  
*(Improves access control and reduces credential sprawl)*

---

## What Phase 6 Is — and Is Not

### Phase 6 **IS**
- Central login (SSO)
- Identity Provider (IdP)
- Optional MFA (future-ready)
- Cleaner auth for weak apps
- Edge-based authentication (Traefik)

### Phase 6 **IS NOT**
- Runtime enforcement
- NetworkPolicies
- Pod security lockdown
- Mandatory for every app

Phase 6 is **opt-in per application**.

---

## Architecture Overview (Simple)

```
User
  ↓
Traefik (Ingress)
  ↓
Authentik (Identity Provider)
  ↓
Application
```

Important:
- Apps **do not** store passwords
- Apps **delegate authentication**
- Traefik can enforce auth *before* traffic reaches apps

---

## Why Authentik?

Authentik is chosen because it:
- Is self-hosted
- Supports OIDC, OAuth2, SAML
- Integrates cleanly with Traefik
- Has good UX
- Scales from simple → advanced

Alternatives exist, but Authentik strikes the best balance here.

---

## Deployment Model

Authentik will run:
- Inside Kubernetes
- Behind Traefik
- With TLS via cert-manager
- With secrets sourced from Vault (via ESO)

Core components:
- authentik-server
- authentik-worker
- PostgreSQL (existing or dedicated)
- Redis (recommended)

---

## How Apps Integrate (Two Models)

This is the most important part.

---

### Model 1 — App-Native SSO (Preferred)

Some apps support OIDC directly (e.g. Nextcloud).

Flow:
```
User → App → Authentik → App
```

Steps:
1. Create OIDC provider in Authentik
2. Register application
3. Configure redirect URI
4. Store client secret in Vault
5. Configure app to use OIDC

Pros:
- Clean
- App-aware users
- Best UX

Cons:
- App must support OIDC

---

### Model 2 — Traefik Forward Authentication

For apps that **do not support SSO**.

Flow:
```
User → Traefik → Authentik → App
```

Traefik checks identity *before* forwarding traffic.

Example middleware concept:
```
forwardAuth:
  address: https://auth.kagiso.me/outpost.goauthentik.io/auth/traefik
```

Pros:
- Works with almost any app
- No app changes

Cons:
- App does not know user identity deeply

---

## Which Model Should You Use?

| App Type | Recommended |
|-------|-------------|
Modern apps (Nextcloud) | OIDC |
Internal tools | Forward-auth |
Admin dashboards | Forward-auth |
Public apps | App-native SSO |

You can mix both models safely.

---

## Day-1 Recommended Scope

To avoid complexity:

- Start with **one app only**
- Do not enable MFA yet
- Do not protect everything
- Validate login flow end-to-end

Nextcloud is a good first candidate.

---

## Operational Considerations

- Authentik is now a **critical service**
- If Authentik is down:
  - Forward-auth protected apps are inaccessible
- Keep ingress access minimal
- Backups will be handled in Phase 8

---

## When Phase 6 Is “Done”

Phase 6 is considered complete when:

- [ ] Authentik is reachable over TLS
- [ ] Admin login works
- [ ] One app uses SSO successfully
- [ ] Secrets are Vault-backed
- [ ] Traefik integration is tested
- [ ] Documentation updated

---

## What Comes Next

After Phase 6:
- Phase 7 — Backup, Recovery & Integrity
- Auxiliary — OpenCanary
- Optional future hardening (Suricata, etc.)


---
