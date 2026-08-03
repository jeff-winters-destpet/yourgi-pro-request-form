# Test Results: Find-a-Pro Flat-Rate Concierge Form

**Executed:** 2026-07-30, against https://lpyourgi.github.io/concierge-landing-page/ ("Best Care Guarantee" build)
**Method:** scripted browser execution (desktop Chromium engine + 375px viewport). Network stubbed for failure/gating tests; exactly **one real submission** sent at the end (Boarding, 80227, (720) 555-0100, concierge-test@example.com, $53) - that card is in Booking Form now and doubles as the E2 eyeball check.
**Plan:** [[Test Plan]]

## Verdict summary

**42 checks executed: 34 pass, 3 fail, 5 flagged decisions/observations. 9 checks pending (require humans, real devices, or infra access).**

Launch blockers found: **2** (analytics absent; out-of-market promise with no mechanism) plus **1 decision blocker** (rate table unconfirmed).

---

## A. Experiment intent - PASS (6/6)

| ID | Result | Evidence |
|----|--------|----------|
| A1 | PASS | Total ("$53") renders on load with zero contact fields filled; quote fully interactive before any personal data |
| A2 | PASS | No provider browsing anywhere; copy uses the canonical anti-browsing line ("You shouldn't have to scroll through hundreds of profiles...") |
| A3 | PASS | Confirmation states: "Usually within an hour, 8am-8pm MT. Sent overnight? You'll hear from us first thing." Explicit SLA, overnight case covered. (Minor: "Usually" is a hedge - see I4) |
| A4 | PASS | "A real person will text you to lock in your boarding (3 nights, 1 pet, about $158) and match you with the right pro." Human explicit, price echoed |
| A5 | PASS | Card and screen use the same computation; verified in code and via the live $53 submission - confirm on the card in Booking Form |
| A6 | PASS | In-market proceeds; out-of-market gets the OOM screen and posts nothing (0 network calls) |

## B. Quote engine - PASS with 1 decision flag, 1 bug

| ID | Result | Evidence |
|----|--------|----------|
| B1 | **DECISION NEEDED** | Implemented rates: Boarding $50/night, Daycare $40/day, House Sitting $50/night, Dog Walking $20/walk, +5% fee on top. Matches the meeting's base numbers; the fee-inclusive variant (PRD open question 2) is NOT what's built. Also: House Sitting at $50 is below the network median ($55), Daycare $40 above it ($35). Kai to confirm |
| B2 | PASS | Default 1 night x 1 pet = $52.50 -> displays $53 (round half up) |
| B3 | PASS | 3 nights x 2 pets x $50 + 5% = $315, breakdown correct |
| B4 | PASS | Same-day range counts as 1 unit. **BUG (cosmetic): "1 nights x 1 pet" - units never singularize** ("1 nights", "1 days", "1 walks") |
| B5 | PASS | Units per service render correctly (nights/days/walks). Note: daycare/walk quantity = length of the date range; confirm with concierge that "3 walks" means what they'll assume |
| B6 | PASS | Pets clamp at 1 and 8 |
| B7 | PASS | Service switch recomputes rate, unit, total instantly |
| B8 | PASS | Fractional totals round consistently on screen, in the confirmation echo, and in the card (same function) |
| B9 | PASS | 30 nights x 8 pets renders "$12,600" with separator |
| B10 | PASS | Month boundary (Aug 30-Sep 2) and year boundary (Dec 30-Jan 2) both = 3 nights, $158 |

## C. Validation - PASS (6/6)

| ID | Result | Evidence |
|----|--------|----------|
| C1 | PASS | 7205550100 auto-formats to (720) 555-0100 |
| C2 | PASS | Rejected: area code starting 1, exchange starting 1, 8-digit number. NANP rules enforced |
| C3 | PASS | ZIP strips non-digits, requires 5 |
| C4 | PASS | Email required and format-checked (per meeting spec) |
| C5 | PASS | Errors clear on correction, per field |
| C6 | PASS | All three errors display simultaneously; no post fires while invalid |

## D. Market gating - 2 findings

| ID | Result | Evidence |
|----|--------|----------|
| D1 | PASS | All six market ZIPs (80227, 75204, 76102, 77002, 02138, 97209) proceed, exactly one post each |
| D2 | PASS | 10001, 60601, 99501 -> OOM screen, zero posts |
| D3 | **DECISION NEEDED** | Confirmed: 77590 (Texas City) and 80512 (rural CO) are quoted and accepted as in-market by the prefix match. Fine if the concierge triages; decide and note |
| D4 | **FAIL (blocker)** | OOM screen says "Leave your info and we'll reach out the moment we land nearby" - but there is no input, no submit, and nothing is posted or saved. The copy promises follow-up that cannot happen. Either add lead capture (PRD open question 5) or change the copy |

## E. Delivery - PASS (executed portion)

