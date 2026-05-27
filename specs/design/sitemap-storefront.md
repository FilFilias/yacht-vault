---
title: Site Map — Storefront
tags:
  - spec
  - design
  - sitemap
  - storefront
status: draft
last-updated: 2026-05-04
---

# Site Map — Storefront (yachtbay.com)

**App**: Public consumer-facing site — discovery, search, and booking for renters.
**Framework**: React Router 7 (SSR mode)
**Auth**: Shared JWT cookie with owner panel (`.yachtbay.com` domain)

---

## Page Hierarchy

```
yachtbay.com/
│
├── / ................................. Homepage
├── /search ........................... Search results
│
├── /yachts/:id ........................ Yacht detail
│   └── /yachts/:id/checkout ........... Checkout (protected)
│       └── /yachts/:id/checkout/pay ... Payment step (protected)
│
├── /bookings .......................... My bookings (protected)
│   └── /bookings/:id .................. Booking detail (protected)
│       └── /bookings/:id/confirmed .... Booking confirmation (protected)
│
├── /profile ........................... Profile & account (protected)
│
├── /register .......................... Register
├── /login ............................. Login
├── /forgot-password ................... Forgot password
├── /reset-password .................... Reset password (token in URL)
└── /verify-email ...................... Email verification (token in URL)
│
├── /how-it-works ...................... How YachtBay works
├── /cancellation-policy ............... Platform cancellation policy
├── /terms ............................. Terms of service
└── /privacy ........................... Privacy policy
```

---

## Pages

### Public Pages

---

#### `/` — Homepage
**Auth**: Public
**SSR**: Yes
**Stories**: R-005

The entry point for renters. Primary goal: get the renter to search.

**Key sections:**
- Hero with search bar (location, check-in date, check-out date, guests)
- Featured / popular yacht listings (curated or highest-rated)
- How it works — 3 steps (Browse → Book → Sail)
- Popular destinations in Greece (Santorini, Mykonos, Corfu, Rhodes — map pins or photo cards)
- Trust signals (number of yachts, number of bookings, average rating)
- CTA for owners ("List your yacht")

**Notes**: Search bar is the most important element. Hero occupies the top fold. Pre-populating search with popular destinations is a nice-to-have.

---

#### `/search` — Search Results
**Auth**: Public
**SSR**: Yes (initial load), then client-side for filter/sort updates
**Stories**: R-005, R-006, R-007, R-008, R-009

The core discovery page. All search, filter, sort, and map interactions happen here.

**Key sections:**
- Search bar at top (editable — renter can change dates/location without going back to homepage)
- Results count ("34 yachts available")
- Filter panel (yacht type, crew option, price range, capacity)
- Sort selector (Price low→high, Price high→low, Relevance)
- Split view: listing cards (left) + Mapbox map (right)
- Listing card: primary photo, name, type, home port, capacity, crew options, price/day
- Pagination or infinite scroll (paginated at MVP)
- Empty state with suggestions

**URL structure**: `/search?location=Mykonos&checkIn=2026-07-01&checkOut=2026-07-08&guests=4`
All search params in URL — shareable and bookmarkable.

**Notes**: Map pins update when filters change. Clicking a map pin highlights the corresponding listing card and shows a mini preview popup (photo, name, price, "View yacht" link).

---

#### `/yachts/:id` — Yacht Detail
**Auth**: Public
**SSR**: Yes (critical for SEO)
**Stories**: R-010, R-011, R-012, R-013

The listing page. Goal: give the renter enough information to decide, and make booking frictionless.

**Key sections:**
- Photo gallery (full-width, lightbox on click)
- Yacht title, type, home port, capacity
- Booking widget (sticky on desktop, bottom sheet on mobile):
  - Date range picker (availability-aware)
  - Crew option selector (with price per option)
  - Price preview (updates dynamically)
  - "Book Now" CTA → proceeds to checkout
- Specs: length, build year, manufacturer, model
- Description
- Crew options detail (what each includes)
- Availability calendar (visual, read-only — booking done via widget)
- Cancellation policy (clearly displayed)
- Owner card (first name, member since — no contact details until booked)
- Location map (Mapbox, showing home port)

