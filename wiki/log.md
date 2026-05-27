---
title: Wiki Log
tags:
  - meta
last-updated: 2026-05-01
---

# Wiki Log

Append-only record of all vault operations. Never delete or edit past entries.

---

## 2026-05-01 — Vault initialized

**Action**: Created vault structure and initial documentation from founding conversation.

**Added:**
- `CLAUDE.md` — vault instructions for Claude (rewritten for YachtBay)
- `wiki/index.md` — master table of contents
- `wiki/hot.md` — session context cache
- `wiki/log.md` — this file
- `wiki/concepts/vision-and-mission.md`
- `wiki/concepts/business-model.md`
- `wiki/concepts/commission-model.md`
- `wiki/concepts/target-market.md`
- `wiki/concepts/booking-model.md`
- `wiki/concepts/crew-options.md`
- `wiki/concepts/user-personas.md`
- `decisions/2026-05-01-commission-from-owners.md`
- `decisions/2026-05-01-greece-first-launch.md`
- `decisions/2026-05-01-instant-booking.md`
- `decisions/2026-05-01-platform-name-yachtbay.md`
- `roadmap/overview.md`

**Source**: Founding conversation with the platform founder.

---

## 2026-05-01 — Stripe Connect research and payment decision

**Action**: Researched Stripe Connect options, documented findings, recorded payment decision.

**Added:**
- `wiki/concepts/stripe-connect.md` — full reference: account types, fees, charge types, payment flow
- `decisions/2026-05-01-stripe-connect-express.md` — decision to use Express for MVP

**Updated:**
- `wiki/index.md` — added new entries
- `wiki/hot.md` — added payment decision to recent decisions

**Source**: Stripe official documentation (stripe.com/connect/pricing, docs.stripe.com/connect/accounts).

---

## 2026-05-05 — Google Stitch design brief created

**Action**: Created complete storefront design brief for Google Stitch — all 18 pages, all renter user stories (R-001–R-022), shared components, design direction, business rules, and route summary.

**Added:** `specs/design/stitch-brief.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

**Purpose**: This file is fed into Google Stitch to generate the storefront UI. After design, Google Stitch MCP connects to Claude Code to transform the UI into React Router 7 SSR code.

---

## 2026-05-05 — Environment variables documented

**Action**: Documented all environment variables for all 4 apps with Railway config notes and pre-launch secrets checklist.

**Added:** `specs/env-variables.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-05 — Development roadmap written

**Action**: Created 7-milestone Phase 1 development roadmap with build order, dependencies, and testable checkpoints per milestone.

**Added:** `roadmap/phase-1-development.md`
**Updated:** `roadmap/overview.md`, `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-05 — API contract written

**Action**: Wrote complete API contract — 54 endpoints across 8 modules with full request/response shapes.

**Added:** `specs/api-contract.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-05 — Data model finalized

**Action**: Audited draft data model against all 68 user stories. Found 18 gaps. Produced final Prisma schema.

**Added:** `specs/data-model.md`

**Key gaps fixed:** roles array, yacht_photos table, slug, prep_days, charter_types, email verification fields, password reset fields, phone, bio, profile_photo_url, commission_rate, user status/suspension, guest_count on reservations, triggered_by/admin_id on events, stripe_refund_id on payments, error_message on outbox.

**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-05 — All site maps complete

**Action**: Created admin panel site map. All three apps now mapped.

**Added:** `specs/design/sitemap-admin.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

**Total site map**: Storefront 18 + Owner Panel 25 + Admin 10 = **53 pages**

---

## 2026-05-05 — Owner panel site map created

**Action**: Created owner panel site map — 25 pages including 6-step listing creation wizard.

**Added:** `specs/design/sitemap-owner-panel.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-04 — Storefront site map created

**Action**: Created storefront site map — 18 pages with routes, auth requirements, SSR strategy, and navigation structure.

**Added:** `specs/design/sitemap-storefront.md`
**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-04 — All user stories complete

**Action**: Wrote Admin user stories. All three personas now complete.

**Added:**
- `specs/user-stories/admin.md` — 16 stories across 6 epics

**Story totals**: Renter 26 · Owner 26 · Admin 16 = **68 total stories**

**Updated:** `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-04 — Owner user stories written

**Action**: Wrote all owner user stories (MVP + Phase 2) with acceptance criteria and technical requirements.

**Added:**
- `specs/user-stories/owner.md` — 26 stories across 7 epics

**Updated:**
- `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-04 — Renter user stories written

