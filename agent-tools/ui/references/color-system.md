# Color System

## Use HSL, Not Hex

Hex and RGB are not intuitive -- colors that look similar visually look nothing alike in
code. HSL maps directly to human color perception:

- **Hue** (0-360deg): Position on the color wheel. 0deg = red, 120deg = green, 240deg = blue.
- **Saturation** (0-100%): 0% = grey (no color), 100% = vivid/intense.
- **Lightness** (0-100%): 0% = pure black, 50% = pure color, 100% = pure white.

**HSL vs HSB:** Design tools (Figma, Sketch) use HSB, not HSL. In HSB, 100% brightness
at 100% saturation gives a vivid color (equivalent to 50% lightness in HSL). Always
convert to HSL for CSS -- browsers only understand HSL.

## Building a Full Palette

A 5-color palette from a generator is not enough for real UI work. You need three
categories of color, each with multiple shades.

### Category 1: Greys (8-10 Shades)

Greys make up the bulk of any interface -- text, backgrounds, panels, borders, form controls.

**Never use true black or true white.** True black (`#000`) looks unnatural against any
background. Start with a very dark grey and work up to near-white.

**Saturate your greys.** Zero-saturation grey looks lifeless. Real-world "greys" have a
color temperature:
- Cool greys: add a hint of blue saturation
- Warm greys: add a hint of yellow or orange saturation

Increase saturation slightly on the lightest and darkest shades to maintain consistent
temperature across the full range.

### Category 2: Primary Colors (5-10 Shades Each)

One or two primary colors for: primary buttons, active navigation, links, focus rings,
overall brand identity.

- Ultra-light shades (100-200): tinted backgrounds for alerts, highlights, badges
- Mid shades (400-600): buttons, links, active states
- Dark shades (800-900): text on tinted backgrounds

### Category 3: Accent Colors (5-10 Shades Each)

Semantic and categorical colors:

| Purpose | Color |
|---------|-------|
| Destructive/danger | Red |
| Warning | Yellow/amber |
| Success/positive | Green |
| Info/neutral | Blue (if not primary) |
| Highlight/new feature | Teal, pink, or another eye-grabbing hue |

Plus any colors needed for categorical distinction: graph lines, calendar events, tags,
status badges. Complex UIs may need up to **10 different colors x 5-10 shades each**.

## Defining Shades

**Never generate shades on the fly** with `lighten()`, `darken()`, or opacity tricks.
This produces dozens of nearly-identical shades with no system. Define shades upfront.

### The 9-Shade Process

Label shades 100 (lightest) through 900 (darkest). Base color = 500.

**Step 1: Pick the base (500).**
Choose a shade that works well as a button background. Don't follow a formula like
"start at 50% lightness" -- every hue behaves differently. Trust your eyes.

**Step 2: Pick the edges.**
- **900 (darkest):** Should work as text color. Test it on a white background.
- **100 (lightest):** Should work as a subtle tinted background. Test it as a background
  for an alert or badge.

A good test: create an alert component with your 900 as text and 100 as background. Both
should look right simultaneously.

**Step 3: Fill in the gaps.**
- Pick 700 and 300 first (midpoints of the two large gaps)
- Then fill 800, 600, 400, 200

**Step 4: Refine.**
Once you start using shades in real designs, tweak saturation and lightness as needed.
But resist adding new shades outside the system. The discipline of limiting yourself to
9 options is what makes the system useful.

## Advanced Color Techniques

### Compensating for Lightness Wash-Out

In HSL, saturation has less visual impact as lightness approaches 0% or 100%. A shade at
90% lightness with the same saturation as one at 50% lightness will look washed out.

**Fix:** Increase saturation proportionally as lightness moves away from 50%.

If the base color is already at 100% saturation and can't go higher, use hue rotation
instead (see below).

### Perceived Brightness and Hue Rotation

Not all hues are equally bright at the same HSL lightness. Yellow appears much brighter
than blue at identical lightness values.

**Perceived brightness peaks and troughs:**
- Brightest hues: 60deg (yellow), 180deg (cyan), 300deg (magenta)
- Darkest hues: 0deg (red), 120deg (green), 240deg (blue)

**Using this to build better shade scales:**

To make a shade appear lighter without washing it out: rotate the hue toward the nearest
bright hue (60, 180, or 300deg).

To make a shade appear darker without losing richness: rotate the hue toward the nearest
dark hue (0, 120, or 240deg).

**Example:** Building a yellow shade scale. As you decrease lightness for darker shades,
also rotate the hue toward orange (toward 0deg/red). The darker shades feel warm and rich
instead of dull brown.

**Limit:** Don't rotate more than 20-30deg or the shade will look like a completely
different color rather than a lighter/darker variant.

Combine hue rotation with lightness adjustment for the best results.

### Text on Colored Backgrounds

Grey text on colored backgrounds looks faded and disabled. White text at reduced opacity
looks washed out and lets the background bleed through.

**The right approach:** Hand-pick a text color based on the background:
1. Start with the background's hue
2. Adjust saturation and lightness until contrast is right
3. The result reduces contrast without the faded look that opacity or grey produces

For de-emphasized text on colored backgrounds, use a lighter version of the background
color, not grey or semi-transparent white.

## Accessibility

### Contrast Requirements (WCAG)

| Text Type | Minimum Contrast Ratio |
|-----------|----------------------|
| Normal text (under ~18px) | 4.5:1 |
| Large text (18px+ bold, or 24px+) | 3:1 |

### Technique: Flip the Contrast

White text on a dark colored background requires the background to be very dark to meet
4.5:1, which creates heavy visual weight and unintended hierarchy.

**Alternative:** Use dark colored text on a light colored background of the same hue. The
color still communicates meaning without dominating the visual hierarchy.

### Technique: Hue Rotation for Accessible Colored Text

When lightening colored text on a colored background to meet contrast requirements,
simply increasing lightness pushes the text toward white, losing its color.

**Fix:** Rotate the hue toward a brighter color (cyan, magenta, or yellow) to increase
perceived brightness while preserving the color identity.

### Never Rely on Color Alone

Color-blind users (~8% of men) cannot distinguish certain colors reliably. Red-green
color blindness is most common.

**Always pair color with another signal:**
- Positive/negative metrics: add up/down arrow icons alongside green/red
- Graph lines: use contrast (light vs dark values) not just different hues
- Status indicators: add icons, text labels, or shape differences
- Form validation: pair red borders with error text and an icon

**Principle:** Color should reinforce something the design already communicates. It should
never be the sole channel of information.
