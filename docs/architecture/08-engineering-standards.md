# Document 8 — Engineering Standards

Baseline: Documents 1–7, `decision-log.md`.
Tags: **[C]** confirmed · **[R]** recommended, needs approval · **[A]** assumption · **[F]** future.
Status: **Draft.** These are the rules the team (and AI-assisted development sessions) must follow. "MUST" items are enforced by automation wherever possible; a rule that cannot be checked by a machine is reviewed by a human using the checklists in §12.

Versions: exact versions of Node.js (current LTS at kickoff), TypeScript, NestJS, PostgreSQL and libraries are **pinned at project kickoff** from official release pages, then reviewed monthly. This document deliberately does not hard-code version numbers that were not verified.

---

## 1. Engineering principles

1. **Domain first.** Code mirrors the restaurant domain (booking, allocation, service day, hold) in the language the business uses; names come from the glossary (§13.1).
2. **Make illegal states unrepresentable.** Use types, value objects and database constraints; validate at boundaries and trust inside.
3. **Pragmatism over ceremony.** Use DDD and Clean Architecture where they reduce risk (reservations, commission ledger, entitlements, configuration); use simple CRUD patterns for simple modules (templates, layout editor). No layers or abstractions without a present need.
4. **One way to do each thing.** A single approved pattern per concern (errors, validation, transactions, events, logging, config). New patterns need an ADR.
5. **Everything is tested at the lowest level that proves it,** and the critical invariants are tested at the database level too.
6. **Security and tenant isolation are everyone's job** and automated gates enforce them (Doc 6).
7. **Readability beats cleverness.** Code is written for the next maintainer (human or AI).

---

## 2. Repository and project structure **[R]**

One repository (ADR-16), workspace-based, with a build tool that understands dependencies and module boundaries (candidate tools: pnpm workspaces with Nx or Turborepo; chosen at kickoff by a short comparison of boundary-enforcement support).

```
/apps
  /api                 NestJS app: composition root, HTTP, auth middleware
  /public-edge         Public booking process (same modules, different composition)
  /worker              Jobs, outbox relay, schedulers
  /realtime            SSE / WebSocket gateway
  /web-staff           React PWA: host, manager, admin
  /web-platform        React: platform admin + partner portal
  /web-booking         Guest booking pages (server-rendered)
  /widget              Embeddable booking widget
/modules                           ← one folder per business module (Doc 2 §5.1)
  /reservations
    /domain  /application  /infrastructure  /api  /public  /tests
  /floor  /guest  /iam  /org  /config  /subscription  /commission  /notifications  /integrations  /audit  /reporting
/packages
  /kernel              ids, clock, money, errors, tenancy context, event base types, tracing (no business rules)
  /contracts           OpenAPI documents, generated clients, event schemas
  /ui                  Design system components (accessible by default)
  /testing             Fixtures, factories, DB test harness, provider fakes
  /config-schema       Typed configuration schema
/infra                 OpenTofu modules and environment stacks
/docs                  Architecture, ADRs, runbooks
/tools                 Generators, lint rules, codemods
```

Rules:
- **Apps compose; modules contain business logic.** An app wires modules and contains no business rules.
- `modules/*` may import only `packages/kernel` and **other modules' `public/`** (enforced).
- `packages/kernel` stays small; a PR that adds business meaning to it is rejected. There is **no `common`/`utils` dumping ground**; shared code must have a named home and an owner.
- Front-end apps import only generated clients and `packages/ui`, never server module code.
- Each module has a `README` stating its responsibility, owned tables, public interface, events in/out and how to test it.

---

## 3. Language and style standards

### 3.1 TypeScript **[R]**
- `strict` mode and the strictest practical compiler flags (no implicit any, exact optional properties, no unchecked indexed access). **No `any`** (use `unknown` and narrow); exceptions need a comment with reason and are counted in CI.
- No type assertions (`as`) to bypass checks except in tested boundary adapters.
- Prefer immutable data (`readonly`), discriminated unions for states, exhaustive `switch` with a `never` check.
- Money, ids, instants and local dates are **branded/value types** from the kernel; raw `number`/`string`/`Date` must not represent them in domain code.
- No default exports in modules; named exports only; no barrel files outside `public/`.
- Formatting by an automatic formatter; lint with a shared strict config. Style arguments are settled by tooling, not review comments.

