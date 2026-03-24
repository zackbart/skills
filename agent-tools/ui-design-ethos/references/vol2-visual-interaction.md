# DESIGN ETHOS
## VOL. 2: VISUAL LANGUAGE & INTERACTION
*Micro-interactions, Motion, Typography, Color, and the Linear Aesthetic*

> **Source:** Derived from Linear's design method, Stripe Dashboard architecture, Vercel design engineering principles, Airbnb's visual language system, Uber's motion design, and Apple HIG.

---

## The Visual Language Philosophy

The best interfaces in the world — Linear, Stripe, Vercel, Airbnb — don't look good by accident. They look good because every visual decision is a direct expression of the product's function. Aesthetic is not decoration. It is communication.

This document defines the visual and interaction standards that govern your interfaces. These are the rules for how things look, how they move, and how they respond.

> **From Linear:** Linear's design language is "professional to engineers" — dark, precise, high-contrast, and keyboard-native. Every visual choice communicates: *this is a tool built for serious work.*

---

# PART I — Micro-Interactions

Micro-interactions are the single-purpose, task-based feedback moments that separate a product that feels alive from one that feels dead. They are not decoration — they are a core communication system.

Dan Saffer's model: every micro-interaction has a **Trigger**, **Rules**, **Feedback**, and **Loops/Modes**. Design all four, not just the visual effect.

---

## The Four Rules of Micro-Interactions

### 1. Immediate Feedback (The 400ms Rule)

Every user action must be acknowledged within 400ms. This is the Doherty Threshold — the boundary between "feels instant" and "feels broken."

- Button press states should respond in 100–200ms — before the server responds.
- Use skeleton screens for content loading, not blank white space.
- Optimistic UI: show the expected result immediately, reconcile with reality asynchronously.
- Form field validation happens on blur and on keystroke — never only on submit.

> **The rule:** If the user can't feel that their action was received within 400ms, the interface is broken — regardless of what the server is doing.

---

### 2. Animation Timing

| Interaction Type | Timing & Easing |
|-----------------|-----------------|
| Hover state change | 150ms — ease-out |
| Button press / click | 100–150ms — ease-in-out |
| Toggle / switch | 200ms — ease-in-out |
| Modal open | 200–250ms — ease-out |
| Modal close | 150–200ms — ease-in |
| Page / view transition | 250–350ms — ease-in-out |
| Destructive action confirmation | 350–400ms — slightly slow (builds intention) |
| Success / completion celebration | 600–800ms — spring or bounce |
| Skeleton to content | 200ms — fade, ease-out |

The rule: frequent, small interactions should be fast (100–200ms). Significant or infrequent interactions can be slightly slower (300–400ms) to signal their weight. Anything decorative that exceeds 600ms will feel slow after the 10th use.

---

### 3. Design for Repeated Use

A micro-interaction that delights on first use but annoys on the 100th is a failure. Every interaction must survive the long-term relationship.

- Avoid motion that cannot be disabled for users with motion sensitivity (`prefers-reduced-motion`).
- Celebration animations (confetti, sparkles) should be reserved for genuinely significant moments — not every save.
- Sounds should always be opt-in, never default-on.

---

### 4. Purpose Before Delight

Every micro-interaction must serve a functional purpose. If you cannot articulate what information it communicates, remove it.

- A button spinner communicates: "your request is being processed."
- A red shake animation communicates: "that input is invalid."
- A green checkmark communicates: "that action succeeded."
- A blob flying across the screen communicates: nothing. Remove it.

---

## Micro-Interaction Patterns Library

### Form Validation
- Show requirements as the user types — check marks appear as each condition is met.
- Error state: field border turns red, error message appears below in 160ms.
- Success state: subtle green border or checkmark — never as prominent as an error.
- Never clear a field on error. The user's effort is valuable.

### Loading & Async States
- **0–400ms:** No indicator needed (action feels instant).
- **400ms–1s:** Subtle spinner or pulse animation on the triggering element.
- **1s–10s:** Skeleton screen or progress bar with estimated time.
- **10s+:** Explicit message + estimated time + option to cancel.

### Success & Completion
- Inline success: replace the action element with a brief confirmation, then restore.
- Page-level success: toast notification, bottom of screen, auto-dismisses in 3–4s.
- Major milestone: celebrate intentionally — but once, and only once.

