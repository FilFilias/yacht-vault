---
title: Admin User Stories
tags:
  - spec
  - user-stories
  - admin
status: draft
last-updated: 2026-05-04
---

# Admin User Stories

**Persona**: The admin is an internal YachtBay team member (initially the founder) who manages the platform — monitoring activity, resolving issues, managing users and listings, and tracking revenue.

> [!note]
> The admin panel is an internal tool. It does not need to be polished or SEO-optimised. Functionality over aesthetics. Access is restricted to users with `admin` role.

---

## Epic 1 — Authentication

---

### A-001 · Login to admin panel
**Phase**: MVP

As an admin, I want to log in to the admin panel with my credentials, so that I can access platform management tools.

#### Acceptance Criteria
- [ ] Admin logs in with email and password
- [ ] Only accounts with `admin` role can access the admin panel — all other roles are rejected with a 403
- [ ] Admin session uses the same JWT auth system as the rest of the platform
- [ ] No self-registration — admin accounts are created manually (seeded or via CLI command)

#### Technical Requirements
**Endpoints**: `POST /api/v1/auth/login`
**Business rules**: `RolesGuard` requires `admin` role on all admin routes. Admin panel served on a separate subdomain (`admin.yachtbay.com`) or restricted route.
**Notes**: Admin accounts created via NestJS seed script or a protected `POST /api/v1/admin/users` endpoint. Not publicly accessible.

---

## Epic 2 — User Management

---

### A-002 · View all users
**Phase**: MVP

As an admin, I want to see a list of all registered users, so that I can monitor platform growth and investigate issues.

#### Acceptance Criteria
- [ ] Table shows all users: name, email, role(s), registration date, status (active / suspended)
- [ ] Searchable by name or email
- [ ] Filterable by role (renter / owner / both / admin)
- [ ] Sortable by registration date

#### Technical Requirements
**Entities**: `users`
**Endpoints**: `GET /api/v1/admin/users`
**Notes**: Paginated (50 per page). Password hashes never returned.

---

### A-003 · View user details
**Phase**: MVP

As an admin, I want to view a specific user's full profile and activity, so that I can investigate issues or verify information.

#### Acceptance Criteria
- [ ] User detail page shows: personal details, role(s), registration date, Stripe account status (if owner), bookings as renter, listings as owner, total spend (renter) / total earned (owner)

#### Technical Requirements
**Endpoints**: `GET /api/v1/admin/users/:id`
**Notes**: Aggregated data from `reservations` and `payments` tables.

---

### A-004 · Suspend or reactivate a user
**Phase**: MVP

As an admin, I want to suspend a user account, so that I can prevent bad actors from using the platform.

#### Acceptance Criteria
- [ ] Admin can set a user's status to `suspended`
- [ ] Suspended users cannot log in — receive a clear message on login attempt
- [ ] Suspended owners' listings are automatically paused
- [ ] Admin can reactivate a suspended account
- [ ] Suspension reason recorded internally (not shown to user)

#### Technical Requirements
**Entities**: `users` (status: active | suspended)
**Endpoints**: `PATCH /api/v1/admin/users/:id/suspend`, `PATCH /api/v1/admin/users/:id/reactivate`
**Business rules**: On suspension, all active listings set to `paused`. On reactivation, listings remain paused — owner must re-activate manually.

---

## Epic 3 — Listing Management

---

### A-005 · View all listings
**Phase**: MVP

As an admin, I want to see all yacht listings on the platform, so that I can monitor inventory and flag issues.

#### Acceptance Criteria
- [ ] Table shows all listings: yacht name, owner name, type, home port, status (draft / active / paused), creation date, total bookings
- [ ] Filterable by status
- [ ] Searchable by yacht name or owner name

#### Technical Requirements
**Endpoints**: `GET /api/v1/admin/yachts`

---

### A-006 · View listing details
**Phase**: MVP

As an admin, I want to view the full details of any listing, so that I can review it for quality or policy compliance.

#### Acceptance Criteria
- [ ] Shows all listing fields: photos, specs, pricing, crew options, availability, booking history
- [ ] Shows owner details with link to their user profile

#### Technical Requirements
**Endpoints**: `GET /api/v1/admin/yachts/:id`

---

### A-007 · Pause or remove a listing
**Phase**: MVP

As an admin, I want to pause or remove any listing, so that I can take action on listings that violate platform policies.

#### Acceptance Criteria
- [ ] Admin can set any listing to `paused` regardless of owner action
- [ ] Admin can permanently delete listings with no confirmed future bookings
- [ ] Owner notified by email when their listing is paused or removed by admin
- [ ] Reason recorded internally

#### Technical Requirements
**Endpoints**: `PATCH /api/v1/admin/yachts/:id/status`, `DELETE /api/v1/admin/yachts/:id`
**Business rules**: Cannot delete listings with `CONFIRMED` or `CHECKED_IN` bookings.

---

## Epic 4 — Booking Management

---

### A-008 · View all bookings
**Phase**: MVP

As an admin, I want to see all bookings on the platform, so that I can monitor activity and investigate issues.

#### Acceptance Criteria
- [ ] Table shows all bookings: booking reference, yacht name, renter name, owner name, dates, status, total amount, platform commission earned
- [ ] Filterable by status and date range
- [ ] Searchable by booking reference, renter name, yacht name

#### Technical Requirements
**Endpoints**: `GET /api/v1/admin/bookings`
**Notes**: Shows `platform_fee_cents` per booking — this is the admin's primary revenue view.

---

### A-009 · View booking details
**Phase**: MVP

As an admin, I want to view the full details of any booking, so that I can investigate disputes or payment issues.

