---
title: Owner User Stories
tags:
  - spec
  - user-stories
  - owner
status: draft
last-updated: 2026-05-04
---

# Owner User Stories

**Persona**: The owner is a Greek yacht owner — individual with 1–3 boats or a small fleet operator — who wants to list their vessel(s) and earn revenue from charters with minimal admin overhead.

---

## Epic 1 — Authentication & Onboarding

---

### O-001 · Register as an owner
**Phase**: MVP

As a yacht owner, I want to create an owner account, so that I can list my yacht and start receiving bookings.

#### Acceptance Criteria
- [ ] Owner can register with email, password, first name, last name, phone number
- [ ] Alternatively, an existing renter account can be upgraded to include the `owner` role without creating a new account — "Become an owner" flow
- [ ] Email must be unique — duplicate registrations rejected with clear error
- [ ] Verification email sent after registration
- [ ] Owner cannot create listings until email is verified
- [ ] After verification, owner is redirected to the owner panel dashboard

#### Technical Requirements
**Entities**: `users` (role: owner | both)
**Endpoints**: `POST /api/v1/auth/register`, `POST /api/v1/users/me/add-owner-role`
**Business rules**:
- If existing renter account: add `owner` to `roles` array — no new account created
- Phone number required for owners (renters: optional) — needed for Stripe KYC and booking communication
**Notes**: Separate registration page on owner panel (`owners.yachtbay.com/register`). Same backend auth endpoints as storefront.

---

### O-002 · Login to owner panel
**Phase**: MVP

As an owner, I want to log in to the owner panel, so that I can manage my listings and bookings.

#### Acceptance Criteria
- [ ] Owner logs in with email and password on the owner panel login page
- [ ] If account does not have the `owner` role, access to the owner panel is denied with a clear message and a link to become an owner
- [ ] On success: access token + refresh token issued (same as storefront auth)
- [ ] httpOnly cookie set on `.yachtbay.com` domain — allows seamless access to both storefront and owner panel without re-login

#### Technical Requirements
**Endpoints**: `POST /api/v1/auth/login`
**Business rules**: `RolesGuard` on all owner panel routes — requires `owner` role in JWT payload
**Notes**: Cookie `domain: .yachtbay.com` enables shared auth between `yachtbay.com` and `owners.yachtbay.com`.

---

### O-003 · Complete Stripe Connect Accounts v2 onboarding
**Phase**: MVP

As an owner, I want to connect my bank account via Stripe, so that I can receive payouts automatically after each charter.

#### Acceptance Criteria
- [ ] Stripe onboarding is triggered when the owner attempts to publish their first listing
- [ ] Owner is redirected to Stripe's hosted Express onboarding flow
- [ ] After completing Stripe onboarding, owner is redirected back to the owner panel with a success message
- [ ] If Stripe onboarding is incomplete, owner can resume it from the dashboard
- [ ] Stripe account status displayed in dashboard: Pending / Active / Restricted
- [ ] Owner cannot receive payouts until status is Active

#### Technical Requirements
**Entities**: `users` (stripe_account_id, stripe_account_status)
**Endpoints**: `POST /api/v1/stripe/connect/onboard` (creates Account, returns onboarding URL), `GET /api/v1/stripe/connect/status`
**Business rules**:
- Stripe Account created with: `fees_collector: application`, `losses_collector: application`, `dashboard: express`
- `stripe_account_id` stored on `users` record after account creation
- `stripe_account_status` updated via Stripe webhook (`account.updated` event)
- Listing cannot be set to `active` if `stripe_account_status !== active`
**Events consumed**: Stripe `account.updated` webhook → update `users.stripe_account_status`
**Notes**: Stripe handles all KYC/identity verification. Owner never sees raw bank details in YachtBay's UI.

---

### O-004 · Switch to renter mode
**Phase**: MVP

As an owner who also wants to rent yachts, I want to switch to the renter-facing storefront, so that I can browse and book yachts without creating a separate account.

#### Acceptance Criteria
- [ ] "Browse as Renter" link visible in owner panel navigation
- [ ] Link opens the storefront (`yachtbay.com`) — auth cookie already valid, no re-login required
- [ ] Renter can switch back to owner panel via the storefront profile menu (R-022)

#### Technical Requirements
**Notes**: No backend logic needed — shared auth cookie handles this. Frontend navigation link only.

---

## Epic 2 — Listing Management

---

### O-005 · Create a new yacht listing
**Phase**: MVP

As an owner, I want to create a listing for my yacht, so that renters can find and book it on YachtBay.

