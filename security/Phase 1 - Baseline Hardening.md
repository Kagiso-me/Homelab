# 🥉 Phase 1 — Baseline Hardening (Security Level: 3/10)

This guide covers **host-level and edge security hardening** for a self-hosted Linux environment.
It intentionally **excludes Kubernetes-specific steps**, which are documented in a separate guide.

---

## 🎯 Goal of Phase 1

Establish strong **baseline security hygiene** with minimal operational complexity:

- Reduce attack surface
- Enforce least privilege
- Eliminate common misconfigurations
- Make compromise harder and noisier
- Prepare the foundation for detection and monitoring

This phase focuses on **servers, SSH, firewalls, networking, TLS, logging, and backups**.

---

## ✅ What “Done” Looks Like

By the end of this phase:

- SSH is hardened and key-only
- Root login is disabled
- Firewall is deny-by-default
- Only required ports are exposed
- TLS is enforced everywhere
- Logs exist and are retained
- Backups are verified and recoverable

---

## 1️⃣ Operating System Hardening

### 1.1 Update the System

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install unattended-upgrades -y
```

Enable automatic security updates:

```bash
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

---

### 1.2 Create a Non-Root Admin User

```bash
sudo adduser kagiso
sudo usermod -aG sudo kagiso
```

Lock the root account:

```bash
sudo passwd -l root
```

---

## 2️⃣ SSH Hardening (Critical)

Edit `/etc/ssh/sshd_config`:

```conf
Port 22
Protocol 2
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes

X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no

ClientAliveInterval 300
ClientAliveCountMax 2
MaxAuthTries 3
LoginGraceTime 20
```

Restart SSH:

```bash
sudo systemctl restart sshd
```

**Result:**  
- No password authentication  
- No root login  
- Reduced brute-force and lateral attack risk  

---

## 3️⃣ Firewall Hardening (UFW)

### 3.1 Install UFW

```bash
sudo apt install ufw -y
```

### 3.2 Baseline Rules

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP
sudo ufw allow 443/tcp   # HTTPS

# Allow internal LAN traffic (adjust as needed)
sudo ufw allow from 10.0.0.0/8

sudo ufw enable
```

Verify:

```bash
sudo ufw status verbose
```

**Important:**  
Do NOT expose:
- Databases
- Admin dashboards
- Monitoring endpoints
- Internal APIs

---

## 4️⃣ Network Exposure Rules

### 4.1 Router / NAT Configuration

Only forward:
- TCP 80 → Reverse proxy
- TCP 443 → Reverse proxy

Never forward:
- SSH (22)
- Database ports
- Internal management services

Verify externally:

```bash
nmap -Pn your.public.ip
```

---

## 5️⃣ TLS Everywhere

### 5.1 HTTPS Enforcement

Ensure all services redirect HTTP → HTTPS.

Recommended:
- HSTS enabled
- TLS 1.2+ only
- Valid certificates (Let’s Encrypt or internal CA)

### 5.2 Disable Weak Ciphers

At the reverse proxy level:
- Disable TLS 1.0 / 1.1
- Prefer ECDHE + AES-GCM or ChaCha20

---

## 6️⃣ Secrets Hygiene (Host-Level)

### 6.1 Avoid Plaintext Secrets

Avoid:
- Passwords in shell history
- Credentials in config files
- Secrets committed to Git

Prefer:
- `.env` files with restricted permissions
- Environment variables
- OS-level secret stores

Example:

```bash
chmod 600 .env
```

### 6.2 Git Hygiene

Add to `.gitignore`:

```gitignore
.env
*.env
secrets/
values-prod.yaml
```

---

## 7️⃣ Logging & Time

### 7.1 Persistent Logging

Ensure logs are retained:

```bash
journalctl --disk-usage
```

Confirm log rotation is enabled.

### 7.2 Time Synchronization

```bash
timedatectl set-ntp true
```

Accurate time is **critical for security investigations**.

---

## 8️⃣ Fail2Ban (Optional but Recommended)

Install:

```bash
sudo apt install fail2ban -y
```

Check status:

```bash
sudo fail2ban-client status
```

Protects:
- SSH
- Auth endpoints

---

## 9️⃣ Backups (Non-Negotiable)

### 9.1 System Configuration

Back up at minimum:
- `/etc`
- Reverse proxy configs
- Application configuration directories

### 9.2 Application Data

Ensure:
- Databases are dumped regularly
- User uploads are backed up
- Backups are stored off-host

Test restores regularly.

> Untested backups do not exist.

---

## 🧾 Phase 1 Checklist

- [ ] OS fully updated
- [ ] Root login disabled
- [ ] SSH key-only access
- [ ] Firewall deny-by-default
- [ ] Only 80/443 exposed publicly
- [ ] TLS enforced everywhere
- [ ] Secrets removed from repos
- [ ] Logs retained
- [ ] Time synchronized
- [ ] Backups tested

---

## ⏭️ Next Steps

- **Phase 2 — Detection & Monitoring**
  - Wazuh
  - Suricata
  - Centralized logging
  - Alerting

Kubernetes hardening is intentionally documented separately.

---

**Security Level Achieved:** ⭐⭐⭐☆☆☆☆☆☆☆ (3/10)  
Solid foundation. Ready for visibility tooling.
