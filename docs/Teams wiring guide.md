# Pro Request Form: Teams wiring guide

Goal: every form submission posts an Adaptive Card into the **Booking Form** Teams channel, and the concierge replies to the pet parent via Send Blue.

KPI: **M1 Booking Conversion Rate** (see [[KPI Framework]]). The form is a search-request-to-booking-request surface; instrumentation events are listed at the bottom.

Architecture: the public page has no login, so every submitter is anonymous. The phone number is the match key. The match against existing pet parent data happens **after** submit: manually by the concierge during the pilot, by the backend in production.

## One-time setup (done 2026-07-30)

1. Create the channel (ours: **Booking Form**).
2. Channel `...` > **Workflows** > pick the webhook template. Template names vary by tenant; ours was **"Send webhook alerts"** (becomes "Send webhook alerts to Booking Form"). Any template built on the trigger **"When a Teams webhook request is received"** works.
3. Copy the workflow's HTTP URL from the trigger step. Treat it like a password: anyone who has it can post into the channel.
4. **Do not edit the flow.** The stock template works as-is (leave "Do Not Remove FlowIL" alone). The `*.powerplatform.com` trigger endpoint answers CORS preflight, so the form posts `application/json` straight from the browser.
5. Wire the form: open it as `.../?webhook=<url>` (or paste the URL into `CONFIG.webhookUrl` in the file). The status dot turns green ("Live: posting to Booking Form").
6. Test: submit the form; the card lands in Booking Form within seconds. Server-side test without the form:

```powershell
$url = "<webhook url>"
Invoke-RestMethod -Uri $url -Method Post -ContentType "application/json" -Body (Get-Content sample-payload.json -Raw)
```

Verified 2026-07-30: `application/json` returns HTTP 202 and posts the card. Note the endpoint **rejects** `text/plain` with HTTP 400, so any client must send real JSON.

## What the concierge sees

One Adaptive Card per request:

- **Request ID** (YRQ-XXXX) for referencing the request in Send Blue threads
- Service, ZIP + likely market (rough prefix match: Denver, Dallas, Fort Worth, Houston, Boston, Portland, or "Outside current markets")
- Pets (count; "6+" means six or more)
- Date or drop-off/pick-up range, time window
- Phone (the Send Blue channel and the match key), email if provided
- **Match step**: the card reminds the concierge to look up the phone (and email) against existing pet parent data before texting, so known pet parents get greeted by name and their pets' names

## Delivery semantics

The form treats HTTP 202 from Power Automate as delivered and only then shows the confirmation screen; on any failure the pet parent sees "That didn't send. Give it one more try." and the form stays put. 202 means Power Automate *accepted* the request; a failure inside the flow itself would only show in the flow's run history, so spot-check that during the pilot.

## Known limits of the pilot wiring

- **The webhook URL is a bearer secret.** Never commit it; pass it via `?webhook=`. The form only accepts HTTPS URLs on `*.logic.azure.com` / `*.powerplatform.com`, which blocks payload-exfiltration links.
- **The match is manual during the pilot.** The concierge looks up the phone number in our data by hand. Automating it (backend enriches the Teams card with the matched pet parent and pet names) needs an eng endpoint.
- **No spam protection.** A public form would need rate limiting and abuse controls on a real backend.
- **Flow ownership.** The workflow lives under Jeff's account. Add a co-owner in Power Automate so it survives org changes.

## Production path (for the eng feasibility conversation)

1. Replace the direct webhook with a Yourgi backend endpoint: it validates, runs the post-submit pet-parent lookup (phone first, email as fallback), enriches the Teams card with the matched account and pet names, posts to Teams server-side, and fires analytics.
2. Gate or flag ZIPs by SLA coverage (5+ available active Pros within 5-6 miles) instead of the prefix map.
3. Host on yourgi.com.
4. Mixpanel events (already stubbed in the form via `track()`; today they go to `dataLayer` + console):
   - `request_form_viewed` (props: webhookConfigured)
   - `service_selected` (props: service)
   - `request_form_submitted` (props: service, zip) -> **numerator input for M1**
   - `request_post_result` (props: ok, status)

## Copy provenance

All customer-facing copy was drafted in the Yourgi voice and passed through the [brand voice tool](https://brand-voice-tool-delta.vercel.app/); service one-liners are the canonical approved lines. Per the [[AI Pilot AUP]], review before anything ships to a customer surface. No em dashes anywhere in customer copy.
