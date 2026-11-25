# Cheery: Technology Stack Verification

Prepared 2026-11-06. Status: research input for ADRs, not a decision record.

## How to read this document

Evidence tags used on every fact:

- **[P]** = page was fetched directly (WebFetch) on 2026-11-06 and the fact read from the vendor/official page. URL given.
- **[S]** = fact came from a web-search result summary only (the vendor page was not opened by me). Treat as "probably right, re-check before committing money". URL given is the page the summary cited.
- **UNVERIFIED** = could not be confirmed from an official source in this pass. Several items rely on engineering knowledge, flagged as such.

Retrieval date for every URL below: **2026-11-06**.

Caveat on scope: this pass covered ~30 searches and ~20 page fetches. Many vendor pricing pages could not be opened, so prices marked [S] must be re-checked on the vendor page before any contract. Third-party aggregator figures (cost-comparison sites) were deliberately not relied on where they conflicted with vendor pages.

---

## 0. Headline findings (challenges to the preliminary stack)

1. **Auth0 is the weakest-fit identity option on price.** B2B Essentials is $150/mo for 500 MAU, $2,100/mo at 10,000 MAU, with 3 SSO connections and $100/mo per extra connection [P] https://auth0.com/pricing. WorkOS AuthKit is free for the first 1M MAU [P] https://workos.com/pricing. Restaurants rarely need enterprise SSO, so per-MAU cost dominates.
2. **Stripe cannot be the only billing/payments answer for India.** Stripe is invite-only for Indian businesses [P] https://docs.stripe.com/india-exports, and Connect in India is "preview" only [P] https://docs.stripe.com/connect/explore-connect-guide. Razorpay Route has turnover eligibility thresholds that exclude many small restaurants [P] https://razorpay.com/docs/payments/route/.
3. **Exclusion constraints on partitioned tables only work if the partition key is in the constraint with equality.** Partitioning reservations by time (a range) will NOT allow the overlap constraint [P] https://www.postgresql.org/docs/current/ddl-partitioning.html. Partition by tenant (hash/list) or do not partition reservations.
4. **RDS Proxy pins connections on PostgreSQL SET statements** [S] https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy-pinning.html. If tenant context uses session SET, RDS Proxy gives little multiplexing. PgBouncer in transaction mode with `SET LOCAL` / `set_config(..., true)` is the safer pattern.
5. **Redis licence changed twice; Valkey (BSD-3) is the low-risk default on AWS** [S] https://www.infoq.com/news/2025/05/redis-agpl-license. Also question whether Redis is needed in phase 1 at all.
6. **Terraform is BUSL (IBM-owned); OpenTofu is the open alternative, CNCF Sandbox since 2025-04-23** [S] https://spacelift.io/blog/hashicorps-license-change. Lean OpenTofu.
7. **Spring Boot 4.0 loses free OSS support 2026-12-31** [S] https://www.herodevs.com/blog-posts/spring-boot-versions-eol-dates-and-latest-releases-april-2026, so a Kotlin/Spring path must start on 4.1 or later.
8. **Two compliance items are already live for messaging:** Australian SMS Sender ID Register in force since 2026-07-01 [S] https://www.acma.gov.au/sms-sender-id-register (via search), and India DLT (entity/header/template) for any SMS to Indian numbers [S] https://www.plivo.com/docs/sms/concepts/dlt-registration-process.
9. **Sydney-only hosting for India customers** is an unexamined assumption (latency, data-residency, DPDP Act). All UNVERIFIED here; needs an ADR.

---

## 1. Decision-by-decision table

### 1.1 Backend language/framework

| Option | Current version / support | Notes |
|---|---|---|
| Node.js (runtime for NestJS) | Node 24: Active LTS, EOL 2028-04-30; Node 22: EOL 2027-04-30; Node 20 EOL 2026-04-30; Node 26 becomes LTS Oct 2026, EOL 2029-04-30; project moving to one major per year from 2027 [S] https://endoflife.date/nodejs . Authoritative source is nodejs.org release schedule.json, not opened (UNVERIFIED primary). |
| NestJS | 12.0.0 is current major, 12.1.x point releases; requires Node 20.19+/22.12+; all core packages ESM; Standard Schema validation; Rspack default bundler [S] https://github.com/nestjs/nest/releases/tag/v12.0.0 . NestJS publishes no LTS/EOL table that I found (UNVERIFIED). |
| .NET | .NET 10 LTS, GA 2025-11-11, end of support 2028-11-14 [S] https://dotnet.microsoft.com/platform/support/policy/dotnet-core |
| Spring Boot / Kotlin | Spring Boot 4.0 (Spring Framework 7) GA 2025-11-20, OSS support ends 2026-12-31; 4.1 GA 2026-06-10, OSS support to 2027-07-31 (estimate); all 3.x out of OSS support [S] https://spring.io/blog/2025/11/20/spring-boot-4-0-0-available-now/ and herodevs link above. Kotlin support timeline UNVERIFIED. |

