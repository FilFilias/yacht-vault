---
title: Site Map — Owner Panel
tags:
  - spec
  - design
  - sitemap
  - owner-panel
status: draft
last-updated: 2026-05-05
---

# Site Map — Owner Panel (owners.yachtbay.com)

**App**: Owner-facing dashboard — listing management, availability, booking management, payouts.
**Framework**: React Router 7 (SPA mode — no SSR needed, all routes behind auth)
**Auth**: Shared JWT cookie with storefront (`.yachtbay.com` domain)

---

## Page Hierarchy

```
owners.yachtbay.com/
│
├── /dashboard .......................... Overview dashboard
│
├── /listings ........................... All listings
│   │
│   ├── /listings/new ................... Create listing (multi-step wizard)
│   │   ├── /listings/new/details ....... Step 1 — Basic details + location
│   │   ├── /listings/new/photos ........ Step 2 — Upload photos
│   │   ├── /listings/new/pricing ....... Step 3 — Pricing
│   │   ├── /listings/new/configuration . Step 4 — Crew options + charter types
│   │   ├── /listings/new/availability .. Step 5 — Prep days + min duration
│   │   └── /listings/new/review ........ Step 6 — Review & publish
│   │
│   └── /listings/:id ................... Listing overview
│       ├── /listings/:id/edit .......... Edit basic details
│       ├── /listings/:id/photos ........ Manage photos
│       ├── /listings/:id/pricing ....... Manage pricing
│       ├── /listings/:id/availability .. Availability calendar + date blocking
│       └── /listings/:id/configuration . Crew options + charter types + prep days
│
├── /bookings ........................... All bookings
│   └── /bookings/:id ................... Booking detail
│
├── /payouts ............................ Payout history + earnings summary
│
├── /profile ............................ Owner profile
│
├── /settings ........................... Account settings
│   └── /settings/stripe ................ Stripe Connect status + onboarding
│
├── /login .............................. Login
├── /register ........................... Register as owner
├── /forgot-password .................... Forgot password
├── /reset-password ..................... Reset password
└── /verify-email ....................... Email verification
```

---

## Pages

### Dashboard

---

#### `/dashboard` — Overview Dashboard
**Auth**: Protected (owner role required)
**Stories**: O-014, O-019, O-022

The owner's home screen. Provides a quick read on performance and pending actions.

**Key sections:**
- Stripe account status banner (if not Active — prompts to complete onboarding)
- Stats row: Total bookings (month), Earnings this month, Upcoming charters, Active listings
- Upcoming bookings (next 3 — compact cards with dates, renter name, yacht)
- Recent activity feed (new booking, cancellation, payout released)
- Quick actions: "Add new listing", "View all bookings", "View payouts"
- My listings summary (active listings count, paused listings count)

**Notes**: If owner has no listings yet → show a prominent "Create your first listing" CTA. Empty state is important for new owner onboarding.

---

### Listings

---

#### `/listings` — All Listings
**Auth**: Protected
**Stories**: O-005, O-010

Overview of all the owner's yacht listings.

**Key sections:**
- "Add new listing" button (top right)
- Listing cards: yacht photo, name, type, status badge (Active / Paused / Draft), total bookings, earnings to date
- Filter by status (All / Active / Paused / Draft)
- Empty state: "You haven't listed a yacht yet" + CTA

---

#### Listing Creation Wizard — `/listings/new/*`
**Auth**: Protected
**Stories**: O-005, O-006, O-007, O-008

Multi-step form for creating a new listing. Progress is saved after each step — owner can close and resume. Steps are separate routes enabling clean back/forward navigation.

**Step indicator**: Persistent at top of each step (1 of 6 — Basic Details).

---

##### `/listings/new/details` — Step 1: Basic Details + Location
**Key fields:**
- Yacht name
- Type (Sailboat / Catamaran / Motor yacht / Other)
- Length (meters)
- Guest capacity
- Build year
- Manufacturer
- Model
- Home port (text search via Mapbox Geocoding + map pin confirmation)
- Description (rich text — basic formatting only)

**Notes**: Location selected via Mapbox search-as-you-type + draggable map pin for precision. `lat/lng` stored, not just the text label.

---

##### `/listings/new/photos` — Step 2: Photos
**Key sections:**
- Drag-and-drop photo upload zone
- Uploaded photos grid with reorder (drag), delete, and set-as-cover actions
- Upload progress indicators
- Validation: minimum 3 photos required to proceed
- Format/size guidance: JPG/PNG/WebP, max 10MB each

**Notes**: Pre-signed R2 URL upload — photos go directly from browser to Cloudflare R2. Backend stores object keys.

