---
title: One BullMQ queue at MVP — payout-queue only
date: 2026-05-27
status: accepted
tags:
  - decision
  - technical
  - backend
  - queues
---

# One BullMQ queue at MVP — payout-queue only

## Context

The original architecture specified four BullMQ queues: `email-queue`, `payment-queue`, `host-queue`, `audit-queue`. With synchronous payment capture and synchronous emails (see companion decisions), three of these queues have no jobs to process at MVP.

Redis + BullMQ remain in the stack but their introduction is deferred to Milestone 4 (when the booking flow genuinely needs delayed payout scheduling).

## Decision

**One BullMQ queue at MVP: `payout-queue`.**

The payout queue handles the one job that genuinely requires persistent delayed execution: releasing the Stripe transfer to the owner 24–48h after the charter check-in timestamp. This cannot be done synchronously (the delay is 24+ hours). BullMQ's persistent delayed jobs are exactly the right tool.

Redis is provisioned in Milestone 4 (not Milestone 1) since it has no other consumers before that point.

## Consequences

- Redis not provisioned until Milestone 4 — reduces infrastructure surface for M1-M3
- `QUEUE_PAYOUT="payout-queue"` is the only active BullMQ queue name env var
- Other queue name env vars (`QUEUE_EMAIL`, `QUEUE_PAYMENT`, etc.) documented but commented out
- `payout-queue` worker handles retries with exponential backoff (3 attempts)

## Migration path

Each additional queue is ~1 day of work and should be added when evidence warrants:

- `email-queue` — add when email delivery reliability becomes a measurable problem
- `host-queue` — add when owner notification reliability becomes a measurable problem
- `audit-queue` — add when `reservation_events` volume requires async writes
- `payment-queue` — add if sync Stripe capture causes lock contention at scale

The BullMQ infrastructure is already in place after M4. Adding a queue means: create the worker, update the adapter injection, add the queue name env var.

## Related

- [[specs/backend-architecture]]
- [[specs/env-variables]]
- [[decisions/2026-05-27-defer-transactional-outbox]]
- [[decisions/2026-05-27-sync-payment-capture]]
- [[roadmap/phase-1-development]]
