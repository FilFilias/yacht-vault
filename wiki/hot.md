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

**Backend M1–M6 complete.** Full charter lifecycle (book → check-in → complete → payout) plus cancellation (pre-transfer + snapshot-based refund) plus admin (list/suspend users, list/refund bookings — refund auto-clawbacks via Stripe `reverse_transfer` if payout already sent). All 6 milestones built with NestJS + Fastify + Prisma per Direction B. **153 e2e + 20 unit green.** **Next: Milestone 7 (Polish & Launch)** — backend hardening (rate limiting, Helmet, CORS for prod domains, audit existing tests), then operational launch (Railway env, domains, Stripe live keys, soft launch). Frontend not started.

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

1. **Start Milestone 7 (Polish & Launch)** — backend hardening only (ops/launch is manual): rate limiting via `@nestjs/throttler` (strict on auth, looser on reads), `helmet` security headers, CORS finalized for prod domains, a Stripe-webhook-signature sanity check, and any critical-path test gaps the audit surfaces (e.g., a `computeRefund`-via-policy-strategy unit if we extract one). M7 is mostly hardening, so the plan can be written directly without a separate stress-test.
2. **Operational launch** (manual, not code) — set Railway env to live values, switch Stripe to live keys (`sk_live_`/`pk_live_`), point custom domains (`api.yachtbay.com`, `yachtbay.com`, `owners.yachtbay.com`), confirm R2 pre-signed URL expiry + CORS, soft-launch with friends-and-family, watch logs.
3. **Frontend** (not started) — the storefront + owner panel can already consume the full live API. Feed [[specs/design/stitch-brief]] into Google Stitch, then connect the Stitch MCP to Claude Code to generate React Router 7 SSR code.
