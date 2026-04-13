# Depth, Shadows, and Images

## Light Source Simulation

All depth in UI follows one rule: **light comes from above.** Users look slightly downward
at screens. Every raised or inset element should be consistent with overhead lighting.

### Raised Elements (Buttons, Cards)

The top edge faces the light source and catches more light. The bottom casts a shadow on
the surface below.

**Implementation:**
- **Top edge:** Slightly lighter than the element face. Use a top border or an inset
  box-shadow with a slight positive vertical offset.
- **Bottom shadow:** Small, dark box-shadow with a slight positive vertical offset. Only
  visible below the element.
- **Blur radius:** Keep it small -- a couple of pixels. Raised objects in the real world
  (wall outlets, window frames) cast sharp-edged shadows.

**Hand-pick the lighter top-edge color.** Don't use semi-transparent white -- it desaturates
the underlying color and looks washed out.

### Inset Elements (Text Inputs, Wells, Checkboxes)

The user looks slightly downward, so only the bottom lip of the recessed area is visible
(it faces upward toward the light).

**Implementation:**
- **Bottom edge:** Slightly lighter than the element, using a bottom border or inset
  box-shadow with a negative vertical offset.
- **Top shadow:** Small, dark inset box-shadow with a slight positive vertical offset.
  This simulates the lip above blocking light from reaching the top of the well.
- Make sure the top inset shadow doesn't extend through the bottom of the element.

**Same treatment works for:** text inputs, textareas, checkboxes, and any recessed/well UI.

### Don't Overdo It

Borrow visual cues from the real world to create a sense of depth, but don't try to make
things look photo-realistic. Over-detailed shadows make interfaces busy and unclear.

## Shadow Elevation System

Shadows communicate where an element sits on a virtual z-axis. More shadow = closer to
the user = more attention-grabbing.

### Define 5 Fixed Levels

Like your color, type, and spacing systems, shadows should be a constrained set of options.
Five levels is enough for any interface:

```css
/* Level 1: Subtle raise -- buttons, cards at rest */
--shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);

/* Level 2: Slight elevation -- hover states, raised cards */
--shadow-md: 0 2px 4px rgba(0, 0, 0, 0.06),
             0 1px 2px rgba(0, 0, 0, 0.04);

/* Level 3: Medium elevation -- dropdowns, popovers */
--shadow-lg: 0 4px 8px rgba(0, 0, 0, 0.07),
             0 2px 4px rgba(0, 0, 0, 0.04);

/* Level 4: High elevation -- sticky elements, drawers */
--shadow-xl: 0 8px 16px rgba(0, 0, 0, 0.08),
             0 4px 8px rgba(0, 0, 0, 0.03);

/* Level 5: Highest elevation -- modals, dialogs */
--shadow-2xl: 0 16px 32px rgba(0, 0, 0, 0.1),
              0 8px 16px rgba(0, 0, 0, 0.02);
```

### Usage by Component

| Component | Shadow Level | Why |
|-----------|-------------|-----|
| Buttons | sm | Present but not dominant |
| Cards | sm or md | Slightly raised off the page |
| Hover states | md (promoted from sm) | Subtle feedback |
| Dropdowns | lg | Sits above the page content |
| Popovers/tooltips | lg | Clearly floating |
| Sticky headers | lg or xl | Above scrolling content |
| Drawers/panels | xl | Overlays but doesn't dominate |
| Modals/dialogs | 2xl | Captures full user attention |

### Two-Part Shadow Technique

A single shadow looks flat. Two shadows layered give more natural, controlled depth:

**Shadow 1 (cast shadow):**
- Large vertical offset
- Large blur radius
- Subtle opacity
- Simulates the shadow cast by a direct light source

**Shadow 2 (ambient occlusion):**
- Small vertical offset
- Small blur radius
- Darker, more defined
- Simulates the dark area directly beneath the element where ambient light can't reach

**Key behavior:** As elevation increases, the tight ambient shadow (Shadow 2) should become
more subtle. At the lowest elevation, it's quite visible. At the highest elevation, it's
nearly invisible -- physically accurate because a high-floating object is far from the
surface it would shadow.

