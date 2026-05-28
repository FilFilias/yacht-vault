---
title: API Contract
tags:
  - spec
  - technical
  - backend
  - api
status: approved
last-updated: 2026-05-27
---

# API Contract

**Summary**: Complete endpoint definition for the YachtBay backend API. All endpoints are prefixed with `/api/v1/`. Breaking changes get a new version prefix (`/api/v2/`).

**Base URL**: `https://api.yachtbay.com/api/v1`

**Auth**: JWT via httpOnly cookie (`access_token`). Protected routes return `401` if missing, `403` if role insufficient.

**Conventions**:
- Monetary amounts: integer cents (e.g. `13000` = €130.00)
- Dates: ISO 8601 date string (`YYYY-MM-DD`)
- Timestamps: ISO 8601 datetime (`YYYY-MM-DDTHH:mm:ssZ`)
- IDs: cuid strings
- Errors: `{ error: string, message: string, statusCode: number }`

---

## Module Index

1. [Auth](#1-auth)
2. [Users](#2-users)
3. [Yachts](#3-yachts)
4. [Availability](#4-availability)
5. [Bookings](#5-bookings)
6. [Payments](#6-payments)
7. [Stripe Connect](#7-stripe-connect)
8. [Admin](#8-admin)

---

## 1. Auth

---

### `POST /auth/register`
**Auth**: Public | **Rate limit**: 5 req/min per IP

**Request body**:
```json
{
  "firstName": "string (required)",
  "lastName": "string (required)",
  "email": "string (required, valid email)",
  "password": "string (required, min 8 chars)",
  "phone": "string (optional)"
}
```

**Response `201`**:
```json
{
  "message": "Verification email sent. Please verify your email to continue."
}
```

**Errors**: `409` email already registered | `422` validation error

**Notes**: Sends verification email via Resend. User cannot log in until email is verified.

---

### `POST /auth/verify-email`
**Auth**: Public

**Request body**:
```json
{ "token": "string (required)" }
```

**Response `200`**:
```json
{ "message": "Email verified successfully." }
```

**Errors**: `400` token invalid or expired

---

### `POST /auth/login`
**Auth**: Public | **Rate limit**: 5 req/min per IP

**Request body**:
```json
{
  "email": "string (required)",
  "password": "string (required)"
}
```

**Response `200`** (sets httpOnly cookies `access_token` + `refresh_token`):
```json
{
  "user": {
    "id": "string",
    "firstName": "string",
    "lastName": "string",
    "email": "string",
    "roles": ["RENTER"],
    "emailVerified": true
  }
}
```

**Errors**: `401` invalid credentials | `403` email not verified | `403` account suspended

**Notes**: Cookies set with `domain: .yachtbay.com`, `httpOnly: true`, `secure: true`, `sameSite: lax`. Access token expires in 15min, refresh token in 7 days.

---

### `POST /auth/refresh`
**Auth**: Refresh token cookie

**Response `200`**: Sets new `access_token` cookie.

**Errors**: `401` refresh token invalid or expired

---

### `POST /auth/logout`
**Auth**: JWT

**Response `200`**: Clears both cookies.

---

### `POST /auth/forgot-password`
**Auth**: Public | **Rate limit**: 5 req/min per IP

**Request body**:
```json
{ "email": "string (required)" }
```

**Response `200`**:
```json
{ "message": "If that email is registered, a reset link has been sent." }
```

**Notes**: Always returns 200 regardless of whether email exists (prevents enumeration). Token expires in 1 hour.

---

### `POST /auth/reset-password`
**Auth**: Public

**Request body**:
```json
{
  "token": "string (required)",
  "password": "string (required, min 8 chars)"
}
```

**Response `200`**:
```json
{ "message": "Password reset successfully." }
```

**Errors**: `400` token invalid or expired | `422` validation error

**Notes**: Token is single-use. All active sessions invalidated on success.

---

## 2. Users

---

### `GET /users/me`
**Auth**: JWT

**Response `200`**:
```json
{
  "id": "string",
  "firstName": "string",
  "lastName": "string",
  "email": "string",
  "phone": "string | null",
  "roles": ["RENTER"],
  "bio": "string | null",
  "profilePhotoUrl": "string | null",
  "stripeAccountStatus": "PENDING | ACTIVE | RESTRICTED | null",
  "notificationPreferences": {
    "new_booking": true,
    "cancellation": true,
    "payout_released": true
  },
  "createdAt": "timestamp"
}
```

---

### `PATCH /users/me`
**Auth**: JWT

**Request body** (all fields optional):
```json
{
  "firstName": "string",
  "lastName": "string",
  "phone": "string",
  "bio": "string (max 500 chars)",
  "notificationPreferences": {
    "new_booking": "boolean",
    "cancellation": "boolean",
    "payout_released": "boolean"
  }
}
```

**Response `200`**: Updated user object (same shape as `GET /users/me`)

---

### `POST /users/me/add-owner-role`
**Auth**: JWT (RENTER role)

**Request body**: None

**Response `200`**:
```json
{ "message": "Owner role added. Complete Stripe onboarding to publish listings." }
```

**Notes**: Adds `OWNER` to `roles[]`. Does not create a new account.

---

### `POST /users/me/profile-photo/upload-url`
**Auth**: JWT

**Request body**:
```json
{ "contentType": "image/jpeg | image/png | image/webp" }
```

**Response `200`**:
```json
{
  "uploadUrl": "string (pre-signed R2 URL, expires in 5 min)",
  "publicUrl": "string (Cloudflare CDN URL to store after upload)"
}
```

**Notes**: Client uploads directly to R2 using `uploadUrl`. Then calls `PATCH /users/me` with `profilePhotoUrl` set to `publicUrl`.

---

## 3. Yachts

---

### `GET /yachts`
**Auth**: Public | **Rate limit**: 60 req/min per IP

**Query params**:
```
location     string    (place name — resolved to lat/lng via Mapbox)
lat          number    (direct lat — alternative to location)
lng          number    (direct lng — alternative to location)
radiusKm     number    (default: 50)
checkIn      date      (YYYY-MM-DD)
checkOut     date      (YYYY-MM-DD)
guests       integer
type         YachtType (SAILBOAT | CATAMARAN | MOTOR_YACHT | OTHER)
crewOption   CrewOption (BAREBOAT | SKIPPERED | CREWED)
minPrice     integer   (cents per day)
maxPrice     integer   (cents per day)
sortBy       string    (price_asc | price_desc | relevance)
page         integer   (default: 1)
limit        integer   (default: 20, max: 50)
```

**Response `200`**:
```json
{
  "data": [
    {
      "id": "string",
      "name": "string",
      "slug": "string",
      "type": "YachtType",
      "homePort": "string",
      "capacityGuests": 8,
      "crewOptions": { "bareboat": true, "skippered": true, "crewed": false },
      "charterTypes": { "day_trip": true, "multi_day": true, "weekly": true },
      "coverPhotoUrl": "string",
      "basePriceCents": 35000,
      "calculatedPriceCents": 245000,
      "location": { "lat": 37.4467, "lng": 25.3289 }
    }
  ],
  "meta": {
    "total": 34,
    "page": 1,
    "limit": 20,
    "totalPages": 2
  }
}
```

**Notes**: `calculatedPriceCents` is the total price for the full requested date range (if `checkIn`/`checkOut` provided) — used on listing cards. Only `ACTIVE` listings returned. Haversine geo filter applied when location/lat/lng provided.

---

### `POST /yachts`
**Auth**: JWT (OWNER role)

**Request body**:
```json
{
  "name": "string (required)",
  "type": "YachtType (required)",
  "lengthMeters": "number (required)",
  "capacityGuests": "integer (required)",
  "buildYear": "integer (required)",
  "manufacturer": "string (optional)",
  "model": "string (optional)",
  "homePort": "string (required)",
  "locationLat": "number (required)",
  "locationLng": "number (required)",
  "description": "string (required, min 50 chars)"
}
```

**Response `201`**:
```json
{
  "id": "string",
  "slug": "string",
  "status": "DRAFT"
}
```

**Notes**: Creates listing in `DRAFT` status. Slug auto-generated from name. Owner must complete all steps and have active Stripe account before publishing.

---

### `GET /yachts/:id`
**Auth**: Public (ACTIVE listings) | JWT required for DRAFT/PAUSED (owner only)

**Response `200`**:
```json
{
  "id": "string",
  "name": "string",
  "slug": "string",
  "type": "YachtType",
  "lengthMeters": 12.5,
  "capacityGuests": 8,
  "buildYear": 2019,
  "manufacturer": "string | null",
  "model": "string | null",
  "homePort": "string",
  "location": { "lat": 37.4467, "lng": 25.3289 },
  "description": "string",
  "basePriceCents": 35000,
  "crewOptions": { "bareboat": true, "skippered": true, "crewed": false },
  "charterTypes": { "day_trip": true, "multi_day": true, "weekly": true },
  "minCharterDays": 1,
  "prepDays": 1,
  "status": "ACTIVE",
  "photos": [
    { "id": "string", "url": "string", "displayOrder": 0, "isCover": true }
  ],
  "owner": {
    "id": "string",
    "firstName": "string",
    "bio": "string | null",
    "profilePhotoUrl": "string | null",
    "memberSince": "timestamp"
  },
  "createdAt": "timestamp"
}
```

**Errors**: `404` not found or paused listing

---

### `PATCH /yachts/:id`
**Auth**: JWT (OWNER — must own the listing)

**Request body** (all fields optional — partial update):
```json
{
  "name": "string",
  "type": "YachtType",
  "lengthMeters": "number",
  "capacityGuests": "integer",
  "buildYear": "integer",
  "manufacturer": "string",
  "model": "string",
  "homePort": "string",
  "locationLat": "number",
  "locationLng": "number",
  "description": "string"
}
```

**Response `200`**: Updated yacht object.

---

### `PATCH /yachts/:id/status`
**Auth**: JWT (OWNER — must own the listing)

**Request body**:
```json
{ "status": "ACTIVE | PAUSED | DRAFT" }
```

**Response `200`**: `{ "status": "ACTIVE" }`

**Errors**: `409` cannot activate if Stripe account not active | `409` cannot activate if required fields incomplete

---

### `DELETE /yachts/:id`
**Auth**: JWT (OWNER — must own the listing)

**Response `204`**: No content

**Errors**: `409` cannot delete listing with confirmed future bookings

---

### `PATCH /yachts/:id/pricing`
**Auth**: JWT (OWNER — must own the listing)

**Request body**:
```json
{
  "basePriceCents": "integer (required, > 0)",
  "weeklyRateCents": "integer (optional — applied automatically for 7-day bookings)",
  "crewOptionPricing": {
    "skippered": "integer (cents/day — required if skippered enabled)",
    "crewed": "integer (cents/day — required if crewed enabled)"
  }
}
```

**Response `200`**: `{ "message": "Pricing updated." }`

**Notes**: Crew option prices stored as `pricing_rules` with `rule_type: crew_option`. Weekly rate stored as `pricing_rules` with `rule_type: long_charter`.

---

### `PATCH /yachts/:id/configuration`
**Auth**: JWT (OWNER — must own the listing)

**Request body**:
```json
{
  "crewOptions": {
    "bareboat": "boolean",
    "skippered": "boolean",
    "crewed": "boolean"
  },
  "charterTypes": {
    "day_trip": "boolean",
    "multi_day": "boolean",
    "weekly": "boolean"
  },
  "minCharterDays": "integer (default: 1)",
  "prepDays": "integer (0-3)"
}
```

**Response `200`**: `{ "message": "Configuration updated." }`

**Errors**: `422` at least one crew option required | `422` at least one charter type required

---

### `GET /yachts/:id/pricing`
**Auth**: Public | **Rate limit**: 60 req/min per IP

**Query params**:
```
checkIn      date (required)
checkOut     date (required)
crewOption   CrewOption (required)
```

**Response `200`**:
```json
{
  "breakdown": [
    { "label": "Base rate (5 days × €350/day)", "amountCents": 175000 },
    { "label": "Skipper fee (5 days × €100/day)", "amountCents": 50000 }
  ],
  "totalCents": 225000,
  "currency": "EUR"
}
```

**Errors**: `422` invalid date range | `422` below minimum charter duration | `409` dates not available

**Notes**: Price always calculated server-side. Frontend never computes price.

---

### `POST /yachts/:id/photos/upload-url`
**Auth**: JWT (OWNER — must own the listing)

**Request body**:
```json
{
  "fileName": "string",
  "contentType": "image/jpeg | image/png | image/webp",
  "fileSizeBytes": "integer (max 10485760 = 10MB)"
}
```

**Response `200`**:
```json
{
  "uploadUrl": "string (pre-signed R2 URL, expires in 10 min)",
  "r2Key": "string",
  "publicUrl": "string"
}
```

---

### `POST /yachts/:id/photos`
**Auth**: JWT (OWNER — must own the listing)

**Request body**:
```json
{
  "r2Key": "string",
  "url": "string",
  "isCover": "boolean (optional, default false)"
}
```

**Response `201`**:
```json
{ "id": "string", "url": "string", "displayOrder": 3, "isCover": false }
```

---

### `PATCH /yachts/:id/photos/reorder`
**Auth**: JWT (OWNER)

**Request body**:
```json
{ "orderedPhotoIds": ["id1", "id2", "id3"] }
```

**Response `200`**: `{ "message": "Photos reordered." }`

---

### `DELETE /yachts/:id/photos/:photoId`
**Auth**: JWT (OWNER)

**Response `204`**: No content

**Errors**: `409` cannot delete — minimum 3 photos required

---

## 4. Availability

---

### `GET /yachts/:id/availability`
**Auth**: Public

**Query params**:
```
from    date (required, YYYY-MM-DD)
to      date (required, YYYY-MM-DD, max 12 months from now)
```

**Response `200`**:
```json
{
  "data": [
    { "date": "2026-07-01", "status": "AVAILABLE" },
    { "date": "2026-07-02", "status": "RESERVED" },
    { "date": "2026-07-03", "status": "PREP" }
  ]
}
```

**Notes**: Public response returns only `AVAILABLE` / `RESERVED` / `BLOCKED` / `PREP` — no internal owner notes. Owner panel requests (JWT + owner) receive `ownerNote` as well.

---

### `POST /yachts/:id/availability/block`
**Auth**: JWT (OWNER — must own the listing)

**Request body**:
```json
{
  "dates": ["2026-08-01", "2026-08-02"],
  "note": "string (optional, internal only)"
}
```

**Response `200`**: `{ "blocked": 2 }`

**Errors**: `409` one or more dates already reserved — lists conflicting dates

---

### `DELETE /yachts/:id/availability/block`
**Auth**: JWT (OWNER — must own the listing)

**Request body**:
```json
{ "dates": ["2026-08-01", "2026-08-02"] }
```

**Response `200`**: `{ "unblocked": 2 }`

**Errors**: `409` one or more dates are reserved (not owner-blocked) — cannot unblock

---

## 5. Bookings

---

### `POST /bookings`
**Auth**: JWT (RENTER role) | **Rate limit**: 10 req/min per IP | **Idempotency**: Required

**Request headers**:
```
Idempotency-Key: <uuid> (required)
```

**Request body**:
```json
{
  "yachtId": "string (required)",
  "checkIn": "date (required)",
  "checkOut": "date (required)",
  "crewOption": "CrewOption (required)",
  "guestCount": "integer (required)",
  "stripePaymentMethodId": "string (required — from Stripe Elements)"
}
```

**Response `201`**:
```json
{
  "id": "string",
  "status": "CONFIRMED",
  "totalPriceCents": 225000,
  "platformFeeCents": 29250,
  "ownerPayoutCents": 195750,
  "stripePaymentIntentId": "string",
  "yacht": { "id": "string", "name": "string" },
  "checkIn": "2026-07-01",
  "checkOut": "2026-07-06",
  "createdAt": "timestamp"
}
```

**Errors**: `409` dates no longer available | `409` below minimum charter duration | `402` payment failed — transaction rolled back, no reservation created | `422` validation error | `409` idempotency key already used (returns original response)

**Notes**:
- Entire booking creation + payment capture in a single DB transaction with `SELECT FOR UPDATE`
- `Stripe.paymentIntents.create({ confirm: true })` called inside the transaction — on Stripe failure, ROLLBACK is issued and `402` is returned
- `status` goes directly to `CONFIRMED` (instant book — PENDING is unreachable in the MVP booking path)
- After commit: confirmation emails sent synchronously via EmailPort (try-catch — email failures are logged but do not fail the booking)

---

### `GET /bookings`
**Auth**: JWT

**Query params**:
```
status    ReservationStatus (optional filter)
page      integer (default: 1)
limit     integer (default: 20)
```

**Response `200`**:
```json
{
  "data": [
    {
      "id": "string",
      "status": "CONFIRMED",
      "checkIn": "2026-07-01",
      "checkOut": "2026-07-06",
      "crewOption": "SKIPPERED",
      "totalPriceCents": 225000,
      "yacht": {
        "id": "string",
        "name": "string",
        "coverPhotoUrl": "string",
        "homePort": "string"
      },
      "createdAt": "timestamp"
    }
  ],
  "meta": { "total": 5, "page": 1, "limit": 20 }
}
```

**Notes**: Returns bookings where user is renter OR owner depending on role context. Owner panel passes `?role=owner` to get owner-side bookings.

---

### `GET /bookings/:id`
**Auth**: JWT (renter who made the booking OR owner of the yacht)

**Response `200`**:
```json
{
  "id": "string",
  "status": "CONFIRMED",
  "checkIn": "2026-07-01",
  "checkOut": "2026-07-06",
  "crewOption": "SKIPPERED",
  "guestCount": 4,
  "totalPriceCents": 225000,
  "platformFeeCents": 29250,
  "ownerPayoutCents": 195750,
  "cancellationPolicySnapshot": { ... },
  "yacht": { "id": "string", "name": "string", "type": "string", "homePort": "string", "coverPhotoUrl": "string" },
  "renter": { "firstName": "string", "email": "string", "phone": "string" },
  "owner": { "firstName": "string", "email": "string", "phone": "string" },
  "payment": { "status": "CAPTURED", "stripePaymentIntentId": "string" },
  "createdAt": "timestamp"
}
```

**Notes**: Both renter and owner contact details are included (post-booking only). Renter sees owner contact, owner sees renter contact.

---

### `PATCH /bookings/:id/cancel`
**Auth**: JWT (renter who made the booking OR owner of the yacht)

**Request body**:
```json
{ "reason": "string (optional)" }
```

**Response `200`**:
```json
{
  "status": "CANCELLED",
  "refundAmountCents": 112500,
  "refundPolicy": "50% refund — cancelled 16 days before charter"
}
```

**Errors**: `409` booking already cancelled or completed | `409` version conflict (retry)

**Notes**:
- Refund calculated from `cancellationPolicySnapshot` (not current platform policy)
- Platform commission never refunded
- Stripe refund issued synchronously via `stripe.refunds.create()` inside the cancel handler
- Availability released back to `AVAILABLE` (PREP days also released)

---

### `PATCH /bookings/:id/check-in`
**Auth**: JWT (OWNER of the yacht)

**Response `200`**: `{ "status": "CHECKED_IN" }`

**Errors**: `409` booking not in CONFIRMED status | `409` check-in date is in the future

---

### `PATCH /bookings/:id/complete`
**Auth**: JWT (OWNER of the yacht)

**Response `200`**: `{ "status": "COMPLETED" }`

**Errors**: `409` booking not in CHECKED_IN status | `409` check-out date is in the future

**Notes**: Triggers payout release to owner. Payout is also auto-released 24–48h after check-in via scheduler — this manual action is a secondary trigger and creates an audit event.

---

## 6. Payments

---

### `GET /payments`
**Auth**: JWT (OWNER role)

**Query params**:
```
page    integer (default: 1)
limit   integer (default: 20)
```

**Response `200`**:
```json
{
  "data": [
    {
      "id": "string",
      "paymentType": "DEPOSIT",
      "status": "CAPTURED",
      "ownerAmountCents": 195750,
      "stripeTransferId": "string | null",
      "reservation": {
        "id": "string",
        "checkIn": "2026-07-01",
        "checkOut": "2026-07-06",
        "yacht": { "name": "string" }
      },
      "createdAt": "timestamp"
    }
  ],
  "meta": { "total": 12, "page": 1, "limit": 20 },
  "summary": {
    "totalEarnedCents": 1245000,
    "thisMonthCents": 390000
  }
}
```

---

### `POST /payments/webhook`
**Auth**: Stripe signature header (`Stripe-Signature`)

**Request body**: Stripe webhook event payload (raw)

**Response `200`**: `{ "received": true }`

**Notes**: Signature verified via `stripe.webhooks.constructEvent()` before processing. Events handled: `payment_intent.succeeded`, `payment_intent.payment_failed`, `account.updated` (Stripe Connect), `transfer.created`.

---

## 7. Stripe Connect

---

### `POST /stripe/connect/onboard`
**Auth**: JWT (OWNER role)

**Response `200`**:
```json
{
  "onboardingUrl": "string (Stripe-hosted Express onboarding URL)",
  "stripeAccountId": "string"
}
```

**Notes**: Creates Stripe Account v2 with `fees_collector: application`, `losses_collector: application`, `dashboard: express`. Stores `stripeAccountId` on user. Returns the Stripe-hosted onboarding URL to redirect the owner.

---

### `GET /stripe/connect/status`
**Auth**: JWT (OWNER role)

**Response `200`**:
```json
{
  "stripeAccountId": "string",
  "status": "PENDING | ACTIVE | RESTRICTED",
  "onboardingComplete": true
}
```

---

### `GET /stripe/connect/dashboard-link`
**Auth**: JWT (OWNER role)

**Response `200`**:
```json
{ "url": "string (Stripe Express Dashboard login link, expires in 60s)" }
```

---

## 8. Admin

All admin endpoints require `ADMIN` role. Prefix: `/admin/`.

### Phase 1 — MVP (soft launch)

Four endpoints implemented at Milestone 6. Schema fields for all deferred endpoints are already in place.

---

#### `GET /admin/users`
**Query params**: `search`, `role`, `status`, `page`, `limit`

**Response `200`**: Paginated list of users with `id`, `firstName`, `lastName`, `email`, `roles`, `status`, `createdAt`.

---

#### `PATCH /admin/users/:id/suspend`
**Request body**: `{ "reason": "string (required)" }`
**Response `200`**: `{ "status": "SUSPENDED" }`
**Notes**: Sets all owner's listings to `PAUSED`.

---

#### `GET /admin/bookings`
**Query params**: `search`, `status`, `checkInFrom`, `checkInTo`, `page`, `limit`
**Response `200`**: Paginated bookings with renter, owner, yacht, financials including `platformFeeCents`.

---

#### `POST /admin/bookings/:id/refund`
**Request body**:
```json
{
  "amountCents": "integer (required, max = totalPriceCents)",
  "reason": "string (required)"
}
```
**Response `200`**: `{ "stripeRefundId": "string", "amountCents": 112500 }`
**Notes**: If payout already transferred to owner, clawback must be handled manually — response includes warning flag `{ "ownerPayoutAlreadySent": true }`.

---

### Deferred to post-launch

The following 12 endpoints are designed and documented here but not implemented at MVP. Schema fields are already in place for all of them.

| Endpoint | Description |
|---|---|
| `GET /admin/users/:id` | Full user profile with Stripe status, booking counts, spend/earned |
| `PATCH /admin/users/:id/reactivate` | Reactivate suspended user (listings stay PAUSED) |
| `PATCH /admin/users/:id/commission` | Set per-owner commission rate override |
| `GET /admin/yachts` | Paginated listings with owner info and booking count |
| `GET /admin/yachts/:id` | Full listing detail with photos, owner, booking history |
| `PATCH /admin/yachts/:id/status` | Pause or reactivate listing, notify owner by email |
| `DELETE /admin/yachts/:id` | Delete listing (blocked if confirmed future bookings) |
| `GET /admin/bookings/:id` | Full booking detail with event log, payment records, Stripe IDs |
| `POST /admin/bookings/:id/trigger-payout` | Manually trigger owner payout release |
| `GET /admin/revenue` | Commission summary + monthly breakdown |
| `GET /admin/settings` | Platform commission rate + cancellation policy thresholds |
| `PATCH /admin/settings` | Update commission rate, cancellation policy, platform name |

---

## Endpoint Summary

| Module | Endpoints | Total |
|---|---|---|
| Auth | register, verify-email, login, refresh, logout, forgot-password, reset-password | 7 |
| Users | me (GET/PATCH), add-owner-role, profile-photo/upload-url | 4 |
| Yachts | list, create, get, update, status, delete, pricing (GET/PATCH), configuration, photos (upload-url, register, reorder, delete) | 13 |
| Availability | get, block, unblock | 3 |
| Bookings | create, list, get, cancel, check-in, complete | 6 |
| Payments | list, webhook | 2 |
| Stripe Connect | onboard, status, dashboard-link | 3 |
| Admin (Phase 1) | list users, suspend user, list bookings, refund booking | 4 |
| Admin (deferred) | +12 post-launch endpoints — see §8 above | — |
| **Total (MVP)** | | **42** |

---

## Related Pages

- [[specs/data-model]]
- [[specs/backend-architecture]]
- [[specs/user-stories/renter]]
- [[specs/user-stories/owner]]
- [[specs/user-stories/admin]]
