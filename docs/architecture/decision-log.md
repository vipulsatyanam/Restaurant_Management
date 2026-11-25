# Cheery — Decision Log (Stage B working record)

Status key: **Confirmed** = stated by the product owner. **Recommended** = proposed, awaiting approval. **Open** = unanswered.

## Confirmed

| # | Decision | Source | Architectural consequence |
|---|---|---|---|
| D-01 | Near term: **integrate** with third-party POS systems (e.g. Square). Future: add a **native POS module** so Cheery becomes a one-stop restaurant platform. | Product owner, Stage B | Ordering, payments and POS-facing data must sit behind provider-neutral ports so an external POS and a future native POS are interchangeable sources. Build the integration boundary first; do not build native POS now, but do not design it out. |
| D-02 | The HTML prototype is **context, not specification**. It shows what *might* be in the product; it is not 100% correct. Architecture should favour what is most applicable and usable for restaurants. | Product owner, Stage B | Prototype features are not requirements by default; each needs classification (core / configurable / later / drop). Prototype data model is not binding. |
| D-03 | Hierarchy needs a real **tenant level** with **inheritance** (organisation → brand → branch), not the prototype's flat, global rules/menu. | Product owner, Stage B | Configuration, menu/pricing and rules designed with scoped overrides. |
| D-04 | Country behaviour (currency, tax, language, payments, etc.) must be **flexible and data-driven** so any country can launch with **minimum modification and no blocker**. Prototype country settings are illustrative only. | Product owner, Stage B | "Country pack" configuration model; no country constants in core code; country onboarding is configuration plus adapters, not a code fork. |

## Accepted recommendations (Stage B batch 1, accepted as a set)

| # | Decision | Consequence / still needed |
|---|---|---|
| R-01 | Release 1 = SaaS core (tenant, identity, subscription) + branch/table setup + reservations, guests, waitlist, host/floor ops, notifications. QR ordering, KDS and in-restaurant payment later, behind POS-neutral ports. Deposits/no-show fees use a provider-hosted payment page (no card data on our systems). | Needs a payment provider shortlist (batch 2). |
| R-02 | The Sydney group is design partner / first customer, with **no customer-specific code**. Group-specific items (JA² Club, weekend surcharge, nopCommerce widget) become configuration, integrations, or later scope. | Open: signed customer with launch date, or prospect? |
| R-03 | First POS integrations are **read-only** (sales/spend into guest history; link bookings to POS checks). Order push deferred. Square (AU) first. | Open: other POS vendors used by target customers. |
| R-04 | Validate the configuration model on paper against Australia, India and one third country with a different tax/payment shape (UK or US suggested). | Open: confirm the third country. |
| R-05 | Assisted onboarding for restaurant groups first; tenant creation exposed as an API so self-serve can be added later. | |
| R-06 | **No cross-tenant consumer marketplace or shared guest identity in the initial releases.** Guest data stays inside each tenant. Booking via tenant's own site/page, optional Reserve with Google. | **Update (product owner): a marketplace IS a goal in a future phase.** Keep the door open without building toward it: tenant guest data stays tenant-owned; any future marketplace uses a separate consumer identity and per-guest, per-tenant consent for sharing; restaurants opt in to listing; public availability and listing data exposed via a defined public-read boundary rather than direct access to tenant tables. Nothing marketplace-specific is built now. |

## Stage B batch 2 — answers received

| # | Decision | Consequence / still needed |
|---|---|---|
| D-05 | **Technology is not a constraint** (development is done with Claude Code). Architect must audit and recommend the best-fit stack for an enterprise SaaS that will keep gaining modules. | Stack recommendation is an ADR set, to be verified against current sources in Stage C. |
| D-06 | **Sales-partner commission is a core business requirement.** Cheery employs salespeople who onboard restaurants; they earn **30% commission** on sales. Rate must be **configurable**. Needs a module/system to **track commission, payouts and balances remaining**. | New domain: Partner & Commission. Open: 30% of what, for how long, payout rules (see conversation). |
| D-07 | Year-1 scale target: **10 customers, ~100 branches.** | Launch scenario sizing confirmed. |
| D-08 | **Third-party billing service** for subscriptions (not built in-house). | Billing port + adapter; vendor selection ADR. |
| D-09 | Plan tiers and prices: owner wants architect's help drafting them. | Needs competitor scan and unit economics incl. commission. |
| D-10 | **Offline is speculative → out of release 1**, but needed in a later release. | Keep cheap offline-readiness in the design (client-generated IDs, idempotent writes, event-style change records); no offline client built in release 1. |

