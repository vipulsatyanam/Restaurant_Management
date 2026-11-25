# RB-02 — Database failover and recovery
**Trigger:** primary unreachable or failing; high replication lag; storage or connection exhaustion; data-corruption suspicion.
**Impact:** Sev-1 — bookings and host tools unavailable or stale.
**Confirm:** cloud database status/events (`<console>`); application connection errors; replica lag.
**Immediate actions**
1. Declare Sev-1; freeze deployments.
2. **Availability-zone or instance failure:** managed multi-AZ failover should occur automatically; confirm the new primary, that applications reconnect, and that the pooler has no stale connections.
3. **Connection exhaustion:** reduce task counts/pool sizes, restart the pooler, identify the runaway process or tenant (per-tenant limits).
4. **Storage full:** extend storage, then find the cause (outbox/audit/message growth); purge/archive per policy.
5. **Corruption or bad change:** stop writes if needed (kill switch for write endpoints `<flag>`); choose **point-in-time recovery** to a new instance just before the incident; **verify** invariants (no overlapping allocations; ledger lines sum to zero; row counts) before switching traffic.
**Recovery:** switch the application to the recovered instance via configuration; reconcile events after the restore point (payments, messages, webhooks) using reconciliation jobs; tell affected customers about any lost recent changes.
**Targets (proposals):** RPO ≤ 5 min, RTO ≤ 4 h (Doc 9 §3.2).
**After:** record actual RPO/RTO; fix root cause; update capacity thresholds.
