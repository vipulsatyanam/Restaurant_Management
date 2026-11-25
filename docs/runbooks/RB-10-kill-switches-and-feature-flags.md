# RB-10 — Using kill switches and feature flags
**Purpose:** stop or limit a risky capability in seconds without a deployment (Doc 7 §6.4). Kill switches are not plans or entitlements.
**Who may use them:** on-call engineer or incident lead; every change is audited with a reason and ticket/incident id.
**Switch catalogue (to be implemented; names are placeholders)**
| Switch | Effect | Typical use |
|---|---|---|
| `messaging.send` (global / per tenant / per channel) | Stop outbound messages | Wrong or runaway messages; provider incident |
| `payments.deposit`, `payments.charge_fee` | Stop deposit capture / fee charging | Duplicate charges, card testing |
| `public.booking` (per venue / global) | Disable public booking with a neutral message | Severe abuse, tenant isolation incident |
| `res.write` | Read-only mode for reservations | Database recovery |
| `integrations.<provider>` | Disable an integration | Provider fault, credential leak |
| Feature flags per release | Disable a new feature | Bad release (RB-01) |
**How to use**
1. Decide the narrowest scope that stops the harm (tenant → venue → global).
2. Flip the switch in `<flag service/admin console>` with reason and incident id.
3. Verify the effect (logs/metrics/UI); confirm guests see neutral, accurate messaging.
4. Record start time; set a reminder to review within 1 hour.
5. Re-enable gradually after the fix; verify.
**Guardrails:** flags never gate security or tenant-isolation logic; do not use a kill switch to hide an incident from customers; stale flags are removed after rollout.
