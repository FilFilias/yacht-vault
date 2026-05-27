---
title: "Source: yacht-booking-system-summary.docx"
tags:
  - source
  - technical
  - backend
last-updated: 2026-05-03
---

# Source: Yacht Booking System Architecture Summary

**Summary**: Architecture proposal from a dedicated Claude session on backend design. Provided the foundation for the final backend architecture spec.

**Source file**: `raw/yacht-booking-system-summary.docx`

---

## What This Document Proposed

- NestJS + Fastify adapter as the HTTP framework
- PostgreSQL + Prisma as database + ORM
- BullMQ + Redis for job queues and async processing
- JWT auth (short-lived access + refresh tokens)
- Stripe for payments (generic — no Connect specifics)
- SendGrid / Twilio for email + SMS
- Modular monolith architecture
- State machine for reservation lifecycle
- Repository pattern per module
- Strategy pattern for pricing engine
- Transactional outbox for reliable event delivery
- Pessimistic locking (`SELECT FOR UPDATE`) for double-booking prevention
- Optimistic concurrency (`version` column) for state transitions
- `UNIQUE (yacht_id, date)` as DB-level safety net
- One row per yacht per date in `yacht_availability` for O(1) lookup
- PREP status for cleaning days around bookings
- API versioning at `/api/v1/`
- Rate limiting per endpoint class
- Idempotency keys on booking + payment endpoints

## What Was Adopted

The core architecture is almost entirely adopted: modular monolith, state machine, three-layer double-booking prevention, transactional outbox, BullMQ, Prisma, NestJS + Fastify adapter, strategy pattern for pricing.

## What Was Changed or Added

- **Stripe** → specified as **Stripe Connect Accounts v2** with destination charges and `application_fee_amount` for marketplace commission splitting
- **SendGrid** → replaced with **Resend + React Email** (better TypeScript/React DX)
- **SMS (Twilio)** → deferred, not in MVP
- **CQRS** → explicitly added via `@nestjs/cqrs` (was implied but not named)
- **Ports & Adapters** → explicitly named and structured as a pattern
- **Image storage** → **Cloudinary** added (absent from original)
- **Maps/geo search** → **PostGIS + Mapbox** added (absent from original)
- **Extensibility section** → added with explicit extension points per future feature
- **Scalability path** → added (MVP → growth → scale phases)
- **Data model** → extended with `platform_fee_cents`, `owner_payout_cents`, `stripe_transfer_id`, `cancellation_policy_snapshot`
- **Terminology** → Host/Guest → Owner/Renter (to match YachtBay domain language)

## Related Pages

- [[specs/backend-architecture]]
- [[decisions/2026-05-03-backend-architecture]]
