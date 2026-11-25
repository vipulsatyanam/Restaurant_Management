# Document 9 — Performance, Scalability, Capacity and Cost

Baseline: Documents 1–8, `decision-log.md` (D-07 year-1 target ≈ 10 organizations / 100 branches; D-25/D-26 margin).
Tags: **[C]** confirmed · **[R]** recommended · **[A]** assumption · **[Q]** open.

> **Important honesty note.** There is no production traffic yet. Every workload figure below is **derived from stated assumptions (marked [A]) by arithmetic you can check**, not measured. The first job of Phase 0–2 load testing is to replace these assumptions with measurements. Unit prices of third-party services are **illustrative placeholders**, not quotes.

---

## 1. Growth scenarios

| Scenario | Organizations | Branches | When it matters |
|---|---|---|---|
| **S1 – Launch (Year 1 target [C])** | 10 | ~100 | First Australian customers |
| **S2 – Growth** | 100 | ~1,000 | Proven product-market fit in Australia |
| **S3 – Scale** | 1,000 | ~10,000 | Multiple countries / large groups; likely multiple cells |

The brief also asks about "large volumes of concurrent guest reservations", "peak dining-hour traffic" and "large numbers of simultaneous kitchen and waiter operations". Release 1 includes reservations and host operations only; kitchen/waiter volumes are treated as a **future load profile** (§7).

---

## 2. Workload model

### 2.1 Assumptions per branch **[A]** (to be validated with the Sydney group's real data)

> Update (D-32): the Sydney group has **one branch** performing at an average level, so its data validates *per-branch* behaviour only (bookings per month, online share, peaks), not multi-branch or group-level load. Group-level load stays a modelled assumption until a larger customer exists.

| Parameter | Assumed value | Range to test | Basis |
|---|---|---|---|
| Bookings per branch per month | 600 (≈ 20 per open day) | 300–1,200 | Reasonable for a mid-size venue; **unverified** |
| Share created online | 60% | 30–80% | Assumption |
| Messages per booking | 3 (confirmation + 2 reminders) | 2–5 | From the prototype's template set; cancellations/waitlist add more |
| Availability searches per confirmed online booking | 10 | 5–50 | Search-to-book ratio assumption; bots add traffic |
| Busiest-hour share of a day's online requests | 10% | 8–20% | Evening peaks |
| Peak-day factor vs average day | 1.5× | 1.2–3× | Weekends and special dates |
| Burst factor on special dates / releases | 5× on the busiest hour | up to 10× | Valentine's/Mother's Day, popular-venue releases |
| Staff devices connected per branch | 3 | 2–6 | Host tablet, manager, spare |
| Share of branches active in the evening peak | 80% | — | Evening peaks are time-zone correlated across a country |

### 2.2 Derived load (arithmetic from §2.1)

Average-day online bookings per branch = 600 × 0.6 ÷ 30 = **12/day**; peak day ×1.5 = 18; busiest hour 10% = **1.8 bookings per branch in the busiest hour**.

| Metric (peak day, busiest hour) | S1 (100) | S2 (1,000) | S3 (10,000) |
|---|---|---|---|
| Online booking confirmations / hour | 180 | 1,800 | 18,000 |
| …per second (average in that hour) | 0.05 | 0.5 | 5 |
| Availability searches per second (×10) | 0.5 | 5 | 50 |
| …with 5× special-date burst: confirms/s | 0.25 | 2.5 | 25 |
| …with 5× burst: searches/s | 2.5 | 25 | 250 |
| Staff realtime connections (evening) | ~240 | ~2,400 | ~24,000 |
| Messages per month (3/booking) | 180,000 | 1.8 million | 18 million |
| Messages per day (average) | 6,000 | 60,000 | 600,000 |

**Reading:** the **steady-state request rates are small** even at S3 (tens to a few hundred requests per second at the public edge). A single well-indexed PostgreSQL primary with a replica can serve S1 and S2 comfortably, and S3 is a question of cells, connections and message throughput, not raw query speed. These conclusions depend on the assumptions; the load tests in §8 confirm or overturn them.

Hot-spot scenario (the real concurrency risk) **[R test case]**: one popular branch releases a Saturday night with 20 tables and receives 500 hold attempts within 60 seconds (≈ 8/s on the same resources). Pass criteria: no double booking, no deadlocks, ≤ 20 holds succeed per free-resource slot as capacity allows, p95 hold latency < 800 ms, remaining requests receive a clean `slot_unavailable` with alternatives.

