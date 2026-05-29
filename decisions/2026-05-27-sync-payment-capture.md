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

> [!warning] Amended 2026-05-29 — see the **Amendment** section below.
> The headline still holds (instant `CONFIRMED`, no PENDING state, no cleanup job, the whole booking completes in one `POST /bookings` request, `402` on payment failure). Two mechanics changed: (1) the card is **authorized client-side** and **captured after commit** — a server-side confirm can't satisfy EU SCA/3DS in one round-trip; (2) money flows via **separate charges + transfers**, not destination charges, so the `payout-queue` controls payout timing.

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

## Amendment — 2026-05-29: SCA + separate transfers

Stress-testing M4 before implementation surfaced two gaps in the original Option B:

**1. SCA / 3D Secure (Greece/EU).** A server-side `paymentIntents.create({ confirm: true })` with a raw PaymentMethod frequently returns `requires_action` for European cards (Strong Customer Authentication), which cannot complete in a single server round-trip. **Resolution:** the renter authorizes a **manual-capture** PaymentIntent **client-side** via Stripe.js (3DS handled in-browser, reaching `requires_capture`), then `POST /bookings` receives the authorized `stripePaymentIntentId`. A new precursor endpoint `POST /bookings/payment-intent` issues the `clientSecret`.

**2. Capture-after-commit closes the "charged-but-no-booking" window.** Capture runs *after* the transaction commits, so a commit failure leaves only an **uncaptured authorization, which expires automatically** — Stripe never charges the customer without a reservation. If capture itself fails post-commit, the reservation is compensated (cancelled + dates released) and the auth voided. On a date race / validation failure the PaymentIntent is voided so the hold is released. A declined authorization → `402`.

**3. Separate charges + transfers (not destination charges).** Funds are captured to the **platform** balance at booking. The `payout-queue` worker later creates a `Transfer` of the owner's share to their connected account (enqueued at check-in, released ~24–48h after charter start — M5). This gives explicit payout-timing control and makes pre-charter refunds trivial (no transfer to reverse). CLAUDE.md's payments line and the api-contract `POST /bookings` were updated to match.

**4. Lock target.** The pessimistic lock is `SELECT FOR UPDATE` on the parent **`Yacht` row**, not per-date availability rows — the availability model stores no row for free dates (absence = AVAILABLE), so there is nothing per-date to lock. `UNIQUE(yachtId, date)` remains the final race safety net.

**What did NOT change:** instant book → `CONFIRMED`, no PENDING state, no stuck-reservation cleanup job, the booking completes within the single `POST /bookings` request, and `402` on payment failure. The async migration path above still applies if volume ever demands it.

## Related

- [[specs/backend-architecture]]
- [[specs/api-contract]]
- [[decisions/2026-05-27-defer-transactional-outbox]]
- [[decisions/2026-05-27-scoped-bullmq-usage]]