| ID | Result | Evidence |
|----|--------|----------|
| E1 | PASS | Live submission: confirmation appeared only after the real 202. Card "Boarding in 80227 at flat rate $53" now in Booking Form |
| E2/E3 | PASS* | Card carries Service, Quoted-with-math, Dates, Pets, ZIP, Phone, Email, Match reminder, timestamp. *One-glance confirmation of the live card recommended |
| E4 | PASS | Forced 401: retry message shows, no confirmation, button re-enables. Exception path (network down) behaves the same |
| E5 | PASS | Triple-click during a slow send -> exactly one post |
| E6 | PENDING | Flow-off drill not run (didn't touch the production flow); run against a test flow copy |
| E7 | PARTIAL | FactSet values are plain text (no injection surface); exotic email content not exercised against the real channel |
| E8 | OBSERVATION | No request ID in this build's card (the prototype had YRQ-XXXX). Worth adding for Send Blue threading |

## F. Ops dry-run - PENDING (human)

Not executable by script. Schedule the table-top with the concierge team per the plan (F1-F7); the live $53 test card can serve as F1's trigger if used today.

## G. Analytics - **FAIL (launch blocker on this host)**

| ID | Result | Evidence |
|----|--------|----------|
| G1-G5 | **FAIL / BLOCKED** | This build contains no Mixpanel, no Klaviyo, no analytics of any kind. Quote-to-submit conversion - the graduation metric - cannot be measured. The Webflow variant has the Mixpanel/Klaviyo handler; port it (or equivalent) before ad traffic. Until then the experiment can count Teams cards but has no denominator |

## H. Devices - desktop + small viewport PASS, real devices pending

| ID | Result | Evidence |
|----|--------|----------|
| H3 | PASS | Full functional suite ran on desktop engine |
| H4 | PASS | 375px: no horizontal overflow, nothing clipped. Minor: stepper buttons are 39x38px, just under the 44px tap-target guideline |
| H1/H2/H5 | PENDING | Real iOS Safari and Android Chrome passes (the custom calendar is the risk), plus throttled-connection behavior |

## I. Brand, copy, legal - 1 pass set, findings to fix

| ID | Result | Evidence |
|----|--------|----------|
| I1 | PASS | Consent line present adjacent to submit: "By submitting, you agree Yourgi may text you about this request. Message and data rates may apply." Legal wording review still open |
| I2 | PASS | "Coupons don't apply to this flat-rate offer." present |
| I3 | PRESENT | Trust section carries the canonical Yourgi Guarantee copy; legal boundary review remains an open PRD item |
| I4 | **FINDINGS** | Three em dashes in customer copy (voice rule): "No scrolling, no surprises — tell us...", "Got it — we're on it.", "Yourgi Pros aren't in your area — yet." (Review attributions "— K. S" match the brand PDF's own style; fine.) Plus the "1 nights" grammar bug (B4) and the "Usually" hedge (A3) |
| I5 | PASS | OOM copy names Denver, Dallas, Fort Worth, Houston, Boston, Portland - matches the brand markets |
| I6 | **PARTIAL FAIL** | Good: labels on all fields, alt text everywhere, aria-pressed on service buttons, lang/viewport/charset set. Failing: error messages have no aria-live (screen readers won't hear them), and the custom calendar is mouse/touch-only (readonly input, click-only day cells) - not keyboard operable |

## J. Resilience - known-risk state confirmed

| ID | Result | Evidence |
|----|--------|----------|
| J1/J2 | PENDING | Rotation drill and garbage-post drill are human exercises; schedule once before ads |
| J3 | PASS (light) | Static page; reload resets state cleanly, restart buttons work. Full back/refresh matrix worth one manual pass |
| J4 | CONFIRMED | Webhook URL is in the page source - the accepted risk per the PRD. No other secrets found in this build (the Webflow variant carries Klaviyo/Mixpanel tokens; this one is clean) |

---

## The list that matters

**Blockers before ads:**
1. **No analytics on this build (G)** - the experiment cannot compute its own success criteria.
2. **OOM promise with no mechanism (D4)** - add lead capture or soften the copy.
3. **Rate table confirmation (B1)** - $50/$40/$50/$20 +5% is built; Kai must bless it (and the fee-inclusive question).

**Fix-fast (small, pre-ads ideally):**
4. "1 nights" singularization (one-line fix).
5. Three em dashes in customer copy.
6. aria-live on error messages; keyboard path for the calendar (or an accessible fallback).
7. D3 decision: accept prefix false-positives or tighten the ZIP list.
8. Consider adding a request ID to the card (E8).

**Pending human passes:** concierge ops dry-run (F), flow-off drill (E6), real iOS/Android (H1/H2), throttled network (H5), rotation + garbage drills (J1/J2), legal review of consent + guarantee wording.

**Test artifact to ignore:** one card in Booking Form from (720) 555-0100 / concierge-test@example.com, Boarding $53 - that was this test run.
