# ADR-006 — Messaging providers
**Status:** Open — Twilio intended (owner D-39, tentative) · **Date:** 2026-11-25
**Context:** SMS/email/WhatsApp confirmations and reminders. Australian sender-ID registration and India DLT registration are live constraints (research, partly unverified). Twilio list prices read on 2026-11-08: AU $0.0515/segment, India $0.0832/segment (currency not named on the page; carrier fees extra). Alphanumeric sender IDs are one-way, so reply-to-cancel needs a number.
**Options:** Twilio (SMS, and optionally email/WhatsApp via its products); other SMS aggregators; a separate email provider.
**Decision (pending confirmation):** Twilio as the intended SMS provider behind `MessagingPort`; confirm sender-ID steps, number strategy (shared vs per tenant), India DLT handling and the email provider in a short vendor check. Provider failover is a port capability, not a promise.
**Consequences:** Message metering and allowances are required (SUB-06); template length and encoding affect segment cost; Twilio must appear in the sub-processor register.
**Revisit when:** unit prices or registration rules change, or deliverability is poor.
**Related:** Doc 9 §6 and §12, Doc 11 E-B1.