## Commission and payments — answers received

| # | Decision | Consequence |
|---|---|---|
| D-11 | **Cheery takes no percentage of restaurants' guest payments.** Restaurants use their own merchant accounts; Cheery's revenue is subscriptions/add-ons. | Payment layer is orchestration only; no platform fee/settlement/payout of guest funds. Lighter licensing and PCI scope. |
| D-12 | Sales commission is **30% by default, paid to salespeople for onboarding new restaurants**; mostly on subscription revenue, but the **commission basis is configurable** (e.g. subscription, add-ons, setup fees; tax/discount treatment). | Commission rule has a configurable revenue-basis selector. |
| D-13 | **Commission duration is configurable**, and **each salesperson can have their own configuration** (rate, basis, duration). | Per-salesperson commission plan with effective dates and versioning; changes never rewrite past earnings. |
| D-14 | **Clawback applies**: commission is reversed if a customer cancels or does not pay within a period. | Clawback window configurable (default proposed 90 days); reversals tied to refund/chargeback/cancellation events. |
| D-15 | **Attribution is decided by top management**, not by automatic rule. | Lead-ownership assignment is a manual, permissioned, audited action; conflicting claims go to an approval queue. |
| D-16 | Salespeople are **contractors to start with.** | Payout design assumes contractor payments; tax treatment stays configurable per country and needs accountant/legal confirmation. |

## Defaults accepted

| # | Decision | Consequence |
|---|---|---|
| D-17 | Commission payouts **monthly with a minimum balance** (both configurable). | Payout run is a scheduled, approvable batch. |
| D-18 | **No SOC 2 / ISO 27001 certification required at launch.** Build to the standards; certify when a customer requires it. Legal compliance (privacy, tax, payments) is a separate track needing professional advice. | Controls mapped to standards from day one; evidence collection kept cheap. |
| D-19 | **AWS, Sydney region** for the Australian launch; separate regional stacks later for data residency. | Cloud ADR starts from AWS ap-southeast-2. |
| D-20 | Validate the country-configuration model against **Australia, India and the UK.** | Paper-test the country pack on all three before building. |

## Customer and POS facts — answers received

| # | Decision | Consequence |
|---|---|---|
| D-21 | **The Sydney group is a signed customer.** | Release 1 has a real first customer. Their contractual commitments (scope, dates, service levels) must be captured before the roadmap is fixed. Still no customer-specific code (R-02). |
| D-22 | The Sydney group **uses Square** as its POS. Other POS systems may be needed in future. | Square read-only integration (R-03) is a release-1 priority. POS access stays behind a vendor-neutral port with adapters, so another POS is a new adapter, not a core change. |

## Sydney group scope and pricing inputs — answers received

| # | Decision | Consequence |
|---|---|---|
| D-23 | **No hard launch deadline** for the Sydney group; time is available for proper planning and implementation. | Roadmap can include a Phase 0 reservation-engine spike and quality gates without schedule compromise. |
| D-24 | **First release is mainly reservations**; other modules are planned for later releases. | Confirms R-01. Ordering/KDS/QR stay out of release 1. |
| D-25 | **Cost per branch is unknown; owner expects about 50% margin.** | Cost per branch to be estimated in the performance/capacity document. The margin definition needs confirming (see open items); commission at 30% materially changes the result. |

| D-26 | The **~50% margin target is measured after sales commission.** Working assumption (mine, not yet confirmed): averaged over the customer lifetime, as a contribution margin per branch (not fully loaded). | Pricing model uses P ≥ C ÷ (1 − commission share − billing fee share − 0.5). With a 12-month 30% commission over a 3-year lifetime this is P ≥ C ÷ 0.37 (billing fee assumed 3%, unverified). A stricter year-1 reading would need P ≥ C ÷ 0.17. |

## Stage C approvals