### 3.2 Naming **[R]**
| Thing | Rule | Example |
|---|---|---|
| Files | `kebab-case` | `confirm-booking.handler.ts` |
| Types/classes | `PascalCase`, nouns | `Booking`, `AllocationPolicy` |
| Functions/methods | `camelCase`, verbs | `confirmBooking`, `findFreeResources` |
| Commands/queries | Verb + object | `ConfirmBooking`, `GetBookingsForServiceDay` |
| Events | `domain.past_tense` (type string), `PastTenseEvent` (class) | `booking.confirmed`, `BookingConfirmed` |
| Booleans | `is/has/can` prefix | `isActive`, `hasDeposit` |
| DB | `snake_case`, singular tables, `*_id`, `*_at` | `res.booking.starts_at` |
| Permissions | `area.action` | `booking.cancel` |
| Feature flags | `area.feature` with owner + expiry | `res.waitlist_v2` |
| Constants | `UPPER_SNAKE` only for true constants; settings live in config | — |

Use the domain glossary terms consistently (booking vs reservation, resource vs table, service day). Avoid abbreviations except those in the glossary.

### 3.3 Comments and documentation in code **[R]**
Code explains **what** through names and types; comments explain **why** (business rule, trade-off, legal reason, ticket id). Public interfaces (`public/`) have doc comments including invariants and error cases. Business rules reference requirement ids (e.g. `// RES-05`). TODOs must carry an owner and ticket and are counted in CI.

---

## 4. Architecture rules in code

### 4.1 Layering inside a module **[R]**
```
api  ──►  application  ──►  domain
              │                ▲
              ▼                │
        infrastructure ────────┘   (implements interfaces defined by domain/application)
```
- **Domain:** entities, value objects, domain services, domain events, repository *interfaces*. No framework, ORM, HTTP or database imports. Pure and unit-testable.
- **Application:** use cases (command/query handlers). Own the **transaction boundary**, authorisation call, idempotency handling and event publishing. Orchestrate; do not contain business rules that belong in the domain.
- **Infrastructure:** repositories, provider adapters, queue handlers. Implements interfaces.
- **API:** controllers/DTOs, mapping, input validation, OpenAPI annotations. Thin: parse → call use case → map result.

### 4.2 Business logic placement **[R]**
| Logic | Where |
|---|---|
| Invariants of one aggregate (a booking cannot be seated if cancelled) | Domain entity |
| Rules spanning aggregates (allocation choosing resources) | Domain service |
| Flow, authorisation, transactions, events | Application use case |
| Concurrency safety (no overlap) | **Database constraint** (plus domain check for friendly errors) |
| Formatting/mapping | API layer |
| Provider specifics | Infrastructure adapter |
Never in: controllers, SQL triggers for workflows, front end (the front end may duplicate for UX but the server decides), shared utility files.

### 4.3 Dependency injection **[R]**
Constructor injection through the framework's container at the composition root; depend on interfaces (ports) for anything external or non-deterministic (clock, id generator, random, payment, messaging, repository). No service locators, no global singletons holding state, no module-level `new` of infrastructure in domain/application code.

### 4.4 Patterns used (and when) **[R]**
Repository (aggregate persistence), Unit of Work (one transaction per use case), Strategy (country rules, pricing/commission basis, allocation scoring), Adapter (providers), State machine (booking, tenant, waitlist lifecycles with explicit transition tables), Outbox (events), Specification/Policy (cancellation and fee policies), Factory (aggregate creation with invariants), Command/Query separation. Avoid: inheritance hierarchies for business behaviour, generic repositories over everything, event sourcing (not needed), speculative plugin systems.

