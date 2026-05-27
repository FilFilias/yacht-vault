---
title: Google Stitch Design Brief — Storefront (yachtbay.com)
tags:
  - spec
  - design
  - stitch
status: review
last-updated: 2026-05-07
---

# YachtBay Storefront — Design Brief for Google Stitch

This document is the complete design brief for the YachtBay storefront (`yachtbay.com`) — the consumer-facing site where renters discover, compare, and book yacht charters in Greece. Feed this document to Google Stitch to generate the UI.

---

## 1. About the Product

**YachtBay** is a peer-to-peer yacht rental marketplace launching in Greece. Yacht owners list their vessels; renters browse, compare, and instantly book. There is no broker, no waiting for owner approval — the booking is confirmed the moment payment goes through.

**Core value proposition for renters:**
- Find and compare yachts in Greece in one place
- Transparent pricing — no hidden fees, price calculated upfront
- Instant book — confirmed immediately, no waiting
- Secure payment via Stripe
- Clear cancellation policy

**Charter types:** Day trips and multi-day / weekly charters.

**Crew options (defined per yacht by the owner):**
- **Bareboat** — renter operates the yacht themselves
- **Skippered** — professional skipper included
- **Fully crewed** — skipper + additional crew (cook, deckhand)

**Commission model:** ~13% commission charged to owners only. Renters pay the listed price with no platform surcharge.

**Geography:** Greece only at launch. Key destinations: Mykonos, Santorini, Corfu, Rhodes, Athens/Piraeus.

---

## 2. The Renter (Primary User)

**Who they are:**
A mid-range traveler — international tourist or local Greek — booking a yacht for a group. Traveling as a couple, with friends, or with family. Wants a quality experience but doesn't want to go through a broker or spend hours researching scattered listings.

**What they need from the UI:**
- Clean, trustworthy, visually beautiful — this is a premium-but-accessible product
- Fast search and comparison — photos, specs, price, location at a glance
- Confidence before booking — clear cancellation policy, transparent pricing, no surprises
- Frictionless booking — as few steps as possible between "I want this yacht" and "booking confirmed"

**Key frustrations with existing platforms:**
- Hard to find yachts online — scattered across broker sites
- Pricing is opaque — hard to compare like-for-like
- No instant booking — must wait for owner to respond
- Poor mobile experience

---

## 3. Design Direction

**Feel:** Dark, premium, Mediterranean nocturne. Rich and immersive — deep near-black backgrounds with warm gold accents and crisp editorial typography. The aesthetic evokes a luxury night on the Aegean: dark water, golden light, quiet exclusivity. Not gothic or heavy — warm, refined, and aspirational.

**Visual priorities:**
- Large, beautiful photography — yachts and Greek landscapes dominate; images glow against dark surfaces
- Generous spacing — dark mode requires breathing room to avoid feeling oppressive
- Trust signals — clear pricing, visible cancellation policy, Stripe payment badge
- The booking widget is the most important element on the yacht detail page — it must always be visible and prominent
- Gold accent (`#f2ca50`) is reserved for CTAs, prices, highlights, and interactive states — never overused

**Mode:** Dark. All screens use the dark color system below. Do not mix light and dark surfaces unless explicitly specified.

**Mobile-first:** Most renters will discover and browse on mobile. The storefront must be fully responsive. On mobile, the booking widget on the yacht detail page becomes a sticky bottom bar ("From €X/day — Book Now").

---

## 3a. Design System — Canonical Tokens

This is the single source of truth for the design system. Every screen must use these tokens exactly. **Do not deviate from these values.**

### Color Palette

| Token | Value | Usage |
|---|---|---|
| `background` | `#121414` | Page background — all screens |
| `surface` | `#121414` | Card and panel base |
| `surface-container-lowest` | `#0c0f0f` | Deepest recessed surfaces |
| `surface-container-low` | `#1a1c1c` | Subtle elevation |
| `surface-container` | `#1e2020` | Default card/panel fill |
| `surface-container-high` | `#282a2b` | Raised cards |
| `surface-container-highest` | `#333535` | Highest elevation, dropdowns |
| `surface-variant` | `#333535` | Bordered panel fills |
| `surface-bright` | `#37393a` | Hover fills, active row |
| `surface-dim` | `#121414` | Same as background |
| `on-surface` | `#e2e2e2` | Primary body text |
| `on-background` | `#e2e2e2` | Primary body text on page bg |
| `on-surface-variant` | `#d0c5af` | Secondary / muted text (warm grey) |
| `primary` | `#f2ca50` | Gold accent — CTAs, prices, highlights |
| `primary-container` | `#d4af37` | Darker gold — filled button bg |
| `on-primary` | `#3c2f00` | Text on gold buttons |
| `primary-fixed` | `#ffe088` | Lightest gold tint |
| `primary-fixed-dim` | `#e9c349` | Mid gold |
| `secondary` | `#c8c6c5` | Secondary UI text |
| `secondary-container` | `#4a4949` | Chip/badge fills |
| `on-secondary` | `#313030` | Text on secondary fills |
| `tertiary` | `#c0ceec` | Soft blue — accents, links |
| `tertiary-container` | `#a5b3d0` | Blue fill surfaces |
| `on-tertiary` | `#233148` | Text on blue fills |
| `outline` | `#99907c` | Borders, dividers |
| `outline-variant` | `#4d4635` | Subtle borders |
| `inverse-surface` | `#e2e2e2` | Light surface for reversed components |
| `inverse-on-surface` | `#2f3131` | Text on light reversed surface |
| `error` | `#ffb4ab` | Error text / destructive |
| `on-error` | `#690005` | Text on error fills |
| `error-container` | `#93000a` | Error background |
| `on-error-container` | `#ffdad6` | Text on error container |

**Status colors (not in token palette — hardcoded):**
- Confirmed / success: `#22c55e` (green-500)
- Cancelled / danger: `#ef4444` (red-500)
- Pending / warning: `#f59e0b` (amber-500)
- Completed / neutral: `#6b7280` (grey-500)

### Typography

All text uses **Noto Serif**. No other font family is used anywhere on the site.

| Scale | Size | Weight | Line Height | Letter Spacing | Usage |
|---|---|---|---|---|---|
| `display-xl` | 72px | 400 | 1.1 | −0.02em | Hero headline |
| `headline-lg` | 48px | 400 | 1.2 | — | Section hero titles |
| `headline-md` | 32px | 500 | 1.3 | — | Page headings, card titles |
| `body-lg` | 20px | 400 | 1.6 | — | Lead body text |
| `body-md` | 16px | 400 | 1.6 | — | Standard body text |
| `label-sm` | 12px | 600 | 1.4 | 0.1em | Badges, labels, nav items, uppercase UI |

