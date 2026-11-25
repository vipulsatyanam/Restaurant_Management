# Stage E — Critical Architecture Review

Scope: Documents 1–10, `decision-log.md`, research files. Reviewer stance: adversarial; the aim is to find what would hurt the business, not to defend the design.
Date: 2026-11-06. Status of all findings: **open until addressed or accepted by the owner.**

## Verdict

**The architecture is NOT declared production-ready.** It is a coherent *design baseline* suitable for the Phase 0 spikes, but the items marked **Critical** and **High** below must be resolved (by spikes, decisions or document changes) before Phase 1 begins or, where stated, before first customer go-live. Several statements in the documents rest on assumptions, unverified research or professional advice that has not been obtained; those are listed explicitly rather than treated as settled.

Severity: **Critical** = could end the business or block launch; **High** = serious, fix before the relevant phase; **Medium** = fix in normal course; **Low** = note.
Status after this review: every item is **Open** unless stated otherwise.

---

## 1. Findings

### 1.1 Security gaps
| ID | Sev | Finding | Recommended action | Where fixed / owner |
|---|---|---|---|---|
| E-S1 | High | **Restaurant merchant onboarding is missing.** Doc 1/5 assume each restaurant has its own merchant account for deposits and fees, but there is no flow, module or risk analysis for onboarding thousands of merchants (KYC, provider platform accounts, payout responsibility). Research: Stripe platform onboarding is generally available in Australia but not in India. | Add a "Merchant onboarding" capability to Integration Hub/Payments (provider platform-account model, KYC status, restrictions), decide who is merchant of record, and get provider/legal confirmation of PCI and licensing position. | Add to Doc 1/5; Phase 0 vendor spike S-3; counsel |
| E-S2 | High | **Public booking abuse resistance is only specified, not proven.** Fake holds could exhaust capacity of a popular venue; deposit flows can be used for card testing. | Add card-testing protections (velocity limits, minimum friction on deposit flow, provider fraud tools), bot-challenge selection, hold-to-confirm ratio monitoring, per-venue "hold budget". Test in Phase 2. | Doc 5/6 update; Phase 2 tests |
| E-S3 | Medium | **Guest manage-link tokens** are bearer secrets in URLs (messages, browser history, referrers). | Single-purpose, expiring, rotate on use for sensitive actions; `Referrer-Policy: no-referrer`; limit what a token reveals. | Doc 5 §3.1 addendum |
| E-S4 | Medium | **Support access model unresolved**: whether customers must pre-approve, and tenant-visible logging details. | Decide in contract terms; implement time-boxed, tenant-visible access either way. | Owner decision (Doc 3 #2) |
| E-S5 | Medium | **Secrets for tenants' integration credentials** (Square OAuth tokens) are a high-value target; rotation and revocation flows are only described. | Define token storage, refresh, revoke-on-disconnect, and monitoring of unusual use; include in threat model update. | Doc 6 update |
| E-S6 | Medium | **Cheery internal staff/partner portal** shares infrastructure with tenant surfaces. | Separate hostnames, stricter WAF rules, IP/device policies for the platform realm; consider a separate deployment unit later. | Doc 7 update |

### 1.2 Data-isolation weaknesses
| ID | Sev | Finding | Recommended action | Where fixed |
|---|---|---|---|---|
| E-I1 | **Critical** | **RLS correctness through connection pooling is untested.** Research notes RDS Proxy may pin connections on `SET`; the entire isolation model depends on transaction-scoped context working with the chosen pooler. | Spike S-2 is a **hard gate (G1)**: prove no context leakage under concurrency with the selected pooler, and that failures fail closed. | Phase 0 |
| E-I2 | High | **Branch-level isolation is application-enforced only** (RLS covers tenant). A scope bug leaks between branches of the same customer — plausible in a 100-branch group with delegated managers. | Central scope filter in the data-access layer (not per-query code); authorisation matrix tests across every endpoint; optional branch RLS for sensitive tables (guest notes) decided at Phase 1 with performance measurement. | Doc 3/8; Phase 1 |
| E-I3 | High | **Tenant directory is a global single point of failure and a global trust anchor** (routing, cell mapping). A bug or outage affects all tenants; a write bug could mis-route. | Cache with short TTL, read-only replicas, change via audited platform API only, integrity checks, runbook; consider embedding tenant→cell in signed token claims. | Doc 3/7 update |
| E-I4 | Medium | **`person` is global and platform-level**: staff who belong to several tenants link identities across customers. | Document the privacy position; tenants see only their own membership data; consult counsel on cross-tenant identity linkage. | Doc 4/6; counsel |
| E-I5 | Medium | **Guest cross-brand sharing** within an organization is an option (GST-05) but the consent and privacy-notice implications are undefined. | Default off; require notice and consent wording per organization; DPIA before enabling. | Doc 1/6 |
| E-I6 | Medium | **Time-ordered UUIDs** expose creation time; combined with enumeration attempts. | Never expose raw ids in public URLs; use references/tokens; accepted for internal ids. | Doc 4 note |

### 1.3 Scalability risks
| ID | Sev | Finding | Recommended action |
|---|---|---|---|
| E-C1 | High | **All load figures are assumptions** (bookings per branch, search ratios, messages per booking). | Obtain the Sydney group's real data; Phase 0 rig; re-baseline. |
| E-C2 | Medium | **Correlated evening peaks and special dates** (5× burst) stress the public edge, realtime gateway and message providers at the same time. | Pre-warm scaling for known dates; provider throughput limits tested; graceful degradation modes defined. |
| E-C3 | Medium | **Single primary database** is a shared failure and write bottleneck for S2+. | Cell design accepted; define cell ceilings from load tests; avoid cross-tenant locks. |
| E-C4 | Medium | **Reminder scheduling at volume** (scans/timers) can create thundering-herd effects. | Indexed schedule with bounded batch claim and per-tenant fairness; jitter. |
| E-C5 | Medium | **Hot branch-night contention** on the exclusion index when hundreds attempt the same tables. | Hot-spot test (Doc 9 §2.2); hold budget; short transactions. |

### 1.4 Unnecessary complexity (things to resist or simplify)
| ID | Sev | Finding | Recommendation |
|---|---|---|---|
| E-X1 | Medium | **Cells, country packs, configurable commission, lockable settings, composite tenant keys, RLS, outbox, ports for every provider** together are a lot of architecture for 10 customers. | Keep the *seams* but build only what Release 1 needs: one cell, Australia pack only (India/UK on paper), the smallest set of lockable settings, ports only where a second implementation is plausible (billing, payments, messaging, POS, identity). Review at G1. |
| E-X2 | Medium | **Maker–checker, immutable plan versions and a double-entry ledger** for a handful of partners. | Justified (money, disputes) but implement the minimum: append-only ledger and versioned plans; skip workflow engines; approval is a simple two-step. |
| E-X3 | Low | **Separate public-edge process** from day one. | Same codebase and image; a deployment option. Do not build separate repos or datastores. |
| E-X4 | Low | **Mutation testing on four modules** adds CI time. | Run nightly rather than per PR. |

### 1.5 Missing modules and capabilities
| ID | Sev | Missing item | Why it matters | Placement |
|---|---|---|---|---|
| E-M1 | High | **Customer data import / migration tooling** (existing bookings, guests, tables from the prior system) | The Sydney group and every later customer arrive with data; without import, onboarding is manual and error-prone | Phase 3; add to Doc 1 |
| E-M2 | High | **Merchant onboarding** (see E-S1) | Needed for deposits/fees | Phase 2/3 |
| E-M3 | High | **Per-tenant email authentication and sender configuration** (SPF/DKIM/DMARC on customer domains, sending identities) | Deliverability and spoofing risk | Phase 2 |
| E-M4 | Medium | **Message usage metering and allowance billing** | Pricing viability (Doc 9) | Added to Phase 3; add `usage_counter` (done in Doc 4) |
| E-M5 | Medium | **Venue-level legal content** (terms, privacy notice, cancellation policy text) managed per brand/country with versioning and consent capture | Legal exposure | Doc 1 add; Phase 2 |
| E-M6 | Medium | **Support tooling** (ticket links, tenant diagnostics, safe impersonation, tenant-visible audit view) | Operational burden for a small team | Phase 3 |
| E-M7 | Medium | **Product analytics and cookie/consent handling on booking pages** | Privacy compliance and product learning | Phase 2; counsel |
| E-M8 | Medium | **Customer-facing status/notification and in-app communications** (maintenance, incidents) | Trust | Phase 3 |
| E-M9 | Low | Reviews/ratings, referral programmes, CRM marketing campaigns | Not required now | Later |
| E-M10 | Medium | **Tenant offboarding/export packaging** is described but not specified (formats, size, delivery) | Contractual expectation | Doc 3/4 detail in Phase 3 |

### 1.6 Integration risks
| ID | Sev | Finding | Recommended action |
|---|---|---|---|
| E-N1 | High | **India provider constraints** (payments, billing, SMS registration) are unresolved; research found Stripe invite-only and platform onboarding limited in India. | Do not commit an India date until ADRs 04–06 are settled; plan an India-capable provider early. |
| E-N2 | Medium | **Square integration details unverified** (API scopes, rate limits, review/approval process, data model mapping, webhook behaviour). | Spike with sandbox in Phase 0/2; read official docs; confirm app-approval lead times. |
| E-N3 | Medium | **Reserve with Google** requirements not confirmed. | Verify before promising. |
| E-N4 | Medium | **Provider lock-in** if ports leak vendor concepts. | Keep ports behavioural; run adapter contract tests against fakes and sandboxes. |
| E-N5 | Medium | **Billing vendor model vs. per-branch + usage + multi-currency** may not fit one vendor. | Evaluate with real plan catalogue in S-3. |

### 1.7 Technical-debt risks
| ID | Sev | Finding | Recommendation |
|---|---|---|---|
| E-T1 | Medium | **Monolith boundary erosion** | Fitness checks from day one; module owners; quarterly boundary review. |
| E-T2 | Medium | **Query-builder vs ORM choice** may be revisited after code exists. | Decide by spike; wrap in repositories to limit blast radius. |
| E-T3 | Medium | **Country-pack abstraction built from one country** may be wrong. | Paper-test India/UK; plan a refactor budget at the first non-AU launch. |
| E-T4 | Medium | **AI-generated code volume** can outpace review capacity. | Small PRs, review budget, mandatory tests, architectural lint. |
| E-T5 | Low | **Documentation drift** across ten documents. | ADR discipline; the index lists the source of truth for each decision. |

### 1.8 Compliance gaps (all require professional input)
| ID | Sev | Finding | Action |
|---|---|---|---|
| E-L1 | High | **No legal review has occurred**: processor/controller roles, sensitive-data classification of allergies, consumer-law wording for deposits/cancellation, messaging consent per country, retention periods. | Engage counsel in Phase 0; budget it. |
| E-L2 | High | **PCI scope depends on provider/acquirer confirmation** and, for iframe-hosted fields, script-integrity controls. | Written confirmation before pilot; script inventory process. |
| E-L3 | Medium | **Research files contain UNVERIFIED regulatory items** (India DPDP rule dates, Australian tax pages, ISO 27002 edition, several URLs). | Re-verify from primary sources with counsel before any compliance statement to customers. |
| E-L4 | Medium | **Commission payouts to contractors** have tax and employment-classification implications per country. | Accountant/legal advice before first payout. |
| E-L5 | Medium | **Data residency and sub-processor locations** (identity, messaging, billing, monitoring vendors) unverified. | Vendor due diligence; register. |

### 1.9 Operational risks
| ID | Sev | Finding | Recommendation |
|---|---|---|---|
| E-O1 | High | **Support/on-call model undefined** while customers run live service on the system on weekend evenings. | Decide hours and staffing before pilot; tie to contract. |
| E-O2 | Medium | **Small-team key-person risk** | Deputies per module; runbooks; documentation standards. |
| E-O3 | Medium | **Untested backups/restore and DR** until exercised | Scheduled tests; first test before pilot. |
| E-O4 | Medium | **Cost visibility**: messaging and SaaS vendors can erode margin silently | Cost tags, budgets, monthly review; metering. |
| E-O5 | Medium | **Release during service** | Peak-hour freeze enforced in pipeline. |

### 1.10 Business and commercial risks
| ID | Sev | Finding | Action |
|---|---|---|---|
| E-B1 | **Critical** | **Messaging cost can exceed the subscription price** under common pricing (Doc 9 §6.2–6.3). The 50% margin target after commission is not achievable if SMS is the default channel and the price is competitive. | Decide messaging policy and allowance/overage model **before** setting prices; verify real SMS unit prices for Australia and India. |
| E-B2 | High | **Margin definition unconfirmed** (lifetime-averaged vs year-1; contribution vs fully loaded). | Owner to confirm (decision-log D-26). |
| E-B3 | High | **Team size and composition unknown**, so schedule is unknowable. | Owner to provide. |
| E-B4 | High | **Contract with the Sydney group not reviewed**: SLAs, penalties, data location, exit terms may drive requirements. | Obtain and review. |
| E-B5 | Medium | **Customer-specific scope pressure** (JA² Club, surcharges, widget specifics) could pollute the core. | Classification process (core/configurable/integration/later/decline); no custom code (R-02). |
| E-B6 | Medium | **Competitor landscape thin on pricing facts**; research found few published prices. | Customer interviews and quotes. |
| E-B7 | Medium | **Commission plan economics**: 30% for 12 months per partner reduces early margin; lifetime commission would break the model. | Keep default 12-month; finance review of each partner plan. |

### 1.11 Consistency issues found between documents
| ID | Sev | Issue | Resolution |
|---|---|---|---|
| E-D1 | Low | Doc 5 referenced `integ.reconciliation_run` not defined in Doc 4 | **Fixed**: added to Doc 4 §4.11 |
| E-D2 | Low | Doc 2 lists a Reporting module with read models; Doc 4 had no reporting tables | **Fixed**: added §4.11a |
| E-D3 | Low | Doc 9 requires message metering; Doc 4 lacked a table | **Fixed**: `integ.usage_counter` added |
| E-D4 | Low | Doc 1 R1 scope did not mention usage-based billing for messages | Add to Doc 1 SUB requirements when finalised (Doc 10 already includes) |
| E-D5 | Low | Doc 6 lists owner roles that don't exist yet | Replace after team structure is known |
| E-D6 | Low | Several research-sourced facts carry `[S]`/UNVERIFIED in source files but appear as plain statements in some documents | Consolidated list in the index; verify before commitments |
| E-D7 | Low | The decision log contains one note that a "preliminary stack" differs slightly from final (Valkey, OpenTofu) | Doc 2/7 are the source of truth |

---

## 2. Required before Phase 1 starts (gate G1)

1. Spike S-1 (reservation engine & concurrency) passes.
2. Spike S-2 (RLS + pooler) passes — **E-I1**.
3. Vendors decided for identity, billing, messaging, WAF; India impact recorded (E-N1).
4. Messaging policy decided and unit prices verified (E-B1).
5. Owner provides team size, margin definition, Sydney group contract and real volumes (E-B2…B4, E-C1).
6. Counsel engaged (E-L1).

## 3. Required before the pilot (gate G3)
Penetration test and remediation; PCI scope confirmation in writing (E-L2); merchant onboarding working (E-S1/M2); data import tooling (E-M1); email authentication (E-M3); support model and on-call defined (E-O1); restore test passed (E-O3); abuse protections tested (E-S2).

## 4. Items accepted as risk (proposed)
- E-X3 (separate process from day one) — accepted as a deployment option only.
- E-M9 (reviews/referrals) — postponed.
- E-S6 (platform realm hardening beyond separate hostnames) — revisit with enterprise demand.

## 5. Conclusion
The design is internally consistent and addresses the stated priorities (tenant isolation, concurrency safety, configurability, replaceable integrations, country flexibility). Its principal weaknesses are *unvalidated assumptions* (volumes, pooler behaviour, vendor capabilities), *missing commercial/legal inputs* (messaging economics, contract terms, counsel advice, team size) and a few *omitted capabilities* (merchant onboarding, data import, email authentication). None of these requires redesign, but several can invalidate schedule, pricing or specific technical choices, which is why the plan front-loads spikes and decisions.

---

## 6. Update after owner answers (2026-11-07)

| Item | Change | Status |
|---|---|---|
| E-B3 team size | Team of 10 (D-29); composition unknown. Calendar recomputed in Doc 10 §10 (≈ 5.5–9 months for Release 1, low confidence) | Partly resolved |
| E-B4 Sydney contract | **No written terms exist** (D-30). No contractual SLA constraints, but also no terms of service, data-processing agreement, or exit terms. Real guest data should not be processed without at least minimal terms | **Raised to High**; decide before pilot |
| E-C1 volumes | One branch only (D-32); per-branch numbers still wanted; group-level load stays assumed | Partly resolved |
| E-O1 support model | Business hours at launch (D-31); evening/weekend gap acknowledged; mitigations in Doc 7 §7.3 | Resolved with accepted risk |
| E-L1 legal review | **No budget decision yet** (D-34) for counsel or penetration test. These remain gate items for G3. | **Open — owner decision before G3** |
| New E-B8 (High) | **The pilot customer has one branch**, so multi-branch hierarchy, inheritance, group roles and org-wide reporting are untested by any real customer | Find a multi-branch design partner or run structured multi-branch simulations; do not claim group features as proven |
| New E-L6 (High) | Pilot without terms, a data-processing agreement or counsel review means processing guest personal data (including allergy notes) with no legal basis documented in a contract | Minimum viable legal set before pilot: (1) customer terms of service, (2) data-processing terms with sub-processor list, (3) privacy notice and consent wording for booking pages, (4) cancellation/deposit wording check. These can be obtained at modest cost from a qualified adviser; the budget decision is the owner's. Mitigation if delayed: collect minimal data, disable sensitive-notes and deposits for the pilot. |
| New E-O6 (Medium) | Business-hours support but evening-heavy usage | Honest terms; automated alerts; Sev-1 best-effort escalation during pilot; no 24×7 promise |
| E-N5/A5 POS | Target customers' POS systems unknown | Customer survey in Phase 0; Square remains the only planned adapter |

## 7. Update 2026-11-08
| Item | Change |
|---|---|
| E-B8 (one-branch pilot) | Owner will add more pilot branches (D-35). Mitigated once ≥ 2–3 real branches under one organization are live; which branches/customers is still to be stated. |
| E-L6 (no terms/DPA) | Draft legal pack written in `docs/legal/` (D-36). Status: **drafts only, no lawyer review yet**; E-L1/E-L6 stay open until a qualified adviser reviews and the owner decides the open commercial placeholders (entity, prices, liability, data-location promise, pilot terms). |