### Interactive Shadows

Don't think about the shadow itself. Think about where the element should sit on the
z-axis, then assign the appropriate shadow.

- **Drag-to-sort:** When a user clicks and begins dragging a list item, add a shadow so
  the item appears to pop forward above its siblings. This communicates draggability.
- **Button press:** On click, switch to a smaller shadow or remove it entirely. The button
  feels like it's pressing into the page.
- **Hover:** Promote to the next shadow level up. The element lifts slightly toward the user.

## Depth in Flat Design

Flat design (no shadows, no gradients) can still communicate depth through three techniques:

### 1. Color-Based Depth

- **Lighter elements feel closer** to the viewer
- **Darker elements feel further away**
- Make an element lighter than the background to create a raised effect
- Make an element darker than the background for an inset/well effect

This works in non-flat designs too -- color is always a valid depth tool alongside shadows.

### 2. Solid Shadows

Use a short, vertically offset shadow with **zero blur radius** to make cards and buttons
stand off the page while preserving the flat aesthetic:

```css
box-shadow: 0 2px 0 hsl(220, 10%, 80%);
```

This creates a subtle, clean edge of depth without the softness of blurred shadows.

### 3. Overlapping Elements

Overlapping elements across boundaries creates a strong sense of layering:

- **Cross backgrounds:** Offset a card so it straddles the transition between two
  different background colors.
- **Overflow parent:** Make an element taller than its container so it extends beyond
  the edges on both sides.
- **Small components too:** Carousel controls, badges, avatars can all overlap their
  containers to create depth.

**Overlapping images:** When images overlap, they clash visually. Fix this by giving
images an "invisible border" -- a border that matches the background color. This creates
visual separation without a harsh visible border.

---

## Working with Images

### Photo Quality

Bad photos ruin a design regardless of everything else.

- Hire a professional for unique photography needs
- Use high-quality stock photography (Unsplash for free options)
- **Never design with placeholder images** and plan to swap in smartphone photos later. It never works.

### Text Over Background Images

Photos have both light and dark areas. No single text color works uniformly across both.

**Method 1: Semi-Transparent Overlay**
- Black overlay → tones down light areas → use with light text
- White overlay → brightens dark areas → use with dark text

**Method 2: Lower Image Contrast**
- Reduces dynamics across the whole image
- Adjust brightness to compensate for the overall shift

**Method 3: Colorize the Image**
1. Lower image contrast
2. Desaturate the image
3. Add a solid color fill using **multiply blend mode**
- Bonus: makes the image harmonize with brand colors

**Method 4: Text Shadow (Glow)**
- Large blur radius, **no offset** (creates a glow, not a directional shadow)
- Preserves image dynamics while increasing contrast where needed
- Combine with slightly reduced overall image contrast

### Scaling Rules

**Don't scale up icons.** Icons drawn at 16-24px look chunky and unprofessional at 3-4x
size because they were designed with minimal detail for small rendering. Even SVGs have
this problem.

**Fix:** Enclose the icon in a shape (circle, rounded square) with a background color. The
icon stays at its intended size; the shape fills the larger space.

**Don't scale down screenshots.** A full app screenshot shrunk 70% turns 16px text into
~4px text -- unreadable.

**Alternatives:**
- Take screenshots at a smaller viewport (tablet) and give them more display space
- Crop to show only a portion of the interface
- Draw a simplified wireframe version with small text replaced by lines

**Don't scale down large icons to small sizes.** A 128px logo shrunk to 16px (favicon)
turns to mush.

**Fix:** Redraw a simplified version at the target size. Control the compromises yourself
rather than leaving them to the browser's downscaling algorithm.

### User-Uploaded Images

You can't control user photo quality, contrast, color, or cropping.

**Control shape and size:**
- Center images in fixed-dimension containers and crop overflow
- CSS: `background-size: cover` or `object-fit: cover`
- Never let user images display at their native aspect ratio

**Prevent background bleed:**
When a user image's background color matches the UI background, the image loses its edge.
- Don't use a visible border (it clashes with image colors)
- Use a **subtle inner box-shadow**: nearly invisible but defines the boundary
- Alternative: semi-transparent inner border
