---
title: CQRS scoped to the booking state machine only
date: 2026-05-27
status: accepted
tags:
  - decision
  - technical
  - backend
  - architecture
---

# CQRS scoped to the booking state machine only

## Context

The original architecture applied CQRS (`@nestjs/cqrs`) across all modules. CQRS earns its overhead where commands have: pessimistic locking + pricing engine invocation + optimistic concurrency version bumps + domain event emission. For simple CRUD (users, yachts, availability, pricing config, admin), it's pure boilerplate with no benefit.

## Decision

**Apply `@nestjs/cqrs` to the booking state machine only:**

- `CreateBookingCommand` — full complexity: lock, price, Stripe, confirm
- `CancelBookingCommand` — lock, refund calculation, Stripe refund, availability release
- `CheckInCommand` — state transition, payout scheduling
- `CompleteBookingCommand` — state transition, payout trigger

**All other modules use plain `@Injectable()` NestJS services:**
- Users, Yachts, Availability, Pricing, Payments, Stripe Connect, Search, Admin

## Consequences

- Less boilerplate across 7 of 8 modules
- Booking module retains full CQRS structure where it's justified
- New developers can understand Users/Yachts/etc. without learning CQRS first
- Query side uses direct Prisma calls (no query bus overhead)

## Migration path

If a module grows in command complexity — for example, if the Availability module develops a complex multi-date blocking command with conflict resolution — extract it to CQRS at that point. The pattern is already established in the bookings module to copy from.

## Related

- [[specs/backend-architecture]]
- [[decisions/2026-05-03-backend-architecture]]