### Border Radius

| Token | Value | Usage |
|---|---|---|
| `DEFAULT` | 0.25rem (4px) | Cards, inputs, most elements |
| `lg` | 0.5rem (8px) | Larger cards, modals |
| `xl` | 0.75rem (12px) | Sheets, overlays |
| `full` | 9999px | Pills, avatar circles |

### Spacing

| Token | Value | Usage |
|---|---|---|
| `base` | 8px | Grid unit |
| `gutter` | 24px | Column gutter |
| `section-gap` | 120px | Vertical space between page sections |
| `margin-desktop` | 64px | Horizontal page margin (desktop) |
| `margin-mobile` | 20px | Horizontal page margin (mobile) |

---

## 3b. Global Navigation — Consistency Rules

**These elements must be pixel-identical across every screen, unless explicitly instructed otherwise for a specific page.**

### Top Navigation Bar

The top nav from the Homepage is the canonical header for all screens. It must appear unchanged at the top of every page.

- Background: `surface-container` (`#1e2020`), with a subtle bottom border in `outline-variant`
- Left: YachtBay logo (wordmark)
- Center (desktop): compact search bar
- Right (desktop): "How it works" · "List your yacht" · "Log in" · "Register"
- Right (logged in): avatar + name dropdown → My Bookings / Profile / Switch to Owner / Log out
- Mobile: logo left, hamburger right; search bar below nav, full width
- The nav is **sticky** — it stays fixed to the top of the viewport on all pages

### Footer

The footer from the Homepage is the canonical footer for all screens.

- Background: `surface-container-lowest` (`#0c0f0f`)
- Four columns: YachtBay / For Renters / For Owners / Legal
- Text: `on-surface-variant` (`#d0c5af`) for links, `on-surface` (`#e2e2e2`) for column headings
- Bottom bar within footer: copyright line + YachtBay wordmark, centered

### Mobile Bottom Booking Bar

On the Yacht Detail page only, a sticky bottom bar replaces the right-column booking widget on mobile:
- Background: `surface-container-high` (`#282a2b`)
- "From €X/day" in `primary` (`#f2ca50`) on the left
- "Book Now" full-width gold CTA button on the right
- Top border: `outline-variant`
- This bar is defined on the Yacht Detail page spec — it does not appear on other pages

---

## 4. Technology Context

- **Framework:** React Router 7 (SSR mode)
- **Maps:** Mapbox (`react-map-gl`)
- **Payments:** Stripe Elements (card input on payment page)
- **Auth:** JWT via httpOnly cookies
- **Image storage:** Cloudflare R2 + CDN

This context is relevant for component behavior: date pickers, maps, and the Stripe Elements card input are third-party embeds.

---

## 5. Information Architecture

### Site Structure

```
yachtbay.com/
│
├── /                          Homepage
├── /search                    Search results
│
├── /yachts/:id                Yacht detail page
│   └── /yachts/:id/checkout   Checkout + Payment (auth required)
│
├── /bookings                  My bookings (auth required)
│   └── /bookings/:id          Booking detail (auth required)
│       └── /confirmed         Booking confirmation (auth required)
│
├── /profile                   Profile & account (auth required)
│
├── /register                  Register
├── /login                     Login
├── /forgot-password           Forgot password
├── /reset-password            Reset password
└── /verify-email              Email verification
│
├── /how-it-works              How YachtBay works
├── /cancellation-policy       Platform cancellation policy
├── /terms                     Terms of service
└── /privacy                   Privacy policy
```

**Total: 17 pages**

---

## 6. Global Navigation

> **Canonical reference:** The Homepage screen. Every page must use this header and footer exactly.

### Top Navigation (mobile — primary target)

Three-column bar, `border-b border-outline-variant`, background `surface-container`:
- **Left (1/4):** mail icon (`text-primary`) — links to contact
- **Center (1/2):** YachtBay logo image, `h-10`, centered
- **Right (1/4):** language picker — current flag + `expand_more` icon; dropdown shows 🇬🇷 🇫🇷 🇮🇹

The nav is **sticky** — fixed to the top of the viewport on scroll.

> Note: The desktop nav (search bar, How it works, Login etc.) is defined for future desktop builds. Mobile nav as above is the canonical form for Stitch mobile screens.

### Bottom Navigation Bar (mobile — persistent on most pages)

Fixed to the bottom of the screen. Background `surface-container-low`, `border-t border-outline-variant`. Two tabs visible in current design:
- **Explore** (active state: `text-primary`) — icon: `explore`
- **Anchor** tab — icon: `anchor`

Additional tabs for logged-in state: Bookings / Saved / Profile (see Checkout for that variant).

> **Checkout** uses a different bottom nav (Explore / Bookings / Saved / Profile) and a simplified header (see PAGE 4).

### Footer

Identical across all pages. Two sections:

**Top block** — flex row (stacks on mobile):
- Left: YachtBay wordmark in `text-primary`, contact details below in `label-sm`: email, registration number, phone, address
- Right: Newsletter widget card (`bg-white/5`, `border-outline-variant/30`, rounded-xl) — email input + "Join" gold button

**Bottom block** — 3-column accordion (mobile) / 3-column static (desktop), `border-t border-outline-variant/30`:
- **For Renters:** How it works · Browse yachts · Cancellation policy
- **For Owners:** List your yacht · Owner FAQ · Insurance coverage
- **Legal:** Terms of service · Privacy policy · Cookie settings
- On mobile each column is a `<details>` accordion with `expand_more` icon
- Column headings: `label-sm uppercase tracking-widest text-primary`

**Copyright line** below: `© 2024 YACHTBAY INTERNATIONAL. ALL RIGHTS RESERVED.` — `label-sm`, centered

**Social strip** — appears *above* the footer on most pages: `border-t border-outline-variant`, centered row — "Follow Our Journey" label (`text-primary`, `label-sm`, `tracking-[0.2em]`) + 3 icon buttons (Instagram `photo_camera` / Facebook / X) in `rounded-full bg-white/10`

---

## 7. Pages

---

### PAGE 1 — Homepage (`/`)

**Purpose:** Entry point. Primary goal is to get the renter to search.
**Auth:** Public
**SSR:** Yes

#### Layout

**[Global nav — see Section 6]**

---

