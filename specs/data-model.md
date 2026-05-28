---
title: Data Model — Final Prisma Schema
tags:
  - spec
  - technical
  - data-model
  - backend
status: approved
last-updated: 2026-05-27
---

# Data Model — Final Prisma Schema

**Summary**: Complete, finalized Prisma schema for YachtBay. Produced by auditing the draft data model in [[specs/backend-architecture]] against all 68 user stories.

---

## Gap Analysis — What Changed from Draft

The following gaps were found during the user story audit:

### `users` table
| Gap | Source | Fix |
|---|---|---|
| `role` is a single enum — a user who is owner + renter needs multiple roles | R-022, O-001 | Changed to `roles String[]` array |
| Missing `phone` | O-001 (required for owners), R-021 | Added `phone` (nullable) |
| Missing `status` (active / suspended) | A-004 | Added `status` enum |
| Missing `suspension_reason` | A-004 | Added (nullable, internal only) |
| Missing `email_verified` | R-001, O-001 | Added boolean |
| Missing `email_verification_token` + expiry | R-001 | Added (nullable) |
| Missing `password_reset_token` + expiry | R-004 | Added (nullable) |
| Missing `bio` (owner about text shown on listing) | O-021 | Added (nullable) |
| Missing `profile_photo_url` | O-021 | Added (nullable) |
| Missing `commission_rate` (per-owner override) | A-013 | Added (nullable Decimal) |
| Missing `notification_preferences` | /settings in owner panel | Added (nullable Json) |

### `yachts` table
| Gap | Source | Fix |
|---|---|---|
| Missing `slug` (for SEO URLs `/yachts/:id-:slug`) | R-010, sitemap | Added (unique String) |
| Missing `prep_days` | O-013 | Added (Int, default 0) |
| Missing `charter_types` (which duration types are enabled) | O-008 | Added (Json: `{day_trip, multi_day, weekly}`) |
| `manufacturer` and `model` should be optional | O-005 (not all yachts have these) | Made nullable |

### `yacht_photos` table
| Gap | Source | Fix |
|---|---|---|
| Table entirely missing from draft | O-006, R-010 | Added new table |

### `yacht_availability` table
| Gap | Source | Fix |
|---|---|---|
| Missing `owner_note` for blocked date internal notes | O-012 | Added (nullable String) |

### `reservations` table
| Gap | Source | Fix |
|---|---|---|
| Missing `guest_count` | O-015 (booking detail shows guest count) | Added (Int) |

### `reservation_events` table
| Gap | Source | Fix |
|---|---|---|
| Missing `triggered_by` (who caused the event) | A-009 (audit trail) | Added String enum values |
| Missing `admin_id` (for admin-triggered events) | A-010, A-011 | Added (nullable String) |

### `payments` table
| Gap | Source | Fix |
|---|---|---|
| Missing `stripe_refund_id` | A-011 (manual refund) | Added (nullable String) |

### `outbox_events` table
| Gap | Source | Fix |
|---|---|---|
| Missing `error_message` for failed delivery debugging | Operational need | Added (nullable String) |

---

## Final Prisma Schema

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ─── ENUMS ────────────────────────────────────────────────────────────────────

enum Role {
  RENTER
  OWNER
  ADMIN
}

enum UserStatus {
  ACTIVE
  SUSPENDED
}

enum StripeAccountStatus {
  PENDING
  ACTIVE
  RESTRICTED
}

enum YachtType {
  SAILBOAT
  CATAMARAN
  MOTOR_YACHT
  OTHER
}

enum YachtStatus {
  DRAFT
  ACTIVE
  PAUSED
}

enum AvailabilityStatus {
  AVAILABLE
  RESERVED
  BLOCKED
  PREP
}

enum ReservationStatus {
  PENDING
  CONFIRMED
  CHECKED_IN
  COMPLETED
  CANCELLED
}

enum CrewOption {
  BAREBOAT
  SKIPPERED
  CREWED
}

