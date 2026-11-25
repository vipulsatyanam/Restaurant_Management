# ADR-010 — Organization = tenant
**Status:** Accepted (owner, D-27) · **Date:** 2026-11-25
**Context:** The brief proposed Platform → Tenant → Group → Brand → Branch.
**Options:** keep separate tenant and group levels; merge them.
**Decision:** The organization is the tenant (one contract, one subscription, one data wall); Brand and Branch sit below; roles scope to organization, brand or branch. No separate "group" level.
**Consequences:** A customer with several legal entities needs several billing accounts under one organization, or separate organizations (decide per case). A single restaurant is just one brand with one branch.
**Revisit when:** a customer requires multiple legal entities with separate data walls and separate billing within one hierarchy.
**Related:** Doc 3 §1.
