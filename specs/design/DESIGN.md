---
name: Editorial Luxury
colors:
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

## Colors

The palette is defined by a high-contrast relationship between shadow and light. The primary background uses a deep Aegean navy (#0d1c32), providing a rich, immersive canvas that feels more sophisticated than pure black. 

The primary accent is a refined, metallic gold (#D4AF37). This specific hue is used sparingly for critical interactive elements, active states, and premium iconography to signify value and focus. Secondary surfaces utilize slightly lighter navy tones to create subtle depth without breaking the dark-mode immersion. Typography primarily uses pure white for headlines and a muted, cool grey for body copy to ensure legibility while maintaining the moody, editorial atmosphere.

## Typography

Typography in this design system follows a strict editorial hierarchy. **Noto Serif** is the cornerstone of the identity, used for both headlines and body text to create a cohesive, literary feel. Headlines feature tight tracking and generous line heights to command the page, while body text is optimized for long-form readability against the dark background.

A functional secondary typeface, **Inter**, is introduced exclusively for utility labels, navigation items, and small metadata. This sans-serif contrast ensures that functional UI elements remain distinct from editorial content. All labels should utilize wide letter-spacing and uppercase styling to enhance their "instrumental" feel against the classic serif backdrop.

## Layout & Spacing

The layout utilizes a **Fixed Grid** model inspired by premium broadsheet layouts. A 12-column grid provides the framework, but the design system prioritizes large, asymmetrical margins (4rem minimum) to create a sense of "gallery space" around the content. 

Vertical rhythm is governed by a generous stacking scale. Significant sections are separated by 8rem (stack-xl) gaps to allow the eye to rest. Content clusters use a 2rem (stack-md) rhythm. Gutters are kept wide at 2rem to ensure that even dense data tables feel airy and accessible. Alignment should lean towards centered or left-justified with significant whitespace on the right to maintain the editorial cadence.

## Elevation & Depth

In this design system, depth is communicated through **Tonal Layers** and **Low-contrast Outlines** rather than traditional shadows. Because the background is a deep navy, elevation is achieved by lightening the surface color of the container.

1.  **Base:** Aegean Navy (#0d1c32).
2.  **Raised:** Secondary Navy (#1a2b45) with a 1px hairline border in a 10% white opacity.
3.  **Overlay:** Used for modals or menus, utilizing a subtle backdrop blur (10px) to maintain a sense of translucency and place.

Avoid heavy drop shadows. If a shadow is required for extreme contrast, it should be a sharp, low-opacity navy tint rather than a soft black blur, maintaining the crispness of the editorial style.

## Shapes

The shape language of this design system is **Sharp (0)**. To reflect the precision of maritime engineering and the classic nature of editorial print, UI elements utilize 0px border radii. 

Buttons, input fields, image containers, and cards should all feature perfectly square corners. This architectural rigidity provides a sophisticated contrast to the organic forms found in yacht photography and the fluid nature of the typography. The only exception to this rule is iconography, which may feature internal curves, but all containing boxes must remain sharp.

## Components

### Buttons
Primary buttons are solid blocks of Refined Gold (#D4AF37) with black or deep navy text for maximum contrast. Secondary buttons use a 1px gold outline with transparent backgrounds. All buttons are sharp-edged and feature a subtle color shift on hover to a slightly brighter gold.

### Input Fields
Inputs are defined by a bottom-border only (hairline width) in a muted grey. When active, the border transitions to Refined Gold. Labels sit above the input in the sans-serif label style.

### Cards
Cards are background-less by default, using only white-space and a 1px subtle navy border to define boundaries. Image-heavy cards should bleed to the edge of their containers, maintaining the sharp-corner aesthetic.

### Icons
Icons must be thin-stroke (1px to 1.5px) and monochromatic. Use Refined Gold for active navigation icons and white for decorative or informational icons.

### Navigation
The main navigation uses the uppercase Inter label style. The "Active" state is indicated by a simple 2px gold underline or a gold text color change. Avoid bulky background highlights for navigation items.