#### Acceptance Criteria
- [ ] Owner can create a listing with the following required fields: yacht name, type (sailboat / catamaran / motor yacht), length (meters), guest capacity, build year, manufacturer, model, home port (text + map pin), description
- [ ] Listing saved as `draft` initially — not visible to renters until published
- [ ] Owner must complete Stripe onboarding (O-003) before publishing
- [ ] Listing creation is a multi-step form: Basic Details → Photos → Pricing → Crew Options → Review & Publish
- [ ] Owner can save progress and return to complete the listing later

#### Technical Requirements
**Entities**: `yachts`
**Endpoints**: `POST /api/v1/yachts` (creates draft), `PATCH /api/v1/yachts/:id` (updates), `PATCH /api/v1/yachts/:id/publish`
**Business rules**:
- `status: draft` on creation
- Cannot set `status: active` unless: all required fields complete + at least 3 photos + at least one pricing option + Stripe account active
- `owner_id` set to authenticated user's ID
**Notes**: Multi-step form saves progress via PATCH to draft listing after each step.

---

### O-006 · Add and manage listing photos
**Phase**: MVP

As an owner, I want to upload photos of my yacht, so that renters can see what they're booking.

#### Acceptance Criteria
- [ ] Owner can upload multiple photos (minimum 3 required to publish, maximum 20)
- [ ] Photos uploaded directly to Cloudflare R2 from the browser (pre-signed URL flow)
- [ ] Owner can reorder photos — first photo is the primary/cover image
- [ ] Owner can delete individual photos (minimum 3 must remain after deletion)
- [ ] Photos displayed in upload order on the listing page
- [ ] Accepted formats: JPG, PNG, WebP. Max size: 10MB per photo

#### Technical Requirements
**Entities**: `yacht_photos` (yacht_id, storage_key, display_order)
**Endpoints**: `POST /api/v1/yachts/:id/photos/upload-url` (returns pre-signed S3-compatible URL), `POST /api/v1/yachts/:id/photos` (register uploaded photo), `DELETE /api/v1/yachts/:id/photos/:photoId`, `PATCH /api/v1/yachts/:id/photos/reorder`
**Business rules**: Photos uploaded directly to R2 via pre-signed URL — never transit the backend server. Backend stores the R2 object key and Cloudflare CDN URL.
**Notes**: Resize/compress client-side before upload (browser Canvas API or a library like `browser-image-compression`) since R2 has no built-in transformation.

---

### O-007 · Configure listing pricing
**Phase**: MVP

As an owner, I want to set the price for my yacht, so that renters see accurate costs when browsing and booking.

#### Acceptance Criteria
- [ ] Owner sets a base day rate (€/day) for the yacht
- [ ] Owner optionally sets a weekly rate (€/week) — applied automatically when booking duration = 7 days
- [ ] Owner sets a separate price for each enabled crew option (bareboat / skippered / crewed)
- [ ] Prices entered in EUR, stored as integer cents
- [ ] Owner can see a preview of what renters will see (price breakdown example)

#### Technical Requirements
**Entities**: `yachts` (base_price_cents), `pricing_rules` (crew option rules)
**Endpoints**: `PATCH /api/v1/yachts/:id/pricing`
**Business rules**:
- Crew option prices stored as `pricing_rules` with `rule_type: crew_option`
- Weekly rate stored as `pricing_rules` with `rule_type: long_charter`
- PricingEngine applies rules in priority order at booking time
**Notes**: Platform commission (13%) is deducted at payout — the price owner sets is what YachtBay displays to renters. Owner earns price minus commission minus Stripe fees.

---

### O-008 · Configure crew options and charter types
**Phase**: MVP

As an owner, I want to specify which crew options and charter types my yacht supports, so that renters see accurate booking options.

#### Acceptance Criteria
- [ ] Owner selects which crew options are available: bareboat, skippered, fully crewed (at least one required)
- [ ] Owner selects which charter types are enabled: day trips, multi-day, weekly (at least one required)
- [ ] Owner sets minimum charter duration in days (e.g. 7 days minimum in summer)
- [ ] For bareboat: owner can add a note about licence requirements (displayed on listing)

#### Technical Requirements
**Entities**: `yachts` (crew_options jsonb, min_charter_days)
**Endpoints**: `PATCH /api/v1/yachts/:id/configuration`
**Business rules**: Minimum charter duration enforced at search and booking creation — not just display.

---

### O-009 · Edit an existing listing
**Phase**: MVP

As an owner, I want to edit my listing details, so that I can keep information accurate as things change.

