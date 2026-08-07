# Test Plan: Find-a-Pro Flat-Rate Concierge Form

**Pages under test:** https://lpyourgi.github.io/concierge-landing-page/ (current build, "Best Care Guarantee"), https://yourgi-pet-a5d957.webflow.io/find-a-pro (Webflow variant), and the production URL once mapped (the card attribution says `yourgi.com/book/best-care-guarantee`)
**Date:** 2026-07-30 · **Owner:** Jeff Winters · **Status:** Draft
**Reference:** [[PRD]] (Concierge Landing Page "Potemkin" Flat-Rate Test), [[Teams wiring guide]]

**Build-state notes (verified 2026-07-30):** the lpyourgi.github.io build already carries the fixed submit code (Adaptive Card envelope, `application/json`, delivery-gated confirmation with retry message), the SMS consent line, the coupon disclaimer, and a trust section - so E1/E4, I1, and I2 start from a passing baseline there. The Webflow variant still has the broken text/plain submit unless the fix has been pasted. **The lpyourgi build has no Mixpanel at all** - if it is the ad destination, section G is a launch blocker until instrumentation is added (the Webflow variant has the Mixpanel/Klaviyo handler; this one does not).

## What we are actually testing

The experiment's intent, per the PRD: *will pet parents accept a flat-rate, "we'll find the pro for you" concierge model instead of browsing providers themselves?* The form is the instrument that measures this. So the test plan has two jobs:

1. **The instrument works** - quotes are right, submissions arrive, the ops loop closes.
2. **The instrument measures what we think it measures** - a submission genuinely means "a customer accepted a flat-rate quote," and the funnel data can answer the graduation criteria. A broken quote, a silent delivery failure, or missing analytics doesn't just annoy a customer; it corrupts the experiment before Kai puts ad dollars behind it.

Every section below traces back to one of those two jobs.

## Prerequisites and test hygiene

- **Do not pollute the experiment.** Before ad traffic starts, testing on the live webhook is fine (concierge team forewarned). After launch, create a **Save As copy of the flow pointed at a test channel**, and run test passes against the Webflow staging URL wired to that test webhook.
- **Test identity convention:** all test submissions use phone numbers in the (XXX) 555-01XX range and emails at example.com, so concierge and analytics can filter them.
- Confirm with Kai **before** the delivery tests: the official rate table (PRD open question 2: $50/$40/$20 base vs. fee-inclusive) and the launch-market ZIP list. Section B and D cannot pass or fail without them.

---

## A. Experiment intent checks (the page tests the thesis)