**Hero section** — `relative`, `aspect-[4/5]` (mobile), full-bleed background image (yacht in Greek waters, golden hour)
- Content anchored to **bottom-left** of the image, `z-10`, `px-margin-mobile pb-24`
- `h1` — "Sail Greece, your way" — `font-display-xl text-4xl text-white`
- `p` — "Browse and instantly book private yacht charters" — `font-body-lg text-lg text-white opacity-90`

**Search bar** — glass-morphic card, `-mt-16 relative z-20` (overlaps the hero bottom), `px-margin-mobile`
- Card: `bg-white/5 border border-outline-variant p-6 rounded-xl shadow-[0_8px_30px_rgb(0,0,0,0.4)]`
- Stacked inputs inside, each with `border-b border-outline-variant pb-2`:
  1. **Location** — `location_on` icon + text input, placeholder "Where in Greece?"
  2. **Check-in / Check-out** — `grid grid-cols-2 gap-4`, each with date input in `label-sm`
  3. **Guests** — `group` icon + number input
- **Search CTA** — `w-full bg-primary text-on-primary py-4 font-label-sm uppercase rounded-lg`

---

**Popular Destinations section** — `py-16`
- Header row (`px-margin-mobile flex justify-between items-end mb-6`):
  - Left: `h2 font-headline-md text-2xl text-white` — "Popular Destinations" + `p text-white text-sm` — "Handpicked locales for your next voyage"
- **Swipeable carousel** — `flex overflow-x-auto gap-4 px-6 no-scrollbar snap-x snap-mandatory`
  - Each card: `min-w-[85%] h-[400px] snap-center relative rounded-2xl overflow-hidden`
  - Full-bleed photo background + `bg-gradient-to-t from-on-tertiary-fixed` overlay
  - Top-left badge chip (e.g. "Best Seller", "Exclusive", "Economy") — `label-sm`
  - Bottom-left text: destination name in `font-headline-lg text-2xl text-primary` + `"From €X / day"` in `text-sm text-white`
  - Destinations: Mykonos (€350), Santorini (€420), Crete (€280) + more
- Dot pagination indicators below carousel (active dot: `w-6 bg-primary`, inactive: `w-2 bg-outline/30`)

---

**How it Works section** — `py-16 px-margin-mobile border-y border-outline-variant/30`
- `h2 font-headline-lg text-center text-white mb-8` — "How it Works"
- **2×2 grid** (`grid grid-cols-2 gap-4 max-w-lg mx-auto`) — four steps as glass cards:
  - Card: `bg-white/5 p-5 rounded-2xl border border-outline-variant/50 flex flex-col items-center text-center`
  - Icon (Material Symbols) in `text-primary-container`, then `h3 font-body-md font-semibold text-white text-sm`, then `p text-[12px] text-white`
  - Step 1 **Browse** — `travel_explore` — "Curated premium yachts in the Med."
  - Step 2 **Book** — `event_available` — "Secure dates with transparent pricing."
  - Step 3 **Sail** — `sailing` — "Embark from partner marinas."
  - Step 4 **Review** — `verified` — "Book with verified community trust."

---

**Our Fleet section** — `py-16`
- Header row (same pattern as Destinations): "Our Fleet" + "Find the perfect vessel for your journey"
- **Swipeable carousel** — same structure as Destinations carousel
  - Cards: Sailboats (€350), Catamarans (€600), Motor Yachts (€850), Luxury Gulets (€1,200)
  - No top badge chip; destination name replaced by yacht type name
- Dot pagination below

---

**Owner CTA banner** — full-width glass panel `bg-white/5 border border-outline-variant/20 p-8 relative overflow-hidden`
- `flex flex-col md:flex-row items-center justify-between` with decorative blurred circles (gold + tertiary)
- Left: `h2 font-headline-lg text-3xl text-white` — "Own a yacht?" + `p font-body-md text-white opacity-90` describing the platform
- Right: `bg-primary text-on-primary px-10 py-4 font-label-sm uppercase` — "List your boat"

---

**[Social strip — see Section 6]**

**[Footer — see Section 6]**

**[Bottom nav bar — see Section 6]**

---

### PAGE 2 — Search Results (`/search`)

**Purpose:** Core discovery page. Renters filter, sort, and browse results.
**Auth:** Public
**SSR:** Yes (initial load), client-side for filter/sort updates
**URL example:** `/search?location=Mykonos&checkIn=2026-07-01&checkOut=2026-07-08&guests=4`

#### Layout

**[Global nav — see Section 6]**

---

**Current Search bar** — `px-margin-mobile mb-6`
- Single row card: `border border-outline-variant/30 p-4 flex items-center justify-between`
- Left: label "Current Search" (`label-sm text-on-surface-variant uppercase`) + active search summary text ("Mykonos • Jul 1 - Jul 8 • 4 guests") in `on-surface`
- Right: `edit` icon (`text-primary`)
- Tapping opens inline search editing

**Filter chips row** — `flex gap-3 px-margin-mobile overflow-x-auto no-scrollbar pb-2 mb-8`
- First chip: **Filters icon button** (`tune` icon, `w-10 h-10 border border-outline-variant/50`) — opens full-screen Filters Modal
- Then pill dropdowns (each `flex-shrink-0 px-4 py-2 border border-outline-variant/50 flex items-center gap-2`):
  - "Yacht Type" + `expand_more`
  - "Crew Option" + `expand_more`
  - "Price" + `expand_more`

**Results count heading** — `px-margin-mobile mb-6`
- `h1 font-headline-md text-on-surface border-l-2 border-primary pl-4` — "34 yachts available in Mykonos"

**Listing cards** — `px-margin-mobile space-y-12` — single column, full-width
- Each card: `border rounded-xl bg-surface-container border-2 border-outline-variant overflow-hidden group`
  - **Photo** — `relative aspect-[4/3] overflow-hidden` — `img w-full h-full object-cover group-hover:scale-105 transition-transform duration-700`
  - Top-right: `favorite` icon (heart, save to wishlist)
  - **Body** — `p-4 space-y-2`:
    - `h3 font-headline-md text-on-surface` — yacht name
    - `p font-body-md text-on-surface-variant` — type + port (e.g. "Luxury Catamaran • Ornos Bay, Mykonos")
    - Stats row `flex items-center gap-4 py-2 border-y border-outline-variant/20`: `group` icon + "X GUESTS" · `directions_boat` icon + "X ft" — all `label-sm uppercase`
    - Crew option tags `flex gap-2` — plain text pills (e.g. "Skippered", "Full Crew") in `label-sm`
    - Price row `pt-4 border-t border-outline-variant/10 flex justify-end`: `font-headline-md text-primary` — "€X,XXX/day"

