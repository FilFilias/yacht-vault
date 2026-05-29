---
title: Hot — Session Context Cache
tags:
  - meta
last-updated: 2026-05-29
---

# Hot — Session Context Cache

> [!info]
> This file is read at the start of every session. It captures recent decisions, current focus, and open questions. Always update it after significant changes.

---

## Current Focus

**Backend M1–M3 complete.** Auth + users (M1), yacht listings + photos + pricing config (M2), and discovery — pricing engine + availability + search (M3) — all built and tested (97 e2e + 15 unit green). Built with NestJS + Fastify + Prisma per Direction B. **Next: Milestone 4 (Booking Core)** — Stripe Connect, BullMQ payout-queue, bookings CQRS with synchronous capture. Frontend not started.

## Recent Decisions (2026-05-27 — architecture simplification)

- Platform name: **YachtBay** (temporary — to be revisited)
- Launch market: **Greece only**
- Booking model: **Instant book**
- Revenue: **~13% commission from owners** (exact rate under analysis)
- Crew options: **Bareboat, skippered, fully crewed** — defined per yacht by owner
- Charter types: **Day trips + multi-day/weekly**
- Target customer: **Mid-range travelers** (primary)
- Payments: **Stripe Connect Accounts v2** — destination charges, synchronous capture inside booking transaction, payout 24–48h after charter start
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

1. **Start Milestone 4 (Booking Core)** — Stripe Connect onboarding + webhooks, Redis + BullMQ payout-queue, Resend adapter (`EmailPort`), bookings module (`CreateBooking` CQRS command: `SELECT FOR UPDATE` + synchronous Stripe capture inside the transaction), wire `applyPrepDays` on confirmation
2. **Frontend** (not started) — feed [[specs/design/stitch-brief]] into Google Stitch, then connect the Stitch MCP to Claude Code to generate React Router 7 SSR code (storefront search/detail can now consume the live M3 discovery API)
