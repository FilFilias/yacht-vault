---
title: Backend Architecture
tags:
  - spec
  - technical
  - backend
status: approved
last-updated: 2026-05-03
---

# Backend Architecture

**Summary**: Definitive backend architecture for YachtBay — technology stack, architecture pattern, module structure, data model, booking workflow, payment model, event architecture, and extensibility strategy.

> [!info] Sources
> Synthesized from two sessions: the founding product/payments session and a dedicated architecture session. See `raw/yacht-booking-system-summary.docx` for the architecture session output.

---

## 1. Technology Stack

| Layer             | Choice                                            | Rationale                                                                                                            |
| ----------------- | ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Runtime**       | Node.js                                           | Team expertise, strong async I/O for marketplace workloads                                                           |
| **Language**      | TypeScript (strict)                               | Type safety across the entire backend — non-negotiable                                                               |
| **Framework**     | NestJS                                            | Enforced modular structure, built-in DI, CQRS module, Guards, Pipes — matches our architecture pattern exactly       |
| **HTTP adapter**  | Fastify (via NestJS adapter)                      | 2–3× better throughput than Express with near-zero code changes in NestJS                                            |
| **Database**      | PostgreSQL                                        | ACID transactions, row-level locking, date-range queries, PostGIS for geo search                                     |
| **ORM**           | Prisma                                            | Type-safe queries, auto-generated migrations, good NestJS integration, `$queryRaw` for locking operations            |
| **Job queues**    | BullMQ + Redis                                    | Persistent async jobs, retry on failure, delayed jobs (payout scheduling)                                            |
| **Auth**          | Custom JWT via `@nestjs/jwt` + `@nestjs/passport` | Full control, role-based (owner/renter/admin), no vendor dependency                                                  |
| **Payments**      | Stripe Connect Accounts v2                        | Marketplace payment splitting, platform-controlled payouts, non-deprecated API                                       |
| **Email**         | Resend + React Email                              | TypeScript-native, React-based templates, excellent DX                                                               |
| **Image storage** | Cloudflare R2                                     | S3-compatible, zero egress fees, pairs with Cloudflare CDN. No image transformation — handle resizing at upload time |
| **Maps**          | Mapbox                                            | Frontend map display, location picker. See [[wiki/concepts/tech-stack]] for alternatives                             |
| **Geo search**    | PostGIS (PostgreSQL extension)                    | DB-level geographic queries — sufficient at MVP scale, no extra service                                              |
| **Scheduler**     | `@nestjs/schedule`                                | Outbox poller, payout release scheduler, availability cleanup                                                        |
| **Hosting**       | Railway                                           | Managed PostgreSQL + Redis, simple Node.js deployments, scales to containers                                         |

---

## 2. Architecture Pattern

### Modular Monolith — single deployable, bounded internal modules

One deployable unit. Code is organized into domain modules with strict boundaries enforced by NestJS's module system. Cross-module access must go through explicit module exports — no direct service imports across domain boundaries.

This is the right choice because:
- Single deployment — no distributed systems complexity at MVP
- Stateless API — horizontal scaling is trivial (add instances behind a load balancer)
- Clear migration path to microservices: bounded modules + event bus → extract module → replace internal bus with Kafka/RabbitMQ
- A well-built modular monolith handles tens of thousands of concurrent users without issues

### Four patterns working together

```
┌─────────────────────────────────────────────────────┐
│                  HTTP Layer (NestJS + Fastify)        │
│         Guards (auth) · Pipes (validation)           │
├──────────────────────────┬──────────────────────────┤
│     Commands (writes)    │      Queries (reads)      │
│   @nestjs/cqrs CQRS      │   Direct DB via repos     │
├──────────────────────────┴──────────────────────────┤
│              Domain Modules                          │
│  Listings · Bookings · Pricing · Payments · Users   │
├─────────────────────────────────────────────────────┤
│           Domain Events + Transactional Outbox       │
│         EventEmitter → BullMQ → Job Handlers        │
├─────────────────────────────────────────────────────┤
│           Ports & Adapters (infrastructure)          │
│      StripeAdapter · ResendAdapter · R2Adapter       │
├─────────────────────────────────────────────────────┤
│              PostgreSQL + Redis                      │
└─────────────────────────────────────────────────────┘
```