**Pagination** — `flex items-center justify-center pt-8`
- `chevron_left` button · numbered buttons (active: `bg-primary-container text-on-primary-container`, inactive: `border border-outline-variant`) · `...` ellipsis · `chevron_right` button

**Floating "Show Map" button** — `fixed bottom-24 left-1/2 -translate-x-1/2 z-40`
- `bg-primary text-on-primary px-8 py-3 flex items-center gap-3 shadow-2xl` — `map` icon + "Show Map"
- Tapping switches to map view (Mapbox, pins per yacht)

---

#### Filters Modal (full-screen overlay)

Triggered by the `tune` icon chip. Covers the full screen (`fixed inset-0 z-[100]`, background `surface-container-lowest`).

**Modal header** (`border-b border-outline-variant`):
- Left: `close` icon (dismiss)
- Center: "Filters" — `font-headline-md text-primary uppercase tracking-widest`
- Right: "Reset" — `label-sm text-primary uppercase`

**Modal body** (`flex-1 overflow-y-auto p-margin-mobile space-y-10 pb-32`):
- **Price Range** — dual `min`/`max` input fields + range slider (`range-slider` with `progress` fill in `primary`)
- **Guests** — label + subtext "Total number of travelers" + `−` / `+` counter buttons
- **Bedrooms** — label + subtext "Minimum cabins required" + `−` / `+` counter buttons
- **Boat Length (ft)** — single slider, current value displayed
- **Engine Power (hp)** — min/max text inputs
- **Flexible Cancellation** — toggle switch (`peer-checked` → `bg-primary`) + label + subtext

**Modal footer** (`sticky bottom-0 border-t border-outline-variant p-4`):
- Full-width gold CTA: `bg-primary text-on-primary py-4` — "Show 34 results"

---

**[Social strip — see Section 6]**

**[Footer — see Section 6]**

**[Bottom nav bar — see Section 6]**

---

### PAGE 3 — Yacht Detail (`/yachts/:id`)

**Purpose:** The listing page. Give the renter enough information to decide, then make booking frictionless.
**Auth:** Public
**SSR:** Yes (critical for SEO)
**URL format:** `/yachts/abc123-oceanblue-sailing` (id + slug)

#### Layout

**[Global nav — see Section 6]**

---

**Photo carousel** — `relative w-full overflow-hidden mb-gutter`
- Full-width horizontal swipe carousel, `snap-x snap-mandatory no-scrollbar`
- Each slide: `flex-shrink-0 w-full aspect-[16/9] snap-center relative` — `img w-full h-full object-cover` + gradient overlay (`yacht-carousel-gradient`)
- Dot indicators below: active `w-2 h-2 bg-[#D4AF37]`, inactive `bg-outline-variant/50`

**Photo thumbnails strip** — `px-margin-mobile mb-12`
- Horizontal scroll row of `w-32 aspect-square rounded-lg overflow-hidden border border-outline-variant` thumbnails
- Last thumbnail: `opacity-50` overlay with "+12 Photos" label centered — tapping opens fullscreen lightbox

---

**Yacht header** — `px-margin-mobile mb-4`
- Collection badge: small chip "Premier Collection" in `label-sm text-primary uppercase tracking-widest`
- `h1 font-headline-lg text-[40px] text-on-surface` — yacht name
- `p text-body-lg font-body-lg text-on-surface-variant italic` — type + home port (e.g. "Luxury Catamaran · Ornos Bay, Mykonos")
- Star rating row: 5 `star` icons in `text-primary` + "X.X (XX Verified Charters)" in `text-on-surface-variant text-sm`

**Stats grid** — `px-margin-mobile mb-16` — `grid grid-cols-2 gap-y-10 py-10 border-y border-outline-variant/30`
- 4 cells, each `flex flex-col items-center gap-2`:
  - Material icon (e.g. `groups`, `straighten`, `calendar_today`, `bed`)
  - Value: `text-on-surface font-semibold`
  - Label: `text-[10px] uppercase tracking-widest text-on-surface-variant`
- Stats: Guests capacity · Length (ft) · Build year · Number of cabins

**About the Yacht** — `px-margin-mobile mb-16`
- `h3 text-label-sm font-label-sm text-primary uppercase tracking-widest mb-6` — "About the Yacht"
- Multi-paragraph description in `text-body-md text-on-surface-variant`

**Exclusive Amenities** — `px-margin-mobile mb-16`
- `h3 text-label-sm font-label-sm text-primary uppercase tracking-widest mb-6` — "Exclusive Amenities"
- Initial grid `grid grid-cols-2 gap-y-6`: 4 amenities shown, each `flex items-center gap-3` with Material icon + label
- `<details>` expand/collapse — "Show all amenities" / "Show fewer amenities" in `border border-primary/40 rounded-sm label-sm text-primary uppercase`
- Expanded: shows additional amenities in same 2-column grid

**Charter Options** — `px-margin-mobile mb-16`
- Outer card: `bg-surface-container-low border border-outline-variant/20 p-6 rounded-lg`
- `h3 text-headline-md font-headline-md text-on-surface mb-8` — "Charter Options"
- 3 crew option cards stacked (`space-y-4`), each `flex flex-col gap-3 p-5 rounded border cursor-pointer`:
  - **Selected state:** `border-primary/40 bg-primary/5`
  - **Unselected state:** `border-outline-variant/30 bg-background/50`
  - Row 1: crew option name (bold `text-on-surface`) + price (`text-primary font-bold`) + "/day"
  - Row 2: description in `text-on-surface-variant text-sm`
  - Options: Bareboat · Skippered · Fully Crewed

**Location** — `px-margin-mobile mb-16`
- `h3 text-headline-md font-headline-md text-primary mb-6` — "Location"
- Map container `relative w-full aspect-video rounded-lg overflow-hidden border border-outline-variant/20`:
  - Background map image `opacity-60 brightness-50`
  - Centered pin overlay: `location_on` icon + label chip (`bg-surface-container-highest/90 backdrop-blur border border-primary/40`)
  - Bottom-right zoom controls: `+` / `−` buttons in a stacked glass panel
- Below map: `anchor` icon + "Home Port: [Port Name, Location]"

