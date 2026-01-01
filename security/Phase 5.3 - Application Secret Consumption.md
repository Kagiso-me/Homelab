# Phase 5.3 — Application Secret Consumption

## Purpose

This phase defines **how applications consume secrets** in the platform.

Applications consume **only Kubernetes Secrets**.
Vault is invisible to workloads.

---

## Approved Secret Flow

```
Vault (kv/)
   ↓
External Secrets Operator
   ↓
Kubernetes Secret
   ↓
Application
```

---

## Example — Nextcloud

### Secrets in Vault

Path:
```
kv/apps/nextcloud
```

---

### ExternalSecret

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: nextcloud-secrets
  namespace: apps
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: nextcloud-secrets
    creationPolicy: Owner
  data:
    - secretKey: db-username
      remoteRef:
        key: kv/apps/nextcloud
        property: db_user
    - secretKey: db-password
      remoteRef:
        key: kv/apps/nextcloud
        property: db_password
```

---

### Helm Configuration

```yaml
externalDatabase:
  enabled: true
  existingSecret:
    name: nextcloud-secrets
    usernameKey: db-username
    passwordKey: db-password
```

---

## Phase Outcome

At the end of Phase 5.3:

- Applications consume Kubernetes Secrets only
- Vault is abstracted away
- No sidecars exist