### Destructive Actions
- **Single-step delete:** Undo option appears for 4–5s after the action. No modal required.
- **Irreversible delete:** Two-step confirmation. First click reveals "Are you sure?" Second click executes.
- **Mass destructive action** (delete all, clear history): Type the word to confirm pattern.

---

# PART II — The Visual Language System

## Typography

Typography is the largest part of most interfaces. Getting it right is getting the interface right.

### The Type Scale

| Role | Spec |
|------|------|
| Display / Hero | 48–64px, Bold, -0.02em tracking, tight leading (1.1) |
| Page Heading (H1) | 32–40px, Bold, -0.01em tracking, 1.2 leading |
| Section Heading (H2) | 24–28px, Semibold, -0.01em tracking, 1.3 leading |
| Subsection (H3) | 18–20px, Semibold, normal tracking, 1.4 leading |
| Body / Primary | 15–16px, Regular, normal tracking, 1.6 leading |
| UI Labels / Secondary | 13–14px, Medium, +0.01em tracking, 1.4 leading |
| Captions / Meta | 11–12px, Regular, +0.02em tracking, 1.4 leading |
| Code / Mono | 13–14px, Mono font, 1.6 leading |

- Font choice: Inter (web, SaaS), SF Pro (Apple native), `system-ui` for maximum performance.
- Never set body text smaller than 13px. Never.
- Tight tracking (-0.02em) on large headings makes them feel polished and intentional.
- Line-height 1.6 for body text is the comfortable reading standard. 1.4 for UI labels.

### Font Weight as Communication

Weight conveys hierarchy. Use it sparingly and intentionally — typically 2–3 weights maximum.

- **Bold (700):** Primary headings, labels on active states, critical numbers.
- **Semibold (600):** Secondary headings, interactive labels, important metadata.
- **Medium (500):** UI element labels, navigation items.
- **Regular (400):** All body text, descriptions, supporting content.

> **The rule:** If everything is bold, nothing is. Reserve bold for the 3–5 pieces of information the user must see first on any given screen.

---

## Color System

### The Color Architecture

Every color in your system should have a defined role. If it doesn't have a role, it doesn't belong in the system.

| Color Role | Usage |
|-----------|-------|
| Background (Base) | The deepest surface. Usually #0D0D0D dark, #FFFFFF light. |
| Surface (Elevated) | Cards, panels, modals. 1 step lighter than base. |
| Surface (Overlay) | Dropdowns, tooltips. 2 steps lighter than base. |
| Border / Divider | Subtle separation. Use sparingly — white space is better. |
| Text (Primary) | Main content. High contrast. Never pure black/white. |
| Text (Secondary) | Supporting info, labels, timestamps. 50–60% opacity of primary. |
| Text (Tertiary) | Placeholders, disabled states. 30–40% opacity. |
| Brand / Accent | Primary actions, links, active states. ONE color only. |
| Semantic: Success | Green. Confirmations, complete states. |
| Semantic: Warning | Amber. Soft alerts, approaching limits. |
| Semantic: Error | Red. Validation failures, destructive states. |
| Semantic: Info | Blue. Neutral informational states. |

### The One Accent Rule

A single accent color is not a limitation — it is a superpower. When every interactive element shares the same accent, users learn immediately: this color means "you can act here."

- Using two accent colors for primary actions destroys the visual hierarchy.
- Tint/shade the accent for hover, active, and disabled states — don't introduce a second color.
- The accent should be visible on both dark and light surfaces.

### Dark Mode Color Rules

