# ADR-007 — Hosting: AWS Sydney, ECS Fargate, OpenTofu
**Status:** Proposed (region approved, D-19) · **Date:** 2026-11-25
**Context:** Australian data residency for first customers; small team; need low operational burden.
**Options:** AWS ECS Fargate; AWS EKS; Azure Container Apps; other.
**Decision:** AWS ap-southeast-2 (Sydney), multi-account structure, ECS on Fargate (no Kubernetes initially), managed PostgreSQL (RDS or Aurora, decided after a sizing spike), infrastructure as code with OpenTofu.
**Consequences:** No cluster to operate; deployments by task definitions; cell model for later regions. Managed-PostgreSQL version and extension support (`btree_gist`) and pooler behaviour must be verified before commitment.
**Revisit when:** many services or special workloads justify Kubernetes, or customers require another cloud.
**Related:** Doc 7 §2, Doc 11.
