# Yourgi Pro Queue (concept)

Low-fi concept wireframes for the **Pro request queue** - the kanban board Pros and the
Yourgi Concierge team would work requests on. Nothing here is production and nothing
connects to a live system.

- **Index:** https://jeff-winters-destpet.github.io/yourgi-pro-request-form/
- **Pro queue board:** https://jeff-winters-destpet.github.io/yourgi-pro-request-form/wireframes/pro-queue.html

## The concept

A booking is created for the Pro the pet parent asked for. From there:

1. **Direct** - it sits on that Pro's board with a claim window.
2. **Declined or expired** - it runs through the **Yourgi Match tool** and fans out to the
   top 3 matching Pros' boards at once. First to accept gets it.
3. **Still unaccepted after 2 hours** - it lands in the Yourgi Concierge team's
   **Needs Rematched** column. An agent picks it up (which assigns it to them by name so
   two agents don't duplicate outreach), works the **Yourgi Match Widget** call list, and
   assigns the request directly to whichever Pro agrees.

Sign in as any of five users to see the same board from different angles: three Pros
(Colette Jackson, Alex Garcia, Priya Raman) and two Concierge agents (Tasha Boone,
Miguel Santos).

Two demo controls make the lifecycle watchable without waiting: **Simulate new booking**
and **Advance 1 hour**.

## Not part of this project

The **Booking Request Form** is a separate, production project owned elsewhere. It lives at
`lpyourgi.github.io/concierge-landing-page` and feeds the Booking Form Teams channel via
Power Automate. The Pro Queue concept starts further downstream, after a booking already
exists and is assigned to a Pro.

Our earlier work on that form - the demo build, the landing page wireframe, its PRD, test
plan and test results - is in [`archive/`](archive/) and is deprecated. See
[`archive/README.md`](archive/README.md).

## Open questions

Tracked on the project board (private). The significant one: this concept has Concierge
manually coordinating with Pros to rematch stalled requests, which conflicts with Kai's
Concierge Workflow Proposal - that document says Concierge should not chase Pros and
should instead offer the customer a free equal-or-lesser rebook. Needs a decision on
which model governs.
