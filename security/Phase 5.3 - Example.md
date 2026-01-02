# Static Vault Secrets with External Secrets Operator (ESO)

This document supplements the Phase 5 guide and clarifies exactly how static secrets flow from Vault to your applications (e.g. Nextcloud) using External Secrets Operator (ESO).

Key principle:
Applications never talk to Vault directly.
Vault is the source of truth, ESO is the sync mechanism, and apps only consume normal Kubernetes Secrets.

---

## 1. Mental Model

Vault (static secret)
        ↓
External Secrets Operator (ESO)
        ↓
Kubernetes Secret
        ↓
Application (Nextcloud, etc.)

- The application does not authenticate to Vault
- The application does not need Vault libraries or tokens
- Vault is not a runtime dependency for the app
- ESO handles authentication, fetching, and syncing

---

## 2. Create a Static Secret in Vault

On the Vault host (sentinel):

```
vault kv put kv/apps/nextcloud \
  db_name=nextcloud \
  db_user=nextcloud \
  db_password=supersecretpassword \
  redis_password=redispassword
```

Vault is now the single source of truth.

---

## 3. Define an ExternalSecret (Vault → Kubernetes)

```
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: nextcloud-secrets
  namespace: nextcloud
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: nextcloud-secrets
    creationPolicy: Owner
  data:
    - secretKey: POSTGRES_DB
      remoteRef:
        key: kv/apps/nextcloud
        property: db_name

    - secretKey: POSTGRES_USER
      remoteRef:
        key: kv/apps/nextcloud
        property: db_user

    - secretKey: POSTGRES_PASSWORD
      remoteRef:
        key: kv/apps/nextcloud
        property: db_password

    - secretKey: REDIS_PASSWORD
      remoteRef:
        key: kv/apps/nextcloud
        property: redis_password
```

ESO will authenticate to Vault, read the secret, and create a Kubernetes Secret named nextcloud-secrets.

---

## 4. Reference the Secret in Your Deployment

Example using environment variables:

```
env:
  - name: POSTGRES_DB
    valueFrom:
      secretKeyRef:
        name: nextcloud-secrets
        key: POSTGRES_DB

  - name: POSTGRES_USER
    valueFrom:
      secretKeyRef:
        name: nextcloud-secrets
        key: POSTGRES_USER

  - name: POSTGRES_PASSWORD
    valueFrom:
      secretKeyRef:
        name: nextcloud-secrets
        key: POSTGRES_PASSWORD
```

Example using Helm values:

```
externalDatabase:
  enabled: true
  type: postgresql
  existingSecret:
    enabled: true
    secretName: nextcloud-secrets
    usernameKey: POSTGRES_USER
    passwordKey: POSTGRES_PASSWORD
```

---

## 5. Secret Rotation

Rotate the secret in Vault:

```
vault kv put kv/apps/nextcloud \
  db_password=newpassword
```

ESO updates the Kubernetes Secret, but pods must be restarted manually:

```
kubectl rollout restart deployment nextcloud -n nextcloud
```

---

## 6. Why Static Secrets Are Recommended

Static secrets are ideal for:
- Stateful apps like Nextcloud
- Databases with long-lived connections
- Apps that cannot reload credentials dynamically
- Predictable and controlled rotations

---

## 7. Summary

Static Vault secrets + ESO provide secure, simple, and predictable secret management.
Applications remain unaware of Vault and consume standard Kubernetes Secrets only.
