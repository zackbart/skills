# DESIGN ETHOS
## VOL. 3: SAAS & PRODUCT PATTERNS
*Onboarding, Navigation, Data Density, Component Patterns, and Platform Standards*

> **Source:** Derived from Linear Method, Stripe Dashboard architecture, Atlassian Design System, Radix UI primitives, Google Material Design 3, and Mobbin's real-world UI flow library.

---

## From Principles to Product

Volumes 1 and 2 covered the "why" (psychology, heuristics) and the "how" (visual language, interaction). This volume covers the "what" — the specific component patterns, screen-level decisions, and product design conventions that the best SaaS products in the world have converged on.

These are not arbitrary preferences. They are patterns that have been tested against millions of users, refined through iteration, and validated through business results. When you deviate from them, you should do so with intention and evidence — not because you wanted something different.

---

# PART I — Onboarding & First-Run Experience

Onboarding is the peak moment of your user's experience (see the Peak-End Rule, Vol. 1). A bad onboarding poisons everything that follows. The goal is to get users to their first "aha moment" as fast as possible.

---

## The Onboarding Principles

- **Blur, don't block.** Never gate every feature behind a mandatory tutorial. Show the product, then teach.
- **Defer optionality.** Profile setup, preferences, and customization should be optional in the first session.
- **Show progress.** A setup checklist with completion percentage keeps users moving and investing in the product.
- **Personalize immediately.** If you ask a question during signup (role, use case, team size), use that answer within 60 seconds to tailor the experience.
- **The "empty state" is the most critical design decision in your app.** It is where new users are most likely to abandon. Make it teach, not just display nothing.

> **Linear's model:** Linear starts you in your first project immediately — no wizard, no video tour. The product teaches through use. Every empty state has a shortcut and a context hint.

---

## The Empty State Formula

Every empty state in your application should follow this structure:

1. **Icon or illustration** — matching the section (not a generic "nothing here" image).
2. **A headline** — explaining what this section is for (not "No items found").
3. **A single action CTA** — the most natural first step from this state.
4. **Optional: one-line explanation** — why this space is empty and what will appear here.

> ❌ **Bad:** "No tasks found." — Tells the user nothing. Does nothing.
>
> ✅ **Good:** "Your sprint is empty. Add issues to start tracking progress. [+ Add Issue]" — Teaches, enables, invites.

---

## Sign-Up Flow Standards

| Step | Standard |
|------|----------|
| Email / SSO entry | One field, one button. No name, no phone number, no role at this step. |
| Email verification | Magic link preferred over 6-digit code. If code, auto-advance on completion. |
| Profile setup | Name + optional avatar. Defer everything else. |
| Personalization survey | Maximum 3 questions. Skip option visible. Use answers to customize UI immediately. |
| First meaningful action | Land the user directly in the product, on a task they can complete in under 2 minutes. |

---

# PART II — Navigation Architecture

Navigation is the skeleton of your application. If it's wrong, no amount of great UI on individual pages can save the experience. Get it right before designing anything else.

---

## The Navigation Hierarchy

| Navigation Level | Pattern & Standard |
|-----------------|-------------------|
| Global / Primary | Left sidebar (SaaS/desktop) or top navigation bar. Max 6–8 items. Always visible. |
| Secondary / Contextual | Sub-navigation within a section. Tabs or nested sidebar. Max 5 items. |
| Tertiary / Page-level | Anchor links, breadcrumbs, section tabs. Used for content-heavy single pages. |
| Utility | Search, notifications, profile/settings. Always top-right corner. Not part of primary nav. |

---

## Command Palette — The Power User's Interface

The command palette (Cmd+K / Ctrl+K) is now a table-stakes feature for any professional SaaS product. It is the single most important keyboard interaction you can add.

- Open with `Cmd+K` (Mac) / `Ctrl+K` (Windows/Linux). This is the universal shortcut.
- Search scope: all actions, all navigation items, recent items, and search-as-you-type results.
- Show keyboard shortcuts next to each command result.
- Group results: Recent, Actions, Navigation, Search Results.
- Close with `Escape`. Always. No exceptions.

> **The Linear standard:** In Linear, you almost never need the mouse. The command palette is how you navigate, create, assign, and transition everything. Design your app so power users can operate entirely from the keyboard.