**Availability Calendar** — `px-margin-mobile mb-16`
- `h2 text-headline-md font-headline-md text-primary mb-12` — "Availability Calendar"
- Monthly grid (`grid grid-cols-7`), one month at a time:
  - Day-of-week headers: `text-[10px] text-on-surface-variant`
  - Available: `p-3 text-on-surface`
  - Blocked/reserved: `p-3 bg-error-container/20 text-on-error-container rounded`
  - Selected/in-range: `p-3 bg-primary/20 text-primary rounded border border-primary/40 font-bold`
  - Past days: `opacity-20`

**Owner Card** — `px-margin-mobile mb-16`
- Card: `bg-surface-container-low p-8 rounded-lg border border-outline-variant/10 text-center`
- Avatar: `w-24 h-24 rounded-full overflow-hidden border-2 border-primary mx-auto mb-6`
- Role label: "Charter Host" in small `text-on-surface-variant`
- Name: `text-headline-md font-headline-md text-on-surface mb-2`
- Meta: `text-on-surface-variant mb-8 text-sm` — "Member since [year] · Platinum Host · 500+ Voyages"
- CTA button: `w-full py-3 border border-primary text-primary text-[10px] font-bold uppercase tracking-widest` — "Message Owner"
- > No contact details shown — revealed only after confirmed booking

---

**Mobile Booking Widget — Sticky Bottom Bar**
Pinned above the bottom nav. Background `surface-container-high`, `border-t border-outline-variant`:
- Left: "From €X/day" in `text-primary font-headline-md`
- Right: "Book Now" — `bg-primary text-on-primary px-6 py-3 rounded font-label-sm uppercase`
- Tapping "Book Now" slides up a bottom sheet with the full booking widget:
  - Date range picker (check-in / check-out, availability-aware)
  - Crew option selector (links to Charter Options above)
  - Live price breakdown: base × days · weekly discount · crew fee · **Total**
  - Full-width "Proceed to Checkout" gold CTA
  - Cancellation policy one-liner ("Free cancellation until [date]")

---

**[Social strip — see Section 6]**

**[Footer — see Section 6]**

**[Bottom nav bar — see Section 6]**

---

### PAGE 4 — Checkout & Payment (`/yachts/:id/checkout`)

**Purpose:** Single-page flow — renter reviews the booking summary, enters card details, and pays without navigating to a second page.
**Auth:** Protected (redirect to `/login?redirect=...` if not logged in)
**SSR:** Partial (page shell SSR; Stripe Elements loads client-side)

#### Layout

> **Header:** Simplified header (not the global nav). Fixed top bar, `bg-surface text-primary flex justify-between`: left = sailing icon + "YachtBay" wordmark (`font-headline-md text-primary`); right = `contact_support` icon + language picker ("EN" + `language` icon).

**Main content** — `pt-24 pb-32 px-margin-mobile max-w-lg mx-auto` (single column, centered, scrollable)

**Page title** — `mb-8`
- `h2 font-headline-lg text-headline-lg text-primary` — "Checkout"
- `p font-body-md text-on-surface-variant mt-2` — "Review your booking and pay to confirm."

**Yacht summary card** — `mb-10 bg-on-tertiary/10 rounded-xl overflow-hidden border border-white/5`
- Hero image strip `relative h-48 w-full`: full-bleed yacht photo + `bg-gradient-to-t from-background/80 to-transparent`
  - Bottom-left overlay: "Confirmed Availability" chip + `h3 font-headline-md text-white` — yacht name
- Body `p-6 space-y-4`:
  - `h4 font-label-sm text-primary uppercase tracking-widest` — "Booking Summary"
  - 2×2 grid (`grid grid-cols-2 gap-4`): DATES · DURATION (right-aligned) · OPTION · PORT (right-aligned)

**Price breakdown card** — `mb-10 p-6 bg-on-tertiary/10 rounded-xl border border-white/5`
- `h4 font-label-sm text-primary uppercase tracking-widest mb-6` — "Price breakdown"
- Line items `space-y-4 border-b border-white/10 pb-6`, each `flex justify-between`:
  - Base rate + amount · Weekly discount (`text-primary-fixed-dim`) · Crew fee + amount
- Total row `pt-6 flex justify-between items-baseline`: "Total" label + bold `text-primary` amount

**Cancellation info** — `bg-primary/5 p-4 rounded-lg border border-primary/20 flex items-start gap-3`
- `info` icon + "Free cancellation until **[date]**. Full refund guaranteed if cancelled before this date."

**Payment form** — `mb-10`
- `h4 font-label-sm text-primary uppercase tracking-widest mb-6` — "Payment"
- **Stripe Elements card input** — embedded Stripe card form (card number, expiry, CVC) styled to match the dark surface: transparent background, `border border-outline-variant`, `text-on-surface`
- Inline error state (no page reload): e.g. "Your card was declined. Please check your details or try a different card." — `text-error text-sm`
- Trust signals row below the input: `lock` icon + "SSL encrypted" · Stripe badge "Secured by Stripe" — `label-sm text-on-surface-variant`

**Confirm CTA** — `w-full h-16 bg-primary text-on-primary font-label-sm text-[16px] tracking-widest` — "Confirm and Pay €X,XXX"
- This is the single action that completes the booking — no intermediate step
- On success: redirect to `/bookings/:id/confirmed`

**Back link** — `block text-center font-label-sm text-on-surface-variant hover:text-primary` — "Back to listing"

---

> **Footer:** 4-column variant — YachtBay tagline · Fleet links · Support links · Newsletter. Copyright line below.

> **Bottom nav:** `fixed bottom-0` 4-tab bar — Explore · Bookings (active) · Saved · Profile.

---

### PAGE 5 — Booking Confirmation (`/bookings/:id/confirmed`)

**Purpose:** Celebrate the booking. Set expectations. Provide owner contact.
**Auth:** Protected
**SSR:** Yes

#### Layout

**Success banner (top, full width)**
- Green success background or confetti animation
- Large heading: "You're going sailing! 🎉"
- Subheading: "Your booking is confirmed. See you on the water."

**Booking summary card:**
- Booking reference number (e.g. "Booking #YB-00123")
- Yacht name + primary photo
- Dates (check-in → check-out)
- Crew option
- Total paid

**"What happens next" section:**
- Step 1: "Confirmation email sent to [email]"
- Step 2: "Owner will be in touch before your charter"
- Step 3: "Any questions? Use the contact details below"

**Owner contact details (revealed for the first time):**
- Owner first name
- Owner email
- Owner phone number
- (Contact details only visible here and on the booking detail page — never on the listing)

**CTA buttons:**
- Primary: "View booking details" → `/bookings/:id`
- Secondary: "Browse more yachts" → `/search`

---

### PAGE 6 — My Bookings (`/bookings`)