---

##### `/listings/new/pricing` — Step 3: Pricing
**Key fields:**
- Day rate (€/day) — required
- Weekly rate (€/week) — optional (applied automatically for 7-day bookings)
- Live preview: "Renters will see this price. YachtBay deducts 13% commission — you receive €X per day."

**Notes**: Crew option pricing is set in Step 4 (after crew options are selected). Prices always in EUR.

---

##### `/listings/new/configuration` — Step 4: Crew Options + Charter Types
**Key sections:**
- Crew options (multi-select toggle): Bareboat / Skippered / Fully crewed
  - Price input per enabled option (adds to base day rate)
  - Bareboat: optional note about licence requirements
- Charter types (multi-select toggle): Day trips / Multi-day / Weekly
- Minimum charter duration (numeric input, in days — e.g. 7 for weekly minimum)

---

##### `/listings/new/availability` — Step 5: Availability Settings
**Key fields:**
- Preparation days between bookings (0 / 1 / 2 / 3 days — selector)
- Default availability: "Available for all dates" (owner then blocks specific dates manually after publishing)

**Notes**: The full availability calendar management is in the listing detail page after publishing. This step just captures global settings.

---

##### `/listings/new/review` — Step 6: Review & Publish
**Key sections:**
- Full listing preview (exactly as renters will see it on the storefront)
- Checklist: all required fields complete ✓, minimum photos ✓, Stripe account active ✓
- If Stripe not connected: "Connect your bank account to publish" CTA → `/settings/stripe`
- "Publish listing" button → sets `status: active`, listing goes live on storefront
- "Save as draft" option (listing saved but not published)

---

#### `/listings/:id` — Listing Overview
**Auth**: Protected (owner must own the listing)
**Stories**: O-009, O-010

Hub page for managing a specific listing. Shows performance summary and links to all management sections.

**Key sections:**
- Listing header: photo, name, status badge, "View on storefront" link (opens yachtbay.com/yachts/:id)
- Performance: total bookings, total earnings, occupancy rate (this month), avg booking value
- Quick stats: upcoming bookings for this yacht
- Management links (cards or tabs): Edit Details / Photos / Pricing / Availability / Configuration
- Danger zone: Pause listing / Delete listing (draft only)

---

#### `/listings/:id/edit` — Edit Basic Details
**Auth**: Protected
**Stories**: O-009

Same fields as Step 1 of the creation wizard. Saves on submit — changes apply immediately to the live listing.

---

#### `/listings/:id/photos` — Manage Photos
**Auth**: Protected
**Stories**: O-006

Same interface as Step 2 of the creation wizard. Owner can add, remove, reorder, and change cover photo.

---

#### `/listings/:id/pricing` — Manage Pricing
**Auth**: Protected
**Stories**: O-007

Same interface as Step 3 + crew option pricing from Step 4. Live preview of what renters see.

---

#### `/listings/:id/availability` — Availability Calendar
**Auth**: Protected
**Stories**: O-011, O-012

The primary availability management page.

**Key sections:**
- Full calendar view (6 months, current + next 5)
- Colour legend: Available (green) / Reserved — booked (blue) / Blocked by owner (grey) / PREP days (light grey)
- Click a date to see status + booking reference (if reserved)
- Select date range → "Block dates" action with optional internal note
- Select blocked dates → "Unblock" action
- Cannot block or unblock reserved/PREP dates

---

#### `/listings/:id/configuration` — Crew Options + Charter Types + Prep Days
**Auth**: Protected
**Stories**: O-008, O-013

Same interface as Steps 4 + 5 of the creation wizard combined. Update crew options, charter types, minimum duration, and preparation days.

---

### Bookings

---

#### `/bookings` — All Bookings
**Auth**: Protected
**Stories**: O-014

Booking management dashboard across all listings.

**Key sections:**
- Tab navigation: Upcoming | Active (checked in) | Completed | Cancelled
- Booking card: yacht photo, renter first name, check-in/out dates, crew option, owner payout amount, status
- Filter by listing (if owner has multiple yachts)
- Search by booking reference or renter name

---