### 4.5 Cross-module communication **[R]**
1. **Query:** through the other module's `public/` interface (read-only, returns DTOs).
2. **Command in the same business transaction:** through `public/` interface, only when atomicity is required and documented.
3. **Everything else:** domain events via outbox; consumers idempotent.
Forbidden: importing another module's internals, reading its tables, sharing entities across modules, circular dependencies, calling a module through HTTP from inside the monolith.

### 4.6 Shared libraries and common services **[R]**
Add code to `kernel` only if it is (a) needed by ≥ 3 modules, (b) has no business meaning, (c) is stable. Otherwise duplicate small helpers until the abstraction is proven (Rule of Three). Cross-cutting services (auth, audit, entitlements, config resolver, clock, idempotency) live in their owning modules and are used via `public/`.

### 4.7 Duplication and abstraction **[R]**
Don't copy business rules: a rule has **one home** and is called from elsewhere. Don't abstract speculatively: prefer a little duplication to the wrong abstraction. A new abstraction needs two real uses or a stated seam (e.g., a provider port), and a reviewer's agreement.

---

## 5. Data access and transactions

- **SQL-first, typed query builder** (candidate libraries: Kysely or Drizzle; decided at kickoff by spike) — chosen because RLS, exclusion constraints, `set_config` and explicit SQL must remain visible. A heavyweight ORM that hides these is not used.
- **Repositories** return domain objects/DTOs, never rows or query-builder objects; one repository per aggregate; all queries parameterised; no string-built SQL from input (lint).
- **Tenant context:** the data-access layer opens a transaction and sets the tenant (`set_config(..., true)`) before any query; code cannot obtain a connection without a context (type-enforced). Platform queries use a separate, clearly named API.
- **Transactions:** exactly one per command (handled in the application layer via Unit of Work); no transactions held across network calls to providers; outbound calls happen **after commit** via the outbox/worker or use idempotency keys when unavoidable.
- **Locking and concurrency:** rely on constraints and optimistic version checks; explicit `SELECT … FOR UPDATE` only with a comment and a test; retry on serialization/deadlock errors with limits.
- **Pagination** is cursor-based; no unbounded queries; every list endpoint has a max page size.
- **Migrations:** SQL files owned by the module, expand→contract (Doc 4 §8), reviewed with the database-change checklist (§12.3).
- **N+1 prevention:** repositories expose purpose-built queries; a query-count assertion is part of integration tests for list endpoints.

---

## 6. API standards

Follow Document 5: contract-first OpenAPI, cursor pagination, problem-details errors with stable codes, idempotency keys, `If-Match` concurrency, DTO allow-lists, no tenant id for authorisation, versioned paths, deprecation policy. Additional rules:
- Controllers contain no business logic and no data access.
- Every endpoint declares its required permission, entitlement key and rate-limit class in the contract; a missing declaration fails the build.
- Every response/request DTO is a dedicated class/schema (no returning entities).

---

## 7. Errors, validation and logging

### 7.1 Errors **[R]**
- Two kinds: **domain errors** (expected, typed, carry a stable code and safe message, e.g. `SlotUnavailable`) and **unexpected errors** (bugs/infrastructure). Domain errors map to 4xx; unexpected map to 500 with a trace id and no details.
- Use a result/exception convention consistently (decision at kickoff: typed result values for domain outcomes, exceptions for truly exceptional cases). Never swallow errors; never log-and-rethrow at every layer (log once at the boundary).
- Error codes are part of the API contract and have tests.

### 7.2 Validation **[R]**
Validate shape and limits at the API boundary (generated from OpenAPI); validate business invariants in the domain; validate configuration at startup; validate provider payloads on receipt. Prefer parsing into typed values over checking and passing raw input.

### 7.3 Logging **[R]**
Structured JSON logs; levels (`error` = needs action, `warn` = unexpected but handled, `info` = business events/lifecycle, `debug` off in production); each entry carries request id, trace id, tenant id, actor id; **never** log secrets, tokens, full request bodies, card data, or personal/sensitive guest data (a redaction layer plus tests); no logging in tight loops. Business audit goes to the audit module, not the log.