**Purpose:** Renter's booking dashboard. Overview of all bookings.
**Auth:** Protected
**SSR:** Yes

#### Layout

- Page heading: "My Bookings"
- Tab navigation: **Upcoming** | **Past** | **Cancelled**
- Each tab shows a list of booking cards

**Booking card (per tab):**
- Yacht primary photo (thumbnail, left)
- Yacht name + type
- Charter dates (e.g. "Jul 1 → Jul 8, 2026")
- Crew option
- Status badge: CONFIRMED (green) / COMPLETED (grey) / CANCELLED (red)
- Total paid
- "View details →" link

**Empty state (per tab):**
- Upcoming: "No upcoming bookings. Ready to explore?" + "Browse yachts" button → `/search`
- Past: "No completed bookings yet."
- Cancelled: "No cancelled bookings."

---

### PAGE 7 — Booking Detail (`/bookings/:id`)

**Purpose:** Full details of a specific booking. Post-booking home base for the renter.
**Auth:** Protected (renter must own the booking)
**SSR:** Yes

#### Layout

**Top: booking header**
- Booking reference (e.g. "Booking #YB-00123")
- Status badge (CONFIRMED / COMPLETED / CANCELLED)
- Charter dates

**Yacht details section:**
- Yacht photo + name + type + home port

**Charter details:**
- Check-in date / Check-out date
- Duration
- Crew option

**Price breakdown:**
- Same breakdown as checkout: base rate × days, discounts, total
- Payment status: "Paid — €X,XXX" (green)

**Owner contact details (post-booking only):**
- Owner first name, email, phone
- Framed clearly: "Your skipper / owner contact:"

**Cancellation policy + deadline:**
- "Cancel by [date] for a full refund" (if within full refund window)
- "Cancel by [date] for a 50% refund" (if within partial refund window)
- "No refund available" (if past all windows)

**Actions:**
- "Cancel booking" button — visible only if:
  - Booking status = CONFIRMED
  - Charter start date is in the future
- Clicking "Cancel booking" opens a **cancellation confirmation modal:**
  - Shows refund amount: "You will receive a refund of €X,XXX"
  - Two buttons: "Yes, cancel my booking" (red/destructive) and "Keep my booking" (secondary)

---

### PAGE 8 — Profile & Account (`/profile`)

**Purpose:** Renter's account settings.
**Auth:** Protected
**SSR:** Yes

#### Layout

Single-column, centered, form-based.

**Personal details section:**
- First name (editable)
- Last name (editable)
- Phone number (editable, optional)
- Email (read-only — displayed with label "Contact support to change your email")
- "Save changes" button

**Change password section:**
- Current password
- New password
- Confirm new password
- "Update password" button

**Account section:**
- "Switch to Owner Dashboard" link — visible only if account has owner role
  - Opens `owners.yachtbay.com` in a new tab

---

### PAGE 9 — Register (`/register`)

**Purpose:** New renter creates an account.
**Auth:** Public (redirect to homepage if already logged in)
**SSR:** Yes

#### Layout

Centered card, single column. Clean and minimal.

**Form fields:**
- First name
- Last name
- Email
- Password (with strength indicator)
- Confirm password

**Submit:** "Create account" button — full width

**Below form:**
- "Already have an account? Log in" → `/login`

**After submission:**
- Page transitions to a "Check your email" confirmation state (no full redirect):
  - "We've sent a verification link to [email]"
  - "Click the link in the email to activate your account"
  - "Didn't receive it? Resend email" link

---

### PAGE 10 — Login (`/login`)

**Purpose:** Existing renter logs in.
**Auth:** Public (redirect to intended destination if already logged in)
**SSR:** Yes
**URL:** Supports `?redirect=/yachts/abc123/checkout` — redirects back after login

#### Layout

Centered card, single column.

**Form fields:**
- Email
- Password

**Submit:** "Log in" button — full width

**Below form:**
- "Forgot your password?" → `/forgot-password`
- "Don't have an account? Register" → `/register`

**Error states:**
- Invalid credentials: "Incorrect email or password." (generic — do not reveal which)
- Unverified account: "Please verify your email before logging in. Resend verification email."

---

### PAGE 11 — Forgot Password (`/forgot-password`)

**Purpose:** Renter requests a password reset link.
**Auth:** Public
**SSR:** Yes

#### Layout

Centered card, single column.

**Form fields:**
- Email

**Submit:** "Send reset link" button

**After submission:**
- Always show success state regardless of whether email exists (prevents enumeration):
  - "If an account exists for [email], we've sent a reset link. Check your inbox."

---

### PAGE 12 — Reset Password (`/reset-password`)

**Purpose:** Renter sets a new password using the link from their email.
**Auth:** Public (token in URL)
**SSR:** Yes

#### Layout

Centered card, single column.

**Form fields:**
- New password (with strength indicator)
- Confirm new password

**Submit:** "Reset password" button

**Error states:**
- Expired/invalid token: "This reset link has expired. Request a new one." + link to `/forgot-password`
- Password mismatch: inline field error

---

### PAGE 13 — Email Verification (`/verify-email`)

**Purpose:** Validates the email verification token from the registration email.
**Auth:** Public (token in URL)
**SSR:** Yes

#### Layout

Centered, minimal — status page only (no form).

**Success state:**
- Green checkmark icon
- "Email verified! Your account is ready."
- CTA: "Start exploring yachts" → `/`

**Error state:**
- "This verification link has expired or is invalid."
- "Resend verification email" link (triggers new email)

---

### PAGE 14 — How It Works (`/how-it-works`)

**Purpose:** Explains YachtBay to first-time visitors. Builds trust.
**Auth:** Public
**SSR:** Yes

#### Layout

**For Renters section:**
- Step-by-step: Search → Compare → Book → Sail
- Key points: Instant booking, transparent pricing, secure Stripe payment, 24/7 access to owner contact

**For Owners section:**
- Step-by-step: List your yacht → Set your price → Get bookings → Get paid
- CTA: "List your yacht →" links to `owners.yachtbay.com/register`

**Trust & Safety section:**
- Stripe-secured payments
- Verified listings
- Clear cancellation policy

---

### PAGE 15 — Cancellation Policy (`/cancellation-policy`)

**Purpose:** Full platform cancellation policy — referenced from listings and bookings.
**Auth:** Public
**SSR:** Yes

#### Content

**Day Trips:**

| Time before charter | Refund |
|---|---|
| 5+ days before | 100% refund |
| 1–4 days before | 50% refund |
| Less than 24 hours | No refund |