enum PaymentType {
  DEPOSIT
  BALANCE
  REFUND
}

enum PaymentStatus {
  PENDING
  CAPTURED
  FAILED
  REFUNDED
}

enum OutboxStatus {
  PENDING
  DELIVERED
  FAILED
}

// ─── USERS ────────────────────────────────────────────────────────────────────

model User {
  id    String @id @default(cuid())
  email String @unique

  // Auth
  passwordHash String
  roles        Role[]     @default([RENTER])
  status       UserStatus @default(ACTIVE)

  // Profile
  firstName         String
  lastName          String
  phone             String?
  bio               String?  // owner bio shown on listing page
  profilePhotoUrl   String?

  // Email verification
  emailVerified              Boolean   @default(false)
  emailVerificationToken     String?
  emailVerificationExpiresAt DateTime?

  // Password reset
  passwordResetToken     String?
  passwordResetExpiresAt DateTime?

  // Admin fields
  suspensionReason String? // internal only, never shown to user

  // Owner fields
  stripeAccountId     String?
  stripeAccountStatus StripeAccountStatus?
  commissionRate      Decimal?             @db.Decimal(5, 2)
  // null = use platform default (13%). Set per owner by admin for volume discounts.

  // Preferences
  notificationPreferences Json?
  // {new_booking: true, cancellation: true, payout_released: true}

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  yachts               Yacht[]
  reservationsAsRenter Reservation[] @relation("RenterReservations")
}

// ─── YACHTS ───────────────────────────────────────────────────────────────────

model Yacht {
  id      String @id @default(cuid())
  ownerId String
  owner   User   @relation(fields: [ownerId], references: [id])

  // Core details
  name         String
  slug         String    @unique // generated from name, used in SEO URLs
  type         YachtType
  lengthMeters Decimal   @db.Decimal(5, 1)
  capacityGuests Int
  buildYear    Int
  manufacturer String?
  model        String?
  homePort     String
  locationLat  Float?  // stored as decimal degrees, used in haversine geo search
  locationLng  Float?
  description  String    @db.Text

  // Pricing
  basePriceCents Int // base day rate — crew options add on top via pricing_rules

  // Configuration
  crewOptions  Json // {bareboat: bool, skippered: bool, crewed: bool}
  charterTypes Json // {day_trip: bool, multi_day: bool, weekly: bool}
  minCharterDays Int @default(1)
  prepDays     Int @default(0) // days auto-blocked after each booking for cleaning

  status YachtStatus @default(DRAFT)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  photos       YachtPhoto[]
  availability YachtAvailability[]
  reservations Reservation[]
  pricingRules PricingRule[]

  @@index([ownerId])
  @@index([status])
}

// ─── YACHT PHOTOS ─────────────────────────────────────────────────────────────

model YachtPhoto {
  id           String   @id @default(cuid())
  yachtId      String
  yacht        Yacht    @relation(fields: [yachtId], references: [id], onDelete: Cascade)
  storageKey   String   // S3-compatible object key, provider-neutral (R2 today; used for deletion)
  url          String   // CDN public URL (used for display)
  displayOrder Int      // 0 = cover/primary photo
  isCover      Boolean  @default(false)
  createdAt    DateTime @default(now())

  @@index([yachtId, displayOrder])
}

// ─── YACHT AVAILABILITY ───────────────────────────────────────────────────────

model YachtAvailability {
  id     String @id @default(cuid())
  yachtId String
  yacht  Yacht  @relation(fields: [yachtId], references: [id])
  date   DateTime @db.Date

  status             AvailabilityStatus @default(AVAILABLE)
  priceOverrideCents Int?    // null = use pricing_rules for this date
  reservationId      String? // set when status = RESERVED
  reservation        Reservation? @relation(fields: [reservationId], references: [id])
  ownerNote          String? // internal note when owner manually blocks a date

  @@unique([yachtId, date]) // hard constraint — final safety net against double booking
  @@index([yachtId, status, date])
}