- True black backgrounds (#000) cause halation (glow around bright text) on OLED screens. Use #0D0D0D or #111111.
- Elevation in dark mode = lighter surface, not shadows. Shadows are invisible on dark.
- Increase saturation slightly for semantic colors in dark mode — they can look muddy otherwise.
- Use semantic color tokens (`--color-error`, `--color-success`) that map to different values per mode.
- Never use opacity to create dark mode variants of text colors — always define discrete dark values.

---

## Spacing & Layout

### The 4pt Grid

All spacing in your interface should be multiples of 4. This creates visual rhythm and makes every spacing decision effortless.

| Value | Use Case |
|-------|----------|
| 4px (1 unit) | Icon-to-label gap, minimum inset padding |
| 8px (2 units) | Component internal padding, gap between tight elements |
| 12px (3 units) | Card padding (compact), input padding |
| 16px (4 units) | Standard component padding, gap between related elements |
| 24px (6 units) | Gap between groups/sections within a view |
| 32px (8 units) | Card padding (comfortable), gap between distinct components |
| 48px (12 units) | Section breaks, feature separators |
| 64px (16 units) | Page-level section separation |

- When in doubt, use more space, not less. Density should be a deliberate choice.
- The Linear approach: very dense information architecture with precise 4pt increments — not generous padding, but perfectly consistent spacing.

### Visual Hierarchy Through Space

White space is not empty space. It directs attention, groups related elements, and separates distinct sections. A spacing violation is a hierarchy violation.

- Elements that are related should be closer together than elements that are not.
- The gap between sections should always be larger than the gap within sections.
- Don't use divider lines to separate content — use space. Lines are a crutch for insufficient space.

---

## Iconography

### Standards

- Use a single icon family across your entire product. Never mix styles.
- **Recommended:** Lucide React (MIT licensed, clean, consistent weight), Heroicons, Phosphor Icons.
- Icon size: 16px for inline/UI context, 20px for standard actions, 24px for standalone icons.
- Icon + label: always. Icon-only buttons require a tooltip. Never rely on icons alone for navigation.
- Stroke weight should match your typography weight. Light fonts = light icons. Bold fonts = heavier icons.

---

# PART III — Layout Patterns That Win

## The Bento Box Layout

The bento box is the defining layout trend for SaaS marketing pages, dashboards, and feature showcases. It uses a grid of variable-size cards to communicate multiple features simultaneously, without hierarchy forcing the user to scroll to discover value.

- Works best: Marketing/landing pages, feature showcases, dashboards, pricing tiers.
- Grid: CSS Grid with 4–12 columns. Cards span 1–4 columns and 1–3 rows.
- Key rule: vary the sizes dramatically. A grid of all equal cards is just a grid, not a bento.
- Each card should communicate a single, clear idea. No card should require reading to understand.

> **Reference:** [Bentogrids.com](https://bentogrids.com) — study the size distribution and content density of the top examples. The best bentos are immediately readable at a glance.

---

## The Data-Dense Dashboard

The Stripe Dashboard is the gold standard for information-dense interfaces that don't feel cluttered. The secret: strong typographic hierarchy does all the heavy lifting.

- Lead with the number the user cares most about — large, prominent, top-left.
- Every chart needs a title, a timeframe, and a key number in the heading — the chart is the detail, not the story.
- Use color encoding consistently: green = positive/growth, red = decline/error, gray = neutral.
- Avoid 3D charts, pie charts (for more than 3 segments), and decorative chart elements entirely.
- Ensure the page makes sense without any charts — the data labels and numbers should stand alone.

---

## The Editorial Layout

Clean, modern, content-forward. Used by Siteinspire's featured sites, Linear's marketing, and Vercel's documentation. Strong vertical rhythm, generous white space, centered or slightly off-center content columns.

- Max content width: 680–800px for reading, 1200px for full-page layouts.
- Generous padding: minimum 40px top/bottom on sections, 80–120px for hero sections.
- Typography does the design work. Images and graphics accent, not dominate.
- Navigation is minimal — typically a sticky top bar with logo + 3–5 items + CTA.

---

## Mobile-First Responsive Rules

- Design mobile layout first — it forces the critical hierarchy decisions.
- Touch targets: minimum 44×44pt regardless of visual size.
- Bottom navigation on mobile for primary actions — thumbs can't reach the top.
- Horizontal scrolling: never, except within an intentional carousel.
- Text that's legible on desktop may not be legible on mobile — test both explicitly.

---

# The Summary: What the Best Interfaces Have in Common

After studying Linear, Stripe, Vercel, Airbnb, and Apple, the patterns are clear. The interfaces that win share these characteristics:

- **Speed** above all else. Fast feedback, fast load, fast transitions. Speed is a feature.
- **Restraint.** They removed everything that didn't need to be there. The complexity of removal is where craft lives.
- **A single visual language.** One typeface. One accent color. One icon family. One spacing system. Consistency is trust.
- **Motion that informs, not impresses.** Every animation communicates a state change. Nothing moves just to move.
- **Dark-mode native.** Professional tools live in dark mode. Designed dark-first, not as an afterthought.
- **Keyboard-first power.** The mouse is for exploration. The keyboard is for work. Great tools serve both.

---

*Design Ethos Vol. 2 · Visual Language & Interaction · Source of Truth*
