# Cheery — Stage A: Prototype Analysis and Product Discovery

Source: `Cheery Outlet Platform.html` (1,288 lines, ~226 KB, single file, no backend).
Method: read the full script section (data, reservation logic, all six role views, all action handlers). CSS and the embedded base64 logo were skipped. Three claims below were confirmed by grep and are marked **[verified]**.

Legend — **Real logic**: working business rule in the prototype. **Partial**: works for the demo path only. **Simulated**: UI/text only, nothing behind it. **Claimed**: stated in on-screen copy but not implemented.

---

## 1. What the prototype actually is

- A client-side demo: all state in one in-memory object `S`; refresh or "Reset demo" restores seed data. No server, API, database, auth or persistence.
- A **simulated clock** (starts 6:50 PM, 1 minute per 10 seconds) and only three days (`Today · Sat`, `Sun 25`, `Mon 26`). Times are minutes-since-midnight, days are indexes 0–2. There are no real dates, time zones or recurring schedules.
- Built for **one specific customer** ("Cheery Restaurant Group": Dashi, Champ Thai, JA² Square), with a "Requests from the client" modal listing what the client asked for. Menu is copied from "JA² Square whole-day menu" (Square). Guest-facing strings are hard-coded to "JA² Club", AU phone formats, AUD, GST.
- Six roles, switched by buttons in the header: Guest, Host, Waiter, Kitchen, Branch manager, Group admin.

**Implication:** the prototype is a strong *workflow* reference and a weak *domain model* reference. Its data model was shaped by a demo, not by multi-tenant SaaS needs.

## 2. Hierarchy found in the prototype vs. the proposed hierarchy

| Proposed (master prompt) | In prototype |
|---|---|
| Platform | none |
| Tenant / Customer Organization | none (the one group is implicit) |
| Restaurant Group / Business | "Cheery Restaurant Group" (admin screen title only) |
| Brand | `brandKey` string (dashi, champ, ja2) |
| Branch / Outlet | the `BRANDS` object — **keyed by branch** (`dashi`, `dashi2`, …), mixing brand and branch attributes |
| Tables / Resources | `res[]` list of tables **and counter stools**; `area` is a plain string, not an entity |

Findings:
- Tenant level is absent. Group → Brand → Branch collapses into one flat object.
- Brand-level things live at brand: message templates (`tplOf(brandKey)`), website domain, cuisine. Branch-level: address, tables, services, closures, status, country.
- **Menu, prices, availability, and all rules (`RULES`) are global across every brand and branch.** The settings screen says so explicitly: "in the prototype these settings are shared by all branches; in the full build each branch can override them." Same for menu: "prices are set per brand and availability per branch" is future-only.
- A fourth hierarchy-adjacent concept appears: **"JA² Club"** — a cross-brand guest membership (tiers, points). That is group-level guest data, not tenant-scoped configuration.

## 3. Roles and permissions

| Role (prototype) | Screens |
|---|---|
| Guest (not logged in) | Restaurant picker, booking, details, confirmation/cancel, waitlist, group enquiry, table menu/order/pay, QR flow |
| Host | Seating grid, floor view, list, waitlist, booking drawer, walk-in/phone booking, alerts, offline chip |
| Waiter | "Do next" task list, order entry, payment |
| Kitchen | KDS: New → Cooking → Ready, station filter, all-day counts |
| Branch manager | Tonight KPIs, Enquiries, Tables & seats, Menu, Hours & closures, Features & rules, Messages, Activity |
| Group admin | Brands & branches, Staff & access, Guests, Integrations |

- Permission matrix: 6 permissions × 5 roles (Host, Waiter, Kitchen, Branch manager, Group admin). **[verified]** `S.perm` is only defined, toggled and displayed — **no screen or action ever checks it.** The toggles are cosmetic.
- The matrix has no permission for orders, kitchen actions, floor/reset, payments/refunds, waitlist, QR management or messaging, yet those actions exist.
- Staff list is static (6 people). Sign-in described as "4-digit PIN on shared tablets". No invitation flow, MFA, device registration, or multi-branch membership (one staff row = one place, "All brands" for group admin).
- Activity-log "who" is a role label (`WHO[S.role]`, e.g. "Tom · Host"), not a user identity.
- No waiter-to-table assignment: "All my tables" shows every seated table in the branch.

