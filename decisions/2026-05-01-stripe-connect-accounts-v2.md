---
title: Use Stripe Connect Accounts v2 for payments
date: 2026-05-01
status: accepted
tags:
  - decision
  - payments
  - technical
---

# Use Stripe Connect Accounts v2 for Payments

## Context

YachtBay is a two-sided marketplace. Every booking involves a renter paying for a charter, YachtBay taking a commission, and the owner receiving the remainder. We need payment infrastructure that handles this split automatically without requiring us to build custom payout logic, KYC flows, or compliance systems.

Stripe Connect is the industry-standard solution. We initially considered Express, but after deeper research revised the decision to Accounts v2. See [[decisions/2026-05-01-stripe-connect-express]] (superseded).

## Options Considered

### Standard
Owners connect an existing Stripe account. Stripe bills them directly. Platform has no control over payout timing.

**Why rejected**: Cannot hold funds until the charter starts. Not suitable for a marketplace where renter protection requires escrow-like payout control.

### Express (legacy)
Stripe-managed onboarding, simplified owner dashboard, platform controls payouts.

**Why superseded**: Express is deprecated for new integrations as of late 2025. Starting with a deprecated API creates avoidable migration debt. Accounts v2 achieves identical behaviour with the same configuration.

### Custom (legacy)
Platform builds the entire owner-facing experience from scratch.

**Why rejected**: Significant engineering investment — onboarding UI, owner dashboard, support flows. Overkill for MVP.

### Accounts v2 ✓
Stripe's current unified API. Uses configurable `responsibilities` and `dashboard` properties instead of named account types. Can be configured to replicate Express behaviour exactly.

**Why chosen**: Recommended by Stripe for all new integrations. Achieves the same outcome as Express — Stripe-managed onboarding, owner Express Dashboard, platform controls payouts — without building on a deprecated API.

## Decision

**Stripe Connect Accounts v2**, configured to replicate Express behaviour.

### Required configuration at account creation

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
> `fees_collector` and `losses_collector` cannot be changed after account creation. Must be set correctly from the start.

### Payment implementation

- **Charge type**: Destination charges — platform charges the renter, applies `application_fee_amount`, transfers net to owner's connected account
- **Payout timing**: Release funds **24–48 hours after the charter start date**
- **Owner onboarding**: Stripe-hosted Express onboarding flow (triggered during listing setup)
- **Commission deduction**: Via `application_fee_amount` on each `PaymentIntent` or `Checkout Session`
- **Payout settings**: Managed via Accounts v1 API (v2 does not yet cover payout settings)

### Example API call (Checkout Session)

```bash
curl https://api.stripe.com/v1/checkout/sessions \
  -u "YOUR_SECRET_KEY:" \
  -d "payment_intent_data[application_fee_amount]=13000" \
  -d "payment_intent_data[transfer_data][destination]=OWNER_STRIPE_ACCOUNT_ID" \
  -d mode=payment
```

## Consequences

- Stripe handles all owner KYC and identity verification
- Platform assumes fraud liability — mitigated by Stripe Radar
- Platform is billed €2/month per active connected account + 0.25% + €0.10 per payout
- Owners see their payouts and earnings in the Stripe Express Dashboard
- No migration debt — building on Stripe's current recommended API
- Payout settings require v1 API calls — minor caveat, well-documented
- The data model must store each owner's Stripe `account_id` to route payouts correctly

## Related

- [[decisions/2026-05-01-stripe-connect-express]] (superseded by this decision)
- [[wiki/concepts/stripe-connect]]
- [[wiki/concepts/commission-model]]
- [[wiki/concepts/business-model]]
