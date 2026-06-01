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

**Phase 1 backend ✅ COMPLETE (M1–M7).** Full marketplace works end-to-end: search → SCA authorize → book → check-in → complete → payout; cancel (pre-transfer, snapshot refund); admin (list/suspend/list-bookings/manual-refund with auto-clawback). Hardened: rate limiting (auth + booking writes 10/min/IP, global 100/min), Helmet headers, env-driven CORS, Stripe-signature verification audited. **156 e2e + 20 unit green.** All Direction B decisions respected throughout. **Two parallel tracks remain:** (a) **operational launch** — manual ops per `docs/launch-checklist.md` (Railway env, Stripe live, domains, soft launch); (b) **frontend** — storefront + owner panel + admin panel, not started.

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

1. **Operational launch** (manual, per `yachties-backend/docs/launch-checklist.md`) — set Railway env to live values, switch Stripe to live keys (`sk_live_…` / `pk_live_…` + the live `whsec_…`), point custom domains (`api.yachtbay.com`, `yachtbay.com`, `owners.yachtbay.com`), configure R2 bucket CORS for the prod domains, wire monitoring (Railway logs + Stripe alerts), then soft-launch with friends-and-family.
2. **Frontend** — start the parallel track. The backend API is fully ready (`POST /bookings/payment-intent` → Stripe.js SCA → `POST /bookings`; owner Connect onboarding; admin endpoints). Feed [[specs/design/stitch-brief]] into Google Stitch, then connect the Stitch MCP to Claude Code to generate React Router 7 SSR code. Three apps per [[decisions/2026-05-03-frontend-monorepo]]: storefront (SSR), owner panel (SPA), admin panel (SPA).
3. **Phase 2 (post-traction)** — see `roadmap/overview.md`: reviews/ratings, in-platform messaging, owner-defined cancellation policies, seasonal pricing UI, analytics, mobile apps.