### 2.3 Data growth **[A]**
Per branch per month (retained rows, including ~1.75× index overhead): bookings 0.9 MB, allocations 0.24 MB, booking events 1.5 MB, audit 3 MB, messages and attempts ~1.9 MB, guests 0.3 MB, other ~0.4 MB ≈ **8 MB raw, ≈ 14 MB with indexes**; outbox (~3.5 MB) is purged within weeks.

| | S1 | S2 | S3 |
|---|---|---|---|
| Growth per month | ~1.4 GB | ~14 GB | ~140 GB |
| Growth per year | ~17 GB | ~170 GB | ~1.7 TB |

Implications: S1–S2 fit comfortably in one database; S3 requires archiving/partitioning (audit, messages, outbox, completed allocations — Doc 4 §9) and splitting into multiple cells (e.g. by country or by customer size).

---

## 3. Performance targets **[R, proposals]**

Server-side latency measured at the service boundary; client-perceived latency adds network time.

| Journey | Target |
|---|---|
| Public availability search | p95 < 500 ms, p99 < 1 s |
| Hold creation | p95 < 800 ms |
| Booking confirmation (excluding hosted payment page time) | p95 < 800 ms |
| Host actions (seat, move, extend, cancel) | p95 < 300 ms |
| Host day view load | p95 < 1 s for a busy branch-day |
| Realtime update visible on other devices | p95 < 2 s from commit |
| Reminder sent after scheduled time | 99% within 5 minutes |
| Hold expiry released | 99.9% within 1 minute |
| Report query (operational reports, one branch, 30 days) | p95 < 3 s; org-wide roll-up p95 < 10 s |
| Page load (guest booking page, mobile, mid-range device) | Largest contentful paint < 2.5 s (p75) |
| Staff web app interaction | Input responsiveness meeting current web-vitals "good" thresholds |

### 3.1 SLOs, SLIs and error budgets
As defined in Document 7 §7.2 (99.9% monthly availability for guest booking and host tools; ≈ 43 minutes per month budget; burn-rate alerting; release pause when exhausted). **Quarterly review** adjusts targets to measured reality.

### 3.2 RTO and RPO (consolidated proposals) **[R]**
| Scenario | RPO | RTO |
|---|---|---|
| Instance/task failure | 0 | Automatic, minutes |
| Availability-zone loss | 0 | Minutes (automatic failover) |
| Data corruption or erroneous change | ≤ 5 min | ≤ 4 h |
| Single-tenant data loss | ≤ 5 min | ≤ 4 h |
| Region loss | minutes–hours | ≤ 24 h (rebuild from code + backup copy) |
Customer contracts may require tighter numbers; the Sydney group's contract has **not been reviewed [Q]**.

---

## 4. Bottlenecks and scaling strategy

Likely order in which limits appear (and the response):

| # | Bottleneck | Why | Strategy |
|---|---|---|---|
| 1 | **Messaging cost and throughput** | Messages scale with bookings; per-message cost dominates variable cost (§6) | Channel policy (email/WhatsApp/in-app first, SMS where valuable), allowances and overage, provider throughput limits and queues, delivery receipts |
| 2 | Public edge during bursts and bot traffic | Unauthenticated, spiky | Edge caching of venue info; availability served from replica with short-lived cache; quotas; bot protection; separate process scaling |
| 3 | Database connections | Many containers × pool sizes | Pooler in transaction mode; connection budget per process; tune before scale-out |
| 4 | Hot tenant / hot branch-night | Concentrated demand | Per-tenant limits; short transactions; constraint-based concurrency; queue fairness |
| 5 | Audit/outbox/message table growth | Write-heavy history | Time partitioning (not on `allocation`), retention jobs, archival to cheaper storage |
| 6 | Reporting queries | Scans across branches | Read replica first, then precomputed read models fed by events; analytics warehouse deferred (Doc 2) |
| 7 | Realtime fan-out | Connection count correlated by evening peak | Stateless gateway scaled horizontally; per-branch channels; reconnect backoff with jitter; fall back to polling |
| 8 | Reminder scheduling scans | Timers at volume | Indexed schedule table with partial index and bounded batch claim (`SKIP LOCKED` style) per tenant fairness |
| 9 | Single-primary write capacity (S3) | One cell becomes large | **Cells** by country/customer size; dedicated cells for large tenants (Doc 3 §7) |

