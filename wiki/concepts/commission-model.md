---
title: Commission Model
tags:
  - concept
  - business
  - revenue
last-updated: 2026-05-01
---

# Commission Model

**Summary**: How YachtBay charges for its service — commission structure, rates, and flexibility.

---

## Current Model

Commission is charged to the **owner** on each completed booking.

| Tier | Rate | Applies To |
|------|------|------------|
| Default | ~13% | All owners (exact % under analysis) |
| Fleet / volume | ~10% | Owners who bring a large number of yachts to the platform |

The exact default percentage is still being analysed. The model is intentionally flexible — different rates can be set per owner in the system.

## Why Owner-Side Commission

- Keeps the renter experience clean — no surprise fees at checkout
- Owners are the supply side; incentivising them with volume discounts drives platform growth
- Easier to justify to owners as a cost of doing business (vs. a charge renters may resent)

See [[decisions/2026-05-01-commission-from-owners]].

## Future Options

The platform is designed to support:

- **Per-owner commission rates** — already part of the data model from day one
- **Renter-side service fee** — may be added in a future phase
- **Subscription tiers for owners** — possible future addition, not in MVP

## Design Principle

> Build the data model to support per-owner rates and renter-side fees from day one, even if only a single rate is active at launch. This avoids a painful migration later.

## Related Pages

- [[business-model]]
- [[decisions/2026-05-01-commission-from-owners]]
- [[user-personas]]
