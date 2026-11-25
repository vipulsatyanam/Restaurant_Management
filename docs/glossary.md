# Cheery Domain Glossary (v0.1)

Terms are used consistently across code, documents and UI. Add new terms here first.

| Term | Meaning |
|---|---|
| **Organization** | The customer account and tenant: one contract, one subscription, one data wall |
| **Brand** | A restaurant brand within an organization (name, branding, defaults) |
| **Branch / Venue** | One physical restaurant location of a brand; has a country and time zone |
| **Area** | A named zone in a branch (dining room, garden, counter) |
| **Resource** | A bookable unit: a table or a counter seat |
| **Combination** | A defined set of resources that can be joined for larger parties |
| **Service period** | A recurring opening window (e.g. dinner Mon–Sun 17:00–22:00) |
| **Service day** | The branch-local business day a booking belongs to (a 1 am booking may belong to the previous day) |
| **Booking** | A guest's reservation for a party at a time; has a lifecycle (held, confirmed, seated, completed, cancelled, no-show, expired) |
| **Hold** | A short-lived reservation of resources while the guest completes the booking |
| **Allocation** | The assignment of a booking to resources for a planned time range, protected from overlap |
| **Turn buffer** | Time added after a booking for resetting the table |
| **Dwell time** | Planned duration of a booking |
| **Pacing** | A limit on guests or tables arriving per interval |
| **Walk-in** | A party seated without a prior booking |
| **Waitlist** | A queue of parties waiting for a table, notified when one is ready |
| **Enquiry / Offer** | A large-group request and the manager's proposed space, time and deposit |
| **Guest** | A diner (tenant-owned record) |
| **Consent record** | An append-only record of a guest's consent decision |
| **Member / Membership** | A person's role at a scope (organization, brand, branch) |
| **Permission / Entitlement / Setting** | May this person do it / did the customer buy it / is it switched on and how |
| **Plan** | A subscription package with entitlements |
| **Cell** | A deployment of the platform (database + region) serving a set of tenants |
| **Country pack** | Versioned data describing a country's currency, tax, locale, formats and rules |
| **Partner** | A Cheery salesperson (contractor) who registers leads |
| **Lead** | A prospective restaurant customer registered by a partner |
| **Commission plan** | A partner's rate, basis, duration and clawback terms, versioned |
| **Clawback** | Reversal of commission when a customer cancels/refunds/does not pay within the window |
| **Port / Adapter** | A Cheery-owned interface to an external capability / a provider-specific implementation |
| **Outbox** | A table that records events in the same transaction as the change that produced them |
| **Idempotency key** | A client-supplied token that makes a repeated request safe |
| **Policy snapshot** | The cancellation/deposit terms stored with a booking at the time it was made |