**URL format**: `/yachts/abc123-oceanblue-sailing` (id + slug for SEO)
**Notes**: Booking widget is the most important element. On mobile the booking widget collapses to a bottom bar with "From €X/day — Book Now". Unavailable dates are disabled in the date picker.

---

#### `/yachts/:id/checkout` — Checkout Summary
**Auth**: Protected (must be logged in — redirect to `/login?redirect=...` if not)
**SSR**: Yes
**Stories**: R-014, R-015

Displays full booking summary before payment. Renter reviews and confirms details.

**Key sections:**
- Yacht summary (photo, name, dates, crew option, duration)
- Price breakdown:
  - Base rate × days
  - Crew option fee (if applicable)
  - Weekly rate discount (if applicable)
  - **Total**
- Cancellation policy reminder
- "Proceed to Payment" CTA

**Notes**: No payment input on this page — renter confirms the order before entering card details. Data loaded from URL params or a server-created checkout session. Price always calculated server-side.

---

#### `/yachts/:id/checkout/pay` — Payment
**Auth**: Protected
**SSR**: Partial (page shell SSR, Stripe Elements loads client-side)
**Stories**: R-015, R-016

Renter enters card details and confirms booking.

**Key sections:**
- Compact booking summary (sidebar or top — yacht, dates, total)
- Stripe Elements card input
- "Confirm and Pay €X" button
- Security trust signals (Stripe badge, SSL)
- Error state (card declined, network error — inline, no full-page reload)

**Notes**: Stripe PaymentIntent created server-side before this page loads. Client confirms using Stripe Elements. On success → redirect to `/bookings/:id/confirmed`. Idempotency key prevents duplicate bookings on retry.

---

### Protected Pages (auth required)

---

#### `/bookings` — My Bookings
**Auth**: Protected
**SSR**: Yes
**Stories**: R-017

Renter's booking dashboard. Overview of all bookings.

**Key sections:**
- Tab navigation: Upcoming | Past | Cancelled
- Booking card per tab: yacht photo, name, dates, status badge, total paid, "View details" link
- Empty state per tab ("No upcoming bookings — start exploring")

---

#### `/bookings/:id` — Booking Detail
**Auth**: Protected (renter must own the booking)
**SSR**: Yes
**Stories**: R-018, R-019, R-020

Full booking details page. Also serves as the post-confirmation destination.

**Key sections:**
- Booking reference + status badge
- Yacht details (photo, name, type)
- Charter dates + crew option
- Price breakdown + payment status
- Owner contact details (email + phone — shown only here, post-booking)
- Cancellation policy + deadline ("Cancel by Jul 1 for a full refund")
- Actions:
  - "Cancel booking" (visible if within policy window + status = CONFIRMED)
- Cancellation confirmation modal (shows refund amount before confirming)

---

#### `/bookings/:id/confirmed` — Booking Confirmation
**Auth**: Protected
**SSR**: Yes
**Stories**: R-016

Success page shown immediately after payment. Celebrates the booking and sets expectations.

**Key sections:**
- Success banner ("You're going sailing!")
- Booking summary (dates, yacht, crew option)
- "What happens next" (confirmation email sent, owner contact details below)
- Owner contact details
- CTA: "View booking details" → `/bookings/:id`
- Secondary CTA: "Browse more yachts" → `/search`

**Notes**: This page is only accessible immediately after booking via redirect. Returning to it later redirects to `/bookings/:id`. Prevents confusion from sharing the URL.

---

#### `/profile` — Profile & Account
**Auth**: Protected
**SSR**: Yes
**Stories**: R-021, R-022

Renter's account settings.

**Key sections:**
- Personal details form: first name, last name, phone
- Email display (read-only — with "change email" label for Phase 2)
- Change password section
- "Switch to Owner Panel" link (visible if account has owner role) → `owners.yachtbay.com`
- Danger zone: delete account (Phase 2)

