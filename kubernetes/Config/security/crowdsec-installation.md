# CrowdSec + RPi Edge Firewall Deployment Guide

## 🛡️ Installation & Deployment Guide (RPi Edge + Cluster Protection)

```scss
🌐 Internet
   ↓
📡 Router (port forward 80/443 → RPi)
   ↓
🖥️ Raspberry Pi 3B
   ├─ 🔹 CrowdSec Agent (Detection + LAPI)
   └─ 🔹 Firewall Bouncer (nftables, Layer 3/4 enforcement)
   ↓
☸️ k3s Cluster (Traefik Ingress)
   ↓
📦 Apps: Immich | Nextcloud | Jellyfin
```

---

### Step 1: Prepare & Harden the Raspberry Pi

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install nftables -y
sudo systemctl enable nftables
sudo systemctl start nftables
```

* Set a **static IP** for the Pi.
* Harden SSH (key-only login, no root password).
* Apply baseline **nftables** rules: allow SSH from LAN, HTTP/HTTPS to cluster, block everything else.

| Traffic             | Allowed? | Why                   |
| ------------------- | -------- | --------------------- |
| SSH (22) from LAN   | ✅        | Admin access          |
| HTTP/HTTPS (80/443) | ✅        | ACME + app traffic    |
| Everything else     | ❌        | Reduce attack surface |

#### nftables rules

```nft
#!/usr/sbin/nft -f
flush ruleset

table inet filter {
  chain input {
    type filter hook input priority 0;
    policy drop;

    iif lo accept
    ct state established,related accept
    ip saddr 10.0.10.0/24 tcp dport 22 accept
    tcp dport {80,443} accept
    ip protocol icmp accept
  }

  chain forward { type filter hook forward priority 0; policy drop; }
  chain output  { type filter hook output  priority 0; policy accept; }
}
```

Apply & verify:

```bash
sudo nft -f /etc/nftables.conf
sudo nft list ruleset
```

---

### Step 2: Install CrowdSec Agent (Detection + LAPI)

```bash
# Add CrowdSec repo
curl -s https://packagecloud.io/install/repositories/crowdsec/crowdsec/script.deb.sh | sudo bash
sudo apt install crowdsec -y
sudo systemctl enable crowdsec
sudo systemctl start crowdsec
```

Minimal config (`/etc/crowdsec/config.yaml`):

```yaml
log:
  level: info
  dir: /var/log/crowdsec

daemonize: true
pid_dir: /var/run/
api:
  listen_uri: "http://127.0.0.1:8080"
```

Start agent:

```bash
sudo systemctl restart crowdsec
sudo cscli metrics
```

---

### Step 3: Install Firewall Bouncer (Layer 3/4 Enforcement)

```bash
<<<<<<< HEAD
sudo apt install crowdsec-firewall-bouncer -y
sudo systemctl enable crowdsec-firewall-bouncer
=======
sudo apt install crowdsec-firewall-bouncer
>>>>>>> 30fc3aea4738a8af74f670ccff58c4d37f542d5c
```

Generate API key:

```bash
sudo cscli bouncers add rpi-bouncer
```

Configure bouncer (`/etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml`):

```yaml
mode: nftables
api_key: "<API_KEY_FROM_CSCLI>"
api_url: "http://127.0.0.1:8080/"
pid_dir: /var/run/
update_frequency: 10s
daemonize: true
log_mode: file
log_dir: /var/log/
log_level: info
deny_action: DROP
supported_decisions_types:
  - ban

nftables:
  ipv4:
    enabled: true
    set-only: false
    table: crowdsec
    chain: crowdsec-chain
```

Start bouncer:

```bash
sudo systemctl restart crowdsec-firewall-bouncer
sudo journalctl -u crowdsec-firewall-bouncer -f
sudo cscli decisions list
```

---

### Step 4: Forward Traffic to k3s Cluster

* Router: forward **80/443 → Pi IP**
* NAT on Pi to pass allowed traffic to Traefik NodePort:

```nft
table ip nat {
  chain prerouting {
    type nat hook prerouting priority 0;
    tcp dport {80,443} dnat to <cluster-ip>:<nodeport>
  }
  chain postrouting {
    type nat hook postrouting priority 100;
    oif <lan-interface> masquerade
  }
}
```

* CrowdSec automatically drops malicious IPs.

---

### Step 5: Configure Logs & Detection

* Traefik logs JSON for detection:

```yaml
additionalArguments:
  - "--accesslog=true"
  - "--accesslog.format=json"
```

* Detects brute force, scanners, rate abuse.

---

### Step 6: Test & Validate

1. Trigger failed login → CrowdSec bans IP.
2. Check decisions & rules:

```bash
sudo cscli decisions list
sudo nft list ruleset
```

3. Confirm legitimate uploads work.

---

### Step 7: Optional Authentik (Layer 7 Access Control)

* Authentik handles **SSO, MFA, user authentication**.
* Lightweight SQLite recommended for Pi 3B.

---

### Step 8: Monitoring & Maintenance

* Metrics: `node_exporter` → Prometheus
* Optional: Uptime Kuma
* Weekly updates:

```bash
sudo apt update && sudo apt upgrade -y
cscli update
cscli hub update
```

* Backup configs:

```bash
sudo cp /etc/nftables.conf ~/nftables.conf.backup
sudo cp /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml ~/bouncer.yaml.backup
```

| Task                | Why                   |
| ------------------- | --------------------- |
| OS updates          | Kernel & nft fixes    |
| CrowdSec hub update | New attack signatures |
| Backup configs      | Disaster recovery     |

---

### 🌟 Architecture Diagram

```
🌐 Internet
   │
📡 Router (port forward 80/443 → RPi)
   │
🖥️ Raspberry Pi 3B
 ├─ 🔹 CrowdSec Agent + LAPI
 └─ 🔹 Firewall Bouncer (nftables)
   │
☸️ k3s Cluster (Traefik Ingress)
   │
📦 Apps: Immich | Nextcloud | Jellyfin
```

* **Pi**: detection + network enforcement
* **Cluster**: apps only
* **Traffic flow**: malicious IPs blocked before hitting apps
* **Authentik**: optional Layer 7 auth only