---

## 8. Security and secure coding **[R]**

Mandatory practices (details in Document 6): deny-by-default authorisation on every endpoint; parameterised queries; output encoding; safe template rendering; no secrets in code or config files; verified webhook signatures; outbound URL validation; least-privilege roles; dependency allow-list and lockfile; no use of `eval`/dynamic code; safe file handling; cryptography only through the approved library wrappers (no custom crypto, no weak algorithms); randomness for tokens/references only from the cryptographic generator. The security checklist (§12.2) is part of review for any PR touching auth, tenant data, payments, files, integrations or exports.

---

## 9. Performance standards **[R]**

- Every new query on a request path has an `EXPLAIN` review for expected volumes and a supporting index (justified in the PR).
- No synchronous provider calls on latency-critical paths (availability search, hold creation) except where unavoidable and then with timeout and fallback.
- Cache only with a defined invalidation rule and a measurable benefit; never cache tenant data under a global key.
- Set explicit limits: payload size, page size, job duration, concurrency per tenant.
- Performance budgets from Document 9 are tested in CI on a representative dataset for the critical journeys; regressions beyond a threshold fail the pipeline.

---

## 10. Testing standards

### 10.1 Test layers **[R]**
| Layer | Scope | Tools (candidates) | Notes |
|---|---|---|---|
| Unit | Pure domain and application logic | Vitest or Jest | Fast, no I/O; table-driven for rules |
| Integration (database) | Repositories, constraints, RLS, migrations | Real PostgreSQL in containers | **No mocking the database** for anything involving constraints or isolation |
| Concurrency | Overlap guard, idempotency, hold expiry | Parallel clients against real DB | Release gate |
| API / contract | Endpoints vs OpenAPI, error codes, authorisation matrix | Contract test harness | Includes deny-path tests |
| Provider adapter | Adapters against fixtures and sandboxes | Recorded fixtures + sandbox runs | Fixtures refreshed regularly |
| Component (UI) | Front-end components and states | Testing Library | Accessibility assertions |
| End-to-end | Critical journeys through the browser | Playwright | Book, change, cancel, seat, waitlist, deposit; staff flows; mobile and tablet viewports |
| Accessibility | Automated checks + manual audits | axe-based checks | WCAG 2.2 AA target (Doc 1 NFR) |
| Performance & load | Journeys at stated volumes | k6 or similar | Document 9 |
| Security | SAST, SCA, DAST, secrets, IaC, pen test | Per Doc 6 | |
| Isolation suite | ISO-01…22 | Generated from catalogue | Release gate |
| Disaster recovery & restore | Backups, failover | Scheduled exercises | Doc 7 |
| Offline sync (later) | Local queue, conflicts, idempotency | Spike/test harness | When offline is built |

### 10.2 Rules **[R]**
- **Given/When/Then scenarios** derived from the requirement (not from the implementation) are written for every Must requirement in Document 1; each scenario maps to an automated test where practical.
- **Test data:** factories/builders; deterministic ids and an injected clock; no sleeps; no shared mutable state; each test isolates its tenant.
- **Time-based logic** is tested with fixed clocks across DST transitions, midnight service-day boundaries and each launch country's time zone.
- **Property-based tests** for allocation (never overlaps, never exceeds capacity, deterministic given inputs) and for the ledger (always balances, idempotent accruals).
- **Mutation testing** on the allocation engine, policy/fee rules, entitlement checks and commission ledger to prove the tests catch defects.
- **Bug fixes start with a failing test.**
- Flaky tests are quarantined within a day and fixed within a week; a quarantined test has an owner.

### 10.3 Coverage and acceptance thresholds **[R, proposals]**
| Area | Minimum |
|---|---|
| Domain and application code | ≥ 90% branch coverage |
| Whole backend | ≥ 80% line coverage |
| New/changed code in a PR | ≥ 85% line coverage (diff coverage) |
| Critical modules (reservations allocation, commission ledger, entitlements, authorisation) | 100% of documented rules covered by named tests + mutation score target ≥ 80% |
| Front end | Component tests for stateful components; E2E for all critical journeys |
Coverage is a floor, not the goal: **a PR with high coverage but missing the acceptance scenarios is not done.**

