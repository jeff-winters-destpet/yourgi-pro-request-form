# Header

| **Title**          | PRD \| Pro Request Form (concierge intake experiment) |
|--------------------|--------------------------------------------------------|
| **Author / DRI**   | Jeff Winters                                            |
| **Target release** | Pilot live 2026-07-30. Squad experiment, no engineering involvement by design |
| **Links**          | [Live form](https://jeff-winters-destpet.github.io/yourgi-pro-request-form/) \| [Repo](https://github.com/jeff-winters-destpet/yourgi-pro-request-form) \| [Wiring guide](Teams%20wiring%20guide.md) \| ADO Epic: n/a until graduation |

## Sign-off

| Role | Name | Status | Date |
|------|------|--------|------|
| Product (DRI) | Jeff Winters | ☐ Approved | |
| Concierge / ops owner | | ☐ Approved | |
| Web (public page link placement) | | ☐ Approved | |

> [!info] Experiment, not a build
> This is a squad-run experiment assembled without engineering: a static page, a Teams workflow, and a concierge team. No eng sign-off is required because no eng systems are touched. If the experiment graduates (see decision criteria), a build-grade PRD goes through the normal design/eng review path.

# Problem & user

Pet parents who need Pro care have to search, compare profiles, and hope: our Booking Conversion Rate is under 2% against a 4% target, and every abandoned search is a pet parent who wanted care and left without requesting it. There is no low-effort path for the pet parent who just wants to say what they need and have Yourgi handle the rest, even though concierge-assisted matching is our stated differentiator ("We match you with 3-5 pros" vs. "You search for hours").

# Hypothesis & experiment design

**Hypothesis:** if pet parents can request care in five quick questions and get a human text back, a meaningful share of intent that search currently loses will convert into booking requests.

- **Setup:** brand-styled static form (GitHub Pages) posts an Adaptive Card into the concierge team's **Booking Form** Teams channel; a concierge matches the phone number against existing pet parent data and replies via Send Blue.
- **Traffic:** linked from the public page. No paid promotion during the experiment.
- **Duration:** time-boxed; suggest 6 weeks from public link placement (DRI to confirm).
- **Decision criteria (strawman numbers, DRI to set):** graduate if submissions sustain a meaningful weekly volume (e.g. 25+/week) AND a meaningful share become booking requests (e.g. 30%+); sunset if volume or conversion stays negligible; iterate in between.
- **Graduation means:** a production build with engineering (server-side relay, automated lookup, yourgi.com hosting) goes through the standard PRD review path. Sunset means: unlink the page, turn off the flow, keep the learnings.

# Proposed solution

A single-page public form: ZIP, service, number of pets, dates and time window, phone (email and notes optional). Submitting posts a structured card to the concierge channel; the concierge does the match and texts the pet parent. The form is deliberately anonymous: the public page has no login, so the phone number is the match key and matching happens after submit rather than asking the pet parent to identify themselves. Everything is squad-operable: the page is static, the intake is a stock Teams workflow, and the matching is a human.

# Scope

**In scope:**

- The four live Pro services: Boarding, Daycare, Dog Walking, House Sitting (date ranges for Boarding and House Sitting, single date otherwise)
- Anonymous-only submission; phone required (Send Blue is the reply channel), email and notes optional
- SMS consent line at submit (wording pending legal review)
- Teams Adaptive Card intake with request ID, likely market from ZIP prefix, and a match-lookup reminder
- Delivery confirmation: pet parent sees success only after the request is accepted; failures show an inline retry message
- Analytics events for the M1 funnel (`request_form_viewed`, `service_selected`, `request_form_submitted`, `request_post_result`); wiring the Mixpanel token is a config change, no eng needed

**Out of scope (for the experiment):**

- Any engineering work: backend relay, automated lookup, yourgi.com hosting, native app surfaces
- Login, account creation, or prefill
- Automated Pro matching, real-time availability, pricing, or payment
- Grooming, Training, and Vet Care (not live on the Pro marketplace)
- Replacing the existing search-and-book funnel

## Marketplace check

> [!warning] Required — do not skip

- **Parent / Consumer side changes:** the new request form (web). No changes to existing search or booking flows.
- **Pro / Service Provider side changes:** none directly. Demand reaches Pros through the concierge using existing booking tools. Risk to watch: requests in ZIPs without available Pros create outreach the concierge cannot fulfill (see guardrail metric).
- **Internal ops / admin tooling changes:** Booking Form Teams channel and the "Booking Form intake (pro request form)" Power Automate flow (needs a co-owner). Concierge team takes on manual phone-number lookup and a text-back-within-the-hour expectation during business hours. Flow run history is the failure monitor.

# User stories

- **[Parent]** As a pet parent who doesn't have a person for this, I want to say what I need in five quick questions so that Yourgi finds the right Pro without me scrolling profiles.
- **[Parent]** As a returning pet parent, I want the concierge to already know me and my pets when they text so that I don't repeat myself.
- **[Internal]** As a concierge, I want every request to arrive as a structured card with a request ID, likely market, and contact info so that I can triage and respond within the hour.

# Requirements

### Parent web form (built, live)

- Collects ZIP (5 digits), service (one of four), pet count (1 to 6+), date or date range, time window, phone (10 digits, auto-formatted); email and notes optional, validated only when provided
- Rejects past dates and end-before-start ranges with field-specific, on-voice error messages
- Shows the SMS consent line adjacent to the submit button
- Confirms success only after the request is accepted upstream; on failure, keeps the pet parent on the form with a retry message
- Meets WCAG 2.1 AA (labels, focus management, contrast, keyboard operability) and follows Brand Guidelines V1.0 (National 2 type, palette, canonical service one-liners, no em dashes)

### Intake & ops (squad-operated)

- Teams card carries: request ID, service, ZIP + likely market, pets, dates, time window, phone, email, notes, match status
- Power Automate flow has at least one co-owner beyond the DRI
- Concierge response expectation ("texts back within the hour during business hours") signed off by the channel owner, or the copy softened
- Spam/abuse plan without eng: the webhook URL in the public link is a known, accepted exposure for the experiment. Mitigations: the concierge channel is human-monitored so garbage is visible immediately; if abused, Save As on the flow rotates the URL in minutes and the page link is updated. Optional squad-buildable hardening: a second Power Automate flow that accepts only the form's fields and builds the card itself, so a leaked URL can only produce form-shaped posts.

# Success metrics

**Primary:**

- Weekly `request_form_submitted` volume and the share of submissions that become booking requests (baselines TBD; these are the graduation inputs). Ladders to M1 Booking Conversion Rate (under 2% today, 4% target).

**Guardrail:**

- M2 Match Rate must not regress: requests from ZIPs without available Pros must stay a small share of submissions (threshold TBD), or we gate the form by SLA coverage.
- Concierge load: median first-response time stays within the promised hour during business hours.

# Open questions

- Decision criteria numbers: confirm the strawman volume and conversion thresholds above
- Legal: confirm or replace the SMS consent wording; confirm privacy policy linkage for phone/email collection
- Concierge staffing: who owns the Booking Form channel rota, and is within-the-hour sustainable if the public link drives real volume?
- Fonts: National 2 web license follows the serving domain; github.io hosting needs a marketing/brand check before the public link goes up
- Do we accept all ZIPs and let the concierge triage out-of-market demand, or add a soft out-of-market message?
- Overlap check with Theme 4 "Pro book for a customer" and Theme 6's embeddable widget: same design space; if this graduates, those teams are the first conversation
