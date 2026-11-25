# ADR-011 — Modular monolith with enforced boundaries
**Status:** Proposed (style approved in Stage C, D-27) · **Date:** 2026-11-25
**Context:** The domain is still being learned; small team; need simple transactions and low cost, with room to evolve.
**Options:** microservices; SOA; modular monolith; event-driven hybrid.
**Decision:** One codebase with module-owned schemas, `public/` interfaces, events for cross-module reactions and several runtime process types. Boundaries are enforced by automated checks (Doc 2 §5.4).
**Consequences:** One release train and one database to protect; boundary erosion is the main risk; extraction to services only on measured need.
**Revisit when:** a module has independent scaling, security-isolation or team-ownership needs that a boundary cannot satisfy.
**Related:** Doc 2, Doc 8.
