---
title: Renter User Stories
tags:
  - spec
  - user-stories
  - renter
status: draft
last-updated: 2026-05-04
---

# Renter User Stories

**Persona**: The renter is a traveler (primarily mid-range, international or local Greek) looking to discover, compare, and instantly book a yacht charter in Greece.

---

## Epic 1 — Authentication & Account

---

### R-001 · Register as a renter
**Phase**: MVP

As a renter, I want to create an account with my email and password, so that I can save my details and manage my bookings.

#### Acceptance Criteria
- [ ] Renter can register with email, password, first name, last name
- [ ] Email must be unique — duplicate registrations are rejected with a clear error
- [ ] Password must meet minimum requirements (8+ characters)
- [ ] Verification email is sent after registration
- [ ] Renter cannot book until email is verified
- [ ] After verification, renter is redirected to homepage or intended destination

#### Technical Requirements
**Entities**: `users` (role: renter)
**Endpoints**: `POST /api/v1/auth/register`
**Business rules**:
- Default role assigned: `renter`
- A user can later add `owner` role without creating a new account
- Password hashed with bcrypt before storage
**Events emitted**: None (email sent synchronously via Resend)
**Notes**: Email verification token stored with expiry (24h). If user registers again with same email before verifying, resend verification email.

---

### R-002 · Login
**Phase**: MVP

As a renter, I want to log in with my email and password, so that I can access my account and bookings.

#### Acceptance Criteria
- [ ] Renter can log in with email and password
- [ ] On success: access token (15min) + refresh token (7 days) issued, stored in httpOnly cookies
- [ ] On failure: generic error message (do not reveal whether email exists)
- [ ] Unverified email accounts are rejected with a prompt to verify

#### Technical Requirements
**Entities**: `users`
**Endpoints**: `POST /api/v1/auth/login`
**Business rules**:
- Access token: JWT, 15min expiry
- Refresh token: JWT, 7 days expiry, stored in httpOnly cookie
- Max 5 failed attempts per IP per 15 minutes (rate limiting)
**Notes**: Use `@nestjs/passport` + `@nestjs/jwt`. LocalStrategy for login, JwtStrategy for protected routes.

---

### R-003 · Logout
**Phase**: MVP

As a renter, I want to log out of my account, so that my session is terminated securely.

#### Acceptance Criteria
- [ ] Logout clears the access token and refresh token cookies
- [ ] Renter is redirected to homepage
- [ ] Attempting to use the cleared tokens returns 401

#### Technical Requirements
**Endpoints**: `POST /api/v1/auth/logout`
**Business rules**: Clear httpOnly cookies server-side. Optionally blacklist refresh token in Redis.

---

### R-004 · Reset password
**Phase**: MVP

As a renter, I want to reset my password if I forget it, so that I can regain access to my account.

#### Acceptance Criteria
- [ ] Renter can request a password reset by entering their email
- [ ] Reset email is sent with a secure time-limited link (1 hour)
- [ ] Link opens a form to enter and confirm a new password
- [ ] After successful reset, all active sessions are invalidated
- [ ] If email not found, response is identical (no email enumeration)

#### Technical Requirements
**Endpoints**: `POST /api/v1/auth/forgot-password`, `POST /api/v1/auth/reset-password`
**Business rules**: Token stored hashed in DB with expiry. Single use — invalidated after use.
**Notes**: Resend for email delivery.

---

## Epic 2 — Search & Discovery

---

### R-005 · Search yachts by location and dates
**Phase**: MVP

As a renter, I want to search for available yachts by location and dates, so that I can find options that match my travel plans.

#### Acceptance Criteria
- [ ] Search form accepts: location (text or map pick), check-in date, check-out date, number of guests
- [ ] Results show only yachts available for the full selected date range
- [ ] Results exclude yachts that don't meet minimum charter duration requirements
- [ ] Empty results show a clear message and suggestions
- [ ] Search is accessible from the homepage and persists in the URL (shareable links)

