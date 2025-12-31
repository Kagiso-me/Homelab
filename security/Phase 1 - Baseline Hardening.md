# 🥉 Phase 1 — Baseline Hardening
#### Security Level: 3/10

This phase establishes **foundational security controls** for the **most critical node in the environment**: the **edge node**.

---

## 🧭 Environment Context (Read This First)

Before applying any commands, it’s important to understand **what is being hardened, where, and why**.

### 🛡️ The Sentinel Node (Edge / Gateway)

**Sentinel** is a **Raspberry Pi** that functions as the **edge node** and primary security boundary for the environment.

It serves as:

- Internet-facing entry point  
- Reverse proxy for HTTP/HTTPS traffic  
- Firewall and traffic enforcement boundary  
- Log and security telemetry intake node  
- The **only host directly exposed to the public internet**

> Every external request enters the environment through Sentinel.  
> If Sentinel is compromised, all downstream systems become reachable.

This makes Sentinel:
- The **highest-risk node**
- The **highest-value target**
- The **first place attackers probe**

---

**Phase 1 focuses exclusively on host-level and edge hardening**, because:

- Detection without prevention creates noise
- Kubernetes / Docker security is meaningless if the edge is weak
- Most real-world breaches begin with exposed services, SSH, or firewall misconfigurations

---

## 🎯 Goal of Phase 1

Establish strong **baseline security hygiene** with minimal operational complexity:

- Reduce attack surface
- Enforce least privilege
- Eliminate common misconfigurations
- Make compromise harder and noisier
- Prepare the foundation for monitoring and detection

This phase focuses on:

- Operating system hardening
- SSH security
- Firewall configuration
- Network exposure
- TLS enforcement
- Logging and time integrity
- Backups

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

## 5️⃣ Logging & Time

### 5.1 Persistent Logging

Ensure logs are retained:

```bash
journalctl --disk-usage
```

Confirm log rotation is enabled.

### 5.2 Time Synchronization

```bash
timedatectl set-ntp true
```

Accurate time is **critical for security investigations**.

---

## 6️⃣ Fail2Ban (Optional but Recommended)

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

## 7️⃣ Backups (Non-Negotiable)

### 7.1 System Configuration

Back up at minimum:
- `/etc`
- Reverse proxy configs
- Application configuration directories

### 7.2 Application Data

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

Note: Kubernetes hardening is intentionally documented separately.

---

**Security Level Achieved:** ⭐⭐⭐☆☆☆☆☆☆☆ (3/10)  
Solid foundation. Ready for visibility tooling.

---
⬅️ Previous: [Security-First Homelab Architecture](READme.md)  
➡️ Next: [Phase 2 - Perimeter Awareness](Phase-2---Perimeter-Awareness.md)
---