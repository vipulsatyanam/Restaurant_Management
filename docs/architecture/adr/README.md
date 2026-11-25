# Architecture Decision Records (ADRs)

Format: Status · Context · Options · Decision · Consequences · Revisit triggers · Related.
Status values: **Accepted** (owner approved), **Proposed** (recommended, awaiting approval), **Open** (decision pending a spike or vendor check), **Deferred**, **Superseded**.
An accepted ADR is immutable except its status; changes go in a new ADR that supersedes it. Source of detail: `../02-solution-architecture.md` §9 and the numbered documents.

| ADR | Title | Status |
|---|---|---|
| 001 | TypeScript / NestJS backend | Accepted |
| 002 | Shared database + row-level security, cell-ready tenancy | Accepted |
| 003 | Managed identity provider | Open |
| 004 | Third-party billing service | Open |
| 005 | Guest payments via hosted pages, per-country adapters | Open |
| 006 | Messaging providers (Twilio intended) | Open |
| 007 | Hosting: AWS Sydney, ECS Fargate, OpenTofu | Proposed |
| 008 | Edge / WAF | Open |
| 009 | Offline operation | Deferred |
| 010 | Organization = tenant | Accepted |
| 011 | Modular monolith with enforced boundaries | Proposed (style approved in Stage C) |
| 012 | Transactional outbox + idempotent consumers | Proposed |
| 013 | Database-enforced overlap prevention | Proposed |
| 014 | Settings as scoped data with a resolver | Proposed |
| 015 | Public booking edge as a separate process type | Proposed |
| 016 | Monorepo, single codebase | Proposed |