---

### Auth Pages

---

#### `/register` — Register
**Auth**: Public (redirect to `/` if already logged in)
**SSR**: Yes
**Stories**: R-001

Simple registration form.

**Fields**: First name, last name, email, password, confirm password
**Notes**: After submission → email verification page. "Already have an account? Log in" link.

---

#### `/login` — Login
**Auth**: Public (redirect to intended destination or `/` if already logged in)
**SSR**: Yes
**Stories**: R-002

Login form.

**Fields**: Email, password
**Notes**: Supports `?redirect=/yachts/abc123/checkout` param — redirects back after login. "Forgot password?" link. "Don't have an account? Register" link. If account lacks `renter` role, show appropriate message.

---

#### `/forgot-password` — Forgot Password
**Auth**: Public
**SSR**: Yes
**Stories**: R-004

**Fields**: Email
**Notes**: Always shows success message regardless of whether email exists (prevents enumeration).

---

#### `/reset-password` — Reset Password
**Auth**: Public (token in URL)
**SSR**: Yes
**Stories**: R-004

**Fields**: New password, confirm password
**Notes**: Token validated server-side before page renders. Expired/invalid token → error state with link to request a new one.

---

#### `/verify-email` — Email Verification
**Auth**: Public (token in URL)
**SSR**: Yes
**Stories**: R-001

Validates the email verification token. On success → redirect to `/` or original destination. On failure → error with "resend verification email" option.

---

### Info / Legal Pages

---

#### `/how-it-works` — How It Works
**Auth**: Public
**SSR**: Yes

Explains the platform to first-time visitors. Three sections: For Renters, For Owners (with CTA to owner panel), Trust & Safety.

---

#### `/cancellation-policy` — Cancellation Policy
**Auth**: Public
**SSR**: Yes

Displays the full platform cancellation policy (day trip tiers + weekly charter tiers). Referenced from listing pages and booking detail.

---

#### `/terms` — Terms of Service
**Auth**: Public | **`/privacy`** — Privacy Policy | **Auth**: Public

Standard legal pages. Required before launch.

---

## Route Summary

| Route | Auth | SSR | Stories |
|---|---|---|---|
| `/` | Public | Yes | R-005 |
| `/search` | Public | Yes | R-005–R-009 |
| `/yachts/:id` | Public | Yes | R-010–R-013 |
| `/yachts/:id/checkout` | Protected | Yes | R-014 |
| `/yachts/:id/checkout/pay` | Protected | Partial | R-015 |
| `/bookings` | Protected | Yes | R-017 |
| `/bookings/:id` | Protected | Yes | R-018–R-020 |
| `/bookings/:id/confirmed` | Protected | Yes | R-016 |
| `/profile` | Protected | Yes | R-021–R-022 |
| `/register` | Public | Yes | R-001 |
| `/login` | Public | Yes | R-002 |
| `/forgot-password` | Public | Yes | R-004 |
| `/reset-password` | Public | Yes | R-004 |
| `/verify-email` | Public | Yes | R-001 |
| `/how-it-works` | Public | Yes | — |
| `/cancellation-policy` | Public | Yes | — |
| `/terms` | Public | Yes | — |
| `/privacy` | Public | Yes | — |

**Total pages: 18**

---

## Navigation Structure

### Top Navigation (desktop)
```
[YachtBay Logo]    [Search bar (compact)]    [How it works]  [List your yacht]  [Login / Register]
                                                                                 [Avatar menu if logged in]
```

### Avatar menu (logged in)
```
My Bookings
Profile
──────────
Switch to Owner Panel  (visible if owner role)
──────────
Log out
```

### Footer
```
YachtBay               For Renters          For Owners          Legal
About                  How it works         List your yacht      Terms of service
                       Browse yachts        Owner FAQ            Privacy policy
                       Cancellation policy
```

---

## Related Pages

- [[specs/design/sitemap-owner-panel]]
- [[specs/design/sitemap-admin]]
- [[specs/user-stories/renter]]
- [[specs/mvp-scope]]
