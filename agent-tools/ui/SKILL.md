---
name: ui
description: >
  Practical UI design system for building interfaces that look professionally designed.
  Covers visual hierarchy, spacing, typography, color, shadows, and polish. Produces
  concrete CSS values and design decisions, not abstract theory. Use this skill whenever
  building, styling, or reviewing any user interface -- pages, components, forms, dashboards,
  landing pages, cards, navbars, modals, or any visual element. Also use when the user asks
  to "make it look good", "fix the styling", "improve the design", "add polish", or is
  working with CSS/Tailwind/styled-components. This skill should be consulted for every
  frontend implementation task, even when the user doesn't explicitly ask for design help --
  good defaults prevent bad UIs.
---

# UI Design System

This skill turns you into a designer who writes code. Every rule here is concrete and
actionable -- specific values, specific techniques, specific decisions. Follow these
when building any interface.

## Core Philosophy

1. **Start with the feature, not the layout.** Design the actual functionality first (a search form, a settings panel), not the shell around it. Navigation and chrome come after.
2. **Hierarchy is the single most effective tool.** When everything competes for attention, nothing wins. Deliberately de-emphasize secondary and tertiary content.
3. **Use systems, not ad-hoc values.** Define scales for spacing, type, color, and shadows. Never make the same minor decision twice.
4. **Start with too much white space, then remove.** What feels like "a little too much" for one element becomes "just enough" in a full UI.
5. **Color is an enhancement layer, not a crutch.** If your UI doesn't work in grayscale, color won't save it.

## Decision-Making Workflow

When building any UI element, work through these layers in order:

1. **Hierarchy** -- What's the most important thing? De-emphasize everything else.
2. **Spacing** -- Use the spacing scale. Start generous, tighten only if needed.
3. **Typography** -- Pick from the type scale. Set line-height proportionally.
4. **Color** -- Apply from the predefined palette. Never pick ad-hoc colors.
5. **Depth** -- Add shadows only where elevation is meaningful.
6. **Polish** -- Accent borders, custom bullets, background treatments.

Read the relevant reference file for the layer you're working on.

---

## Quick-Reference Rules

These are the highest-impact rules from each domain. For full detail, see the
reference files listed at the bottom.

### Hierarchy

- Use **font weight and color** to create hierarchy, not just font size.
- Three text colors: dark (primary), grey (secondary), lighter grey (tertiary).
- Two font weights: normal (400-500) and bold (600-700). Never use weights below 400.
- **Labels are a last resort.** If the format communicates meaning (`$19.99`, `janedoe@example.com`), skip the label. Combine labels into prose: "12 left in stock" not "In stock: 12".
- When the primary element doesn't pop, **de-emphasize competitors** instead of adding more emphasis.
- Button hierarchy: primary = solid high-contrast, secondary = outline/muted, tertiary = link-style. Destructive actions get secondary/tertiary treatment unless they're the primary action on a confirmation screen.

### Spacing System

Use a spacing scale where each step is ~25% larger than the previous. Base unit: `16px`.

```
4px  8px  12px  16px  24px  32px  48px  64px  96px  128px  192px  256px
```

- Adjacent values must be noticeably different (at least ~25% apart).
- More space **around** groups than **within** them. If label-to-input gap equals input-to-next-label gap, the label doesn't visually belong to its input.
- Don't fill the whole screen. If 600px is right, use 600px.
- Give elements fixed widths when they have an optimal size. Use `max-width`, not percentage columns.
- Start mobile-first (~400px canvas), then adapt for larger screens.

### Typography

Use a hand-crafted type scale (not a modular/ratio scale):

```
12px  14px  16px  18px  20px  24px  30px  36px  48px  60px  72px
```

- Line length: **45-75 characters** (CSS: `max-width: 20-35em`).
- Line-height scales inversely with font size:
  - Body text (14-18px): `1.5` to `1.75`
  - Large headings (36px+): `1` to `1.25`
  - Wide content needs taller line-height (up to `2`)
- Use `px` or `rem`, never `em` (nesting breaks the scale).
- Mixed font sizes on one line: align by **baseline**, not vertical center.
- System font stack for safe defaults: `-apple-system, Segoe UI, Roboto, Noto Sans, Ubuntu, Cantarell, Helvetica Neue, sans-serif`
- Tighten letter-spacing on large headings. Increase letter-spacing on all-caps text.

### Color System

Build palettes using HSL. Each color needs **9 shades** (100-900):

