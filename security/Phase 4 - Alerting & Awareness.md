# Phase 4 — Alerting & Awareness
#### **Security Level:** 8 / 10  

## Purpose

Phase 4 turns **visibility into action**.

After Phase 3, you can *see* what is happening (logs, metrics, dashboards).  
Phase 4 ensures you are **notified when something requires attention**, without drowning in noise.

> **Goal**
>
> Receive **timely, meaningful alerts** about security, stability, and availability issues — and ignore everything else.

This phase is about **awareness**, not enforcement.

---

## Design Principles (Read First)

1. **Signal > Noise**
   - Few alerts
   - High confidence
   - Actionable

2. **Alerts represent symptoms, not metrics**
   - “Disk will be full in 6 hours”
   - Not “Disk is at 71%”

3. **Dashboards are for humans**
   - Alerts are for interruptions
   - If it’s not interrupt-worthy, it’s not an alert

4. **Alerting builds on observability**
   - No Phase 4 without Phase 3
   - Grafana, Loki, Prometheus already exist

---

## Architecture Overview

```
Logs (Apps / Ingress / System)
        ↓
      Loki
        ↓
   Grafana Alerts  ───────┐
                          ├── Notification Channel
Metrics (Nodes / Pods)    │
        ↓                 │
    Prometheus ───────────┘
```

### Components

- **Grafana** — alert rules, dashboards, notifications
- **Loki** — log-based alert sources
- **Prometheus** — metric-based alert sources
- **Alertmanager** — routing & deduplication (if enabled)

---

## Alert Categories (Authoritative)

Only these categories are allowed.

### 1. Security Signals
- Repeated authentication failures
- Suspicious ingress patterns
- Excessive 401 / 403 responses
- Vault auth failures
- Certificate errors

### 2. Stability Signals
- Pod crash loops
- Containers restarting frequently
- Node not ready
- OOM kills

### 3. Availability Signals
- Service unreachable
- Ingress returning 5xx
- Certificate expiration
- DNS resolution failures

### 4. Resource Exhaustion
- Disk pressure
- Memory pressure
- CPU saturation (sustained, not spikes)

---

## Alerting Strategy

### What We Alert On
- Things that **require intervention**
- Things that **will become outages**
- Things that **indicate compromise or misconfiguration**

### What We Do NOT Alert On
- Short-lived spikes
- One-off pod restarts
- Non-user-facing errors
- Development noise

---

## Implementation — Step by Step

---

## 1. Notification Channels (Required)

Configure at least one channel in Grafana:

- Email
- Telegram
- Slack
- Matrix
- Webhook

> **Rule:** Alerts must reach *you*, not just exist.

Example (conceptual):
- Channel: `primary-ops`
- Severity-aware routing (later)

---

## 2. Metric-Based Alerts (Prometheus → Grafana)

### 2.1 Node Disk Pressure

**Why**
Disk exhaustion causes cascading failures.

**Alert condition**
- Disk usage > 85%
- Sustained for 10 minutes

**Signal**
- Predictive, not reactive

---

### 2.2 Pod CrashLoopBackOff

**Why**
Indicates broken deploys, config errors, or secret issues.

**Alert condition**
- Restart count increasing
- Over a rolling window

---

### 2.3 Node NotReady

**Why**
Loss of capacity or hardware failure.

**Alert condition**
- Node NotReady > 5 minutes

---

## 3. Log-Based Alerts (Loki → Grafana)

Log alerts catch **things metrics cannot**.

---

### 3.1 Repeated Authentication Failures

**Source**
- Ingress logs
- App auth logs

**Alert condition**
- > N failed auth attempts
- From same IP or account
- Within short window

**Why**
Early indicator of brute-force or misconfig.

---

### 3.2 Vault Authentication Errors

**Source**
- Vault audit logs
- ESO logs

**Alert condition**
- Repeated auth failures
- Token permission denied
- Backend unreachable

**Why**
Secrets delivery failure impacts multiple apps.

---

### 3.3 Ingress 5xx Spike

**Source**
- Traefik logs

**Alert condition**
- Sustained 5xx responses
- Over baseline

**Why**
User-visible outage signal.

---

## 4. Certificate & TLS Alerts

### 4.1 Certificate Expiry

**Why**
Expired certs cause total outages.

**Alert condition**
- Cert expires within 14 days
- Escalate at 7 days

---

### 4.2 TLS Handshake Failures

**Why**
Can indicate:
- Client misconfig
- Cert trust issues
- MITM attempts (rare but important)

---

## 5. Alert Severity Model

Use **three levels only**:

| Level | Meaning | Action |
|----|----|----|
Info | Awareness | Review when convenient |
Warning | Degradation | Investigate soon |
Critical | Impact / Risk | Act immediately |

> If everything is critical, nothing is.

---

## 6. Alert Hygiene Rules (Mandatory)

- Every alert must answer:
  - What happened?
  - Why it matters?
  - What to check first?
- Alerts without owners are removed
- Alerts that fire repeatedly without action are deleted

---

## 7. Validation & Testing

Before considering Phase 4 complete:

### Test cases
- Kill a pod → alert fires
- Fill disk artificially → alert fires
- Break ingress → alert fires
- Restore system → alert resolves

---

## 8. What Phase 4 Explicitly Does NOT Do

- No traffic blocking
- No automatic remediation
- No runtime enforcement
- No NetworkPolicies
- No admission control

Those belong to **Phase 6**.

---

## 9. Completion Criteria (Checklist)

Phase 4 is complete when:

- [ ] Notification channel works
- [ ] At least one alert per category exists
- [ ] Alerts are tested
- [ ] Noise is reviewed and reduced
- [ ] Dashboards remain clean

---

## 10. Why Phase 4 Matters

Most incidents are not caused by:
- Zero-days
- Nation-state attacks

They are caused by:
- Silent failures
- Missed warnings
- Expired certs
- Full disks

Phase 4 ensures **you are never surprised**.

---

## Phase 4 Status

✅ **Design complete**  
🚧 **Implementation driven by your environment**

Once Phase 4 is stable, you are ready to:
- Recover confidently (Phase 5)
- Handle secrets safely (Phase 6)
- Centralize identity (Phase 7)

This phase is foundational — do not rush it.