---

## Sidebar Navigation Patterns

### The Inverted-L Chrome (Linear's Pattern)

The most effective SaaS navigation pattern: a vertical sidebar (left) plus a horizontal header. The sidebar controls the main content view. This is Linear's core chrome — called the "inverted-L shape."

- Sidebar width: 220–260px expanded, 56–64px collapsed (icons only).
- Group related items with headers, not dividers. Dividers suggest separation; headers suggest organization.
- Active state: filled background (subtle), left accent border, or font weight change. Never all three.
- Collapse button: visible at the bottom of the sidebar, not hidden in a hover menu.
- Mobile: sidebar becomes a bottom tab bar (max 5 items) or a hamburger drawer.

---

# PART III — Component Patterns

## Forms

Forms are where user effort is highest and abandonment risk is greatest. Every form decision is a retention decision.

### Input Fields

- Label above the field. Placeholder text is not a label.
- Placeholder should show an example, not repeat the label.
- Input height: 36–40px (compact), 44–48px (comfortable). Never below 36px.
- Border: 1px solid on default state. 2px solid accent on focus. 1px solid red on error.
- Group related fields visually. Name + Last Name share a row. Address is its own group.
- Required fields: mark **optional**, not required. Most fields in a well-designed form are required.
- Tab order must follow reading order. Always. Test this explicitly.

### Buttons

| Button Type | Use & Style |
|------------|-------------|
| **Primary** | One per view. The single most important action. Filled, accent color. |
| **Secondary** | Supporting actions. Outlined or ghost style. Same visual weight as primary on hover. |
| **Tertiary / Ghost** | Low-emphasis actions. Text only or very subtle border. Never competes with primary. |
| **Destructive** | Delete, remove, cancel subscription. Red, but never as prominent as primary. Often secondary style. |
| **Icon Button** | Must have a tooltip. Never icon-only for a primary action. |
| **Loading State** | Replace label with spinner. Disable the button. Keep the button at the same size. |

- Button padding: 12–16px horizontal, 8–10px vertical for a standard button. 8–12px and 6–8px for compact.
- Never use all-caps for button labels in product UI. Title case is the standard.
- Never use a button color that requires the user to read the label to know if it's primary or destructive.

---

## Data Tables

Data tables are where SaaS products live or die. The Stripe Dashboard and Atlassian are the gold standards for data-dense tables that don't feel overwhelming.

- Row height: 40–48px for comfortable density. 32px for compact (power users, dense data).
- Column headers: left-aligned text, right-aligned numbers. Always.
- Hover state: subtle row highlight (3–5% opacity background change).
- Sort indicator: small arrow, appears on hover, fills on click. Direction is obvious.
- Inline actions (edit, delete, view): appear on hover, right side of row. Don't take column space.
- Bulk actions: appear above the table when rows are selected. Replace the table header.
- Empty table: use the empty state formula from Part I.
- Loading state: skeleton rows (3–5 rows visible), matching the height and structure of real rows.

> **Stripe's rule:** Every row of data should tell a complete story. The user should be able to read any single row in isolation and understand it. No field should require context from another row.

---

## Modals & Dialogs

### When to Use

- Use for: confirmations, quick edits, focused data entry that requires context from the underlying view.
- Do **not** use for: full workflows, complex forms with 5+ fields, content that needs to be bookmarked.
- If the modal content would scroll, convert it to a full page or side panel.

### Modal Standards

- Max width: 480px (small/confirmation), 640px (standard form), 800px (content-heavy).
- Close on: X button (top right), Escape key, overlay click (only for non-destructive modals).
- Focus trap: keyboard navigation must be contained within the modal while it is open.
- Background: 40–60% dark overlay (`rgba(0,0,0,0.5)`). Blur optional, but adds perceived depth.
- Animation: fade in (200ms ease-out) + subtle scale up (scale from 0.96 → 1.0). Scale down on close.

---

## Toast Notifications

| Type | Use & Behavior |
|------|---------------|
| **Success** | Confirm completed action. Bottom center or bottom right. Auto-dismiss: 3–4s. |
| **Error** | Persistent. User must dismiss. Include action link if there is a resolution step. |
| **Warning** | Alert to a risky state. Auto-dismiss: 5s. Include action if applicable. |
| **Info** | Neutral update. Auto-dismiss: 4s. Optional dismiss button. |
| **Loading** | Ongoing process (background export, etc.). Progress indicator. Manual dismiss or auto on complete. |

