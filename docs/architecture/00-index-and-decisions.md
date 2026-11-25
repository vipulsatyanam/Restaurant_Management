# Cheery Architecture Blueprint — Index and Owner Decisions

Status: **documentation complete for review; no implementation has started.** Stage F (solution skeleton, repository, CI, backlog) begins only after the owner reviews and approves this set.
Prepared 2026-11-06. Working language: documents are drafts for review; every recommendation is labelled and nothing is declared production-ready (see the architecture review).

## 1. Reading order

| # | Document | Purpose |
|---|---|---|
| — | `stage-a-prototype-analysis.md` | What the prototype is and is not; feature inventory and risks |
| — | `decision-log.md` | Every confirmed decision (D-xx), accepted recommendation (R-xx) and open item |
| — | `stage-c-architecture-options.md` | Options compared, recommendation, build-now / prepare / postpone (approved) |
| 1 | `01-product-and-requirements.md` | Goals, classified feature inventory, personas, workflows, numbered requirements, NFRs |
| 2 | `02-solution-architecture.md` | Principles, style, diagrams, module contracts, dependency rules, ADR index |
| 3 | `03-multi-tenant-architecture.md` | Hierarchy, isolation, roles/permissions, inheritance, lifecycle, cell migration, isolation tests |
| 4 | `04-database-architecture.md` | Tables, double-booking constraint, RLS, indexes, migrations, retention, backups |
| 5 | `05-api-and-integration-architecture.md` | API conventions, endpoints, provider ports, events, webhooks, resilience, API security |
| 6 | `06-security-and-compliance.md` | Threat model, controls, risk register, standards and country mapping, privacy, checklist |
| 7 | `07-infrastructure-and-devops.md` | Cloud, environments, CI/CD, releases, observability, backup/DR |
| 8 | `08-engineering-standards.md` | Code, structure, testing, review, documentation, Definition of Done |
| 9 | `09-performance-and-scalability.md` | Workload scenarios, targets, scaling, cost model, plan structure |
| 10 | `10-roadmap-and-implementation-plan.md` | Phases, deliverables, acceptance, effort ranges, go-live prerequisites |
| E | `11-architecture-review.md` | Critical review: findings by category, verdict, required gates |
| R | `research/*.md` | Compliance scan, competitor scan, stack verification (many items marked UNVERIFIED) |

## 2. What is decided (approved by the owner)
Organization = tenant; hybrid multi-tenancy (shared database + row-level security, cell-ready); modular monolith with event-driven internals; TypeScript/NestJS backend; AWS Sydney; third-party billing; reservations-first Release 1; POS integration first (Square read-only), native POS later; sales-partner commission configurable (default 30%, per-partner plans, clawback, contractors, management assigns leads); no percentage of guest payments; ~50% margin target after commission; offline later; marketplace a future goal but not built now; Sydney group signed, no hard deadline. (See `decision-log.md` D-01…D-28.)

## 3. Owner decisions and inputs needed (consolidated)

### A. Inputs only the owner can provide (block estimates or pricing)

> **Update 2026-11-07 (see `decision-log.md` D-29…D-34):** A1 answered — team of 10, composition still needed. A2 answered — **no written contract terms exist** (also means no terms of service or data-processing terms yet). A3 answered in part — the group has **one branch**; rough per-branch numbers still wanted. A5 answered — **unknown**; survey added to Phase 0. A6 answered — **business hours at launch**. A7 — **no budget decision yet**; deferred, still required before the pilot gate or the risk must be explicitly accepted. A4 (margin definition) still to confirm.
| # | Input | Why |
|---|---|---|
| A1 | **Team size and composition** (engineers, design, QA, DevOps) | Turns effort ranges into dates (Doc 10) |
| A2 | **Sydney group contract**: SLAs, penalties, data location, exit terms, support hours, scope promised | May change NFRs, support model, residency |
| A3 | **Sydney group real volumes** (bookings/month, online share, peak times, covers) | Replaces all load assumptions (Doc 9) |
| A4 | **Margin definition**: lifetime-averaged contribution margin per branch (assumed) vs year-1 vs fully loaded | Price floor (D-26) |
| A5 | **Other POS systems** used by target customers | Adapter roadmap |
| A6 | **Support model** (24×7 vs business hours + evenings) | Staffing, cost, contract |
| A7 | Budget for counsel, penetration test, vendors | Phase 0/3 planning |

