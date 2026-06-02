---
title: Wiki Log
tags:
  - meta
last-updated: 2026-06-03
---

# Wiki Log

Append-only record of all vault operations. Never delete or edit past entries.

---

## 2026-06-03 — Full-stack monorepo migration ✅ COMPLETE

**Action**: Executed the full-stack-monorepo migration via subagent-driven development. 10 tasks (skeleton → backend move → test verification → packages → frontend shells → turbo orchestration → root CLAUDE.md → final gate) all green in a single session. The standalone `yachties-backend/` is now ready to be archived; all future development lives in `/Users/philipposphilias/Desktop/Yacht Platfrom/yachtbay/`.

**Monorepo layout (final):**
```
yachtbay/
├── apps/api/         @yachtbay/api — backend, 156 e2e + 20 unit passing
├── apps/storefront/  @yachtbay/storefront — RR7 SSR shell (smoke import of Yacht from @yachtbay/types)
├── apps/panel/       @yachtbay/panel — RR7 SPA shell (ssr:false confirmed in build log), index + /admin placeholder
├── packages/types/   @yachtbay/types — re-exports Prisma model/enum types from @prisma/client; DTOs via tsconfig paths into apps/api/src
└── packages/api-client/ @yachtbay/api-client — typed fetch wrapper with credentials:include + ApiError
```

**Tooling locked-in:**
- pnpm 9.15.0 (pinned via `packageManager` + corepack)
- Turborepo 2.x (cold build 5.4s; warm cache "FULL TURBO" in 34ms)
- Node 20 (`.nvmrc`)
- TypeScript 5 with monorepo-base config; `apps/api/tsconfig.json` overrides `module: commonjs`, `moduleResolution: node`, `noImplicitOverride: false`, `noUncheckedIndexedAccess: false` for NestJS compatibility

**Cross-stack type bridge proven:**
- `packages/types/src/index.ts` re-exports Prisma model/enum types directly from `@prisma/client` (added as a `@yachtbay/types` direct dep — pnpm dedups via virtual store).
- DTO types resolved via tsconfig `paths` mapping `"@yachtbay/api/*": ["../../apps/api/src/*"]` — direct source-file imports, no codegen, no publish step.
- Frontend smoke imports (`import type { Yacht } from '@yachtbay/types'` in storefront; `import type { Role }` in panel) typecheck and build cleanly.

**Acceptance gate cleared:**
- Clean install (`pnpm install --frozen-lockfile`): success — lockfile already in sync, no drift.
- Full build (3 build tasks: api, storefront, panel; types and api-client have no build script): all green in 5.4s.
- Unit tests: 20/20 passing (0.81s).
- E2e tests: 156/156 passing across 12 suites (34.6s) — same exact suite that was green in the standalone repo. No feature code changed, no test code changed.

**Vault updated:**
- `decisions/2026-06-03-full-stack-monorepo.md` (the ADR locking this) — already written before the migration started.
- `decisions/2026-05-03-frontend-monorepo.md` — marked superseded.
- `wiki/hot.md` — Current Focus rewritten to mark migration complete; next track is the admin section of `apps/panel/`.

