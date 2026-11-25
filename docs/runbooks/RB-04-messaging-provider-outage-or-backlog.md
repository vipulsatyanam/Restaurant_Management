# RB-04 — Messaging provider outage or backlog
**Trigger:** message failure rate or queue age alert; provider status page incident; customers report missing confirmations or reminders.
**Impact:** Sev-2 — bookings stay valid but guests are not notified; reminders may miss their window.
**Confirm:** `<messages dashboard>` (sent/delivered/failed, backlog age), provider status, error codes (auth, rate limit, sender registration, carrier blocks).
**Immediate actions**
1. Check whether the cause is ours (credentials, rate limit, template, sender registration) or the provider's.
2. If the provider is down: let the queue retry with backoff; do **not** bulk-resend manually. If a secondary provider/channel is configured, switch via flag (`<flag>`); otherwise fall back to email for confirmations where permitted.
3. If rate-limited: reduce send rate and stagger by tenant fairness caps.
4. Show delivery status in the host UI so staff can phone guests for urgent bookings.
**Recovery:** when the provider recovers, the backlog drains; skip stale reminders whose time has passed (policy: do not send a "2 hours before" reminder after the booking started); run the messaging reconciliation job.
**Communicate:** status page note if delays exceed 30 minutes; tell the Sydney branch directly during the pilot.
**After:** review sender registration, templates, and provider SLAs; consider a second provider if outages recur.
