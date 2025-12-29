# 🥈 Phase 2 — Perimeter Awareness
Security Level: 5 / 10

Phase 1 hardened the operating system and reduced the attack surface.

Phase 2 assumes:
- The host OS is hardened
- Firewall defaults are deny-first
- SSH is restricted and key-based
- Only required services are exposed

This phase adds **visibility and behavioral detection**.

---

## 🎯 Phase Goal

Detect hostile behavior early and prepare the environment for automated blocking.

At the end of this phase:
- The router produces meaningful security logs
- Logs are centralized
- Behavioral attacks are detected
- Local brute-force attempts are contained

No global enforcement yet.

---

## 🔐 Scope

Applies to:
- Security node (from Phase 1)
- MikroTik edge router
- All Linux servers

Out of scope:
- Kubernetes
- Ingress
- Application security

---

## 1️⃣ MikroTik Firewall Logging

The router is your first sensor.

### Logging Principles
- Log intent, not noise
- Log only NEW inbound activity
- Use consistent prefixes

---

### 1.1 Create Firewall Log Rules (MikroTik)

> Adjust interface names and address lists to your environment.

**Log dropped inbound packets**
```mikrotik
/ip firewall filter
add chain=input action=log log-prefix="FW_INPUT_DROP " \
    connection-state=new in-interface-list=WAN
```

**Log SSH attempts**
```mikrotik
/ip firewall filter
add chain=input action=log log-prefix="FW_SSH_ATTEMPT " \
    protocol=tcp dst-port=22 in-interface-list=WAN
```

**Log port scanning**
```mikrotik
/ip firewall filter
add chain=input action=log log-prefix="FW_PORTSCAN " \
    protocol=tcp psd=21,3s,3,1 in-interface-list=WAN
```    
⚠️ Place log rules before drop rules.

### 1.2 Verify Logging

**From MikroTik:**
```mikrotik
/log print where message~"FW_"
```
You should see events with the prefixes above.

---

## 2️⃣ Forward Logs to the Security Node
All firewall logs must be forwarded via syslog.

### 2.1 Enable Syslog on MikroTik

```mikrotik
/system logging action
add name=remote target=remote remote=<SECURITY_NODE_IP> remote-port=514
```

```mikrotik
/system logging
add topics=firewall action=remote
```
### 2.2 Verify Log Reception (Security Node)
On the security node:
```bash
sudo journalctl -f | grep FW_
```
If nothing appears:
- Check firewall rules
- Check UDP 514 allowed
- Verify IP address

⚠️ Do not continue until logs arrive.

---

## 3️⃣ Install CrowdSec (Security Node)
CrowdSec will parse logs and detect behavior.

### 3.1 Install CrowdSec
```bash
curl -s https://install.crowdsec.net | sudo sh
sudo apt update
sudo apt install -y crowdsec
```


### 3.2 Install Required Collections
```bash
sudo cscli collections install crowdsecurity/syslog
sudo cscli collections install crowdsecurity/sshd
sudo cscli collections install crowdsecurity/firewallservices
```
Restart CrowdSec:
```bash
sudo systemctl restart crowdsec
```

### 3.3 Verify CrowdSec Status
```bash
sudo cscli status
```
Confirm:
- CrowdSec is running
- Collections are enabled
- No parser errors

### 3.4 Validate Parsing
Trigger a test SSH attempt from an external IP.

Then run:

```bash
sudo cscli decisions list
sudo cscli metrics
```
You should see:
- Parsed events
- Scenario hits
- Decisions created
If not — stop and fix parsing.

---

## 4️⃣ Install Fail2ban (All Nodes)
Fail2ban provides immediate local protection. Repeat this section on every Linux node.

### 4.1 Install Fail2ban
```bash
sudo apt install -y fail2ban
```
### 4.2 Create Local Jail Configuration
```bash
sudo nano /etc/fail2ban/jail.local
```
Paste:
```ini
[sshd]
enabled = true
port    = ssh
logpath = %(sshd_log)s
maxretry = 3
bantime  = 15m
findtime = 10m
```
### 4.3 Start and Enable Fail2ban
```bash
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban
```

### 4.4 Verify Fail2ban
```bash
sudo fail2ban-client status
sudo fail2ban-client status sshd
```
Test with failed SSH attempts and confirm IP bans.

---

## 5️⃣ End-to-End Validation
At this point:
- MikroTik logs security events
- Logs reach the security node
- CrowdSec parses and detects behavior
- Fail2ban bans locally

✅ Phase Validation Checklist

Do NOT proceed unless all are true:
 - ✔ Firewall logs visible on security node
 - ✔ Syslog timestamps are correct
 - ✔ CrowdSec parses logs
 - ✔ Scenarios trigger on test attacks
 - ✔ Decisions appear in cscli
 - ✔ Fail2ban bans locally
 - ✔ No LAN false positives
 - ✔ Router CPU stable

---

🧭 Phase Output

After Phase 2 you have:
 - Network-level visibility
 - Behavioral detection
 - Local containment
 - A clean foundation for enforcement


**Security Level Achieved:** ⭐⭐⭐⭐⭐☆☆☆☆☆ (5/10) 
