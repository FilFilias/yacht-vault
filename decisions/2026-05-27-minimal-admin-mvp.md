---
title: Minimal admin panel at MVP — 4 endpoints for soft launch
date: 2026-05-27
status: accepted
tags:
  - decision
  - technical
  - backend
  - admin
---

# Minimal admin panel at MVP — 4 endpoints for soft launch

## Context

The original architecture specified 16 admin endpoints covering users, listings, bookings, revenue, and settings. For a soft launch with a small number of real users and no public traffic, most of these are operationally unnecessary before traction is proven.

## Decision

**Implement 4 admin endpoints at Milestone 6 (soft launch):**

1. `GET /admin/users` — list/search users
2. `PATCH /admin/users/:id/suspend` — emergency user suspension
3. `GET /admin/bookings` — list all platform bookings
4. `POST /admin/bookings/:id/refund` — issue manual refund

These cover the two genuine operational needs at soft launch: (1) handling a bad actor by suspending their account, and (2) resolving a payment dispute by issuing a manual refund.

**Deferred to post-launch (12 endpoints):**
- `GET /admin/users/:id`, `PATCH /admin/users/:id/reactivate`, `PATCH /admin/users/:id/commission`
- `GET /admin/yachts`, `GET /admin/yachts/:id`, `PATCH /admin/yachts/:id/status`, `DELETE /admin/yachts/:id`
- `GET /admin/bookings/:id`, `POST /admin/bookings/:id/trigger-payout`
- `GET /admin/revenue`, `GET /admin/settings`, `PATCH /admin/settings`

## Consequences

- Total MVP API endpoints: 42 (down from 54)
- Admin panel frontend: 2 pages (`/users`, `/bookings`) instead of 8
- Revenue reporting, settings editing, per-owner commission management deferred
- All deferred schema fields (`commissionRate`, `suspensionReason`, etc.) are already in the Prisma schema — no schema work required when adding these endpoints later

## Migration path

Post-launch: implement deferred endpoints one by one as operational needs arise. Each endpoint is self-contained — adding them does not require changes to existing endpoints.

## Related

- [[specs/api-contract]]
- [[specs/mvp-scope]]
- [[roadmap/phase-1-development]]
- [[specs/data-model]]