**CQRS** — Commands mutate state (CreateBooking, CancelBooking, ConfirmBooking). Queries read data (GetAvailableYachts, GetBookingDetails). Commands go through handlers with full validation, business rules, and locking. Queries go directly to the DB — no overhead.

**Domain Events + Transactional Outbox** — side effects (email, payment, notifications) are decoupled from core booking logic via events. Critical events are written to the outbox table inside the same Postgres transaction as the domain write — guaranteeing exactly-once delivery even on process crash.

**Ports & Adapters** — all external integrations (Stripe, Resend, Cloudflare R2, Mapbox) are accessed through interfaces. Swapping a provider = new adapter implementation, zero business logic changes.

**Strategy Pattern** — pricing rules, commission rates, cancellation policies, and future promotions are implemented as pluggable strategies. Adding a new rule = new strategy class, no existing code modified.

---

## 3. Module Structure

```
src/
├── modules/
│   ├── yachts/              # Fleet listings, specs, photos, owner management
│   ├── availability/        # Per-date calendar, block management, PREP days
│   ├── bookings/            # Reservation lifecycle, state machine, CQRS commands
│   ├── pricing/             # Rate calculation, strategy engine, rules management
│   ├── payments/            # Stripe Connect integration, webhooks, refunds, payouts
│   ├── users/               # Owner + renter profiles, auth, roles
│   ├── notifications/       # Email dispatch via BullMQ queues
│   └── search/              # Availability search, geo queries, filters
│
├── infrastructure/
│   ├── database/            # Prisma client, migrations, seed
│   ├── redis/               # BullMQ setup, Redis client
│   ├── adapters/
│   │   ├── stripe.adapter.ts
│   │   ├── resend.adapter.ts
│   │   └── r2.adapter.ts
│   └── outbox/              # Outbox poller, event relay to BullMQ
│
├── common/
│   ├── guards/              # JwtAuthGuard, RolesGuard
│   ├── pipes/               # ValidationPipe (class-validator)
│   ├── decorators/          # @CurrentUser, @Roles
│   ├── interfaces/          # Port interfaces (PaymentPort, EmailPort, StoragePort)
│   └── events/              # Domain event definitions
│
└── main.ts                  # Fastify adapter bootstrap
```

### Module responsibilities

| Module | Responsibility | Key tables |
|---|---|---|
| **Yachts** | Fleet listings, specs, photos, owner management | `yachts`, `yacht_features`, `yacht_photos` |
| **Availability** | Per-date calendar, PREP blocking, availability queries | `yacht_availability` |
| **Bookings** | Reservation lifecycle, state machine, CQRS | `reservations`, `reservation_events` |
| **Pricing** | Rate calculation, seasonal rules, strategy engine | `pricing_rules` |
| **Payments** | Stripe Connect Accounts v2, webhooks, refunds, payouts | `payments` |
| **Users** | Owner + renter profiles, auth, roles, Stripe account linking | `users` |
| **Notifications** | Email dispatch, queue management | — (external via Resend) |
| **Search** | Geo + date availability search, filters, ranking | — (queries across modules) |

---

## 4. Data Model

### Core design principles
- Monetary values stored as **integer cents** — no floating point
- One row per yacht per date in `yacht_availability` — O(1) availability lookup
- `version` column on reservations for optimistic concurrency
- `reservation_events` as immutable audit log
- `outbox_events` for reliable cross-boundary event delivery

### Key entities

