---
title: Defer transactional outbox — synchronous emails via EmailPort at MVP
date: 2026-05-27
status: accepted
tags:
  - decision
  - technical
  - backend
  - events
---

# Defer transactional outbox — synchronous emails via EmailPort at MVP

## Context

The original architecture used a transactional outbox: domain events written to `outbox_events` inside the same DB transaction as the domain write, then polled by a `@nestjs/schedule` scheduler every 1–2 seconds and delivered to BullMQ queues (email-queue, host-queue, audit-queue, payment-queue).

This guarantees exactly-once event delivery even if the process crashes between the DB write and the queue enqueue. For a high-volume platform this is the right call. For an MVP with hundreds of bookings per month, it adds operational complexity without measurable benefit.

## Options Considered

- **Option A — Full outbox**: `outbox_events` written in transaction, poller every 2s, 4 BullMQ queues. Guaranteed delivery. Requires outbox poller, BullMQ worker setup, queue monitoring.
- **Option B — Synchronous post-commit via EmailPort**: emails sent directly via `emailPort.send()` after the transaction commits, wrapped in try-catch. Failures are logged but do not fail the booking. Simple, no infrastructure.

## Decision

**Option B — Synchronous emails via EmailPort.**

Email delivery failures at MVP are acceptable (the booking is already confirmed in the DB). The Ports & Adapters design means the migration to queued delivery is a one-file swap — no command handler changes.

The `outbox_events` table **stays in the schema** — it is not deleted. It preserves the future option at zero cost.

## Consequences

- No outbox poller needed at MVP
- `email-queue`, `host-queue`, `audit-queue` not created at MVP
- Email delivery failures are logged; the booking is unaffected
- `outbox_events` table in schema but with 0 rows at MVP
- `reservation_events` are still written synchronously inside the DB transaction (audit trail unaffected)

## Migration path (one-file swap)

1. Create `QueuedEmailAdapter` implementing `EmailPort` — enqueues to BullMQ instead of calling Resend directly
2. Swap the `ResendAdapter` injection for `QueuedEmailAdapter` in the module definition
3. Add the outbox poller (`@nestjs/schedule`) to `infrastructure/outbox/`
4. Wire the BullMQ email/host/audit queues

Zero changes to command handlers. The `outbox_events` table is already in place.

## Related

- [[specs/backend-architecture]]
- [[decisions/2026-05-27-scoped-bullmq-usage]]
- [[decisions/2026-05-27-sync-payment-capture]]