**Action**: Wrote all renter user stories (MVP + Phase 2) with acceptance criteria and technical requirements.

**Added:**
- `specs/user-stories/renter.md` — 26 stories across 8 epics

**Updated:**
- `wiki/index.md`, `wiki/hot.md`

---

## 2026-05-03 — Frontend architecture decided

**Action**: Decided on separate storefront + owner panel apps in a Turborepo monorepo.

**Added:**
- `decisions/2026-05-03-frontend-monorepo.md`

**Updated:**
- `wiki/concepts/tech-stack.md` — frontend section updated with monorepo structure
- `wiki/index.md` — decision added
- `wiki/hot.md` — current focus updated

---

## 2026-05-03 — Competitor research completed

**Action**: Researched 5 competitors, documented entity pages, wrote synthesis analysis, updated cancellation policy based on market findings.

**Added:**
- `wiki/entities/getmyboat.md`
- `wiki/entities/clickandboat.md`
- `wiki/entities/boataround.md`
- `wiki/entities/anchorsbook.md`
- `wiki/entities/sailogy.md`
- `wiki/concepts/competitor-analysis.md`

**Updated:**
- `decisions/2026-05-02-platform-cancellation-policy.md` — revised thresholds based on market research, added day trip vs. weekly split, added platform fees non-refundable rule
- `wiki/index.md` — entities section populated, competitor-analysis added
- `wiki/hot.md` — status and focus updated

**Key findings:** YachtBay's 13% owner-only commission is the lowest in the market. GetMyBoat (market leader) lacks instant book — a real differentiator. Platform fees should be non-refundable on cancellation (industry standard). No dominant Greece-specific P2P platform exists.

---

## 2026-05-03 — Tech stack finalized

**Action**: Confirmed all remaining stack decisions and documented the complete tech stack.

**Added:**
- `wiki/concepts/tech-stack.md` — master tech stack reference with rationale, Mapbox alternatives, and rejected options

**Updated:**
- `specs/backend-architecture.md` — Cloudinary → Cloudflare R2, adapter references updated
- `wiki/index.md` — added tech-stack concept page
- `wiki/hot.md` — current focus and open questions updated

**Key decisions confirmed:** Prisma (ORM), Custom JWT, Cloudflare R2 (images), Resend (email), Railway (hosting), GitHub Actions (CI/CD), Mapbox (maps), PostGIS (geo search), SMS deferred.

---

## 2026-05-03 — Backend architecture finalized

**Action**: Synthesized architecture from two Claude sessions into a definitive backend spec.

**Added:**
- `specs/backend-architecture.md` — full architecture: stack, patterns, data model, booking workflow, payment flow, event architecture, extensibility points, scalability path
- `decisions/2026-05-03-backend-architecture.md` — decision record
- `wiki/sources/yacht-booking-system-summary.md` — ingested raw docx from architecture session

**Updated:**
- `wiki/index.md` — added new spec, decision, and source
- `wiki/hot.md` — updated current focus and vault status

**Source**: This session + `raw/yacht-booking-system-summary.docx`

---

## 2026-05-02 — MVP scope defined

**Action**: Defined full MVP feature set after answering all scoping questions.

**Added:**
- `specs/mvp-scope.md` — full MVP feature list (owner, renter, platform, deferred items)
- `decisions/2026-05-02-web-app-mvp.md` — web app only at MVP
- `decisions/2026-05-02-platform-cancellation-policy.md` — platform-defined cancellation policy

**Updated:**
- `roadmap/overview.md` — linked to MVP spec
- `wiki/index.md` — added spec and new decisions
- `wiki/hot.md` — updated current focus, status, open questions, next steps

**Source**: Founding conversation — MVP scoping session.

---

## 2026-05-01 — Revised payment decision: Express → Accounts v2

**Action**: After deeper research confirmed Express is deprecated for new integrations, revised the payment decision to Accounts v2.

**Added:**
- `decisions/2026-05-01-stripe-connect-accounts-v2.md` — new accepted decision

**Updated:**
- `decisions/2026-05-01-stripe-connect-express.md` — status changed to `superseded`, warning callout added
- `wiki/concepts/stripe-connect.md` — Accounts v2 section updated with full configuration details, comparison table updated, "Our Choice" updated
- `wiki/index.md` — old decision marked superseded, new decision added
- `wiki/hot.md` — payment decision updated
