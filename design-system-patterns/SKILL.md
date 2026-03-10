---
name: design-system-patterns
description: >
  A principled design system skill for building SaaS and product UI that matches
  the quality of Linear, Stripe, Notion, and Vercel. Use this skill whenever the
  user wants to: generate a UI spec for a screen or feature, audit existing UI
  against design principles, or asks questions like "how should I design X",
  "does this UI look right", "review my component", "generate a design spec",
  "what's wrong with this UI". Also triggers on: component audit, UI review,
  dark mode setup, SaaS dashboard design, onboarding flow design, navigation
  design, form design, empty state design. This skill grounds all output in three
  Design Ethos volumes covering core principles (Nielsen, Laws of UX, Apple HIG),
  visual language (typography, color, spacing, motion), and SaaS patterns
  (onboarding, navigation, components). Use it proactively — if the user is
  building or reviewing any kind of product UI, this skill should be active.
allowed-tools: "Read, Grep, Glob, AskUserQuestion"
---

# Design System Skill

This skill produces principled, production-ready design decisions grounded in three
reference volumes. It operates in two modes. Detect the correct mode from context —
don't ask unless genuinely ambiguous.

## Modes

### Mode 1: `/ui-spec` — UI Specification
**Trigger:** User describes a screen, feature, or component they need to build and wants a design spec before coding.

**What it does:** Outputs a structured spec covering layout decisions, component choices, spacing values, interaction states, dark/light variants, accessibility requirements, and implementation notes.

**Read before running:** `references/vol3-saas-patterns.md` (relevant component sections) + `references/vol2-visual-interaction.md` (layout patterns section)

---

### Mode 2: `/ui-audit` — Design Audit
**Trigger:** User shares existing UI (component code, screenshot description, Figma spec, or live URL) and wants a review.

**What it does:** Returns a structured critique organized by violation severity, each mapped to a specific principle in the ethos volumes, with concrete fixes.

**Read before running:** All three reference volumes — load them in order (Vol 1 → 2 → 3).

---

## Mode 1: UI Specification — Full Protocol

### Step 1: Clarify the Screen

If the user's description is vague, ask only what's essential:
- What is the primary task the user is trying to accomplish on this screen?
- What data or content does it need to display?
- Any known constraints (existing component library, screen size priority)?

Don't ask more than 3 questions. Make reasonable assumptions for everything else.

### Step 2: Read Relevant Sections

From `references/vol3-saas-patterns.md`, read the sections relevant to the screen type:
- Dashboard/data screen → Data Tables section
- Form/input screen → Forms section
- Onboarding screen → Part I (Onboarding)
- Navigation structure → Part II (Navigation Architecture)
- Modal/overlay → Modals & Dialogs section

From `references/vol2-visual-interaction.md`:
- Layout Patterns section for overall structure
- Micro-Interactions section for all interactive elements

### Step 3: Output the Spec

Structure the spec as follows:

```
## Screen: [Name]

### Intent
One sentence: what the user accomplishes here.

### Layout
- Overall pattern (e.g., "inverted-L chrome: left sidebar + main content area")
- Content width, max-width, padding
- Responsive behavior at mobile breakpoint

### Components Required
For each component:
- Name & variant
- Key props/states (default, hover, active, disabled, loading, error)
- Spacing (internal padding, gap to adjacent elements)
- Dark/light behavior

### Interaction States
- Loading state behavior
- Empty state (using the empty state formula)
- Error state
- Success/completion state

### Keyboard & Accessibility
- Tab order
- Required keyboard shortcuts
- ARIA requirements

### Anti-Patterns to Avoid
Specific to this screen type, from the ethos docs.

### Pre-Ship Checklist
Relevant items from the Vol. 3 checklist for this screen type.
```

---

## Mode 3: UI Audit — Full Protocol

### Step 1: Load All Three Reference Volumes

Read `references/vol1-core-principles.md`, `references/vol2-visual-interaction.md`, and `references/vol3-saas-patterns.md` before producing any audit output.

### Step 2: Evaluate Against All Layers

Check systematically across:

**Layer 1 — Heuristic Violations (Vol. 1)**
- Visibility of system status
- Error prevention & recovery
- Consistency & standards
- Recognition vs. recall
- Aesthetic minimalism

**Layer 2 — Visual & Interaction (Vol. 2)**
- Animation timing (are transitions within spec?)
- Color system integrity (single accent? semantic colors correct?)
- Typography hierarchy (correct scale? no sub-13px text?)
- Spacing consistency (4pt grid violations?)
- Dark mode correctness

**Layer 3 — Product Patterns (Vol. 3)**
- Component conformance (buttons, forms, tables, modals to spec?)
- Empty states present and correct?
- Loading/error/success states covered?
- Keyboard accessibility?
- Mobile touch targets?

### Step 3: Output the Audit

```
## UI Audit: [Component/Screen Name]

### Critical Issues (ship-blocking)
[Issue]: [What's wrong]
[Principle]: Vol. X — [Specific rule violated]
[Fix]: [Concrete, actionable correction]

### Significant Issues (fix before next release)
[Same format]

### Minor Issues (polish pass)
[Same format]

### Passing
[What's already correct — be specific, not just "looks good"]

### Priority Fix Order
1. [Most impactful fix]
2. ...
```

Never produce a vague audit. Every issue must name the specific principle it violates and give a concrete fix.

---

## Anti-Patterns Reference

These are explicit negatives — things that should never appear in output regardless of mode. Memorize these.

**Color**
- Never true black (#000000) or true white (#ffffff) as surface or text colors
- Never more than one accent color for interactive elements
- Never gray text on a colored background (contrast fails)
- Never hardcoded color values — always tokens

**Typography**
- Never body text below 13px
- Never all-caps in product UI buttons or labels
- Never more than 3 font weights in a single view
- Never a font size below the defined scale (no one-off sizes)

**Spacing**
- Never spacing values that aren't multiples of 4
- Never different padding values on the same component type across views
- Never divider lines where space would suffice

**Interaction**
- Never an animation above 400ms on a frequently-used interaction
- Never bounce/elastic easing on functional UI elements
- Never an action with no feedback state
- Never a destructive action without confirmation or undo

**Components**
- Never two primary buttons on the same screen
- Never an icon-only button without a tooltip
- Never a form that only validates on submit
- Never an empty state that just says "No items found"
- Never a modal that doesn't close on Escape

**Layout**
- Never cards nested inside cards
- Never horizontal scroll on mobile (outside intentional carousels)
- Never more than 7 items in a primary navigation

---

## Relationship to Other Skills

This skill operates **above** `frontend-design`. Use it to:
1. Generate a spec for what to build (this skill)
2. Hand the spec to `frontend-design` or your coding agent for implementation

If Impeccable is installed (`.claude/commands/` contains `/audit`, `/polish`, etc.), run
this skill's `/ui-spec` first to establish intent, then use Impeccable's commands to
enforce quality at the code level. They are sequential layers, not competitors.

---

## Reference Files

| File | When to Read |
|------|-------------|
| `references/vol1-core-principles.md` | Audit mode (Layer 1); any time a principle needs citing |
| `references/vol2-visual-interaction.md` | Spec mode (visual decisions, layout patterns); audit mode (Layer 2) |
| `references/vol3-saas-patterns.md` | Spec mode (component patterns); audit mode (Layer 3) |

Read only what's needed. Don't load all three for a simple spec question.
