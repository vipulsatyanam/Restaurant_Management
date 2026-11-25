# ADR-004 — Third-party billing service
**Status:** Open (Phase 0 spike S-3) · **Date:** 2026-11-25
**Context:** Owner decision D-08: use a third-party billing service. Needs per-branch quantity, trials, monthly/annual terms, AUD/INR/GBP, tax invoices, dunning, usage-based message overage and entitlement events. India recurring payments and GST invoicing add constraints; Stripe is invite-only for Indian businesses per research.
**Options:** Stripe Billing; Chargebee; Paddle (merchant of record); a second regional provider for India.
**Decision (pending):** Evaluate against the real plan catalogue and the three paper-tested countries. Behind `BillingPort`; entitlements derive from billing events, never from direct provider calls in modules.
**Consequences:** Two providers may be needed (AU vs IN). Message metering and overage must be supported, or computed by Cheery and pushed as usage.
**Revisit when:** a target country is unsupported, or fees erode margin (the D-26 model assumes 3%, unverified).
**Related:** Doc 5 §4, Doc 9 §6, Doc 11 E-N5.
