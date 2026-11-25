# Privacy, Consent and Policy Templates — DRAFT v0.1

> **DRAFT FOR LAWYER REVIEW. NOT LEGAL ADVICE.** Templates for Australia first. Wording about consent, sensitive information, fees and refunds must be confirmed by a qualified adviser **[LAWYER]** and adapted per country. The product must actually behave as the text says (retention, consent records, deletion); do not publish text describing features that are not built.

---

## A. Cheery Privacy Policy (Cheery as the responsible entity) — outline and draft text

**Who we are:** [Cheery legal entity] (ABN [ ]), contact [privacy@ address].
**Whose information this covers:** visitors to Cheery's website; restaurant customers' administrators and staff users; billing contacts; sales partners; people who contact us. **It does not cover guests' data entered by restaurants** — those restaurants are responsible for it (see their venue notice and our Data Processing Terms) **[LAWYER: confirm roles]**.

**Information we collect:** contact and account details; role and access logs; billing details held by our billing provider (we do not store card numbers); support communications; technical data (IP address, device, usage); partner identity and payout details (securely stored).
**Why we use it:** to provide and secure the service, bill customers, support users, pay sales partners, meet legal duties, improve the product (in aggregated or de-identified form).
**Who we share it with:** service providers (hosting, identity, billing, messaging, monitoring, support) listed in the sub-processor register; professional advisers; authorities where required by law. We do not sell personal information.
**Overseas disclosure:** some providers may process information outside Australia; the register lists locations **[LAWYER: required statements under the Privacy Act/APPs]**.
**Security and retention:** summary of measures; retention periods per Data Processing Terms and law **[to be filled]**.
**Your rights:** access, correction, and complaint; how to contact us; how to complain to the regulator (Office of the Australian Information Commissioner) **[LAWYER: confirm current requirements and any additional notices]**.
**Cookies and analytics:** [describe once defined; consent where required].
**Changes:** posted at [URL] with date.

---

## B. Venue privacy notice template (the restaurant is the responsible party)

*(Cheery provides this template; each restaurant must review and adopt it, and is responsible for it.)*

> **How [Venue/Brand name] uses your information**
> When you book with [Venue/Brand name] ([legal entity, ABN], [contact]), we collect your name, phone number and email address, your booking details (date, time, party size, seating requests), and anything you choose to tell us such as allergies or dietary needs, accessibility needs, whether you are bringing children, or the occasion. We use this to manage your booking, contact you about it (confirmation, reminders, changes, cancellations, waitlist updates), keep you safe and comfortable, handle deposits and cancellation fees where they apply, run our business, and meet legal requirements.
> **Allergy, dietary and accessibility information** is optional. We ask for it so our team can look after you. Only staff who need it can see it, and we keep it only as long as needed for that purpose **[LAWYER: confirm sensitive/health information treatment]**.
> **Payments:** if a deposit or card is needed, you enter card details on our payment provider's secure page. We do not see or store your full card number.
> **Who helps us:** Cheery (our booking system provider), messaging and payment providers. They process information for us and only on our instructions **[list per register]**. Some may handle data outside Australia **[LAWYER]**.
> **Marketing:** we only send news and offers if you tick the box on the booking form (separate from booking messages). You can unsubscribe at any time.
> **Your choices:** you can ask to see, correct or delete your information by contacting us at [contact]. How long we keep information: [period].
> Complaints: contact us first; you may also complain to [regulator, e.g. OAIC in Australia].

---

## C. Consent wording for the booking form (components)

| Component | Suggested text | Notes |
|---|---|---|
| Booking terms acknowledgement (required) | "I agree to [Venue]'s booking and cancellation policy and have read the privacy notice." (links) | Must link to the venue policy and the notice **[LAWYER]** |
| Allergy / dietary / accessibility (optional) | "Tell us about allergies, dietary needs or accessibility so we can look after you. Optional. Only our staff who need this will see it." | Tick/selection stored as a consent-linked note; not pre-ticked **[LAWYER: explicit consent for sensitive information]** |
| Transactional messages (service) | "We'll text/email you about this booking (confirmation, reminders, changes)." | Service messages; still need sender registration and opt-out handling per country |
| Marketing (optional) | "Send me news and offers from [Brand] by [email/SMS]." | **Unticked by default**, separate from booking, stored with time, text version and source (`consent_record`); one-click unsubscribe; honour opt-out within the legal time (Australia Spam Act: research notes 5 business days, **verify**) **[LAWYER]** |
| Cross-brand sharing (optional, organization setting) | "Allow [Group] to share my booking history across its restaurants to personalise service." | Off by default; only if the organization enables it and has notified guests **[LAWYER]** |
| Deposit / card on file | "[Venue] will hold your card details with its payment provider. You will not be charged now. If you cancel late or do not attend, [fee description] may be charged." | Must match the stored policy snapshot **[LAWYER]** |

Consent records must capture: guest, brand, purpose, channel, granted/withdrawn, source, wording version, timestamp (Doc 4 `guest.consent_record`).

---

## D. Booking, cancellation and deposit policy templates (for restaurants to adapt)

> **Booking policy — [Venue]**
> *Holding your table:* we hold your table for [15] minutes after your booking time. If we have not heard from you after that, we may release it.
> *Changes:* you may change your booking [online / by contacting us] up to [__] hours before your booking time, subject to availability.
> *Cancellation:* you may cancel free of charge up to [24] hours before your booking time. If you cancel after that time, or do not attend, we may charge [fee description, e.g. $[ ] per guest, up to $[ ] total] to the card you provided. The fee [reflects the cost to us of an empty table/ is our reasonable estimate of loss] **[LAWYER: whether and how fees are enforceable; wording must be accurate]**.
> *Deposits (groups of [6]+ / special events):* a deposit of $[ ] per guest is taken at booking to secure the table. It is [deducted from your bill / refunded if you cancel at least [ ] hours before / non-refundable after that time] **[LAWYER: consumer-law position on refunds and non-refundable deposits]**.
> *Large groups:* groups over [8] are handled by our events team; offers expire after [ ] days unless accepted.
> *Allergies:* tell us when booking; we take care but cannot guarantee an allergen-free environment **[venue to confirm; LAWYER]**.
> *Contact:* [phone/email].

**System configuration mapping:** each clause above corresponds to a setting (grace period, free-cancellation window, fee amount, deposit threshold/amount, refund rule, offer expiry); the booking stores a **policy snapshot** of the terms shown at booking time (Doc 4 `res.booking.policy_snapshot`), so later setting changes do not change what a guest agreed to.

---

## E. Other documents not yet drafted (need lawyer input first)
Website terms and cookie/analytics notice; vulnerability disclosure policy (`security.txt`); employee and contractor confidentiality/IP assignment; trade mark and company set-up (L8, L10, L12 in the plan).
