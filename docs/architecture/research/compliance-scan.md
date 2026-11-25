# Cheery - Compliance Scan (research input to architecture blueprint)

Date of scan: 2026-11-06. Not legal advice. Each claim cites a primary source; "UNVERIFIED" means the primary text could not be read in this scan (several regulator pages returned 403/404 or only search extracts were available).

## 1. Australia

**Privacy Act 1988 / APPs - latest amendments (Privacy and Other Legislation Amendment Act 2024; OAIC release dated 29 Nov 2024)**
- Package: statutory tort, expanded OAIC powers/penalty tiers/infringement notices, Children's Online Privacy Code, power to prescribe "adequate" countries/binding schemes for cross-border transfer, ADM transparency in privacy policies. https://www.oaic.gov.au/news/media-centre/pasing-of-bill-a-significant-step-for-australias-privacy-law
- Statutory tort (Schedule 2): OAIC states it commenced 10 June 2025. https://www.oaic.gov.au/privacy/your-privacy-rights/more-privacy-rights/statutory-tort-for-serious-invasions-of-privacy (commencement provision itself not read - partly UNVERIFIED).
- ADM transparency (APP 1.7-1.9): applies from 10 December 2026, to decisions made from that date, where a computer program makes or substantially and directly informs a decision that could significantly affect rights/interests using personal information; privacy policy must describe kinds of personal information and decisions. OAIC APP Guidelines ch.1 (v1.2, Oct 2025) and fact sheets (Sept 2026): https://www.oaic.gov.au/privacy/australian-privacy-principles/australian-privacy-principles-guidelines/chapter-1-app-1-open-and-transparent-management-of-personal-information
- Children's Online Privacy Code: NOT yet in force. Exposure draft consultation 31 Mar - 5 Jun 2026; must be registered by 10 Dec 2026. Applies to APP entities providing social media / relevant electronic / designated internet services (Online Safety Act 2021) likely to be accessed by children, excluding health services (page published 19 Mar 2026, updated 5 Aug 2026). https://www.oaic.gov.au/privacy/privacy-registers/privacy-codes/childrens-online-privacy-code . Whether a booking app is caught: UNVERIFIED.
- A "Privacy Amendment (Personal Data Protection) Bill 2026" is mentioned in an AGD consultation paper; status UNVERIFIED. https://consultations.ag.gov.au/rights-and-protections/privacy-reform/user_uploads/consultation_paper.pdf

**Notifiable Data Breaches (OAIC "Part 4: NDB Scheme", updated Feb 2025)** https://www.oaic.gov.au/privacy/notifiable-data-breaches/preventing-preparing-for-and-responding-to-data-breaches/data-breach-preparation-and-response/part-4-notifiable-data-breach-ndb-scheme
- Eligible breach = unauthorised access/disclosure/loss + serious harm "more probable than not" + not preventable by remedial action.
- Assess within 30 calendar days of suspicion (a maximum). Notice content: entity identity, description, kinds of information, recommended steps.
- Joint holding: a cloud/outsourced provider may "hold" the same records; each holder has obligations but one may notify for all; if none does, all may be in breach; contracts should allocate responsibility. APP 8.1 keeps the overseas discloser responsible.
- Penalty amounts: UNVERIFIED.

**Small business exemption (OAIC)** https://www.oaic.gov.au/privacy/privacy-guidance-for-organisations-and-government-agencies/organisations/small-business (no date shown)
- Small business = turnover $3m or less; exempt unless an exception applies. Exceptions include health service providers, businesses trading in personal information, Commonwealth contracted service providers, credit reporting bodies, AML/CTF reporting entities, CDR-accredited, opted-in (s6EA), related to a covered body.
- No pending removal shown; OAIC has recommended removal in a review submission (recommendation, not law). Look-back wording ("since 2001" vs "2002") differs across OAIC pages - UNVERIFIED.
- Implication: many single-venue restaurants (<= $3m) are exempt, but Cheery as a SaaS vendor is covered if its own turnover exceeds $3m or it "trades in personal information". Allergy data possibly making a venue a "health service provider": UNVERIFIED. Design to APP standard for all tenants.

