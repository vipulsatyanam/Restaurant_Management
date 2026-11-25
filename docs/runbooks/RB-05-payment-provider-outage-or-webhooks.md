# RB-05 — Payment provider outage or webhook problems
**Trigger:** deposit/hosted-page errors; payment webhooks failing signature or not arriving; payment reconciliation discrepancies; provider status incident.
**Impact:** Sev-2 — bookings that require deposits/card-on-file cannot be confirmed; no-show fees cannot be charged.
**Confirm:** provider status; our webhook endpoint logs (signature failures, 5xx); `integ.inbound_event` backlog; reconciliation run results.
**Immediate actions**
1. **Do not confirm** bookings that require payment while the provider is down; show guests a clear message with alternatives (try later, call the venue). Bookings without payment continue.
2. If webhooks fail: check secret rotation, clock skew and raw-body verification; **never disable signature checks**.
3. If events are missing: run the provider-side event re-delivery and our reconciliation job to fetch the authoritative state.
4. If duplicate charges are suspected: stop automated fee charging (flag `<payments.charge_fee>`), identify by provider object ids, and refund through the provider with approval from the owner/finance.
**Recovery:** replay unprocessed events (idempotent), confirm holds that were paid, release holds that expired, notify affected guests.
**Communicate:** tell venues affected which bookings were held/unconfirmed; keep a list.
**After:** verify reconciliation is clean; review retry/timeout settings and runbook gaps.
