# RB-06 — Suspected security incident or data breach
> **This runbook supports, and does not replace, advice from a qualified legal/privacy adviser. Notification duties depend on country and contract [LAWYER].**
**Trigger:** alert (anomalous access, exports, signature failures, privileged use), report from a customer/researcher, leaked secret, or suspicious activity.
**Impact:** Sev-1 until scoped.
**Immediate actions (contain, then assess)**
1. Open a Sev-1 incident; assign an incident lead and a **security lead**; restrict the circle of people informed; keep a timestamped log.
2. **Preserve evidence:** snapshot logs, relevant database state and cloud audit trails before changing anything; do not delete or "clean up".
3. **Contain:** revoke or rotate affected credentials/tokens (RB-11); revoke sessions for affected users; disable affected integrations or endpoints with kill switches (RB-10); block malicious sources at the edge; isolate compromised workloads.
4. **Assess scope:** which tenants, which data classes (flag sensitive guest notes, payment references), which time window, which jurisdictions. Use audit events, access logs and database logs.
**Notifications**
- Start the legal clock from when a breach is suspected/confirmed per the applicable law (research notes: Australia — 30-day assessment duty under the Notifiable Data Breaches scheme; UK — 72 hours to the regulator; India — unverified).
- Notify affected customers (controllers) without undue delay, using the breach-notice contents in `docs/legal/02-data-processing-terms.md` Schedule 3; decide with the adviser who notifies regulators and individuals.
**Recovery:** eradicate the root cause, restore from clean sources if needed, verify with scans and tests, monitor closely for recurrence.
**After:** post-incident report to affected customers, control improvements, update threat model and risk register.
