---
title: Hot — Session Context Cache
tags:
  - meta
last-updated: 2026-06-03
---

# Hot — Session Context Cache

> [!info]
> This file is read at the start of every session. It captures recent decisions, current focus, and open questions. Always update it after significant changes.

---

## Current Focus

**Phase 1 backend ✅ COMPLETE (M1–M7).** Full marketplace works end-to-end: search → SCA authorize → book → check-in → complete → payout; cancel (pre-transfer, snapshot refund); admin (list/suspend/list-bookings/manual-refund with auto-clawback). Hardened: rate limiting (auth + booking writes 10/min/IP, global 100/min), Helmet headers, env-driven CORS, Stripe-signature verification audited. **156 e2e + 20 unit green.** All Direction B decisions respected throughout.

**Next move — full-stack monorepo migration** ([[decisions/2026-06-03-full-stack-monorepo]], supersedes the 2026-05-03 frontend-monorepo ADR). The standalone `yachties-backend` becomes `yachtbay/apps/api/`. Two frontend shells get scaffolded alongside: `apps/storefront/` (RR7 SSR — public consumer site) and `apps/panel/` (RR7 SPA — owners + admins via role-gated routes). Shared `packages/types/` re-exports backend Prisma + DTO types directly (no codegen). Then build the admin section of `apps/panel/` first (smallest scope, 4 endpoints, validates the monorepo shape), then owner views, then storefront.

**Operational launch** (Railway live env, Stripe live keys, domains, soft launch per `yachties-backend/docs/launch-checklist.md`) remains a parallel manual track — can happen any time the API is wanted in production.

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

1. **Monorepo migration** — see plan at `yachties-backend/docs/superpowers/plans/2026-06-03-monorepo-migration.md`. Create `/Users/philipposphilias/Desktop/Yacht Platfrom/yachtbay/`, scaffold pnpm workspaces + Turborepo, move backend into `apps/api/`, scaffold empty `apps/storefront/` and `apps/panel/` shells, scaffold `packages/types/` and `packages/api-client/`, retest the backend suite in its new home, archive `yachties-backend/`, reconfigure Railway service paths.
2. **Admin section of `apps/panel/`** — 4 screens against the existing 4 admin endpoints. Smallest scope; validates monorepo shape end-to-end before tackling owner views or storefront.
3. **Owner section of `apps/panel/`** — yacht management, calendar, payouts, Stripe Connect onboarding.
4. **Storefront** — feed [[specs/design/stitch-brief]] into Google Stitch, then connect the Stitch MCP to Claude Code to generate React Router 7 SSR code (listings, search, detail, checkout via `POST /bookings/payment-intent` → Stripe.js SCA → `POST /bookings`).
5. **Operational launch** (manual, per `yachties-backend/docs/launch-checklist.md` — moves into the monorepo as `apps/api/docs/launch-checklist.md`): Railway live env, Stripe live keys, custom domains (`api.yachtbay.com`, `yachtbay.com`, `owners.yachtbay.com`), R2 bucket CORS, monitoring, soft-launch. Can run in parallel with the frontend track as soon as the API is wanted in production.
6. **Phase 2 (post-traction)** — see `roadmap/overview.md`: reviews/ratings, in-platform messaging, owner-defined cancellation policies, seasonal pricing UI, analytics, mobile apps.
