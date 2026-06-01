---
title: Phase 1 — Development Roadmap
tags:
  - roadmap
  - development
last-updated: 2026-06-01
---

# Phase 1 — Development Roadmap (MVP)

> **Backend progress:** **Phase 1 backend ✅ COMPLETE — M1–M7 (all 7 milestones).** 156 e2e + 20 unit tests green; hardened (rate limiting, Helmet, CORS, audit). Remaining M7 items are operational (manual deploy steps captured in `yachties-backend/docs/launch-checklist.md`). Frontend not started — separate track.

**Summary**: Build order for the YachtBay MVP — 7 milestones from project scaffolding to launch-ready. Each milestone produces a testable, shippable increment.

**Principle**: Backend first, frontend follows. Never build a frontend feature before the API it depends on is complete and tested. Each milestone ends with a clear "what you can now test" checkpoint.

---

## Milestone 1 — Foundation
*Goal: Three deployed apps, auth working end-to-end*

### Backend
- [x] NestJS project initialized with Fastify adapter
- [x] Prisma configured, PostgreSQL connected, first migration run (full schema from [[specs/data-model]])
- [x] Global pipes (ValidationPipe), guards, and error filter wired up
- [x] Auth module: `register`, `verify-email`, `login`, `refresh`, `logout`, `forgot-password`, `reset-password`
- [x] Users module: `GET /users/me`, `PATCH /users/me`, `add-owner-role`
- [x] JWT strategy + `JwtAuthGuard` + `RolesGuard`
- [x] `.env.example` committed

### Frontend
- [ ] Turborepo monorepo initialized (`apps/backend`, `apps/storefront`, `apps/owner-panel`, `packages/ui`, `packages/types`, `packages/api-client`)
- [ ] Storefront: React Router 7 SSR initialized, basic layout shell
- [ ] Storefront: auth pages (`/register`, `/login`, `/forgot-password`, `/reset-password`, `/verify-email`)
- [ ] Owner panel: React Router 7 SPA initialized, basic layout shell
- [ ] Owner panel: auth pages (`/login`, `/register`)
- [ ] Shared `api-client` package with auth endpoints wired up

### Infrastructure
- [ ] GitHub repo created, monorepo pushed
- [ ] Railway: 3 services created (backend, storefront, owner-panel)
- [ ] Railway: PostgreSQL provisioned and connected to backend (Redis deferred to Milestone 4 — see [[decisions/2026-05-27-scoped-bullmq-usage]])
- [ ] GitHub Actions: CI workflow (typecheck + lint on PR)
- [ ] GitHub Actions: deploy to Railway on push to `main`
- [ ] Domains configured: `api.yachtbay.com`, `yachtbay.com`, `owners.yachtbay.com`

### ✅ Milestone 1 checkpoint
> Can register → verify email → log in → view profile → log out → reset password on both storefront and owner panel. Three apps deployed and auto-deploying from GitHub.

---

## Milestone 2 — Listings
*Goal: Owner can create, configure, and publish a yacht listing*

### Backend
- [x] Yachts module: `POST /yachts`, `GET /yachts/:id`, `PATCH /yachts/:id`, `PATCH /yachts/:id/status`, `DELETE /yachts/:id`
- [x] Slug generation on listing creation
- [x] Yacht photos: pre-signed upload URL flow (`StoragePort`), `POST /yachts/:id/photos`, reorder, delete
- [x] Cloudflare R2 adapter wired up (`R2Adapter` implementing `StoragePort`)
- [x] Pricing configuration: `PATCH /yachts/:id/pricing` — creates `pricing_rules` for crew options + weekly rate
- [x] Configuration: `PATCH /yachts/:id/configuration` — crew options, charter types, prep days

### Frontend (Owner Panel)
- [ ] `/listings` — all listings page
- [ ] `/listings/new/*` — 6-step creation wizard:
  - Step 1: Basic details + Mapbox location picker
  - Step 2: Photo upload (drag-and-drop, R2 direct upload)
  - Step 3: Pricing
  - Step 4: Crew options + charter types
  - Step 5: Availability settings (prep days, min duration)
  - Step 6: Review & publish (blocked until Stripe active — shows prompt)
- [ ] `/listings/:id` — listing overview
- [ ] `/listings/:id/edit`, `/listings/:id/photos`, `/listings/:id/pricing`, `/listings/:id/configuration`

### ✅ Milestone 2 checkpoint
> Owner can create a full listing (details, photos, pricing, crew options), save as draft, and attempt to publish (blocked by Stripe — connects in Milestone 4). Listing detail visible via `GET /yachts/:id`.

---

## Milestone 3 — Discovery
*Goal: Renter can search and find yachts on the storefront*

### Backend
- [x] Availability module: `GET /yachts/:id/availability`, `POST /yachts/:id/availability/block`, `DELETE /yachts/:id/availability/block`
- [x] PREP day auto-blocking helper `applyPrepDays` (written now, triggered in Milestone 4)
- [x] Search module: `GET /yachts` with haversine geo filter (`$queryRaw`), date availability join, crew option filter, price filter, sort, pagination, per-result `calculatedPriceCents`
- [x] `GET /yachts/:id/pricing` — PricingEngine with `BasePriceStrategy`, `WeeklyRateStrategy`, `CrewOptionStrategy`