**Architecture refinement from execution:**
- The plan's suggested route for Prisma type imports (`@yachtbay/api/node_modules/@prisma/client`) was wrong — pnpm doesn't nest @prisma/client under apps/api/node_modules; it lives in the virtual store. The cleaner solution (`@prisma/client` as a direct dep of @yachtbay/types) was adopted instead.
- The `noUncheckedIndexedAccess: false` override on `apps/api/tsconfig.json` is a minor pre-existing condition (backend code wasn't written against this strictness). Tracked as future tech debt.

**Operator-only follow-up** (not agent-executable):
- Archive `yachties-backend/` with a final git tag `v1.0-archived`.
- Retarget the auto-push background process at `yachtbay/.git/`.
- Create the GitHub remote for the new monorepo (`gh repo create yachtbay --private --source=. --remote=origin --push`).
- Reconfigure Railway: three services, each with root dir + watch paths (`apps/api/**`, `apps/storefront/**`, `apps/panel/**` + shared `packages/**`).
- Update `apps/api/docs/launch-checklist.md` for the new paths.

**Source**: migration plan at `apps/api/docs/superpowers/plans/2026-06-03-monorepo-migration.md` (which traveled with the backend during Task 2 of the migration). 7 commits in the monorepo: `4890a47`, `e0fc72e`, `5e69634`, `1fb8f80`, `272be84`, `378cf4e`, `3c339a5`.

---

## 2026-06-03 — Full-stack monorepo decision; `yachties-backend` to be archived

**Action**: Switched the post-Phase-1 trajectory from "two separate frontend apps + standalone backend repo" to a **single full-stack monorepo** at `/Users/philipposphilias/Desktop/Yacht Platfrom/yachtbay/`. This is a substantial supersedure of the 2026-05-03 frontend-monorepo ADR, driven by two findings from this session:

1. **Owner panel and admin panel do not need to be separate apps.** The 2026-05-03 ADR's "fundamentally different UX goals" argument was about storefront (consumer, SSR, SEO) vs owner-panel (operational, SPA, behind auth). It does NOT extend to owner vs admin — both are auth-gated, data-dense SPAs with the same rendering mode, auth flow, and design language. Splitting them duplicates layout, nav, and design system code with no UX payoff. Inside a single `apps/panel/`, RR7 route-level role gates (`@Roles(OWNER)` vs `@Roles(ADMIN)`) plus route-level code splitting give the same isolation a separate app would, without the duplication.

2. **Backend-in-monorepo enables direct cross-workspace TypeScript imports.** With backend living in `apps/api/`, `packages/types/` becomes a trivial re-export of Prisma model types and DTOs. A field rename on the backend instantly breaks `tsc` in every frontend file that uses the old name — same PR, same CI run. The three alternatives (private-npm publish ritual, OpenAPI codegen, hand-mirrored types) are all strictly worse, and the type-drift tax compounds weekly for a solo dev iterating both halves.

**Decisions locked:**
- Repo name: **yachtbay**
- Path: **/Users/philipposphilias/Desktop/Yacht Platfrom/yachtbay/**
- Git history: **archive `yachties-backend` (final tag, read-only) and start the monorepo with a clean initial commit** — preserves Phase 1's documented record in the vault, avoids `git subtree`/`filter-branch` surgery for a solo-project marginal benefit.
- Workspaces: `apps/api/`, `apps/storefront/`, `apps/panel/`, `packages/types/`, `packages/api-client/`. **No** `packages/ui/` or `packages/utils/` upfront — YAGNI.
- Railway: three services, each with its own root directory and watch-path config.

**Added in vault:**
- `decisions/2026-06-03-full-stack-monorepo.md` — new ADR (this decision).
- `decisions/2026-05-03-frontend-monorepo.md` — marked **superseded** with a callout pointing at the new ADR; original body preserved for history.
- `wiki/hot.md` — Current Focus and Next Steps rewritten: monorepo migration is #1, admin section of panel #2, owner section #3, storefront #4, operational launch parallel.

**Amended in the backend repo:**
- `yachties-backend/CLAUDE.md` — archive notice at the top pointing at the monorepo path and the new ADR; "Next track" line corrected (panel app covers owner + admin, not three separate apps).

**Migration plan**: written to `yachties-backend/docs/superpowers/plans/2026-06-03-monorepo-migration.md` (will be carried into the monorepo as `apps/api/docs/superpowers/plans/...` during the migration itself, then archived in the standalone backend repo).

**No backend code changed.** The 156 e2e + 20 unit tests must pass unchanged in their new home — that is the acceptance gate for the migration.

**Source**: this session's conversation; full-stack-monorepo ADR.

---

## 2026-06-01 — Milestone 7 (Polish & Launch — backend code) implemented; Phase 1 backend complete

**Action**: Hardened the backend for production: rate limiting, security headers, audit pass, and a launch checklist. With this, **all 7 Phase-1 backend milestones are complete**. Remaining M7 work is operational (Railway env, domains, Stripe live keys, soft launch) and captured in the checklist.

**Added (in the backend repo, not the vault):**

*Rate limiting:*
- Installed `@nestjs/throttler` (v6).
- Global guard via `APP_GUARD` so the throttler runs BEFORE the JWT/Roles guards. Lenient default (100 req/min/IP) keeps M1–M6 traffic unaffected.
- Strict 10 req/min/IP on `AuthController` (class-level) and on `POST /bookings/payment-intent` + `POST /bookings` (handler-level). Matches the api-contract's per-endpoint rate-limit notes.
- In-memory storage at MVP (single Railway instance); flip to Redis-backed in `ThrottlerModule.forRoot` config alone when scaling out.
- Test isolation: `db-reset` now also clears the throttler's in-memory map between tests (otherwise 28 booking tests × 2 logins each would blow the 10/min bucket).

*Security headers:*
- `@fastify/helmet` registered in `main.ts` with `contentSecurityPolicy: false` (the backend serves JSON only — frontends set their own CSP). HSTS, X-Content-Type-Options, X-Frame-Options all confirmed live in the boot smoke test.
- **Pinned to `@fastify/helmet@^11`** for Fastify 4 compatibility — `@nestjs/platform-fastify` bundles Fastify 4.x, while helmet v12+ requires Fastify 5 and crashes prod boot with `FST_ERR_PLUGIN_VERSION_MISMATCH`. The e2e suite couldn't catch this because tests bypass `main.ts`; the boot smoke did.

*Audit (no code change, all clean):*
- CORS env-driven (`STOREFRONT_URL`/`OWNER_PANEL_URL`/`ADMIN_PANEL_URL`) with `credentials: true` — correct for httpOnly cookies.
- Stripe webhook signature verification routes through `PaymentPort.constructWebhookEvent` → `stripe.webhooks.constructEvent(payload, signature, secret)` — invalid signature → 400.
- R2 pre-signed URL default expiry 600s (10 minutes) — within threshold.

*Launch artefact:*
- `docs/launch-checklist.md` — operator's pre-flight: Railway env, Stripe live mode, custom domains, R2 bucket CORS, monitoring, pre-flight smoke flow, rollback plan.

**Tests**: 156 e2e + 20 unit, all green. M7 added a small `throttler.e2e-spec.ts` (3 tests: global default tolerates normal traffic; 11th login = 429; 11th payment-intent = 429 — confirming the throttler runs BEFORE the JWT guard).

**Architecture realization:** Direction B respected throughout — no Redis-backed throttler at MVP, no inflated DTOs, no premature CSP customisation, no Playwright (manual smoke in the checklist is the MVP-grade equivalent).

**No spec/ADR amendments needed.**

**Source**: M7 plan at `yachties-backend/docs/superpowers/plans/2026-06-01-milestone-7-polish-launch.md`; launch checklist at `yachties-backend/docs/launch-checklist.md`.

---

## 2026-06-01 — Milestone 6 (Admin) backend implemented

**Action**: Implemented M6 backend — the 4 admin endpoints from the minimal-admin ADR — via subagent-driven TDD. Plan written directly without a separate stress-test (M6 was small and well-specified).

**Added (in the backend repo, not the vault):**

*Foundations:*
- `PaymentPort.createRefund` extended with optional `reverseTransfer?: boolean` — when set, Stripe passes `reverse_transfer: true` to claw funds back from the connected account. Backward-compatible (M5 cancellation path unchanged).
- `prisma/seed-admin.ts` CLI — `npm run seed:admin -- <email>` promotes an existing user to `ADMIN` (idempotent on re-runs). No public admin registration.
- `createAdminAndLogin` test helper.

*Admin module (1 controller + 3 services + 4 DTOs):*
- **`GET /admin/users`** — paginated; case-insensitive search across email/name; filters by `role` (Prisma `{ has }`) and `status`. `passwordHash` excluded.
- **`PATCH /admin/users/:id/suspend`** — required `reason` (1–500 chars); sets `status=SUSPENDED` + `suspensionReason`.
- **`GET /admin/bookings`** — all bookings paginated, optional `?status` filter; joins yacht + renter.
- **`POST /admin/bookings/:id/refund`** — admin overrides the cancellation policy. Refundable-balance check (422 if `amountCents > deposit.amountCents - sum(prior REFUNDs)`). **Auto-clawback**: when `Payment.stripeTransferId` is set, passes `reverseTransfer: true` so Stripe reverses the connected-account transfer. Writes `REFUND` Payment + `ReservationEvent {eventType:'admin_refund_issued', adminId}` atomically. Response includes `ownerPayoutAlreadySent` warning.

`@Roles(Role.ADMIN)` applied **at the controller class level** — the existing `RolesGuard` reads both handler-level and class-level metadata via `getAllAndOverride`, so non-admins get 403 before any handler runs.

**Tests**: 153 e2e + 20 unit, all green. The admin refund e2e covers the no-clawback path, the auto-clawback path (asserts the fake's `refunds.at(-1).reverseTransfer === true`), the over-amount 422, and the non-admin 403.

**No spec/ADR amendments needed** — M6 implements exactly the 4 endpoints scoped in [[decisions/2026-05-27-minimal-admin-mvp]].

**Source**: M6 plan at `yachties-backend/docs/superpowers/plans/2026-06-01-milestone-6-admin.md`.

---

## 2026-06-01 — Milestone 5 (Operations) backend implemented

**Action**: Implemented M5 backend — booking state machine + payout history + secondary webhook events — via subagent-driven TDD. Closes the full charter lifecycle and clears the two known M4 loose ends.

**Added (in the backend repo, not the vault):**

*Booking state machine (three new plain command-handler classes per Direction B):*
- **`CancelBookingHandler`** — `PATCH /bookings/:id/cancel` (renter or owner). Pre-transfer guard (if `Payment.stripeTransferId` is set → 409, defer to M6 admin clawback). Refund computed from the **snapshotted policy** (`dayTrip` tier when charter length = 1, else `multiDay`; commission non-refundable). Stripe refund **idempotency-keyed to `(reservationId, version)`** so concurrent retries dedupe at Stripe. Version-locked `CONFIRMED → CANCELLED`. `deleteMany WHERE reservationId` releases RESERVED + PREP atomically. Post-commit: `payoutQueue.removeRelease(id)` + cancellation emails.
- **`CheckInBookingHandler`** — `PATCH /bookings/:id/check-in` (owner-only). Version-locked `CONFIRMED → CHECKED_IN`; enqueues the payout job with `PAYOUT_DELAY_HOURS * 3600_000` (env-defaulted to 24) — wires up the M4 plumbing.
- **`CompleteBookingHandler`** — `PATCH /bookings/:id/complete` (owner-only). Version-locked `CHECKED_IN → COMPLETED` + completion emails.

*Supporting infra:*
- `PaymentPort.createRefund` (StripeAdapter + FakePaymentAdapter).
- `PayoutQueue.removeRelease(reservationId)` — best-effort, no-ops if the lazy queue hasn't been instantiated (keeps tests Redis-free).
- `applyPrepDays(yachtId, checkOut, reservationId?)` — backfilled with `reservationId`; `CreateBookingHandler` calls it post-capture so PREP rows are tagged and cancellation releases them in one go (closes the first M4 loose end).
- `computeRefund` pure helper (5 unit tests).
- Cancellation + completion email templates.
- `PAYOUT_DELAY_HOURS` Joi env (default 24).

*Operations endpoints:*
- **`GET /payments`** — owner's CAPTURED `DEPOSIT` payments + summary (`totalEarnedCents`, `thisMonthCents` UTC).
- **Webhook handlers added** — `transfer.created` sets `Payment.stripeTransferId` (idempotent on `stripeTransferId: null`); `payment_intent.payment_failed` flips `PENDING → FAILED` (idempotent on status). Closes the second M4 loose end.

**Tests**: 140 e2e + 20 unit, all green. The `computeRefund` helper has 5 unit tests covering both tier paths. `CancelBookingHandler` e2e covers: 50%-refund math from the snapshot, RESERVED+PREP released, pre-transfer 409, non-`CONFIRMED` 409, owner-cancel, stranger 404, refund + email + dequeue side-effects.

**Architecture realization:** every state transition uses optimistic concurrency via `prisma.reservation.updateMany({ where: { id, version }, data: { version: { increment: 1 }, status } })` → count 0 → 409. No `@nestjs/cqrs`; plain handler classes throughout.

**No spec/ADR amendments needed** — M5 lived entirely within Direction B's locked decisions.

**Source**: M5 plan at `yachties-backend/docs/superpowers/plans/2026-05-29-milestone-5-operations.md`; design one-pager at `yachties-backend/docs/ideas/m5-operations.md`.

---

## 2026-05-29 — Milestone 4 (Booking Core) backend implemented

**Action**: Implemented M4 backend in two plans (4a + 4b), each subagent-driven TDD. Realizes the amended sync-capture ADR (client-side SCA → capture after commit; separate charges + transfers; Yacht-row lock + `UNIQUE(yachtId,date)` safety net).

**Added (in the backend repo, not the vault):**

*M4a — Payments Foundation & Stripe Connect:*
- `PaymentPort` abstract (Connect + webhook surface) with a `StripeAdapter` (Express connected accounts, `transfers` capability)
- `POST /stripe/connect/onboard`, `GET /stripe/connect/status`, `GET /stripe/connect/dashboard-link`
- `POST /payments/webhook` — Fastify raw-body signature verification; `account.updated` → `User.stripeAccountStatus` (closes the loop on the M2 publish gate)
- `FakePaymentAdapter` so the test suite stays Stripe-free

*M4b — Bookings & Payout:*
- Extended `PaymentPort` with PaymentIntent create/retrieve/capture/cancel + transfer; `FakePaymentAdapter` gained the PI lifecycle + `failNextCapture`; new `FakeEmailAdapter` records sent messages
- `POST /bookings/payment-intent` — server-priced manual-capture PI for client-side SCA confirmation
- `CreateBookingHandler` — `SELECT FOR UPDATE` on the **Yacht row** → re-validate availability → reserve dates (`UNIQUE(yachtId,date)` is the race guard → 409) → commit → **capture after commit**. Compensation on capture failure (CANCELLED + dates released + PI voided + 402). Idempotency via the `Idempotency-Key` header.
- `GET /bookings` (renter + `?role=owner`) and `GET /bookings/:id` (contact details post-booking)
- Confirmation emails (renter + owner) via `EmailPort` (Resend), best-effort post-commit
- BullMQ payout-queue (**lazy** — Redis only on enqueue, which is M5), separate `src/worker.ts` entrypoint, `PayoutService.releasePayout()` transfer logic. `REDIS_URL` parsed into ioredis options so Railway's hosted Redis actually works.

**Tests**: 122 e2e + 15 unit, all green. Includes a real-Postgres **concurrency test** (two parallel bookings for the same dates → exactly one 201, one 409).

**Known loose ends carried to M5:** `applyPrepDays` is still not triggered on booking confirmation, and the secondary webhook events (`payment_intent.*` / `transfer.created`) are not routed yet.

**Architecture realization:** CQRS for bookings is implemented as plain command-handler classes (`CreateBookingHandler`), NOT `@nestjs/cqrs` — Direction B preferring the lighter shape; the command/query separation is what matters.

**Source**: M4 plans at `yachties-backend/docs/superpowers/plans/2026-05-29-milestone-4{a,b}-*.md`; design one-pager at `yachties-backend/docs/ideas/m4-booking-core.md`.

---

## 2026-05-29 — M4 Booking Core stress-tested; sync-capture ADR + api-contract amended

**Action**: Pre-implementation stress-test of the M4 booking + Stripe flow (idea-refine). Confirmed three design decisions and amended the affected specs/ADR before writing the plan.

**Decisions:**
- **Separate charges + transfers** (not destination charges) — platform captures at booking; the `payout-queue` transfers the owner's share later (enqueued at check-in, M5). Gives payout-timing control + trivial pre-charter refunds.
- **Lock the `Yacht` row** + `UNIQUE(yachtId, date)` safety net — M3's "absence = AVAILABLE" model means there are no per-date rows to `SELECT FOR UPDATE`.
- **Authorize client-side (SCA/3DS) → capture after commit** — a server-side `confirm` can't satisfy EU SCA in one round-trip; capture-after-commit also closes the "charged-but-no-booking" window (uncaptured auths expire).

**Updated:**
- `decisions/2026-05-27-sync-payment-capture.md` — amendment section (SCA, capture-after-commit, separate transfers, Yacht-row lock)
- `specs/api-contract.md` — added `POST /bookings/payment-intent`; `POST /bookings` now takes `stripePaymentIntentId`; notes rewritten
- `CLAUDE.md` (backend repo) — payments stack line, Direction B payments item, double-booking lock layer

**Artifact**: design one-pager at `yachties-backend/docs/ideas/m4-booking-core.md`.

---

## 2026-05-29 — Milestone 3 (Discovery) backend implemented

**Action**: Implemented the M3 backend — pricing engine, availability module, and search — via subagent-driven TDD. All green (97 e2e + 15 unit).

**Added (in the backend repo, not the vault):**
- **Pricing engine** — strategy pipeline `BasePriceStrategy` → `WeeklyRateStrategy` (flat weekly rate replaces base on 7-day charters) → `CrewOptionStrategy` (per-day skipper/crew fee), folded by `PricingEngine`. Public `GET /yachts/:id/pricing` (422 below min / invalid range, 409 on unavailable dates).
- **Availability module** — public `GET /yachts/:id/availability` (one-row-per-date; absence = AVAILABLE; `ownerNote` hidden; 12-month cap), owner-only `POST`/`DELETE /yachts/:id/availability/block` with reserved-date guards, and `applyPrepDays` helper (written now, triggered by booking in M4).
- **Search** — public `GET /yachts`: one `$queryRaw` haversine query (geo radius + JSON crew-flag filter + `NOT EXISTS` date-availability + capacity/type/price filters + sort + pagination) with per-result `calculatedPriceCents`.

**Scope decisions (Direction B continuity):**
- **No server-side Mapbox geocoding at MVP** — `GET /yachts` takes `lat`/`lng`/`radiusKm`; place-name → coordinates is resolved client-side. A `MapboxGeocodingPort` can be added later without changing the search SQL.
- `days` = nights = `dateDiff(checkOut, checkIn)`; availability occupies `[checkIn, checkOut)` (checkout day free).
- Search-card `calculatedPriceCents` uses the **bareboat** price for the range — a representative "from" price.

**No schema/spec changes** — M3 used the existing data model (`Yacht`, `YachtAvailability`, `PricingRule`); `api-contract.md` already documents these endpoints.

**Source**: M3 plan at `yachties-backend/docs/superpowers/plans/2026-05-29-milestone-3-discovery.md`.

---

## 2026-05-29 — PricingRule.crewOption added (Milestone 2)

**Action**: Milestone 2 (Listings) backend implemented. During planning, a schema gap surfaced: `crew_option` pricing rules had no way to distinguish SKIPPERED vs CREWED fees. Added a nullable `crewOption` enum column to `PricingRule`.

**Updated:**
- `specs/data-model.md` — `PricingRule.crewOption` field + design note

**Also**: earlier in M2, `r2Key` was renamed to `storageKey` (provider-neutral) across `data-model.md`, `api-contract.md`, and `user-stories/owner.md`.

---

## 2026-05-27 — Backend architecture stress-tested and simplified

**Action**: Stress-tested original architecture against solo-developer constraints. Locked in 6 scope decisions deferring complexity that doesn't earn its cost at MVP scale. Updated all affected specs and created 6 ADRs.

**Added:**
- `decisions/2026-05-27-defer-postgis-to-scale.md` — lat/lng floats + haversine SQL; PostGIS deferred
- `decisions/2026-05-27-sync-payment-capture.md` — Stripe capture synchronous inside booking transaction
- `decisions/2026-05-27-defer-transactional-outbox.md` — outbox table preserved; poller deferred; emails sync via EmailPort
- `decisions/2026-05-27-scoped-bullmq-usage.md` — one BullMQ queue (payout-queue); Redis from M4
- `decisions/2026-05-27-cqrs-scoped-to-booking.md` — CQRS only on booking state machine
- `decisions/2026-05-27-minimal-admin-mvp.md` — 4 admin endpoints for soft launch

**Updated:**
- `specs/backend-architecture.md` — tech stack, booking workflow, payment flow, event architecture, "deferred to scale" subsection
- `specs/data-model.md` — Prisma schema (lat/lng floats, no postgresqlExtensions), indexes, design decisions
- `specs/api-contract.md` — POST /bookings notes, Admin section restructured (Phase 1 + deferred), endpoint count 54 → 42
- `specs/mvp-scope.md` — admin panel scope reduced, deferred items added to out-of-scope table
- `specs/env-variables.md` — Redis note, BullMQ section collapsed to payout-queue only
- `roadmap/phase-1-development.md` — M1 (no PostGIS/Redis), M3 (haversine), M4 (sync booking, payout-queue only), M6 (4 admin endpoints)
- `wiki/index.md`, `wiki/hot.md` — 6 new decisions indexed, current focus updated

**Source**: Architecture stress-test session — Direction B: targeted simplifications for solo developer.

---

## 2026-05-01 — Vault initialized

**Action**: Created vault structure and initial documentation from founding conversation.

**Added:**
- `CLAUDE.md` — vault instructions for Claude (rewritten for YachtBay)
- `wiki/index.md` — master table of contents
- `wiki/hot.md` — session context cache
- `wiki/log.md` — this file
- `wiki/concepts/vision-and-mission.md`
- `wiki/concepts/business-model.md`
- `wiki/concepts/commission-model.md`
- `wiki/concepts/target-market.md`
- `wiki/concepts/booking-model.md`
- `wiki/concepts/crew-options.md`
- `wiki/concepts/user-personas.md`
- `decisions/2026-05-01-commission-from-owners.md`
- `decisions/2026-05-01-greece-first-launch.md`
- `decisions/2026-05-01-instant-booking.md`
- `decisions/2026-05-01-platform-name-yachtbay.md`
- `roadmap/overview.md`

**Source**: Founding conversation with the platform founder.

---

## 2026-05-01 — Stripe Connect research and payment decision

**Action**: Researched Stripe Connect options, documented findings, recorded payment decision.

**Added:**
- `wiki/concepts/stripe-connect.md` — full reference: account types, fees, charge types, payment flow
- `decisions/2026-05-01-stripe-connect-express.md` — decision to use Express for MVP

**Updated:**
- `wiki/index.md` — added new entries
- `wiki/hot.md` — added payment decision to recent decisions

**Source**: Stripe official documentation (stripe.com/connect/pricing, docs.stripe.com/connect/accounts).

---

## 2026-05-05 — Google Stitch design brief created

**Action**: Created complete storefront design brief for Google Stitch — all 18 pages, all renter user stories (R-001–R-022), shared components, design direction, business rules, and route summary.

**Added:** `specs/design/stitch-brief.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

**Purpose**: This file is fed into Google Stitch to generate the storefront UI. After design, Google Stitch MCP connects to Claude Code to transform the UI into React Router 7 SSR code.

---

## 2026-05-05 — Environment variables documented

**Action**: Documented all environment variables for all 4 apps with Railway config notes and pre-launch secrets checklist.

**Added:** `specs/env-variables.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-05 — Development roadmap written

**Action**: Created 7-milestone Phase 1 development roadmap with build order, dependencies, and testable checkpoints per milestone.

**Added:** `roadmap/phase-1-development.md`
**Updated:** `roadmap/overview.md`, `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-05 — API contract written

**Action**: Wrote complete API contract — 54 endpoints across 8 modules with full request/response shapes.

**Added:** `specs/api-contract.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-05 — Data model finalized

**Action**: Audited draft data model against all 68 user stories. Found 18 gaps. Produced final Prisma schema.

**Added:** `specs/data-model.md`

**Key gaps fixed:** roles array, yacht_photos table, slug, prep_days, charter_types, email verification fields, password reset fields, phone, bio, profile_photo_url, commission_rate, user status/suspension, guest_count on reservations, triggered_by/admin_id on events, stripe_refund_id on payments, error_message on outbox.

**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-05 — All site maps complete

**Action**: Created admin panel site map. All three apps now mapped.

**Added:** `specs/design/sitemap-admin.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

**Total site map**: Storefront 18 + Owner Panel 25 + Admin 10 = **53 pages**

---

## 2026-05-05 — Owner panel site map created

**Action**: Created owner panel site map — 25 pages including 6-step listing creation wizard.

**Added:** `specs/design/sitemap-owner-panel.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-04 — Storefront site map created

**Action**: Created storefront site map — 18 pages with routes, auth requirements, SSR strategy, and navigation structure.

**Added:** `specs/design/sitemap-storefront.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-04 — All user stories complete

**Action**: Wrote Admin user stories. All three personas now complete.

**Added:**
- `specs/user-stories/admin.md` — 16 stories across 6 epics

**Story totals**: Renter 26 · Owner 26 · Admin 16 = **68 total stories**

**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-04 — Owner user stories written

**Action**: Wrote all owner user stories (MVP + Phase 2) with acceptance criteria and technical requirements.

**Added:**
- `specs/user-stories/owner.md` — 26 stories across 7 epics

**Updated:**
- `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-04 — Renter user stories written

**Action**: Wrote all renter user stories (MVP + Phase 2) with acceptance criteria and technical requirements.

**Added:**
- `specs/user-stories/renter.md` — 26 stories across 8 epics

**Updated:**
- `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-03 — Frontend architecture decided

**Action**: Decided on separate storefront + owner panel apps in a Turborepo monorepo.

**Added:**
- `decisions/2026-05-03-frontend-monorepo.md`

**Updated:**
- `wiki/concepts/tech-stack.md` — frontend section updated with monorepo structure
- `wiki/index.md` — decision added
- `wiki/hot.md` — current focus updated

---

## 2026-05-03 — Competitor research completed

**Action**: Researched 5 competitors, documented entity pages, wrote synthesis analysis, updated cancellation policy based on market findings.

**Added:**
- `wiki/entities/getmyboat.md`
- `wiki/entities/clickandboat.md`
- `wiki/entities/boataround.md`
- `wiki/entities/anchorsbook.md`
- `wiki/entities/sailogy.md`
- `wiki/concepts/competitor-analysis.md`

**Updated:**
- `decisions/2026-05-02-platform-cancellation-policy.md` — revised thresholds based on market research, added day trip vs. weekly split, added platform fees non-refundable rule
- `wiki/index.md` — entities section populated, competitor-analysis added
- `wiki/hot.md` — status and focus updated

**Key findings:** YachtBay's 13% owner-only commission is the lowest in the market. GetMyBoat (market leader) lacks instant book — a real differentiator. Platform fees should be non-refundable on cancellation (industry standard). No dominant Greece-specific P2P platform exists.

---

## 2026-05-03 — Tech stack finalized

**Action**: Confirmed all remaining stack decisions and documented the complete tech stack.

**Added:**
- `wiki/concepts/tech-stack.md` — master tech stack reference with rationale, Mapbox alternatives, and rejected options

**Updated:**
- `specs/backend-architecture.md` — Cloudinary → Cloudflare R2, adapter references updated
- `wiki/index.md` — added tech-stack concept page
- `wiki/hot.md` — current focus and open questions updated

**Key decisions confirmed:** Prisma (ORM), Custom JWT, Cloudflare R2 (images), Resend (email), Railway (hosting), GitHub Actions (CI/CD), Mapbox (maps), PostGIS (geo search), SMS deferred.

---

## 2026-05-03 — Backend architecture finalized

**Action**: Synthesized architecture from two Claude sessions into a definitive backend spec.

**Added:**
- `specs/backend-architecture.md` — full architecture: stack, patterns, data model, booking workflow, payment flow, event architecture, extensibility points, scalability path
- `decisions/2026-05-03-backend-architecture.md` — decision record
- `wiki/sources/yacht-booking-system-summary.md` — ingested raw docx from architecture session

**Updated:**
- `wiki/index.md` — added new spec, decision, and source
- `wiki/hot.md` — updated current focus and vault status

**Source**: This session + `raw/yacht-booking-system-summary.docx`

---

## 2026-05-02 — MVP scope defined

**Action**: Defined full MVP feature set after answering all scoping questions.

**Added:**
- `specs/mvp-scope.md` — full MVP feature list (owner, renter, platform, deferred items)
- `decisions/2026-05-02-web-app-mvp.md` — web app only at MVP
- `decisions/2026-05-02-platform-cancellation-policy.md` — platform-defined cancellation policy

**Updated:**
- `roadmap/overview.md` — linked to MVP spec
- `wiki/index.md` — added spec and new decisions
- `wiki/hot.md` — updated current focus, status, open questions, next steps

**Source**: Founding conversation — MVP scoping session.

---

## 2026-05-01 — Revised payment decision: Express → Accounts v2

**Action**: After deeper research confirmed Express is deprecated for new integrations, revised the payment decision to Accounts v2.

**Added:**
- `decisions/2026-05-01-stripe-connect-accounts-v2.md` — new accepted decision

**Updated:**
- `decisions/2026-05-01-stripe-connect-express.md` — status changed to `superseded`, warning callout added
- `wiki/concepts/stripe-connect.md` — Accounts v2 section updated with full configuration details, comparison table updated, "Our Choice" updated
- `wiki/index.md` — old decision marked superseded, new decision added
- `wiki/hot.md` — payment decision updated