Gotchas: NestJS 12 ESM-only packages break Jest unless Node 24.9+ [S]; TypeScript raised to ^6 in generated projects [S]. Spring free-support windows are short (about 18 months), so you are on an upgrade treadmill. .NET LTS gives 3 years.

Verdict: **OPEN (leaning KEEP)**. NestJS 12 on Node 24 LTS is viable and gives one language across web and API, but the "enterprise with many modules" growth path is better served by .NET 10 LTS or Spring (stronger typing of module boundaries, longer tooling maturity) if the team has that skill. Decide by team skill and hiring market (ADR-01); do not choose on technical merit alone. If KEEP: enforce module boundaries with lint rules (e.g. dependency-cruiser/Nx boundaries), because Nest modules do not enforce them.

### 1.2 PostgreSQL

| Item | Fact |
|---|---|
| Supported majors | 18 (18.6, final release 2030-11-14), 17 (17.11, 2029-11-08), 16 (16.15, 2028-11-09), 15 (15.19, 2027-11-11), 14 (14.24, **2026-11-12**) [P] https://www.postgresql.org/support/versioning/ |
| PG 19 | RC1 released 2026-10-15, no final dates published [P] same URL |
| Policy | 5 years per major, minors at least quarterly [P] same URL |

Gotchas: Which major your managed service offers in ap-southeast-2 (RDS/Aurora) is UNVERIFIED; check before pinning. Choose 17 or 18, never 14/15 for a new build.

Verdict: **KEEP** (PostgreSQL 18, or 17 if RDS lacks 18). Best fit for RLS + exclusion constraints + outbox.

### 1.3 Redis

| Item | Fact |
|---|---|
| Redis 8 | Tri-licensed: RSALv2, SSPLv1, and AGPLv3 [S] https://www.phoronix.com/news/Redis-8.0-Goes-AGPLv3 |
| Valkey | BSD-3 fork of Redis 7.2.4, Linux Foundation; ElastiCache/MemoryDB support Valkey at lower price than Redis OSS (one source: about 20% node, about 33% serverless) [S] https://aws.amazon.com/elasticache/redis/ . Exact AU-region prices UNVERIFIED. |

Verdict: **CHANGE** to "Valkey on ElastiCache, only when needed". Reason: avoids AGPL/SSPL questions and is cheaper; also defer it until a concrete need (rate limiting, caching, websocket fan-out) since Postgres + app caches cover phase 1.

### 1.4 Postgres-backed job queue and transactional outbox

| Item | Fact |
|---|---|
| graphile-worker | MIT; requires PostgreSQL 12+ and Node 18+ per docs; latest version seen 0.16.6, possibly stale [S] https://worker.graphile.org/docs/requirements |
| pg-boss | MIT; current version and PG requirement not confirmed (a 9.0.3 snapshot only) UNVERIFIED. Check npm. |

Gotchas: both rely on SKIP LOCKED and (graphile-worker) LISTEN/NOTIFY. LISTEN/NOTIFY does not work through PgBouncer transaction pooling, so workers need a direct/session connection [S] https://www.netdata.cloud/guides/pgbouncer/pgbouncer-transaction-vs-session-pooling/. Job tables sit in the primary DB: bloat/vacuum pressure at high throughput. Outbox rows must be inserted in the same transaction as the business write; the relay then enqueues. Outbox relay must be idempotent (at-least-once).

Verdict: **KEEP**. Reason: removes a broker from the early footprint; plan an exit to SQS/Kafka behind an interface once throughput or fan-out warrants it. Pick graphile-worker vs pg-boss in an ADR after checking release cadence.

### 1.5 Frontend: React + TypeScript PWA

| Item | Fact |
|---|---|
| React | Sources conflict on newest 19.x (19.2.8 to 19.3.1); React publishes no EOL dates and backports security fixes across 19.x lines [S] https://endoflife.ai/article-react-eol.html . Confirm on react.dev (UNVERIFIED primary). |
| TypeScript | Nest 12 scaffolds TS ^6 [S]; TypeScript 7 native status UNVERIFIED. |

Gotchas: iPad/Safari PWA storage eviction and background-sync limits are known risks for offline POS (engineering knowledge, UNVERIFIED in this pass). See section 4.

Verdict: **KEEP**, with the offline question treated as a separate decision (section 4).

### 1.6 OpenAPI contract-first REST

Not researched against the OpenAPI Initiative site (current spec version UNVERIFIED). Engineering view: fine and low-risk.

Verdict: **KEEP**. Reason: generates typed clients for the PWA and future native clients. Add an API-diff check (breaking-change detection) in CI.

### 1.7 Managed identity

