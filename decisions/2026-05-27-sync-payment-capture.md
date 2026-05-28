---
title: Synchronous payment capture inside the booking transaction
date: 2026-05-27
status: accepted
tags:
  - decision
  - technical
  - backend
  - payments
---

# Synchronous payment capture inside the booking transaction

## Context

The original architecture had payment capture as an async step: `POST /bookings` would create a `PENDING` reservation, write an outbox event, commit, and then a `payment-queue` BullMQ job would call Stripe asynchronously. Booking only moved to `CONFIRMED` after the Stripe webhook confirmed capture.

This design has real advantages (short DB transactions, resilience to transient Stripe failures) but comes with significant complexity for a solo developer: a cleanup job is needed to cancel stuck-PENDING reservations, the webhook handler must be idempotent, and the two-phase booking state makes the frontend more complex (polling or WebSocket for status).

## Options Considered

- **Option A — Async via BullMQ**: PENDING → outbox → payment-queue → Stripe → webhook → CONFIRMED. Resilient, short transactions. Requires cleanup job, webhook idempotency, frontend polling.
- **Option B — Synchronous inside transaction**: `SELECT FOR UPDATE` → `INSERT reservation (CONFIRMED)` → `Stripe.paymentIntents.create({ confirm: true })` → on success COMMIT, on failure ROLLBACK. Single HTTP request, simple response contract, no PENDING state at MVP.

## Decision

**Option B — Synchronous inside the DB transaction.**

The trade-off is holding the DB transaction open for ~300ms during the Stripe API call. At MVP volume (Greece, hundreds of bookings/month), lock contention on individual yacht-date rows is not a concern. A failed Stripe call rolls back the reservation atomically — the frontend gets a clean 402, no orphaned data.

The `PENDING` enum value stays in the schema for future request-to-book flows but is unreachable in the MVP booking path.

## Consequences

- No stuck-PENDING cleanup job needed at MVP
- No frontend polling for payment confirmation — `201 CONFIRMED` is the only success response
- DB transaction held ~300ms during Stripe call — acceptable at MVP volume
- Stripe webhooks remain wired as a secondary/idempotent path (`payment_intent.payment_failed`, `account.updated`, `charge.refunded`)
- `PENDING` status preserved in schema for future use

## Migration path

If sync capture causes measurable lock contention (only at significant booking volume):

1. Reintroduce `PENDING` status in the booking flow
2. Move Stripe call to a `payment-queue` BullMQ job
3. Add outbox event in the transaction
4. Add a cleanup job for stuck-PENDING reservations (e.g. 15-minute TTL)
5. Add frontend polling or WebSocket for booking status

This is a contained change within the bookings module.

## Related

- [[specs/backend-architecture]]
- [[specs/api-contract]]
- [[decisions/2026-05-27-defer-transactional-outbox]]
- [[decisions/2026-05-27-scoped-bullmq-usage]]