### B. Decisions to approve (recommended answer first)
| # | Decision | Recommended | Doc |
|---|---|---|---|
| B1 | Messaging policy: email/in-app first, SMS where valuable, allowances + overage billing in Release 1 | Approve — **critical to pricing** | 9, 11 (E-B1) |
| B2 | Role set and permission keys (Release 1) | Approve | 3 |
| B3 | Support access model: time-boxed, reasoned, tenant-visible; customer pre-approval or not | Time-boxed + visible; pre-approval per contract | 3, 6 |
| B4 | Behaviour of restricted/suspended tenants (does guest booking stop?) | Needs business/contract input | 3 |
| B5 | Retention window after closure (30 days) and default retention table | Approve as provisional pending counsel | 3, 4 |
| B6 | Shared-tablet PIN sign-in: allowed or not | Not in Release 1 unless needed | 1, 6 |
| B7 | Phone verification for deposit bookings | Off by default, on risk signals | 5 |
| B8 | Payments: do not confirm payment-required bookings if provider is down | Approve; confirm with Sydney group | 5 |
| B9 | No production deploys during local dining peaks (e.g. 5–10 pm) | Approve; confirm hours | 7 |
| B10 | RPO ≤ 5 min / RTO ≤ 4 h; region-loss RTO ≤ 24 h | Approve as proposals | 7, 9 |
| B11 | SLO targets (99.9% booking/host availability) | Approve as proposals | 7, 9 |
| B12 | Independent penetration test before first customer | Approve | 6 |
| B13 | Engage Australian privacy/commercial counsel now; India/UK before those launches | Approve | 6, 11 |
| B14 | Engineering standards and quality gates (Doc 8) incl. SQL-first data access | Approve | 8 |
| B15 | Revised phase sequence and mandatory Phase 0 spikes (gate G1) | Approve | 10 |
| B16 | Draft plan structure (Starter/Growth/Enterprise + add-ons) as a hypothesis; prices after cost data and interviews | Approve | 9 |
| B17 | Accessibility standard WCAG 2.2 AA | Approve | 1 |
| B18 | Capacity display ("seats left") default | Off by default, per-branch setting | 1 |
| B19 | Reserve with Google: defer until requirements verified | Approve | 1, 5 |
| B20 | Add capabilities found by the review: merchant onboarding, data import, tenant email authentication, venue legal content, support tooling | Approve | 11 |

### C. Vendor decisions (made in Phase 0 spikes, not now)
Identity (WorkOS vs Cognito); billing (Stripe Billing vs Chargebee, India constraints); guest payments per country; messaging providers (AU sender ID, India DLT); WAF/edge (Cloudflare vs AWS WAF); observability backend; feature-flag service; query-builder library; RDS vs Aurora and pooler.

## 4. Gates before implementation can begin
1. Owner reviews and approves this documentation set (G0).
2. Section 3A inputs supplied (at least A1, A3, A4).
3. Items in `11-architecture-review.md` §2 scheduled for Phase 0.

## 5. Not done, by design
No application code, solution skeleton, migrations, repository scaffolding, CI configuration or backlog have been created (Stage F). Nothing has been committed to version control.

## 6. Source-of-truth map
| Topic | Authoritative document |
|---|---|
| Requirements and scope | Document 1 |
| Module boundaries, dependency rules, ADR index | Document 2 |
| Tenancy, roles, inheritance, lifecycle | Document 3 |
| Tables, constraints, retention | Document 4 |
| API/events/providers | Document 5 |
| Security, privacy, compliance | Document 6 |
| Infrastructure, release, DR | Document 7 |
| Coding/testing/review rules | Document 8 |
| Targets, capacity, cost | Document 9 |
| Phases, effort, gates | Document 10 |
| Open risks | Document 11 |

## 7. Legal pack (added 2026-11-08, decision D-36)
Drafts for lawyer review live in `docs/legal/`: `00-legal-pack-plan.md` (what is needed, when, and what is still missing), `01-customer-terms.md`, `02-data-processing-terms.md` (with sub-processor register template and security/breach schedules), `03-privacy-and-consent-templates.md`, `04-sales-partner-agreement-outline.md`. **They are drafts, not legal advice, and have not been reviewed by a lawyer.** They do not guarantee that no legal issue arises. Lawyer review remains a gate before the pilot (E-L1, E-L6).

## 8. Update 2026-11-08
D-37 (≈30 bookings/week at the Sydney branch), D-38 (no product or legal name yet — company set-up is now an early critical-path item), D-39 (Twilio intended for messaging; list prices captured in Doc 9 §12). Messaging policy decision (B1) still open but its urgency is lower at the Sydney branch's volume; it remains important before pricing.

## 9. ADRs and runbooks (added 2026-11-25)
Individual decision records are in `adr/` (index: `adr/README.md`; 3 Accepted, 7 Open/Deferred, 6 Proposed). Draft operational runbooks are in `../runbooks/` (index and severity levels: `../runbooks/README.md`). Runbooks use placeholder tools and must be rehearsed in staging before the pilot.
