# Archive - Booking Request Form (deprecated)

Everything in this folder belongs to the **Booking Request Form** project, which is
now separate from the Pro Queue concept in this repo.

## Why this is deprecated

The booking request form went to production and was **rebuilt by someone else**. That
production form lives at `lpyourgi.github.io/concierge-landing-page` and feeds the
**Booking Form** Teams channel via Power Automate. It continues to run independently.

The demo form here was our own build of the same idea. It is superseded and kept only
as a record. **Do not develop it further, and do not point anyone at it as a live tool.**

## What is in here

| File | What it was |
|---|---|
| `booking-form-demo.html` | Our demo flat-rate quote form. Fully decoupled - posts nowhere, no webhook capability. |
| `concierge-landing-page-wireframe.html` | Low-fi replica of the production landing page, used to communicate the concept. |
| `docs/PRD.md` | The July 29 "Concierge Landing Page (Potemkin Flat-Rate Test)" project context, from Chris's working session. |
| `docs/Test Plan.md` | Test plan for the flat-rate concierge form. |
| `docs/Test Results 2026-07-30.md` | Results of that test run against the production page: 42 checks, 34 pass, 3 fail. |

The test results still matter to whoever owns the production form - they document three
open launch blockers (no analytics on the page, an out-of-market screen that promises
lead capture it does not perform, and an unconfirmed rate table). Those are tracked on
the project board, not here.

## What replaced the link between them

The demo form used to feed the Pro Queue board directly through a browser-local bridge.
That wiring is **gone**. The two projects are independent:

- **Booking Request Form** (production, separately owned) -> Teams channel -> Concierge
- **Pro Queue** (this repo) -> starts once a booking is assigned to a Pro
