---
title: Use Stripe Connect Express for payments
date: 2026-05-01
status: superseded
superseded-by: "[[decisions/2026-05-01-stripe-connect-accounts-v2]]"
tags:
  - decision
  - payments
  - technical
---

# Use Stripe Connect Express for Payments

> [!warning] Superseded
> This decision was revised on 2026-05-01. Express is deprecated for new Stripe integrations. See [[decisions/2026-05-01-stripe-connect-accounts-v2]] for the current decision.

## Context

YachtBay is a two-sided marketplace. Every booking involves a renter paying for a charter, YachtBay taking a commission, and the owner receiving the remainder. We need a payment infrastructure that handles this split automatically without requiring us to build custom payout logic, KYC flows, or compliance systems.

Stripe Connect is the industry-standard solution for marketplace payments. The question is which account type to use.

## Options Considered

### Standard
Owners connect an existing Stripe account. Stripe bills them directly. The platform has minimal control over the payment flow and cannot hold funds before releasing to the owner.

**Why rejected**: No control over payout timing. We need to be able to hold funds until the charter starts to protect renters from no-shows. Standard doesn't support this.

### Express
Stripe handles owner onboarding and KYC. Owners get a simplified Express Dashboard. Platform controls payout timing and commission via `application_fee`. Stripe bills the platform per active account and per payout.

**Why chosen**: Best balance of control and build effort for an MVP marketplace.

### Custom
Platform builds the entire owner-facing experience from scratch. Maximum control, no Stripe branding.

**Why rejected**: Requires significant engineering investment — onboarding UI, owner dashboard, support flows, compliance updates. Overkill for MVP. Can migrate to Custom later if needed.

### Accounts v2
Stripe's new unified API using controller properties instead of named account types.

**Why deferred**: Newer API with less documentation and community examples. Introduces risk for an MVP build. Worth revisiting for v2 when the integration is stable.

## Decision

**Stripe Connect Express** for the MVP launch.

Implementation details:
- Charge type: **Destination charges** (platform collects from renter, applies `application_fee`, transfers net to owner)
- Payout timing: Release funds **24–48 hours after the charter start date**
- Owner onboarding: Stripe-hosted Express onboarding flow during listing setup
- Commission deduction: Via Stripe `application_fee_amount` on each PaymentIntent

## Consequences

- Stripe handles all owner KYC and identity verification — we don't build it
- Platform assumes fraud liability — mitigated by Stripe Radar (built-in fraud detection)
- Platform is billed €2/month per active connected account + 0.25% + €0.10 per payout
- Owners see their payouts and earnings in the Stripe Express Dashboard
- Migrating to Custom or Accounts v2 later is possible but requires integration work
- The data model must store each owner's Stripe `account_id` to route payouts correctly

## Related

- [[wiki/concepts/stripe-connect]]
- [[wiki/concepts/commission-model]]
- [[wiki/concepts/business-model]]
