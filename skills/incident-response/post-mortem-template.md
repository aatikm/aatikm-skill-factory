# Blameless Post-Mortem Template

**Incident title:** <short description>  
**Incident date:** YYYY-MM-DD  
**Severity:** SEV-<1/2/3>  
**Post-mortem date:** YYYY-MM-DD  
**Facilitator:** <name>  
**Attendees:** <list>  
**Status:** Draft / In review / Final

---

## Summary

> A 2–3 sentence plain-English summary of what happened, the impact, and how it was resolved.
> Write it so that someone not in the incident can understand it immediately.

---

## Impact

| Metric | Value |
|--------|-------|
| Duration | <X hours Y minutes> |
| Users affected | <number or % of user base> |
| Services affected | <list> |
| Revenue / SLA impact | <$ or % or N/A> |
| Data loss | Yes / No |

---

## Timeline

All times in UTC.

| Time | Event |
|------|-------|
| HH:MM | Alert fired / incident detected |
| HH:MM | On-call acknowledged |
| HH:MM | War room opened |
| HH:MM | <key investigation step> |
| HH:MM | Mitigation applied |
| HH:MM | Service restored |
| HH:MM | All clear posted |
| HH:MM | Incident closed |

---

## Root cause analysis

### What happened?

> Describe the technical chain of events that led to the incident.

### Five Whys

1. **Why** did <symptom> occur? → <answer>
2. **Why** did <answer 1> occur? → <answer>
3. **Why** did <answer 2> occur? → <answer>
4. **Why** did <answer 3> occur? → <answer>
5. **Why** did <answer 4> occur? → **Root cause: <root cause>**

### Contributing factors

- <factor 1 — e.g. "No automated test covered this edge case">
- <factor 2 — e.g. "Alert threshold was too high to catch early degradation">
- <factor 3>

---

## What went well

> Highlight the things the team did right — fast detection, good communication, effective rollback, etc.

- <positive 1>
- <positive 2>

---

## What could be improved

> Identify gaps in detection, response, communication, or tooling — without blame.

- <area 1>
- <area 2>

---

## Action items

| # | Action | Owner | Due | Status |
|---|--------|-------|-----|--------|
| 1 | | | | Open |
| 2 | | | | Open |
| 3 | | | | Open |

> Each action item should make a recurrence **less likely** or **faster to detect and mitigate**.

---

## Supporting links

- Incident channel: `#incident-<date>-<title>`
- Alert / PagerDuty: <link>
- Logs during incident: <link>
- Dashboard screenshots: <link>
- Related ADR / runbook: <link>
