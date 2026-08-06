# Yourgi Pro Request Form (concept demo)

Concept demo for the flat-rate concierge experiment: a branded "Get a Price" request form
and a Pro request queue board, wired together as a same-origin demo.

**Decoupled from production (2026-08-06).** This repo does not interface with the production
webhook, Power Automate flow, or Teams channel in any way. Form submissions here feed only
the queue board demo in your own browser (localStorage); nothing is sent anywhere.

- Form: https://jeff-winters-destpet.github.io/yourgi-pro-request-form/
- Pro queue board: https://jeff-winters-destpet.github.io/yourgi-pro-request-form/wireframes/pro-queue.html
- Landing page wireframe: https://jeff-winters-destpet.github.io/yourgi-pro-request-form/wireframes/concierge-landing-page.html

Demo script: open the form and the board in two tabs of the same browser, submit a request,
and watch it land in New Requests with a mock push notification. Claim it as one Pro, switch
identities, and see the card locked. Accepting reveals contact info and writes the mock booking.

Docs in `docs/`: the PRD of record (July 29 project context), test plan, and test results.
The production wiring guide lives with the production project, not here.