#### `/bookings/:id` — Booking Detail
**Auth**: Protected (owner must own the booking's yacht)
**Stories**: O-015, O-016, O-017

Full booking detail with action buttons.

**Key sections:**
- Booking reference + status badge
- Renter contact details (name, email, phone)
- Charter details (yacht, dates, crew option, guest count)
- Owner payout amount + payment status
- Reservation event log (state history — collapsed by default)
- Actions (based on status):
  - `CONFIRMED` + date ≥ check-in → "Mark as Checked In"
  - `CHECKED_IN` + date ≥ check-out → "Mark as Completed"
- Confirmation modal before each action

---

### Payouts

---

#### `/payouts` — Payout History + Earnings
**Auth**: Protected
**Stories**: O-019, O-020

**Key sections:**
- Earnings summary: This month / Last month / All time (total owner payout)
- Payout table: booking reference, yacht, charter dates, payout amount, payout date, status (Pending / Paid)
- "Open Stripe Dashboard" button → generates Stripe Express Dashboard link, opens in new tab

---

### Profile & Settings

---

#### `/profile` — Owner Profile
**Auth**: Protected
**Stories**: O-021

**Key sections:**
- Personal details: first name, last name, phone, profile photo (R2 upload)
- Bio / about (displayed on listing pages under owner card)
- Email (read-only)
- Change password section
- "Switch to Renter View" link → `yachtbay.com`

---

#### `/settings` — Account Settings
**Auth**: Protected

General account settings. At MVP: primarily a hub pointing to Stripe settings.

**Key sections:**
- Notification preferences (email — on/off per event type): new booking, cancellation, payout released
- Link to `/settings/stripe`

---

#### `/settings/stripe` — Stripe Connect Status
**Auth**: Protected
**Stories**: O-003, O-022

**Key sections:**
- Stripe account status (Active / Pending / Restricted) with colour indicator
- If Pending or not connected: "Complete Stripe onboarding" button → redirect to Stripe-hosted flow → return to this page on completion
- If Restricted: explanation of what's needed + "Resolve in Stripe" button
- If Active: "Open Stripe Dashboard" link (same as payouts page)
- Payout schedule info: "Payouts are released 24–48 hours after your charter start date"

---

### Auth Pages

---

#### `/login` — Login
**Auth**: Public (redirect to `/dashboard` if already logged in)
**Stories**: O-002

**Fields**: Email, password
**Notes**: If authenticated account lacks `owner` role → show "Your account doesn't have owner access" with a link to add owner role or register. "Forgot password?" and "Create an owner account" links.

---

#### `/register` — Register as Owner
**Auth**: Public
**Stories**: O-001

**Fields**: First name, last name, email, phone, password, confirm password
**Notes**: After registration → email verification. Phone required (vs optional on storefront). "Already have a renter account? Add owner access" link → triggers add-owner-role flow for existing renter accounts.

---

#### `/forgot-password`, `/reset-password`, `/verify-email`
**Auth**: Public
**Stories**: O-001, O-002

Same patterns as storefront auth pages. Shared backend endpoints.

---

## Route Summary

| Route | Auth | Stories |
|---|---|---|
| `/dashboard` | Protected | O-014, O-019, O-022 |
| `/listings` | Protected | O-005, O-010 |
| `/listings/new/details` | Protected | O-005 |
| `/listings/new/photos` | Protected | O-006 |
| `/listings/new/pricing` | Protected | O-007 |
| `/listings/new/configuration` | Protected | O-008 |
| `/listings/new/availability` | Protected | O-013 |
| `/listings/new/review` | Protected | O-005 |
| `/listings/:id` | Protected | O-009, O-010 |
| `/listings/:id/edit` | Protected | O-009 |
| `/listings/:id/photos` | Protected | O-006 |
| `/listings/:id/pricing` | Protected | O-007 |
| `/listings/:id/availability` | Protected | O-011, O-012 |
| `/listings/:id/configuration` | Protected | O-008, O-013 |
| `/bookings` | Protected | O-014 |
| `/bookings/:id` | Protected | O-015, O-016, O-017 |
| `/payouts` | Protected | O-019, O-020 |
| `/profile` | Protected | O-021 |
| `/settings` | Protected | — |
| `/settings/stripe` | Protected | O-003, O-022 |
| `/login` | Public | O-002 |
| `/register` | Public | O-001 |
| `/forgot-password` | Public | — |
| `/reset-password` | Public | — |
| `/verify-email` | Public | — |

**Total pages: 25**

---

## Navigation Structure

### Sidebar (persistent, desktop)
```
[YachtBay Owner]

Dashboard
Listings
  ├─ All listings
  └─ + Add listing
Bookings
Payouts

──────────────

Profile
Settings
  └─ Stripe Connect

──────────────

[Switch to Renter View] ↗
```

### Mobile
Sidebar collapses to a hamburger menu. Bottom tab bar for primary navigation: Dashboard / Listings / Bookings / Payouts.

---

## Related Pages

- [[specs/design/sitemap-storefront]]
- [[specs/design/sitemap-admin]]
- [[specs/user-stories/owner]]
- [[specs/mvp-scope]]
