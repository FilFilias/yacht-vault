---
title: Full-stack monorepo — api + storefront + panel under one repo
date: 2026-06-03
status: accepted
supersedes:
  - 2026-05-03-frontend-monorepo
tags:
  - decision
  - architecture
  - monorepo
  - frontend
  - backend
---

# Full-Stack Monorepo

## Context

The [[decisions/2026-05-03-frontend-monorepo]] ADR committed the **frontends** to a Turborepo (storefront SSR + owner panel SPA). The backend lived in a separate repo (`yachties-backend`). The original ADR also assumed the admin panel would be a third app, alongside owner panel.

Between then and now, two things changed:

1. **Admin is just another role-gated view** (`@Roles(ADMIN)`), not a separate UX class. Both owner and admin are auth-gated data-dense SPAs — same rendering mode, same auth flow, same design language. The original ADR's argument for separating storefront from owner-panel ("consumer experience vs operational tool") does **not** extend to owner-vs-admin. Splitting them into two apps duplicates layout, auth, navigation, and design system code with no UX payoff.

2. **Backend ↔ frontend type drift is the most expensive thing a solo dev does.** With backend in a separate repo, `packages/types/` either codegen-s from OpenAPI (loses Prisma model fidelity), publishes to private npm (release ritual on every change), or hand-mirrors types (drift waiting to happen). All three are strictly worse than direct cross-workspace TypeScript imports.

Phase 1 backend (M1–M7) is shipped and `yachties-backend/` has zero downstream consumers yet — the cheapest possible moment to restructure.

## Options Considered

**A. Two separate frontend repos + standalone backend repo (status quo from 2026-05-03 ADR)**
- 3 repos total. Each frontend has its own CI, deps, types. Backend stays put.
- Pro: simplest per-repo, no monorepo tooling.
- Con: type drift across 3 boundaries; duplicated UI / API client / auth in both frontends; no atomic cross-stack PRs.

**B. Frontend monorepo (3 apps: storefront, owner-panel, admin-panel) + standalone backend repo**
- The original 2026-05-03 plan.
- Pro: shared frontend code via packages.
- Con: still has the backend ↔ frontend type drift problem; admin-as-separate-app duplicates auth/layout/nav with owner-panel for no UX benefit.

**C. Full-stack monorepo (this ADR)** — `apps/api`, `apps/storefront`, `apps/panel` (owner + admin role-gated) under one repo.
- Pro: direct TypeScript imports from backend → `packages/types/` → frontends; atomic cross-stack PRs; one CI; one dependency upgrade workflow; one shared design system.
- Con: ~half-day migration cost; Railway services need root-directory + watch-path config.

## Decision

**Option C — full-stack monorepo at `/Users/philipposphilias/Desktop/Yacht Platfrom/yachtbay/`**, with `yachties-backend` archived after the migration.

```
yachtbay/
├── apps/
│   ├── api/                    NestJS + Fastify + Prisma (current yachties-backend)
│   ├── storefront/             React Router 7 SSR — public consumer site
│   └── panel/                  React Router 7 SPA — owner + admin, role-gated routes
├── packages/
│   ├── types/                  re-exports backend DTOs & Prisma types
│   └── api-client/             typed fetch + React hooks for the API
├── pnpm-workspace.yaml
├── turbo.json
└── package.json
```

`packages/ui/` and `packages/utils/` are **not** created upfront — YAGNI. Add them only when at least two apps need genuinely shared code.

## Why Owner + Admin Live in the Same App

| | Storefront | Panel |
|---|---|---|
| **Audience** | Renters (public) | Owners + Admins (authenticated) |
| **UX style** | Visual, photo-led | Functional, data-dense |
| **SEO** | Required — listings crawlable | None |
| **Rendering** | React Router 7 SSR | React Router 7 SPA |
| **Routes** | `/`, `/yachts/:id`, `/checkout`, `/account` | `/yachts`, `/bookings`, `/payouts` (OWNER), `/admin/users`, `/admin/bookings` (ADMIN) |

Inside `panel/`, role gates work the same way the backend already does — RR7 loaders check the JWT's `roles` claim. Route-level code splitting ensures owners don't ship admin bundle to their browsers, so the security/payload argument for separating admin into its own app collapses.

A founder who is both OWNER and ADMIN (you) gets one app, one login, one navigation, role-aware sidebar. A future support hire gets `ADMIN` only and sees only admin routes — no extra app needed.

## Why the Backend Belongs In

The killer feature is direct TypeScript imports:

```ts
// packages/types/src/index.ts
export type { Reservation, Yacht, User } from '@yachtbay/api/prisma-client';
export type { CreateBookingDto } from '@yachtbay/api/src/modules/bookings/dto/create-booking.dto';
```

- No build step between backend and frontend types.
- A Prisma field rename instantly breaks `tsc` in every frontend file that uses the old name — same PR, same CI run.
- Cross-stack changes ship atomically — no "deploy backend first, then frontend, hope no in-flight requests carry the old shape" dance.

## Consequences

- **Single repo `yachtbay/`** replaces `yachties-backend/`. The standalone repo gets archived (tagged + kept read-only) but no longer accepts new commits.
- **Railway** — three services (api, storefront, panel), each configured with its own root directory (`apps/api`, `apps/storefront`, `apps/panel`) and watch paths (`apps/{name}/**` + shared `packages/**` + root configs).
- **CLAUDE.md** moves to the monorepo root and is rewritten to cover the whole stack. Per-app CLAUDE.md files in `apps/*/` are added only if they carry app-specific guidance the root doesn't.
- **`packages/types/` is the contract.** Both `apps/storefront/` and `apps/panel/` import API types only via `@yachtbay/types` — never reach into `@yachtbay/api/...` directly. This preserves a clear "the backend is a dependency, not an internal detail" boundary.
- **CI** runs `turbo run lint test build` from the root, with `--filter` flags for service-scoped pipelines.
- **Phase 1 backend test suite (156 e2e + 20 unit) must pass unchanged in its new home** as the acceptance gate for the migration.
- **The frontend dev experience gains hot-reloading types** — when the backend changes a DTO, the frontend dev server sees the new shape on next save.
- **`yacht-vault/` stays separate.** It is documentation, not code, and has its own Obsidian workflow.

## Migration Approach

- **Git history:** start the monorepo with a clean initial commit; archive `yachties-backend/` with a final tag. Trying to preserve history via `git subtree` / `filter-branch` adds 1-2 hours of surgery for marginal benefit — Phase 1 is already documented in the vault log.
- **Backend code:** copied wholesale into `apps/api/`. Internal Nest module structure (`src/modules/*`) does not change. The 176 tests must pass on first run in the new location.
- **Frontend apps:** scaffolded as empty React Router 7 shells in this migration; actual screens come in follow-up milestones.
- **Auto-push background process:** retargeted at the new repo.

See the migration plan at `yachties-backend/docs/superpowers/plans/2026-06-03-monorepo-migration.md` (archived after migration; equivalent will move into the new repo).

## Related

- [[decisions/2026-05-03-frontend-monorepo]] — superseded by this ADR
- [[decisions/2026-05-27-minimal-admin-mvp]] — admin is 4 endpoints; informs the "no separate admin app" call
- [[wiki/concepts/tech-stack]]
- [[specs/mvp-scope]]
