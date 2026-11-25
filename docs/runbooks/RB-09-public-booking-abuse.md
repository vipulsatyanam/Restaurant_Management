# RB-09 — Public booking abuse (hold storm, bots, card testing)
**Trigger:** hold-to-confirm ratio collapses; surge in holds or availability searches from few IPs/ASNs; deposit failures with many different cards; a popular venue shows no capacity while real demand is low; edge rate-limit alerts.
**Impact:** Sev-2 — real guests cannot book, or the payment provider may flag the account.
**Actions**
1. Identify patterns (IP ranges, user agents, contact details, card fingerprints, timing) in edge and API logs.
2. **Tighten limits:** lower per-IP/per-contact quotas; enable stricter bot challenge for the affected venue/endpoint (`<edge rule>`); reduce hold TTL for that venue; cap concurrent holds per venue (hold budget).
3. **Card testing:** pause the deposit flow for the venue or globally (flag `<payments.deposit>`), enable provider fraud tools, and contact the provider if cards were tested.
4. **Block** abusive sources at the edge with expiry; avoid blocking shared networks broadly.
5. Release abusive holds (bulk expire by pattern) after capturing evidence.
**Verify:** hold-to-confirm ratio and booking success recover; real guests can book; payment failures return to normal.
**Communicate:** tell the venue if capacity was blocked and for how long.
**After:** add detection rules; review bot-protection settings; consider phone verification on risk signals (Doc 5 decision).
