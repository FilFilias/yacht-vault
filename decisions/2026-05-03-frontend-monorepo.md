---
title: Separate frontend apps in a monorepo — storefront + owner panel
date: 2026-05-03
status: accepted
tags:
  - decision
  - frontend
  - architecture
---

# Separate Frontend Apps in a Monorepo

## Context

YachtBay has two distinct user types — renters (consumer experience) and owners (business tool). The question was whether to build a single unified frontend or two separate applications.

## Options Considered

**Single unified platform**
One app, role-based sections. Owner dashboard lives inside the same app as the storefront. Airbnb model.

- Pro: one codebase, one deployment, seamless owner/renter switching
- Con: fundamentally different UX goals in one app — dashboard clutter risks compromising consumer experience and vice versa. Role-based conditional rendering pollutes components.

**Separate apps in a monorepo**
Two applications sharing code via internal packages. Single GitHub repo, Railway deploys both automatically.

- Pro: clean separation, each app optimised for its purpose, independent deployments, shared packages avoid duplication
- Con: more initial setup — neutralised by AI-assisted development and Railway auto-deploy

## Decision

**Separate apps in a monorepo**, orchestrated with Turborepo.

```
apps/
  storefront/       ← React Router 7 (SSR) — public consumer site
  owner-panel/      ← React Router 7 (SPA mode) — owner dashboard
packages/
  ui/               ← shared components (Button, Input, Card, etc.)
  types/            ← shared TypeScript types and interfaces
  api-client/       ← shared API client and hooks
  utils/            ← shared utilities and helpers
```

## Why the Two Apps Are Fundamentally Different

| | Storefront | Owner Panel |
|---|---|---|
| **Goal** | Discovery, conversion, trust | Operations, management |
| **UX style** | Visual, consumer-grade | Functional, data-heavy |
| **SEO** | Required — listing pages must be crawlable | Not needed — behind auth |
| **Rendering** | SSR (React Router 7) | SPA mode (no SSR overhead) |
| **Key UI** | Photo galleries, maps, booking flows | Tables, calendars, forms, charts |

## Consequences

- Storefront uses React Router 7 in full SSR mode — critical for listing page SEO
- Owner panel uses React Router 7 in SPA mode — lighter, no SSR needed since no SEO requirement
- Shared `ui` package means design consistency without duplicating components
- Shared `types` package ensures type safety across frontend and backend API contracts
- Turborepo caches builds — only rebuilds what changed
- Railway deploys both apps from the same repo via separate service configs
- A user who is both owner and renter has two entry points — a clear "Switch to Owner Dashboard" link on the storefront handles this

## Related

- [[decisions/2026-05-03-backend-architecture]]
- [[wiki/concepts/tech-stack]]
- [[specs/mvp-scope]]