- Stack toasts vertically — never overlap. Maximum 3 visible at once.
- Always provide an Undo option for success toasts on destructive actions.
- Position: bottom-right (desktop), bottom-center (mobile).

---

# PART IV — Platform Standards

## Web / SaaS Application

- Keyboard shortcuts for all primary actions. Document them. Show them in tooltips and in a help modal (`?` key convention).
- URL state: every view, filter, and sort should be reflected in the URL. Users should be able to share links to any state.
- Infinite scroll vs. pagination: pagination for content the user needs to reference; infinite scroll for feeds.
- Responsive breakpoints: 320px (mobile), 768px (tablet), 1024px (desktop), 1440px (wide). Design all four.
- Focus states: visible, high-contrast focus rings on all interactive elements. Never remove `outline` without an alternative.

---

## Mobile Web & Native

- Touch targets: minimum 44×44pt. Bigger for primary actions.
- Bottom navigation: primary actions must be reachable with one thumb.
- Swipe gestures: use system conventions. Left swipe to delete is a pattern. Don't invent new gestures.
- Haptic feedback: use for confirmations, selections, and errors. Match the device's haptic vocabulary.
- Keyboard avoidance: form views must shift content up when the keyboard appears — never overlap the active input.

---

## Radix UI — The Primitive Foundation

Radix UI is the open-source primitive library that underpins many of the best SaaS UIs built today (including shadcn/ui). Understanding its design philosophy is key to using it well.

- Unstyled by default — bring your own visual design, get accessible behavior for free.
- Every component handles: focus management, keyboard navigation, ARIA attributes, and screen reader support.
- Use Radix primitives for: Dialogs, Popovers, Tooltips, Dropdowns, Sliders, Tabs, Accordions.
- Don't fight the primitives. They encode years of accessibility expertise. Override styles, not behavior.

> **The rule:** If you're building a component that Radix already covers, use Radix. Accessibility is not a nice-to-have — it is a core product requirement.

---

## Material Design 3 — Adaptive & Fluid UI

Google's Material Design 3 introduces "adaptive" and "fluid" UI — interfaces that respond meaningfully to device type, screen size, and user preference, not just resizing.

- **Dynamic Color:** Derive accent palette from user's wallpaper/preference. Useful for consumer apps.
- **Motion:** Material's emphasis curves (standard, decelerate, accelerate) are the right model for functional animation.
- **Elevation:** In dark mode, elevated surfaces use lighter fills (tonal elevation), not shadows.
- **Component tokens:** Every Material component exposes design tokens for color, shape, and motion. Use the token system, not hardcoded values.

---

# The Complete SaaS UI Checklist

Use this before every screen review. If you can't check every box, the screen is not ready to ship.

| Category | Checklist Item |
|----------|---------------|
| **Status** | Every async action has a loading state, success state, and error state. |
| **Status** | Loading states use skeleton screens, not spinners, for content areas. |
| **Errors** | All error messages are human sentences with a suggested next step. |
| **Errors** | Form validation is inline and real-time, not on submit only. |
| **Navigation** | The user always knows where they are in the application. |
| **Navigation** | Every view has a sensible, shareable URL. |
| **Keyboard** | All primary actions are keyboard accessible and have documented shortcuts. |
| **Keyboard** | Focus order follows the reading/task order on every screen. |
| **Empty States** | Every empty state has a heading, a CTA, and optionally a brief explanation. |
| **Data** | Every table row tells a complete story without needing adjacent rows. |
| **Modals** | All modals close on Escape. Focus is trapped inside while open. |
| **Dark Mode** | All interactive states (hover, active, disabled, error) are tested in both modes. |
| **Accessibility** | Minimum 4.5:1 contrast ratio for all text (AA standard). |
| **Accessibility** | All icon-only buttons have a tooltip with the action label. |
| **Mobile** | All touch targets are minimum 44×44pt. |
| **Mobile** | The view is usable with one hand on a 390pt width screen. |
| **Performance** | The primary action on any screen responds within 400ms. |

---

*Design Ethos Vol. 3 · SaaS & Product Patterns · Source of Truth*
