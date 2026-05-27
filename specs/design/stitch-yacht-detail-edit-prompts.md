---
title: Stitch Edit Prompts — Yacht Detail Page Improvements
tags:
  - spec
  - design
  - stitch
status: ready
last-updated: 2026-05-09
---

# Stitch Edit Prompts — Yacht Detail Page

Use these prompts one at a time on the **"Yacht Detail - Additional Services & Equipment"** screen.
Each prompt is self-contained. Paste directly into the Stitch edit chat.

> **Design tokens to remember across all edits:**
> - Background: `#0d1c32` | Cards: `bg-surface-container-low` (`#1a1c1c`) | Primary gold: `#D4AF37`
> - Body text: `#e2e2e2` | Muted: `#d0c5af` | Borders: `#4d4635`
> - Error/warning: `#ffb4ab` | Font: Noto Serif + Playfair Display

---

## CRITICAL CHANGES (do these first)

---

### PROMPT 1 — Add "Instant Book" badge to the title block

```
In the yacht title section, directly below the yacht name "The Celestial Horizon" and above the star rating row, add an "Instant Book" badge chip.

Badge design:
- Background: rgba(34,197,94,0.12) (green tint)
- Border: 1px solid rgba(34,197,94,0.35)
- Text: "INSTANT BOOK" — 10px, uppercase, letter-spacing 0.1em, color #4ade80
- Icon: bolt (Material Symbols, filled, 14px) to the left of the text
- Padding: 4px 10px, border-radius full (pill shape)
- Display: inline-flex, align-items center, gap 4px

This badge must remain visible immediately below the yacht name without scrolling. Do not move or modify the yacht name, subtitle, location row, or star rating.
```

---

### PROMPT 2 — Add Save and Share icons to the title block

```
In the yacht title section, add a row of two icon buttons positioned to the top-right of the title block (or as a flex row to the right of the yacht name, space-between layout).

- Heart icon (favorite_border, Material Symbols): saves the listing to wishlist. Color #d0c5af, 24px. On tap changes to favorite (filled) in #D4AF37.
- Share icon (share, Material Symbols): shares the listing. Color #d0c5af, 24px.

Both icons should be in a flex row with gap-4, no background, no border. They should sit visually to the right of or above the yacht name. Do not move the existing yacht name, badge, location or star rating elements.
```

---

### PROMPT 3 — Add "Premier Collection" badge above the yacht name

```
Above the <h1> "The Celestial Horizon" yacht name, add a small collection badge chip on its own line.

Badge design:
- Text: "PREMIER COLLECTION" — 10px, uppercase, letter-spacing 0.15em, color #D4AF37
- No background, no border — plain text label with a thin 1px bottom underline in #D4AF37 at 40% opacity
- OR: Small pill with background rgba(212,175,55,0.08), border 1px solid rgba(212,175,55,0.25), color #D4AF37
- Margin-bottom: 8px before the h1

Do not modify any other element in the title block.
```

---

### PROMPT 4 — Re-add Charter Options section (currently missing)

```
Add a "Charter Options" section between the "Additional Services & Equipment" section and the "Location" section.

Section heading: "Charter Options" — Playfair Display, 24px (text-2xl), color #D4AF37, margin-bottom 32px.

Show 3 selectable option cards stacked vertically, gap 16px between them. Each card:
- Padding: 24px
- Border-radius: 4px
- Default state: border 1px solid rgba(212,175,55,0.3), background rgba(212,175,55,0.03)
- Selected state: border 1px solid #D4AF37, background rgba(212,175,55,0.08)

Card 1 — BAREBOAT (default selected):
- Row 1: "Bareboat" label (10px uppercase, letter-spacing 0.2em, #D4AF37, font-bold) + "€1,250" (Playfair Display, 24px, #D4AF37) + "/day" (12px, #d0c5af)
- Row 2: "You navigate. Full freedom, no crew included. Fuel and port fees not included." — 14px, #d0c5af, margin-top 8px

Card 2 — SKIPPERED:
- Row 1: "Skippered" + "€1,650 /day"
- Row 2: "Professional skipper included. You enjoy, they navigate. Fuel included." — 14px, #d0c5af

Card 3 — FULLY CREWED:
- Row 1: "Fully Crewed" + "€2,100 /day"
- Row 2: "Skipper + stewardess/chef. The complete luxury experience." — 14px, #d0c5af

Only one card can be selected at a time. Tapping a card marks it selected (gold border + bg tint). The selected price should reflect in the sticky bottom booking bar.
```