**Spam Act 2003 / Do Not Call (ACMA)**
- Commercial electronic messages (email, SMS): consent, sender identification, working unsubscribe actioned within 5 business days and functional 30 days after sending, free, no login. https://www.acma.gov.au/avoid-sending-spam (direct fetch timed out; points from ACMA fact sheet https://www.acma.gov.au/sites/default/files/2024-05/Fact%20sheet%20-%20email%20and%20SMS%20unsubscribe%20rules.pdf ).
- Transactional/factual messages (booking confirmations, reminders): carve-out wording not read - UNVERIFIED.
- Do Not Call Register: voice telemarketing only; business numbers cannot be registered; hours 9am-8pm Mon-Fri, 9am-5pm Sat, none Sun. https://www.acma.gov.au/say-no-to-telemarketers ; https://www.donotcall.gov.au/industry/industry-overview/industry-standards . Does not govern SMS/email.

**GST (ATO / business.gov.au)**
- Registration at GST turnover $75,000; tax invoice needed to claim credits above $82.50 incl. GST; customer-requested invoice within 28 days unless sale <= $82.50. https://business.gov.au/registrations/register-for-taxes/register-for-goods-and-services-tax-gst . ATO tax-invoice page returned 403, so required-field lists are UNVERIFIED: https://www.ato.gov.au/businesses-and-organisations/gst-excise-and-indirect-taxes/gst/tax-invoices
- Restaurant GST-inclusive menu pricing rule (ACCC) not found - UNVERIFIED. Overseas-supplier SaaS GST: https://www.ato.gov.au/businesses-and-organisations/international-tax-for-business/gst-for-non-resident-businesses/how-australian-gst-works (details UNVERIFIED).

## 2. India

