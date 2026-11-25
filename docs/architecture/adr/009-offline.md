# ADR-009 — Offline operation
**Status:** Deferred (D-10) · **Date:** 2026-11-25
**Context:** The prototype claimed offline tablets; today the need is speculative and no customer requests it. Two disconnected devices cannot guarantee global availability.
**Options:** none in Release 1; later a PWA with a local store and a sync engine (PowerSync, RxDB, ElectricSQL) or native.
**Decision:** Out of Release 1. Keep cheap readiness: client-generated ids, idempotency keys on writes, change records as events. Later scope is limited to host-safe operations (view, seat walk-ins, mark arrivals/state); shared availability and money remain online-only.
**Consequences:** No offline client now; no promise to customers.
**Revisit when:** a customer asks for offline with a commercial commitment; run a sync-engine spike first.
**Related:** Doc 2 §5.5, Doc 9 §7.
