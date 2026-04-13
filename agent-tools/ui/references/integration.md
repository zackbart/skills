# Design System Integration

This skill has strong opinions. When a project's existing tokens contradict the principles
here, the answer is to fix the tokens, not to inherit bad defaults. There are two modes:
**Fix** (existing project -- audit and correct) and **Establish** (new project -- build
tokens from scratch).

Every UI task in an existing project starts with the audit below. Even "just add a
button" inherits whatever's broken. Spend the minutes to check before building.

## Detection Workflow

Run through this checklist when first touching a project:

### 1. Check for Tailwind

```bash
ls tailwind.config.* 2>/dev/null
```

If present, read the file. Look for:
- `theme.extend.colors` -- existing color tokens
- `theme.extend.spacing` -- custom spacing scale
- `theme.extend.fontSize` -- type scale
- `theme.extend.boxShadow` -- shadow system
- `theme.extend.fontFamily` -- font stack

If empty/minimal, the project is using Tailwind defaults (which are actually quite good).

### 2. Check for CSS Custom Properties

Look at `globals.css`, `index.css`, `app.css`, `styles.css`:

```css
:root {
  --color-primary: ...;
  --spacing-4: ...;
  --shadow-md: ...;
}
```

If present, these are the tokens. Use `var(--*)` references, not raw values.

### 3. Check for shadcn/ui

Tells: `components/ui/` directory, `components.json` file, tailwind config with `hsl(var(--foreground))` patterns.

shadcn uses CSS custom properties for its design tokens, defined in `globals.css`. The
tokens are semantic (`--background`, `--foreground`, `--primary`, `--muted`, etc.) not
scale-based. Respect the semantic names.

### 4. Check for Theme Files

```bash
fd -e ts -e js "theme|tokens|design" src/
```

Component libraries (Mantine, Chakra, MUI, styled-components) expose a theme object through
a provider. The theme object is the source of truth.

### 5. If Nothing Found

The project has no design system. You're in **Establish mode** -- propose and create one.

---

## Mapping This Skill to Tailwind

### Spacing

Tailwind's default spacing scale is already close to what this skill recommends. It uses
a 4px base with natural progression:

```
0.5 (2px)  1 (4px)  2 (8px)  3 (12px)  4 (16px)  6 (24px)  8 (32px)
12 (48px)  16 (64px)  24 (96px)  32 (128px)  48 (192px)  64 (256px)
```

**Use the defaults.** Don't extend unless the project needs values between what's provided.
The most common addition: intermediate sizes for fine-tuning (`theme.extend.spacing.18`).

### Typography

Default Tailwind type scale:
```
text-xs (12px)  text-sm (14px)  text-base (16px)  text-lg (18px)
text-xl (20px)  text-2xl (24px)  text-3xl (30px)  text-4xl (36px)
text-5xl (48px)  text-6xl (60px)  text-7xl (72px)
```

This matches the skill's recommended scale exactly. Use it directly.

Tailwind's `leading-*` utilities pair with font sizes via a preset:
- `text-xs` → leading 1.33
- `text-sm` → leading 1.43
- `text-base` → leading 1.5
- `text-4xl+` → leading 1

Override with explicit `leading-*` utilities when a specific component needs different
line-height (e.g., `leading-tight` for headings at 1.25, `leading-relaxed` for wide body text).

### Colors

Tailwind's default palette ships with 11-shade scales (50, 100-900, 950) for each color.
This exceeds the skill's 9-shade minimum.

**Integration pattern:** Extend with brand colors matching the same shade structure.

```ts
// tailwind.config.ts
theme: {
  extend: {
    colors: {
      brand: {
        50:  'hsl(220, 100%, 97%)',
        100: 'hsl(220, 95%, 93%)',
        200: 'hsl(220, 90%, 85%)',
        300: 'hsl(220, 85%, 70%)',
        400: 'hsl(220, 80%, 60%)',
        500: 'hsl(220, 80%, 52%)',  // base
        600: 'hsl(220, 85%, 45%)',
        700: 'hsl(220, 90%, 38%)',
        800: 'hsl(220, 95%, 30%)',
        900: 'hsl(220, 100%, 22%)',
        950: 'hsl(220, 100%, 15%)',
      },
    },
  },
}
```