---

### PROMPT 5 — Add prices to Optional and Obligatory services

```
In the "Additional Services & Equipment" section, update each service row to show a price next to the badge.

Current rows to update:

OPTIONAL items — add price in #D4AF37 font-bold text-sm to the right of the badge:
- Stand-Up Paddleboard: "+€25 /day"
- WiFi (10GB): "+€15 /day"

OBLIGATORY items — add price in #ffb4ab (error color) font-bold text-sm to the right of the badge, and change the badge color:
- Transit Log: "€45" — change badge to: border border-red-500/30, text #ffb4ab, background rgba(255,180,171,0.06), label "OBLIGATORY"
- Final Cleaning: "€180" — same badge style as above

INCLUDED items — no price shown, keep green badge as-is.

The layout of each row should be:
[Icon] [Label]  ·  [Price if applicable]  [Badge]

Do not add or remove any rows. Do not modify section heading or surrounding layout.
```

---

### PROMPT 6 — Make Optional extras selectable (Add buttons)

```
In the "Additional Services & Equipment" section, for the two OPTIONAL items only (Stand-Up Paddleboard and WiFi 10GB), replace the plain "OPTIONAL" badge with an interactive "Add" button.

Add button design:
- Text: "+ ADD"
- Size: 28px height, px-3 padding
- Border: 1px solid rgba(212,175,55,0.5)
- Background: transparent
- Text color: #D4AF37
- Font: 10px uppercase, letter-spacing 0.1em, font-bold
- Border-radius: 4px

When tapped, the button toggles to "ADDED" state:
- Background: rgba(212,175,55,0.15)
- Border: 1px solid #D4AF37
- Text: "✓ ADDED" in #D4AF37

Add a small note below the Optional section: "Optional extras are added to your booking total." — 11px, #d0c5af, italic.

Do not change the INCLUDED or OBLIGATORY items.
```

---

## TRUST & CONTENT CHANGES

---

### PROMPT 7 — Move Captain section above Reviews

```
Move the entire "Meet Captain Nikos" section (avatar, bio, stats grid, Message Owner button) so it appears BEFORE the "Guest Reviews" section, not after Similar Boats.

The new section order from top to bottom should be:
1. Title block
2. Specs grid
3. About the Yacht
4. Exclusive Amenities
5. Additional Services & Equipment
6. Charter Options
7. Location
8. Meet Captain Nikos  ← moved here
9. Guest Reviews
10. Cancellation Policy
11. FAQ
12. Similar Boats

Do not modify the content or styling of the Captain section. Only move its position.
```

---

### PROMPT 8 — Improve the Captain section

```
In the "Meet Captain Nikos" section, make the following changes:

1. Add a "Verified Host" badge directly below the avatar image, centered:
   - Text: "✓ VERIFIED HOST" — 10px uppercase, letter-spacing 0.1em, color #4ade80
   - Background: rgba(34,197,94,0.1), border 1px solid rgba(34,197,94,0.25), border-radius full, padding 3px 10px
   - Display: inline-flex, margin: 0 auto 16px

2. Change the "MESSAGE OWNER" button text to: "Ask Captain Nikos a question"
   Keep all existing button styling unchanged.

Do not modify any other element in this section.
```

---

### PROMPT 9 — Improve Guest Reviews section

