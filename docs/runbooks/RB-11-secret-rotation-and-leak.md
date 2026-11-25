# RB-11 — Secret or credential rotation and suspected leak
**Trigger:** scheduled rotation; secret found in code/logs/chat/repository; staff departure; provider breach notice; anomalous use of a credential.
**Classes:** database credentials; provider API keys (billing, messaging, payments, identity, cloud); signing keys (tokens, webhooks); per-tenant integration credentials (e.g. POS tokens); CI/CD credentials; platform admin accounts.
**If a leak is suspected (treat as Sev-1 until scoped)**
1. **Revoke or rotate immediately** at the provider (`<console>`); do not wait for investigation. Prefer rotation with overlap for signing keys so valid traffic is not broken.
2. Find all places the secret is used; deploy the new value via the secrets manager (no code change, no new static cloud keys).
3. Review audit logs for use of the old credential between exposure and revocation; assess impact (RB-06 if data may have been accessed).
4. Remove the secret from history where it was exposed (repository history rewrite only with the owner's approval; assume it is compromised regardless).
5. Scan for similar leaks (secret scanning in CI and repositories).
**Scheduled rotation**
- Database credentials: automatic where supported; quarterly check.
- Provider API keys: on a calendar (proposal every 6–12 months) and at staff changes; rehearse once before the pilot.
- Webhook signing secrets: rotate with overlap (accept old and new for a window).
- Per-tenant credentials: re-authorise on disconnect or expiry; revoke at the provider when the tenant disconnects.
**Verify:** all services healthy with the new secret; old secret rejected; rotation recorded with date and owner.