```sql
-- Users (owners and renters share one table, roles are flags)
users
  id, email, password_hash, first_name, last_name
  role: owner | renter | both | admin
  stripe_account_id          -- Stripe Connect Accounts v2 account ID (owners only)
  stripe_account_status      -- pending | active | restricted
  created_at, updated_at

-- Yacht listings
yachts
  id, owner_id (→ users), name, type, length_meters
  capacity_guests, build_year, manufacturer, model
  home_port, location_lat, location_lng  -- PostGIS point
  description, base_price_cents
  crew_options: bareboat | skippered | crewed | all  -- jsonb flags
  min_charter_days           -- e.g. 7 for summer weekly minimum
  status: draft | active | paused
  created_at, updated_at

-- Availability calendar (one row per yacht per date)
yacht_availability
  id, yacht_id (→ yachts), date
  status: available | reserved | blocked | prep
  price_override_cents       -- null = use pricing_rules
  reservation_id (→ reservations, nullable)
  UNIQUE (yacht_id, date)    -- hard DB constraint, final safety net

-- Reservations
reservations
  id, yacht_id (→ yachts), renter_id (→ users)
  check_in, check_out
  crew_option: bareboat | skippered | crewed
  status: pending | confirmed | checked_in | completed | cancelled
  total_price_cents
  platform_fee_cents         -- YachtBay commission (13%)
  owner_payout_cents         -- total_price_cents - platform_fee_cents
  cancellation_policy_snapshot  -- jsonb, snapshot at time of booking
  version                    -- optimistic concurrency
  idempotency_key            -- prevent duplicate submissions
  created_at, updated_at

-- Immutable reservation event log
reservation_events
  id, reservation_id (→ reservations)
  event_type: created | confirmed | checked_in | completed | cancelled | payment_captured
  payload (jsonb)            -- full state snapshot at time of event
  created_at                 -- never updated, never deleted

-- Pricing rules (Strategy Pattern — one row per rule)
pricing_rules
  id, yacht_id (→ yachts, nullable — null = platform-wide rule)
  rule_type: seasonal | weekend | long_charter | crew_option | promotional
  date_from, date_to         -- nullable for non-date rules
  modifier_type: percentage | fixed_override
  modifier_value             -- e.g. 20 for +20%, or 50000 for €500 flat rate
  priority                   -- higher priority wins on conflict
  active (boolean)
  created_at, updated_at

-- Payments (multiple per reservation: deposit, balance, refunds)
payments
  id, reservation_id (→ reservations)
  payment_type: deposit | balance | refund
  amount_cents
  platform_fee_cents         -- application_fee_amount captured by YachtBay
  owner_amount_cents         -- amount transferred to owner's Stripe account
  status: pending | captured | failed | refunded
  stripe_payment_intent_id
  stripe_transfer_id         -- Stripe Connect transfer to owner
  created_at, updated_at

-- Transactional outbox (reliable event delivery)
outbox_events
  id, aggregate_id, aggregate_type
  event_type
  payload (jsonb)
  status: pending | delivered | failed
  attempts, last_attempt_at
  created_at
```

### Critical indexes

```sql
-- Most queried index in the system
CREATE UNIQUE INDEX ON yacht_availability (yacht_id, date);

-- Availability search by status
CREATE INDEX ON yacht_availability (yacht_id, status, date);

-- Reservation lookups
CREATE INDEX ON reservations (yacht_id, status);
CREATE INDEX ON reservations (renter_id);

-- Outbox polling
CREATE INDEX ON outbox_events (status, created_at) WHERE status = 'pending';

-- Geo search (PostGIS)
CREATE INDEX ON yachts USING GIST (location);
```

---

## 5. Booking Workflow

### Reservation state machine

```
                    ┌─────────┐
                    │ PENDING │ ← booking submitted, availability locked
                    └────┬────┘
                         │ auto-confirm (instant book)
                    ┌────▼────┐
                    │CONFIRMED│ ← deposit captured, emails sent
                    └────┬────┘
                         │ owner marks arrival
                   ┌─────▼──────┐
                   │ CHECKED_IN │ ← charter in progress
                   └─────┬──────┘
                         │ owner marks completion
                   ┌─────▼─────┐
                   │ COMPLETED │ ← balance captured, review request sent
                   └───────────┘

   PENDING / CONFIRMED → CANCELLED (guest or platform, refund per policy)
```

Illegal transitions are rejected at the command handler before any DB write.

### Synchronous flow — inside the HTTP request (target: < 200ms)

