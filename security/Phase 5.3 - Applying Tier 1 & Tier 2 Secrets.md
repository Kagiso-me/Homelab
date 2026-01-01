# 🥇 Phase 5.3 — PostgreSQL Bootstrap & Application Secrets 
**Why we bootstrap. Why Vault comes later. How it all fits together.**

---

> This section exists because most Vault guides get this *wrong*.
>
> They either:
> - try to put PostgreSQL passwords into Vault from day one, or
> - fail to explain *why* certain secrets must never depend on Vault.
>
> This guide fixes that.

---

## 🧭 Where Phase 5.3 Fits in the Bigger Picture

You have already completed:

- **Phase 4** — Human identity & Zero Trust (Authentik + Traefik)
- **Phase 5.1** — Vault installed and secured on Sentinel
- **Phase 5.2** — Kubernetes ↔ Vault trust established

Phase **5.3** answers one critical question:

> **How do we actually use Vault for real applications — without breaking our cluster?**

---

## 🧠 The Core Principle (Read This Twice)

> **Infrastructure must be able to start without Vault.  
> Applications may depend on Vault.**

---

## 🧱 Part 1 — Why PostgreSQL Is Bootstrapped Outside Vault

PostgreSQL is infrastructure, not an application.

It must:
- start independently
- survive Vault downtime
- avoid circular dependencies

Therefore:
- PostgreSQL **must not** depend on Vault to start
- PostgreSQL uses a **bootstrap secret outside Vault**

This is correct architecture, not a compromise.

---

## 🔐 Part 2 — PostgreSQL Bootstrap Secret (Tier 0)

### Create the bootstrap secret

```bash
kubectl create namespace apps
```

```bash
kubectl create secret generic postgres-bootstrap \
  -n apps \
  --from-literal=admin-password='<STRONG_ADMIN_PASSWORD>' \
  --from-literal=password='<STRONG_APP_OWNER_PASSWORD>' \
  --from-literal=replication-password='<STRONG_REPL_PASSWORD>'
```

This secret is:
- used only at database initialization
- never read by applications
- not managed by Vault

---

## 🔒 Locking It Down with RBAC

Only PostgreSQL may read this secret.

```bash
kubectl create serviceaccount postgresql -n apps
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: postgres-bootstrap-reader
  namespace: apps
rules:
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["postgres-bootstrap"]
  verbs: ["get"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: postgres-bootstrap-binding
  namespace: apps
subjects:
- kind: ServiceAccount
  name: postgresql
roleRef:
  kind: Role
  name: postgres-bootstrap-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 🔐 Part 3 — Why Applications Never Use PostgreSQL Secrets

Application pods:
- restart frequently
- scale dynamically
- are untrusted by default

Sharing PostgreSQL credentials would:
- increase blast radius
- prevent rotation
- remove auditability

Vault exists to solve *this* problem.

---

## 🔑 Part 4 — Application Secrets via Vault (Nextcloud)

### Enable KV engine

```bash
vault secrets enable -path=kv kv-v2
```

---

### Store app credentials

```bash
vault kv put kv/nextcloud/db \
  username="nextcloud_app" \
  password="strong-app-password"
```

---

### Create Vault policy

```bash
vault policy write nextcloud-db - <<EOF
path "kv/data/nextcloud/db" {
  capabilities = ["read"]
}
EOF
```

---

### Bind policy to Kubernetes identity

```bash
vault write auth/kubernetes/role/nextcloud \
  bound_service_account_names=nextcloud \
  bound_service_account_namespaces=apps \
  policies=nextcloud-db \
  ttl=1h
```

---

### Inject secrets into Nextcloud

```yaml
metadata:
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "nextcloud"
    vault.hashicorp.com/agent-inject-secret-db: "kv/data/nextcloud/db"
    vault.hashicorp.com/agent-inject-template-db: |
      {{- with secret "kv/data/nextcloud/db" -}}
      export DB_USER="{{ .Data.data.username }}"
      export DB_PASS="{{ .Data.data.password }}"
      {{- end }}
```

---

## ✅ Phase 5.3 Complete When

- PostgreSQL boots without Vault
- Bootstrap secret is RBAC-restricted
- Apps never read PostgreSQL secrets
- Vault injects app secrets correctly
- Access is logged and auditable
