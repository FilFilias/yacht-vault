---
title: Competitor Analysis
tags:
  - concept
  - market
  - competitors
last-updated: 2026-05-03
---

# Competitor Analysis

**Summary**: Analysis of YachtBay's direct competitors — commission structures, booking models, features, cancellation policies, and key takeaways for product and business decisions.

---

## Competitive Landscape

| Competitor       | Model               | Commission                   | Booking        | Greece                  | Instant Book |
| ---------------- | ------------------- | ---------------------------- | -------------- | ----------------------- | ------------ |
| [[getmyboat]]    | P2P global          | 8–10% owner + 15–20% renter  | Inquiry-based  | ✓ Strong (1,100+ boats) | ✗            |
| [[clickandboat]] | P2P + pro, Europe   | Up to 23% owner + renter fee | Instant book   | ✓ Present               | ✓            |
| [[boataround]]   | Marketplace, Europe | Undisclosed (owner-side)     | Online booking | ✓ Present               | Unclear      |
| [[anchorsbook]]  | Marketplace, Greece | 20% owner only               | Online booking | ✓ Local                 | Unclear      |
| [[sailogy]]      | Pro charter, global | Undisclosed                  | Online booking | ✓ Present               | ✓            |

---

## Key Insights

### 1. Commission — YachtBay is the cheapest option for owners

| Platform | Owner commission | Renter fee | Total platform take |
|---|---|---|---|
| GetMyBoat | 8–10% | 15–20% | ~23–30% |
| Click&Boat | Up to 23% | Undisclosed (~5–8%) | ~25–30% |
| AnchorsBook | 20% | None | 20% |
| **YachtBay (planned)** | **13%** | **None (MVP)** | **13%** |

**Takeaway**: YachtBay's 13% owner-only commission is significantly lower than all competitors. This is a strong owner acquisition argument — "lowest commission in Greece."

**Important nuance**: Most competitors charge both sides. Adding a renter service fee of 5–8% in Phase 2 would bring total platform take to ~18–21% — still competitive vs. Click&Boat and GetMyBoat. Worth revisiting when traction is established. See [[commission-model]].

---

### 2. Booking model — Instant book is table stakes, and GetMyBoat doesn't have it

GetMyBoat — the market leader — still uses an inquiry-based model. This is a meaningful friction point: renters must wait for the owner to respond before the booking is confirmed.

YachtBay's instant book is a **real differentiator against the largest player in the market.** This should be highlighted prominently in owner and renter marketing.

---

### 3. Cancellation policy — market standard is more generous than our draft

Our proposed thresholds (30 days full refund, 14 days 50%, under 14 days 0%) are tighter than the market:

| Platform/operator | Full refund | Partial refund | No refund |
|---|---|---|---|
| GetMyBoat (default) | 5+ days | — | Under 5 days |
| Click&Boat Flexible | 1+ day | — | Under 1 day |
| Click&Boat Moderate | 10+ days | — | Under 10 days |
| Click&Boat Strict | 30+ days (60%) | — | Under 30 days |
| Sunsail (Greece) | 30+ days | Under 30 days | Under 14 days |
| The Moorings (Greece) | 90+ days | 60–89 days | Under 30 days |
| **YachtBay (draft)** | **30+ days** | **14–29 days (50%)** | **Under 14 days** |

**Takeaway**: For **weekly charters** (which are the higher-value bookings in Greece), our thresholds are tighter than professional operators like The Moorings (90 days) and Sunsail (30 days). We should consider separate policies for day trips vs. weekly charters. See [[decisions/2026-05-02-platform-cancellation-policy]].

**Recommended revision**:
- Day trips: 5+ days full refund, 1–4 days 50%, under 24h no refund
- Multi-day / weekly: 30+ days full refund, 14–29 days 50%, under 14 days no refund

---

### 4. Platform fees are non-refundable — industry standard