### 4.1 Scaling triggers (starting thresholds; tuned by measurement) **[R]**
| Signal | Threshold | Action |
|---|---|---|
| API/edge CPU or request queue depth | sustained > 60% / rising queue | Add tasks (autoscale) |
| Database CPU | > 60% sustained at peak | Index/query review → larger instance → replica reads |
| Database connections | > 70% of budget | Pooler tuning, reduce per-task pools |
| Replication lag | > 5 s sustained | Investigate; route sensitive reads to primary |
| Storage growth | Projected full in < 90 days | Archive/purge or resize |
| Job backlog age | > 2 min for reminders/holds | Add workers; check fairness |
| Messages per second vs provider limit | > 70% | Add provider/route; stagger sends |
| Cell size | Branches or data over agreed ceiling (set after load tests) | Create new cell, migrate tenants |

### 4.2 Horizontal scaling model
Stateless API/edge/worker/realtime tasks scale by count; state lives in PostgreSQL and object storage; no sticky sessions; the tenant directory routes to cells. Vertical database scaling and read replicas come before sharding; sharding inside a cell is **not planned** (cells serve that purpose).

### 4.3 Caching **[R]**
Start with none beyond HTTP caching of public venue data and in-process short-TTL caches for effective settings and entitlements (invalidated by events). Add a shared cache (Valkey) only when profiling shows a hot read path that cannot be fixed by queries/indexes. All cache keys include tenant; no cache stores secrets.

### 4.4 Search
Postgres indexes and trigram search for guest lookup by name/phone are sufficient for Release 1 (per-tenant data sizes are small). A dedicated search engine is **not introduced** until measurements show a need.

### 4.5 Files
Object storage with lifecycle rules; exports expire; uploads limited; delivery through the CDN with signed URLs.

---

## 5. Capacity planning process **[R]**
1. Phase 0: build a **load-test rig** (k6 or similar) with realistic data generators (two tenant shapes, 100/1,000/10,000-branch datasets).
2. Measure per-journey resource cost (CPU-seconds, DB time, queries, bytes) on the staging environment.
3. Convert measurements into capacity per task/instance and a safe utilisation target (≤ 60% at peak).
4. Project from observed growth monthly; review before onboarding any customer larger than a set threshold (e.g. > 30 branches).
5. Re-run the hot-spot and soak tests on every major release and before known peak dates (e.g. Valentine's Day, Mother's Day).

---

## 6. Cost model and unit economics

### 6.1 Cost of serving one branch (structure)
```
C(branch) = messaging + infrastructure share + third-party SaaS per-seat/MAU + monitoring share + payment/billing fees + support allocation
```
| Component | Driver | Status |
|---|---|---|
| Messaging | messages × unit price | Largest likely variable cost; unit prices **unverified** |
| Infrastructure share | (fixed baseline + variable) ÷ branches | Fixed baseline (prod + staging + observability) dominates at S1; to be priced with the cloud calculator in Phase 0 |
| Identity provider | per active user / SSO connection | Research: WorkOS free for first 1M MAU, SSO $125/connection; Auth0 B2B Essentials $150/mo at 500 MAU [P] |
| Billing service | % of revenue + per-transaction | Assumed 3% for modelling (**unverified**) |
| Monitoring/error tracking | volume tiers | Estimate in Phase 0 |
| Support | tickets × time | Staffing decision |
| Sales commission | 30% of subscription revenue for the first 12 months (default, configurable) | Treated as a cost per D-26 |

### 6.2 Messaging cost sensitivity (illustrative unit prices)
Cost per branch per month = messages × unit price. Messages = bookings × 3.

| Bookings / month | Messages | at $0.02 | at $0.04 | at $0.08 |
|---|---|---|---|---|
| 300 | 900 | $18 | $36 | $72 |
| 600 | 1,800 | $36 | $72 | $144 |
| 1,200 | 3,600 | $72 | $144 | $288 |

**Finding:** if reminders go by SMS at typical per-message prices, **messaging alone can exceed the subscription price** of low-priced competitors. Pricing and product must include a **messaging policy** (see §6.4).

### 6.3 Required price (from D-26)
Working definition (assumed, to be confirmed): 50% margin after commission, averaged over a 3-year life with 12-month 30% commission and 3% billing fees ⇒ price floor **P ≥ C ÷ 0.37** (derivation: commission share 30%×12/36 = 10%; billing 3%; margin 50% ⇒ C/P ≤ 1 − 0.10 − 0.03 − 0.50 = 0.37).

