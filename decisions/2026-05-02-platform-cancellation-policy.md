---
title: Platform-defined cancellation policy at MVP, extensible per owner
date: 2026-05-02
status: accepted
tags:
  - decision
  - product
  - business
---

# Platform-Defined Cancellation Policy at MVP

## Context

When a renter cancels a booking, we need a clear refund policy. The policy can be defined by the platform (one standard policy for all listings) or by each owner individually.

## Options Considered

- **Platform-defined**: YachtBay sets one standard cancellation policy applied to all listings.
- **Owner-defined**: Each owner sets their own policy per listing.
- **Hybrid**: Platform sets a default, owners can override.

## Decision

**Platform-defined policy at MVP.** The data model is built to support per-owner policies in the future without a migration.

## Default Policy (updated after competitor research)

Competitor research revealed our original thresholds (30/14 days) are tighter than market standard for weekly charters. Revised to separate day trips from multi-day/weekly:

### Day trips
| Cancellation timing | Refund |
|---|---|
| 5+ days before charter | 100% |
| 1–4 days before charter | 50% |
| Under 24 hours | No refund |

### Multi-day / Weekly charters
| Cancellation timing | Refund |
|---|---|
| 30+ days before charter | 100% |
| 14–29 days before charter | 50% |
| Under 14 days | No refund |

> [!important] Platform fees are non-refundable
> YachtBay's commission is never refunded on cancellation — the renter receives back the rental price portion only. This is industry standard (GetMyBoat, Click&Boat both follow this rule).

> [!note]
> These thresholds align with Sunsail and Click&Boat Strict policy for weekly charters. The Moorings uses 90 days for full refund — we may need to revise upward for premium/longer charters in Phase 2.

## Consequences

- Simple and consistent — renters know what to expect across all listings
- Builds renter trust — one clear policy is easier to communicate than per-owner variations
- Owners cannot differentiate on cancellation terms at MVP
- The data model must support a `cancellation_policy` field per listing from day one — even if all listings point to the same platform default at launch
- Per-owner custom policies can be unlocked in a future phase (e.g. as a premium owner feature)

## Related

- [[specs/mvp-scope]]
- [[wiki/concepts/business-model]]
- [[wiki/concepts/competitor-analysis]]
