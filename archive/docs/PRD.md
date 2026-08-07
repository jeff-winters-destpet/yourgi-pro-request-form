# Project Context: Concierge Landing Page ("Potemkin" Flat-Rate Test)
**Date:** 29 July 2026
**Source:** Working-session notes from the Concierge Landing Page meeting (Wed, 29 July 2026; attendees Chris, Jeff, Lauren, Ree). Structured meeting notes plus a partial verbal PRD; Chris to follow with written notes and a partial PRD.
**Version:** 1.0

---

## 1. Problem Statement
Yourgi doesn't yet know whether pet-care customers will accept a flat-rate, "we'll find the pro for you" concierge model instead of browsing and selecting a provider themselves. There's a parallel belief — surfacing in meet-and-greet feedback — that for pet care customers actually want a person, not automated matching. Validating either by building real platform integration is a multi-week engineering effort; the team wants a cheap answer first. Why now: the homepage is being given the match widget, and Chris wants a lightweight lane for the team to test this thesis directly, outside the engineering pipeline, at a cost of roughly half a day of vibe coding rather than a full build.

## 2. ICP (Ideal Customer Profile)
**Side:** Pet parents (primary, demand-side). Concierge staff are operationally central — they receive every request and create the booking by hand — so this is also a de facto internal-ops project.

Pet parents in covered markets who arrive from a Kai-run geo-targeted ad, need a pro service (boarding, daycare, house sitting, or dog walking), and want a fast, upfront flat price without browsing or account creation. Behaviorally they favor "quote first, someone will handle it for me" over evaluating providers themselves — the same posture that makes the Handy home-cleaning flow work.

**Cross-side impact:** Providers never touch this front end. Every request becomes a booking the concierge team creates manually on the back end, and because the customer pays a flat rate while Yourgi "eats the difference" against the pro's actual price, this introduces margin/pricing pressure Yourgi absorbs rather than the customer. The provider-side experience — which pros get selected, how they're notified, how a manually created booking appears to them, and how account-less customers get an account created (concierge may "masquerade" into it) — is only partially addressed in the input and is flagged in Gaps.

## 3. Pain Points
The input describes friction on two fronts. For the business: testing the concierge/flat-rate thesis today would require a multi-week engineering build, which is too expensive for a hypothesis that might be wrong. For the customer experience being replaced: Yourgi's existing offers page is seen as beatable (Chris's read is the team could build something better than both it and Handy's page), and there's recorded skepticism that automated matching suits pet care at all — the signal is that customers want a human. One concrete failure mode was named directly: a "we'll reach out" confirmation with no stated timeframe (the car-shipping analogy) is a bad experience, so vague follow-up promises are a pain point to design against.

## 4. Proposed Solution
- Users can land on a standalone page (e.g. `yourgi.com/find-me-a-sitter`), separate from the homepage, and state the service they want without picking a provider.
- Users can enter service type, dates, number of pets, location/city, and required phone and email (contact info) via form fields or buttons (team's call).
- Users can press a "Get a Price"–style button and see an instant flat-rate estimated total calculated from their inputs plus the 5% fee — before any account creation or provider browsing.
- Users can see a coupon disclaimer stating coupons aren't valid with this flat-rate offer, and light Yourgi Guarantee messaging (with layout space reserved for Kai's web devs to expand later).
- Users can see trust signals (guarantee, vetted pros, 24/7 support, review volume) below the fold rather than as a selection step.
- Users can submit the request and reach a confirmation screen — no live matching — that sets explicit response-time expectations and makes clear a human will reach out and connect them with a pro.
- Users submitting from an out-of-market location can receive a "thanks for the interest, we're not there yet" response (possibly saved as a lead).
- Staff (concierge) can receive the submitted request (destination undecided — see Open Questions), follow up by text via SendBlue, and create the booking manually on the back end.

## 5. Success Metrics
The stated test question is whether customers will accept the flat-rate, concierge-matched model — a demand-side validation. The input frames success qualitatively ("cheap answer" if disproven, "lean in" if proven) and puts the cost of being wrong at roughly half a day of vibe coding, but sets **no quantitative target, baseline, or metric definition** (e.g. submission rate, quote-to-submit conversion, quality of leads). Metric definition and baseline: Not defined in source material. Logged in Gaps.

