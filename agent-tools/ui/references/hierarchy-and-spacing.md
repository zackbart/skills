# Visual Hierarchy and Spacing

## Visual Hierarchy

Visual hierarchy is the single most effective tool for making an interface feel "designed."
When everything competes for attention, the result is noisy and chaotic. The fix is almost
always to de-emphasize, not add more emphasis.

### Text Hierarchy With Weight and Color

Font size alone is a blunt tool -- it leads to primary content that's too large and secondary
content too small to read. Instead, combine three levers:

**Color (primary lever):**
- Dark color (e.g., `hsl(0, 0%, 10%)`) for primary content -- headlines, key data
- Medium grey (e.g., `hsl(0, 0%, 45%)`) for secondary content -- dates, metadata
- Light grey (e.g., `hsl(0, 0%, 65%)`) for tertiary content -- footnotes, captions

**Weight:**
- Normal: 400 or 500 (depends on the typeface) for most text
- Bold: 600 or 700 for emphasized text
- Never use weights below 400 for UI. If you need to de-emphasize, use lighter color or smaller size instead -- thin weights are unreadable at small sizes.

**Size (supporting lever):**
- Use sparingly. The type scale gives you enough range.
- Small size differences (14px vs 16px) are subtle. Color and weight differences are more impactful.

### Emphasize by De-Emphasizing

When a primary element doesn't stand out, don't pump it up further. Instead, tone down
its competitors:

- Active nav item doesn't pop? Give inactive items a softer color so they recede.
- Sidebar competes with main content? Remove the sidebar's background color so it sits directly on the page background.
- A heading gets lost? De-emphasize the paragraph below it, not just the heading.

This principle applies at every scale -- from a single component to an entire page layout.

### Labels and Data Display

The default `Label: Value` format gives everything equal emphasis and kills hierarchy.

**When to skip the label entirely:**
- The format is self-evident: `janedoe@example.com` (email), `(555) 765-4321` (phone), `$19.99` (price)
- Context already tells the user what the data means: "Customer Support" under a name = department

**When labels are needed, combine them into prose:**
- "12 left in stock" instead of "In stock: 12"
- "3 bedrooms" instead of "Bedrooms: 3"

**When you must use label:value format:**
- Dashboards with lots of scannable, similar data
- Treat the label as supporting content -- smaller, lighter color, lighter weight
- Exception: on information-dense spec pages where users scan for the label word itself (e.g., "depth" on a phone spec page), emphasize the label slightly and de-emphasize the value

### Document Hierarchy vs. Visual Hierarchy

HTML heading tags (`h1`, `h2`, etc.) are for semantics and accessibility. They should not
dictate visual styling. A section title in an app UI often acts like a label -- it should
be small and supportive, not large and dominant.

Style headings for visual hierarchy independently of their semantic level. Sometimes a
section title should be visually hidden entirely if the content speaks for itself.

### Button Hierarchy

Every page has a hierarchy of actions: one primary, a few secondary, a few tertiary.

| Level | Style | Example |
|-------|-------|---------|
| Primary | Solid, high-contrast background | "Save changes", "Place order" |
| Secondary | Outline or muted background | "Cancel", "Export" |
| Tertiary | Link-style, no border/background | "Learn more", "Skip" |

**Destructive actions:** Being destructive does NOT mean big, red, and bold. A "Delete"
button in a settings list should be secondary or tertiary. Reserve the prominent red
treatment for the confirmation dialog where "Delete" IS the primary action.

### Balancing Weight and Contrast

Elements that cover more surface area (bold text, solid icons, thick borders) feel heavier
and draw more attention. Use contrast to compensate:

- **Icons** next to text feel over-emphasized because they cover more area. Lower the icon's
  contrast (softer color) to balance.
- **Thin borders** (1px, soft color) can feel too subtle. Don't just darken them (harsh).
  Increase the border width instead -- adds emphasis without losing the soft quality.

---

## Spacing

### The Spacing Scale

A linear scale (multiples of 4px) doesn't help you choose between 120px and 125px. You need
a scale where adjacent values are at least ~25% apart -- tight at small sizes, spreading out
at large sizes.

**Recommended scale:**
```
4px  8px  12px  16px  24px  32px  48px  64px  96px  128px  192px  256px
```

Base unit: 16px (matches default browser font size, divides cleanly).

**How to use:**
1. Start with a gut feeling (e.g., "this padding should be about 16px").
2. Try the values on either side of your guess (12px and 24px).
3. Two of the three options will feel obviously wrong. Use the remaining one.
4. If the outer value wins, make it the new center and repeat.

### White Space Strategy

The common mistake is adding white space reactively -- only when things look cramped. This
produces interfaces with the bare minimum breathing room.

**Start with too much.** Way too much. Then selectively tighten until it feels right. The
result is interfaces that breathe and feel spacious, which always looks more professional
than cramped.

Dense UIs (dashboards, data tables) are valid -- but make density a deliberate choice, not
a default.

### Ambiguous Spacing

When using space (not borders) to group elements, the space within a group must be visibly
less than the space between groups:

**Form fields:** The gap between a label and its input must be clearly smaller than the gap
between one field group and the next. If they're equal, labels float between inputs and
users put data in the wrong field.

**Section headings:** Add more space above a heading than below it. Otherwise it appears to
belong to the previous section, not the following one.

**Bulleted lists:** The gap between bullets should be larger than the line-spacing within a
single multi-line bullet. Otherwise the list reads as one undifferentiated block.

### Screen Filling

Just because you have 1200-1400px available doesn't mean you should use it all.

- If 600px is right for your form, use 600px. Don't stretch it to fill.
- Give each element the width it needs. A form and a nav don't need to be the same width.
- Use `max-width` to prevent elements from growing past their optimal size. Let them shrink only when the viewport demands it.

**When content feels too narrow on a wide screen:** Split into columns rather than making
the element wider. Put supporting text in a side column. Add a related info panel. Don't
make the primary element worse just to fill horizontal space.

### Grids

Grid systems are useful but not sacred.

- **Fixed widths** are often better than percentage columns. A sidebar at 280px fixed width + flexible main area works better than 25%/75% fluid split (which makes the sidebar too wide on large screens, too narrow on small).
- **Don't use percentages** to size something unless you actually want it to scale with the viewport.
- An element that's 500px wide at its best shouldn't be forced into a 6-of-12 column at 50%. Give it `max-width: 500px` and let it be its natural size.

### Responsive Spacing

Large elements must shrink faster than small elements at smaller viewports. The visual
difference between large and small elements should be less extreme on mobile.

- Desktop heading: 48px. Mobile heading: 24px (50% reduction).
- Desktop body: 18px. Mobile body: 16px (11% reduction).
- The ratio between heading and body is completely different at each breakpoint. Don't encode it.

Fine-tune sizes independently per context. Let go of proportional scaling.