Both GetMyBoat and Click&Boat explicitly state that platform fees (service fee + commission) are never refunded on cancellation, regardless of the cancellation policy. The renter gets back the rental price (minus fees) if they cancel within policy.

**Takeaway**: YachtBay should adopt this standard — platform commission is non-refundable on cancellation. Protects revenue, aligns with market expectation.

---

### 5. Owner-defined vs platform-defined cancellation — a Phase 2 opportunity

GetMyBoat and Click&Boat both let owners choose their cancellation policy. Click&Boat offers 4 predefined tiers (flexible, moderate, strict, custom). This gives owners more control and attracts both flexible and strict operators.

**Takeaway**: Our MVP platform-defined policy is the right starting point. Phase 2 should introduce Click&Boat-style tiered options (Flexible / Moderate / Strict) with the platform default as "Moderate."

---

### 6. Payment reliability is a trust signal — and a real competitive gap

Zizoo went bankrupt in 2024 partly due to payment issues (customer payments not passed to charter companies). Boataround had reported payment reliability problems in 2025. AnchorsBook pays via PayPal/Bank Wire 12 hours after rental.

**Takeaway**: YachtBay's Stripe Connect Accounts v2 architecture — where owner payouts are automated, transparent, and released on a defined schedule — is a genuine trust differentiator. This should be communicated clearly to owners during onboarding. "Your money, automatically, on time."

---

### 7. Features that are table stakes (must have at MVP)

Based on what all major competitors offer, these are expected by renters and cannot be skipped:

- [x] Map-based search — all competitors have it
- [x] Date + location + capacity + charter type filters
- [x] Yacht detail page with photos, specs, pricing, availability calendar
- [x] Reviews and ratings display — **risk: we deferred this to Phase 2**
- [x] Instant book or clear booking flow
- [x] Booking confirmation email
- [x] Cancellation policy clearly displayed on listing

**Review risk**: All competitors prominently feature reviews. Launching with zero reviews is expected for a new platform, but the absence of a review system may reduce renter trust. Consider at minimum displaying a "New listing" badge and adding reviews as an early Phase 2 priority.

---

### 8. Features that are differentiators (Phase 2 opportunities)

| Feature | Who has it | Opportunity |
|---|---|---|
| AI pricing recommendations | GetMyBoat | Help owners price competitively |
| Promoted/featured listings | GetMyBoat | Revenue stream for Phase 2 |
| Add-ons (catering, insurance) | GetMyBoat | Upsell revenue |
| Owner-defined cancellation tiers | Click&Boat | Phase 2 owner flexibility |
| Flexible booking days (not Sat-Sat only) | Boataround | Already planned ✓ |
| Mobile app | GetMyBoat, Click&Boat | Phase 2 after traction ✓ |

---

### 9. Greece-specific observations

- No dominant Greece-specific P2P platform exists — the market is served by global/European generalists
- AnchorsBook is the most local competitor but small
- **This is the opportunity**: a Greece-first, locally-operated platform with lower commission, instant book, and reliable payments has a clear value proposition for both Greek owners and international renters

---

## YachtBay Positioning Summary

| Dimension | Market | YachtBay |
|---|---|---|
| **Commission** | 20–30% total | 13% (owner only) — lowest in market |
| **Booking** | Mostly inquiry-based | Instant book |
| **Payment reliability** | Mixed (Zizoo, Boataround issues) | Automated Stripe payouts |
| **Geographic focus** | Generic European/global | Greece-first, local expertise |
| **Charter flexibility** | Often Saturday-Saturday | Day trips + flexible multi-day |

---

## Related Pages

- [[getmyboat]]
- [[clickandboat]]
- [[boataround]]
- [[anchorsbook]]
- [[sailogy]]
- [[wiki/concepts/commission-model]]
- [[wiki/concepts/booking-model]]
- [[decisions/2026-05-02-platform-cancellation-policy]]
- [[specs/mvp-scope]]