#### Technical Requirements
**Entities**: `yachts`, `yacht_availability`
**Endpoints**: `GET /api/v1/yachts?location=...&checkIn=...&checkOut=...&guests=...`
**Business rules**:
- Availability check: all dates in range must have `status = available` in `yacht_availability`
- Minimum charter duration enforced per yacht (`min_charter_days`)
- Geo search via PostGIS: `ST_DWithin(location, ST_Point(lng, lat), radius_meters)`
- Location input resolved to lat/lng via Mapbox Geocoding API
**Notes**: Results paginated (20 per page). URL params drive search state — enables shareable/bookmarkable search URLs.

---

### R-006 · Filter search results
**Phase**: MVP

As a renter, I want to filter search results by yacht type, crew option, and price range, so that I can narrow down options to what suits me.

#### Acceptance Criteria
- [ ] Filters available: yacht type (sailboat, catamaran, motor yacht), crew option (bareboat / skippered / crewed), price range (min–max per day), capacity
- [ ] Filters apply without a full page reload
- [ ] Active filters are clearly visible and individually dismissible
- [ ] Filter state persists in URL params

#### Technical Requirements
**Endpoints**: `GET /api/v1/yachts` (additional query params: `type`, `crewOption`, `minPrice`, `maxPrice`, `capacity`)
**Notes**: All filtering done server-side on PostgreSQL. No client-side filtering.

---

### R-007 · Sort search results
**Phase**: MVP

As a renter, I want to sort search results, so that I can find the best match quickly.

#### Acceptance Criteria
- [ ] Sort options: Price (low to high), Price (high to low), Relevance (default)
- [ ] Selected sort option persists in URL params

#### Technical Requirements
**Endpoints**: `GET /api/v1/yachts` (additional query param: `sortBy=price_asc|price_desc|relevance`)
**Notes**: Relevance sort = distance from searched location + availability completeness. No ML needed at MVP.

---

### R-008 · View yachts on a map
**Phase**: MVP

As a renter, I want to see yacht home ports on a map, so that I can understand where yachts are based relative to my destination.

#### Acceptance Criteria
- [ ] Search results page shows a map alongside the results list (split view)
- [ ] Each yacht is represented by a pin on the map at its home port location
- [ ] Clicking a pin highlights the corresponding listing card and shows a mini preview
- [ ] Map updates when filters change
- [ ] Map is powered by Mapbox

#### Technical Requirements
**Entities**: `yachts` (location_lat, location_lng)
**Notes**: Use `react-map-gl` (Mapbox React SDK). Pins use custom yacht icon. Mini preview on pin click shows photo, name, price — links to detail page.

---

### R-009 · View search result listing cards
**Phase**: MVP

As a renter, I want to see key information for each yacht in the search results, so that I can quickly compare options without opening each one.

#### Acceptance Criteria
- [ ] Each listing card shows: primary photo, yacht name, type, home port, capacity, available crew options, price per day, average rating (Phase 2 — show "New" badge if no reviews)
- [ ] Card links to the yacht detail page
- [ ] Price shown reflects selected dates if dates were entered in search

#### Technical Requirements
**Endpoints**: `GET /api/v1/yachts` returns all fields needed for listing cards
**Notes**: Price shown = lowest available crew option price for the selected dates + duration.

---

## Epic 3 — Yacht Detail

---

### R-010 · View yacht detail page
**Phase**: MVP

As a renter, I want to view the full details of a yacht, so that I can decide whether it meets my needs before booking.

#### Acceptance Criteria
- [ ] Page shows: photo gallery (multiple photos, fullscreen), yacht name, type, length, capacity, build year, manufacturer/model, home port, description
- [ ] Available crew options with pricing for each
- [ ] Cancellation policy clearly displayed
- [ ] Owner info (first name, response info — no contact details until booked)
- [ ] Page is server-side rendered and crawlable by search engines (SEO)
- [ ] Page URL format: `/yachts/[id]-[slug]`

#### Technical Requirements
**Entities**: `yachts`, `users` (owner)
**Endpoints**: `GET /api/v1/yachts/:id`
**Business rules**: Only active (`status = active`) listings are publicly accessible. Draft/paused listings return 404.
**Notes**: SSR via React Router 7 loader. Meta tags (title, description, OG image) populated from yacht data for SEO and social sharing.

---

### R-011 · View yacht availability calendar
**Phase**: MVP