// ─── RESERVATIONS ─────────────────────────────────────────────────────────────

model Reservation {
  id       String @id @default(cuid())
  yachtId  String
  yacht    Yacht  @relation(fields: [yachtId], references: [id])
  renterId String
  renter   User   @relation("RenterReservations", fields: [renterId], references: [id])

  // Charter details
  checkIn    DateTime   @db.Date
  checkOut   DateTime   @db.Date
  crewOption CrewOption
  guestCount Int

  // Status
  status ReservationStatus @default(PENDING)

  // Financials (all in EUR cents)
  totalPriceCents  Int // full amount paid by renter
  platformFeeCents Int // YachtBay commission (13% or per-owner rate)
  ownerPayoutCents Int // totalPriceCents - platformFeeCents

  // Policy frozen at booking time — owner/platform policy changes don't affect this booking
  cancellationPolicySnapshot Json

  // Concurrency control
  version        Int    @default(1) // incremented on every state change (optimistic locking)
  idempotencyKey String @unique     // prevents duplicate bookings from network retries

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relations
  payments     Payment[]
  events       ReservationEvent[]
  availability YachtAvailability[]

  @@index([yachtId, status])
  @@index([renterId])
}

// ─── RESERVATION EVENTS (immutable audit log) ─────────────────────────────────

model ReservationEvent {
  id            String      @id @default(cuid())
  reservationId String
  reservation   Reservation @relation(fields: [reservationId], references: [id])

  eventType   String
  // created | confirmed | checked_in | completed | cancelled
  // payment_captured | admin_refund | admin_payout_triggered

  triggeredBy String
  // renter | owner | system | admin

  adminId String?
  // populated when triggeredBy = admin — links to the admin user who took the action

  payload   Json     // full context snapshot at time of event
  createdAt DateTime @default(now()) // never updated, never deleted

  @@index([reservationId])
}

// ─── PRICING RULES ────────────────────────────────────────────────────────────

model PricingRule {
  id      String  @id @default(cuid())
  yachtId String? // null = platform-wide rule
  yacht   Yacht?  @relation(fields: [yachtId], references: [id])

  ruleType String
  // seasonal | weekend | long_charter | crew_option | promotional

  // Date range (nullable for non-date-based rules like crew_option)
  dateFrom DateTime? @db.Date
  dateTo   DateTime? @db.Date

  modifierType  String
  // percentage (e.g. +20 for 20% surcharge, -10 for 10% discount)
  // fixed_override (e.g. 50000 = €500 flat rate for the period)

  modifierValue Decimal @db.Decimal(10, 2)

  priority Int     @default(0) // higher number = higher priority when rules overlap
  active   Boolean @default(true)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([yachtId, active])
  @@index([ruleType, active])
}

// ─── PAYMENTS ─────────────────────────────────────────────────────────────────

model Payment {
  id            String      @id @default(cuid())
  reservationId String
  reservation   Reservation @relation(fields: [reservationId], references: [id])

  paymentType PaymentType
  status      PaymentStatus @default(PENDING)

  // Amounts (EUR cents)
  amountCents      Int // total charged to renter
  platformFeeCents Int // application_fee_amount — goes to YachtBay
  ownerAmountCents Int // transferred to owner's Stripe Connect account

  // Stripe references
  stripePaymentIntentId String? // PaymentIntent ID
  stripeTransferId      String? // Connect transfer to owner
  stripeRefundId        String? // populated when a refund is issued

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([reservationId])
}

// ─── TRANSACTIONAL OUTBOX ─────────────────────────────────────────────────────

