# DESIGN ETHOS
## VOL. 1: CORE PRINCIPLES
*The Psychological & Heuristic Foundations*

> **Source:** Synthesized from Nielsen Norman Group (NN/g), Laws of UX by Jon Yablonski, Apple Human Interface Guidelines, and research-backed psychology of human-computer interaction.

---

## Why Principles Come Before Pixels

Before writing a single line of code or placing a single element in Figma, the best teams in the world start with shared principles. These aren't opinions — they're the distilled result of decades of research into how humans perceive, process, and interact with digital systems.

This document is your source of truth for the *why* behind every design decision. When you feel the pull to add another option, another animation, or another visual flourish, come back here first.

---

# PART I — Nielsen's 10 Usability Heuristics

First published in 1990 and refined in 1994, these are the closest thing design has to physical laws. They have not changed in 30+ years because human cognition has not changed. Apply these as a mandatory checklist before shipping any screen.

---

## 1. Visibility of System Status

The design must always keep users informed about what is going on, through appropriate feedback within a reasonable time.

- Always show loading states, progress indicators, and success/error confirmations.
- Users should never wonder if their action was registered.
- Progress feedback under 0.1s feels instant. Under 1s maintains flow. Beyond 10s, users disengage.

> **Applied:** A button that changes to a spinner on click, then a checkmark on success, satisfies this heuristic completely.

---

## 2. Match Between System and the Real World

Speak the user's language. Use words, phrases, and concepts familiar to the user — not system-oriented jargon. Follow real-world conventions.

- Labels should reflect how your users describe their own work, not how the database is structured.
- Icons should map to universal mental models (trash = delete, envelope = messages).

---

## 3. User Control and Freedom

Users often choose functions by mistake. They need a clearly marked "emergency exit" — undo, back, cancel — without going through an extended process.

- Every destructive action must be reversible or require confirmation.
- Never trap a user in a flow with no escape route.
- Cmd+Z / Ctrl+Z is a fundamental expectation in any application.

---

## 4. Consistency and Standards

Users should not have to wonder whether different words, situations, or actions mean the same thing. Follow platform and industry conventions.

- A "primary action" button should always look the same across your app.
- Navigation placement should be consistent across all views.
- **Jakob's Law:** Users spend most of their time on other products. Your UI should work like the patterns they already know.

---

## 5. Error Prevention

Even better than good error messages is a design that prevents problems from occurring in the first place. Eliminate error-prone conditions, or check for them and present users with a confirmation option.

- Disable submit buttons until required fields are filled.
- Validate form inputs inline and in real-time, not just on submit.
- Present confirmation dialogs before irreversible actions.

---

## 6. Recognition Rather Than Recall

Minimize the user's memory load. Make options, actions, and objects visible. Users should not have to remember information between steps.

- Keep navigation visible — don't hide it behind gestures or icons users may not discover.
- Autocomplete and suggestions reduce the burden of recall.
- Tooltips on hover for icon-only buttons bridge the recognition gap.

---

## 7. Flexibility and Efficiency of Use

Accelerators — unseen by novice users — allow expert users to speed up interactions. The design should serve both.

