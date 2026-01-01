# 🥇 Phase 5.3 — Tiered Secret Consumption (Practical Implementation)
**From Identity → Authorization → Real Secrets**

This phase builds directly on **Phase 5.2**, where:
- Kubernetes identities were verified
- Vault roles and policies were enforced
- No secrets were yet exposed

Phase 5.3 is where Vault begins delivering **real value**:
secure, auditable, short‑lived secrets consumed by workloads.

We deliberately split secret usage into **two tiers**, based on risk and operational reality.

---

## 🧠 The Tiered Secret Model (Why This Exists)

Not all secrets are equal.

| Tier | Secret Type | Characteristics | Examples |
|----|----|----|----|
| **Tier 1** | Static credentials | Low churn, infra-bound | DB passwords |
| **Tier 2** | Dynamic / high-risk | Rotated, identity-bound | App auth tokens |

This allows:
- gradual adoption
- app compatibility
- reduced blast radius
- operational sanity

---

## 🔐 Tier 1 — Static Secrets (PostgreSQL Example)

### What Tier 1 Is (and Is Not)

Tier 1:
- replaces Kubernetes Secrets
- centralizes sensitive config
- improves auditability

Tier 1 **does not yet**:
- rotate automatically
- issue per-pod credentials

This is intentional.

---

## 1️⃣ Enable KV Secrets Engine

On Sentinel:

```bash
vault secrets enable -path=kv kv-v2
```

Verify:

```bash
vault secrets list
```

---

## 2️⃣ Store PostgreSQL Credentials

Example PostgreSQL credentials:

```bash
vault kv put kv/postgres/app \
  username="app_user" \
  password="supersecretpassword"
```

---

## 3️⃣ Create a Vault Policy for PostgreSQL Access

```bash
vault policy write postgres-read - <<EOF
path "kv/data/postgres/app" {
  capabilities = ["read"]
}
EOF
```

---

## 4️⃣ Bind Policy to a Kubernetes Role

```bash
vault write auth/kubernetes/role/postgres-app \
  bound_service_account_names=postgres-sa \
  bound_service_account_namespaces=apps \
  policies=postgres-read \
  ttl=1h
```

---

## 5️⃣ Kubernetes Deployment Example (Tier 1)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-client
  namespace: apps
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres-client
  template:
    metadata:
      labels:
        app: postgres-client
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "postgres-app"
        vault.hashicorp.com/agent-inject-secret-db: "kv/data/postgres/app"
        vault.hashicorp.com/agent-inject-template-db: |
          {{- with secret "kv/data/postgres/app" -}}
          export DB_USER="{{ .Data.data.username }}"
          export DB_PASS="{{ .Data.data.password }}"
          {{- end }}
    spec:
      serviceAccountName: postgres-sa
      containers:
      - name: app
        image: postgres:16
        command: ["sh", "-c", "sleep 3600"]
```

---

## 🔐 Tier 2 — Dynamic Secrets (Nextcloud Example)

---

## 6️⃣ Enable Database Secrets Engine

```bash
vault secrets enable database
```

---

## 7️⃣ Configure PostgreSQL Connection

```bash
vault write database/config/postgres \
  plugin_name=postgresql-database-plugin \
  allowed_roles="nextcloud" \
  connection_url="postgresql://{{username}}:{{password}}@postgres.apps.svc.cluster.local:5432/nextcloud?sslmode=disable" \
  username="vault_admin" \
  password="vault_admin_password"
```

---

## 8️⃣ Create Dynamic DB Role

```bash
vault write database/roles/nextcloud \
  db_name=postgres \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT ALL PRIVILEGES ON DATABASE nextcloud TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"
```

---

## 9️⃣ Nextcloud Policy

```bash
vault policy write nextcloud-db - <<EOF
path "database/creds/nextcloud" {
  capabilities = ["read"]
}
EOF
```

---

## 🔟 Nextcloud Deployment Snippet

```yaml
metadata:
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "nextcloud"
    vault.hashicorp.com/agent-inject-secret-db: "database/creds/nextcloud"
    vault.hashicorp.com/agent-inject-template-db: |
      {{- with secret "database/creds/nextcloud" -}}
      export DB_USER="{{ .Data.username }}"
      export DB_PASS="{{ .Data.password }}"
      {{- end }}
```

---

## 🧾 Audit & Validation

```bash
sudo tail -f /var/log/vault_audit.log
```

---

## ✅ Phase 5.3 Done When

- Tier 1 static secrets work
- Tier 2 dynamic credentials rotate
- No Kubernetes Secrets used
- Vault audit logs populated
