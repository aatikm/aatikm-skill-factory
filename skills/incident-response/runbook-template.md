# Incident Runbook Template

**Service / component:** <service name>  
**Runbook version:** 1.0  
**Last updated:** YYYY-MM-DD  
**Owner:** <team name>

---

## Overview

> Brief description of what this service does and why it is critical.

---

## Severity levels

| Severity | Definition | Response time | Example |
|----------|-----------|---------------|---------|
| SEV-1 | Service down, data loss, or major revenue impact | Immediate (< 15 min) | All users cannot log in |
| SEV-2 | Degraded service; core functionality impaired | < 30 min | Checkout page errors for 20% of users |
| SEV-3 | Minor issue; workaround exists | < 4 hours | Slow report generation |
| SEV-4 | Cosmetic or low-impact | Next business day | Typo in error message |

---

## Contacts

| Role | Name / Channel | How to page |
|------|---------------|-------------|
| Primary on-call | | PagerDuty / Slack: <handle> |
| Secondary on-call | | PagerDuty / Slack: <handle> |
| Escalation (tech lead) | | Slack: <handle> |
| Escalation (engineering manager) | | Slack: <handle> |
| External vendor support | | <link / phone> |

---

## Monitoring & alerting

| Dashboard | URL |
|-----------|-----|
| Service health | <link> |
| Error rates | <link> |
| Latency (p50 / p95 / p99) | <link> |
| Infrastructure (CPU, memory, disk) | <link> |
| Logs | <link> |

---

## Incident response steps

### Step 1 — Acknowledge and assess (0–5 min)

1. Acknowledge the alert in PagerDuty / Slack.
2. Check the service health dashboard.
3. Determine the severity (SEV-1 through SEV-4).
4. If SEV-1 or SEV-2, open a War Room channel: `#incident-YYYY-MM-DD-<short-title>`.
5. Post an initial status update to the stakeholder channel.

### Step 2 — Mitigate (5–30 min)

> Try to restore service first; investigate root cause second.

Common quick mitigations for this service:

- **Restart pods / services:**
  ```
  <command>
  ```
- **Roll back deployment:**
  ```
  <command>
  ```
- **Enable maintenance mode / circuit breaker:**
  ```
  <command>
  ```
- **Scale up:**
  ```
  <command>
  ```

### Step 3 — Investigate

Check the following in order:

1. Recent deployments (last 24 hours): `<link to deployment history>`
2. Error logs: `<log query>`
3. Database performance: `<link to DB dashboard>`
4. Upstream / downstream service health: `<links>`
5. Recent config or infrastructure changes

### Step 4 — Communicate

Send status updates every **15 minutes** during a SEV-1/2 incident.

Template:
```
*Incident update — HH:MM UTC*
Status: Investigating / Mitigating / Monitoring / Resolved
Impact: <what users are experiencing>
Next update: HH:MM UTC
```

### Step 5 — Resolve and monitor

1. Confirm the service has recovered (error rate back to baseline, latency normal).
2. Post an "All clear" message.
3. Keep monitoring for **30 minutes** before closing the incident.
4. Schedule a post-mortem within **48 hours** for SEV-1/2 incidents.

---

## Known issues and playbooks

| Symptom | Likely cause | Playbook |
|---------|-------------|---------|
| High memory usage | Memory leak in worker | Restart workers, then investigate |
| Slow DB queries | Missing index / lock contention | Check slow query log |
| 502 errors from load balancer | All instances unhealthy | Check instance health, restart, rollback |
| Queue depth growing | Consumer lag or consumer crash | Restart consumers, check dead-letter queue |

---

## Links

- Architecture diagram: <link>
- Related ADRs: <link>
- Post-mortem history: <link>
- Dependency map: <link>
