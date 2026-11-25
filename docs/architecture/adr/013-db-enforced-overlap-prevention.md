# ADR-013 — Database-enforced overlap prevention
**Status:** Proposed · **Date:** 2026-11-25
**Context:** Double-booking is a core failure mode; application checks alone fail under concurrency.
**Options:** application locks; advisory locks; exclusion constraint on time ranges.
**Decision:** `allocation` rows hold a time range per resource; a GiST exclusion constraint prevents overlaps among active rows (including the turn buffer). Holds are rows too; expired holds are cleared in the allocating transaction. The application proposes, the database decides.
**Consequences:** The allocation table must not be partitioned by time; archive instead. The constraint covers planned occupancy only; real-world overruns are handled operationally. Requires `btree_gist` support on the managed database.
**Revisit when:** the extension is unavailable, or measured contention is unacceptable (spike S-1).
**Related:** Doc 4 §5.