As a renter, I want to see which dates are available for a yacht, so that I can plan my booking around the yacht's availability.

#### Acceptance Criteria
- [ ] Calendar shows current and next 3 months by default
- [ ] Available dates shown in green, reserved/blocked in grey, PREP days in light grey
- [ ] Renter cannot select unavailable dates in the booking form
- [ ] Calendar updates in real time when dates are selected

#### Technical Requirements
**Entities**: `yacht_availability`
**Endpoints**: `GET /api/v1/yachts/:id/availability?from=...&to=...`
**Notes**: Returns array of dates with status. Calendar rendered client-side from this data.

---

### R-012 · View price for selected dates and crew option
**Phase**: MVP

As a renter, I want to see the price for my specific dates and chosen crew option, so that I know the exact cost before committing to a booking.

#### Acceptance Criteria
- [ ] Price updates dynamically as renter selects dates and crew option
- [ ] Price breakdown shown: base rate × days, crew fee (if applicable), total
- [ ] Weekly rate applied automatically if duration = 7 days
- [ ] Platform commission is NOT shown to renter (included in owner price)
- [ ] Cancellation policy for the booking shown alongside price

#### Technical Requirements
**Endpoints**: `GET /api/v1/yachts/:id/pricing?checkIn=...&checkOut=...&crewOption=...`
**Business rules**: PricingEngine calculates price server-side. Returns breakdown per line item.
**Notes**: Price calculation is always server-side — client never calculates price.

---

## Epic 4 — Booking Flow

---

### R-013 · Select booking parameters
**Phase**: MVP

As a renter, I want to select my check-in/check-out dates and crew option, so that I can configure my booking before paying.

#### Acceptance Criteria
- [ ] Renter selects check-in and check-out dates from an availability-aware date picker
- [ ] Renter selects crew option from available options for that yacht
- [ ] Unavailable dates are disabled in the date picker
- [ ] Minimum charter duration is enforced — invalid date ranges show an error
- [ ] Selection can be changed before confirming

#### Technical Requirements
**Entities**: `yacht_availability`, `yachts`
**Notes**: Date picker uses availability data fetched from `GET /api/v1/yachts/:id/availability`. Validation also happens server-side at booking creation.

---

### R-014 · View booking summary and price breakdown
**Phase**: MVP

As a renter, I want to see a clear summary of my booking before paying, so that I can confirm all details are correct.

#### Acceptance Criteria
- [ ] Summary shows: yacht name + photo, check-in/check-out dates, duration, crew option, price breakdown (base rate, crew fee, total), cancellation policy
- [ ] "Confirm and Pay" button proceeds to payment
- [ ] Renter can go back and change parameters without losing their selection

#### Technical Requirements
**Notes**: Summary page populated from `GET /api/v1/yachts/:id/pricing` response. No DB writes at this stage — booking is only created on payment confirmation.

---

### R-015 · Instant book — complete payment
**Phase**: MVP

As a renter, I want to pay for my booking instantly, so that my charter is confirmed without waiting for owner approval.

#### Acceptance Criteria
- [ ] Renter enters card details on a Stripe-hosted or Stripe Elements form
- [ ] Payment is processed immediately on submission
- [ ] If payment fails, a clear error is shown and renter can retry
- [ ] On success, booking is confirmed instantly — no owner approval required
- [ ] Idempotency key prevents duplicate bookings from network retries
- [ ] Renter is redirected to booking confirmation page

#### Technical Requirements
**Entities**: `reservations`, `payments`, `yacht_availability`, `outbox_events`
**Endpoints**: `POST /api/v1/bookings`
**Business rules**:
- `SELECT FOR UPDATE` on `yacht_availability` rows — prevents double booking
- `application_fee_amount` = 13% of total (platform commission via Stripe Connect)
- `transfer_data.destination` = owner's `stripe_account_id`
- Booking created with `status = PENDING`, immediately auto-confirmed to `CONFIRMED` (instant book)
- Availability rows updated to `status = reserved`
- PREP days blocked automatically after confirmed dates
- Outbox event written in same transaction
**Events emitted**: `reservation.created` → `reservation.confirmed`
**Notes**: Idempotency key = UUID generated client-side, sent as header. Stripe PaymentIntent created server-side, client confirms with Stripe Elements.

