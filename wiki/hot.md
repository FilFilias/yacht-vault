---
title: Hot — Session Context Cache
tags:
  - meta
last-updated: 2026-05-05
---

# Hot — Session Context Cache

> [!info]
> This file is read at the start of every session. It captures recent decisions, current focus, and open questions. Always update it after significant changes.

---

## Current Focus

All pre-development documentation complete. Ready to start scaffolding (Milestone 1).

## Recent Decisions (2026-05-01 — updated)

- Platform name: **YachtBay** (temporary — to be revisited)
- Launch market: **Greece only**
- Booking model: **Instant book**
- Revenue: **~13% commission from owners** (exact rate under analysis)
- Crew options: **Bareboat, skippered, fully crewed** — defined per yacht by owner
- Charter types: **Day trips + multi-day/weekly**
- Target customer: **Mid-range travelers** (primary)
- Payments: **Stripe Connect Accounts v2** (Express-style config) — destination charges, platform controls payout timing (release 24–48h after charter start). Express deprecated for new integrations.

## Vault Status

- [x] CLAUDE.md written
- [x] Folder structure created
- [x] Core concept pages written (vision, business model, commission, market, booking, crew, personas)
- [x] 4 decision records written
- [x] Roadmap stub created
- [x] wiki/index.md initialized
- [x] Stripe Connect research documented
- [x] MVP scope defined — [[specs/mvp-scope]]
- [x] Competitor research complete — [[wiki/concepts/competitor-analysis]]
- [x] Backend tech stack decided — [[wiki/concepts/tech-stack]]
- [x] Technical architecture defined — [[specs/backend-architecture]]
- [ ] Product design — not started

## Open Questions

- What is the exact default commission rate? (~13% but needs final confirmation)
- What is the final platform name?

## Next Steps

1. ~~Product design~~ → **In progress** — Google Stitch design brief created at [[specs/design/stitch-brief]]
2. Feed stitch-brief.md into Google Stitch to generate storefront UI
3. Connect Google Stitch MCP to Claude Code and transform UI into React Router 7 SSR code
