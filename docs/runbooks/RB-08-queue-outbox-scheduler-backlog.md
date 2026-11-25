# RB-08 — Queue, outbox or scheduler backlog
**Trigger:** job age, outbox lag, reminder or hold-expiry lag alerts; stuck jobs; growing dead-letter queue.
**Impact:** Sev-2 — reminders/confirmations delayed, holds not expiring on time (but expired holds are still cleared at allocation time), webhooks and reports delayed.
**Confirm:** queue depth and age per job type; worker health and logs; database locks or slow queries; a single tenant monopolising capacity.
**Immediate actions**
1. Scale out workers if CPU/memory bound; check database connection budget before scaling.
2. If one tenant floods the queue (bulk import, reminders): apply the per-tenant concurrency cap; pause that tenant's jobs if needed; keep others flowing.
3. Failing jobs: inspect the error class; fix or skip poison messages to the dead-letter queue after capturing them; **never delete without recording**.
4. Outbox lag: check the relay process, long transactions blocking visibility, and consumers failing repeatedly.
**Recovery:** let the backlog drain at a controlled rate; drop or mark stale time-sensitive jobs (e.g. reminders past their time); run reconciliation jobs.
**After:** adjust capacity and fairness limits; add alerts for the root cause; test with the load rig (Doc 9 §8).
