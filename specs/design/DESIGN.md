---
name: Editorial Luxury
modes:
  - dark    # default
  - light
colors:
  dark:
    surface: '#121414'
    surface-dim: '#121414'
    surface-bright: '#37393a'
    surface-container-lowest: '#0c0f0f'
    surface-container-low: '#1a1c1c'
    surface-container: '#1e2020'
    surface-container-high: '#282a2b'
    surface-container-highest: '#333535'
    on-surface: '#e2e2e2'
    on-surface-variant: '#d0c5af'
    inverse-surface: '#e2e2e2'
    inverse-on-surface: '#2f3131'
    outline: '#99907c'
    outline-variant: '#4d4635'
    surface-tint: '#e9c349'
    primary: '#f2ca50'
    on-primary: '#3c2f00'
    primary-container: '#d4af37'
    on-primary-container: '#554300'
    inverse-primary: '#735c00'
    secondary: '#b9c7e4'
    on-secondary: '#233148'
    secondary-container: '#3c4962'
    on-secondary-container: '#abb9d6'
    tertiary: '#becff0'
    on-tertiary: '#20314b'
    tertiary-container: '#a2b3d4'
    on-tertiary-container: '#354561'
    error: '#ffb4ab'
    on-error: '#690005'
    error-container: '#93000a'
    on-error-container: '#ffdad6'
    primary-fixed: '#ffe088'
    primary-fixed-dim: '#e9c349'
    on-primary-fixed: '#241a00'
    on-primary-fixed-variant: '#574500'
    secondary-fixed: '#d6e3ff'
    secondary-fixed-dim: '#b9c7e4'
    on-secondary-fixed: '#0d1c32'
    on-secondary-fixed-variant: '#39475f'
    tertiary-fixed: '#d6e3ff'
    tertiary-fixed-dim: '#b6c7e8'
    on-tertiary-fixed: '#091c35'
    on-tertiary-fixed-variant: '#374763'
    background: '#121414'
    on-background: '#e2e2e2'
    surface-variant: '#333535'
  light:
    surface: '#faf6ec'                     # warm ivory — main canvas
    surface-dim: '#ede7d6'                 # dimmer cream for resting surfaces
    surface-bright: '#fefcf6'              # brightest cream — almost white but warm
    surface-container-lowest: '#ffffff'    # pure white reserved for highest contrast
    surface-container-low: '#f5efde'       # subtle elevation
    surface-container: '#f0e9d4'           # standard card surface
    surface-container-high: '#ebe3c8'      # elevated card surface
    surface-container-highest: '#e5dcbc'   # highest elevation
    on-surface: '#1d1c14'                  # deep warm ink (not pure black)
    on-surface-variant: '#4d4635'          # warm khaki for muted text
    inverse-surface: '#2f3131'             # dark variant for snackbar/inverse zones
    inverse-on-surface: '#e2e2e2'          # light text on inverse
    outline: '#7d7563'                     # medium warm grey for borders
    outline-variant: '#d0c5af'             # light warm beige for hairlines
    surface-tint: '#d4af37'                # gold-tinted elevation overlay
    primary: '#735c00'                     # deep gold for accessible buttons on cream
    on-primary: '#ffffff'                  # white text on deep gold
    primary-container: '#ffe088'           # pale gold tonal button background
    on-primary-container: '#241a00'        # very dark amber on pale gold
    inverse-primary: '#f2ca50'             # bright dark-mode gold preserved for inverse zones
    secondary: '#485879'                   # deep slate blue
    on-secondary: '#ffffff'                # white text on deep slate
    secondary-container: '#d6e3ff'         # pale blue tonal button
    on-secondary-container: '#313f57'      # dark navy on pale blue
    tertiary: '#485977'                    # deep slate blue (subtly distinct from secondary)
    on-tertiary: '#ffffff'                 # white text on deep slate
    tertiary-container: '#d6e3ff'          # pale blue
    on-tertiary-container: '#2f4060'       # dark navy on pale blue
    error: '#ba1a1a'                       # MD3 standard light error red
    on-error: '#ffffff'                    # white on error red
    error-container: '#ffdad6'             # pale red error background
    on-error-container: '#410002'          # very dark red on pale red
    primary-fixed: '#ffe088'                # mode-agnostic per MD3
    primary-fixed-dim: '#e9c349'            # mode-agnostic
    on-primary-fixed: '#241a00'             # mode-agnostic
    on-primary-fixed-variant: '#574500'     # mode-agnostic
    secondary-fixed: '#d6e3ff'              # mode-agnostic
    secondary-fixed-dim: '#b9c7e4'          # mode-agnostic
    on-secondary-fixed: '#0d1c32'           # mode-agnostic
    on-secondary-fixed-variant: '#39475f'   # mode-agnostic
    tertiary-fixed: '#d6e3ff'               # mode-agnostic
    tertiary-fixed-dim: '#b6c7e8'           # mode-agnostic
    on-tertiary-fixed: '#091c35'            # mode-agnostic
    on-tertiary-fixed-variant: '#374763'    # mode-agnostic
    background: '#faf6ec'                   # alias of surface
    on-background: '#1d1c14'                # alias of on-surface
    surface-variant: '#ede2c6'              # warm beige variant
