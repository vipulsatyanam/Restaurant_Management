# ADR-008 — Edge, WAF and DDoS protection
**Status:** Open · **Date:** 2026-11-25
**Context:** Public booking surfaces need bot protection, rate limiting and a WAF; the origin should accept traffic only from the edge.
**Options:** Cloudflare (research: WAF on all plans; Pro $20/mo, Business $200/mo); AWS WAF with CloudFront.
**Decision (pending):** Choose with the hosting design in Phase 0, considering custom tenant domains, bot management, cost at traffic levels and operational simplicity.
**Consequences:** Whichever is chosen becomes a sub-processor; tenant custom-domain certificates are handled at the edge.
**Revisit when:** attack patterns exceed the plan's protection, or costs diverge.
**Related:** Doc 6 §4.5, Doc 7 §2.5.