- **Greys:** 8-10 shades. Never use true black (#000) or true white (#fff). Saturate greys slightly for warmth (yellow/orange) or coolness (blue).
- **Primary:** 5-10 shades. Base (500) should work as a button background. Light shades (100-200) for tinted backgrounds. Dark shades (800-900) for text.
- **Accents:** Red (destructive), yellow (warning), green (success), plus any brand accents. Multiple shades each.

Key color rules:
- As lightness moves away from 50%, **increase saturation** to compensate for wash-out.
- To lighten: rotate hue toward 60deg (yellow), 180deg (cyan), or 300deg (magenta). To darken: rotate toward 0deg (red), 120deg (green), or 240deg (blue). Limit rotation to 20-30deg.
- On colored backgrounds, **hand-pick text color** by adjusting the background's hue/saturation/lightness. Never use grey text or white-at-reduced-opacity on colored backgrounds.
- WCAG: **4.5:1** contrast for text under 18px. **3:1** for large text.
- Never rely on color alone -- always pair with icons, labels, or contrast differences.

### Shadows and Depth

Define **5 shadow levels** for your elevation system:

```css
--shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
--shadow-md: 0 2px 4px rgba(0,0,0,0.06), 0 1px 2px rgba(0,0,0,0.04);
--shadow-lg: 0 4px 8px rgba(0,0,0,0.07), 0 2px 4px rgba(0,0,0,0.04);
--shadow-xl: 0 8px 16px rgba(0,0,0,0.08), 0 4px 8px rgba(0,0,0,0.03);
--shadow-2xl: 0 16px 32px rgba(0,0,0,0.1), 0 8px 16px rgba(0,0,0,0.02);
```

- Buttons: `sm`. Dropdowns: `lg`. Modals: `2xl`.
- Use **two-part shadows** (large soft + tight dark) for realism.
- Light comes from above: raised elements get a lighter top edge and a small shadow below. Inset elements (inputs, wells) get a lighter bottom edge and a dark inset shadow at top.
- On click/press, reduce or remove the shadow (element "pushes in").
- Flat-design depth: lighter = closer, darker = farther. Use solid shadows (zero blur) for flat aesthetic.

### Polish and Finishing

- Replace default bullets with icons (checkmarks, arrows, domain-specific).
- Add **accent borders** (top of cards, side of alerts, under headings, top of page).
- Background gradients: two hues **no more than 30deg apart**.
- Empty states are a user's first impression -- add illustrations, emphasize the CTA, hide UI that doesn't work yet.
- **Use fewer borders.** Alternatives: box shadows, different background colors, or extra spacing.
- Custom-style checkboxes/radios with brand colors.
- Don't default to boring component shapes -- dropdowns can have sections and icons, tables can have images and hierarchy, radio buttons can be selectable cards.

### Responsive Design

- **Large elements shrink faster than small ones.** A 45px desktop heading becomes 20-24px on mobile, not a proportional 35px.
- Don't encode proportional relationships with `em`. Fine-tune sizes independently per breakpoint.
- Give components `max-width` instead of percentage-based columns. Only shrink when the viewport actually demands it.
- Button padding is **not proportional** -- more generous at large sizes, tighter at small sizes.

---

## Personality Quick-Check

Before building, know the personality:

| Element | Elegant | Playful | Neutral |
|---------|---------|---------|---------|
| Font | Serif | Rounded sans | Neutral sans |
| Border radius | 0 | Large (12-16px) | Small (4-6px) |
| Color | Gold, muted tones | Pink, bright | Blue, grey |
| Copy tone | Formal | Casual, humor | Professional |

Pick one lane and stay consistent. Mixing square and rounded corners in the same interface almost always looks worse than committing to one.

---

## Anti-Patterns (Never Do These)

- Never use font weights below 400 for UI text
- Never grey text on colored backgrounds (hand-pick the color instead)
- Never true black or true white as surface/text colors
- Never more than 2-3 font sizes visible on a single screen without hierarchy
- Never labels when the data format is self-evident
- Never spacing values outside your scale
- Never percentage-based widths unless you want the element to scale
- Never scale up icons past their intended size (16-24px) -- enclose in a shape instead
- Never scale down screenshots -- crop or simplify instead
- Never design the navigation before the features
- Never a border where spacing or background color would work
- Never an empty state that just says "No items found"

---

## Reference Files

| File | When to Read |
|------|-------------|
| `references/hierarchy-and-spacing.md` | Building layouts, establishing visual hierarchy, spacing decisions |
| `references/typography.md` | Font selection, type scales, line-height, alignment, letter-spacing |
| `references/color-system.md` | Building palettes, defining shades, HSL techniques, accessibility |
| `references/depth-and-shadows.md` | Shadows, elevation, light simulation, flat-design depth, images |
| `references/polish-and-finishing.md` | Final touches, accent borders, empty states, backgrounds, component innovation |

Read only what's relevant to the current task. Don't load all five for a simple spacing question.