---

### R-016 · Receive booking confirmation
**Phase**: MVP

As a renter, I want to receive a booking confirmation with all details, so that I have a record of my charter and know what to expect.

#### Acceptance Criteria
- [ ] Confirmation page shown immediately after successful payment
- [ ] Confirmation email sent to renter within 60 seconds
- [ ] Email contains: booking reference, yacht name + photo, dates, crew option, total paid, owner first name, owner contact details (email/phone), cancellation policy reminder
- [ ] Booking confirmation page accessible later via renter dashboard

#### Technical Requirements
**Entities**: `reservations`, `users` (owner)
**Events consumed**: `reservation.confirmed` → email-queue (Resend)
**Notes**: Owner contact details shared only after confirmed booking — not visible on listing page.

---

## Epic 5 — Booking Management

---

### R-017 · View all my bookings
**Phase**: MVP

As a renter, I want to see all my bookings in one place, so that I can track my upcoming and past charters.

#### Acceptance Criteria
- [ ] Renter dashboard shows bookings grouped by: Upcoming, Past, Cancelled
- [ ] Each booking card shows: yacht photo, name, dates, status, total paid
- [ ] Links to full booking detail page

#### Technical Requirements
**Endpoints**: `GET /api/v1/bookings?renterId=...`
**Notes**: Behind `JwtAuthGuard`. Returns only bookings belonging to the authenticated renter.

---

### R-018 · View booking details
**Phase**: MVP

As a renter, I want to view the full details of a specific booking, so that I have all the information I need for my charter.

#### Acceptance Criteria
- [ ] Detail page shows: booking reference, yacht details (name, type, home port), dates, crew option, price breakdown, payment status, owner contact details, cancellation policy and deadline
- [ ] For upcoming bookings: cancellation option visible if within policy window

#### Technical Requirements
**Endpoints**: `GET /api/v1/bookings/:id`
**Business rules**: Ownership check — renter can only view their own bookings.

---

## Epic 6 — Cancellation

---

### R-019 · Cancel a booking
**Phase**: MVP

As a renter, I want to cancel a booking, so that I can get a refund if my plans change.

#### Acceptance Criteria
- [ ] Cancel button visible on booking detail page for upcoming bookings
- [ ] Before cancelling, renter sees: applicable refund amount based on cancellation policy + days remaining, and must confirm
- [ ] On confirmation, booking status changes to `CANCELLED`
- [ ] Refund processed automatically via Stripe
- [ ] Availability released back to `available` on cancelled dates (PREP days also released)
- [ ] Cancellation confirmation email sent to renter
- [ ] Owner notified of cancellation by email
- [ ] Platform commission is **not** refunded

#### Technical Requirements
**Entities**: `reservations`, `payments`, `yacht_availability`
**Endpoints**: `PATCH /api/v1/bookings/:id/cancel`
**Business rules**:
- Refund calculated by `CancellationPolicyStrategy`:
  - Day trips: 5+ days = 100%, 1–4 days = 50%, <24h = 0%
  - Weekly: 30+ days = 100%, 14–29 days = 50%, <14 days = 0%
- Platform commission (`platform_fee_cents`) never refunded
- Stripe refund issued for renter-eligible amount only
- `version` check on reservation (optimistic concurrency)
**Events emitted**: `reservation.cancelled`
**Notes**: Cancellation policy snapshot stored on reservation at booking time — cancellation uses that snapshot, not the current platform policy.

---

### R-020 · Receive cancellation confirmation
**Phase**: MVP

As a renter, I want to receive confirmation of my cancellation with refund details, so that I know when to expect my money back.

#### Acceptance Criteria
- [ ] Cancellation confirmation email sent within 60 seconds
- [ ] Email contains: booking reference, cancelled dates, refund amount, expected timeline (5–10 business days for card refunds)
- [ ] Booking status updated to `CANCELLED` in renter dashboard immediately

#### Technical Requirements
**Events consumed**: `reservation.cancelled` → email-queue (Resend)
**Notes**: Stripe refund processing time is outside YachtBay's control — email should clarify this.

---

## Epic 7 — Profile

---