#### Acceptance Criteria
- [ ] Owner can edit any listing field at any time (active or draft)
- [ ] Changes to active listings take effect immediately for future bookings
- [ ] Changes do not affect existing confirmed bookings (price snapshot stored on reservation)
- [ ] Owner receives a confirmation message after saving changes

#### Technical Requirements
**Endpoints**: `PATCH /api/v1/yachts/:id`
**Business rules**: `cancellation_policy_snapshot` and `total_price_cents` stored on reservation at booking time — listing changes do not retroactively affect confirmed bookings.

---

### O-010 · Pause or unpublish a listing
**Phase**: MVP

As an owner, I want to temporarily hide my listing, so that I can stop receiving new bookings without deleting the listing.

#### Acceptance Criteria
- [ ] Owner can set listing status to `paused` from the owner dashboard
- [ ] Paused listings are not visible in search results or accessible via direct URL
- [ ] Existing confirmed bookings are unaffected by pausing
- [ ] Owner can re-activate a paused listing at any time
- [ ] Owner can permanently delete a `draft` listing (listings with confirmed bookings cannot be deleted)

#### Technical Requirements
**Endpoints**: `PATCH /api/v1/yachts/:id/status`
**Business rules**:
- `status: paused` hides from public search — `GET /api/v1/yachts` excludes paused listings
- `status: active` listings with confirmed future bookings cannot be deleted — only paused
- Direct URL (`/yachts/:id`) returns 404 for paused listings

---

## Epic 3 — Availability Management

---

### O-011 · View availability calendar
**Phase**: MVP

As an owner, I want to see my yacht's availability calendar, so that I can understand which dates are booked, blocked, or available.

#### Acceptance Criteria
- [ ] Calendar shows current and next 6 months
- [ ] Dates colour-coded: Available (green), Reserved/booked (blue), Blocked by owner (grey), PREP days (light grey)
- [ ] Clicking a date shows its status and any associated booking reference
- [ ] Calendar updates automatically when new bookings are confirmed

#### Technical Requirements
**Entities**: `yacht_availability`
**Endpoints**: `GET /api/v1/yachts/:id/availability?from=...&to=...`
**Notes**: Owner sees all statuses including PREP. Renter-facing calendar only shows available/unavailable.

---

### O-012 · Block dates manually
**Phase**: MVP

As an owner, I want to block specific dates on my calendar, so that I can prevent bookings when my yacht is unavailable (maintenance, personal use, etc.).

#### Acceptance Criteria
- [ ] Owner can select one or more dates and set them to `blocked`
- [ ] Owner can add an optional internal note to blocked dates (not visible to renters)
- [ ] Blocked dates cannot be booked by renters
- [ ] Owner can unblock dates that are not reserved

#### Technical Requirements
**Entities**: `yacht_availability`
**Endpoints**: `POST /api/v1/yachts/:id/availability/block`, `DELETE /api/v1/yachts/:id/availability/block`
**Business rules**: Only `available` dates can be blocked — cannot block `reserved` dates. Cannot unblock `reserved` or `prep` dates.

---

### O-013 · Set preparation days between bookings
**Phase**: MVP

As an owner, I want to set how many preparation days I need between bookings, so that I have time to clean and prepare the yacht.

#### Acceptance Criteria
- [ ] Owner sets a number of PREP days (0–3) required after each confirmed booking
- [ ] PREP days are automatically blocked after each confirmed booking and released if the booking is cancelled
- [ ] PREP days are visible on the owner calendar but shown as unavailable to renters

#### Technical Requirements
**Entities**: `yachts` (prep_days field), `yacht_availability`
**Endpoints**: `PATCH /api/v1/yachts/:id/configuration`
**Business rules**: When `reservation.confirmed` event fires, booking handler blocks `check_out + 1` to `check_out + prep_days` with `status: prep`. Released when `reservation.cancelled`.

---

## Epic 4 — Booking Management

---

### O-014 · View incoming bookings
**Phase**: MVP

As an owner, I want to see all bookings for my yacht(s), so that I can track upcoming charters and plan accordingly.

#### Acceptance Criteria
- [ ] Bookings dashboard shows all bookings grouped by: Upcoming, Active (checked in), Completed, Cancelled
- [ ] Each booking card shows: renter first name, check-in/out dates, crew option, total amount (owner payout amount), booking status
- [ ] If owner has multiple listings, bookings are shown across all yachts with a filter by yacht

