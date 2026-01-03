# 🥇 Phase 3 — Active Defense & Enforcement

#### **Security Level:** 8 / 10

> **Detection is passive. Enforcement changes outcomes.**
> Phase 3 turns CrowdSec decisions into **real, automated blocks** at **multiple layers**.

---

## 🧠 Context

By the end of Phase 2:

* Logs are centralized on the security node (**Sentinel**)
* CrowdSec analyzes router, system, and ingress logs
* Malicious behavior is detected reliably

Phase 3 introduces **enforcement** — automatic, observable, and reversible.

---

## 🎯 Phase Goal

Transform detection into **active defense**.

By the end of this phase:

* Malicious IPs are automatically blocked
* Blocking happens at the **earliest effective layer**
* Enforcement is consistent across the stack
* Actions are reversible and auditable
* No manual intervention is required

---

## 🔐 Scope

**In scope**

* Sentinel (CrowdSec decision engine)
* MikroTik edge router
* Traefik ingress controller
* Internet-facing services

**Out of scope**

* Kubernetes NetworkPolicy
* Application-level authentication
* Zero Trust identity (Phase 4)

---

## 🧱 Enforcement Philosophy

> **Block early. Block upstream. Block surgically.**

We do **not**:

* Geo-block blindly
* Permanently ban without review
* Push enforcement deep into workloads

We **do**:

* Enforce as close to the edge as possible
* Use behavior-based decisions
* Maintain observability and reversibility

---

## 🗺️ Enforcement Flow (Updated)

```
Internet
  ↓
MikroTik Router (L3/L4 enforcement)
  ↓
Traefik Ingress (L7 / HTTP enforcement)
  ↓
Services
```

* **Sentinel (CrowdSec)** is the single decision engine
* **Bouncers** enforce decisions at different layers

---

## 1️⃣ CrowdSec Enforcement Model

CrowdSec does **not** block traffic itself.

It produces **decisions**:

* IP
* Scenario
* Duration

Decisions are enforced by **bouncers**.

---

## 2️⃣ Enforcement Strategy

Given this environment:

* MikroTik is the **network choke point**
* Traefik is the **application choke point**
* Sentinel is the **authority**

We therefore enforce at **two layers**:

| Layer | Enforcement      | Purpose                   |
| ----- | ---------------- | ------------------------- |
| L3/L4 | MikroTik bouncer | Global network protection |
| L7    | Traefik bouncer  | HTTP-aware protection     |

---

# 🔒 PART A — MikroTik Enforcement (Edge)

## 3️⃣ Prepare MikroTik

### 3.1 Create Address List

```mikrotik
/ip firewall address-list
add list=crowdsec_blacklist address=0.0.0.0 disabled=yes comment="placeholder"
```

### 3.2 Enforce Blocklist

```mikrotik
/ip firewall filter
add chain=input src-address-list=crowdsec_blacklist action=drop \
    comment="Drop traffic from CrowdSec decisions"
```

Rule placement:

* Below established/related
* Above allow rules

---

## 4️⃣ MikroTik Bouncer Setup (Sentinel)

### 4.1 Install Bouncer

```bash
sudo apt install -y crowdsec-firewall-bouncer-iptables
```

### 4.2 Create API Key

```bash
sudo cscli bouncers add mikrotik-bouncer
```

Save the API key.

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

## 6️⃣ Validate MikroTik Enforcement

### Trigger Test Attack

```bash
for i in {1..10}; do ssh invalid@<PUBLIC_IP>; done
```

### Verify CrowdSec

```bash
sudo cscli decisions list
```

### Verify MikroTik

```mikrotik
/ip firewall address-list print where list=crowdsec_blacklist
```

---

# 🚦 PART B — Traefik Enforcement (Ingress)

## 7️⃣ Why Traefik Enforcement Is Needed

MikroTik blocks **network abuse**, but cannot see:

* HTTP probing (404 scans)
* CMS / API enumeration
* Auth endpoint abuse
* Large-upload DoS attempts

These appear **only** in Traefik access logs.

---

## 8️⃣ Traefik Bouncer Overview

The Traefik bouncer:

* Runs inside Traefik as a plugin
* Queries CrowdSec LAPI per request
* Blocks malicious IPs before routing

It **enforces**, it does not detect.

---

## 9️⃣ Create Traefik API Key (Sentinel)

```bash
sudo cscli bouncers add traefik-bouncer
```

Save the API key.

---

## 🔟 Enable Traefik CrowdSec Plugin

In Traefik Helm values:

```yaml
experimental:
  plugins:
    crowdsec:
      moduleName: github.com/maxlerebourg/crowdsec-bouncer-traefik-plugin
      version: v1.3.0
```

Upgrade Traefik:

```bash
helm upgrade traefik traefik/traefik -n traefik -f values.yaml
```

---

## 1️⃣1️⃣ Create Traefik CrowdSec Middleware

```yaml
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: crowdsec-bouncer
  namespace: traefik
spec:
  plugin:
    crowdsec:
      enabled: true
      crowdsecLapiHost: http://crowdsec.sentinel.svc.cluster.local:8080
      crowdsecLapiKey: <API_KEY>
      crowdsecMode: live
```

---

## 1️⃣2️⃣ Attach Middleware to IngressRoutes

Example:

```yaml
middlewares:
  - name: crowdsec-bouncer
    namespace: traefik
```

Apply to all internet-facing services.

---

## 1️⃣3️⃣ Enable Traefik Log Collection

Ensure Traefik access logs are enabled:

```yaml
accessLog:
  enabled: true
  format: json
```

Install Traefik collection on Sentinel:

```bash
sudo cscli collections install crowdsecurity/traefik
sudo systemctl restart crowdsec
```

---

## 1️⃣4️⃣ Validate Traefik Enforcement

### Trigger HTTP Probing

```bash
curl https://cloud.example.com/wp-login.php
curl https://cloud.example.com/admin
```

### Verify CrowdSec Decision

```bash
sudo cscli decisions list
```

### Verify Traefik Blocking

```bash
kubectl logs -n traefik deploy/traefik | grep crowdsec
```

---

## 1️⃣5️⃣ Safety & Rollback

### Remove Traefik Enforcement

```bash
kubectl delete middleware crowdsec-bouncer -n traefik
```

### Remove MikroTik Enforcement

```mikrotik
/ip firewall filter disable [find comment~"CrowdSec"]
```

Detection continues in both cases.

---

## 🧭 Phase Output

After Phase 3 you have:

* Network-level automated blocking
* Application-layer automated blocking
* Unified decision engine
* Full observability
* Reversible enforcement

**Security Level Achieved:** ⭐⭐⭐⭐⭐⭐⭐⭐☆☆ (8 / 10)

---

⬅️ Previous: Phase 2 — Perimeter Awareness
➡️ Next: Phase 4 — Identity & Zero Trust
