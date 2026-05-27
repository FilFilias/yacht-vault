---
title: Backend architecture — NestJS modular monolith with CQRS, Domain Events, and Ports & Adapters
date: 2026-05-03
status: accepted
tags:
  - decision
  - technical
  - backend
  - architecture
---

# Backend Architecture Decision

## Context

YachtBay needs a backend that:
- Ships an MVP fast (solo developer)
- Handles booking complexity (state machine, double-booking prevention, payment splits)
- Is extensible without rewriting (promotions, custom pricing, messaging, reviews)
- Scales with traffic growth (stateless, horizontally scalable)
- Is built on TypeScript throughout

## Decision

**Modular Monolith** — single deployable unit with bounded domain modules, combined with:

- **CQRS** via `@nestjs/cqrs` — explicit command handlers for complex writes, direct queries for reads
- **Domain Events + Transactional Outbox** — decoupled side effects (email, payment, notifications) with guaranteed delivery
- **Strategy Pattern** — pluggable pricing rules, commission rates, cancellation policies
- **Ports & Adapters** — all external integrations (Stripe, Resend, Cloudinary) behind interfaces

## Technology Stack Summary

| Concern | Choice |
|---|---|
| Framework | NestJS + Fastify adapter |
| Database | PostgreSQL + Prisma |
| Queues | BullMQ + Redis |
| Auth | Custom JWT (`@nestjs/jwt` + `@nestjs/passport`) |
| Payments | Stripe Connect Accounts v2 |
| Email | Resend + React Email |
| Images | Cloudinary |
| Hosting | Railway |

## Why Modular Monolith over Microservices

Booking + payment atomicity is trivial in a monolith and painful in microservices (distributed transactions). A well-structured modular monolith handles significant traffic. Migration to microservices later is possible — bounded modules + event bus is the same foundation.

## Why NestJS

The architecture we chose (CQRS, DI, Guards, Pipes, domain modules) maps exactly to NestJS built-ins. Using Hono or Fastify standalone would require building the same structure manually.

## Consequences

- Clean extension points for every planned future feature (promotions, messaging, custom pricing, reviews)
- Stateless API — horizontal scaling with zero architectural changes
- Transactional outbox guarantees no lost payment events on process crash
- Three-layer double-booking prevention (pessimistic lock + optimistic concurrency + DB constraint)
- Ports & Adapters means any infrastructure can be swapped with a new adapter

## Related

- [[specs/backend-architecture]]
- [[decisions/2026-05-01-stripe-connect-accounts-v2]]
- [[wiki/concepts/stripe-connect]]