**Multi-day and Weekly Charters:**

| Time before charter | Refund |
|---|---|
| 30+ days before | 100% refund |
| 14–29 days before | 50% refund |
| Less than 14 days | No refund |

**Notes:**
- Platform commission is never refunded (industry standard)
- Refunds are processed automatically and returned to the original payment method
- Card refund processing time: 5–10 business days (Stripe dependent)

---

### PAGE 16 — Terms of Service (`/terms`)

Standard legal page. Clean typography, well-structured with section headings. No special UI.

---

### PAGE 17 — Privacy Policy (`/privacy`)

Standard legal page. Clean typography, well-structured with section headings. No special UI.

---

## 8. Shared Components

These components appear across multiple pages and should be designed consistently.

### Listing Card
Used on: Homepage (featured section), Search results
- Photo (4:3 ratio, full card width)
- "New" label badge (top left — replaces star rating at MVP)
- Yacht name (bold, 1 line, truncate)
- Type + Home port (secondary text, e.g. "Sailboat · Mykonos")
- Capacity (e.g. "Up to 8 guests")
- Crew option badges (small pills: "Bareboat", "Skippered", "Fully Crewed")
- Price: "From €350 / day" (bottom right, prominent)
- Hover state: subtle shadow lift

### Booking Widget
Used on: Yacht detail page (sticky sidebar), Checkout, Payment
- Card with border and shadow
- Date range picker (check-in / check-out fields)
- Crew option selector
- Price preview (live-updating)
- "Book Now" CTA

### Status Badges
- CONFIRMED: green background, white text
- COMPLETED: grey background, white text
- CANCELLED: red background, white text
- PENDING: amber background, white text

### Crew Option Badge
Small pill label, neutral background:
- "Bareboat" — blue pill
- "Skippered" — teal pill
- "Fully Crewed" — indigo pill

### Date Picker
- Availability-aware: blocked dates visually greyed and not selectable
- Inline calendar, 1 month shown on mobile, 2 months on desktop
- Selected date range highlighted (start → end, filled band between)

### Price Breakdown
Displayed on: Yacht detail widget, Checkout, Booking detail
```
Base rate (€350/day × 7 days)      €2,450
Weekly rate discount (−10%)        −€245
Crew fee (Skippered)               +€150
────────────────────────────────────────
Total                              €2,355
```

### Cancellation Modal
Destructive action confirmation. Two-button pattern:
- Primary action (destructive): "Yes, cancel my booking" — red/danger button
- Secondary: "Keep my booking" — outline button
- Always shows the refund amount before confirming

### Empty States
Each empty state should have:
- Illustrative icon or minimal illustration
- Short, friendly heading
- Subtext
- Optional CTA button

---

## 9. Renter User Stories (MVP)

All stories are for the renter persona. Phase = MVP unless noted.

---

### Epic 1 — Authentication & Account

**R-001 — Register as a renter**
As a renter, I want to create an account with my email and password, so that I can save my details and manage my bookings.

Acceptance criteria:
- Renter can register with email, password, first name, last name
- Email must be unique — duplicate registrations rejected with a clear error
- Password minimum: 8 characters
- Verification email sent after registration
- Renter cannot book until email is verified
- After verification, renter is redirected to homepage or original intended destination

---

**R-002 — Login**
As a renter, I want to log in with my email and password, so that I can access my account and bookings.

Acceptance criteria:
- Renter can log in with email and password
- On success: session established, renter redirected to intended destination or homepage
- On failure: generic error ("Incorrect email or password" — do not reveal whether the email exists)
- Unverified email: rejected with prompt to verify + option to resend verification

---

**R-003 — Logout**
As a renter, I want to log out of my account, so that my session is terminated securely.

Acceptance criteria:
- Logout clears the session
- Renter is redirected to homepage
- Previously active session tokens no longer work

---

**R-004 — Reset password**
As a renter, I want to reset my password if I forget it, so that I can regain access to my account.

Acceptance criteria:
- Renter requests reset by entering email
- Reset email sent with a secure link, valid for 1 hour
- Link opens a form to enter and confirm a new password
- After reset, all active sessions are invalidated
- If email not found, response is identical (no email enumeration)

---

### Epic 2 — Search & Discovery

**R-005 — Search yachts by location and dates**
As a renter, I want to search for available yachts by location and dates, so that I can find options that match my travel plans.

Acceptance criteria:
- Search form accepts: location (text with autocomplete), check-in date, check-out date, number of guests
- Results show only yachts available for the full selected date range
- Empty results show a clear message and suggestions to adjust search
- Search state persists in the URL (shareable and bookmarkable links)
- Search is accessible from both the homepage and the search page

---

**R-006 — Filter search results**
As a renter, I want to filter search results by yacht type, crew option, and price range, so that I can narrow down options to what suits me.

Acceptance criteria:
- Filters available: yacht type, crew option, price range (min–max per day), capacity
- Filters apply without a full page reload
- Active filters are clearly visible as dismissible tags
- Filter state persists in URL params

---

**R-007 — Sort search results**
As a renter, I want to sort search results, so that I can find the best match quickly.

Acceptance criteria:
- Sort options: Price (low to high), Price (high to low), Relevance (default)
- Sort selection persists in URL params

---

**R-008 — View yachts on a map**
As a renter, I want to see yacht home ports on a map, so that I can understand where yachts are based relative to my destination.

Acceptance criteria:
- Search results page shows a map alongside the results list (split view on desktop)
- Each yacht is represented by a pin at its home port
- Clicking a pin highlights the listing card and shows a mini preview (photo, name, price, link)
- Map updates when filters change
- Map powered by Mapbox

---

**R-009 — View listing cards**
As a renter, I want to see key information for each yacht in search results, so that I can quickly compare options without opening each one.

Acceptance criteria:
- Card shows: primary photo, yacht name, type, home port, capacity, crew options available, price per day
- Price shown reflects the cheapest available crew option for the selected dates
- "New" badge shown (no reviews at MVP)
- Card links to the yacht detail page

---

### Epic 3 — Yacht Detail

**R-010 — View yacht detail page**
As a renter, I want to view the full details of a yacht, so that I can decide whether it meets my needs before booking.

Acceptance criteria:
- Page shows: photo gallery (multiple photos, fullscreen lightbox), yacht name, type, length, capacity, build year, manufacturer/model, home port, description
- Available crew options with pricing for each
- Cancellation policy clearly displayed
- Owner info: first name + member since (no contact details until booked)
- Page is server-side rendered (SEO)

---