```
POST /api/v1/bookings
  1. JWT validation + role check (Guard)
  2. Request validation (ValidationPipe + class-validator)
  3. Idempotency key check — return cached response if duplicate
  4. Dispatch CreateBookingCommand
     └─ CreateBookingHandler:
        a. Fetch active pricing_rules for yacht + dates
        b. Calculate total via PricingEngine (Strategy pipeline)
        c. Calculate platform_fee_cents (13% via commission strategy)
        d. BEGIN Postgres transaction
           - SELECT FOR UPDATE on yacht_availability rows (pessimistic lock)
           - Assert all rows have status = 'available'
           - INSERT reservation (status=PENDING, version=1)
           - UPDATE yacht_availability rows (status=reserved)
           - INSERT outbox_event (reservation.created)  ← same transaction
        e. COMMIT
  5. Return 201 with reservation details
```

### Asynchronous flow — after transaction commits

Four BullMQ jobs enqueued in parallel, each with independent retry policy:

```
reservation.created outbox event → BullMQ:
  ├── payment-queue:    Capture deposit via Stripe Connect
  ├── email-queue:      Confirmation email to renter (Resend)
  ├── host-queue:       Booking notification to owner (Resend)
  └── audit-queue:      Append to reservation_events
```

---

## 6. Double-Booking Prevention

Three complementary layers — all three must be present:

**Layer 1 — Pessimistic locking (primary guard)**
`SELECT FOR UPDATE` on `yacht_availability` rows inside the Postgres transaction. Concurrent requests for the same yacht + overlapping dates are serialised at DB level. The second request waits, then sees rows already reserved and returns 409.

**Layer 2 — Optimistic concurrency (state transition guard)**
Every reservation carries a `version` integer. Any state update must include `WHERE id = $1 AND version = $2`. Zero rows updated = concurrent modification detected, operation rejected. Protects state transitions (confirm, cancel, check-in) without long-held locks.

**Layer 3 — DB constraint (final safety net)**
`UNIQUE (yacht_id, date)` on `yacht_availability`. Even if application logic fails, the database rejects duplicate rows. This is the last line of defence and should never be reached in normal operation.

---

## 7. Pricing Engine

The pricing engine is a **strategy pipeline** — an ordered list of pricing strategies applied sequentially to a base price. Adding a new pricing rule = new strategy class implementing `PricingStrategy` interface. No existing code is modified.

```typescript
interface PricingStrategy {
  applies(context: PricingContext): boolean;
  apply(price: number, context: PricingContext): number;
  priority: number;
}

// Active strategies (MVP)
class BasePriceStrategy       // Base day rate × number of days
class WeeklyRateStrategy      // Flat weekly rate if duration = 7 days
class CrewOptionStrategy      // Add skipper / crew fee per day

// Future strategies (plug in without touching existing code)
class SeasonalPricingStrategy    // Summer peak surcharge
class LongCharterDiscountStrategy // Discount for 14+ day charters
class PromotionalStrategy         // Discount codes, campaigns
class OwnerCustomPricingStrategy  // Per-owner custom rules
class EarlyBirdStrategy           // Discount for far-future bookings
```

The `PricingEngine` sorts active strategies by priority, runs each that `applies()` to the context, and returns the final price. The `pricing_rules` table drives which strategies are active and with what parameters.

Platform commission is calculated separately as a fixed percentage — currently 13% — via a `CommissionStrategy` that reads per-owner rates from the `users` table.

---

## 8. Payment Architecture (Stripe Connect Accounts v2)

### How it works

```
Renter pays €1,000
  → Stripe processes payment
  → application_fee_amount: €130 (13%) → YachtBay's Stripe account
  → transfer_data.destination: owner's Stripe account → ~€855 (net of Stripe fees)
  → Payout to owner's bank: released 24–48h after charter start
```

### Owner onboarding

1. Owner creates YachtBay account
2. During first listing setup: Stripe Connect Accounts v2 onboarding triggered
3. Account created with configuration:
```json
{
  "defaults": {
    "responsibilities": {
      "fees_collector": "application",
      "losses_collector": "application"
    }
  },
  "dashboard": "express"
}
```
4. `stripe_account_id` stored on the `users` record
5. Owner completes KYC on Stripe-hosted Express onboarding flow
6. `stripe_account_status` updated to `active` via webhook

