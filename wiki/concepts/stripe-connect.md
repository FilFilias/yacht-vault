---
title: Stripe Connect
tags:
  - concept
  - payments
  - technical
last-updated: 2026-05-01
---

# Stripe Connect

**Summary**: How Stripe Connect works as the payment infrastructure for YachtBay — account types, fees, and how the commission split is handled.

---

## What is Stripe Connect

Stripe Connect is Stripe's marketplace and platform product. It allows YachtBay to:

- Accept payments from renters
- Automatically split each payment — platform commission goes to YachtBay, the remainder goes to the owner
- Onboard owners as connected accounts with Stripe handling KYC/identity verification
- Control payout timing (e.g. hold funds until the charter starts)

This eliminates the need to build custom payment splitting, payout logic, or compliance flows.

---

## How the Payment Flow Works

For each booking on YachtBay:

1. **Renter pays** the full charter amount at booking (e.g. €1,000)
2. **Stripe deducts** its processing fee (e.g. 1.5% + €0.25 for a European card = ~€15.25)
3. **YachtBay deducts** its commission (e.g. 13% = €130) as an `application_fee`
4. **Owner receives** the remainder (€1,000 − €15.25 − €130 = ~€854.75) on the defined payout schedule

YachtBay controls when the owner receives their payout — standard practice is to release funds 24–48 hours after the charter start date.

---

## Account Types

> [!warning] 2026 Note
> Express, Standard, and Custom are **deprecated for new integrations**. Stripe recommends **Accounts v2** for all new builds. YachtBay uses Accounts v2 configured to replicate Express behaviour. See [[decisions/2026-05-01-stripe-connect-accounts-v2]].

### Standard

Owners connect an existing Stripe account (or create one) and have full access to the Stripe Dashboard.

| | |
|---|---|
| **Integration effort** | Lowest — OAuth flow |
| **Onboarding** | Stripe-managed |
| **Owner dashboard** | Full Stripe Dashboard |
| **KYC / identity** | Stripe handles it |
| **Liability (fraud)** | Shared between Stripe and platform |
| **Platform controls payout timing** | No |
| **Stripe bills connected account** | Yes (directly) |

**Pros:**
- Almost no integration work
- Owners familiar with Stripe already have an account
- Stripe handles all compliance and billing

**Cons:**
- Least control — platform cannot control payout timing or customise the owner experience
- Owners can disconnect their account at any time
- Not ideal for marketplaces where the platform needs to hold funds before releasing

---

### Express

Owners are onboarded through a Stripe-hosted flow. They get access to a simplified Express Dashboard (payouts, earnings). YachtBay controls the payment flow.

| | |
|---|---|
| **Integration effort** | Low — API only |
| **Onboarding** | Stripe-managed (hosted UI) |
| **Owner dashboard** | Express Dashboard (simplified — payouts, balance) |
| **KYC / identity** | Stripe handles it |
| **Liability (fraud)** | Platform |
| **Platform controls payout timing** | Yes |
| **Stripe bills connected account** | No — platform is billed |

**Pros:**
- Stripe handles all owner KYC/identity verification
- Platform controls payout timing — can hold funds until charter starts
- Clean onboarding experience without building it from scratch
- Owners have enough dashboard access to track their earnings without overwhelming them
- Good balance of control and build effort

**Cons:**
- Platform assumes fraud liability (mitigated by Stripe Radar)
- Platform is billed per active account and per payout (see fees below)
- Owner dashboard is limited — owners who want more detail may find it insufficient

---

### Custom

Platform builds the entire owner-facing experience. Owners have no Stripe dashboard access.

| | |
|---|---|
| **Integration effort** | Highest — full build required |
| **Onboarding** | Platform-built (or Stripe Connect Onboarding used as a base) |
| **Owner dashboard** | None — platform builds it |
| **KYC / identity** | Platform collects, Stripe verifies |
| **Liability (fraud)** | Platform |
| **Platform controls payout timing** | Yes |
| **Stripe bills connected account** | No — platform is billed |

**Pros:**
- Maximum control — full white-label experience
- Platform builds exactly the dashboard and flow owners see
- No Stripe branding in the owner experience

**Cons:**
- Significant engineering investment — must build onboarding UI, owner dashboard, support flows
- Platform is responsible for all compliance updates
- Adding new countries requires integration changes
- Overkill for an MVP

---

### Accounts v2 ✓ (Our Choice)

Stripe's current unified API for new integrations. Instead of named account types, you configure `responsibilities` and `dashboard` properties per account. Can replicate Express behaviour exactly while building on a non-deprecated API.

