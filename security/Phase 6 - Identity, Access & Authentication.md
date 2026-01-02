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

## Secrets Strategy (Important)

We **do not hardcode secrets**.

All secrets live in Vault:
```
kv/apps/authentik
```

Example secrets:
- AUTHENTIK_SECRET_KEY
- POSTGRES_PASSWORD
- REDIS_PASSWORD
- AUTHENTIK_BOOTSTRAP_PASSWORD

ESO syncs these into Kubernetes Secrets.

This keeps Phase 6 consistent with Phase 5.

---

## Step-by-Step: Installing Authentik

### 1. Prepare Vault Secrets

On the Vault host:

```
vault kv put kv/apps/authentik \
  secret_key=<random-64-char-string> \
  postgres_password=<strong-password> \
  redis_password=<strong-password> \
  bootstrap_password=<initial-admin-password>
```

---

### 2. Create ExternalSecret

This exposes Vault secrets to Kubernetes:

```
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: authentik-secrets
  namespace: authentik
spec:
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: authentik-secrets
  data:
    - secretKey: AUTHENTIK_SECRET_KEY
      remoteRef:
        key: kv/apps/authentik
        property: secret_key

    - secretKey: AUTHENTIK_BOOTSTRAP_PASSWORD
      remoteRef:
        key: kv/apps/authentik
        property: bootstrap_password
```

---

### 3. Install Authentik via Helm

Add repo:

```
helm repo add authentik https://charts.goauthentik.io
helm repo update
```

Install:

```
helm install authentik authentik/authentik \
  -n authentik --create-namespace \
  -f values.yaml
```

Key values to set:
- Use existing secret (`authentik-secrets`)
- Disable embedded secrets
- Configure external DB if needed

---

## Ingress & TLS Integration (Traefik)

Authentik is exposed via Traefik:

```
https://auth.kagiso.me
```

Key points:
- cert-manager handles certificates
- Traefik terminates TLS
- Authentik must be reachable by Traefik

This is a **normal ingress**, nothing special yet.

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

## Common Mistakes to Avoid

- Protecting all apps immediately
- Enabling MFA too early
- Hardcoding secrets
- Mixing enforcement into this phase

---

## Final Notes

Phase 6 is about **control and clarity**, not maximal lockdown.

If it ever feels heavy:
- Pause
- Reduce scope
- Make it optional again

This phase should **help you**, not fight you.

---

## What Comes Next

After Phase 6:
- Phase 8 — Backup, Recovery & Integrity
- Auxiliary — OpenCanary
- Optional future hardening (Suricata, etc.)

Phase 7 (runtime enforcement) remains intentionally scrapped.

---

End of Phase 6 Guide