### Frontend (Storefront)
- [ ] Homepage (`/`) — hero with search bar, featured listings section, how it works, trust signals
- [ ] Search page (`/search`) — listing cards + Mapbox map split view, filters, sort, pagination
- [ ] Yacht detail page (`/yachts/:id`) — photo gallery, specs, booking widget with dynamic price preview, availability calendar, owner card, cancellation policy
- [ ] Shared `ui` package: Button, Input, Card, Badge, DatePicker components

### Frontend (Owner Panel)
- [ ] `/listings/:id/availability` — availability calendar with date blocking UI

### ✅ Milestone 3 checkpoint
> End-to-end discovery flow: owner creates listing → renter searches by location + dates → finds the listing → views full detail page with live price calculation. Map pins show yacht home ports.

---

## Milestone 4 — Booking Core
*Goal: Renter can instant-book a yacht and payment is processed*

### Backend
- [x] Stripe Connect module: `POST /stripe/connect/onboard`, `GET /stripe/connect/status`, `GET /stripe/connect/dashboard-link`
- [x] Stripe webhook handler: `account.updated` → updates `stripe_account_status` on user
- [x] Redis + BullMQ provisioned (payout queue only) — lazy queue, no Redis required at boot
- [x] BullMQ worker: payout-queue (separate `src/worker.ts` entrypoint; trigger wires up in M5 check-in)
- [x] Resend adapter wired up (`ResendAdapter` implementing `EmailPort`) — also used for booking confirmation emails
- [x] Bookings module: `POST /bookings` (CreateBookingHandler — Yacht-row `SELECT FOR UPDATE` + reserve + commit + **capture after commit**; client-side SCA; compensation on capture failure → 402; idempotency via `Idempotency-Key`) *(per amended `decisions/2026-05-27-sync-payment-capture`)*
- [x] Bookings module: `GET /bookings`, `GET /bookings/:id`
- [ ] Stripe webhook handler: `payment_intent.payment_failed`, `charge.refunded` (secondary path) — *deferred to M5*
- [x] Confirmation emails: renter + owner (sent synchronously post-commit, best-effort)

### Frontend (Storefront)
- [ ] `/yachts/:id/checkout` — booking summary page
- [ ] `/yachts/:id/checkout/pay` — Stripe Elements payment form
- [ ] `/bookings/:id/confirmed` — booking confirmation page
- [ ] Auth redirect flow: unauthenticated renter hitting checkout → login → return to checkout

### Frontend (Owner Panel)
- [ ] `/settings/stripe` — Stripe Connect status + onboarding CTA
- [ ] Stripe onboarding button → redirect to Stripe → return with success message
- [ ] Publish listing now unblocked once Stripe active

### ✅ Milestone 4 checkpoint
> Full booking flow: owner connects Stripe → publishes listing → renter searches → finds yacht → instant books → payment captured → renter gets confirmation email → owner gets notification email → availability locked.

---

## Milestone 5 — Operations
*Goal: Full charter lifecycle, cancellations, payouts, and both dashboards working*

### Backend
- [x] Bookings module: `PATCH /bookings/:id/cancel` — `CancelBookingHandler` with pre-transfer guard, version-locked transition, refund from `cancellationPolicySnapshot` (idempotency-keyed to version), availability + PREP released, payout dequeued
- [x] Bookings module: `PATCH /bookings/:id/check-in`, `PATCH /bookings/:id/complete` (version-locked, date guards)
- [x] Payments module: `GET /payments` (owner payout history + `totalEarnedCents`/`thisMonthCents` summary)
- [x] Check-in handler enqueues payout-queue delayed job (`PAYOUT_DELAY_HOURS` env, default 24)
- [x] Cancellation emails: renter + owner via `emailPort.send()` (best-effort post-commit)
- [x] Completion emails: owner + renter via `emailPort.send()` (best-effort post-commit)
- [x] M4 backfill: `applyPrepDays(yachtId, checkOut, reservationId)` fires on confirmation; secondary webhooks (`transfer.created`, `payment_intent.payment_failed`) wired idempotently

### Frontend (Storefront)
- [ ] `/bookings` — renter bookings dashboard (Upcoming / Past / Cancelled tabs)
- [ ] `/bookings/:id` — booking detail with cancel action + refund preview modal
- [ ] `/profile` — profile edit, switch to owner panel link

### Frontend (Owner Panel)
- [ ] `/dashboard` — stats, upcoming bookings, Stripe status banner, empty state
- [ ] `/bookings` — all bookings across listings (tabs by status)
- [ ] `/bookings/:id` — booking detail with Check In / Mark Complete actions
- [ ] `/payouts` — payout history + earnings summary + Stripe dashboard link
- [ ] `/profile` — owner profile edit

