# Cheery Data Processing Terms — DRAFT v0.1

> **DRAFT FOR LAWYER REVIEW. NOT LEGAL ADVICE.** Written for Australia first. Whether Cheery is a "processor", "service provider" or controller, and the notification duties that follow, differ by country and **must be confirmed by a qualified adviser [LAWYER]**. Items tagged **[LAWYER]** are open. Do not publish or sign before review.

These terms form part of the Customer Terms of Service. "Customer" is the restaurant organization; "Guest Data" is personal information about Customer's diners; "Staff Data" is personal information about Customer's users.

## 1. Roles and instructions
1.1 For Guest Data and Staff Data entered by Customer, Cheery processes the data **on Customer's documented instructions** (these Terms, the Order Form, and Customer's configuration of the Services) **[LAWYER: confirm role per country; Cheery is likely a processor/service provider for Guest Data, and a controller for its own account, billing and security records]**.
1.2 Customer is responsible for having a lawful basis, giving notices and obtaining consents (including marketing consent and any consent needed for sensitive information such as allergy notes) for the Guest Data it collects.
1.3 Cheery will tell Customer if it believes an instruction would breach law.

## 2. Details of processing
| Item | Description |
|---|---|
| Subject matter | Reservation, waitlist, guest management and notifications for Customer's venues |
| Duration | Term plus export/deletion period |
| Categories of individuals | Customer's guests; Customer's staff users |
| Categories of data | Guest: name, phone, email, booking history, preferences, allergy/dietary and accessibility notes, children flags, consent records, messages sent. Staff: name, email, role, access logs. Payment references (no card numbers) |
| Sensitive data | Allergy/dietary notes and accessibility needs may be sensitive/health-related **[LAWYER]**; restricted in the Services and excluded from logs and general events by design |
| Purpose | Providing, securing, supporting and improving the Services |

## 3. Customer data protection obligations of Cheery
Cheery will: process only as instructed; keep personnel under confidentiality; implement the security measures in Schedule 1; assist Customer, at reasonable cost, with access/correction/deletion requests and impact assessments; make available information reasonably needed to show compliance; not sell Guest Data; not use Guest Data for advertising or marketing to guests on its own account; not combine Guest Data across customers; not re-identify de-identified data.

## 4. Sub-processors
4.1 Customer authorizes the sub-processors in Schedule 2. Cheery will notify Customer of changes at least [30] days in advance (email/status page) so Customer can object on reasonable grounds; if unresolved, Customer may terminate the affected Services.
4.2 Cheery imposes data protection terms on sub-processors no less protective than these and remains responsible for their performance **[LAWYER]**.

## 5. Location and transfers
Primary storage for Australian customers is in Australia (AWS Sydney) **[confirm]**. Some sub-processors (identity, messaging, billing, monitoring, support) may process data outside Australia; Schedule 2 lists their locations and safeguards **[LAWYER: cross-border disclosure obligations, e.g. under the Australian Privacy Principles; India/UK rules before those launches]**.

## 6. Security
Cheery maintains the measures in Schedule 1 and reviews them regularly. Customer is responsible for its own account security (strong credentials, MFA, role assignments, device security).

## 7. Personal data breaches
7.1 Cheery will notify Customer **without undue delay and in any event within [72] hours** of confirming a breach affecting Customer's data **[commercial/LAWYER decision; Doc 6 proposes initial notice within 24–72 h of confirmation]**, with the information in Schedule 3 as it becomes available.
7.2 Customer (as the party with the relationship with guests) decides on and carries out notifications to affected individuals and regulators **unless the parties agree otherwise in writing**; Cheery will cooperate and will notify regulators where it is itself the responsible entity **[LAWYER: Australian Notifiable Data Breaches scheme obligations where both parties hold the same records; clarify who notifies]**.
7.3 Cheery keeps an incident record and provides a post-incident summary.

## 8. Data subject and regulator requests
Cheery will promptly forward requests it receives about Guest Data to Customer and will not respond except as instructed or required by law. The Services include tools for export, correction and deletion that Customer may use.