| # | Decision | Consequence |
|---|---|---|
| D-27 | **Stage C recommendation approved** (`stage-c-architecture-options.md`): modular monolith with event-driven internals and a few process types from one codebase; hybrid multi-tenancy (shared database + row-level security, cell-ready); Organization = tenant (ADR-02, ADR-10); consolidated 12-module map; build-now / prepare / postpone split; revised phase order. | Stage D documents are written to this baseline. |
| D-28 | **ADR-01: backend language = TypeScript (Node LTS, NestJS).** .NET 10 is the documented runner-up. | Exact framework and runtime versions to be pinned in the engineering standards document. |

Still open and **not** approved by this decision: ADR-03 identity, ADR-04 billing, ADR-05 guest payments, ADR-06 messaging, ADR-07 hosting detail, ADR-08 WAF, ADR-09 offline (deferred).

## Answers after documentation review (2026-11-07)

| # | Decision / fact | Consequence |
|---|---|---|
| D-29 | **Team size is 10** (composition not yet given). | Doc 10 calendar recomputed assuming roughly 6–7 engineers plus design, QA, DevOps/security and product roles; composition still needed. |
| D-30 | **No written contract terms exist with the Sydney group** (no SLAs, penalties, data-location or exit terms). | No contractual NFR constraints today. But no terms of service or data-processing terms exist either: these must be created before real guest data is processed (see review E-L1/E-B4). |
| D-31 | **Support is business hours at launch**; 24×7 may be added later if the product succeeds. | Contracts and marketing must not promise 24×7. Evening service gap handled by automated alerting and a best-effort Sev-1 escalation during the pilot (recommendation). |
| D-32 | **The Sydney group has a single branch**, performing at an average level. | The pilot cannot validate multi-branch hierarchy, inheritance or group reporting. A multi-branch design partner or realistic simulated tenants are needed. Pilot volumes are small. |
| D-33 | **Other POS systems used by target customers: unknown.** | Square remains the only planned adapter; a customer survey is added to Phase 0. |
| D-34 | **No budget decision yet for legal advice and penetration testing.** | Deferred, not declined. Doc 10 gates (G3 pilot) still require both; owner must decide before G3 or accept the documented risk explicitly. |

## Open

- Confirm the working assumption in D-26: lifetime-averaged contribution margin, not year-1 and not fully loaded.
- Team composition (roles among the 10).
- Rough numbers for the Sydney group's single branch (bookings per week, share online/phone/walk-in, tables and seats, busiest-night covers).
- Budget decision for counsel and penetration test before G3.
- Vendor decisions: ADR-03 to ADR-08.

## Pilot branches and legal documentation (2026-11-08)

| # | Decision / fact | Consequence |
|---|---|---|
| D-35 | **Owner will add a few more branches for the pilot and initial testing** (in reply to review finding E-B8). Which branches/customers is not yet stated. | Allows real multi-branch validation (hierarchy, inheritance, group roles, cross-branch reports) before GA. Doc 10 risk 6 is partly mitigated once at least 2–3 real branches under one organization are live. |
| D-36 | **Owner asks for all legal documentation and terms to be prepared** so as to avoid legal issues. | Draft legal pack written in `docs/legal/` as **drafts for review by a qualified lawyer**. Drafting does not make Cheery legally compliant and cannot guarantee no legal issue; counsel review remains a gate (E-L1, E-L6). |
| — | Process note | A previous reply to the owner's message of 2026-11-07 ("reply 1") did not address it; this entry and the legal pack answer it. |

## Answers on volumes, naming and messaging (2026-11-08)

| # | Decision / fact | Consequence |
|---|---|---|
| D-37 | Sydney branch volume is **about 30 bookings per week** (≈ 130/month); other numbers (online split, tables, busiest night) not provided. | Per-branch assumption in Doc 9 (600/month) is ~4–5× this one real data point; it is used only to calibrate cost and test data, not to limit design. |
| D-38 | **No product name and no legal entity name decided yet.** "Cheery" is treated as a working name. | Legal pack stays in placeholders; company set-up (entity, ABN/GST, bank, vendor accounts) and name/trade-mark/domain clearance become early critical-path items. |
| D-39 | **Messaging: Twilio is the intended provider, "will see"** (tentative). | ADR-06 narrowed to "Twilio preferred behind the messaging port; confirm by a short RFQ-style check". Twilio list prices checked below (unverified currency). |
