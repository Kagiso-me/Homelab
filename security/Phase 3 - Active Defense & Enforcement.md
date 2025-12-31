# 🥇 Phase 3 — Active Defense & Enforcement
#### **Security Level:** 7 / 10

✅ **Phase 1 hardened the host.**

✅ **Phase 2 gave us visibility.**

**Phase 3 is where the environment starts fighting back.**

This phase introduces automated enforcement — turning detected malicious behavior into **real, measurable consequences**.

---

## 🧠 Context: Where We Are Now

By the end of Phase 2:

- The edge router logs meaningful events
- Logs are centralized on the security node (Sentinel)
- CrowdSec observes behavior patterns
- Local brute-force attempts are detected

Detection alone is not enough.

> **Detection without enforcement is still reactive.**

Phase 3 closes that gap.

---

## 🎯 Phase Goal

Transform passive detection into **active defense**.

By the end of this phase:

- Malicious IPs are automatically blocked
- Blocking happens as close to the edge as possible
- Enforcement is consistent and reversible
- No human intervention is required
- False positives are controlled and observable

---

## 🔐 Scope

**Applies to**
- Sentinel (security node)
- MikroTik edge router
- Internet-facing services

**Out of scope**
- Kubernetes policy
- Application-layer auth
- Zero Trust identity (future phase)

---

## 🧱 Enforcement Philosophy

> **Block early. Block upstream. Block surgically.**

We do not:
- Geo-block blindly
- Permanently ban without review
- Push enforcement deep into workloads

We do:
- Enforce at the edge
- Let behavior decide
- Keep all actions observable and reversible

---

## 🗺️ Enforcement Flow

Internet → MikroTik (Router)→ Sentinel (CrowdSec) → Decision → Bouncer → Edge Block

Detection is centralized.  
Enforcement is upstream.

---

## 1️⃣ CrowdSec Enforcement Model

CrowdSec does not block traffic itself.

It produces **decisions**:
- IP
- Reason
- Duration

Decisions are consumed by **bouncers**.

---

## 2️⃣ Enforcement Strategy

Given this environment:

- MikroTik is the choke point
- Sentinel is the decision engine

**Correct enforcement point:** MikroTik

One block protects everything.

---

## 3️⃣ Prepare MikroTik for Automated Blocking

### 3.1 Create Dedicated Address List

```mikrotik
/ip firewall address-list
add list=crowdsec_blacklist address=0.0.0.0 disabled=yes comment="placeholder"
```

### 3.2 Enforce the Blocklist

```mikrotik
/ip firewall filter
add chain=input src-address-list=crowdsec_blacklist action=drop \
    comment="Drop traffic from CrowdSec decisions"
```

Rule placement:
- Below established/related
- Above allow rules

---

## 4️⃣ CrowdSec Bouncer Setup

### 4.1 Install Bouncer Package

```bash
sudo apt install -y crowdsec-firewall-bouncer-iptables
```

### 4.2 Create API Key

```bash
sudo cscli bouncers add mikrotik-bouncer
```

Save the generated key.

---

## 5️⃣ Connect CrowdSec to MikroTik

### 5.1 Configure Bouncer

```bash
sudo nano /etc/crowdsec/bouncers/mikrotik.yaml
```

```yaml
api_url: http://127.0.0.1:8080/
api_key: <API_KEY>

mikrotik:
  host: 10.0.10.1
  username: crowdsec
  password: <PASSWORD>
  address_list: crowdsec_blacklist
  timeout: 30s
```

### 5.2 Create MikroTik User

```mikrotik
/user add name=crowdsec group=read,write password=<PASSWORD>
```

---

## 6️⃣ Validation

### 6.1 Trigger Test Attack

From an external IP:

```bash
for i in {1..10}; do ssh invalid@<PUBLIC_IP>; done
```

### 6.2 Check CrowdSec

```bash
sudo cscli decisions list
```

### 6.3 Check MikroTik

```mikrotik
/ip firewall address-list print where list=crowdsec_blacklist
```

Traffic should now be blocked upstream.

---

## 7️⃣ Safety & Rollback

### Manual Unban

```bash
sudo cscli decisions delete --ip <IP>
```

### Emergency Disable

```mikrotik
/ip firewall filter disable [find comment~"CrowdSec"]
```

---

## 8️⃣ Phase 3 Validation Checklist

- Decisions created only after malicious behavior
- IPs added automatically to MikroTik
- Bans expire correctly
- No LAN IPs blocked
- Router CPU stable
- Enforcement reversible

---

## 🧭 Phase Output

After Phase 3 you have:

- Automated threat containment
- Edge-level enforcement
- Behavior-driven blocking
- A defensible perimeter

**Security Level Achieved:** ⭐⭐⭐⭐⭐⭐⭐☆☆☆ (7 / 10)
