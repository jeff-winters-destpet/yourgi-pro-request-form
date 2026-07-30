# Header

| **Title**          | PRD \| Pro Request Form (concierge intake) |
|--------------------|--------------------------------------------|
| **Author / DRI**   | Jeff Winters                               |
| **Target release** | Pilot live 2026-07-30 (internal). Public launch: TBD, gated on sign-offs below |
| **Links**          | [Live form](https://jeff-winters-destpet.github.io/yourgi-pro-request-form/) \| [Repo](https://github.com/jeff-winters-destpet/yourgi-pro-request-form) \| [Wiring guide](Teams%20wiring%20guide.md) \| ADO Epic: TBD |

## Sign-off

| Role | Name | Status | Date |
|------|------|--------|------|
| Design lead | | ☐ Approved | |
| Engineering lead | | ☐ Approved | |
| Product (DRI) | Jeff Winters | ☐ Approved | |

> [!info] Before build starts
> Design and eng do not start building until all three sign-offs are recorded. A working pilot already exists (see links); sign-off here covers the public launch scope, not the pilot.

# Problem & user

Pet parents who need Pro care have to search, compare profiles, and hope: our Booking Conversion Rate is under 2% against a 4% target, and every abandoned search is a pet parent who wanted care and left without requesting it. There is no low-effort path for the pet parent who just wants to say what they need and have Yourgi handle the rest, even though concierge-assisted matching is our stated differentiator ("We match you with 3-5 pros" vs. "You search for hours"). Theme 1 (Consumer Bookings Funnel) owns this gap now.

# Proposed solution

A single-page, brand-styled public form where a pet parent answers five quick questions: ZIP, service, number of pets, dates and time window, and phone number (email and notes optional). Submitting posts an Adaptive Card into the concierge team's **Booking Form** Teams channel; a concierge looks the phone number up against existing pet parent data, then texts the pet parent via Send Blue to arrange the booking. The form is deliberately anonymous: the public page has no login, so the phone number is the match key and matching happens after submit rather than asking the pet parent to identify themselves. The bet is that a human-in-the-loop intake converts intent that search currently loses, while the concierge conversation itself produces the booking request that moves M1.

# Scope

**In scope:**

- The four live Pro services: Boarding, Daycare, Dog Walking, House Sitting (date ranges for Boarding and House Sitting, single date otherwise)
- Anonymous-only submission; phone required (Send Blue is the reply channel), email and notes optional
- SMS consent line at submit (wording pending legal review)
- Teams Adaptive Card intake with request ID, likely market from ZIP prefix, and a match-lookup reminder
- Delivery confirmation: pet parent sees success only after the request is accepted; failures show an inline retry message
- Analytics events for the M1 funnel (`request_form_viewed`, `service_selected`, `request_form_submitted`, `request_post_result`)
- For public launch: a server-side relay that owns the webhook secret, validates input, builds the card, rate-limits, and runs the phone/email lookup to enrich the card with the matched pet parent and pet names

**Out of scope:**

- Login, account creation, or prefill on the public page
- Automated Pro matching, real-time availability, pricing, or payment
- Grooming, Training, and Vet Care (not live on the Pro marketplace)
- Native app surfaces (web only)
- Replacing the existing search-and-book funnel (this is an additional path, not a substitute)

## Marketplace check

> [!warning] Required — do not skip

- **Parent / Consumer side changes:** the new request form (web). No changes to existing search or booking flows.
- **Pro / Service Provider side changes:** none directly. Demand reaches Pros through the concierge using existing booking tools. Risk to watch: requests in ZIPs without available Pros create outreach the concierge cannot fulfill (see guardrail metric).
- **Internal ops / admin tooling changes:** new Booking Form Teams channel and Power Automate flow ("Booking Form intake (pro request form)"; needs a co-owner). Concierge team takes on manual phone-number lookup and a text-back-within-the-hour expectation during business hours. Flow run history is the pilot's failure monitor.

# User stories

- **[Parent]** As a pet parent who doesn't have a person for this, I want to say what I need in five quick questions so that Yourgi finds the right Pro without me scrolling profiles.
- **[Parent]** As a returning pet parent, I want the concierge to already know me and my pets when they text so that I don't repeat myself.
- **[Internal]** As a concierge, I want every request to arrive as a structured card with a request ID, likely market, and contact info so that I can triage and respond within the hour.

# Requirements

### Parent web form

- Must collect ZIP (5 digits), service (one of four), pet count (1 to 6+), date or date range, time window, phone (10 digits, auto-formatted); email and notes optional, validated only when provided
- Must reject past dates and end-before-start ranges with field-specific, on-voice error messages
- Must show the SMS consent line adjacent to the submit button
- Must confirm success only after the request is accepted upstream; on failure, keep the pet parent on the form with a retry message
- Must meet WCAG 2.1 AA (labels, focus management, contrast, keyboard operability) and follow Brand Guidelines V1.0 (National 2 type, palette, canonical service one-liners, no em dashes)

### Backend / shared (public launch)

- Relay endpoint must hold the webhook secret server-side; the page must contain no secret
- Must accept only the form's field schema, build the Adaptive Card server-side, and reject arbitrary payloads
- Must rate-limit by IP and include a honeypot check; abuse controls sized for an unauthenticated public endpoint
- Must run the post-submit lookup (phone first, email fallback) and enrich the card with matched pet parent and pet names
- Must emit Mixpanel events server-side for the M1 funnel

### Admin / internal tooling

- Teams card must carry: request ID, service, ZIP + likely market, pets, dates, time window, phone, email, notes, match status
- Power Automate flow must have at least one co-owner beyond the DRI
- Concierge response expectation ("texts back within the hour during business hours") must be signed off by the team that owns the channel, or the copy softened before launch

# Success metrics

**Primary:**

- M1 Booking Conversion Rate: contribute to moving BCR from under 2% toward the 4% target; measured via `request_form_submitted` volume and the share of submissions that become booking requests (baseline TBD, analytics review)

**Guardrail:**

- M2 Match Rate must not regress: requests from ZIPs without available Pros must stay a small share of submissions (threshold TBD), or we gate the form by SLA coverage

# Open questions

- Overlap with Theme 4 "Pro book for a customer" (June 2 batch) and Theme 6's embeddable bookings widget: same design space, needs a coordination check before eng work starts
- Hosting: move off github.io to a yourgi.com path? Resolves the National 2 web font license question (license follows the serving domain) and improves trust and analytics
- Legal: confirm or replace the SMS consent wording; confirm privacy policy linkage for phone/email collection
- Concierge staffing: who owns the Booking Form channel rota, and is within-the-hour sustainable at projected volume?
- Do we gate submissions by SLA coverage (5+ available active Pros within 5-6 miles) at launch, or accept all ZIPs and let the concierge triage out-of-market demand?
- Spam posture: is honeypot + rate limiting enough for launch, or do we add a CAPTCHA despite the friction?
- When does the manual phone lookup become automated (relay enrichment), and does that land with the launch or as a fast follow?
