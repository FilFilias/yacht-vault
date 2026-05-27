---
title: Web app only at MVP, mobile app post-traction
date: 2026-05-02
status: accepted
tags:
  - decision
  - product
  - platform
---

# Web App Only at MVP

## Context

YachtBay needs to decide whether to build a native mobile app (iOS/Android) alongside the web app at launch, or focus on web first.

## Options Considered

- **Web app only (responsive)**: Ship a fully responsive web app that works well on mobile browsers. No native apps.
- **Web + native mobile**: Build web app and iOS/Android apps simultaneously.
- **Mobile first**: Build native apps first, web later.

## Decision

**Web app only at MVP**, fully responsive (mobile-friendly). Native mobile apps will be built once the platform has traction.

## Consequences

- Significantly faster to build — one codebase, one deployment
- Responsive design ensures mobile users are not excluded
- Native app UX advantages (push notifications, offline, camera) are deferred — acceptable for MVP
- Once traction is validated, native apps become a Phase 2 priority
- Tech stack choice should support a future mobile app (e.g. API-first architecture)

## Related

- [[specs/mvp-scope]]
- [[roadmap/overview]]
