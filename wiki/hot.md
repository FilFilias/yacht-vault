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

**Backend M1–M4 complete.** M1 (auth/users), M2 (yacht listings + photos + pricing config), M3 (pricing engine + availability + search), M4a (Stripe Connect onboarding + webhook), M4b (bookings + payout). The full booking flow works end-to-end: search → client-side SCA authorize → `POST /bookings` (Yacht-row lock + reserve + commit + capture-after-commit) → confirmation emails → payout-queue ready to release at M5 check-in. **122 e2e + 15 unit green.** Built with NestJS + Fastify + Prisma per Direction B. **Next: Milestone 5 (Operations)** — cancellation/refunds, check-in/complete, `GET /payments`, wire `applyPrepDays`. Frontend not started.

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

1. **Start Milestone 5 (Operations)** — `PATCH /bookings/:id/cancel` (refund from `cancellationPolicySnapshot` + Stripe refund + availability release + dequeue any pending payout job), `PATCH /bookings/:id/check-in` (enqueues the `payout-queue` delayed job → wires up the M4 plumbing; also triggers `applyPrepDays` — the helper that's been sitting unwired since M3), `PATCH /bookings/:id/complete`, `GET /payments` (owner payout history), cancellation + completion emails. The webhook handler should also gain the secondary `payment_intent.*` / `transfer.created` routes.
2. **Frontend** (not started) — the storefront can already consume the live discovery + booking APIs through `POST /bookings/payment-intent` → Stripe.js (SCA) → `POST /bookings`. Feed [[specs/design/stitch-brief]] into Google Stitch, then connect the Stitch MCP to Claude Code to generate React Router 7 SSR code.