## 4. Screen inventory

**Guest (phone frame):** restaurant list (nearest first, brand filter, party size, "your usual", quick time chips, directions link) → booking (party, date, seating area, slot grid with seats-left, hold text, waitlist offer, >8 → enquiry) → details (allergy, children, high chair, seating request, occasion, deposit or card-hold block, SMS language, marketing opt-in) → confirmation (SMS preview, change, cancel with late-fee warning) → cancelled / waitlist status / enquiry form + confirmation → at-table menu & order → bill & pay → paid (star rating placeholder). QR mode: scan → start-table request → menu.

**Host (tablet):** seating grid timeline (15-min columns, free-seats strip, arrivals-vs-pacing strip, booking blocks, reset buffer, overrun stripes, now-line), floor view, list view, waitlist, booking drawer (details, guest history, move-to, seat, +15 min, no-show, cancel, finish, fee charge/waive, history), booking form (phone booking / walk-in, duration, area, auto-allocation preview, pacing warning/override, guest lookup), alerts, offline chip.

**Waiter (phone):** prioritized to-do (serve ready food, overrun tables, bill requests, take order, finishing-soon nudge, approve QR order, guest help call, table reset), table order entry with per-line notes, payment.

**Kitchen (wall screen):** three columns, ticket colour aging (green <8 min, amber <15, red), station filter, all-day counts.

**Branch manager:** KPI tiles + covers chart + longest-seated table; Enquiries; Tables & seats editor (areas, tables, stools, seats, online toggle, joinable pairs); Menu editor; Hours & closures; Features & rules (see §7); Messages (templates × language, live preview); Activity log.

**Group admin:** Brands & branches (table + add-branch form + "planned" rows), Staff & access (permission matrix + staff list), Guests (static sample list), Integrations (static list with phases).

## 5. Workflow inventory

1. **Online booking:** pick branch → party/date/area → slot (availability + pacing + allocation) → details → deposit or card-hold → confirm → SMS.
2. **Phone booking / walk-in (host):** form with live allocation preview, conflict and pacing checks, guest lookup by name/phone.
3. **Arrival and seating:** booked → seated; if table still occupied, tries to reallocate; otherwise tells host to offer bar/wait.
4. **Overrun handling:** alert when a table passes planned finish; shows next booking; "move next booking".
5. **Late / no-show:** after grace (15 min) prompt → mark no-show → optionally charge or waive fee.
6. **Cancellation:** guest self-cancel; late cancel (inside window, card held) → fee; host cancel.
7. **Waitlist:** walk-in or online join → quoted wait (computed by scanning future allocation) → "table ready" text → seat.
8. **Group enquiry:** guest enquiry (>8 online limit) → manager offers space/time/deposit by email → "guest accepted" (staff button) → booking created.
9. **Order-to-kitchen:** waiter or guest phone → ticket (optionally pending staff approval) → kitchen new/cooking/ready → waiter serves.
10. **Bill and pay:** bill requested → waiter or guest pay → table released as "needs reset" → staff reset → ready.
11. **QR table start:** scan → party size → staff confirm (optional) → table opens for ordering.
12. **Branch setup:** manager edits tables, menu, hours, closures, rules; admin adds branches.

## 6. Business rules found (with status)

| Rule | Status |
|---|---|
| A seat cannot hold two bookings whose time plus 10-min reset buffer overlap (both sides) | **Real logic**, client-side only. Modal says "in the real build this is enforced in the database too" |
| Duration by party size (1–2: 75, 3–4: 90, 5–6: 120, 7+: 135, counter: 60) | Real logic |
| Allocation order: smallest single table that fits (within +2 seats, or exact for ≤2) → counter stools for ≤2 (contiguous run) → joinable pairs → any larger single | Real logic |
| Counter seats are booked per stool; party of N takes N contiguous stools | Real logic |
| Joinable tables are only **pairs** listed manually; adjacency not modelled | Partial |
| Pacing: max guests per 15 min (default 12), optional max tables per 15 min; host override is logged | Real logic |
| Online booking max 8; above → enquiry (or "call the restaurant") | Real logic |
| Deposit from 6+ guests ($20 pp), deducted from bill | Partial — flag only, no payment |
| Card hold + no-show fee from 4+ guests ($25 pp), free cancel until 24 h | Partial — card "•••• 4242" is hard-coded |
| Late hold 15 min then no-show prompt | Real logic (alert only; no-show is manual) |
| Closed day hides online booking; manager told how many booked guests need a call | Real logic |
| Table reset status blocks allocation to walk-ins/QR for tables not yet reset | Real logic |
| Weekend 10% surcharge, GST = total ÷ 11 (AU GST inclusive) | **Hard-coded**: surcharge applied when `day===0`, not a configurable rule |
| Loyalty: 1 point per whole dollar, usable across brands | Claimed (display only) |