#### Technical Requirements
**Endpoints**: `GET /api/v1/bookings?ownerId=...`
**Business rules**: Returns only bookings for yachts owned by the authenticated owner. `owner_payout_cents` shown (not total — owner sees what they earn, not the commission).

---

### O-015 · View booking details
**Phase**: MVP

As an owner, I want to view the full details of a specific booking, so that I have all the information needed to prepare for the charter.

#### Acceptance Criteria
- [ ] Detail page shows: booking reference, renter first name + contact details (email, phone), check-in/out dates, crew option, number of guests, total owner payout, payment status
- [ ] For bareboat bookings: sailing licence details shown (if collected — Phase 2)
- [ ] Actions available based on booking status: Check In (upcoming), Complete (checked in), — (completed/cancelled)

#### Technical Requirements
**Endpoints**: `GET /api/v1/bookings/:id`
**Business rules**: Ownership check — owner can only view bookings for their own yachts.

---

### O-016 · Mark guest as checked in
**Phase**: MVP

As an owner, I want to mark a renter as checked in, so that the charter is officially underway and the system reflects this.

#### Acceptance Criteria
- [ ] "Check In" button visible on booking detail page for confirmed upcoming bookings
- [ ] Only available on or after the check-in date
- [ ] Booking status changes to `CHECKED_IN`
- [ ] Owner receives confirmation, renter is notified by email

#### Technical Requirements
**Endpoints**: `PATCH /api/v1/bookings/:id/check-in`
**Business rules**:
- Only allowed if `status = CONFIRMED` and current date >= `check_in`
- `version` check (optimistic concurrency)
**Events emitted**: `reservation.checked_in`

---

### O-017 · Mark charter as completed
**Phase**: MVP

As an owner, I want to mark a charter as completed, so that the payout is released and the charter is officially closed.

#### Acceptance Criteria
- [ ] "Mark Complete" button visible on booking detail for checked-in bookings
- [ ] Only available on or after the check-out date
- [ ] Booking status changes to `COMPLETED`
- [ ] Payout to owner released via Stripe (balance capture if applicable)
- [ ] Owner and renter receive completion notification emails
- [ ] Review request sent to renter (Phase 2)

#### Technical Requirements
**Endpoints**: `PATCH /api/v1/bookings/:id/complete`
**Business rules**:
- Only allowed if `status = CHECKED_IN` and current date >= `check_out`
- `version` check
- Payout release: Stripe transfer to owner's connected account triggered
**Events emitted**: `reservation.completed`
**Notes**: In practice, payout is scheduled for release 24–48h after check-in date automatically via `@nestjs/schedule` — this manual completion step is a secondary trigger and audit record.

---

### O-018 · Receive booking notifications
**Phase**: MVP

As an owner, I want to be notified when a new booking is confirmed or cancelled, so that I can plan accordingly without logging in constantly.

#### Acceptance Criteria
- [ ] Email notification sent within 60 seconds of booking confirmation
- [ ] Notification email contains: renter first name, charter dates, crew option, yacht name, owner payout amount, link to booking detail
- [ ] Email notification sent on booking cancellation with: dates released, refund info (if any)

#### Technical Requirements
**Events consumed**: `reservation.confirmed` → host-queue (Resend), `reservation.cancelled` → host-queue (Resend)
**Notes**: Separate email templates for new booking and cancellation.

---

## Epic 5 — Payouts & Earnings

---

### O-019 · View payout history
**Phase**: MVP

As an owner, I want to see a history of my payouts, so that I can track my earnings and reconcile them with my bookings.

#### Acceptance Criteria
- [ ] Payouts section in owner dashboard lists all past payouts
- [ ] Each payout shows: amount, booking reference, charter dates, payout date, status (pending / paid)
- [ ] Total earnings shown (month to date, all time)

#### Technical Requirements
**Entities**: `payments`
**Endpoints**: `GET /api/v1/payments?ownerId=...`
**Notes**: Shows `owner_amount_cents` per payout. Platform fee not shown to owner.

---

### O-020 · Access Stripe Express Dashboard
**Phase**: MVP

As an owner, I want to access my Stripe dashboard, so that I can see detailed payout information and manage my bank account details directly.

#### Acceptance Criteria
- [ ] "View Stripe Dashboard" link available in payouts section
- [ ] Link opens Stripe's hosted Express Dashboard in a new tab
- [ ] Dashboard shows: available balance, upcoming payouts, earnings history
- [ ] Owner can update bank account details directly in Stripe

