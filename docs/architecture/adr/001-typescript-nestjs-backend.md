# ADR-001 — TypeScript / NestJS backend
**Status:** Accepted (owner, D-28) · **Date:** 2026-11-25
**Context:** Technology is not a constraint (D-05); the product will grow many modules and later an offline-capable client; development is AI-assisted, so strong static typing gives useful guardrails.
**Options:** (a) TypeScript on Node LTS with NestJS; (b) C# / .NET 10 LTS; (c) Kotlin/Java with Spring Boot; (d) Python/Django.
**Decision:** TypeScript on Node LTS with NestJS. One language across API, web apps, booking widget and a future offline client, with shared contracts. Concurrency-critical correctness lives in PostgreSQL either way.
**Consequences:** Exact Node/TypeScript/NestJS versions are pinned at kickoff (not fixed here). Needs discipline against untyped escapes (strict mode, no `any`, Doc 8). Runtime and ecosystem churn is a risk (dependency policy).
**Revisit when:** the maintaining team is much stronger in another stack, or a measured need (CPU-bound workloads) appears. Runner-up: .NET 10.
**Related:** Doc 2 §8, Doc 8.