Build the scale using the 9-shade process in `color-system.md`, then expose 50 and 950 for
extreme cases.

### Shadows

Tailwind's default shadows are close to the skill's 5-level system but slightly muted:

| Tailwind | Skill Equivalent | Use |
|----------|------------------|-----|
| `shadow-sm` | sm | Buttons, subtle raise |
| `shadow` | md | Default cards |
| `shadow-md` | md | Hover states |
| `shadow-lg` | lg | Dropdowns, popovers |
| `shadow-xl` | xl | Sticky elements |
| `shadow-2xl` | 2xl | Modals, dialogs |

Use them as-is. Only extend if the project needs a two-part shadow for a specific effect.

---

## Mapping This Skill to shadcn/ui

shadcn uses semantic CSS variables, not scale-based. The defaults in `globals.css` look like:

```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  --muted: 210 40% 96.1%;
  --muted-foreground: 215.4 16.3% 46.9%;
  --accent: 210 40% 96.1%;
  --border: 214.3 31.8% 91.4%;
  /* ...etc */
}
```

### Integration Rules

- **Always use the semantic tokens.** `bg-primary` not `bg-blue-600`. `text-muted-foreground`
  not `text-gray-500`.
- **Modify tokens in `globals.css`**, not in components. Changing `--primary` changes every
  primary button at once.
- The HSL-space-separated format (`220 80% 52%`) is required so tokens can be consumed as
  `hsl(var(--primary) / 0.5)` for opacity.

### Auditing a shadcn System

Common gaps:
- **No primary scale.** shadcn ships one `--primary`, not a 9-shade scale. Add it to
  `globals.css` if the project needs variants (`--primary-50` through `--primary-950`).
- **Dark mode drift.** Verify `:root` and `.dark` define the same tokens. Missing tokens
  in one mode cause broken styles.
- **Hardcoded values in components.** Grep for `#[0-9a-f]{3,6}` and `bg-gray-` — these
  bypass the token system.

---

## Mapping This Skill to Plain CSS (CSS Custom Properties)

For vanilla projects or non-framework React, define tokens in `:root`:

```css
:root {
  /* Spacing */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  --space-12: 48px;
  --space-16: 64px;

  /* Type */
  --text-xs: 12px;
  --text-sm: 14px;
  --text-base: 16px;
  --text-lg: 18px;
  --text-xl: 20px;
  --text-2xl: 24px;
  --text-3xl: 30px;
  --text-4xl: 36px;

  /* Colors: use the 9-shade process from color-system.md */
  --color-primary-500: hsl(220, 80%, 52%);
  /* ... */

  /* Shadows */
  --shadow-sm: 0 1px 2px rgb(0 0 0 / 0.05);
  --shadow-md: 0 2px 4px rgb(0 0 0 / 0.06), 0 1px 2px rgb(0 0 0 / 0.04);
  --shadow-lg: 0 4px 8px rgb(0 0 0 / 0.07), 0 2px 4px rgb(0 0 0 / 0.04);
  --shadow-xl: 0 8px 16px rgb(0 0 0 / 0.08), 0 4px 8px rgb(0 0 0 / 0.03);
  --shadow-2xl: 0 16px 32px rgb(0 0 0 / 0.1), 0 8px 16px rgb(0 0 0 / 0.02);
}

@media (prefers-color-scheme: dark) {
  :root {
    /* Override tokens for dark mode */
  }
}
```

All components reference `var(--*)`. No raw values in component CSS.

---

## Mapping This Skill to Component Libraries

### Mantine / Chakra / MUI

These expose a theme object via a provider. Extend the theme, don't override components
directly.

