---
title: MVP Scope
tags:
  - spec
status: draft
last-updated: 2026-05-27
---

# MVP Scope

**Summary**: The full feature set for YachtBay's first release. Covers the core marketplace loop — owner lists yacht, renter finds and instant-books, transaction completes.

---

## Core Loop

```
Owner creates listing → Renter searches & compares → Renter instant-books → Payment processed → Owner receives payout
```

---

## Owner Features

### Account & Onboarding
- [ ] Sign up / log in (email + password)
- [ ] Owner profile (name, contact info, about)
- [ ] Stripe Connect Accounts v2 onboarding — triggered during first listing setup

### Yacht Listing
- [ ] Create a listing with:
  - Photos (multiple, reorderable)
  - Title and description
  - Yacht specs: type (sailboat, catamaran, motor yacht, etc.), length, capacity (guests), build year, manufacturer/model
  - Home port / location
  - Pricing: day rate, multi-day rate, weekly rate
  - Availability calendar (block dates, set open dates)
  - Crew options: select which configurations are available (bareboat / skippered / fully crewed) with separate pricing per option
  - Charter types: day trip, multi-day, weekly (owner enables/disables each)
- [ ] Edit existing listing
- [ ] Pause / unpublish listing

### Booking Management
- [ ] View incoming bookings (upcoming, past, cancelled)
- [ ] Email notifications: new booking received, booking cancelled
- [ ] Post-booking: renter contact details shared by email

### Payouts
- [ ] Payouts via Stripe (released 24–48h after charter start date)
- [ ] Stripe Express Dashboard accessible for earnings and payout history

---

## Renter Features

### Account
- [ ] Sign up / log in (email + password)
- [ ] Renter profile (name, contact info)

### Search & Discovery
- [ ] Search yachts by:
  - Location
  - Date range
  - Charter duration (day trip / multi-day / weekly)
  - Crew option (bareboat / skippered / fully crewed)
- [ ] Filter results by: yacht type, capacity, price range
- [ ] Sort results by: price, relevance
- [ ] Yacht listing card: photo, title, location, price, crew options available

### Yacht Detail Page
- [ ] Photo gallery
- [ ] Full specs (type, length, capacity, year, home port)
- [ ] Description
- [ ] Available crew options with pricing per option
- [ ] Availability calendar
- [ ] Pricing breakdown (day rate, weekly rate)
- [ ] Cancellation policy (platform standard)

### Booking
- [ ] Select dates + duration
- [ ] Select crew option
- [ ] Pricing summary before payment
- [ ] Instant book — pay via Stripe (card)
- [ ] Booking confirmation page + confirmation email
- [ ] Post-booking: owner contact details shared by email

### Booking Management
- [ ] View bookings (upcoming, past, cancelled)
- [ ] Cancel booking (platform cancellation policy applied automatically)
- [ ] Email notifications: booking confirmed, cancellation confirmed

---

## Platform & Admin

### Payments
- [ ] Stripe Connect Accounts v2 integration
- [ ] Destination charges with `application_fee_amount` (13% commission)
- [ ] Payout release 24–48h after charter start
- [ ] Cancellation refunds processed automatically per platform policy

### Cancellation Policy (Platform Default)
| Timing | Refund |
|---|---|
| 30+ days before charter | 100% |
| 14–29 days before charter | 50% |
| Under 14 days before charter | 0% |

> Exact thresholds to be confirmed. See [[decisions/2026-05-02-platform-cancellation-policy]].

### Email Notifications
- [ ] Booking confirmed (owner + renter)
- [ ] Booking cancelled (owner + renter)
- [ ] Payout released (owner)
- [ ] Booking reminder (renter, 48h before charter)

### Admin Panel (internal)
Reduced scope for soft launch — 4 endpoints only. Full admin panel deferred to post-traction.

- [ ] `GET /admin/users` — list/search users
- [ ] `PATCH /admin/users/:id/suspend` — emergency user suspension
- [ ] `GET /admin/bookings` — list all platform bookings
- [ ] `POST /admin/bookings/:id/refund` — issue manual refund

See [[decisions/2026-05-27-minimal-admin-mvp]].

---

## Explicitly Out of Scope (Phase 2)

| Feature | Reason deferred |
|---|---|
| Reviews & ratings | No completed bookings at launch — no value until traction |
| In-platform messaging | Nice to have — share contact details post-booking instead |
| Per-owner cancellation policies | Platform policy sufficient for MVP |
| Per-owner commission rates | Data model supports it, UI not needed yet |
| Calendar sync (iCal / Google) | Manual calendar management acceptable for MVP |
| Native mobile apps (iOS / Android) | Build after traction — responsive web covers mobile for now |
| Owner analytics dashboard | Basic booking list sufficient for MVP |
| Sailing licence verification (bareboat) | Deferred — owner responsible for verifying at MVP |
| Promotional / discount tools | Phase 2 |
| Multi-language support | English only at MVP |
| Multi-currency | EUR only at MVP |
| Full admin panel (revenue, settings, per-owner commission UI) | Soft launch needs only user suspension and manual refund — schema fields already in place |
| Transactional outbox | Schema table exists, preserved for future option; poller deferred until reliability evidence warrants ~1 day of work |
| Async email/host/audit queues | Emails sent synchronously via EmailPort (Resend) with try-catch. BullMQ used for payout scheduling only |

---

## Key Decisions Referenced

- [[decisions/2026-05-01-instant-booking]] — instant book, no owner approval step
- [[decisions/2026-05-01-stripe-connect-accounts-v2]] — Stripe Connect Accounts v2 for payments
- [[decisions/2026-05-01-commission-from-owners]] — 13% commission from owners
- [[decisions/2026-05-02-platform-cancellation-policy]] — platform-defined cancellation policy
- [[decisions/2026-05-02-web-app-mvp]] — web app only, mobile post-traction

## Related Pages

- [[wiki/concepts/business-model]]
- [[wiki/concepts/booking-model]]
- [[wiki/concepts/crew-options]]
- [[wiki/concepts/stripe-connect]]
- [[roadmap/overview]]