typography:
  display-xl:
    fontFamily: Noto Serif
    fontSize: 64px
    fontWeight: '700'
    lineHeight: '1.1'
    letterSpacing: -0.02em
  headline-lg:
    fontFamily: Noto Serif
    fontSize: 40px
    fontWeight: '600'
    lineHeight: '1.2'
    letterSpacing: -0.01em
  headline-md:
    fontFamily: Noto Serif
    fontSize: 32px
    fontWeight: '500'
    lineHeight: '1.3'
  body-lg:
    fontFamily: Noto Serif
    fontSize: 20px
    fontWeight: '400'
    lineHeight: '1.6'
  body-md:
    fontFamily: Noto Serif
    fontSize: 16px
    fontWeight: '400'
    lineHeight: '1.6'
  label-caps:
    fontFamily: Inter
    fontSize: 12px
    fontWeight: '600'
    lineHeight: '1.0'
    letterSpacing: 0.1em
spacing:
  container-max: 1440px
  margin-edge: 4rem
  gutter: 2rem
  stack-xl: 8rem
  stack-lg: 4rem
  stack-md: 2rem
  stack-sm: 1rem
---

## Brand & Style

The design system is engineered for a high-end maritime editorial experience, evoking the atmosphere of a private lounge overlooking the Mediterranean. It balances the timeless authority of traditional print media with the fluid precision of modern digital interfaces.

The aesthetic is rooted in **Minimalism** with a focus on negative space and architectural alignment. It aims to evoke feelings of exclusivity, heritage, and quiet confidence. Every element is intentional, avoiding decorative clutter to allow the high-quality imagery and refined typography to command attention. The interaction model is deliberate and smooth, mirroring the calm of deep waters.

**The system supports two atmospheric modes — `dark` (default) and `light` — both expressing the same brand intent through different registers.** Dark mode evokes midnight at sea: deep charcoal surfaces, gold lanterns, the quiet of a yacht at anchor under stars. Light mode evokes Mediterranean dawn: warm ivory paper, gold sunlight on limestone, the calm of an early morning before the marina wakes. Both modes are first-class — neither is a compromise of the other.

## Mode Strategy

The two modes are switched via a `data-theme` attribute on the document root: `<html data-theme="dark">` or `<html data-theme="light">`. If the attribute is absent, the app falls back to the user's OS preference (`prefers-color-scheme`).

**Persistence**: user choice is stored in `localStorage` under the key `yachtbay.theme`. A no-op script in the document head reads it before paint to avoid flash-of-wrong-theme on SSR boots.

**Default**: `dark`. New users without a preference and without a system signal see the dark editorial-luxury surface first — it is the brand's signature register.

**Toggle placement**: a single button in the panel's user menu and the storefront's footer. No prominent above-the-fold toggle — mode is preference, not a primary action.

## Colors

### Dark mode (default)

