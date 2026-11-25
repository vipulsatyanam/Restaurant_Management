# ADR-003 — Managed identity provider
**Status:** Open (decision at Phase 0 spike S-3) · **Date:** 2026-11-25
**Context:** Buy rather than build authentication. Needs: organization/membership model, MFA, future per-tenant SSO, data-residency options, price at 100 and 1,000 branches.
**Options:** WorkOS; Amazon Cognito; Auth0; Microsoft Entra External ID; self-hosted Keycloak.
**Evidence (research; unverified items are marked in the research file):** Auth0 B2B plan priced $150/mo at 500 MAU and $2,100/mo at 10k MAU; WorkOS AuthKit free for the first 1M MAU with SSO at $125 per connection; Entra residency add-on limited to a few regions; Keycloak carries operational burden.
**Decision (pending):** Shortlist WorkOS and Cognito; decide by a spike scoring membership-model fit, MFA, later SSO, residency, cost and exit portability.
**Consequences:** Identity sits behind `IdentityPort`; memberships stay in Cheery, so switching providers is possible.
**Revisit when:** pricing or residency terms change, or enterprise SSO demands exceed the provider's model.
**Related:** Doc 5 §4, Doc 6 §4.1.