| Illustration (not a forecast) | C per branch | Required P |
|---|---|---|
| Messaging 1,800 × $0.02 + other variable $10 | $46 | ≈ $124 |
| Messaging 1,800 × $0.04 + other variable $10 | $82 | ≈ $222 |
| Messaging 1,800 × $0.04 with SMS limited to reminders only (900 SMS, rest email) + $10 | $46 | ≈ $124 |
| Messaging 900 × $0.02 + $10 | $28 | ≈ $76 |
(These ignore fixed platform costs and team cost; at S1 (100 branches) the fixed baseline per branch is large, so the margin at launch will be lower than the model's contribution margin.)

### 6.4 Levers (recommendations)
1. **Message policy:** confirmation by email/in-app link where possible; SMS reserved for time-critical items (2-hour reminder, waitlist-ready) or offered as an allowance; WhatsApp where cheaper and permitted (verify per country).
2. **Allowances and overage:** include a monthly message allowance per branch in each tier, with pass-through overage — this means **usage-based billing for messaging is needed earlier than previously assumed** (D-12 era assumption: "defer usage billing").
3. **Branch-size tiering:** volume discounts for groups, since fixed costs spread.
4. **Annual prepayment** to improve cash and reduce billing fees.
5. **Commission design:** the 12-month default is far cheaper than lifetime commission; keep configurable per partner (D-13).

### 6.5 Draft plan structure (hypothesis for validation, not final pricing) **[R]**
| Tier | Target customer | Includes | Notes |
|---|---|---|---|
| **Starter** | Single venue | Reservations, floor management, guests, confirmations by email, limited SMS allowance, 1 branch, basic reports | Self-serve later |
| **Growth** | Multi-branch groups | Starter + waitlist, deposits/card holds, group enquiries, roles per branch, org-wide reporting, Square read-only integration, larger message allowance | Per-branch price with volume bands |
| **Enterprise** | Large groups | Growth + custom roles, SSO, dedicated cell option, API/webhooks, data-residency options, SLA, priority support | Quote-based |
| **Add-ons** | Any | Extra message bundles, additional integrations, future modules (menu/ordering/KDS/QR, POS) | Priced when built |
**Price points are not proposed here** because they depend on validated costs (§6.2–6.3) and customer willingness to pay. Method: (1) set the floor from measured costs, (2) test the Growth and Starter price against 5–10 customer interviews and the observed competitor anchors (research: published figures exist only for a few vendors, e.g. Eat App 0/99/199/389 per month in an unstated currency; Square–OpenTable sync $129/location/month on its Plus plan), (3) set bands with the 30% commission in the model.

---

## 7. Future load profiles (prepared, not built)
- **Ordering/KDS/waiter:** each seated table generates tens of order events per service; per branch tens of events/min at peak; kitchen screens need sub-second updates; load is concentrated within a branch (low fan-out) but correlated across branches. Expect writes to be dominated by order lines and ticket status changes; they stay in the same architecture (modules + realtime gateway) but must be capacity-tested before launch.
- **Native POS:** adds payments and end-of-day volumes; may justify a separate payments service boundary later.
- **Offline sync:** adds bursty upload on reconnect; idempotent batch endpoints and per-device rate limits.
- **Marketplace:** large public read traffic; would use a separate read-optimised store and CDN; not in current scope.

---

## 8. Load, stress and soak testing plan **[R]**

| Test | Scenario | Pass criteria |
|---|---|---|
| Baseline | S1 dataset, evening profile | All targets in §3 met with ≥ 40% headroom |
| Growth | S2 dataset, 10× S1 rates | Targets met; DB CPU < 60% |
| Scale probe | S3 dataset, 10,000-branch rates | Identify limits; decision on cell split ceiling |
| Hot-spot | 500 holds on one branch-night in 60 s | See §2.2 |
| Burst | 5× special-date profile for 30 min | Error rate < 0.1%; no queue growth beyond SLO |
| Soak | 24 h at 1× evening cycle ×3 | No memory/connection leaks; stable latency |
| Failure | AZ failover, DB failover, provider outage, worker crash during job | Recovery within RTO; no lost/duplicated bookings or messages |
| Stress | Increase until failure | Graceful degradation (429/503 with Retry-After), no data corruption |
| Isolation under load | Noisy tenant at 10× | Other tenants within SLO |
| Messaging | 10× reminder batch | Backlog drains within SLO; provider limits respected |
| Migration under load | Expand migration with traffic | No errors, no long locks |

Test data is generated; production-shaped statistics only from approved sources; results stored with the release.

---

## 9. Decisions requiring approval

| # | Decision | Recommendation |
|---|---|---|
| 1 | Targets and SLOs in §3 | Approve as proposals; revise after Phase 0 measurements |
| 2 | RPO/RTO proposals, especially region-loss ≤ 24 h | Approve; check against the Sydney group's contract |
| 3 | Messaging policy: allowances + overage, SMS only where valuable | **Approve in principle — it reshapes pricing and billing scope (usage billing needed in R1)** |
| 4 | Plan structure in §6.5 | Approve as a hypothesis; price points after Phase 0 cost data and interviews |
| 5 | Cell size ceilings determined by tests, not guesses | Approve |
| 6 | Defer shared cache, search engine and analytics warehouse until evidence | Approve |
| 7 | Provide rough numbers for the Sydney group's single branch: bookings per week, share online/phone/walk-in, number of tables and seats, covers on the busiest night | **Needed from the owner** to replace the per-branch assumptions (a rough figure is enough) |

## 10. Risks
1. **Assumption error** (volumes 3× higher or lower) → measure early; design has headroom; all numbers re-derived after Phase 0.
2. **Messaging cost shock** → policy, allowances, usage billing, provider RFQ; verify unit prices.
3. **Correlated evening peaks** → capacity planned on peak, not average; autoscaling with pre-warm for known dates.
4. **Test realism** → data generators reflect two tenant shapes; periodic calibration with real traffic.
5. **Over-building for S3** → no multi-region, no sharding, no extra stores until triggers are hit.

## 11. Next
Document 10 (Roadmap and Implementation Plan): phased plan with scope, deliverables, dependencies, risks, acceptance criteria, testing, effort ranges and go-live prerequisites.

---

## 12. Update 2026-11-08 — real volume point and Twilio list prices

**Real data point (D-37):** the Sydney branch reports about **30 bookings/week ≈ 130/month**, i.e. roughly **one fifth of the 600/month assumption** in §2.1. Messages ≈ 130 × 3 = **390/month** for this branch. One branch is one data point; the design is unaffected, but cost per branch and test data should use a *range* (e.g. 130–600 bookings/month) rather than the single assumed value.

**Twilio list prices read from Twilio's public pages on 2026-11-08** (read via a summarising tool; **re-verify before relying; the pages show "$" without naming a currency**; carrier fees are extra and not shown):
| Item | Figure shown | Note |
|---|---|---|
| Australia, outbound SMS (mobile number or alphanumeric sender ID) | $0.0515 per message segment | Mobile-prefix number: $8.25 per month; alphanumeric sender ID: no number fee |
| India, outbound SMS | $0.0832 per message segment | DLT registration requirements not shown on that page; "contact sales" for domestic discounts |
| Failed message processing | $0.001 per failed message | |
| Alphanumeric sender ID | "can only be used to send SMS" per Twilio's page | **One-way:** guests cannot reply, so "reply C to cancel" needs a real number (extra monthly fee) |

**Implications**
- Message cost per branch per month (before carrier fees, assuming one segment per message): at 390 messages ≈ **$20**; at the earlier 1,800-message assumption ≈ **$93**; India at 390 messages ≈ $32. These sit inside the illustrative $0.02–$0.08 range used in §6.2, near its middle.
- **Segments:** a message longer than one segment (about 160 GSM characters; shorter for non-Latin scripts such as Japanese/Chinese templates) is billed as several segments; keep templates short and measure.
- **Two-way SMS** (reply to cancel) requires a number per sender (≈ $8.25/month per number on the page); decide per tenant or shared-number design (shared numbers reduce cost but complicate routing, replies must be mapped to the right tenant).
- Updated price floor illustration using D-26's P ≥ C ÷ 0.37 (other variable cost still an assumption of $10): at 390 messages C ≈ $20 + $10 = $30 → **P ≈ $81**; at 1,800 messages C ≈ $93 + $10 = $103 → **P ≈ $278** (currency unconfirmed; ignores fixed costs and the team).
- The unit-economics conclusion is unchanged in direction: price must be tied to message volume (allowances/overage) or channels other than SMS must carry reminders. At the Sydney branch's actual volume the SMS cost is modest.
