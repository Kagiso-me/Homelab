# Phase 6.1 — Deploy Authentik (Hands‑On Implementation)

This guide is **pure execution**.  
No theory, no alternatives, no bikeshedding.

You should already have:
- Traefik installed and working
- cert-manager installed
- Vault + External Secrets Operator working
- A public domain: **auth.kagiso.me**

---

## Phase 6.1 Goal

By the end of this phase:

- Authentik is running in Kubernetes
- Reachable at **https://auth.kagiso.me**
- TLS secured via Let’s Encrypt
- All secrets sourced from Vault
- Admin login works

Nothing else.  
No apps yet. No MFA yet.

---

## Architecture (Concrete)

```
Browser
   ↓
https://auth.kagiso.me
   ↓
Traefik (Ingress)
   ↓
Authentik (server + worker)
```

---

## 1. Namespace

```
kubectl create namespace authentik
```

---

## 2. Vault Secrets (Authoritative)

Create secrets **on the Vault host**:

```
vault kv put kv/apps/authentik \
  secret_key="$(openssl rand -hex 32)" \
  bootstrap_password="CHANGE_ME_STRONG" \
  postgres_password="CHANGE_ME_STRONG" \
  redis_password="CHANGE_ME_STRONG"
```

---

## 3. ExternalSecret (ESO)

This syncs Vault → Kubernetes.

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

    - secretKey: POSTGRES_PASSWORD
      remoteRef:
        key: kv/apps/authentik
        property: postgres_password

    - secretKey: REDIS_PASSWORD
      remoteRef:
        key: kv/apps/authentik
        property: redis_password
```

Apply:

```
kubectl apply -f authentik-externalsecret.yaml
```

Verify:

```
kubectl get secret authentik-secrets -n authentik
```

---

## 4. Helm Repository

```
helm repo add authentik https://charts.goauthentik.io
helm repo update
```

---

## 5. values.yaml (Minimal & Safe)

Create `values.yaml`:

```
authentik:
  secret_key: ""
  existingSecret: authentik-secrets

server:
  ingress:
    enabled: true
    hosts:
      - auth.kagiso.me
    annotations:
      kubernetes.io/ingress.class: traefik
      cert-manager.io/cluster-issuer: letsencrypt-prod
    tls:
      - secretName: authentik-tls
        hosts:
          - auth.kagiso.me

postgresql:
  enabled: true
  auth:
    existingSecret: authentik-secrets
    secretKeys:
      adminPasswordKey: POSTGRES_PASSWORD

redis:
  enabled: true
  auth:
    enabled: true
    existingSecret: authentik-secrets
    existingSecretPasswordKey: REDIS_PASSWORD
```

---

## 6. Install Authentik

```
helm install authentik authentik/authentik \
  -n authentik \
  -f values.yaml
```

Watch pods:

```
kubectl get pods -n authentik -w
```

---

## 7. Verify TLS & Ingress

```
kubectl get ingress -n authentik
kubectl describe certificate -n authentik
```

Wait until certificate is **Ready=True**.

---

## 8. First Login

Open:

```
https://auth.kagiso.me
```

Login:
- **Username:** akadmin
- **Password:** value of `bootstrap_password`

Immediately:
- Change admin password
- Verify dashboard loads

---

## 9. Validation Checklist (Required)

Phase 6.1 is **not complete** until:

- [ ] auth.kagiso.me loads over HTTPS
- [ ] Valid Let’s Encrypt certificate
- [ ] Admin login works
- [ ] Pods stable (no crashloops)
- [ ] Secrets NOT visible in Helm values
- [ ] Vault remains source of truth

---

## What We Do NOT Do Yet

- ❌ No app integrations
- ❌ No forward-auth
- ❌ No MFA
- ❌ No policy tuning

Those belong to:
- Phase 6.2 — App Integration (Nextcloud)
- Phase 6.3 — Forward Auth

---

## Troubleshooting Quick Hits

- **Pods crashlooping**
  - Check missing secrets
  - Check Redis/Postgres auth keys

- **TLS not issuing**
  - Check cert-manager logs
  - Confirm DNS for auth.kagiso.me

- **Login fails**
  - Confirm bootstrap password
  - Restart server pod after secret sync

---

## Phase 6.1 Status

🚧 In progress  
✅ Defined  
➡️ Ready to execute  

Once complete, move to:

👉 **Phase 6.2 — Nextcloud OIDC Integration**

---

End of Phase 6.1