| ID | Check | Expected |
|----|-------|----------|
| A1 | Price appears before any contact info is requested | Quote (service, dates, pets, total) is visible and interactive with zero personal fields filled |
| A2 | No provider browsing anywhere in the flow | No Pro profiles, photos, or selection step; copy says a human finds the Pro |
| A3 | Confirmation sets an explicit response-time expectation | Named timeframe (per Chris/Kai's SLA answer), not "we'll reach out" (the car-shipping failure mode named in the PRD) |
| A4 | Confirmation makes the human explicit | Copy says a real person will text, and echoes the quoted price so the flat-rate acceptance is unambiguous |
| A5 | The quote is the same number the concierge sees | Displayed total on screen == "Quoted" value in the Teams card, to the dollar, on the same submission |
| A6 | Out-of-market visitors get the "we're not there yet" experience | In-market ZIP proceeds; out-of-market ZIP gets the OOM screen, never the confirmation |

## B. Quote engine correctness

Rates in the page live in the service buttons' `data-r` attributes; quantity is derived from the selected date range; total = rate x units x pets x 1.05, rounded.

| ID | Case | Expected |
|----|------|----------|
| B1 | Rate table matches Kai's confirmed numbers, per service | data-r values == official rates; fee handling (added 5% vs fee-inclusive) matches the decision |
| B2 | Default state (today-today, 1 pet, boarding) | Total = rate x 1 x 1 x 1.05; breakdown line matches |
| B3 | Multi-night: Aug 10 - Aug 13, 2 pets, boarding $50 | qty=3 nights, total = 50x3x2x1.05 = $315; breakdown "3 nights x 2 pets x $50 + 5% service fee" |
| B4 | Same-day range (start == end) | qty=1, not 0; sensible for daycare/walks; decide whether "1 night" boarding same-day is acceptable copy |
| B5 | Unit semantics per service | Boarding/house sitting count nights; daycare/walking count days/walks. Verify a 3-day range means what the concierge will assume it means for each service |
| B6 | Pets stepper bounds | Min 1, max 8; buttons visibly disable or no-op at bounds; price updates each step |
| B7 | Service switch recomputes | Changing service mid-flow updates rate, unit label, and total immediately |
| B8 | Rounding | Totals that produce fractional cents (e.g. $35 x 3 x 1 x 1.05 = $110.25) round consistently on screen, in the confirmation echo, and in the Teams card |
| B9 | Thousands formatting | A large quote (e.g. 30 nights x 8 pets) renders with separators everywhere it appears |
| B10 | Calendar edges | Past days unpickable; range spanning a month/year boundary computes correctly; picking end before start restarts the range as the code intends |

## C. Input validation and error UX

| ID | Case | Expected |
|----|------|----------|
| C1 | Phone: valid NANP formats ((720) 555-0134, 7205550134, 1-720-555-0134) | Accepted and auto-formatted |
| C2 | Phone: invalid (9 digits, area code starting 0/1, letters) | Inline error, cannot proceed |
| C3 | ZIP: 5 digits required; letters and short entries rejected | Inline error, cannot proceed |
| C4 | Email: required and format-checked (this form requires it, per the meeting spec) | Invalid or empty email blocks with inline error |
| C5 | Errors clear on correction | Fixing a field hides its error without a page interaction elsewhere |
| C6 | All three fields invalid at once | All three errors show together; first invalid field gets focus |

## D. Market gating

Page prefixes: `['80','75','76','77','021','022','970','971','972']`. Verify against Kai's official launch ZIP list - the prefix list is coarse.

| ID | Case | Expected |
|----|------|----------|
| D1 | One in-market ZIP per market (80227 Denver, 75204 Dallas, 76102 Fort Worth, 77002 Houston, 02138 Boston, 97209 Portland) | Proceeds to submit |
| D2 | Out-of-market ZIPs (10001 NYC, 60601 Chicago, 99501 Anchorage) | OOM screen; nothing posts to the concierge channel |
| D3 | Prefix false positives | ZIPs that match a prefix but are outside the actual market (e.g. 77590 Texas City vs Houston metro; 80512 rural CO) - decide whether "close enough" is acceptable for the pilot and note it |
| D4 | OOM lead capture | Whatever the answer to PRD open question 5 is (save as lead vs discard), verify the behavior matches the decision - today the OOM path posts nothing anywhere |

## E. Delivery and the Teams card

| ID | Case | Expected |
|----|------|----------|
| E1 | Happy path submit | Card lands in Booking Form within seconds; confirmation shows only after delivery (202) |
| E2 | Card completeness | Service, **Quoted with math**, dates, pets, ZIP, phone, email, match reminder, timestamp - everything the concierge needs to text back without asking basics |
| E3 | Quote fidelity (repeat of A5, at the data level) | Card's Quoted == on-screen total for boarding, daycare, walking, sitting each |
| E4 | Delivery failure | Temporarily break the webhook URL (staging copy): user sees the retry message, no confirmation, button re-enables; nothing false-positive |
| E5 | Double-click / rapid resubmit | One card per submission; button disabled during the send |
| E6 | Flow-off drill | Turn the (test) flow off, submit: user gets the failure path, not a silent success. Documents what an outage looks like |
| E7 | Special characters | Name-less form, but email/ZIP/phone with edge content (quotes, unicode in email local part) neither breaks the card nor injects formatting |
| E8 | Duplicate detection aid | Two identical submissions minutes apart are distinguishable in the channel (timestamps; consider whether a request ID is worth adding - the Webflow payload currently has none) |

## F. Ops loop dry-run (end-to-end with a human)

Run once as a table-top with the concierge team before ads start, using a scripted persona.

| ID | Step | Expected |
|----|------|----------|
| F1 | Card arrives, concierge acknowledges | Within the SLA Chris confirms (target: the timeframe promised on the confirmation screen) |
| F2 | Match lookup | Concierge checks the phone (and email) against pet parent data before texting; known pet parent is greeted by name |
| F3 | SendBlue first text | References the service, dates, and the quoted flat rate; the customer never has to repeat what the form captured |
| F4 | Account-less customer | Concierge creates the account and masquerades in (PRD open question 4 flow); document each manual step and its duration - this is the graduation cost data |
| F5 | Booking created at the flat rate | Booking exists in the platform; the margin delta (flat rate vs Pro's actual price) is recorded somewhere Finance can see |
| F6 | Off-hours submission | A 9 PM submission gets picked up next business morning within expectations; confirmation copy doesn't overpromise |
| F7 | Unfulfillable request | In-market ZIP but no available Pro: what does the concierge tell the customer, and where does that outcome get logged? (Feeds the M2 guardrail) |

## G. Analytics and decision-criteria measurability

The graduation decision needs quote-to-submit conversion and submission volume. Verify the data exists before traffic starts.

| ID | Check | Expected |
|----|-------|----------|
| G1 | Page-view and quote-engagement events fire | Mixpanel receives distinct events for landing, quote interaction (service/date/pets change), and submit; if only page-view exists, flag the gap now |
| G2 | Submit event carries the decision inputs | Service, total, ZIP (or market), in/out-of-market at minimum |
| G3 | Attribution survives | UTM/gclid from a Kai-style ad URL lands on the Mixpanel profile and the Klaviyo profile (the page's Klaviyo handler stitches via mixpanel_distinct_id - verify with one real ad-parameter visit) |
| G4 | Test-data filtering | The 555-01XX convention is filterable in Mixpanel and known to whoever reads the numbers |
| G5 | Denominator sanity | Sessions on the page (Mixpanel) roughly match Webflow/ad-platform clicks for the same window |

## H. Devices and browsers

Ad traffic will skew mobile. Minimum matrix, full flow each:

| ID | Environment | Notes |
|----|-------------|-------|
| H1 | iOS Safari (current) | The custom calendar is the risk: tap targets, month nav, range selection, no double-fire on touch |
| H2 | Android Chrome (current) | Same calendar pass; keyboard type for phone/ZIP (numeric) |
| H3 | Desktop Chrome + Edge | Baseline |
| H4 | Small viewport (360px) | Quote card, steppers, and error messages don't clip or overlap |
| H5 | Slow connection (throttled) | Submit spinner/disabled state visible; no double-submit while waiting |

## I. Brand, copy, and legal

| ID | Check | Expected |
|----|-------|----------|
| I1 | SMS consent line present | Explicit consent copy adjacent to submit. Present on the lpyourgi build (verify placement and legibility); **verify on whichever host actually receives ad traffic**. Blocks launch if missing |
| I2 | Coupon disclaimer visible | "Coupons don't apply to this flat-rate offer" per the PRD |
| I3 | Guarantee copy reviewed | Light guarantee messaging only, and the wording cleared per the PRD's trust & liability constraint |
| I4 | Brand compliance | V1.0 guidelines: palette, National 2 type, canonical service one-liners if used, no em dashes in customer copy |
| I5 | Covered-markets copy | Named markets match Kai's list; OOM message tone is on-voice |
| I6 | Accessibility pass | Labels on all fields, keyboard-operable calendar and steppers, visible focus, error announcements, contrast (PRD logs a11y as a gap - this is the minimum bar for an ad-driven public page) |

## J. Resilience and abuse (accepted-risk verification)

| ID | Check | Expected |
|----|-------|----------|
| J1 | Webhook rotation drill | Save As the flow, swap the URL in Webflow, publish, verify - time the drill; this is the incident response for URL abuse and should be under 15 minutes |
| J2 | Garbage post visibility | Post junk to the webhook directly; confirm the concierge team recognizes and ignores it, and knows who to tell (triggers J1) |
| J3 | Refresh / back-button mid-flow | No duplicate submission; no broken state |
| J4 | The published page contains the webhook URL | Known and accepted per the PRD; confirm nobody has added anything *else* sensitive to the page source |

## Exit criteria

Launch (ads on) requires: all of A, B, C, D, E green; F dry-run completed with SLA signed off; G1-G4 green; H1-H2 green; I1-I3 green. J is verified once and documented.

**Known accepted risks (not test failures):** webhook URL exposed in page source with rotation as the response; manual concierge matching; coarse ZIP-prefix market gating (pending D3 decision).

## Logging

Track failures as: ID, environment, steps, expected vs actual, screenshot, severity (blocks-launch / fix-fast / note). Keep the log in this project folder next to this plan.