## 7. Configuration found ("Features & rules" + other editors)

- **Feature switches** (`RULES`): pacing, deposits, no-show protection, waitlist (+ online waitlist), table reset, reminders, group enquiries, QR ordering (+ mode view/confirm/direct, staff-start, pay-at-table, call-waiter), counter bookable online.
- **Numeric:** durations, buffer, grace, max online, deposit threshold/amount, fee threshold/amount, cancellation window, pace limits, waitlist hold minutes.
- **Branch:** services (Lunch/Dinner, days, times, on/off), closures (day + reason), tables/areas/stools, joinable pairs, online toggle per seat.
- **Brand:** message templates (7 types × en/ja/zh), domain.
- **Admin:** country → currency/time zone/language/payment provider mapping (`CTRY`: AU/Square, JP/Square JP, HK/Stripe or Airwallex).

Important distinctions the architecture must preserve:
- **Business settings** (e.g. pacing limits) vs. **subscription entitlements** (whether the customer's plan includes pacing at all) are mixed together in the prototype's "feature switches".
- **[verified]** Services/hours editor **does not drive availability**: the slot grid uses constants `OPEN=17:00`, `CLOSE=22:00`; `services` is only read to print "Open today · …". Lunch is configured but never bookable.
- **[verified]** `holdMin` (waitlist hold) only appears in message text; nothing expires a notified waitlist entry.

## 8. Entities discovered

Reservation (also used as walk-in, waitlist-seated, QR-opened table — it doubles as the *dining visit* and the *check*), Table/Counter stool ("res"), Area (string), Joinable pair, Service period, Closure, Waitlist entry, Group enquiry, Order (`orders[resId].lines`), Ticket (kitchen), Menu item (flat, category string, station, tags, origin, availability), Guest profile (**keyed by name string** in the prototype), Message template, QR request, Help request, Table state (dirty/resetting), Activity log entry (free text), Brand/Branch, Staff, Role/permission.

Architecture implications:
- One `reservation` object is overloaded as booking, seating, order owner and bill. A real model needs at least **Booking → Seating/Visit → Order/Check → Payment**, with walk-ins having no booking.
- Guest identity must be keyed by verified contact (phone/email) with merge rules, not by name.
- Activity log must become structured events (actor, entity, before/after, tenant/branch), not strings.
- Menu needs: item vs. price vs. availability separated by scope (brand price, branch availability), modifiers, tax category, allergens (currently `tags` and a one-letter origin code with no legend).

## 9. Integrations named

On the Integrations tab (static): brand websites on **nopCommerce** (booking widget), **Square POS** (sales/spend history → guest profile; AU, JP), **SMS & WhatsApp** (incl. reply-to-cancel), **Reserve with Google**, **Stripe/Airwallex** (HK), **Uber Eats / DoorDash** (delivery → kitchen screen). Also used in-UI: Google Maps links, QR generation (qrcodejs from a CDN), Google Fonts.

None of these are implemented. The tab labels them Phase 1/3/4 but those phases do not match the master prompt's phase plan.

## 10. Simulated or claimed-but-missing (do not treat as scope without confirmation)

| Area | What the prototype shows | What exists |
|---|---|---|
| Offline operation | "Online · synced / Offline · N saved on tablet" chip | **A toggle and a counter.** No local storage, queue, or sync. Everything keeps running against in-memory state. |
| Payments | Deposit, card hold, pay-at-table, "Card paid via Square terminal", split by guest/item | Buttons set status flags. No gateway, no tokens, no refunds, no tips, no split. Card number is a literal. |
| Notifications | SMS/email previews, reminders at 24 h/2 h, reply "C" to cancel | Template text rendering and toasts only. No sending, scheduling, inbound handling or opt-out. |
| Authentication | PIN sign-in on shared tablets | Role buttons. No sign-in at all. |
| Permissions | Role × permission matrix | **[verified]** Never enforced. |
| Loyalty | JA² Club tiers/points, "1,240 points" | Static sample data; one hard-coded profile pre-fills every booking. |
| Reporting | KPI tiles, covers chart, "turns" | Revenue is **seeded numbers** ($3,180 / $2,410 / $1,960) plus demo payments; "target 2.5 on Saturday" and "seats free at 8:30 PM" are hard-coded. |
| Table QR security | "Replace code" invalidates sticker | Token is a 5-char client-side hash; nothing validates it. |
| Booking modification | "Change time" | Cancels the booking immediately, then starts a new booking — **the old slot is released before a new one is secured**. |
| Slot hold | "Your table is held 7:00–8:30" | No hold during checkout. `gconfirm` recomputes allocation at confirm time (can differ from what was displayed); no deposit-payment window or TTL. |
| Review/rating | Star buttons | Not wired. |
| Kitchen "ready pings waiter" | Toast | No notification channel. |
| Delivery / catering | "JA² Catering" planned row; Uber Eats/DoorDash | Nothing. |
| Group enquiry acceptance | "Guest accepted · create booking" | Staff clicks it; there is no guest-facing accept/pay link or offer expiry. |
| Multi-language UI | Message templates in en/ja/zh | UI strings are English only. |

## 11. Hard-coded country/commercial assumptions (conflicts with "no hardcoding")

AUD `$` everywhere (`money()`), GST ÷11 inclusive-pricing, AU phone formats (`04xx xxx xxx`), "NSW/Sydney" addresses, `Australia/Sydney`-style timezone strings as labels only, "Square" as the payment provider, weekend surcharge as a built-in, English-only UI, Sat/Sun/Mon labels. Admin `CTRY` covers **AU, JP, HK — not India.**

## 12. Missing requirements (not in the prototype at all)

Tenant/organization model, sign-up/onboarding, real authentication and MFA, subscriptions/entitlements/billing, real calendar and time zones, recurring weekly hours and public holidays, service periods driving availability, booking edit (party/time) as a true transaction, deposit refund on cancellation (deposit bookings have no cancellation-fee path), refunds/voids/discounts (a permission exists, no function), tips, split bills, tax configuration, invoices/receipts, GST/tax reporting, kitchen course firing/recall/printing, waiter section assignment, shifts and scheduling, guest consent records and data-subject requests (access/correction/deletion/export), data retention, audit immutability, reporting exports/BI, multi-device concurrency, conflict resolution, rate limiting/abuse protection on public booking, inbound SMS handling, branding per tenant, accessibility conformance, observability.

## 13. Architectural implications and risks (ranked)

1. **Scope identity — replace the POS or integrate with it?** The prototype does ordering and payment, yet the integration list syncs from Square POS and the waiter pays "via Square terminal". This decides whether Orders/Payments are core domains or integration adapters, and drives PCI scope.
2. **Prototype is client-shaped, not SaaS-shaped.** Tenant, subscription and per-branch overrides are all absent; global menu and rules must be re-modelled with inheritance (org → brand → branch).
3. **Reservation concurrency is the product's core risk.** Today's guard is client-side check-then-write. Needs server-side atomic allocation (exclusion constraints/locks, idempotency keys, short-lived holds). Offline hosts must not be allowed to promise availability.
4. **Offline is a claim, not a design.** Needs an explicit operation-by-operation offline/online matrix before any architecture commits to offline-first.
5. **Overloaded `reservation` entity** — split into booking/seating/order/payment lifecycles.
6. **Guest data and privacy:** cross-brand profiles keyed by name; allergies and children are health-adjacent personal data; marketing opt-in exists but no consent record; cross-brand sharing within one group vs. across tenants needs a rule. Requires legal confirmation, not assumptions.
7. **Derived-from-wall-clock logic** (`effEnd`, `isLate`, `isOver`) must become stored timestamps/events in a real system to be auditable and testable.
8. **Entitlement vs. configuration vs. permission** are all conflated in RULES/PERMS; the master prompt requires them separate.
9. **Localization:** nothing is parameterised; GST, currency, phone, language and surcharge must move to country/tenant/branch configuration.
10. **Customer-driven scope risk:** several features come from one client's list (JA² Club, nopCommerce widget, Square, weekend surcharge). A commercial multi-tenant product needs these classified as core, configurable, or per-customer integration.

## 14. Feature → domain → ownership trace (proposed, for confirmation)

Domains refer to the master prompt's list (and note where consolidation looks right).

| Feature (prototype) | Status | Proposed domain | Owns data | Key dependency / note |
|---|---|---|---|---|
| Brand/branch list, status, address, add branch | Partial | Brand & Branch (3) | branch, brand, address | Tenant hierarchy missing |
| Country/currency/timezone/payment mapping | Simulated | Localization & Regional Config (20) | country profile | Must be data, not constants |
| Staff, roles, permission matrix, PIN | Simulated | Identity & Access (1) | user, membership, role, permission | Never enforced today |
| Feature switches (pacing, QR…) | Partial | Restaurant Configuration (5) + Entitlements (4) | config values; entitlement grants | Separate config from plan entitlement |
| Tables, areas, stools, joinable pairs | Real | Table & Seating (8) | resource, area, combination | Areas should be entities; adjacency |
| Booking rules (duration, buffer, grace, max online) | Real | Restaurant Configuration (5) | rule set with inheritance | Branch override of org default |
| Slot availability, allocation, conflict guard | Real (client) | Reservation (7) + Table & Seating (8) | booking, allocation | Server-side atomic; core risk |
| Pacing | Real | Reservation (7) | pacing config; counts derived | Per branch/service |
| Online booking, guest details, SMS preview | Partial | Reservation (7), Guest (6), Notification (15) | booking, guest, consent | Slot hold + idempotency |
| Deposit, no-show fee, card hold | Simulated | Payment (14) + Reservation (7) | payment intent, card token ref | PCI scope; hosted fields |
| Cancel / late cancel / no-show | Partial | Reservation (7) | status history, fee decision | Refund path missing |
| Walk-in, waitlist, quote | Real | Waitlist (9) | waitlist entry | Hold expiry missing |
| Group enquiry & offer | Partial | Reservation (7) / Events sub-area | enquiry, offer | Guest-facing accept/pay |
| Floor view, table reset | Real | Floor & Staff Ops (13) | table state | |
| Menu, prices, availability, stations | Partial | Menu & Catalog (10) | item, price, availability, modifier | Global today; scope inheritance needed |
| Ordering (waiter, guest, QR approve) | Partial | Ordering (11) | order, line, modifier | Detach from reservation |
| Kitchen tickets, stations, aging | Real | KDS (12) | ticket, status events | Real-time channel needed |
| Bill, GST, surcharge, pay, points | Simulated | Payment (14), Ordering (11), Localization (20) | check, tender, tax lines | Tax from country config |
| QR codes, table tokens, regenerate | Partial | QR Management (16) | qr token, revocation | Signed server-side tokens |
| Message templates × language × brand | Partial | Notification & Communication (15) | template, delivery log | Provider adapters, opt-out |
| KPIs, charts | Simulated | Reporting & Analytics (17) | derived/read models | Seeded numbers today |
| Activity log | Partial | Audit & Activity (18) | structured audit event | Actor identity |
| Guest profiles, history, notes, JA² Club | Simulated | Guest & Customer (6) (+ Loyalty later) | guest, visit history, preferences | Keying, consent, cross-brand rule |
| Integrations tab | Simulated | Integration Management (19) | connection, credentials, sync log | Scope question #1 |
| Offline host tablet | Simulated | Cross-cutting (client sync) | local op queue | Needs offline matrix |

Possible consolidation to evaluate in Stage C: Table & Seating with Reservation allocation; Guest with consent/privacy; Audit with Activity; QR into Ordering; Localization as a configuration capability rather than a module.

---

## Handoff to Stage B

Items I cannot resolve from the prototype and will put into prioritized question batches: (1) POS replacement vs. integration, (2) who the first customer is and how much of the prototype is theirs vs. product-generic, (3) India vs. the Japan/Hong Kong markets the prototype shows, (4) offline scope, (5) guest identity and cross-brand data sharing, (6) payment model (provider, who is merchant of record, deposits/fees), (7) team size and existing stack.
