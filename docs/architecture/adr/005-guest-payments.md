# ADR-005 — Guest payments via provider-hosted pages, per-country adapters
**Status:** Open (Phase 0) · **Date:** 2026-11-25
**Context:** Deposits and no-show fees are charged on each restaurant's own merchant account (D-11: no percentage taken by Cheery). Cheery must not store or process card data, to keep PCI scope minimal. India constraints (Stripe invite-only; platform onboarding in preview; Razorpay route turnover threshold) differ from Australia.
**Options:** Stripe (hosted checkout + saved payment method) via the platform-account model for AU; Adyen for Platforms; local providers per country.
**Decision (pending):** Provider-hosted payment pages and saved-method charging behind `GuestPaymentPort`, with a merchant-onboarding flow (MRC-01). Stripe first for Australia, subject to written provider/acquirer confirmation of PCI scope and platform terms.
**Consequences:** Restaurants must complete provider KYC before deposits/fees are enabled; refund and dispute flows are required; card holds last only days, so no-show charges use saved payment methods.
**Revisit when:** the provider changes platform terms, or an India launch requires a different provider.
**Related:** Doc 5, Doc 6 §7.2, Doc 11 E-S1.
