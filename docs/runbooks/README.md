# Runbooks — Index, Severity and Template

> **Status: draft procedures written before any system exists.** Commands, dashboards, alert names and tools are placeholders (`<…>`) to be replaced when the platform is built, and every runbook must be rehearsed in staging before the pilot (game days, Doc 7 §9.3). Support is **business hours at launch** (D-31); out-of-hours Sev-1 response is best-effort during the pilot and must not be promised to customers.

## Severity levels (proposal)
| Sev | Meaning | Examples | Response target (business hours / out of hours) |
|---|---|---|---|
| **Sev-1** | Customer-impacting outage or data exposure | Booking or host tools unavailable; suspected cross-tenant data access; double-bookings occurring; payment/charge errors | Start immediately / best-effort during pilot |
| **Sev-2** | Major degradation or risk | Messages delayed > 30 min; one provider down; elevated error rate; backups failing | Within 1 hour / next business day |
| **Sev-3** | Minor or contained | Single tenant issue, cosmetic, non-critical job failure | Within 1 business day |
Targets are proposals; the owner sets them (E-O1).

## Roles during an incident
Incident lead (decides, communicates); technical responder(s); communications owner (customers/status page); scribe (timeline); security lead (for security incidents); owner/management (informed on Sev-1).

## Standard flow
1. Acknowledge the alert; open an incident record (time, severity, owner).
2. **Stabilise first** (rollback, kill switch, failover) before investigating root cause.
3. Communicate: internal channel; customers via status page for Sev-1/2.
4. Preserve evidence for any security-related event (do not delete logs or restart without capturing state).
5. Resolve, verify with the checks in the runbook, and monitor for recurrence.
6. Post-incident review within 5 business days (blameless): timeline, root cause, what worked, actions with owners.

## Runbook template
Trigger/alert · Impact · How to confirm · Immediate actions · Diagnosis · Mitigation options · Escalation · Recovery and verification · Communication · After the incident.

## Index
| ID | Runbook | Typical severity |
|---|---|---|
| RB-01 | Bad deployment — rollback | 1–2 |
| RB-02 | Database failover and recovery | 1 |
| RB-03 | Restore a single tenant's data | 1–2 |
| RB-04 | Messaging provider outage or backlog | 2 |
| RB-05 | Payment provider outage or webhook problems | 2 |
| RB-06 | Suspected security incident or data breach | 1 |
| RB-07 | Suspected tenant-isolation failure | 1 |
| RB-08 | Queue, outbox or scheduler backlog | 2 |
| RB-09 | Public booking abuse (hold storm, card testing) | 2 |
| RB-10 | Using kill switches and feature flags | — |
| RB-11 | Secret or credential rotation / suspected leak | 1–2 |