```
In the "Guest Reviews" section, make the following changes:

1. Change the second review card (Marcus) from 5 stars to 4 stars — show 4 filled stars and 1 empty star (star_outline). Keep the review text as-is.

2. Add an avatar initial circle to each review card, to the left of the reviewer name:
   - Size: w-8 h-8 (32px), border-radius full
   - Background: rgba(212,175,55,0.15), border 1px solid rgba(212,175,55,0.3)
   - Text: first letter of reviewer name — 14px, font-bold, color #D4AF37, centered
   - Cards: "Eleni" → "E", "Marcus" → "M", "Sophie" → "S"
   - Layout: flex row, gap 12px, avatar left, [name + date] right

3. Change review card width from w-[85%] to w-72 (288px fixed). This shows a better peek of the next card and works across screen widths.

4. Change review card background from #0f2540 to bg-surface-container-low and border from #1e3a5a to border-outline-variant (#4d4635). Do not change text colors.

Do not add or remove review cards. Do not change review text content.
```

---

### PROMPT 10 — Improve Similar Boats section

```
In the "Similar Boats" section, make the following changes:

1. Change the card border from border-[1.5px] border-gold to border border-outline-variant (#4d4635). The gold border makes all cards too visually heavy.

2. Change the card background from #0f2540 to bg-surface-container-low.

3. On each card, change the price display from "€850" to "From €850" (add "From" prefix in 10px #d0c5af before the price).

4. Add a star rating row to each card, between the crew option badges and the price:
   - 5 filled stars (star, Material Symbols, FILL 1, 12px) in #D4AF37
   - Next to them: a rating number in 11px #d0c5af
   - Card 1 (Aegean Spirit): 4.8
   - Card 2 (Ocean Pearl): 4.7
   - Card 3 (Azure Breeze): 4.9

5. Use different yacht photos for each card. Use the existing project photos already uploaded — pick 3 different yacht/sea images from the project assets. Do not use the same image for all 3 cards.

Do not change card structure, headings, or crew badge styling.
```

---

## VISUAL POLISH CHANGES

---

### PROMPT 11 — Fix section heading size consistency

```
Standardize all H2 section headings in the yacht detail content sheet to the same size. Use Playfair Display, 24px (text-2xl), color #D4AF37 for all of these sections:

- "About the Yacht" (currently text-xl — increase to text-2xl)
- "Exclusive Amenities" (currently text-xl — increase to text-2xl)
- "Additional Services & Equipment" (currently text-xl — increase to text-2xl)

The following are already at text-2xl and should remain unchanged:
- "Location", "Guest Reviews", "Similar Boats", "Frequently Asked Questions", "Cancellation Policy"

Do not modify any other styling, spacing, or content.
```

---

### PROMPT 12 — Fix FAQ and Cancellation Policy card colors

```
In both the "Frequently Asked Questions" section and the "Cancellation Policy" section, replace the hardcoded background and border colors with design system tokens:

- Replace background #0f2540 with: background-color: #1a1c1c (surface-container-low)
- Replace border #1e3a5a with: border-color: #4d4635 (outline-variant)

Apply this to:
- All <details> accordion items in the FAQ section
- The policy table card in the Cancellation Policy section

Do not change text colors, font sizes, padding, or any other styling. Do not change content.
```

---

### PROMPT 13 — Clarify Cancellation Policy intro text

```
In the "Cancellation Policy" section, replace the current introductory paragraph:
"Cancellations up to 3 days after booking are free."

With this two-part structure:

First line (above the table):
Label: "COOLING-OFF PERIOD" — 10px uppercase, letter-spacing 0.1em, color #D4AF37, margin-bottom 4px
Text: "Free cancellation within 72 hours of booking, as long as the charter is more than 30 days away." — 13px, color #d0c5af, margin-bottom 16px

Then a divider line (border-t border-outline-variant, margin-bottom 16px), then the label:
"CANCELLATION SCHEDULE" — 10px uppercase, letter-spacing 0.1em, color #D4AF37, margin-bottom 12px

Then the existing 3-row policy table (unchanged).

Do not modify the table rows, the link to /cancellation-policy, or any other content.
```