## 9. Audits
Cheery provides on request a summary of its security practices and, once available, relevant third-party assessment summaries (e.g. penetration test executive summaries). Customer-conducted audits are limited to once per year on reasonable notice, subject to confidentiality, and at Customer's cost **[LAWYER; Cheery is not certified to ISO 27001 or SOC 2 at launch — do not state or imply it is]**.

## 10. Return and deletion
On termination, Cheery makes data available for export for [30] days, then deletes Customer's data from live systems and from backups as they age out within [ ] days, except records legally required to be kept, which remain protected and are used only for that purpose **[LAWYER: retention]**.

## 11. Liability
Subject to the Customer Terms liability clauses; consider a separate cap for data protection claims **[commercial/LAWYER]**.

---

## Schedule 1 — Security measures (summary, aligned with the architecture)
- **Access:** managed identity provider; multi-factor authentication for administrators and platform staff; role- and scope-based permissions enforced on the server; least privilege; separate platform and customer realms.
- **Tenant isolation:** database row-level security and tenant-scoped data access; automated isolation tests as a release gate.
- **Encryption:** TLS in transit; encryption at rest for databases, backups and storage; secrets in a secrets manager.
- **Payments:** card details are entered only on payment-provider-hosted pages; Cheery does not store card numbers.
- **Application security:** input validation, protection against common web and API attacks, rate limiting and bot protection on public pages, dependency/code/secret scanning, human code review.
- **Infrastructure:** private networks, managed services, infrastructure as code, no standing production access for staff, audited just-in-time access.
- **Logging and audit:** immutable audit records of administrative and sensitive actions; logs exclude sensitive guest data by default.
- **Resilience:** automated backups with point-in-time recovery; restore testing; documented incident response.
- **Testing:** independent penetration test **[state only once done]**; vulnerability management.
- **Support access:** time-boxed, reasoned, and recorded; customer-visible log.
- **People:** confidentiality obligations; security awareness.
> Do not publish statements here that are not true on the date of signing. Items marked "state only once done" must be removed or qualified until completed.

## Schedule 2 — Sub-processor register (TEMPLATE — fill with actual vendors once chosen)
| Vendor | Purpose | Data categories | Processing location(s) | Safeguards / agreement | Status |
|---|---|---|---|---|---|
| Amazon Web Services | Hosting, storage, backups | All Customer Data | Australia (Sydney) [+ backup location, TBC] | AWS data processing terms | Planned |
| [Identity provider — WorkOS or Cognito, TBD] | Staff authentication | Staff names, emails, login events | [TBC] | [TBC] | Decision pending (ADR-03) |
| [Billing service — TBD] | Subscription billing | Customer billing contacts | [TBC] | [TBC] | Decision pending (ADR-04) |
| [Payment provider — TBD] | Hosted deposit/fee payments (Customer's merchant account) | Guest name/contact for payment, payment references | [TBC] | [TBC] | Decision pending (ADR-05) |
| [SMS provider — TBD] | Text messages | Guest phone numbers, message text | [TBC] | [TBC] | RFQ pending (ADR-06) |
| [Email provider — TBD] | Email messages | Guest email, message text | [TBC] | [TBC] | RFQ pending |
| [Monitoring/error tracking — TBD] | Observability | Technical data (no guest data intended) | [TBC] | [TBC] | Decision pending |
| [Support tooling — TBD] | Customer support | Staff contact details, ticket content | [TBC] | [TBC] | Not selected |
| Square (customer-authorised integration) | POS read-only sync | Sales summaries linked to guests | Per Square | Customer's own agreement with Square | Planned |
**Vendor due diligence checklist (L11):** data processing terms available; location of processing; sub-sub-processors; breach notification duty to Cheery; security attestations; deletion on exit; no training or secondary use of data; pricing/volume limits; regional availability (e.g. Australian sender-ID, India DLT).

## Schedule 3 — Breach notice contents
Date/time of discovery; what happened; categories and approximate number of individuals and records affected; data types involved (flag any sensitive data); likely consequences; containment steps taken; contact for follow-up; recommended steps for Customer; planned updates. Cheery provides updates as facts develop and a post-incident summary within [ ] days.