### Payment flow

```
CreateBookingCommand confirms → enqueue payment-queue job
  → PaymentHandler:
     - Create Stripe PaymentIntent with:
         amount: total_price_cents
         application_fee_amount: platform_fee_cents
         transfer_data.destination: owner.stripe_account_id
     - Capture deposit immediately (instant book)
     - Stripe webhook confirms capture → update payments table
     - Payout to owner scheduled: release at charter_start + 24h
```

### Cancellation refunds

Refund amounts calculated by `CancellationPolicyStrategy` based on days until charter:

| Days before charter | Refund |
|---|---|
| 30+ days | 100% |
| 14–29 days | 50% |
| < 14 days | 0% |

The `cancellation_policy_snapshot` on the reservation captures the policy at booking time — protects renters if the platform policy changes later.

> The `cancellation_policy` is stored as a strategy reference, not hardcoded — future per-owner policies require no schema changes.

---

## 9. Event Architecture

### Event types and delivery

| Event | Persisted | Mechanism | Consumers |
|---|---|---|---|
| `reservation.created` | Yes — outbox | Transactional outbox → BullMQ | Payment, email (renter), host notification, audit |
| `reservation.confirmed` | Yes — outbox | Transactional outbox → BullMQ | Payment capture, email |
| `reservation.cancelled` | Yes — outbox | Transactional outbox → BullMQ | Refund, email, availability release |
| `reservation.checked_in` | BullMQ only | EventEmitter → BullMQ | Host notification |
| `reservation.completed` | BullMQ only | EventEmitter → BullMQ | Balance capture, review request (Phase 2) |
| `payment.captured` | reservation_events | Stripe webhook → handler | Reservation status update |

### Transactional outbox

For payment-triggering events (`reservation.created`, `reservation.confirmed`, `reservation.cancelled`), the event is written to `outbox_events` inside the same Postgres transaction as the domain write. A `@nestjs/schedule` poller runs every 1–2 seconds, picks up pending outbox events, and delivers them to BullMQ. This guarantees exactly-once delivery — the event cannot be lost between the domain write and the queue enqueue.

### Why this matters

Without the outbox: booking created → process crashes before enqueuing → payment job never runs → renter's card not charged, owner not notified.

With the outbox: even if the process crashes, the outbox row is committed. On restart, the poller picks it up and delivers it.

---

## 10. API Design

All endpoints prefixed `/api/v1/`. Breaking changes get a new version prefix (`/api/v2/`) added alongside — existing clients are never broken.

### Core endpoints

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `GET` | `/yachts` | Search with filters (dates, location, capacity, crew) | Public |
| `GET` | `/yachts/:id` | Yacht detail page | Public |
| `GET` | `/yachts/:id/availability` | Availability calendar for date range | Public |
| `GET` | `/yachts/:id/pricing` | Price calculation for date range + crew option | Public |
| `POST` | `/yachts` | Create listing | Owner |
| `PATCH` | `/yachts/:id` | Update listing | Owner |
| `POST` | `/bookings` | Create reservation (instant book) | Renter |
| `GET` | `/bookings/:id` | Get reservation details | Owner / Renter |
| `PATCH` | `/bookings/:id/cancel` | Cancel with refund calculation | Owner / Renter |
| `PATCH` | `/bookings/:id/check-in` | Mark guest arrived | Owner |
| `PATCH` | `/bookings/:id/complete` | Mark charter finished | Owner |
| `POST` | `/payments/webhook` | Stripe webhook handler | Stripe (signature verified) |
| `POST` | `/auth/register` | Register account | Public |
| `POST` | `/auth/login` | Login, return JWT + refresh token | Public |
| `POST` | `/auth/refresh` | Refresh access token | Authenticated |

### Guards

- `JwtAuthGuard` — validates access token on protected routes
- `RolesGuard` — `@Roles('owner')` / `@Roles('renter')` decorators
- `OwnershipGuard` — ensures user can only modify their own resources

### Rate limiting

