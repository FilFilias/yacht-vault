---
title: Charge commission from owners, not renters
date: 2026-05-01
status: accepted
tags:
  - decision
  - revenue
---

# Charge Commission from Owners, Not Renters

## Context

YachtBay needs a revenue model. The two natural options in a two-sided marketplace are: charge the supply side (owners), the demand side (renters), or both.

## Options Considered

- **Owner-only commission**: Owner pays a % of each booking to YachtBay
- **Renter-only service fee**: Renter pays a fee on top of the rental price
- **Both sides**: Owner commission + renter service fee (e.g. Airbnb model)

## Decision

Start with **owner-only commission at ~13%** (exact rate under analysis). The platform is built to support per-owner rates from day one — fleet owners who bring volume may negotiate lower rates (e.g. 10%).

A renter-side service fee is explicitly kept as a future option but not activated at launch.

## Consequences

- Renter checkout is clean — no surprise fees, which helps conversion
- Owners absorb the full cost — commission rate must be competitive vs. brokers
- The data model must support variable commission rates per owner from day one
- Adding a renter fee later is a product and legal decision; the architecture should not block it

## Related

- [[wiki/concepts/commission-model]]
- [[wiki/concepts/business-model]]
