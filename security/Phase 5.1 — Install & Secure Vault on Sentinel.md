# 🥇 Phase 5.1 — Installing Vault on Sentinel  
**Practical Implementation Guide**

---

## 📖 Purpose

This phase installs **HashiCorp Vault** as the control plane for workload trust.

Vault is deployed on **Sentinel**, our hardened security node, to:
- Isolate trust infrastructure
- Centralise logging and protection
- Reduce blast radius

---

## 🧠 Why Sentinel (Trade-offs)

**Benefits**
- Hardened host with minimal attack surface  
- Protected by firewall, CrowdSec, and restricted access  
- Clear separation between *trust infrastructure* and *application workloads*  

**Drawbacks**
- Sentinel becomes critical infrastructure  
- Requires backup and recovery discipline  
- Single-node (non-HA) deployment  

This trade-off is **intentional and appropriate** for the current homelab scope.

---

## 1️⃣ Install Vault

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y vault
```

Verify installation:

```bash
vault version
```

---

## 2️⃣ TLS Setup (Mandatory)

Vault **must never run without TLS**, even on an internal network.

Vault tokens are bearer credentials — without TLS, the entire Zero Trust model collapses.

We use a **self-signed certificate with proper SANs**, generated locally on Sentinel.  
This keeps the setup simple, secure, and dependency-free.

---

### 2.1 Create TLS Directory

```bash
sudo mkdir -p /opt/vault/tls
sudo chown vault:vault /opt/vault/tls
sudo chmod 750 /opt/vault/tls
```

---

### 2.2 Create OpenSSL Config with SAN

```bash
sudo nano /tmp/vault-openssl.cnf
```

Paste the following:

```ini
[ req ]
default_bits       = 4096
prompt             = no
default_md         = sha256
distinguished_name = dn
x509_extensions    = v3_req

[ dn ]
C  = ZA
ST = Gauteng
L  = Home
O  = Kagiso Security
CN = vault.local.lannister.com

[ v3_req ]
subjectAltName = @alt_names
keyUsage = keyEncipherment, digitalSignature
extendedKeyUsage = serverAuth

[ alt_names ]
DNS.1 = vault.local.lannister.com
```

---

### 2.3 Generate TLS Key and Certificate

```bash
sudo openssl genrsa -out /opt/vault/tls/tls.key 4096
sudo openssl req -new -x509 \
  -key /opt/vault/tls/tls.key \
  -out /opt/vault/tls/tls.crt \
  -days 365 \
  -config /tmp/vault-openssl.cnf
```

Set correct permissions:

```bash
sudo chown vault:vault /opt/vault/tls/tls.key /opt/vault/tls/tls.crt
sudo chmod 600 /opt/vault/tls/tls.key
sudo chmod 644 /opt/vault/tls/tls.crt
```

---

### 2.4 Verify SAN (Do Not Skip)

```bash
sudo openssl x509 -in /opt/vault/tls/tls.crt -noout -text | grep -A2 "Subject Alternative Name"
```

Expected output:

```
DNS:vault.local.lannister.com
```

If this is missing, Vault **will not work** with modern TLS clients.

---

## 3️⃣ Baseline Vault Configuration

Edit the Vault configuration file:

```bash
sudo nano /etc/vault.d/vault.hcl
```

Use the following configuration:

```hcl
ui = true

listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 0
  tls_cert_file = "/opt/vault/tls/tls.crt"
  tls_key_file  = "/opt/vault/tls/tls.key"
}

storage "file" {
  path = "/opt/vault/data"
}

api_addr = "https://vault.local.lannister.com:8200"
```

Restart Vault:

```bash
sudo systemctl restart vault
sudo systemctl status vault
```

---

## 4️⃣ Trust the TLS Certificate (Recommended)

To allow the Vault CLI and automation tools to trust the certificate:

```bash
sudo cp /opt/vault/tls/tls.crt /usr/local/share/ca-certificates/vault.crt
sudo update-ca-certificates
```

Verify HTTPS connectivity:

```bash
curl https://vault.local.lannister.com:8200/v1/sys/health
```

You should receive JSON output without TLS errors.

---

## 5️⃣ Initialize and Unseal Vault

### 5.1 Initialize (Run Once)

```bash
export VAULT_ADDR=https://vault.local.lannister.com:8200
vault operator init -key-shares=5 -key-threshold=3
```

⚠️ Store unseal keys and the root token **securely and off the server**.

---

### 5.2 Unseal Vault (3 Keys Required)

```bash
vault operator unseal
vault operator unseal
vault operator unseal
```

Verify:

```bash
vault status
```

---

## 6️⃣ Enable Audit Logging (Non-Negotiable)

Audit logs are mandatory for Zero Trust and incident response.

```bash
vault audit enable file file_path=/var/log/vault_audit.log
```

Set permissions:

```bash
sudo touch /var/log/vault_audit.log
sudo chown vault:vault /var/log/vault_audit.log
sudo chmod 600 /var/log/vault_audit.log
```

Verify:

```bash
vault audit list
```

---

## ✅ Phase 5.1 Done When

- Vault is running and unsealed  
- TLS is enforced with SAN-correct certificate  
- HTTPS works without `-k`  
- Audit logging is enabled  
- Port 8200 is not publicly exposed  

Vault is now a **secure trust foundation**, ready for Kubernetes integration in Phase 5.2.
