# ADR-002 — Shared database + row-level security, cell-ready
**Status:** Accepted (owner, D-27) · **Date:** 2026-11-25
**Context:** Many tenants, from a single restaurant to 100-branch groups; data isolation is the existential risk; some customers may later require dedicated infrastructure or another region.
**Options:** shared DB/shared schema; schema per tenant; database per tenant; hybrid.
**Decision:** Hybrid. All tenants start in one shared database with `tenant_id` on every tenant table, forced RLS, composite tenant foreign keys and a non-bypass application role. A tenant directory maps tenants to cells so a tenant can move to a dedicated database or another region without code changes.
**Consequences:** Strict test discipline (isolation suite ISO-01…22 is a release gate); pooler compatibility must be proven (spike S-2); branch-level scoping is application-enforced; migrations must work in every cell.
**Revisit when:** the pooler spike fails, a customer demands database-per-tenant from day one, or RLS performance is unacceptable at measured load.
**Related:** Doc 3, Doc 4, Doc 11 E-I1.
