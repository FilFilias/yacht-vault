---
title: Site Map — Admin Panel
tags:
  - spec
  - design
  - sitemap
  - admin
status: draft
last-updated: 2026-05-05
---

# Site Map — Admin Panel (admin.yachtbay.com)

**App**: Internal platform management tool — users, listings, bookings, revenue, commission.
**Framework**: React Router 7 (SPA mode)
**Auth**: Separate login, `admin` role required on all routes
**Design principle**: Functionality over aesthetics. Data density over visual polish. Tables, filters, actions.

> [!warning]
> The admin panel should never be publicly linked or discoverable. Access restricted to accounts with `admin` role. No self-registration — accounts created via seed script or CLI.

---

## Page Hierarchy

```
admin.yachtbay.com/
│
├── /dashboard ...................... Platform overview
│
├── /users .......................... All users
│   └── /users/:id .................. User detail
│
├── /listings ....................... All listings
│   └── /listings/:id ............... Listing detail
│
├── /bookings ....................... All bookings
│   └── /bookings/:id ............... Booking detail
│
├── /revenue ........................ Revenue & commission dashboard
│
├── /settings ....................... Platform settings
│
└── /login .......................... Admin login
```

---

## Pages

### Dashboard

---

#### `/dashboard` — Platform Overview
**Auth**: Protected (admin)
**Stories**: A-012

The admin's home screen. High-level health check of the platform at a glance.

**Key sections:**
- Stats row (today / this month / all time):
  - Total bookings
  - Total commission earned (€)
  - New users registered
  - Active listings
- Alerts panel (items requiring manual action):
  - Owners with Stripe account status `restricted` or `pending` > 48h
  - Bookings with `COMPLETED` status but payout not triggered
  - Failed payout jobs (from BullMQ)
- Recent bookings table (last 10): reference, renter, yacht, dates, status, commission
- Recent registrations (last 10 users): name, email, role, date

**Notes**: Alerts panel is the most important section — surfaces exceptions that need admin intervention before they become problems.

---

### Users

---

#### `/users` — All Users
**Auth**: Protected
**Stories**: A-002

Full user directory.

**Key sections:**
- Search bar (by name or email)
- Filter by role (All / Renter / Owner / Both / Admin)
- Filter by status (Active / Suspended)
- Sort by registration date (newest first by default)
- Users table:

| Column | Content |
|---|---|
| Name | First + last name |
| Email | Email address |
| Role(s) | Renter / Owner / Both / Admin badges |
| Registered | Date |
| Status | Active / Suspended badge |
| Actions | "View" link |

- Pagination (50 per page)
- Export to CSV (Phase 2)

---

#### `/users/:id` — User Detail
**Auth**: Protected
**Stories**: A-003, A-004, A-013

Full profile and activity for a specific user.

**Key sections:**
- **Profile header**: name, email, phone, role(s), registration date, status badge
- **Owner section** (visible if role includes `owner`):
  - Stripe account status + `stripe_account_id`
  - Custom commission rate (editable inline — overrides platform default)
  - Number of active listings (link to `/listings?ownerId=...`)
  - Total earnings (sum of `owner_amount_cents` across completed payments)
- **Renter section** (visible if role includes `renter`):
  - Total bookings made
  - Total spend (sum of `total_price_cents` across confirmed bookings)
  - Link to all bookings by this renter
- **Activity log**: recent bookings (as renter or owner), registrations, suspensions
- **Actions**:
  - Suspend account (with required reason field) → confirmation modal
  - Reactivate account (if suspended)
  - Set custom commission rate (owner only) → inline edit with save confirmation

---

### Listings

---

#### `/listings` — All Listings
**Auth**: Protected
**Stories**: A-005

Full listings directory.

**Key sections:**
- Search bar (by yacht name or owner name/email)
- Filter by status (All / Active / Paused / Draft)
- Filter by type (Sailboat / Catamaran / Motor yacht)
- Listings table:

| Column | Content |
|---|---|
| Yacht | Photo thumbnail + name |
| Owner | Owner name (link to `/users/:id`) |
| Type | Yacht type |
| Location | Home port |
| Status | Active / Paused / Draft badge |
| Created | Date |
| Bookings | Total booking count |
| Actions | "View" link |

- Pagination (50 per page)

---

#### `/listings/:id` — Listing Detail
**Auth**: Protected
**Stories**: A-006, A-007

Full listing detail for admin review and management.

**Key sections:**
- **Listing header**: primary photo, yacht name, status badge, "View on storefront" link
- **Details**: all listing fields (type, length, capacity, home port, description, crew options, pricing, charter types)
- **Photos**: photo gallery grid (read-only)
- **Owner**: owner name + link to `/users/:id`
- **Booking history**: table of all bookings for this listing (reference, dates, renter, status, amount)
- **Actions**:
  - Pause listing (with reason field)
  - Re-activate listing
  - Delete listing (only if no confirmed future bookings — confirmation modal with warning)
  - Owner notification sent automatically on pause/delete

---

### Bookings

---

#### `/bookings` — All Bookings
**Auth**: Protected
**Stories**: A-008

Full bookings directory.

**Key sections:**
- Search bar (by booking reference, renter name, or yacht name)
- Filter by status (All / Pending / Confirmed / Checked In / Completed / Cancelled)
- Date range filter (check-in from/to)
- Bookings table:

