# Pro Request Form: Teams wiring guide

Goal: every form submission posts an Adaptive Card into a concierge Teams channel, and the concierge replies to the pet parent via Send Blue.

Architecture note: the public page has no login, so every submitter is anonymous. The phone number is the match key. The match against existing pet parent data happens **after** submit: manually by the concierge during the pilot, by the backend in production.

KPI: **M1 Booking Conversion Rate** (see [[KPI Framework]]). The form is a search-request-to-booking-request surface; instrumentation events are listed at the bottom.

## One-time setup (about 10 minutes, you do this part)

### 1. Create the channel

1. In Teams, pick the team that should own concierge intake (or create one).
2. `...` next to the team name > **Add channel** > name it **Concierge Requests** > Standard channel > Create.

### 2. Add the webhook workflow

1. Hover the new channel > `...` > **Workflows**.
2. Search for **"Post to a channel when a webhook request is received"** and select it.
3. Confirm the team + channel, then **Add workflow**.
4. Copy the **URL** it shows you. Treat it like a password: anyone who has it can post into the channel.

### 3. Make the flow accept browser posts (one small edit)

Browsers cannot send cross-origin `application/json` posts to Power Automate (the CORS preflight fails), so the form sends the same JSON as `text/plain`. The flow needs one expression change to parse it:

1. Go to [make.powerautomate.com](https://make.powerautomate.com) > **My flows** > open the workflow you just created > **Edit**.
2. Open the **"Send each adaptive card"** step (an Apply to each).
3. Replace its input (currently the trigger's `attachments` dynamic content) with this expression:

```
json(string(triggerOutputs()?['body']))?['attachments']
```

4. **Save.**

This handles both browser posts (`text/plain`) and server posts (`application/json`), so the same flow works for testing and for the real form.

### 4. Wire the form

Either:

- Open the form with the webhook in the URL: `yourgi-pro-request.html?webhook=<paste-url-here>`, or
- Edit the file: near the bottom, in the `CONFIG` block, paste the URL into `webhookUrl`.

The status dot in the prototype bar turns green ("Live") when a webhook is configured.

### 5. Test

- Submit the form once. The card should land in **Concierge Requests** within a few seconds.
- Server-side test without the form (PowerShell):

```powershell
$url = "<webhook url>"
$body = Get-Content sample-payload.json -Raw
Invoke-RestMethod -Uri $url -Method Post -ContentType "application/json" -Body $body
```

## What the concierge sees

One Adaptive Card per request:

- **Request ID** (YRQ-XXXX) for referencing the request in Send Blue threads
- Service, ZIP + likely market (rough prefix match: Denver, Dallas, Fort Worth, Houston, Boston, Portland, or "Outside current markets")
- Pets (count)
- Date or drop-off/pick-up range, time window
- Phone (the Send Blue channel and the match key), email if provided
- **Match step**: the card reminds the concierge to look up the phone (and email) against existing pet parent data before texting, so known pet parents get greeted by name and their pets' names

## Known limits of the pilot wiring (be honest about these)

- **Fire and forget.** Because of the `no-cors` workaround, the browser cannot confirm delivery. The pet parent sees the confirmation screen as long as the request left their device. Fine for a pilot; not fine for production.
- **The webhook URL is a bearer secret.** Do not put the wired form anywhere public. For an internal pilot (concierge team testing, demo to stakeholders) it is fine.
- **The match is manual during the pilot.** The concierge looks up the phone number in our data by hand. Automating it (backend enriches the Teams card with the matched pet parent and pet names) needs an eng endpoint.
- **No spam protection.** A public form would need rate limiting and abuse controls on a real backend.
- **Flow ownership.** The workflow lives under your account. Add a co-owner in Power Automate so it survives org changes.

## Production path (for the eng feasibility conversation)

1. Replace the direct webhook with a Yourgi backend endpoint: it validates, runs the post-submit pet-parent lookup (phone first, email as fallback), enriches the Teams card with the matched account and pet names, posts to Teams server-side (proper `application/json`, delivery confirmation), and fires analytics.
2. Gate or flag ZIPs by SLA coverage (5+ available active Pros within 5-6 miles) instead of the prefix map.
3. Host on yourgi.com.
4. Mixpanel events (already stubbed in the form via `track()`; today they go to `dataLayer` + console):
   - `request_form_viewed` (props: webhookConfigured)
   - `service_selected` (props: service)
   - `request_form_submitted` (props: service, zip) -> **numerator input for M1**
   - `request_post_result` (props: ok)

## Copy provenance

All customer-facing copy was drafted in the Yourgi voice and passed through the [brand voice tool](https://brand-voice-tool-delta.vercel.app/); service one-liners are the canonical approved lines. Per the [[AI Pilot AUP]], review before anything ships to a customer surface. No em dashes anywhere in customer copy.
