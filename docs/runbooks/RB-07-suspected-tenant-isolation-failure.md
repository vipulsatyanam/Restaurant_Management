# RB-07 — Suspected tenant-isolation failure
**Trigger:** customer reports seeing another customer's data; isolation test failure in production monitors; RLS "no tenant context" error spike; authorization anomalies; a pooler/connection-context anomaly.
**Impact:** Sev-1 — the highest-risk event for the platform. Treat as a data breach until disproved (then follow RB-06).
**Immediate actions**
1. Declare Sev-1 and assign the security lead; preserve evidence (request ids, trace ids, queries, timestamps, user and tenant ids).
2. **Stop the exposure:** disable the affected endpoint/feature with a kill switch (RB-10); if unclear, put the affected surface in read-only/maintenance mode; consider taking the public edge offline briefly.
3. Reproduce safely in staging with the same code version using the isolation suite and the failing request.
4. Check the likely causes in order: missing tenant context on a path (job, event consumer, report, export); pooler reusing a session setting; role with bypass privilege used by mistake; cache key without tenant; file path without tenant; query that joins across tenants.
**Scope:** determine exactly which tenants and records were exposed, using access logs and audit events; list affected individuals and data classes.
**Recovery:** deploy a fix with a regression test added to the isolation suite; re-run the full suite (ISO-01…22) and schema-catalogue test; re-enable gradually.
**Notifications:** follow RB-06 with legal advice; tell all affected tenants.
**After:** full post-incident review; consider additional controls (branch/row policies, stricter linting).