### R-021 · View and edit profile
**Phase**: MVP

As a renter, I want to view and update my personal details, so that my account information stays accurate.

#### Acceptance Criteria
- [ ] Profile page shows: first name, last name, email (read-only — email changes require re-verification), phone number (optional)
- [ ] Renter can update first name, last name, phone number
- [ ] Changes saved with confirmation message

#### Technical Requirements
**Entities**: `users`
**Endpoints**: `GET /api/v1/users/me`, `PATCH /api/v1/users/me`
**Notes**: Email changes not supported at MVP — add as Phase 2 with re-verification flow.

---

### R-022 · Switch to owner mode
**Phase**: MVP

As a renter who also owns a yacht, I want to access the owner panel from my account, so that I can manage both roles without creating a separate account.

#### Acceptance Criteria
- [ ] If renter's account has `role` that includes `owner`, a "Switch to Owner Dashboard" link is visible in the nav/profile menu
- [ ] Link opens the owner panel (separate app) in a new tab or navigates to it
- [ ] Authentication is shared — no re-login required

#### Technical Requirements
**Business rules**: JWT token includes `roles` array. Owner panel reads this and grants access if `owner` role present.
**Notes**: Auth token is shared via httpOnly cookie across subdomains (e.g. `yachtbay.com` and `owners.yachtbay.com`). Cookie `domain` set to `.yachtbay.com`.

---

## Epic 8 — Phase 2 Stories

---

### R-023 · Leave a review after completed charter
**Phase**: Phase 2

As a renter, I want to leave a review after my charter, so that I can share my experience and help other renters make informed decisions.

#### Acceptance Criteria
- [ ] Review prompt sent by email 24h after charter completion
- [ ] Renter rates the yacht (1–5 stars) and leaves a written review
- [ ] Review is visible on the yacht listing page after submission
- [ ] Owner can respond to reviews

#### Technical Requirements
**Entities**: `reviews` (new table)
**Events consumed**: `reservation.completed` → review-request email
**Notes**: Triggered by `ReservationCompleted` event — no changes to reservation module.

---

### R-024 · Message owner through platform
**Phase**: Phase 2

As a renter, I want to message the owner through the platform, so that I can ask questions before or after booking.

#### Acceptance Criteria
- [ ] Messaging available on listing page (pre-booking) and booking detail page (post-booking)
- [ ] Messages are threaded per booking/inquiry
- [ ] Both parties receive email notifications for new messages
- [ ] Message history persists in renter and owner dashboards

#### Technical Requirements
**Entities**: `conversations`, `messages` (new tables)
**Notes**: Isolated `MessagingModule`. WebSocket support via `@nestjs/websockets` + Socket.io. Does not touch booking module.

---

### R-025 · Save a yacht to wishlist
**Phase**: Phase 2

As a renter, I want to save yachts I'm interested in, so that I can compare them later without losing track.

#### Acceptance Criteria
- [ ] "Save" button on listing card and detail page
- [ ] Saved yachts accessible in renter dashboard under "Saved"
- [ ] Wishlist persists across sessions

#### Technical Requirements
**Entities**: `wishlists` (new table: user_id, yacht_id)
**Notes**: Simple save/unsave toggle. No complex logic.

---

### R-026 · Re-book a previous charter
**Phase**: Phase 2

As a renter, I want to quickly re-book a yacht I've chartered before, so that I can repeat an experience I enjoyed without searching again.

#### Acceptance Criteria
- [ ] "Book again" button on completed booking detail page
- [ ] Pre-populates the booking form with the same yacht and crew option
- [ ] Renter selects new dates

#### Technical Requirements
**Notes**: Shortcut to the booking flow with yacht ID + crew option pre-filled. No new backend logic.

---

## Story Count

| Phase | Count |
|---|---|
| MVP | 22 stories (R-001 → R-022) |
| Phase 2 | 4 stories (R-023 → R-026) |
| **Total** | **26 stories** |

## Related Pages

- [[specs/mvp-scope]]
- [[specs/backend-architecture]]
- [[specs/user-stories/owner]]
- [[specs/user-stories/admin]]
- [[wiki/concepts/user-personas]]
- [[wiki/concepts/booking-model]]