- Keyboard shortcuts for all primary actions (Linear's model: every action reachable without a mouse).
- Command palette / global search (Cmd+K) is the gold standard for power users.
- Allow personalization: custom views, saved filters, pinned items.

---

## 8. Aesthetic and Minimalist Design

Every extra unit of information in a dialog competes with the relevant information and diminishes its relative visibility. Interfaces should contain no irrelevant or rarely needed information.

- Every element on a screen should earn its place.
- If you can't articulate why something is there, remove it.
- White space is not empty space — it is breathing room that directs attention.

> **Linear's Rule:** Remove everything that doesn't serve the core task. Then remove everything that only sometimes serves it. What remains is the interface.

---

## 9. Help Users Recognize, Diagnose, and Recover From Errors

Error messages should be expressed in plain language (no error codes), precisely indicate the problem, and constructively suggest a solution.

- Never show raw API errors or stack traces to end users.
- Write error messages as sentences: "Your password must be at least 8 characters." Not: "ERR_PWD_LENGTH".
- Always offer a next step: "Try again," "Go back," "Contact support."

---

## 10. Help and Documentation

Even though it is better if the system can be used without documentation, it may be necessary to provide help. Such help should be easy to search, focused on the user's task, and include concrete steps.

- Inline contextual help is better than a separate help center.
- Empty states are an opportunity to teach, not just display a blank screen.
- Onboarding tooltips and checklists reduce the need for external docs.

---

# PART II — The Laws of UX

These are the psychological principles — validated by research and behavioral science — that govern how users actually make decisions, perceive information, and interact with interfaces. Compiled by Jon Yablonski.

---

## Fitts's Law

The time to acquire a target is a function of the distance to it and its size. Bigger and closer = faster and fewer errors.

- Primary actions (CTA buttons, submit, confirm) must be large and positioned where the user's attention already is.
- Touch targets on mobile should be at minimum 44×44pt (Apple) or 48×48dp (Google).
- Destructive buttons (delete, remove) should be small and distant from the primary flow.

> **Rule:** The most used action deserves the most real estate. Design the hit target around the task, not the visual element.

---

## Hick's Law

The time to make a decision increases with the number and complexity of choices. More choices = slower decisions = more frustration.

- Limit primary navigation to 5–7 items maximum.
- Progressive disclosure: reveal complexity only when the user asks for it.
- Pricing pages with 3 options convert better than those with 5.
- Reduce options to the critical path. Everything else is a secondary menu.

---

## Jakob's Law

Users spend most of their time on other products. This means they expect your product to work the same way as the ones they already know.

- Place logos top-left. Navigation top or left sidebar. Search top center.
- Convention is not a creativity failure — it's a gift to your users.
- When you deviate from convention, the deviation must be obviously better, not just different.

---

## Miller's Law

The average person can hold approximately 7 (±2) items in working memory at one time.

- Group related information into chunks of no more than 7 items.
- Navigation menus, option lists, form sections — all should be chunked.
- When a list exceeds ~7 items, provide search/filter.

---

## Doherty Threshold

Productivity soars when a computer and its users interact at a pace under 400ms. Delays beyond this break flow, frustration compounds.

- Provide UI feedback (button press state, skeleton screens) within 100–200ms — even if data takes longer.
- Use optimistic UI: show the expected result immediately, then confirm with the server.
- Animations should run at 60fps. Anything less feels broken.
- The 400ms rule applies to micro-interactions. For loading: show a skeleton, not a blank screen.

---

## The Aesthetic-Usability Effect

Users perceive aesthetically pleasing designs as more usable — even when they aren't. Appearance influences perception of quality.

- Beautiful design earns patience and trust that functional design alone cannot.
- This is not an excuse to sacrifice function for form — it's a reminder that they are not opposites.
- Linear, Stripe, and Vercel are beautiful because their visual language is a direct expression of their function.

---

## The Peak-End Rule

People judge an experience based on how they felt at its peak moment and at its end, not the average across the whole experience.

- Nail the most important moment in your flow (the "aha" moment) and the final confirmation screen.
- Celebrate completions: the confetti on Duolingo, the checkmark on Linear.
- A painful onboarding poisons the whole experience even if everything else is smooth.

---

## Von Restorff Effect (Isolation Effect)

When multiple similar objects are present, the one that differs from the rest is most likely to be remembered.

- Use accent color for the single most important action on any given screen.
- Do not have two "primary" buttons — one must be visually dominant.
- Highlight the recommended pricing plan. Break the visual pattern to direct the eye.

---

# PART III — Apple's Core Design Philosophy

The Apple Human Interface Guidelines represent over 45 years of refined thinking about human-computer interaction. These four principles provide the framework for every design decision — including situations the guidelines don't explicitly address.

| Principle | What It Means for Your Work |
|-----------|----------------------------|
| **Clarity** | Every element should be immediately understandable. Legible text, precise icons, purposeful space. Never sacrifice readability for aesthetics. |
| **Deference** | Your UI should never compete with the content. Elements exist to enable the content, not to impress the viewer. Get out of the way. |
| **Depth** | Hierarchy, layering, and motion create a sense of place and navigation. Users should always know where they are and how they got there. |
| **Consistency** | Leverage familiar patterns. Behaviors and visual conventions should match user expectations — within your app and across the platform. |

---

## Dark Mode as a First-Class Citizen

Nearly 82% of smartphone users prefer dark mode (Earthweb, 2024). This is not a secondary feature — it is a primary design surface.

- Use semantic colors that adapt automatically — never hardcode light-mode colors.
- Dark mode surfaces are not just "inverted light mode." They use elevated surfaces, not shadows, to create depth.
- True black (#000000) causes halation on OLED screens. Use deep grays (#0D0D0D, #1A1A1A).
- Increase luminance slightly for interactive elements in dark mode to maintain visibility.
- Test every state — hover, active, disabled, error — in both modes before shipping.

> **The Linear Rule:** Professional software lives in dark mode. Design dark-first, then ensure the light mode is equally intentional.

---

# Quick Reference: The Non-Negotiables

These are the rules derived from all of the above. Violating them is not a stylistic choice — it is a user experience defect.

| Rule | Source |
|------|--------|
| Always show system status within 400ms | Heuristic #1 + Doherty Threshold |
| Every destructive action is reversible or confirmed | Heuristic #3 |
| Limit primary choices to ≤7 items | Hick's Law + Miller's Law |
| Primary CTAs are large, proximate, and singular | Fitts's Law + Von Restorff Effect |
| Follow platform navigation conventions | Jakob's Law + Heuristic #4 |
| Validate inline, in real-time | Heuristic #5 (Error Prevention) |
| Keyboard shortcuts for all primary actions | Heuristic #7 |
| Every element earns its place — or is removed | Heuristic #8 (Minimalism) |
| Error messages are human sentences with next steps | Heuristic #9 |
| Design dark-first with semantic colors | Apple HIG Dark Mode |
| Nail the peak moment and the ending | Peak-End Rule |

---

*Design Ethos Vol. 1 · Core Principles · Source of Truth*