model OutboxEvent {
  id            String       @id @default(cuid())
  aggregateId   String       // e.g. reservation ID
  aggregateType String       // e.g. "reservation"
  eventType     String       // e.g. "reservation.confirmed"
  payload       Json
  status        OutboxStatus @default(PENDING)
  attempts      Int          @default(0)
  lastAttemptAt DateTime?
  errorMessage  String?      // populated on delivery failure for debugging

  createdAt DateTime @default(now())

  @@index([status, createdAt]) // outbox poller queries: WHERE status = PENDING ORDER BY createdAt
}
```

---

## Critical Indexes Summary

```sql
-- Double-booking prevention (most important index)
UNIQUE INDEX ON yacht_availability (yacht_id, date)

-- Availability search
INDEX ON yacht_availability (yacht_id, status, date)

-- Reservation lookups
INDEX ON reservations (yacht_id, status)
INDEX ON reservations (renter_id)
UNIQUE INDEX ON reservations (idempotency_key)

-- Outbox poller
INDEX ON outbox_events (status, created_at) WHERE status = 'PENDING'

-- Photo ordering
INDEX ON yacht_photos (yacht_id, display_order)

-- Pricing rules
INDEX ON pricing_rules (yacht_id, active)
INDEX ON pricing_rules (rule_type, active)

-- Geo search (haversine — B-tree composite)
INDEX ON yachts (status, location_lat, location_lng)

-- Listings by owner
INDEX ON yachts (owner_id)
INDEX ON yachts (status)
```

---

## Design Decisions

### Roles as array
`roles String[]` replaces the single `role` enum. A user can be `[RENTER]`, `[OWNER]`, `[RENTER, OWNER]`, or `[ADMIN]`. This supports the "add owner role to renter account" flow (R-022, O-001) without creating a new account.

### Slug on yachts
`slug` is generated server-side from the yacht name at creation (e.g. "Ocean Blue" → `ocean-blue`). Combined with the ID in URLs (`/yachts/abc123-ocean-blue`) for SEO while keeping routing ID-based.

### Charter types as Json
`charterTypes: {day_trip: bool, multi_day: bool, weekly: bool}` on the yacht. The minimum charter duration (`minCharterDays`) enforces the day-count constraint at booking time.

### Location as lat/lng floats (not PostGIS)
`locationLat Float?` + `locationLng Float?` replace what would have been a `Unsupported("geometry(Point, 4326)")` PostGIS column. Geo search uses `$queryRaw` with a haversine formula and a B-tree composite index on `(status, location_lat, location_lng)`. At Greece-only MVP scale this is sufficient. Migration path: when listings exceed ~100k or geographic scope expands, add the PostGIS extension, populate a geometry column from the float pair, swap the search SQL, drop the floats. See [[decisions/2026-05-27-defer-postgis-to-scale]].

### Commission rate
`commissionRate` on `User` is nullable. `null` = use platform default (currently 13%). When admin sets a custom rate (A-013), it's stored here and read by `CommissionStrategy` at pricing time.

### Cancellation policy snapshot
Stored as JSON on the reservation at booking time. Even if the platform policy changes, existing bookings use the policy that was in effect when they were created. Required for legal clarity.

### `triggered_by` on reservation events
Enables the admin audit trail (A-009) to show who caused each state change. Values: `renter | owner | system | admin`. `adminId` is also stored when `triggered_by = admin` for accountability.

---

## What This Schema Does NOT Include (intentional)

| Excluded | Reason |
|---|---|
| `reviews` table | Phase 2 — no completed bookings at launch |
| `conversations` / `messages` tables | Phase 2 — contact details shared by email at MVP |
| `wishlists` table | Phase 2 |
| `promotions` table | Phase 2 |
| `featured_listings` flag | Phase 2 |
| Multi-currency fields | Phase 3 (Mediterranean expansion) — all amounts in EUR cents |

All Phase 2 tables are additive — no schema changes to existing tables required.

---

## Related Pages

- [[specs/backend-architecture]]
- [[specs/user-stories/renter]]
- [[specs/user-stories/owner]]
- [[specs/user-stories/admin]]
- [[decisions/2026-05-03-backend-architecture]]