**R-011 — View yacht availability calendar**
As a renter, I want to see which dates are available for a yacht, so that I can plan my booking.

Acceptance criteria:
- Calendar shows current and next 3 months
- Available dates visually distinct from blocked/reserved dates
- Renter cannot select unavailable dates in the booking widget
- Calendar is read-only — date selection happens in the booking widget

---

**R-012 — View price for selected dates and crew option**
As a renter, I want to see the price for my specific dates and chosen crew option, so that I know the exact cost before committing.

Acceptance criteria:
- Price updates dynamically as renter selects dates and crew option
- Price breakdown shown: base rate × days, crew fee (if applicable), weekly discount (if applicable), total
- Weekly rate applied automatically if duration = 7 days
- Platform commission NOT shown to renter (included in owner's listed price)
- Price always calculated server-side — client never calculates price

---

### Epic 4 — Booking Flow

**R-013 — Select booking parameters**
As a renter, I want to select check-in/check-out dates and crew option, so that I can configure my booking before paying.

Acceptance criteria:
- Date picker is availability-aware — unavailable dates are disabled
- Crew option selector shows all options available for that yacht with prices
- Minimum charter duration enforced — invalid ranges show a clear error
- Selection can be changed before confirming

---

**R-014 — View booking summary and price breakdown**
As a renter, I want to see a clear summary of my booking before paying, so that I can confirm all details are correct.

Acceptance criteria:
- Summary shows: yacht name + photo, dates, duration, crew option, full price breakdown, cancellation policy
- "Proceed to Payment" CTA proceeds to payment step
- Renter can go back without losing their selection

---

**R-015 — Instant book — complete payment**
As a renter, I want to pay for my booking instantly, so that my charter is confirmed without waiting for owner approval.

Acceptance criteria:
- Renter enters card details via Stripe Elements
- Payment processed immediately on submission
- If payment fails, inline error shown — renter can retry without starting over
- On success: booking confirmed instantly — no owner approval required
- Renter redirected to booking confirmation page

---

**R-016 — Receive booking confirmation**
As a renter, I want to receive a booking confirmation with all details, so that I have a record and know what to expect.

Acceptance criteria:
- Confirmation page shown immediately after successful payment
- Confirmation email sent within 60 seconds
- Email contains: booking reference, yacht name + photo, dates, crew option, total paid, owner contact details, cancellation policy reminder
- Owner contact details (email + phone) revealed for the first time on this page

---

### Epic 5 — Booking Management

**R-017 — View all my bookings**
As a renter, I want to see all my bookings in one place, so that I can track my upcoming and past charters.

Acceptance criteria:
- Dashboard shows bookings grouped by: Upcoming / Past / Cancelled
- Each booking card shows: yacht photo, name, dates, status, total paid
- Links to full booking detail page

---

**R-018 — View booking details**
As a renter, I want to view the full details of a specific booking, so that I have all the information I need for my charter.

Acceptance criteria:
- Detail page shows: booking reference, yacht details, dates, crew option, price breakdown, payment status, owner contact details, cancellation policy and deadline
- For upcoming bookings: cancellation option visible if within policy window

---

### Epic 6 — Cancellation

**R-019 — Cancel a booking**
As a renter, I want to cancel a booking, so that I can get a refund if my plans change.

Acceptance criteria:
- Cancel button visible on booking detail for upcoming CONFIRMED bookings
- Before confirming cancellation: renter sees the exact refund amount based on the cancellation policy
- On confirmation: booking status → CANCELLED
- Refund processed automatically via Stripe
- Cancellation confirmation email sent to renter
- Owner notified of cancellation by email
- Platform commission is NOT refunded

---

**R-020 — Receive cancellation confirmation**
As a renter, I want to receive confirmation of my cancellation with refund details, so that I know when to expect my money back.

Acceptance criteria:
- Cancellation confirmation email sent within 60 seconds
- Email contains: booking reference, cancelled dates, refund amount, expected timeline (5–10 business days)
- Booking status updated to CANCELLED immediately

---

### Epic 7 — Profile

**R-021 — View and edit profile**
As a renter, I want to view and update my personal details, so that my account information stays accurate.

Acceptance criteria:
- Profile page shows: first name, last name, email (read-only), phone number (optional)
- Renter can update first name, last name, phone number
- Changes saved with confirmation message ("Profile updated")

---

**R-022 — Switch to owner mode**
As a renter who also owns a yacht, I want to access the owner panel from my account, so that I can manage both roles without creating a separate account.

Acceptance criteria:
- If the account has an owner role, a "Switch to Owner Dashboard" link appears in the nav avatar menu and on the profile page
- Link opens the owner panel in a new tab
- Authentication is shared — no re-login required

---

## 10. Key Business Rules (for UI Behavior)

These rules drive dynamic UI behavior — Stitch should reflect them in the design.

| Rule | UI Impact |
|---|---|
| Instant booking | No "Request to book" or "Await owner approval" states — booking is CONFIRMED on payment success |
| Price always server-side | Never show a price as "estimated" — only show the confirmed server-calculated price |
| Owner contact hidden pre-booking | Owner email + phone only appear on booking confirmation page and booking detail |
| Email verification required to book | Unverified users can browse but cannot proceed to checkout |
| Cancellation policy is snapshot | The policy shown at booking time is frozen — the renter's cancellation terms never change even if platform policy changes later |
| Platform commission non-refundable | On cancellation, show clearly: refund = total − platform commission |
| PREP days | Days immediately before/after a booking are automatically blocked — not selectable in the date picker |
| Currency | EUR only at MVP |
| Language | English only at MVP |

---

## 11. Route Summary

| Route | Auth | Description |
|---|---|---|
| `/` | Public | Homepage |
| `/search` | Public | Search results + map |
| `/yachts/:id` | Public | Yacht detail + booking widget |
| `/yachts/:id/checkout` | Protected | Checkout summary + Stripe payment (single page) |
| `/bookings` | Protected | My bookings dashboard |
| `/bookings/:id` | Protected | Booking detail + cancel |
| `/bookings/:id/confirmed` | Protected | Post-booking confirmation |
| `/profile` | Protected | Account settings |
| `/register` | Public | Create account |
| `/login` | Public | Log in |
| `/forgot-password` | Public | Request password reset |
| `/reset-password` | Public | Set new password |
| `/verify-email` | Public | Verify email token |
| `/how-it-works` | Public | Platform explainer |
| `/cancellation-policy` | Public | Full cancellation policy |
| `/terms` | Public | Terms of service |
| `/privacy` | Public | Privacy policy |