| Endpoint class | Limit |
|---|---|
| Booking endpoints | 10 req/min per IP |
| Payment endpoints | 5 req/min per user |
| Search endpoints | 60 req/min per IP |
| Auth endpoints | 5 req/min per IP |

---

## 11. Extensibility Points

Every planned future feature has a clear, non-destructive extension point.

| Future feature | How it plugs in |
|---|---|
| **Promotions / discount codes** | New `PromotionalStrategy` in PricingEngine + `promotions` table. Zero changes to existing pricing strategies. |
| **Campaigns** | New `CampaignModule`. Listens to `reservation.created` event — no booking module changes. |
| **Custom pricing per owner** | New `OwnerCustomPricingStrategy`. Reads from `pricing_rules` — already in schema. |
| **Promoted listings** | New `RankingStrategy` interface in Search module. Affects sort order only. |
| **Reviews & ratings** | New `ReviewModule`. Triggered by `reservation.completed` event — no booking changes. |
| **In-app messaging** | New `MessagingModule` with WebSocket (`@nestjs/websockets` + Socket.io). Completely isolated. |
| **Per-owner cancellation policies** | `CancellationPolicyStrategy` already reads from `cancellation_policy_snapshot` — add new policy types to the strategy. |
| **Per-owner commission rates** | `CommissionStrategy` already reads from `users.commission_rate` — just populate the field per owner. |
| **Multi-currency** | New `CurrencyPort` interface + conversion adapter. All amounts stored in EUR cents — conversion at API response layer. |
| **Mobile app** | API-first — no changes needed. Add push notifications via new `PushNotificationAdapter`. |
| **Microservice extraction** | Replace internal `EventEmitter` with Kafka/RabbitMQ adapter. Module boundaries are already clean. |

---

## 12. Scalability Path

### MVP (current)

Single Railway deployment. One NestJS instance + Fastify. Managed PostgreSQL + Redis. Handles hundreds of concurrent users trivially.

### Growth phase (traction achieved)

- Horizontal scaling: add instances behind Railway's load balancer. Stateless JWT auth means any instance handles any request.
- Redis caching: cache availability queries and pricing calculations. Hot data (popular yachts, peak season) hits Redis, not PostgreSQL.
- PostgreSQL read replica: route `GET` search queries to replica, writes stay on primary. Availability search is the heaviest read load.
- Connection pooling: add PgBouncer or use Prisma's built-in pool tuning.

### Scale phase (significant traffic)

- `yacht_availability` table partitioned by year — it will grow large (one row per yacht per date).
- Dedicated search service: replace PostGIS queries with Meilisearch or Algolia for sub-50ms search.
- Extract high-load modules (Search, Availability) to separate services if warranted — module boundaries are already clean for this.

---

## 13. What Was Rejected and Why

| Option | Rejected because |
|---|---|
| Microservices from day one | Distributed transaction complexity for bookings + payments. Operational overhead kills solo developer velocity. |
| Express adapter (default NestJS) | Fastify gives 2–3× throughput at zero code cost. |
| Medusa | E-commerce framework — fights the marketplace domain model. |
| Simple layered architecture | Business logic (state machine, pricing engine, concurrency) becomes tangled in services. Not extensible. |
| Drizzle ORM | Less mature NestJS integration vs Prisma. Prisma's `$queryRaw` handles the few cases (SELECT FOR UPDATE) where raw SQL is needed. |
| Clerk / Auth0 | Vendor dependency, cost at scale. Custom JWT with `@nestjs/passport` is well-documented and gives full control. |
| SendGrid | Resend has better TypeScript/React Email DX and is the modern choice for this stack. |
| Cloudinary | R2 chosen for cost efficiency (zero egress fees). Cloudinary remains the migration path if image transformation becomes needed. |
| Algolia / Meilisearch | PostGIS is sufficient at MVP scale. Migrate if search performance becomes a bottleneck. |
| Google Maps | More expensive than Mapbox. Mapbox free tier covers MVP comfortably. |

## Related

- [[decisions/2026-05-03-backend-architecture]]
- [[wiki/concepts/stripe-connect]]
- [[wiki/concepts/commission-model]]
- [[specs/mvp-scope]]
- [[roadmap/overview]]