## 6. Design Constraints
**Platform:** Web — a single standalone landing page.

**Geography:** No Geo-IP gating required. Copy names the covered areas and Kai runs geo-targeted campaigns; out-of-market submissions get a "we're not there yet" message and may be saved as leads. The specific launch markets to name in copy are Not defined in source material (logged in Gaps).

**Accessibility:** Not defined in source material (logged in Gaps).

**Technical:** Webflow is the default (not mandatory for a standalone page); easiest path is copying an existing landing page as a starting point. Vibe coding with Claude is encouraged. A footer with standard content is likely wanted. Concierge texts customers via SendBlue. Where form submissions land is the main open technical unknown — options raised are the SendBlue API, a Webflow form, Power Automate → a Teams channel (Jeff's instinct, simplest), or a shared concierge inbox; Kai decides. Downstream, Jeff owns how the booking actually gets created, including creating an account for account-less customers and concierge "masquerading" into it. Longer-term wish: move off the Webflow MCP (called janky; a 2.0 shipped but is untested) toward Storyblok so the site can be fully coded.

**Brand:** Follows the yourgi-brand skill. Must match the brand style guide; Lauren has Figma access.

**Trust & liability:** The page carries Yourgi Guarantee messaging — kept light for now with space reserved for Kai's web devs to expand. Because the Guarantee is Yourgi's central claim and its legal boundaries are undocumented, any guarantee copy needs review before it ships. The flat-rate model also requires a coupon disclaimer (coupons not valid with this offer). Note the model creates real bookings for real pets while bypassing the normal matching/selection path, so how the Guarantee and vetting apply to a concierge-created booking should be confirmed.

**Other:** Timeline is end of week / Friday. Standard footer content expected.

## 7. Open Questions
1. **Where do form submissions land, and who owns that pipe?** Options raised: SendBlue API, a Webflow form, Power Automate → Teams channel (Jeff's instinct), or a shared concierge inbox. Kai decides since he manages the concierge team; Jeff to determine — this blocks the build.
2. **Are displayed prices fee-inclusive, and which rate set is correct?** Either $50 / $40 / $20 (base) or $50 / $44 / $24 (base + 5% fee) — possibly a transcription artifact. Tied to whether the 5% fee is rounded into the displayed price. Confirm with Kai before building.
3. **What response-time commitment do we make to the customer?** Concierge is staffed through typical US business hours; realistic turnaround is an hour or two, but a 3 AM submission waits. Chris to confirm the SLA with Kai; copy must set the expectation explicitly.
4. **How does concierge create a booking for a customer with no account?** An account may need creating, then concierge masquerades into it. Jeff to think through the downstream flow.
5. **Do we keep out-of-market submissions as leads, or discard them?**
6. **Form fields or buttons for the inputs?** Explicitly left as the team's call.

## 8. Gaps
1. **Success Metrics** — no target, baseline, or metric definition for what "customers accept the model" means in numbers (submission volume, quote-to-submit conversion, lead quality). Matters because it determines what counts as the thesis being proven or disproven before Kai puts ad dollars behind it. Ask Chris / Kai / Scott, who scoped the concierge test.
2. **Design Constraints → Accessibility** — no requirements stated. Matters because a public, ad-driven landing page has accessibility exposure. Owner unknown; likely Kai's web team (Paul, Mike) or Lauren.
3. **Design Constraints → Geography (specific markets)** — copy is meant to name covered areas, but the areas aren't listed in the input. Matters because it blocks writing the location copy and the out-of-market message. Ask Kai (runs the geo-targeted campaigns).
4. **Cross-side / provider-side detail** — how pros are chosen for a request, how they're notified, and how a concierge-created (possibly masqueraded) booking appears to the provider is only partially addressed. Matters because the manual booking flow lands entirely on the provider side and on concierge ops. Ask Jeff (downstream flow) and Kai (concierge team).
5. **Confirmation-screen copy and Guarantee copy specifics** — the intent is set (human will reach out, set expectations, light guarantee) but exact wording and the legal boundaries of the Guarantee claim aren't defined. Matters because guarantee language needs review before publish. Ask Kai (brand/marketing) and follow the yourgi-brand skill.

---
*Generated by yourgi-project-context. Update as decisions are made and questions are resolved.*