---

## 11. Pull requests, review and quality gates

### 11.1 Mandatory automated checks **[R]**
Build and type check; lint/format; unit, integration, concurrency, contract and E2E (relevant subset on PR, full on merge); architecture fitness checks (Doc 2 §5.4); migration lint; OpenAPI breaking-change check; SAST, dependency, secret, IaC and container scans; coverage thresholds; bundle size/performance budget checks for front end; stale-flag and TODO-owner checks; licence check for new dependencies.

### 11.2 Pull request rules **[R]**
- Small, single-purpose PRs (target < ~400 changed lines excluding generated files); description states **what, why, how tested, risk, rollback** and links requirement ids/ADR.
- At least **one human reviewer** (two for high-risk areas: auth, tenant isolation, payments, migrations, commission, infrastructure). AI-generated code is reviewed exactly like human code; the author remains accountable.
- CODEOWNERS per module; a change to a module's `public/` interface requires that module owner's approval.
- No merge with failing checks; no force-push to main; squash or rebase merge to keep history linear; commit messages follow a convention (type, scope, subject; ticket reference).

### 11.3 Code review standards **[R]**
Reviewers check, in order: correctness against requirement and acceptance scenarios; security and tenant isolation; data integrity and concurrency; failure modes and observability; API/contract compatibility and migrations; tests (do they prove the behaviour?); simplicity and naming; performance. Comments are classified (blocking / suggestion / question). Review turnaround target: within one working day.

### 11.4 Architecture fitness checks **[R]**
Automated tests fail the build when: a module imports another's internals; cycles appear; `domain/` imports infrastructure or frameworks; a tenant table lacks RLS/policy/tests; raw SQL is built from strings; a provider SDK is used outside an adapter; country/currency/tax constants appear outside country-pack data; `Date.now()`/`new Date()` is used in domain or application code; a destructive migration lacks an approved plan; an endpoint lacks permission/entitlement declaration; a public API change is breaking without a version bump. Exceptions require a recorded waiver with expiry.

---

## 12. Review checklists

### 12.1 General
- [ ] Meets the requirement ids cited; acceptance scenarios automated
- [ ] No business logic in controllers, SQL triggers or the front end
- [ ] Errors typed, mapped, and tested; no swallowed exceptions
- [ ] Logs free of secrets and personal/sensitive data
- [ ] Observability: metrics/traces added for new journeys; alert implications noted

### 12.2 Security
- [ ] Authorisation: permission, scope, entitlement, state checked; denial tested
- [ ] Tenant context used correctly; new tables have RLS, composite keys, tests
- [ ] Inputs validated; outputs allow-listed; no injection paths
- [ ] Secrets only via secrets manager; no new static credentials
- [ ] New dependency approved, pinned, licence-checked
- [ ] Webhooks verified and idempotent; outbound URLs validated

### 12.3 Database change
- [ ] Expand→contract plan; no long locks; indexes `CONCURRENTLY`; constraints `NOT VALID` then validated
- [ ] Backward compatible with previous release; rollback/forward-fix plan written
- [ ] Rehearsed on production-shaped data; destructive steps explicitly approved
- [ ] RLS/FK/index requirements met; data classification comments added
- [ ] Impact on retention, backups and the cell-move procedure considered

### 12.4 API change
- [ ] OpenAPI updated first; breaking-change check green
- [ ] Permission/entitlement/rate-limit declared; idempotency where state changes
- [ ] Error codes documented; examples added

---

## 13. Documentation requirements

