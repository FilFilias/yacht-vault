---
title: Wiki Index
tags:
  - index
last-updated: 2026-05-01
---

# YachtBay Wiki Index

Master table of contents for the YachtBay knowledge vault.

---

## Concepts

| Page | Description |
|------|-------------|
| [[vision-and-mission]] | What YachtBay is, the problem it solves, and why Greece first |
| [[business-model]] | P2P marketplace structure, how value is created and captured |
| [[commission-model]] | Owner commission rates, flexible tiers, future fee options |
| [[target-market]] | Greece launch, key regions, renter and owner segments |
| [[booking-model]] | Instant book, day trips and multi-day charters, renter journey |
| [[crew-options]] | Bareboat, skippered, and fully crewed configurations |
| [[user-personas]] | Owner persona and renter persona |
| [[stripe-connect]] | How Stripe Connect works — account types, fees, payment flow |
| [[tech-stack]] | Final technology decisions — backend, frontend, infrastructure, third-party services |
| [[competitor-analysis]] | Full competitor analysis — commissions, features, cancellation policies, YachtBay positioning |

## Decisions

| File | Summary |
|------|---------|
| [[decisions/2026-05-01-commission-from-owners]] | Charge ~13% commission from owners only at launch |
| [[decisions/2026-05-01-greece-first-launch]] | Focus on Greece first, expand later |
| [[decisions/2026-05-01-instant-booking]] | Instant book as default — no owner approval step |
| [[decisions/2026-05-01-platform-name-yachtbay]] | YachtBay as working name (temporary) |
| [[decisions/2026-05-01-stripe-connect-express]] | ~~Use Stripe Connect Express~~ — superseded |
| [[decisions/2026-05-01-stripe-connect-accounts-v2]] | Use Stripe Connect Accounts v2 for marketplace payments |
| [[decisions/2026-05-02-web-app-mvp]] | Web app only at MVP, native mobile apps post-traction |
| [[decisions/2026-05-02-platform-cancellation-policy]] | Platform-defined cancellation policy, extensible per owner |
| [[decisions/2026-05-03-backend-architecture]] | NestJS modular monolith with CQRS, Domain Events, Ports & Adapters |
| [[decisions/2026-05-03-frontend-monorepo]] | Separate storefront + owner panel apps in a Turborepo monorepo |

## Specs

| Page | Description |
|------|-------------|
| [[specs/mvp-scope]] | Full MVP feature list — owner, renter, platform, and what's deferred |
| [[specs/backend-architecture]] | Definitive backend architecture — stack, patterns, data model, booking workflow, extensibility |
| [[specs/user-stories/renter]] | Renter user stories R-001→R-026 — auth, search, booking, cancellation, profile |
| [[specs/user-stories/owner]] | Owner user stories O-001→O-026 — onboarding, listings, availability, bookings, payouts |
| [[specs/user-stories/admin]] | Admin user stories A-001→A-016 — users, listings, bookings, revenue, commission |
| [[specs/design/sitemap-storefront]] | Storefront site map — 18 pages, route structure, auth requirements, navigation |
| [[specs/design/sitemap-owner-panel]] | Owner panel site map — 25 pages, listing wizard, availability, bookings, payouts |
| [[specs/design/sitemap-admin]] | Admin panel site map — 10 pages, users, listings, bookings, revenue, settings |
| [[specs/design/stitch-brief]] | Complete Google Stitch design brief — all 18 storefront pages, user stories, components, design direction |
| [[specs/data-model]] | Final Prisma schema — all entities, enums, indexes, and design decisions |
| [[specs/api-contract]] | Full API contract — 54 endpoints across 8 modules with request/response shapes |
| [[specs/env-variables]] | All environment variables for backend, storefront, owner panel, and admin |

## Roadmap

| Page | Description |
|------|-------------|
| [[roadmap/overview]] | High-level phases: MVP, Growth, Expansion |
| [[roadmap/phase-1-development]] | 7-milestone MVP build order — foundation to launch |

## Entities

| Page | Description |
|------|-------------|
| [[getmyboat]] | Largest global P2P boat rental marketplace. Operates in Greece. Inquiry-based booking. |
| [[clickandboat]] | European P2P + pro marketplace. Instant book. Up to 23% owner commission. |
| [[boataround]] | European marketplace, active in Greece. Flexible booking days. Payment issues reported. |
| [[anchorsbook]] | Local Greek marketplace. 20% owner-only commission. Smaller scale. |
| [[sailogy]] | Large global charter marketplace (absorbed Zizoo + Borrow A Boat). Pro-charter focused. |

## Sources

| Page | Description |
|------|-------------|
| [[sources/yacht-booking-system-summary]] | Architecture proposal from dedicated backend design session — what was adopted and what was changed |
