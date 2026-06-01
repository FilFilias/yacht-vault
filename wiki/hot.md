---
title: Hot — Session Context Cache
tags:
  - meta
last-updated: 2026-06-01
---

# Hot — Session Context Cache

> [!info]
> This file is read at the start of every session. It captures recent decisions, current focus, and open questions. Always update it after significant changes.

---

## Current Focus

**Backend M1–M5 complete.** The full charter lifecycle works end-to-end: search → client-side SCA authorize → `POST /bookings` (lock + reserve + commit + capture-after-commit) → check-in (enqueues payout) → complete → payout transferred. Cancellation (pre-transfer) refunds from the snapshotted policy and releases availability + PREP atomically. Owner payout history at `GET /payments`. Secondary webhooks for `transfer.created` + `payment_intent.payment_failed` are routed. **140 e2e + 20 unit green.** Built with NestJS + Fastify + Prisma per Direction B. **Next: Milestone 6 (Admin)** — 4 endpoints (list/suspend users, list bookings, manual refund). Frontend not started.

## Recent Decisions (2026-05-27 — architecture simplification)

- Platform name: **YachtBay** (temporary — to be revisited)
- Launch market: **Greece only**
- Booking model: **Instant book**
- Revenue: **~13% commission from owners** (exact rate under analysis)
- Crew options: **Bareboat, skippered, fully crewed** — defined per yacht by owner
- Charter types: **Day trips + multi-day/weekly**
- Target customer: **Mid-range travelers** (primary)
- Payments: **Stripe Connect Accounts v2** — **separate charges + transfers** (platform captures, payout-queue transfers later); card **authorized client-side (SCA/3DS) → captured after commit**; payout released ~24–48h after check-in. *(Amended 2026-05-29 — see [[decisions/2026-05-27-sync-payment-capture]].)*
- Geo search: **Lat/lng + haversine SQL** — PostGIS deferred to scale
- Events: **Synchronous emails via EmailPort** — outbox table in schema, poller deferred
- Queues: **payout-queue only** — Redis from Milestone 4, other queues deferred
- CQRS: **Booking module only** — other modules use plain NestJS services
- Admin: **4 endpoints at MVP** — full admin panel post-traction

## Vault Status

- [x] CLAUDE.md written
- [x] Folder structure created
- [x] Core concept pages written (vision, business model, commission, market, booking, crew, personas)
- [x] 4 decision records written
- [x] Roadmap stub created
- [x] wiki/index.md initialized
- [x] Stripe Connect research documented
- [x] MVP scope defined — [[specs/mvp-scope]]
- [x] Competitor research complete — [[wiki/concepts/competitor-analysis]]
- [x] Backend tech stack decided — [[wiki/concepts/tech-stack]]
- [x] Technical architecture defined — [[specs/backend-architecture]]
- [ ] Product design — not started

## Open Questions

- What is the exact default commission rate? (~13% but needs final confirmation)
- What is the final platform name?

## Next Steps

1. **Start Milestone 6 (Admin)** — 4 endpoints, all `@Roles(Role.ADMIN)`: `GET /admin/users` (paginated list/search by name/email/role/status), `PATCH /admin/users/:id/suspend` (sets `status=SUSPENDED` + `suspensionReason`), `GET /admin/bookings` (paginated list with filters), `POST /admin/bookings/:id/refund` (admin-issued manual refund — bypasses the cancellation policy; warns `ownerPayoutAlreadySent: true` for post-transfer cases). Admin users seeded via a small CLI/script (no public registration). M6 is small and well-specified — the plan can be written directly without a separate stress-test.
2. **Frontend** (not started) — the storefront can already consume the live discovery + booking APIs (`POST /bookings/payment-intent` → Stripe.js SCA → `POST /bookings`); the owner panel can consume the full charter lifecycle. Feed [[specs/design/stitch-brief]] into Google Stitch, then connect the Stitch MCP to Claude Code to generate React Router 7 SSR code.
