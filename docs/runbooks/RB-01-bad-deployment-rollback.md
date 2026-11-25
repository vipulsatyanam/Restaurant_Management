# RB-01 — Bad deployment: rollback
**Trigger:** post-deploy checks fail; error rate or latency burn alert within 60 minutes of a release; booking-success rate drops; users report new breakage.
**Impact:** any surface served by the new version.
**Confirm:** compare error/latency/booking-success against the pre-deploy baseline (`<dashboard>`); check release record (version, migrations, flags changed).
**Immediate actions**
1. Pause further deploys (`<pipeline lock>`).
2. If a feature flag controls the change: **disable the flag first** (RB-10); verify recovery.
3. Otherwise redeploy the previous **image digest** (`<command/pipeline job>`); confirm health checks pass in all zones.
**Database:** do **not** roll back migrations. Expand-phase changes are compatible with the previous version; if a migration caused the issue, forward-fix. If a destructive change was released against policy, stop and escalate to the incident lead before acting.
**Verify:** error rate and latency return to baseline; synthetic booking journey passes; no stuck jobs.
**Communicate:** Sev-1/2: status page note; tell affected customers once resolved.
**After:** review why gates missed it; add a regression test; decide whether the change returns behind a flag.