**DPDP Act 2023 / DPDP Rules 2025**
- Rules notified 14 November 2025 (a PIB release of 3 Dec 2025 says 13 Nov - discrepancy). PIB describes an 18-month phased compliance timeline (about mid-May 2027 is my arithmetic, not stated). https://www.pib.gov.in/PressReleasePage.aspx?PRID=2190014&reg=3&lang=2 and https://www.pib.gov.in/PressReleasePage.aspx?PRID=2198217&reg=3&lang=1 (both 403 on direct fetch; from search extracts). Rules PDF: https://static.pib.gov.in/WriteReadData/specificdocs/documents/2025/nov/doc20251117695301.pdf (text not extractable).
- Per-rule commencement split (immediate / 12 / 18 months) and consent-manager timing: UNVERIFIED. Data Protection Board stated as notified (PIB, 3 Dec 2025).
- Consent managers must be Indian companies; notices standalone and purpose-specific (PIB).
- Children: child = under 18 (Act: https://www.meity.gov.in/static/uploads/2024/06/2bf1f0e9f04e6fb4f8fef35e82c42aa5.pdf ); verifiable parental consent with exemptions for essential purposes (healthcare, education, real-time safety) per PIB; rule-level detail UNVERIFIED.
- Breach: individuals notified in plain language (PIB); 72-hour Board intimation UNVERIFIED.
- Processor vs fiduciary (Act s.8 contract/liability), cross-border (Act s.16, Rule 15): wording UNVERIFIED. Penalties up to INR 200 crore reported by PIB.

**TRAI / DLT (TCCCPR 2018)** https://www.trai.gov.in/sites/default/files/2024-09/RegulationUcc19072018.pdf ; https://trai.gov.in/advice-to-senders
- Senders register as Principal Entity; headers (<=11 chars) and content templates (fixed + variable parts) registered on operator DLT; traffic scrubbed against DLT records and preference/consent registers. 2023 re-verification directions: https://www.trai.gov.in/sites/default/files/2024-08/PR_No.12of2023.pdf . March 2026 draft consultation proposes tighter rules (https://www.trai.gov.in/sites/default/files/2026-03/Draft_CP_13032026_0.pdf) - final status UNVERIFIED. Template categories (transactional/service/promotional): UNVERIFIED.

**GST (India)**
- E-invoice IRP reporting within 30 days for AATO >= INR 10 crore from 1 Apr 2025; e-invoicing mandatory from INR 5 crore AATO (portal notices). https://einvoice6.gst.gov.in/content/revised-time-limit-for-e-invoice-reporting-for-businesses-with-aato-of-%E2%82%B910-crores-above/ . An Oct 2026 advisory says the GST Council (57th meeting) recommended extending e-invoicing to reverse-charge/import-of-services cases; rollout date not announced.
- Restaurant GST rates / ITC conditions, SaaS invoice fields: UNVERIFIED.

**RBI**
- "Digital Payments - E-mandate Framework, 2026" (RBI/DPSS/2026-27/396), issued 21 Apr 2026, effective immediately: AFA at registration and first transaction; recurring up to INR 15,000 without AFA (INR 1,00,000 for mutual funds, insurance, credit-card bills); AFA to modify/withdraw; issuer pre-debit notification at least 24h before. https://www.rbi.org.in/Scripts/BS_ViewMasDirections.aspx?id=13374
- Card-on-file tokenisation: since 1 Jan 2022 only issuers/networks may store actual card data; CoFT needs explicit consent + AFA; merchant must allow token de-registration. https://rbi.org.in/Scripts/NotificationUser.aspx?Id=12159 (search extract; verify current text).

## 3. UK (validation only)
- DUAA 2025: ICO says all stages in force (19 Jun 2026 update); most data-protection provisions from 5 Feb 2026; complaints-procedure duty 19 Jun 2026; PECR fines aligned to UK GDPR (up to GBP 17.5m or 4%). https://ico.org.uk/about-the-ico/media-centre/news-and-blogs/2026/02/statement-on-the-commencement-of-the-data-use-and-access-act-duaa/
- Breach: report to ICO within 72h of awareness; processors notify controller without undue delay. https://ico.org.uk/for-organisations/report-a-breach/personal-data-breach/personal-data-breaches-a-guide/
- PECR: electronic marketing needs consent, with commercial soft opt-in for existing customers (not bought-in lists). https://ico.org.uk/for-organisations/direct-marketing-and-privacy-and-electronic-communications/guidance-on-direct-marketing-using-electronic-mail/
- VAT: on-premises restaurant food at standard rate 20% per HMRC (page old; verify) https://www.gov.uk/guidance/vat-reduced-rate-for-hospitality-holiday-accommodation-and-attractions . HMRC Revenue and Customs Brief 5/2026 on a temporary reduced rate for children's meals - details UNVERIFIED: https://www.gov.uk/government/publications/revenue-and-customs-brief-5-2026-temporary-reduced-rate-of-vat-for-childrens-meals-tickets-and-family-attractions/temporary-reduced-rate-of-vat-for-childrens-meals-tickets-and-family-attractions . Consumer VAT-inclusive display rule, Art 28 terms, transfer mechanisms: UNVERIFIED.

## 4. PCI DSS
- v4.0.1 is the only active version; v4.0 retired 31 Dec 2024. All future-dated requirements (51) became effective 31 Mar 2025; v4.0.1 did not change that. https://blog.pcisecuritystandards.org/just-published-pci-dss-v4-0-1 ; https://blog.pcisecuritystandards.org/now-is-the-time-for-organizations-to-adopt-the-future-dated-requirements-of-pci-dss-v4-x . Existence of a newer version as of Nov 2026: UNVERIFIED.
- SAQ A (v4.0.1 r1, Jan 2025) is for fully outsourced e-commerce. New eligibility criterion: merchant confirms site is not susceptible to script attacks; 6.4.3, 11.6.1, 12.3.1 removed from SAQ A but remain in the standard. https://blog.pcisecuritystandards.org/important-updates-announced-for-merchants-validating-to-self-assessment-questionnaire-a ; https://docs-prv.pcisecuritystandards.org/SAQ%20(Assessment)/SAQ/PCI-DSS-v4_0_1-SAQ-A-r1.pdf
- PCI FAQ 1588: script criterion applies to merchants embedding a TPSP iframe/form, not to full redirects; both still meet reqs 2, 6, 8, 11 on their web servers. https://www.pcisecuritystandards.org/faqs/1588/
- SAQ A-EP applies where the merchant's page affects how card data is sent (not provider-hosted). Hosted fields/iframe generally targets SAQ A, but the final call is the acquirer/PSP's - UNVERIFIED for any specific PSP. Cheery's own service-provider obligations: QSA.

## 5. Standards (owner bodies)
| Standard | Current edition found | Source |
|---|---|---|
| ISO/IEC 27001 | 3rd ed. 2022 (Oct 2022) + Amd 1:2024 (climate action, Feb 2024) | https://www.iso.org/standard/27001 ; https://www.iso.org/standard/88435.html |
| ISO/IEC 27002 | 2022 - UNVERIFIED (ISO page not fetched) | https://www.iso.org/standard/75652.html |
| OWASP ASVS | 5.0.0 stable, 30 May 2025 (previous 4.0.3) | https://asvs.owasp.org/ |
| OWASP Top 10 | 2025 | https://owasp.org/Top10/2025/ |
| OWASP API Security Top 10 | 2023 (latest) | https://owasp.org/API-Security/ |
| SOC 2 TSC | 2017 TSC with revised points of focus (2022); AICPA exposure drafts of proposed 2027 TSC exist | https://www.aicpa-cima.com/topic/audit-assurance/audit-and-assurance-greater-than-soc-2/ |

## Requirement -> architectural implication -> needs legal confirmation
| Requirement | Architectural implication | Legal? |
|---|---|---|
| AU APPs; NDB 30-day assessment; joint holding | Tenant-scoped data model; breach runbook and per-tenant incident records; DPA allocating notifier role; access audit log | Y |
| AU small-business exemption / Cheery's own coverage | Build to APP standard for all tenants; no cross-tenant reuse of guest data ("trading") | Y |
| Allergy and children data | Field-level encryption, restricted roles, minimal retention, separate purpose; never used for marketing | Y |
| Statutory tort (in force) | Privacy-by-design; no covert tracking/recording features | Y |
| ADM transparency from 10 Dec 2026 | Inventory automated seating/waitlist/no-show scoring; keep human override; privacy-policy template hooks | Y |
| Children's Online Privacy Code (pending) | Child is a guest attribute, not a user; no child-facing UX; watch final Code | Y |
| Spam Act consent/unsubscribe/sender ID | Per-channel, per-purpose consent ledger; separate transactional and marketing streams; automate STOP well inside 5 business days | Y |
| Do Not Call | Out of scope unless outbound voice is added | Y |
| AU GST invoices | Tax-invoice generator (ABN, GST, inclusive display); store tax-inclusive price plus tax breakdown | Y |
| DPDP consent, notice, children VPC | Versioned notices, purpose-bound consent records with withdrawal; under-18 guest handling; Cheery as processor under contract | Y |
| DPDP phasing / cross-border | Region-aware config; keep India-region hosting option; transfer register | Y |
| TRAI DLT | Per-tenant sender/header and template registry; pluggable Indian SMS aggregator; template-ID sends; no free-text SMS in India | Y |
| India GST / e-invoice | Pluggable invoice service (IRP); HSN/SAC and rate per item; defer to POS phase | Y |
| RBI e-mandate / tokenisation | Never store PAN; PSP tokens/CoFT; PSP-run mandates; surface 24h pre-debit in UX | Y |
| UK GDPR / PECR / VAT | Same consent ledger and DSR tooling; UK region flag; tax class per menu item incl. children's meals | Y |
| PCI DSS 4.0.1 | Provider-hosted page/iframe only; no card data in Cheery; script-integrity monitoring if Cheery controls the embedding page; target SAQ A | N (QSA/acquirer) |
| ISO 27001 / SOC 2 | Access, logging, change and vendor controls from day one for later certification | N |
| ASVS 5.0 / Top 10 2025 / API Top 10 2023 | ASVS 5.0 L2 as baseline; per-tenant authorization (BOLA) tests in CI | N |

## Gaps / could not verify
- Spam Act factual/transactional message carve-out; penalty amounts (Spam, NDB, new 2024 civil penalty tiers).
- Tort commencement provision text; status of any 2026 privacy reform bill.
- Whether a restaurant booking SaaS falls under the Children's Code; whether allergy data makes a venue a "health service provider".
- ATO tax-invoice required fields (403); ACCC GST-inclusive menu pricing; ATO treatment of overseas SaaS.
- DPDP Rules: per-rule commencement, consent-manager timing, Rule 15, breach timings, Section 8 processor wording, VPC mechanics (read the gazetted Rules).
- TRAI template categories and final status of March 2026 draft; DLT onboarding costs.
- India GST: restaurant rate/ITC, SaaS invoice fields.
- RBI tokenisation: only search extracts read.
- UK: Art 28, transfers, consumer VAT display, RCB 5/2026 details.
- PCI: SAQ A-EP v4.0.1 text; any version newer than 4.0.1; PSP-specific SAQ outcome.
- ISO/IEC 27002 edition page; SOC 2 2027 TSC exposure-draft status.
