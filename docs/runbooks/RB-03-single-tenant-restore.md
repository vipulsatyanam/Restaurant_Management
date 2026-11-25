# RB-03 — Restore a single tenant's data
**Trigger:** a tenant's data was deleted or corrupted (user error, bad import, bug) and the rest of the platform is healthy.
**Principle:** never overwrite other tenants; restore into a scratch instance, extract, verify, then apply through a reviewed procedure.
**Steps**
1. Confirm scope with the tenant and the incident lead: which tenant, which tables/time window; get written approval from the tenant owner.
2. Record the target restore time (before the damage) from audit/outbox history.
3. Restore a snapshot/PITR to a **scratch instance** in the same account (`<procedure>`).
4. Extract the tenant's rows by `tenant_id` for the affected tables, preserving ids.
5. Compare with current state; produce a diff report (what will be inserted/updated); review by a second engineer.
6. Apply under the **platform restore role** in a transaction, with allocation overlap and ledger checks; log an audit event.
7. Reconcile side effects: messages not re-sent; payment references verified against the provider.
8. Verify with the tenant (sample bookings, guests, settings).
9. Destroy the scratch instance and temporary extracts after approval.
**Do not:** copy data across tenants, bypass RLS in the application path, restore the whole database to fix one tenant.
**After:** document the root cause; consider a soft-delete or confirmation step to prevent recurrence. Quarterly rehearsal (Doc 7 §9.3).
