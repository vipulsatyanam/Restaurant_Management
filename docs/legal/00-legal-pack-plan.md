# Cheery Legal Pack — Plan and Status

> **DRAFTS ONLY. NOT LEGAL ADVICE.** These documents are working drafts prepared to give a qualified lawyer a head start. They have **not** been reviewed by a lawyer, and nothing here makes Cheery legally compliant or guarantees that no legal issue will arise. Law differs by country and changes; several points below are explicitly marked **[LAWYER]** for professional confirmation. Do not publish, sign or send any of these documents to a customer until a qualified adviser has reviewed them.

Prepared 2026-11-08 in response to decision D-36. Placeholders in `[square brackets]` must be completed. Related: `docs/architecture/06-security-and-compliance.md`, `11-architecture-review.md` (E-L1, E-L6).

## 1. Documents needed and when

| # | Document | Who it binds | Needed before | Draft | Needs lawyer? |
|---|---|---|---|---|---|
| L1 | Customer Terms of Service (subscription agreement) incl. acceptable use, support and availability schedule | Cheery ↔ restaurant organization | Any customer uses the product (pilot) | `01-customer-terms.md` | **Yes** (consumer/small-business law, liability, GST) |
| L2 | Data Processing Terms (DPA) with sub-processor list and security/breach schedule | Cheery ↔ restaurant | Before processing guest data | `02-data-processing-terms.md` | **Yes** |
| L3 | Cheery Privacy Policy (for Cheery's own customers, staff, partners, website visitors) | Public notice | Before pilot | `03-privacy-and-consent-templates.md` §A | **Yes** |
| L4 | Venue privacy notice and consent wording **template** (each restaurant is the controller of its guests' data; Cheery supplies a template) | Restaurant → guests | Before pilot booking pages go live | same file §B–§C | **Yes** |
| L5 | Booking, cancellation and deposit policy wording **templates** | Restaurant → guests | Before deposits/fees enabled | same file §D | **Yes** (consumer law, fee enforceability) |
| L6 | Sales Partner (independent contractor) agreement outline | Cheery ↔ salesperson | Before any commission is promised or paid | `04-sales-partner-agreement-outline.md` | **Yes** (contractor vs employee, tax) |
| L7 | Sub-processor register | Public/customer-facing | With L2 | inside L2 | Yes (vendor locations) |
| L8 | Website terms, cookie/analytics notice, security.txt/vulnerability disclosure policy | Public | Before public website/booking pages | not drafted yet | Yes |
| L9 | Pilot agreement / order form (short) | Cheery ↔ Sydney group (and other pilot branches) | Before pilot | outline in L1 §Order Form | Yes |
| L10 | Employee/contractor confidentiality and IP assignment (engineering team of 10) | Cheery ↔ staff | Before development begins | not drafted | Yes |
| L11 | Third-party vendor agreements review (billing, messaging, identity, cloud) | Cheery ↔ vendors | Vendor selection | checklist in L2 §Sub-processors | Yes |
| L12 | Trade mark/domain/company set-up (entity, ABN/GST registration, brand clearance) | Cheery | Before first invoice | not drafted | Yes |

## 2. What cannot be prepared without decisions from you
1. **Contracting entity** name, address, registration numbers (ABN/ACN or equivalents), governing-law state and courts.
2. **Commercial terms**: plan names, prices, trial length, billing cycle, notice periods, message allowances (Doc 9).
3. **Liability position**: caps and exclusions — a risk decision with insurance implications.
4. **Insurance**: professional indemnity/cyber cover is normally expected by larger customers.
5. **Pilot terms**: pilot length, fees (free or discounted), data handling during pilot, success criteria.
6. **Data location** promise (Australia-only hosting for AU customers is the design; confirm what you will commit to).
7. **Support commitment** (business hours; no 24×7) — drafted as such (D-31).

## 3. Honest limits of this pack
- A lawyer is still required: **[LAWYER]** items cover Australian Consumer Law and unfair-contract-term rules for standard-form small-business contracts, Privacy Act obligations and roles (controller/processor), consent and sensitive/health information rules, SMS/email consent, GST on subscriptions, contractor classification, limitation of liability enforceability, and India/UK requirements before those launches.
- Research in `docs/architecture/research/compliance-scan.md` has many **UNVERIFIED** regulatory items; the lawyer should confirm current law from primary sources.
- Extra pilot branches (D-35) mean real guests' data: the venue notice (L4), DPA (L2) and consent wording must be in place **before** guests use those booking pages.

## 4. Suggested sequence
1. Owner decides entity, commercial placeholders and liability stance (§2).
2. Engage a qualified lawyer with this pack (L1–L6 first); ask for a fixed-fee review.
3. Lawyer returns reviewed versions; Cheery configures the product (consent capture, policy snapshot, retention) to match.
4. Publish L3, L4/L5 templates and sign L1, L2, L9 with pilot customers.
5. Revisit for India/UK before those launches.

## 5. Status
| Item | Status |
|---|---|
| Plan | Done |
| L1 Customer terms | Draft written |
| L2 DPA + sub-processors + breach schedule | Draft written |
| L3–L5 privacy/consent/policy templates | Draft written |
| L6 Partner agreement outline | Draft written |
| L8, L10, L12 | Not drafted (need lawyer input first) |
| Lawyer review | **Not started** (no budget decision, D-34) |

## 6. Update 2026-11-08
No product name and **no legal entity** exist yet (D-38). Consequences: the pack remains in placeholders; contracts cannot be signed and many vendor accounts cannot be opened until an entity exists. Recommended order: (1) pick the name and check availability (trade mark, domain, social), (2) incorporate/register the entity and ABN/GST, (3) then insert details into the drafts and send to the lawyer. Twilio is the intended messaging provider (D-39) and must be added to the sub-processor register (Schedule 2 of the data-processing terms) with its processing locations once confirmed.
