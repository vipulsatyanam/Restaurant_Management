# ADR-015 — Public booking edge as a separate process type
**Status:** Proposed · **Date:** 2026-11-25
**Context:** Unauthenticated guest traffic is abuse-prone and spiky; staff tools must stay responsive.
**Options:** same process; separate process from the same codebase; separate service and datastore.
**Decision:** Same codebase and image, deployed as a separate process type with its own scaling, rate limits and read-optimised queries (replica).
**Consequences:** Independent scaling and isolation of abuse; no separate repository or datastore.
**Revisit when:** the edge needs a different security boundary or an independent release cadence.
**Related:** Doc 2 §4, Doc 5 §3.1.