---

### PROMPT 14 — Fix the Amenities section duplicate items

```
In the "Exclusive Amenities" section, fix two duplicate items:

1. Remove "High-Speed Starlink" from the hidden/expanded grid (extra-amenities). "Starlink WiFi" in the main grid already covers this. Replace it with a new item: icon "anchor" + label "Safety Equipment Onboard"

2. Remove "Paddleboards" from the main amenities grid. It is already listed as a service in the Additional Services section. Replace it with: icon "kitchen" + label "Chef's Galley Kitchen"

All other amenity items remain unchanged. Do not change the expand/collapse logic or grid layout.
```

---

### PROMPT 15 — Add specs grid dividers and expand to 6 stats

```
In the specs grid section (the 2×2 grid with Guests, Length, Build Year, Cabins), make the following changes:

1. Add a vertical divider between the two columns: each left-column cell gets border-r border-gold/20.

2. Add a horizontal divider between row 1 and row 2: cells in the second row get border-t border-gold/10 and padding-top 24px.

3. Change the grid from 2×2 to 2×3 by adding 2 more stat cells:
   - Cell 5: icon "sailing" + value "Catamaran" + label "Type"
   - Cell 6: icon "verified" + value "Certified" + label "Safety Rating"

Keep all existing cell content and styling unchanged. Only add the dividers and the 2 new cells.
```

---

### PROMPT 16 — Improve Location map

```
In the "Location" section, make the following changes:

1. Increase the map background image opacity from 40% to 55% (change opacity-40 to opacity-55 or style="opacity: 0.55").

2. Change the map container border from border border-gold to border border-outline-variant (#4d4635).

3. Below the existing "Home Port: Ornos Bay, Mykonos" anchor row, add an "Open in Maps" link:
   - Icon: open_in_new (Material Symbols, 14px, #d0c5af)
   - Text: "Open in Google Maps" — 11px, color #d0c5af, underline on hover
   - Href: "#" (placeholder)
   - Display: flex, align-items center, gap 6px, margin-top 8px

Do not change any other element in this section.
```

---

### PROMPT 17 — Improve breadcrumb navigation

```
In the breadcrumb navigation at the top of the content sheet, make two changes:

1. Before "Home", add a back button:
   - Icon: arrow_back (Material Symbols, 18px)
   - Color: #d0c5af
   - No label, no background
   - Margin-right: 8px
   - On tap: goes back (href="#")
   - Separated from "Home" by a small gap, not a chevron

2. Change the middle breadcrumb from "Mykonos" to "Mykonos Catamarans" — same styling as before (12px, #94a3b8, hover:text-primary).

Do not change the final breadcrumb ("The Celestial Horizon" in #D4AF37) or any other element.
```

---

## SECTION ORDER (do this last, after individual changes are done)

---

### PROMPT 18 — Reorder page sections

```
Reorder the sections in the yacht detail content sheet to match this sequence from top to bottom:

1. Breadcrumb navigation
2. Title block (yacht name, badges, location, rating, save/share)
3. Specs grid
4. About the Yacht
5. Exclusive Amenities
6. Additional Services & Equipment
7. Charter Options
8. Meet Captain Nikos
9. Guest Reviews
10. Location
11. Cancellation Policy
12. Frequently Asked Questions
13. Similar Boats

Then at the bottom: Social strip → Footer (unchanged).

Do not modify the content or styling of any section. Only change their vertical order within the page.
Keep the sticky bottom booking bar and bottom nav exactly as they are.
```

---

## NOTES

- Run prompts 1–6 first (critical/conversion)
- Run prompts 7–10 next (trust & content)
- Run prompts 11–17 next (visual polish)
- Run prompt 18 last (reorder) — only after all content changes are done, so Stitch doesn't lose the new sections during reorder
- Each prompt is designed to be used individually. Do not combine multiple prompts into one Stitch request — Stitch performs better with focused, single-concern instructions.