```ts
// Mantine example
const theme = createTheme({
  colors: {
    brand: [
      '#eff4ff', '#dce5ff', '#b8cbff', '#8ba7ff',
      '#6384ff', '#4362ff', '#3051ff', '#2442e0',
      '#1c34b8', '#162a90',
    ],
  },
  primaryColor: 'brand',
  fontSizes: { xs: '12px', sm: '14px', md: '16px', lg: '18px', xl: '20px' },
  shadows: { /* ... */ },
});
```

Components consume via `color="brand"` or `size="md"` -- never hex values.

### styled-components / Emotion

Via `ThemeProvider`:

```ts
const theme = {
  colors: { /* ... */ },
  spacing: { /* ... */ },
  shadows: { /* ... */ },
};

// In components:
const Button = styled.button`
  padding: ${p => p.theme.spacing[4]};
  background: ${p => p.theme.colors.primary[500]};
`;
```

---

## The Audit (Mode 1 -- Always Run This First)

When a design system exists, audit it before building. The goal is to catch broken tokens
that would compromise whatever you're about to build. Run every category; most take
seconds.

### Spacing
- [ ] Scale has granularity at small sizes (4, 8, 12, 16, 24, 32)
- [ ] Adjacent values at least ~25% apart
- [ ] No one-off values in components (grep for `px-\[`, `padding:\s*\d+px` outside config)

### Type
- [ ] Font sizes defined as a scale, not ad-hoc
- [ ] Line-height tokens exist for at least 3 ranges (tight/normal/relaxed)
- [ ] Font weights limited to 2-3 options
- [ ] No sizes below 12px
- [ ] Type scale uses `px` or `rem`, never `em` (breaks on nesting)

### Color
- [ ] Each color has 5-10 shades, not just one (critical -- without shades you can't build
      alerts, hovers, or disabled states correctly)
- [ ] Greys have slight saturation (not pure `#888`)
- [ ] No true black (`#000`) or true white (`#fff`) in tokens
- [ ] Semantic tokens: danger, warning, success, info
- [ ] Contrast ratios meet WCAG (4.5:1 for text, 3:1 for large text)
- [ ] Dark mode tokens parallel light mode (same keys in both)

### Shadows
- [ ] 3-5 shadow levels defined
- [ ] Each level has a clear use case (buttons, cards, dropdowns, modals)
- [ ] Hover states use a different level than rest

### Violations Scan (find hardcoded values bypassing the system)
- [ ] `grep -rE '#[0-9a-fA-F]{3,6}' src/` -- should be near-zero in components
- [ ] Raw `px` in component styles outside config: low count
- [ ] Inline `style={{}}` with visual values: zero
- [ ] `color: rgb(` or `color: hsl(` in components: zero

### Report Structure

Produce findings with clear severity:

```
## [Project] Design System Audit

### Critical (blocks good UI -- fix before building)
- [Issue]: [principle violated]
  Found in: [file:line]
  Fix: [exact config change]

### Significant (worth fixing soon)
- [Same format]

### Minor (polish pass)
- [Same format]

### Strong points
- [What's already working -- be specific, not "looks good"]
```

Severity is not optional. Critical issues must be fixed before building new UI on top of
them; a "fix later" posture means every new feature bakes in the defect.

Every recommendation must include the exact config change -- `add these 4 shades to
theme.extend.colors.brand in tailwind.config.ts`, not `improve the color palette`.

---

## When Establishing a New System

No design system exists. Create one using this skill's defaults:

1. **Match the project's stack.** Tailwind project → extend the config. React project with
   CSS modules → add `:root` vars. Mantine → extend the theme.
2. **Start minimal.** Ship with greys, one primary, spacing, type scale, and 3-5 shadows.
   Add accent colors as features need them.
3. **Put tokens in one place.** Don't scatter them across multiple config files.
4. **Document the *why* in short comments above tokens.** The config file is the design
   system -- don't create parallel markdown documentation that will drift from code.