### ✅ Milestone 5 checkpoint
> Full charter lifecycle testable end-to-end: book → check-in → complete → payout released. Cancellation with correct refund amount. Both dashboards functional. Renter and owner can manage their bookings.

---

## Milestone 6 — Admin
*Goal: Admin panel operational, platform manageable without code deploys*

### Backend
- [x] Admin module: 4 MVP endpoints from [[specs/api-contract]] §8 (Phase 1)
  - [x] `GET /admin/users` — list/search users (search, role, status filters)
  - [x] `PATCH /admin/users/:id/suspend` — emergency suspension (required reason)
  - [x] `GET /admin/bookings` — list all bookings (status filter + pagination)
  - [x] `POST /admin/bookings/:id/refund` — issue manual refund (auto-clawback via `reverse_transfer` when payout already sent; `ownerPayoutAlreadySent` warning flag)
- [x] Admin user provisioning: `npm run seed:admin -- <email>` CLI (no public admin registration)

### Frontend (Admin Panel)
- [ ] Admin panel: React Router 7 SPA initialized, sidebar layout
- [ ] `/login` — admin login
- [ ] `/dashboard` — platform overview (stats tiles + recent activity)
- [ ] `/users` — user list with search + suspend action
- [ ] `/bookings` — booking list with manual refund action

### ✅ Milestone 6 checkpoint
> Admin can view all users, suspend a user, view all bookings, and issue a manual refund. Full admin panel (revenue, settings, commission management) deferred to post-traction.

---

## Milestone 7 — Polish & Launch
*Goal: Production-ready, publicly launchable*

### Remaining pages
- [ ] Storefront: `/how-it-works`, `/cancellation-policy`, `/terms`, `/privacy`
- [ ] Owner panel: `/settings` (notification preferences)
- [ ] 404 and 500 error pages (all three apps)

### Quality & security
- [x] Rate limiting applied (NestJS Throttler — global 100/min/IP + strict 10/min on auth + booking writes; in-memory at MVP, swap to Redis later)
- [x] CORS configured (env-driven in `main.ts` with `credentials: true`; operator must set the three URL env vars to prod domains in Railway)
- [x] Helmet headers configured (`@fastify/helmet@^11` — pinned for Fastify 4 compat; HSTS / X-Content-Type-Options / X-Frame-Options confirmed)
- [x] Input validation across all endpoints (class-validator DTOs + global `ValidationPipe { whitelist, forbidNonWhitelisted, transform }`)
- [x] Stripe webhook signature verification confirmed (`PaymentPort.constructWebhookEvent` → `stripe.webhooks.constructEvent`)
- [x] R2 pre-signed URL expiry confirmed (10-min default); bucket CORS is operator-side (in `docs/launch-checklist.md`)

### Performance
- [ ] Critical DB queries reviewed against indexes
- [ ] `yacht_availability` availability queries profiled for expected load
- [ ] Image lazy loading on listing cards and detail pages

### Testing
- [x] Unit tests: PricingEngine strategies (5 specs, M3) + `computeRefund` (5 specs, M5; consumes the policy snapshot) + e2e for CreateBooking (M4b)
- [x] Integration tests: booking creation availability lock + double-booking prevention (real-Postgres concurrency e2e in `bookings.e2e-spec`)
- [x] Manual E2E smoke flow: register → search → book → cancel (documented in `docs/launch-checklist.md` Pre-flight smoke test section; Playwright deferred)

### Launch
- [ ] Production environment variables set in Railway
- [ ] Custom domains verified and SSL active
- [ ] Stripe account in live mode (not test mode)
- [ ] Soft launch: test with real owners and renters (friends & family)
- [ ] Monitor: Railway logs, Stripe dashboard, error rates

### ✅ Milestone 7 checkpoint
> Platform is live. Real bookings can be made. Admin can manage the platform. Ready for public launch.

---

## Build Order Summary

```
M1: Foundation ──────── Auth + Users + 3 apps deployed
      ↓
M2: Listings ─────────── Yacht CRUD + photos + pricing (owner panel)
      ↓
M3: Discovery ────────── Availability + Search + Storefront (read-only)
      ↓
M4: Booking Core ──────── Stripe Connect + Bookings + Checkout flow
      ↓
M5: Operations ───────── Cancellations + Payouts + Both dashboards
      ↓
M6: Admin ────────────── Admin panel + platform management
      ↓
M7: Polish & Launch ──── Edge cases + security + testing + go live
```

---

## Phase 2 Preview (post-traction)

Once the platform has real bookings and revenue, Phase 2 adds:
- Reviews and ratings system
- In-platform messaging
- Owner-defined cancellation policies (Flexible / Moderate / Strict)
- Seasonal and custom pricing rules (owner-facing UI)
- Owner analytics dashboard
- Mobile apps (iOS + Android)
- Promoted listings (revenue stream)

See [[roadmap/overview]] for Phase 2 and Phase 3 detail.

---

## Related Pages

- [[specs/data-model]]
- [[specs/api-contract]]
- [[specs/mvp-scope]]
- [[specs/user-stories/renter]]
- [[specs/user-stories/owner]]
- [[specs/user-stories/admin]]