| | |
|---|---|
| **Integration effort** | Low-medium — API only, Stripe docs are solid |
| **Onboarding** | Stripe-managed (Express Dashboard hosted by Stripe) |
| **Owner dashboard** | Express Dashboard — set `dashboard: "express"` |
| **KYC / identity** | Stripe handles it |
| **Liability (fraud)** | Platform — set `losses_collector: "application"` |
| **Platform controls payout timing** | Yes |
| **Stripe bills connected account** | No — platform is billed |
| **Deprecated** | No — recommended for new integrations |

**Configuration to replicate Express behaviour:**
```json
{
  "defaults": {
    "responsibilities": {
      "fees_collector": "application",
      "losses_collector": "application"
    }
  },
  "dashboard": "express"
}
```

> [!warning]
> `fees_collector` and `losses_collector` cannot be changed after account creation.

> [!note]
> Payout settings still require the v1 Accounts API — v2 does not yet cover this. Minor caveat, well-documented by Stripe.

**Pros:**
- Recommended by Stripe for all new integrations — no migration debt
- Identical outcome to Express (same dashboard, same liability model, same payout control)
- One Account object handles all interactions — cleaner data model
- Future-proof as Stripe invests in this API

**Cons:**
- Fewer community examples than legacy Express (API launched Dec 2025)
- Payout settings require v1 API calls — minor extra complexity

---

## Comparison Summary

| | Standard | Express (legacy) | Custom (legacy) | Accounts v2 ✓ |
|---|---|---|---|---|
| **Build effort** | Minimal | Low | High | Low-medium |
| **Owner dashboard** | Full | Simplified | None | Configurable (Express-style) |
| **Platform controls payouts** | No | Yes | Yes | Yes |
| **Stripe handles KYC** | Yes | Yes | Partial | Yes |
| **Platform fraud liability** | Shared | Yes | Yes | Yes |
| **Deprecated for new builds** | Yes | Yes | Yes | No |
| **Best for** | — | Existing integrations only | Existing integrations only | All new integrations |

---

## Fees (Greece — EUR)

### Card Processing

These fees are charged per transaction by Stripe, regardless of account type.

| Card Type | Fee |
|---|---|
| European EEA cards (standard) | 1.5% + €0.25 |
| European EEA cards (premium) | 1.9% + €0.25 |
| UK cards | 2.5% + €0.25 |
| Other international cards | 3.25% + €0.25 |
| Currency conversion (if applicable) | +2% |

> [!tip]
> Most renters in Greece will pay with European EEA cards (1.5% + €0.25). International tourists (US, non-EU) will use the 3.25% + €0.25 rate. Factor this into pricing analysis.

### Connect Platform Fees (Express / Custom — "You Handle Pricing" model)

| Fee | Amount |
|---|---|
| Active connected account | €2 / month (only months when payouts are sent) |
| Payout to connected account | 0.25% + €0.10 per payout |
| Instant payouts | 1% of payout volume |
| Cross-border payouts | From 0.25% of payout volume |

> [!note]
> For Standard accounts, Stripe bills the connected account directly and the platform pays nothing. For Express/Custom, the platform is billed.

### Example: €1,000 Booking (European EEA card, Accounts v2)

| Item | Amount |
|---|---|
| Renter pays | €1,000.00 |
| Stripe processing fee (1.5% + €0.25) | −€15.25 |
| YachtBay commission (13%) | −€130.00 |
| Stripe payout fee (0.25% + €0.10) | −€2.60 |
| **Owner receives** | **~€852.15** |
| **YachtBay earns** | **€130.00** |

> The owner never sees the Stripe fees — they are deducted before the owner's share is calculated in practice. The exact accounting treatment should be defined in the terms of service.

---

## Charge Types

When using Express or Custom, platforms choose how charges are structured:

| Type | How it works | Best for |
|---|---|---|
| **Destination charge** | Platform charges the customer, transfers net to owner | Single-vendor transactions (most YachtBay bookings) |
| **Direct charge** | Owner charges the customer directly | Not suitable — platform loses control |
| **Separate charge + transfer** | Platform charges customer, then manually transfers to owner | Multi-vendor or complex splits |

**YachtBay should use Destination Charges** — platform collects from the renter, takes the `application_fee`, transfers the rest to the owner's connected account.

---

## Our Choice

YachtBay uses **Stripe Connect Accounts v2**, configured with Express-style dashboard and platform-side liability. See [[decisions/2026-05-01-stripe-connect-accounts-v2]] for the full rationale.

## Related Pages

- [[decisions/2026-05-01-stripe-connect-express]]
- [[commission-model]]
- [[business-model]]
