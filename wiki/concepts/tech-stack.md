---
title: Tech Stack
tags:
  - concept
  - technical
  - stack
last-updated: 2026-05-03
---

# Tech Stack

**Summary**: Final technology decisions for YachtBay — backend, frontend, infrastructure, and third-party services. Includes rationale and alternatives where future migration may be needed.

---

## Backend

| Layer | Choice | Rationale |
|---|---|---|
| **Runtime** | Node.js | Team expertise, strong async I/O |
| **Language** | TypeScript (strict) | Type safety — non-negotiable |
| **Framework** | NestJS | Enforced modular structure, built-in DI, CQRS module, Guards, Pipes |
| **HTTP adapter** | Fastify (via NestJS adapter) | 2–3× better throughput than Express with near-zero code changes |
| **ORM** | Prisma | Type-safe queries, auto-generated migrations, Prisma Studio, best NestJS ecosystem support |
| **Job queues** | BullMQ + Redis | Persistent async jobs, retry on failure, delayed jobs (payout scheduling) |
| **Auth** | Custom JWT — `@nestjs/jwt` + `@nestjs/passport` | Full control, no vendor dependency, role-based (owner / renter / admin) |
| **Scheduler** | `@nestjs/schedule` | Outbox poller, payout release, availability cleanup |

See [[specs/backend-architecture]] for full architecture detail.

---

## Frontend

Two separate apps sharing code via a monorepo (Turborepo). Single GitHub repo, Railway deploys both.

| Layer | Choice | Rationale |
|---|---|---|
| **Monorepo tool** | Turborepo | Build orchestration, caching, single repo for all frontend apps |
| **Storefront framework** | React Router 7 (SSR mode) | Founder expertise (3 years), SSR for listing page SEO, consumer-facing |
| **Owner panel framework** | React Router 7 (SPA mode) | Same stack, no SSR needed (behind auth, no SEO requirement) |
| **Language** | TypeScript (strict) | Consistent with backend |
| **Shared packages** | `ui`, `types`, `api-client`, `utils` | Code reuse across apps without duplication |

See [[decisions/2026-05-03-frontend-monorepo]] for full rationale.

---

## Database

| Layer | Choice | Rationale |
|---|---|---|
| **Primary DB** | PostgreSQL | ACID transactions, row-level locking, date-range queries |
| **Geo extension** | PostGIS | Geographic search ("yachts near X") — built on top of PostgreSQL, no extra service |
| **Cache / queues** | Redis | BullMQ backing store, future caching layer |

---

## Infrastructure & Hosting

| Layer | Choice | Rationale |
|---|---|---|
| **Hosting** | Railway | Managed PostgreSQL + Redis, simple Node.js deployments, scales to containers |
| **CI/CD** | GitHub Actions | Push to `main` → tests → Railway auto-deploy. Free, simple, sufficient for solo developer |
| **Image storage** | Cloudflare R2 | S3-compatible, zero egress fees, pairs with Cloudflare CDN. No built-in image transformation — handle resizing at upload time |

> [!note] R2 vs Cloudinary
> Cloudflare R2 is pure object storage. Unlike Cloudinary it does not transform images on-the-fly. Yacht photo resizing and optimization must be handled client-side (before upload) or via a Cloudflare Worker. If image transformation becomes a pain point, **Cloudinary** is the natural migration path — it supports R2 as a source.

---

## Third-Party Services

| Service | Choice | Rationale |
|---|---|---|
| **Payments** | Stripe Connect Accounts v2 | Marketplace payment splitting, platform-controlled payouts, non-deprecated API |
| **Email** | Resend + React Email | TypeScript-native, React-based templates, excellent DX |
| **Maps** | Mapbox | See below |
| **Geo search** | PostGIS (PostgreSQL extension) | Sufficient at MVP scale, no extra service |
| **SMS** | Deferred (Phase 2) | Email covers MVP needs — add Twilio when data justifies it |

---

## Maps — Mapbox (current) and Alternatives

### Current choice: Mapbox

| | |
|---|---|
| **Free tier** | 50,000 map loads/month |
| **Pricing beyond free** | ~$0.50–$1 per 1,000 loads (cheaper than Google) |
| **React SDK** | `react-map-gl` — clean, well-maintained |
| **Customisation** | Full style control via Mapbox Studio |
| **Use case at MVP** | Display yacht home ports, location-based search area picker |

### Alternatives for future consideration

**Google Maps**
- More familiar to end users — higher trust recognition
- Better POI data (marinas, harbours labelled more accurately)
- More expensive: $7 per 1,000 map loads, strict terms of service
- Migrate to if: user research shows renters expect Google Maps, or POI accuracy becomes important

**HERE Maps**
- Strong marine and nautical chart data — relevant for a yacht platform
- Competitive pricing, less known but solid API
- Migrate to if: nautical-specific map layers become a product requirement

**OpenStreetMap + Leaflet**
- Free, open-source, no vendor lock-in
- Less polished out of the box, requires more custom styling
- Migrate to if: cost becomes a concern at scale and custom styling investment is acceptable

> [!tip] Migration path
> All map interactions will be abstracted behind a `MapPort` interface in the frontend. Swapping Mapbox for another provider = new adapter implementation, no component changes.

---

## What Was Rejected and Why

| Option | Rejected because |
|---|---|
| Drizzle ORM | Less mature NestJS integration, fewer examples. Prisma wins for a developer new to Node.js ORMs |
| Clerk / Auth0 | Vendor dependency, cost at scale, loss of user data control |
| Cloudinary | R2 chosen for cost efficiency. Cloudinary remains the migration path if image transformation is needed |
| Algolia / Meilisearch | PostGIS is sufficient at MVP scale, no extra service or cost needed |
| Google Maps | More expensive, Mapbox free tier is generous for MVP |
| Express (NestJS default) | Fastify gives 2–3× throughput at near-zero extra cost |
| Medusa | E-commerce framework — wrong domain model for a P2P marketplace |
| Microservices | Distributed transaction complexity, operational overhead for a solo developer |

---

## Related Pages

- [[specs/backend-architecture]]
- [[wiki/concepts/stripe-connect]]
- [[decisions/2026-05-03-backend-architecture]]
