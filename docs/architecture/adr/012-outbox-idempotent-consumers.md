# ADR-012 — Transactional outbox + idempotent consumers
**Status:** Proposed · **Date:** 2026-11-25
**Context:** Reliable events between modules and to webhooks without an external broker, and no dual-write inconsistency.
**Options:** outbox in PostgreSQL; message broker; direct calls.
**Decision:** Events are written to an outbox table in the same transaction as the state change; a relay publishes to in-process subscribers and the job queue; consumers record processed event ids (consumer inbox) and are idempotent.
**Consequences:** At-least-once delivery; consumers must tolerate duplicates and reordering; the outbox table needs partitioning and purge.
**Revisit when:** event volume or latency needs exceed what PostgreSQL-backed queues deliver.
**Related:** Doc 2 §6.3, Doc 4 §4.12, Doc 5 §5.