| Option | Pricing / terms | Notes |
|---|---|---|
| Auth0 | B2B Essentials from $150/mo (500 MAU), $300 at 1,000, $700 at 2,500, $1,300 at 5,000, $2,100 at 10,000, $3,800 at 20,000; 3 enterprise connections included, $100/mo each extra, max 30; Professional from $800/mo, 5 connections; yearly = 11x monthly; B2C capped at 10 organisations, B2B unlimited orgs [P] https://auth0.com/pricing |
| WorkOS | AuthKit first 1M MAU free, then $2,500/mo per extra 1M; SSO and Directory Sync $125/connection (1-15), $100 (16-30), $80 (31-50), $65 (51-100), custom above; custom domain $99/mo; audit-log SIEM streaming $125/mo [P] https://workos.com/pricing |
| Amazon Cognito | Lite / Essentials / Plus tiers; Essentials $0.015/MAU and Plus $0.02/MAU, 10,000 MAU free on Lite/Essentials, SAML/OIDC federated users only 50 free MAU; Lite volume rates disagree across sources [S] https://aws.amazon.com/cognito/pricing/ . Sydney-region price UNVERIFIED. |
| Entra External ID | MAU model with a free tier (50,000 MAU per search summary [S] https://azure.microsoft.com/en-au/pricing/details/microsoft-entra-external-id/); add-ons (M2M, SMS, Go-Local) priced separately; **Go-Local data residency add-on currently available only in Australia and Japan** [P] https://learn.microsoft.com/en-us/entra/external-id/external-identities-pricing . Paid per-MAU rate not shown (UNVERIFIED). |
| Keycloak | Latest 26.8.2 on download page [P] https://www.keycloak.org/downloads (no support policy or licence stated there). CNCF incubating since 2023 [S] (stage today UNVERIFIED). A third-party tracker says 26 is not an LTS; patches go to the current minor only [S] https://www.versio.io/en/product-release-end-of-life-eol-keycloak-keycloak.html . |

Gotchas and challenges:
- Multi-tenant fit: "organisation" per restaurant group vs. "tenant" model. Auth0 Organizations and WorkOS Organizations map well; Cognito needs a pool-per-tenant or custom-claims design (pool-per-tenant has quotas; UNVERIFIED limits).
- Staff PIN/tablet login (shared devices) is not a standard OIDC flow; budget for a custom device-token + PIN layer regardless of vendor.
- Keycloak ops burden (HA, upgrades each minor, DB, theming) is real; with no LTS you would be upgrading frequently. It removes per-MAU fees but adds a platform-team cost. UNVERIFIED on exact upgrade cadence.
- Vendor lock-in is mostly in user-store migration (password hashes export) and tenant/org model; keep app-side tenant/role model in your own DB and treat IdP as authentication only.

Verdict: **OPEN (CHANGE away from Auth0 as default)**. Reason: at restaurant-scale MAU counts Auth0 B2B costs several times WorkOS/Cognito, and Entra's AU data residency add-on is a real differentiator if AU residency is contractual. Short-list WorkOS AuthKit and Cognito; run a 2-week spike on tenant model and PIN login (ADR-03).

### 1.8 Billing (SaaS subscriptions)

| Option | Facts |
|---|---|
| Stripe Billing (AU) | Pay-as-you-go 0.7% of Billing volume (excludes one-off invoices); or annual-contract paid monthly: A$930/mo up to A$150k/mo volume, A$2,250 up to A$375k, A$4,400 up to A$750k, A$8,600 up to A$1.5M, 0.67% above cap. Card processing 1.7% + A$0.30 domestic, 3.5% + A$0.30 international [P] https://stripe.com/au/billing/pricing . Stripe fees attract GST in Australia (tax invoice issued monthly) [S] https://support.stripe.com/questions/goods-and-services-tax-gst-for-australia-based-businesses . Stripe Tax pricing not on that page (UNVERIFIED). |
| Stripe for India | Invite-only; Indian businesses cannot self-sign-up; accounts focus on **export** (international) payments, presentment 135+ currencies, payouts in INR, IEC may be required; international cards need 3DS [P] https://docs.stripe.com/india-exports . Domestic Indian recurring (UPI/e-mandate) support: UNVERIFIED. |
| Chargebee | Public page shows Flow and Enterprise growth plans and "$0 + 0.80%" / "$99 + 0.65%" rates without clear labels; Performance/RevRec by demo only; no AU/IN pricing shown [P] https://www.chargebee.com/pricing/ . Indian GST docs exist https://www.chargebee.com/docs/billing/1.0/taxes/indian-gst [S]. |
| Paddle | Merchant of Record, 5% + $0.50 per transaction [S] https://dodopayments.com/blogs/paddle-fees-explained (third-party; Paddle's own pricing page not opened, UNVERIFIED). AUD, GBP, INR are conversion currencies, and Australia and India are supported selling countries with tax-inclusive pricing [P] https://developer.paddle.com/concepts/sell/supported-countries-locales.md . Settlement currency/payout to AU entity UNVERIFIED. |

India recurring-payment constraint: the RBI Digital Payments E-Mandate Framework (2026-04-21) reportedly requires additional-factor authentication at registration and on the first transaction, no AFA for recurring debits up to INR 15,000, AFA above that; also covers cards, UPI and prepaid instruments [S] https://www.businesstoday.in/personal-finance/news/story/rbi-caps-recurring-payments-at-rs15000-without-otp-under-new-e-mandate-framework-526759-2026-04-21 . Primary RBI notification not opened (https://www.rbi.org.in/scripts/FS_Notification.aspx?Id=12341 is the 2022 version) so details are UNVERIFIED. Pre-debit notification rules also exist (engineering knowledge, UNVERIFIED here).

GST/tax invoicing: Australian GST invoice and Indian GST invoice (GSTIN, HSN/SAC, place of supply) are different documents. Neither Stripe Tax coverage of India nor Chargebee's AU GST handling was verified. Treat "tax-compliant invoice generation" as an in-house or Chargebee/MoR responsibility and decide in ADR-04.

Verdict: **OPEN**. Reason: Stripe Billing is the natural AU/UK/NZ choice, but India needs either an Indian-entity gateway (Razorpay Subscriptions/e-mandates, UNVERIFIED here) or a Merchant of Record (Paddle) which removes GST/VAT registration burden at 5% + $0.50. Design a billing port so providers can differ per region.

### 1.9 SMS / email / WhatsApp

| Channel | Facts |
|---|---|
| SMS, Australia | ACMA SMS Sender ID Register in force from 2026-07-01; only alphanumeric IDs need registration; unregistered IDs show as "Unverified" (one vendor says may be blocked); you register through a participating telco/provider, not ACMA directly [S] https://www.acma.gov.au/sms-sender-id-register (summary of ACMA user guide at acma.gov.au 2026-01 PDF). |
| SMS, India | DLT (TRAI) needs entity, header (sender ID, 6 chars) and content-template registration, plus binding your entity to your provider's telemarketer ID; entity approval about 1 to 3 business days; alpha sender IDs generally not supported for international senders; sources conflict on whether international routes bypass DLT [S] https://www.plivo.com/docs/sms/concepts/dlt-registration-process . AWS End User Messaging has an India sender-ID registration page [S] https://docs.aws.eu/sms-voice/latest/userguide/registrations-sms-senderid-india.html . |
| WhatsApp | Per-message (template) billing since 2025-07-01; marketing/utility/authentication/service categories; service messages free; India example rates (third-party) INR 0.86 marketing, 0.13 utility/auth; Australia rates not found [S] https://developers.facebook.com/documentation/business-messaging/whatsapp/pricing . Verify the current rate card for both countries. |
| Email | Amazon SES $0.10 per 1,000 emails, attachments $0.12/GB [S] https://aws.amazon.com/ses/pricing/ . Free tier terms conflicting; Sydney availability UNVERIFIED. |

Verdict: **OPEN**. Reason: use a messaging abstraction with one aggregator able to hold both AU Sender ID registration and India DLT (candidates seen: Plivo, Infobip, AWS End User Messaging, Twilio; none fully verified), SES for email, and a WhatsApp BSP; decide after a provider RFQ (ADR-06). Restaurant reminders are mostly utility category, which is cheap.

### 1.10 AWS compute: ECS Fargate vs EKS

| Item | Fact |
|---|---|
| EKS control plane | $0.10 per cluster-hour in standard support, $0.60 in extended support; standard support 14 months, extended 12 more [S] https://aws.amazon.com/eks/pricing/ . Roughly US$73 vs US$438 per month per cluster (third-party arithmetic). Worker/Fargate costs extra. |
| ECS Fargate | Region availability and Fargate Spot/ARM pricing in ap-southeast-2 not verified (UNVERIFIED). |

Verdict: **KEEP ECS Fargate**. Reason: lowest operational burden for a small team; EKS adds a Kubernetes upgrade treadmill (14-month support windows) with no current need. Revisit EKS only if you need operators, service mesh, or multi-cloud portability.

Challenge: all-Sydney hosting for Indian tenants is untested: latency (UNVERIFIED), Indian data-localisation expectations (UNVERIFIED), and RBI payment-data localisation applies to payment data held by payment systems (UNVERIFIED). Decide in ADR-07 whether India gets a Mumbai (ap-south-1) cell later; design tenant-to-region routing now.

### 1.11 IaC: Terraform/OpenTofu vs Pulumi

| Item | Fact |
|---|---|
| Terraform | 1.5.7 last MPL release; 1.6+ under BUSL 1.1; IBM completed acquisition of HashiCorp 2025-02-27 [S] https://spacelift.io/blog/hashicorps-license-change |
| OpenTofu | CNCF Sandbox since 2025-04-23; MPL-2.0 per sources; claim of 1.12 on 2026-05-14 is single-source UNVERIFIED |
| Pulumi | Pricing tiers inconsistent between aggregators; Pulumi's own page lists Free/Essentials/Pro/Enterprise [S] https://www.pulumi.com/pricing.md ; prices UNVERIFIED. |

Verdict: **CHANGE** to OpenTofu (HCL, open governance). Reason: BUSL has non-compete wording that is a vendor-risk if Cheery ever offers infrastructure-adjacent features; Pulumi (TypeScript) is a credible OPEN alternative if the team wants one language, at SaaS state-backend cost.

### 1.12 Observability: OpenTelemetry + Sentry

OTel JS SDK/signal stability and semantic-convention stability were not verified (search returned no answer). Sentry Team about $26/mo and Business about $80/mo (annual) with 50k errors / 5M spans per aggregators [S] https://sentry.io/pricing (not opened; UNVERIFIED). NestJS 12 ships an official "Observe" SDK hooking the request lifecycle [S].

Verdict: **KEEP**. Reason: vendor-neutral telemetry; ensure Sentry is not the only backend for traces (send OTLP to a collector). Add PII scrubbing (customer phone/email) in the collector.

### 1.13 Feature flags

| Item | Fact |
|---|---|
| OpenFeature | CNCF incubating (accepted 2023-12) [S] https://www.cncf.io/blog/2023/12/19/openfeature-becomes-a-cncf-incubating-project/ |
| Unleash | Open source Apache 2.0, no licence key; paid self-hosted Enterprise needs licence key [S] https://docs.getunleash.io/deploy/license-keys ; prices conflicting UNVERIFIED |
| Flagsmith | Core BSD-3-Clause (one outlier says Apache 2.0); hosted about $45 and $300/mo tiers per aggregators UNVERIFIED |
| LaunchDarkly | Plans Developer ($0)/Foundation/Enterprise/Guardian; price history contradictory in trackers; high cost at scale UNVERIFIED |

Verdict: **KEEP OpenFeature as the API; provider OPEN**. Reason: the SDK abstraction keeps you vendor-neutral; start with Flagsmith or Unleash OSS (or env-config flags) and avoid per-seat pricing until multi-team.

### 1.14 Edge protection: Cloudflare vs AWS WAF

| Item | Fact |
|---|---|
| Cloudflare | Plans Free $0, Pro $20/mo annual ($25 monthly), Business $200/mo annual ($250 monthly), Contract/Enterprise custom; WAF and Free Managed Ruleset on all plans [P] https://www.cloudflare.com/plans/network-cdn.md . Rate-limiting limits per plan and data localisation not on the page (UNVERIFIED). |
| AWS WAF | US pricing: $5 per web ACL/mo, $1 per rule/mo, $0.60 per million requests; Bot Control extra [S] https://aws.amazon.com/waf/pricing . Sydney price UNVERIFIED. |

Verdict: **OPEN**. Reason: Cloudflare is cheaper and simpler for DDoS/bot protection in front of an AWS origin; AWS WAF keeps everything in one IAM/billing/IaC plane and may matter for AU data-handling clauses. Note Cloudflare in front of AWS means TLS terminates at Cloudflare (data-path question for ADR-08).

---

## 2. Specific check: RLS and exclusion constraints with pooling, partitions, SET LOCAL

### 2.1 Verified facts

- PgBouncer transaction pooling: session-level `SET` (search_path, custom GUCs) does not persist across transactions; `SET LOCAL` is transaction-scoped and works [S] https://www.netdata.cloud/guides/pgbouncer/pgbouncer-transaction-vs-session-pooling/ .
- PgBouncer protocol-level prepared statements in transaction mode need `max_prepared_statements` (added in 1.21, default 200 since 1.24, per one source); SQL-level `PREPARE/EXECUTE` still breaks; temp tables, advisory locks, LISTEN/NOTIFY also don't work [S] https://www.netdata.cloud/guides/pgbouncer/pgbouncer-prepared-statement-does-not-exist/ . Verify against the PgBouncer changelog for the version you deploy.
- RDS Proxy: for PostgreSQL, using SET commands pins the client connection to a database connection, as do temporary tables, cursors, prepared statements and (per one blog) extended-protocol use; AWS suggests moving setup into the proxy's initialization query, and mentions "session pinning filters" whose availability for PostgreSQL is disputed [S] https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/rds-proxy-pinning.html . I found no PostgreSQL-specific statement about `SET LOCAL` under RDS Proxy; test it and watch the `DatabaseConnectionsCurrentlySessionPinned` metric (metric name from engineering knowledge, UNVERIFIED).
- RLS bypass: superusers and roles with BYPASSRLS always bypass; table owners bypass unless `ALTER TABLE ... FORCE ROW LEVEL SECURITY` [S] https://docs-new.arenadata.io/en/ADPG/current/how-to/security/row-security.html (vendor mirror of PG docs; confirm on https://www.postgresql.org/docs/current/ddl-rowsecurity.html, which I did not open).
- Partitioned tables: a unique/PK constraint must include all partition-key columns with no expressions; **an exclusion constraint on a partitioned table must include all partition-key columns and compare them with equality, not `&&`**; other columns may use any operator [P] https://www.postgresql.org/docs/current/ddl-partitioning.html . On PG 16.8 a bug report shows "exclusion constraints are not supported on partitioned tables" [S] https://www.postgresql.org/message-id/ME0P300MB0445ECCEBD36DD0843839E0FB6BC2%40ME0P300MB0445.AUSP300.PROD.OUTLOOK.COM , so support arrived in a later major (believed 17; UNVERIFIED, read PG17 release notes).

### 2.2 Recommended pattern (design guidance; validate in a spike)

- Tenant context: set per transaction with `SELECT set_config('app.tenant_id', $1, true)` (equivalent to SET LOCAL, parameterisable) as the first statement of every transaction. Never use session-level SET with pooling. Policies: `USING (tenant_id = current_setting('app.tenant_id')::uuid)`. Use `current_setting('app.tenant_id', true)` so that an unset value yields NULL (no rows) rather than an error.
- Roles: application connects as a non-owner, non-superuser, non-BYPASSRLS role; migrations run as a separate owner role; add `FORCE ROW LEVEL SECURITY` anyway. Background workers, admin consoles and reporting need their own explicitly-scoped roles.
- Reservations overlap: `EXCLUDE USING gist (tenant_id WITH =, table_id WITH =, tstzrange(starts_at, ends_at, '[)') WITH &&) WHERE (status <> 'cancelled')` requires the `btree_gist` extension for the equality operators on uuid/int columns. If you partition by `tenant_id` (hash or list), the partition key is in the constraint with `=`, which satisfies the documented rule [P]. If you partition by date range, it fails.
- Leakage and pitfalls to test (engineering knowledge, UNVERIFIED in this pass, each needs a test):
  - Views run with owner privileges by default, which bypasses RLS; use `security_invoker` views (PG15+).
  - Functions with `SECURITY DEFINER` bypass RLS for the function owner.
  - Constraint violations (unique/exclusion/FK) can reveal existence of rows the user cannot see; including `tenant_id` in every unique/exclusion key avoids cross-tenant signals.
  - `current_setting()` inside policies can defeat plan caching or index use if not wrapped as an initplan (`(SELECT current_setting(...))`); lead every index with `tenant_id`.
  - Leakproof functions: planner may refuse index/qual pushdown for non-leakproof operators before RLS filter; check EXPLAIN on hot queries.
  - Logical replication, pg_dump, FDW, and external sync engines (PowerSync/Electric) generally read as a privileged role and do not apply your RLS; tenant filtering must be re-implemented in their sync rules.
  - Connection reuse: a forgotten transaction without the `set_config` call must fail closed (NULL tenant means zero rows). Add a CI test that issues queries without context and asserts zero rows, and a pool-poisoning test (tenant A then tenant B on the same connection).
  - Exclusion constraints cannot be used as the arbiter in `INSERT ... ON CONFLICT` (engineering knowledge, UNVERIFIED); handle the violation (SQLSTATE 23P01) and map it to "slot taken".
- Pooling choice: PgBouncer (transaction mode) as a sidecar or small ECS service is more predictable than RDS Proxy for this pattern; RDS Proxy pins on SET and on prepared statements per AWS [S]. Alternatively no pooler at first with a bounded app pool, since Fargate task count is small.
- Many tenants: RLS with one shared schema scales to thousands of tenants; schema-per-tenant is a different pattern and was not researched.

Verdict: **KEEP** RLS + exclusion constraint, with the partitioning restriction recorded as a constraint on the data model (ADR-02).

---

## 3. Specific check: payment providers (AU and India)

| Capability | Australia | India |
|---|---|---|
| Hosted payment pages | Stripe Checkout / Payment Links (Stripe AU confirmed for pricing) [P] https://stripe.com/au/billing/pricing . Other AU PSPs (Tyro, Square, Adyen, Windcave, eWAY, Pin Payments, Airwallex): UNVERIFIED, not researched. | Razorpay Payment Pages / Payment Links exist and can be combined with Route [S] https://razorpay.com/docs/payments/route/ (features UNVERIFIED). Stripe: invite-only, export-focused [P]. |
| Deposits / no-show card holds | Stripe manual capture holds last about 7 days for customer-initiated card-not-present payments (Visa merchant-initiated 5 days, exact 4d18h); **not suitable for holds beyond a week**; extended authorisation exists for eligible cards [P] https://docs.stripe.com/payments/place-a-hold-on-a-payment-method . Practical pattern: save card (SetupIntent) at booking, charge off-session on no-show, or take a captured deposit and refund. | RBI e-mandate/tokenisation rules apply to saving cards and recurring/off-session charges [S]; whether no-show charges on stored cards are permitted without fresh AFA is UNVERIFIED. Treat as a legal/payments-partner question. |
| Marketplace/platform onboarding of merchants | Stripe Connect: AU is in the generally-available list; a platform can have connected accounts per its own country's rules [P] https://docs.stripe.com/connect/explore-connect-guide . Cross-border payouts only for platforms in US/UK/EEA/CA/CH, no `on_behalf_of` [P] https://docs.stripe.com/connect/cross-border-payouts . Adyen for Platforms: Australia listed; acquiring regions AU, CA, EEA, HK, NZ, SG, UK, US [S] https://docs.adyen.com/platforms . | Stripe Connect: IN is **preview** only [P] same Connect URL. Razorpay Route: India and Malaysia availability; eligibility requires FY25 or FY26 domestic turnover above INR 40 lakh (GST-3B) or export turnover above INR 5 lakh (FIRC), written use-case review, direct payer-payee relationship; fees not stated [P] https://razorpay.com/docs/payments/route/ . Adyen for Platforms: India not in official lists (one third-party says Adyen holds an RBI PA licence for local acquiring; UNVERIFIED) [S]. |

Implications:
- **Connect model decision:** Cheery is the platform; restaurants receive deposits. Decide whether deposits settle to the restaurant (Connect/Route, with KYC burden and platform licence/regulatory exposure) or to Cheery then paid out (money-transmission risk, UNVERIFIED legal position in AU/IN). Prefer restaurants connecting their own PSP account (Standard-style) for phase 1.
- A single Stripe Connect platform in Australia cannot onboard Indian restaurants self-serve (no cross-border for AU platforms, India preview). India requires a separate Indian-entity integration (Razorpay Route) with turnover eligibility that may exclude small restaurants; consider the restaurant bringing its own Razorpay/PhonePe/PayU keys instead (UNVERIFIED on those providers).
- Availability can change; re-verify with account managers before building.

Verdict: **OPEN**. Reason: payments need a provider port with per-country adapters (AU: Stripe Connect; IN: Razorpay or BYO-gateway), and a legal opinion on funds flow (ADR-05).

---

## 4. Specific check: native offline-capable tablet client

| Option | Licence / pricing / maturity (as verified) |
|---|---|
| PowerSync | Open Edition (self-host) under Functional Source License; client SDKs Apache-2.0; paid Cloud/Enterprise under commercial licence [S] https://powersync.com/legal/overview . Cloud: Free $0 (2 GB synced/mo, 50 concurrent); Pro from $49/mo (30 GB, 1,000 concurrent included); Team from $599/mo; Enterprise custom [P] https://powersync.com/pricing . Source DBs: Postgres, MongoDB, MySQL, SQL Server (Azure DocumentDB alpha). SDKs: web, Flutter, Kotlin, Swift, React Native, .NET, Node, Tauri, Capacitor, Rust [P] same URL. Open Edition has limited metrics and no dashboard [P]. |
| ElectricSQL | Docs moved to electric.ax; described as a **read-path** sync engine for Postgres using "Shapes" over HTTP; writes go through your own API; licence and GA status not stated on the page [P] https://electric.ax/docs/intro (UNVERIFIED licence/version; check GitHub). |
| RxDB | Core Apache-2.0; premium plugins Pro from $99/mo, Pro Plus $239/mo, Corporate $4,800/yr (annual billing); the page contradicts itself on whether OPFS/IndexedDB storages are free or premium [P] https://rxdb.info/premium/ . |
| Replicache / Zero | UNVERIFIED (not researched). |
| Native (SQLite + custom sync, e.g. Swift/Kotlin/Flutter) | Not researched; most control and best on iPad/Android background and storage guarantees, highest build cost. |

Engineering notes (UNVERIFIED in this pass): browser storage can be evicted by iOS Safari under pressure unless the PWA is installed/persistent-storage granted; OPFS and Background Sync support differs between Safari and Chrome; verify before committing to PWA-only offline for restaurant floor use.

Design stance: for restaurants the main offline cases are order entry and KDS. Conflicts are mostly last-write or append-only (order lines), so you rarely need CRDTs; server-authoritative rules for inventory/reservation capacity should reject at sync time and surface conflicts to staff. Keep the sync boundary (write API + change feed) as an interface.

Verdict: **OPEN**. Reason: PowerSync is the most capable fit with Postgres plus native SDKs for the later tablet client; Electric is read-only sync so writes remain your API (an advantage for RLS control but more work); RxDB adds commercial cost. Do a 2-week spike on PowerSync vs Electric+IndexedDB/OPFS against RLS tenancy (ADR-09). Not needed in phase 1 for server design beyond an outbox-style change log and idempotent write endpoints.

---

## 5. Stack risks (top 8)

1. **India payments gap (high).** Stripe invite-only and Connect preview in India; Razorpay Route turnover thresholds; RBI e-mandate rules. Mitigation: provider port, BYO-gateway for India, legal review before promising India billing/deposits.
2. **Tenant isolation failure through pooling or privileged paths (high).** Session GUC leakage, owner/BYPASSRLS roles, security-definer functions and sync engines bypassing RLS. Mitigation: `set_config(...,true)` per transaction, fail-closed tests, non-owner role, privileged-path inventory.
3. **Reservation integrity vs partitioning (medium-high).** Exclusion constraints require partition key equality; time-partitioning breaks them. Mitigation: do not partition by time on reservation tables, or partition by tenant hash; confirm PG version support.
4. **Identity cost and model lock-in (medium-high).** Auth0 B2B pricing climbs quickly with MAU and per extra SSO connection; Keycloak shifts cost to ops; PIN/tablet login is not covered by any vendor. Mitigation: IdP-neutral OIDC boundary, own tenant/role model, spike.
5. **Messaging compliance (medium-high).** AU Sender ID Register (live since 2026-07-01) and India DLT templates add onboarding lead time per tenant; WhatsApp template approval and per-message pricing changes. Mitigation: aggregator that covers both, per-tenant sender registration workflow, fallbacks to email.
6. **Single-region (Sydney) assumption (medium).** Latency, residency and regulatory expectations for Indian users unverified. Mitigation: region-aware tenant routing from day one; ADR on Mumbai cell.
7. **Licensing drift in infrastructure components (medium).** Redis tri-licence, Terraform BUSL, PowerSync Open Edition FSL, RxDB premium, Auth/flag vendor price changes. Mitigation: licence register in the repo; prefer Valkey, OpenTofu, OpenFeature.
8. **Framework and runtime lifecycle churn (medium).** Node majors every year (move to annual from 2027), NestJS 12 ESM migration and Jest constraint, Spring Boot 4.0 OSS end 2026-12-31, EKS 14-month support windows, PG 14 EOL 2026-11-12. Mitigation: pin to LTS (Node 24, PG 17/18), schedule upgrades, Renovate/Dependabot, avoid EKS until needed.

Runner-up risks: offline PWA reliability on iPad Safari (UNVERIFIED), tax invoicing for AU GST and India GST, and the modular monolith eroding without enforced module boundaries.

---

## 6. Decisions to take via ADR

| ADR | Decision | Inputs from this document |
|---|---|---|
| ADR-01 | Backend language/framework (NestJS 12 vs .NET 10 vs Kotlin/Spring Boot 4.1) | Team skill, hiring, module-boundary enforcement, LTS windows |
| ADR-02 | Tenancy and data model: RLS pattern, role model, pooler (PgBouncer vs none vs RDS Proxy), partitioning policy, exclusion constraints, PG major (17 vs 18) | Section 2 |
| ADR-03 | Identity provider and tenant/org model, staff PIN/device login | Section 1.7 |
| ADR-04 | Subscription billing and tax invoicing per region (Stripe Billing vs Chargebee vs Paddle MoR; India path) | Section 1.8 |
| ADR-05 | Restaurant payments: deposits, card-on-file no-shows, funds-flow model, Connect vs BYO-gateway per country | Section 3 |
| ADR-06 | Messaging providers and per-tenant sender registration (SMS AU/IN, WhatsApp, email) | Section 1.9 |
| ADR-07 | Hosting topology: ECS Fargate, region strategy (Sydney only vs Mumbai cell), data residency | Section 1.10 |
| ADR-08 | Edge/WAF (Cloudflare vs AWS WAF/CloudFront) and TLS termination | Section 1.14 |
| ADR-09 | Offline strategy and sync engine (PWA-only vs PowerSync/Electric vs native) | Section 4 |
| ADR-10 | Platform choices: OpenTofu vs Pulumi, Valkey adoption timing, queue library (pg-boss vs graphile-worker), feature-flag provider, observability backend | Sections 1.3, 1.4, 1.11, 1.12, 1.13 |

## 7. Items to re-verify before any commitment (UNVERIFIED list)

- Node.js official schedule; NestJS support policy; Kotlin/Spring compatibility.
- RDS/Aurora PostgreSQL 17/18 availability in ap-southeast-2; Fargate/ElastiCache/SES/WAF/Cognito Sydney pricing.
- PG release in which exclusion constraints on partitioned tables arrived; official RLS doc pages (leakproof, security_invoker); RDS Proxy behaviour with `SET LOCAL`.
- pg-boss and graphile-worker current versions and PG requirements.
- Stripe Tax coverage for AU/IN; Stripe handling of INR for non-Indian entity; RBI primary e-mandate text; Razorpay payment pages, subscriptions, international payments; Paddle own fees page and payout currencies; Chargebee actual prices.
- Australian PSPs (Tyro, Square, Windcave, eWAY, Pin Payments, Airwallex) and Indian PSPs other than Razorpay (PayU, Cashfree, PhonePe PG).
- ElectricSQL licence/GA status; Replicache/Zero status; Safari OPFS/storage-eviction behaviour.
- OpenTelemetry JS stability, Sentry plans, Unleash/Flagsmith/LaunchDarkly pricing, Cloudflare rate-limiting and data localisation, OpenAPI latest spec version.