The palette is defined by a high-contrast relationship between shadow and light. The primary background uses a deep charcoal (#121414), providing a rich, immersive canvas. The primary accent is a refined, metallic gold (#D4AF37 → #F2CA50 brightened for OLED-friendly emission). This specific hue is used sparingly for critical interactive elements, active states, and premium iconography to signify value and focus. Secondary surfaces lighten in subtle tonal steps to create depth without breaking the dark-mode immersion. Typography uses warm ivory (#E2E2E2) for headlines and a muted cream-beige (#D0C5AF) for body copy to ensure legibility while maintaining the moody, editorial atmosphere.

### Light mode

The palette inverts atmosphere without losing identity. The primary background is a warm ivory (#FAF6EC) — evocative of aged book paper and Aegean limestone, not a sterile white. The gold accent shifts to a deeper amber tone (#735C00) to maintain accessible contrast on the cream canvas; the bright dark-mode gold remains available as `inverse-primary` so any dark-on-light components (snackbars, command palettes, inverse cards) still wear the signature brand color. Surface elevation works in reverse: rather than lightening tiers as in dark mode, light surfaces deepen into warm beige (#EBE3C8 → #E5DCBC) for raised containers. Typography uses a warm dark ink (#1D1C14) — slight olive undertone — rather than pure black, pairing with the cream paper instead of fighting it.

### The `-fixed` family — mode-agnostic

The `*-fixed` and `on-*-fixed` tokens (e.g., `primary-fixed`, `secondary-fixed-dim`) have identical values in both modes by design. They exist for components that must read identically regardless of theme — brand badges, fixed promotional surfaces, marketing assets that survive the mode toggle unchanged.

## Typography

Typography in this design system follows a strict editorial hierarchy. **Noto Serif** is the cornerstone of the identity, used for both headlines and body text to create a cohesive, literary feel. Headlines feature tight tracking and generous line heights to command the page, while body text is optimized for long-form readability against either surface.

A functional secondary typeface, **Inter**, is introduced exclusively for utility labels, navigation items, and small metadata. This sans-serif contrast ensures that functional UI elements remain distinct from editorial content. All labels should utilize wide letter-spacing and uppercase styling to enhance their "instrumental" feel against the classic serif backdrop.

**Default font per app**: the public **storefront** defaults to `font-body` (Noto Serif) for editorial weight on every consumer surface. The **panel** defaults to `font-label` (Inter) for data-row scannability — serifs at small sizes hurt scan speed in dense tables. Display moments inside the panel (page titles, dashboard headers) still use Noto Serif for editorial cadence.

## Layout & Spacing

The layout utilizes a **Fixed Grid** model inspired by premium broadsheet layouts. A 12-column grid provides the framework, but the design system prioritizes large, asymmetrical margins (4rem minimum) to create a sense of "gallery space" around the content.

Vertical rhythm is governed by a generous stacking scale. Significant sections are separated by 8rem (`stack-xl`) gaps to allow the eye to rest. Content clusters use a 2rem (`stack-md`) rhythm. Gutters are kept wide at 2rem to ensure that even dense data tables feel airy and accessible. Alignment should lean towards centered or left-justified with significant whitespace on the right to maintain the editorial cadence.

Spacing tokens are mode-agnostic — the editorial cadence holds whether the canvas is dawn or midnight.

## Elevation & Depth

In this design system, depth is communicated through **Tonal Layers** and **Low-contrast Outlines** rather than traditional shadows. The mechanism inverts by mode:

- **Dark mode** — elevation *lightens*. Base surface is the deepest tone (`surface-container-lowest: #0C0F0F`); raised containers step toward warmer charcoals (`surface-container-high: #282A2B`). A 1px hairline border at ~10% white opacity reinforces the boundary.
- **Light mode** — elevation *deepens*. Base surface is the lightest cream (`surface-container-lowest: #FFFFFF`); raised containers step into warmer beige (`surface-container-high: #EBE3C8`). A 1px hairline border in warm grey (`outline-variant: #D0C5AF`) reinforces the boundary.

**Overlay surfaces** (modals, menus, command palettes) use a subtle backdrop blur (10px) regardless of mode, maintaining a sense of translucency and place.

Avoid heavy drop shadows. If a shadow is required for extreme contrast, it should be a sharp, low-opacity tint (navy in dark mode, warm grey in light mode) rather than a soft black blur, maintaining the crispness of the editorial style.

## Shapes

The shape language of this design system is **Sharp (0)**. To reflect the precision of maritime engineering and the classic nature of editorial print, UI elements utilize 0px border radii in both modes.

Buttons, input fields, image containers, and cards should all feature perfectly square corners. This architectural rigidity provides a sophisticated contrast to the organic forms found in yacht photography and the fluid nature of the typography. The only exception to this rule is iconography, which may feature internal curves, but all containing boxes must remain sharp.

## Components

### Buttons

- **Dark mode** — Primary buttons are solid blocks of bright gold (`primary: #F2CA50`) with very dark amber text (`on-primary: #3C2F00`). Secondary buttons use a 1px gold outline with transparent backgrounds.
- **Light mode** — Primary buttons are solid blocks of deep gold (`primary: #735C00`) with white text (`on-primary: #FFFFFF`). Secondary buttons use a 1px deep-gold outline with transparent backgrounds.

All buttons are sharp-edged and feature a subtle color shift on hover (toward `primary-container` for tonal hover states).

### Input Fields

Inputs are defined by a bottom-border only (hairline width) in `outline-variant`. When active, the border transitions to `primary`. Labels sit above the input in the sans-serif `label-caps` style. The active-border tone matches mode: bright gold on dark, deep gold on light.

### Cards

Cards are background-tinted by elevation tier (`surface-container` for resting, `surface-container-high` for elevated), with a 1px subtle border in `outline-variant` to define boundaries. Image-heavy cards bleed to the edge of their containers, maintaining the sharp-corner aesthetic.

### Icons

Icons must be thin-stroke (1px to 1.5px) and monochromatic. Use `primary` for active navigation icons and `on-surface` for decorative or informational icons. In dark mode the primary icon is bright gold; in light mode it is deep gold — both read as "the brand accent."

### Navigation

The main navigation uses the uppercase Inter label style. The "Active" state is indicated by a simple 2px underline in `primary` or a `primary` text color change. Avoid bulky background highlights for navigation items. The underline tone matches mode automatically via the semantic token.