#### Acceptance Criteria
- [ ] Full booking detail: all reservation fields, payment records, reservation event log, renter and owner contact details
- [ ] Payment status from Stripe visible (captured / pending / refunded)
- [ ] Reservation events log shows full state history

#### Technical Requirements
**Endpoints**: `GET /api/v1/admin/bookings/:id`
**Notes**: `reservation_events` table provides the full audit trail.

---

### A-010 · Manually trigger a payout
**Phase**: MVP

As an admin, I want to manually trigger a payout to an owner, so that I can resolve cases where the automatic payout failed.

#### Acceptance Criteria
- [ ] Admin can trigger a manual Stripe transfer for a specific booking
- [ ] Only available for bookings with `status = COMPLETED` and `payment status = captured` but payout not yet sent
- [ ] Action logged in `reservation_events` with `admin_id` noted
- [ ] Admin receives confirmation of successful transfer

#### Technical Requirements
**Endpoints**: `POST /api/v1/admin/bookings/:id/trigger-payout`
**Business rules**: Idempotency check — cannot trigger payout if transfer already exists for this booking.
**Notes**: Uses Stripe `transfers.create` with the owner's `stripe_account_id`.

---

### A-011 · Issue a manual refund
**Phase**: MVP

As an admin, I want to issue a refund outside of the standard cancellation policy, so that I can resolve exceptional cases (e.g. owner no-show, platform error).

#### Acceptance Criteria
- [ ] Admin can issue a full or partial refund for any booking
- [ ] Refund amount cannot exceed the total amount paid by the renter
- [ ] Reason required — recorded in `reservation_events`
- [ ] Renter notified by email of the refund and reason
- [ ] Owner notified if payout is clawed back as a result

#### Technical Requirements
**Endpoints**: `POST /api/v1/admin/bookings/:id/refund`
**Business rules**: Stripe refund issued via `refunds.create`. If payout already sent to owner, a separate debit or clawback may be needed — flag this edge case for manual review.
**Notes**: Full clawback from owner is a complex Stripe operation — at MVP, flag for manual resolution rather than automating.

---

## Epic 5 — Revenue & Commission

---

### A-012 · View platform revenue summary
**Phase**: MVP

As an admin, I want to see how much commission YachtBay has earned, so that I can track business performance.

#### Acceptance Criteria
- [ ] Revenue dashboard shows: total commission earned (all time), commission this month, commission this year
- [ ] Breakdown by month (simple chart or table)
- [ ] Number of completed bookings vs cancelled bookings
- [ ] Average booking value and average commission per booking

#### Technical Requirements
**Endpoints**: `GET /api/v1/admin/revenue`
**Notes**: Aggregated queries on `payments` table (`platform_fee_cents` summed by month). No external analytics service needed at MVP.

---

### A-013 · Set or update owner commission rate
**Phase**: MVP

As an admin, I want to set a custom commission rate for specific owners, so that I can offer volume discounts to fleet owners without changing the platform default.

#### Acceptance Criteria
- [ ] Admin can set a custom `commission_rate` on any owner's user record
- [ ] If set, this rate overrides the platform default (13%) for all future bookings by that owner
- [ ] Existing confirmed bookings are unaffected
- [ ] Commission rate visible on the owner's admin user detail page

#### Technical Requirements
**Entities**: `users` (commission_rate field — nullable, platform default used if null)
**Endpoints**: `PATCH /api/v1/admin/users/:id/commission`
**Business rules**: `CommissionStrategy` reads `users.commission_rate` first — falls back to platform default if null. This is already designed into the architecture.

---

## Epic 6 — Phase 2 Stories

---

### A-014 · View and moderate reviews
**Phase**: Phase 2

As an admin, I want to review and moderate user-submitted reviews, so that I can remove inappropriate content.

#### Acceptance Criteria
- [ ] Admin can view all reviews with renter name, yacht, rating, content, submission date
- [ ] Admin can hide or delete a review with a recorded reason
- [ ] Owner is notified if their listing's review is removed

#### Technical Requirements
**Entities**: `reviews` (status: visible | hidden)

---

### A-015 · Manage promotions and campaigns
**Phase**: Phase 2

As an admin, I want to create platform-wide promotions, so that I can drive bookings during low seasons.

#### Acceptance Criteria
- [ ] Admin can create a promotion: discount code, percentage or fixed amount discount, valid date range, usage limit
- [ ] Promotions applied at checkout by renter entering a code
- [ ] Admin can view usage stats per promotion

#### Technical Requirements
**Entities**: `promotions` (new table)
**Notes**: `PromotionModule` + `PromotionalStrategy` in PricingEngine — no changes to existing code.

---

### A-016 · Manage featured/promoted listings
**Phase**: Phase 2

As an admin, I want to mark listings as featured, so that they appear at the top of search results.

#### Acceptance Criteria
- [ ] Admin can set any active listing as `featured`
- [ ] Featured listings appear at top of search results with a visual badge
- [ ] Admin can set a featured period (start/end date)

#### Technical Requirements
**Entities**: `yachts` (is_featured, featured_until)
**Notes**: `RankingStrategy` in Search module prioritises featured listings. Owner-paid featured listings = Phase 2 revenue stream.

---

## Story Count

| Phase | Count |
|---|---|
| MVP | 13 stories (A-001 → A-013) |
| Phase 2 | 3 stories (A-014 → A-016) |
| **Total** | **16 stories** |

## Related Pages

- [[specs/mvp-scope]]
- [[specs/backend-architecture]]
- [[specs/user-stories/renter]]
- [[specs/user-stories/owner]]
- [[wiki/concepts/commission-model]]
