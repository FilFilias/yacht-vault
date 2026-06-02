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

**Phase 1 backend ✅ COMPLETE (M1–M7).** Full marketplace works end-to-end. **156 e2e + 20 unit green.** All Direction B decisions respected throughout.

**Full-stack monorepo migration ✅ COMPLETE (2026-06-03).** Backend moved into `yachtbay/apps/api/` at `/Users/philipposphilias/Desktop/Yacht Platfrom/yachtbay/`. Five workspaces wired with pnpm + Turborepo: `@yachtbay/api`, `@yachtbay/storefront` (RR7 SSR shell), `@yachtbay/panel` (RR7 SPA shell), `@yachtbay/types` (Prisma + DTO re-exports), `@yachtbay/api-client` (typed fetch wrapper). Clean-room verification: clean install + full build + 156 e2e + 20 unit, all green in the new home. Root + per-app CLAUDE.md updated. See [[decisions/2026-06-03-full-stack-monorepo]].

**Next implementation track — admin section of `apps/panel/`** (smallest scope, 4 endpoints, 4-5 screens). Validates the monorepo end-to-end before tackling owner views or the SSR storefront. After that: owner section of `apps/panel/`, then `apps/storefront/`.

**Operational launch** (Railway live env, Stripe live keys, domains, soft launch per `yachtbay/apps/api/docs/launch-checklist.md`) remains a parallel manual track — can happen any time the API is wanted in production. **Operator follow-up from the migration:** Railway service paths point at `apps/api/` (root dir + watch paths); archive `yachties-backend/` repo with a final tag; ensure CI installs from monorepo root.

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
