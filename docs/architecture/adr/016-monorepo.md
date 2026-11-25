# ADR-016 — Monorepo, single codebase
**Status:** Proposed · **Date:** 2026-11-25
**Context:** Shared types and contracts across API, web apps, widget and workers; atomic changes; one pipeline.
**Options:** monorepo with workspaces; polyrepo.
**Decision:** One repository with workspace tooling that supports dependency-boundary enforcement (pnpm workspaces with Nx or Turborepo, chosen at kickoff by a short spike).
**Consequences:** CI must use affected-only builds as it grows; code ownership via CODEOWNERS; discipline against cross-module imports.
**Revisit when:** build times become prohibitive or teams need independent release cycles.
**Related:** Doc 8 §2.
