# Yourgi Pro Request Form (prototype)

Internal prototype: a branded public form where pet parents request a Yourgi Pro service
(Boarding, Daycare, Dog Walking, House Sitting). Submissions post an Adaptive Card to the
concierge Teams channel; the concierge replies via Send Blue. KPI: M1 Booking Conversion Rate.

- Live form: https://jeff-winters-destpet.github.io/yourgi-pro-request-form/
- Wiring: see `docs/Teams wiring guide.md`

**Never commit a webhook URL to this repo.** Pass it at runtime instead:
`index.html?webhook=<power-automate-url>` (only `*.logic.azure.com` / `*.powerplatform.com` HTTPS URLs are accepted).

Every submitter is anonymous; the phone number is the match key, and the lookup against
pet parent data happens after submit.

## Wireframes

- [Concierge landing page (low-fi replica)](https://jeff-winters-destpet.github.io/yourgi-pro-request-form/wireframes/concierge-landing-page.html)
- [Pro request queue concept (interactive kanban)](https://jeff-winters-destpet.github.io/yourgi-pro-request-form/wireframes/pro-queue.html)
