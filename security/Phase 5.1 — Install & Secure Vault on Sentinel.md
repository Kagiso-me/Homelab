# 🥇 Phase 5.1 — Installing Vault on Sentinel  
**Practical Implementation Guide**

---

## 📖 Purpose

This phase installs **HashiCorp Vault** as the control plane for workload trust.

Vault is deployed on **Sentinel**, our hardened security node, to:
- Isolate trust infrastructure
- Centralise logging and protection
- Reduce blast radius

This guide focuses on installation and security only.

---

## 🧠 Why Sentinel (Trade-offs)

**Benefits**
- Hardened host
- Protected by firewall and CrowdSec
- Separate from application workloads

**Drawbacks**
- Sentinel becomes critical infrastructure
- Requires backup discipline
- Not horizontally scalable by default

This is acceptable for the current environment.

---

## 1️⃣ Install Vault

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y vault
```

Verify:
```bash
vault version
```

---

## 2️⃣ Baseline Configuration

`/etc/vault.d/vault.hcl`

```hcl
ui = true

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 0
  tls_cert_file = "/etc/vault.d/tls/vault.crt"
  tls_key_file  = "/etc/vault.d/tls/vault.key"
}

storage "file" {
  path = "/opt/vault/data"
}

api_addr = "https://vault.example.com:8200"
```

TLS is mandatory.

---

## 3️⃣ Initialize & Unseal

```bash
export VAULT_ADDR=https://127.0.0.1:8200
vault operator init -key-shares=5 -key-threshold=3
vault operator unseal
vault operator unseal
vault operator unseal
```

Store unseal keys and root token **offline**.

---

## 4️⃣ Harden Access

- Restrict port 8200 to cluster / Traefik only
- No WAN exposure
- Enable audit logging

Vault is now ready.

---

## ✅ Phase 5.1 Done When

- Vault running and unsealed
- TLS enforced
- Audit logging enabled
- Network access restricted

---