#### Technical Requirements
**Endpoints**: `GET /api/v1/stripe/connect/dashboard-link` (generates Stripe Express Dashboard login link)
**Notes**: Stripe Express Dashboard is hosted by Stripe — no custom UI needed. YachtBay generates a login link via Stripe API.

---

## Epic 6 — Profile

---

### O-021 · View and edit owner profile
**Phase**: MVP

As an owner, I want to manage my profile information, so that renters see accurate information about who they're booking with.

#### Acceptance Criteria
- [ ] Profile shows: first name, last name, email (read-only), phone number, profile photo (optional), about/bio (optional — shown on listing page)
- [ ] Owner can update all fields except email
- [ ] Profile photo uploaded to Cloudflare R2 via pre-signed URL

#### Technical Requirements
**Entities**: `users`
**Endpoints**: `GET /api/v1/users/me`, `PATCH /api/v1/users/me`

---

### O-022 · View Stripe account status
**Phase**: MVP

As an owner, I want to see the status of my Stripe account in the dashboard, so that I know if there are any issues preventing me from receiving payouts.

#### Acceptance Criteria
- [ ] Dashboard header/sidebar shows Stripe account status: Active (green) / Pending (yellow) / Restricted (red)
- [ ] If Restricted: clear message explaining what action is needed (links to Stripe onboarding to resolve)
- [ ] If Pending: message indicating Stripe is still verifying the account

#### Technical Requirements
**Entities**: `users` (stripe_account_status)
**Notes**: Status updated via Stripe `account.updated` webhook — no polling needed.

---

## Epic 7 — Phase 2 Stories

---

### O-023 · Set custom seasonal pricing rules
**Phase**: Phase 2

As an owner, I want to set higher prices during peak season, so that I can maximise revenue when demand is highest.

#### Acceptance Criteria
- [ ] Owner can add pricing rules: select date range, set modifier (% surcharge or flat rate)
- [ ] Multiple rules can overlap — priority order determines which applies
- [ ] Rules applied automatically by PricingEngine at booking time
- [ ] Owner can preview the resulting price for any date range

#### Technical Requirements
**Entities**: `pricing_rules` (rule_type: seasonal, date_from, date_to, modifier_value, priority)
**Notes**: `SeasonalPricingStrategy` already defined in PricingEngine — just needs owner-facing UI and rule creation endpoint.

---

### O-024 · View analytics dashboard
**Phase**: Phase 2

As an owner, I want to see analytics about my listings and bookings, so that I can understand my performance and make better decisions.

#### Acceptance Criteria
- [ ] Dashboard shows: total bookings (month/year), total earnings, occupancy rate, top performing listing (if multiple)
- [ ] Booking trends chart (monthly)
- [ ] Average rating per listing (once reviews are enabled)

#### Technical Requirements
**Notes**: Dedicated analytics queries on `reservations` and `payments`. No external analytics service needed at Phase 2 scale.

---

### O-025 · Respond to renter reviews
**Phase**: Phase 2

As an owner, I want to respond to reviews left by renters, so that I can address feedback publicly and show responsiveness.

#### Acceptance Criteria
- [ ] Owner can leave a single response to each review
- [ ] Response is displayed below the renter's review on the listing page
- [ ] Response cannot be edited after submission

#### Technical Requirements
**Entities**: `reviews` (owner_response field)

---

### O-026 · Set owner-defined cancellation policy
**Phase**: Phase 2

As an owner, I want to choose my own cancellation policy, so that I can offer flexibility that suits my business model.

#### Acceptance Criteria
- [ ] Owner selects from: Flexible, Moderate (default), Strict
- [ ] Policy displayed on listing page
- [ ] Renter sees policy before booking and must acknowledge it
- [ ] Policy applied to all future bookings — existing bookings unaffected (snapshot stored on reservation)

#### Technical Requirements
**Entities**: `yachts` (cancellation_policy_type), `reservations` (cancellation_policy_snapshot)
**Notes**: `CancellationPolicyStrategy` already reads from `cancellation_policy_snapshot` — new policy types just need to be added. Data model already supports this (designed from day one).

---

## Story Count

| Phase | Count |
|---|---|
| MVP | 22 stories (O-001 → O-022) |
| Phase 2 | 4 stories (O-023 → O-026) |
| **Total** | **26 stories** |

## Related Pages

- [[specs/mvp-scope]]
- [[specs/backend-architecture]]
- [[specs/user-stories/renter]]
- [[specs/user-stories/admin]]
- [[wiki/concepts/user-personas]]
- [[wiki/concepts/commission-model]]
- [[decisions/2026-05-01-stripe-connect-accounts-v2]]
