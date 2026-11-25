# ADR-014 — Settings as scoped data with a resolver
**Status:** Proposed · **Date:** 2026-11-25
**Context:** Rules vary by country, organization, brand and branch; a branch may override an organization default without affecting others.
**Options:** code constants per country; config files; scoped setting rows with definitions and a resolver.
**Decision:** Setting definitions declare type, scopes, defaults, lockability and entitlement; values are stored per scope; an effective-settings resolver walks branch → brand → organization → country → platform; a "locked" value at a higher level wins; changes are versioned, audited and invalidate caches.
**Consequences:** The UI shows each value's source; cache invalidation correctness needs tests; avoid setting sprawl.
**Revisit when:** setting volume or resolver latency becomes a problem.
**Related:** Doc 3 §4, Doc 4 §4.4.