### 13.1 Required artifacts **[R]**
| Artifact | Where | Owner | Updated when |
|---|---|---|---|
| Domain glossary | `docs/glossary.md` | Product + engineering | New domain term |
| Architecture documents (this set) | `docs/architecture` | Architect | Architecture change (via ADR) |
| ADRs | `docs/architecture/adr/NNN-title.md` (context, options, decision, consequences, status) | Author of the decision | Any significant decision, incl. exceptions to standards |
| Module README | module root | Module owner | Public interface or ownership change |
| OpenAPI documents | `packages/contracts` | Module owner | Any API change (first) |
| Event catalogue | generated from event schemas | Module owner | New/changed event |
| Runbooks | `docs/runbooks` | On-call / DevOps | New alert or procedure |
| Data dictionary | generated from schema comments (classification tags) | Module owner | Schema change |
| Release notes / changelog | generated from commits | Release manager | Each release |
| Onboarding guide | `docs/onboarding.md` | Engineering lead | Tooling change |

### 13.2 ADR process **[R]**
Propose in a PR → discussed → status `proposed/accepted/superseded`; an accepted ADR is immutable except status; consequences and revisit triggers are mandatory.

---

## 14. AI-assisted development rules **[R]**

(Complements Doc 6 §4.7.)
- Sessions work from these standards, the module README and the relevant requirement ids; the repository holds a concise project guide file for AI tools (architecture rules, commands, conventions, forbidden patterns) kept in sync with this document.
- Generated code must pass all gates and carry human review; the author explains it in the PR.
- Dependencies are never added by an AI session without explicit approval; package names are verified to exist and be reputable.
- Migrations, security-sensitive code and commission/payment logic get a second reviewer.
- No customer data, secrets or production dumps in prompts or fixtures.
- Prefer many small, verifiable steps; each step compiles and passes tests before the next.

---

## 15. Definition of Ready and Definition of Done

### 15.1 Definition of Ready (story may start) **[R]**
Requirement ids and acceptance scenarios (Given/When/Then) written; affected modules, tables, events, permissions, entitlements and settings identified; risks (security, privacy, concurrency, migration) noted; UX/design done where UI is involved; dependencies available; estimate agreed.

### 15.2 Definition of Done (story may be called done) **[R]**
- [ ] Acceptance scenarios implemented as automated tests and passing
- [ ] Code reviewed and merged through the gated pipeline; all automated checks green
- [ ] Tenant isolation and authorisation tests added or confirmed unaffected
- [ ] Migration safe, reversible/forward-fixable, rehearsed (if any)
- [ ] OpenAPI/events/docs/README/glossary updated
- [ ] Observability added (metrics, traces, logs without personal data); alerts/runbooks updated where relevant
- [ ] Security checklist applied; no new high/critical findings
- [ ] Accessibility checked for UI changes
- [ ] Feature flag created with owner/expiry (if rollout needed) and entitlement/permission configured
- [ ] Deployed to staging and verified; release notes entered
- [ ] Verified against the original requirement by someone other than the author

A story that is "code complete" but unverified is **not done**.

---

## 16. Decisions requiring approval

| # | Decision | Recommendation |
|---|---|---|
| 1 | Strict TypeScript, no `any`, branded value types | Approve |
| 2 | SQL-first typed query builder instead of a full ORM | Approve; library chosen at kickoff by a spike |
| 3 | Module layering and the rule set in §4 | Approve |
| 4 | Coverage thresholds and mutation testing on critical modules | Approve as proposals |
| 5 | Two-reviewer rule for high-risk areas | Approve |
| 6 | Build-failing architecture fitness checks | Approve |
| 7 | Exact monorepo and CI tooling | Decide at kickoff (spike) |
| 8 | AI-assisted development rules | Approve |

## 17. Risks
1. **Standards too heavy for a small team** → automate; start with a smaller enforced core and grow.
2. **Rules ignored under pressure** → gates in CI, waivers with expiry, no hard deadline (D-23).
3. **Query-builder friction vs ORM productivity** → spike before committing; keep generated types.
4. **Coverage gaming** → acceptance scenarios and mutation tests on critical code.

## 18. Next
Document 9 (Performance and Scalability): workload scenarios, targets, capacity and cost model, scaling strategy, SLOs.