| Column | Content |
|---|---|
| Reference | Booking ID |
| Yacht | Yacht name (link to `/listings/:id`) |
| Renter | Renter name (link to `/users/:id`) |
| Owner | Owner name (link to `/users/:id`) |
| Dates | Check-in → Check-out |
| Status | Status badge |
| Total | `total_price_cents` |
| Commission | `platform_fee_cents` |
| Actions | "View" link |

- Pagination (50 per page)
- Export to CSV (Phase 2)

---

#### `/bookings/:id` — Booking Detail
**Auth**: Protected
**Stories**: A-009, A-010, A-011

Full booking detail with admin action capabilities.

**Key sections:**
- **Booking header**: reference, status badge, created at
- **Parties**: renter (name, email, phone, link) + owner (name, email, link)
- **Charter details**: yacht (link), dates, crew option, guest count
- **Financials**:
  - Total paid by renter
  - Platform commission (YachtBay)
  - Owner payout amount
  - Stripe `payment_intent_id` (link to Stripe dashboard)
  - Stripe `transfer_id` (link to Stripe dashboard — if payout sent)
  - Stripe `refund_id` (if refunded)
- **Reservation event log** (full audit trail — expanded by default for admin):
  - All state transitions with timestamps
  - Who triggered each event (renter / owner / system / admin)
- **Actions** (shown only when applicable):
  - "Trigger payout manually" — visible if `COMPLETED` + payout not sent
  - "Issue refund" — visible for any non-cancelled booking + amount input + required reason

**Trigger payout modal**:
- Shows owner name, payout amount, Stripe account ID
- "Confirm payout" → calls `POST /api/v1/admin/bookings/:id/trigger-payout`

**Issue refund modal**:
- Full or partial refund selector
- Amount input (capped at total paid)
- Required reason field
- Warning: "If payout already sent to owner, clawback must be handled manually"
- "Confirm refund" → calls `POST /api/v1/admin/bookings/:id/refund`

---

### Revenue

---

#### `/revenue` — Revenue & Commission Dashboard
**Auth**: Protected
**Stories**: A-012, A-013

Platform financial overview.

**Key sections:**
- **Summary cards**: Total commission all time / This year / This month / This week
- **Monthly breakdown table**:

| Month | Bookings | Total booking value | Commission earned | Avg commission/booking |
|---|---|---|---|---|
| May 2026 | 12 | €14,400 | €1,872 | €156 |
| ... | | | | |

- **Top owners by commission generated** (table — owner name, total bookings, total commission generated for platform)
- **Booking funnel**: Total bookings / Completed / Cancelled — cancellation rate
- **Average booking value** (month trend)

**Notes**: All figures sourced from `payments` table (`platform_fee_cents`). No external analytics service needed at MVP.

---

### Settings

---

#### `/settings` — Platform Settings
**Auth**: Protected
**Stories**: A-013

Platform-level configuration. Minimal at MVP.

**Key sections:**
- **Default commission rate**: current rate displayed (e.g. 13%), editable with save + confirmation. Affects all future bookings where owner has no custom rate.
- **Cancellation policy thresholds**:
  - Day trips: full refund window (days), partial refund window (days), partial refund percentage
  - Weekly charters: same fields
  - Editable — changes apply to all new bookings (existing bookings use snapshot)
- **Platform information**: platform name (YachtBay — editable for when name is finalised), contact email
- *(Phase 2: Promotions management, featured listing settings)*

---

### Auth

---

#### `/login` — Admin Login
**Auth**: Public (redirect to `/dashboard` if already logged in)
**Stories**: A-001

Minimal login page. No registration link — admin accounts are created via seed.

**Fields**: Email, password
**Notes**: Failed login after 5 attempts → rate limited. No "forgot password" link on the public form — admins reset via CLI or backend. Generic error message only (no email enumeration).

---

## Route Summary

| Route | Auth | Stories |
|---|---|---|
| `/dashboard` | Protected | A-012 |
| `/users` | Protected | A-002 |
| `/users/:id` | Protected | A-003, A-004, A-013 |
| `/listings` | Protected | A-005 |
| `/listings/:id` | Protected | A-006, A-007 |
| `/bookings` | Protected | A-008 |
| `/bookings/:id` | Protected | A-009, A-010, A-011 |
| `/revenue` | Protected | A-012 |
| `/settings` | Protected | A-013 |
| `/login` | Public | A-001 |

**Total pages: 10**

---

## Navigation Structure

### Sidebar (persistent)
```
[YachtBay Admin]

Dashboard
Users
Listings
Bookings
Revenue

──────────────

Settings

──────────────

Log out
```

**No mobile optimisation required** — admin panel is used on desktop only.

---

## Total Site Map Summary

| App | Pages |
|---|---|
| Storefront (yachtbay.com) | 18 |
| Owner Panel (owners.yachtbay.com) | 25 |
| Admin Panel (admin.yachtbay.com) | 10 |
| **Total** | **53 pages** |

---

## Related Pages

- [[specs/design/sitemap-storefront]]
- [[specs/design/sitemap-owner-panel]]
- [[specs/user-stories/admin]]
- [[specs/mvp-scope]]
