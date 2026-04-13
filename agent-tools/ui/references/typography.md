# Typography

## Type Scale

Most UIs use too many font sizes -- every pixel from 10px to 24px ends up somewhere.
This creates inconsistency and slows every decision.

### Why Not Modular Scales

Ratio-based scales (golden ratio, perfect fifth, etc.) have two problems:
1. They produce fractional values (31.25px, 39.063px, 48.828px). Browsers handle subpixel
   rounding differently, so text renders inconsistently.
2. They don't produce enough intermediate sizes for UI work. A 3:4 scale gives 12, 16, 21,
   28 -- you'll constantly wish for sizes between 12-16 and 16-21.

### Recommended Hand-Crafted Scale

```
12px  14px  16px  18px  20px  24px  30px  36px  48px  60px  72px
```

This gives tight spacing at the small end (where 2px matters) and wider gaps at the large
end (where 6-12px between sizes is appropriate). Every value is a whole number.

**Use `px` or `rem` units only.** Never `em` -- nesting breaks the scale. An element at
`1.25em` (20px) with a child at `0.875em` computes to 17.5px, which isn't in any scale.

## Font Selection

### Safe Defaults

For UI design, use a neutral sans-serif. The system font stack gives you something every
user already recognizes:

```css
font-family: -apple-system, Segoe UI, Roboto, Noto Sans, Ubuntu, Cantarell, Helvetica Neue, sans-serif;
```

### Choosing a Custom Font

**Filter by weight variety.** Typefaces with 5+ weights tend to be crafted with more care.
On Google Fonts, filter by 10+ styles (weights + italics) -- this eliminates ~85% of options,
leaving fewer than 50 sans-serifs worth evaluating.

**Check legibility characteristics:**
- Fonts designed for body text: wider letter-spacing, taller x-height (the height of lowercase letters). Good for UI.
- Fonts designed for headlines: tighter letter-spacing, shorter x-height. Good for marketing pages, bad for body text.
- Avoid condensed typefaces with short x-heights for any text users need to read at length.

**Use popularity as a signal.** Popular fonts are popular because they're versatile and
well-designed. Sort by popularity in font directories to limit choices. Also inspect
typography on well-designed sites you admire.

### Personality Through Font Choice

| Goal | Font Style |
|------|-----------|
| Elegant, classic | Serif (e.g., Playfair Display, Lora) |
| Playful, friendly | Rounded sans-serif (e.g., Nunito, Varela Round) |
| Neutral, professional | Standard sans-serif (e.g., Inter, system fonts) |

When using a neutral font, rely on other elements (color, spacing, border radius, copy tone)
to carry the personality.

## Line Length

**Optimal: 45-75 characters per line.** Going slightly wider than 75 can occasionally work
but is risky.

**CSS implementation:** Set `max-width: 20-35em` on text containers. This keeps line length
proportional to font size.

Don't fit text to your layout. Fit the layout to the text. When mixing paragraphs with
images or wide components, still constrain paragraph width even if the content area is
wider. Different widths in the same content area looks intentional and polished.

## Line-Height

The common advice "use 1.5" is an oversimplification. Line-height depends on two factors:

### By Font Size (Inversely Proportional)

| Text Type | Font Size | Line-Height |
|-----------|-----------|-------------|
| Large headings | 36px+ | 1 - 1.25 |
| Subheadings | 24-30px | 1.25 - 1.4 |
| Body text | 14-18px | 1.5 - 1.75 |
| Small/caption text | 12-14px | 1.6 - 1.8 |

Small text needs more line-height because eyes have harder time finding the next line
after a wrap. Large headings need less because the lines are short and easy to track.

### By Line Width

- Narrow content (sidebar, card): line-height ~1.5
- Standard width (article): line-height ~1.5 - 1.65
- Wide content (full-width layout): line-height up to 2

Wider text means more horizontal eye travel to get back to the start of the next line.
More vertical space between lines helps the eye find its place.

## Alignment

### Text Alignment

**Default: left-align everything** (for left-to-right languages).

**Center alignment is fine for:**
- Headlines
- Short independent text blocks (hero taglines, card descriptions)
- Anything under 2-3 lines

**Never center text longer than 2-3 lines.** If a centered block is too long, the fix is
to rewrite the content shorter, not to keep it centered.

### Number Alignment in Tables

**Right-align numbers in tables.** When decimal points line up in the same column position,
values are instantly comparable. Left-aligned numbers make comparison slow and error-prone.

### Justified Text

Justified text can look polished in print-style contexts (magazines, formal content). But
it creates awkward gaps between words.

**If you use justified text, always enable hyphenation:**
```css
text-align: justify;
hyphens: auto;
```

Even in formal contexts, left-aligned text works perfectly well. Justified is a preference,
never a requirement.

### Baseline Alignment

When mixing font sizes on a single line (e.g., a large title with a small action link),
**align by baseline** -- the imaginary line letters rest on -- not by vertical center.

Vertical centering offsets the baselines, which looks awkward when text elements are close
together. Baseline alignment matches the reference line your eyes already perceive.

**CSS:** Use `align-items: baseline` on flex containers with mixed-size text.

## Letter-Spacing

### Default Rule

Trust the typeface designer. Leave letter-spacing alone for most text.

### Exception 1: Tighten Headlines

Body-text fonts (Open Sans, Inter, etc.) have wider built-in letter-spacing optimized for
small sizes. When these fonts are used at headline sizes (36px+), the spacing looks too
loose.

**Apply negative letter-spacing to large headings:** `-0.02em` to `-0.04em` typically works.

Don't try the reverse -- headline fonts with tight spacing rarely work at small sizes even
with increased letter-spacing.

### Exception 2: All-Caps Text

Normal letter-spacing is optimized for sentence case, which has high visual variety
(ascenders, descenders, mixed heights). All-caps text has low visual variety -- every letter
is the same height.

**Increase letter-spacing on all-caps text:** `0.05em` to `0.1em` improves readability
significantly.

```css
.all-caps {
  text-transform: uppercase;
  letter-spacing: 0.05em;
}
```

## Links

### In Content

Links embedded in paragraphs of non-link text need a distinctive treatment -- color,
underline, or both -- to look clickable.

### In Interfaces

When most things on a page are already interactive (navigation, dashboards), heavy link
styling is overbearing. Use subtler emphasis:
- Heavier font weight
- Darker color
- Underline on hover only

For ancillary links (footer, secondary nav) that aren't on the main user path, show the
link treatment only on hover. They remain discoverable without competing for attention.
