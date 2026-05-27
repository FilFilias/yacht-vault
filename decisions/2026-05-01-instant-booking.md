---
title: Instant booking (no owner approval step)
date: 2026-05-01
status: accepted
tags:
  - decision
  - product
---

# Instant Booking

## Context

Yacht rental platforms typically use one of two booking flows: instant book (confirmed immediately) or request-to-approve (owner must accept before the booking is confirmed).

## Options Considered

- **Instant book**: Renter pays, booking is immediately confirmed
- **Request-to-approve**: Renter requests, owner accepts/declines, then renter pays
- **Hybrid**: Owner can choose per listing (instant or request)

## Decision

**Instant book** as the default and primary mode.

## Consequences

- Significantly reduces friction for renters — no waiting period
- Higher conversion rates (renters don't drop off during a wait)
- Aligns with renter expectations from other instant-book platforms (Airbnb, etc.)
- Owners must keep their availability calendars accurate — double booking risk if not maintained
- May need a calendar sync feature (iCal/Google Calendar) to help owners stay up to date
- A request-to-approve mode could be offered as a future option for owners who prefer it

## Related

- [[wiki/concepts/booking-model]]
- [[wiki/concepts/business-model]]
