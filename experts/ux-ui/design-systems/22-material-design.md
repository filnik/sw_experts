# Material Design

## Profile

### What It Is
Material Design is Google's open-source design system that provides comprehensive guidelines, components, and tools for building high-quality digital experiences across platforms. Rooted in the metaphor of physical material and ink, it defines principles for motion, elevation, color, and typography that create intuitive, beautiful interfaces. Material Design 3 (Material You) extends the system with dynamic color, personalization, and updated component specifications.

### Origin and Evolution
- **2014** — Google introduces Material Design at Google I/O, establishing the "material as a metaphor" philosophy with a focus on paper-like surfaces and realistic motion
- **2016** — Material Components for Web (MDC-Web) released, providing official CSS/JS implementations
- **2018** — Material Theming introduced, allowing deeper customization of color, typography, and shape beyond the default Google aesthetic
- **2021** — Material You (Material Design 3) announced at Google I/O, introducing dynamic color extraction from wallpapers, updated components, and a warmer visual language
- **2023–present** — Material Design 3 becomes the default for Android development; Material Web (web components) enters stable releases; the M3 design kit for Figma is widely adopted

### Key Authors and References
- **Google Design Team** — Material Design specification (material.io)
- **Matias Duarte** — VP of Design at Google, principal creator of Material Design
- **Material Design Guidelines** — [m3.material.io](https://m3.material.io) (Material 3 official documentation)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Material as Metaphor | UI surfaces behave like physical material — they have edges, elevation, and cast shadows |
| Elevation | Z-axis depth measured in density-independent pixels (dp), creating shadow hierarchy |
| Dynamic Color | M3 system that extracts a color scheme from a user's wallpaper or a source image |
| Type Scale | A defined set of typographic styles (Display, Headline, Title, Body, Label) with size/weight/tracking |
| Color Roles | Systematic mapping: primary, secondary, tertiary, surface, error, with on-colors and containers |
| Motion | Animations that provide meaning: transitions show spatial relationships and continuity |
| Responsive Layout Grid | Column-based grid system with margins and gutters that adapts to screen size |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Material is the metaphor** — A rational model where UI surfaces are informed by the study of paper and ink. Surfaces have physical properties: they can be stacked, split, and cast shadows.
2. **Bold, graphic, intentional** — Typography, grids, space, scale, and color create hierarchy and meaning. Deliberate design choices guide the user.
3. **Motion provides meaning** — Animation is not decoration. It communicates relationships, confirms actions, and maintains spatial consistency during transitions.
4. **Adaptive and universal** — A single underlying design system adapts to any screen size, input type, or platform while maintaining a coherent identity.
5. **Accessible by design** — Color contrast, touch targets, screen reader support, and keyboard navigation are built into the specification from the ground level.

### When to Use vs. When to Avoid

**Use when:**
- Building Android applications where Material is the native design language
- Rapid prototyping when you need a complete, well-documented component set
- Cross-platform projects (web + mobile) that benefit from a unified system
- Teams without dedicated design resources who need strong default guidelines

**Avoid over-application when:**
- Your brand identity needs to look distinctly non-Google — heavy Material use can feel "Google-esque"
- You are building for Apple platforms where users expect HIG conventions (tab bars, navigation patterns)
- The project requires highly custom, branded UI where Material's opinionated defaults would need extensive overriding

---

## Frameworks and Methodologies

### 1. Material 3 Color System — step-by-step
1. Choose a **source color** (your brand primary color)
2. Use the [Material Theme Builder](https://m3.material.io/theme-builder) to generate a complete palette with tonal palettes (0-100 tone values)
3. Map generated values to **color roles**: Primary, On-Primary, Primary-Container, On-Primary-Container (repeat for Secondary, Tertiary, Error)
4. Define **surface** colors: Surface, On-Surface, Surface-Variant, On-Surface-Variant, Outline
5. Generate both light and dark scheme variants
6. Implement as CSS custom properties or platform-specific tokens

### 2. Material Elevation System — step-by-step
1. Define elevation levels: Level 0 (0dp), Level 1 (1dp), Level 2 (3dp), Level 3 (6dp), Level 4 (8dp), Level 5 (12dp)
2. Map components to elevation levels: FAB at Level 3, App bar at Level 2, Card at Level 1, Dialog at Level 3
3. Implement shadows using CSS `box-shadow` with two layers (key light + ambient light)
4. In M3, elevation is also expressed through **surface tint** (a semi-transparent overlay of the primary color) in addition to shadows

### 3. Responsive Layout Grid — step-by-step
1. Define breakpoints: Compact (0-599dp), Medium (600-904dp), Expanded (905dp+)
2. Set column counts: 4 columns (compact), 8 columns (medium), 12 columns (expanded)
3. Define margins: 16dp (compact), 24dp (medium), 24dp (expanded)
4. Define gutters between columns: 8dp or 16dp depending on density
5. Use the grid to place components, spanning multiple columns as needed

---

## How to Apply

### Decision Checklist
- [ ] Have I generated a complete M3 color scheme using the Theme Builder?
- [ ] Are all color roles defined (primary, secondary, tertiary, surface, error + containers + on-colors)?
- [ ] Is the type scale implemented with all 5 categories (Display, Headline, Title, Body, Label)?
- [ ] Are elevation levels applied consistently to all surfaces?
- [ ] Does the layout grid adapt correctly at compact, medium, and expanded breakpoints?
- [ ] Are touch targets at least 48x48dp for interactive elements?
- [ ] Have I verified color contrast ratios meet WCAG AA (4.5:1 for text, 3:1 for large text)?
- [ ] Are transitions and animations meaningful, not purely decorative?

### Implementation Patterns

**Material 3 Color Tokens as CSS Custom Properties:**
```css
/* Material 3 Light Theme */
:root, [data-theme="light"] {
  /* Primary */
  --md-sys-color-primary: #6750a4;
  --md-sys-color-on-primary: #ffffff;
  --md-sys-color-primary-container: #eaddff;
  --md-sys-color-on-primary-container: #21005d;

  /* Secondary */
  --md-sys-color-secondary: #625b71;
  --md-sys-color-on-secondary: #ffffff;
  --md-sys-color-secondary-container: #e8def8;
  --md-sys-color-on-secondary-container: #1d192b;

  /* Tertiary */
  --md-sys-color-tertiary: #7d5260;
  --md-sys-color-on-tertiary: #ffffff;
  --md-sys-color-tertiary-container: #ffd8e4;
  --md-sys-color-on-tertiary-container: #31111d;

  /* Error */
  --md-sys-color-error: #b3261e;
  --md-sys-color-on-error: #ffffff;
  --md-sys-color-error-container: #f9dedc;
  --md-sys-color-on-error-container: #410e0b;

  /* Surface */
  --md-sys-color-surface: #fffbfe;
  --md-sys-color-on-surface: #1c1b1f;
  --md-sys-color-surface-variant: #e7e0ec;
  --md-sys-color-on-surface-variant: #49454f;
  --md-sys-color-outline: #79747e;
  --md-sys-color-outline-variant: #cac4d0;
}

/* Material 3 Dark Theme */
[data-theme="dark"] {
  --md-sys-color-primary: #d0bcff;
  --md-sys-color-on-primary: #381e72;
  --md-sys-color-primary-container: #4f378b;
  --md-sys-color-on-primary-container: #eaddff;

  --md-sys-color-surface: #1c1b1f;
  --md-sys-color-on-surface: #e6e1e5;
  --md-sys-color-surface-variant: #49454f;
  --md-sys-color-on-surface-variant: #cac4d0;
  --md-sys-color-outline: #938f99;
}
```

**Material 3 Type Scale:**
```css
:root {
  --md-sys-typescale-display-large: 400 3.5625rem/4rem var(--md-ref-typeface-brand);  /* 57px */
  --md-sys-typescale-display-medium: 400 2.8125rem/3.25rem var(--md-ref-typeface-brand); /* 45px */
  --md-sys-typescale-display-small: 400 2.25rem/2.75rem var(--md-ref-typeface-brand);  /* 36px */

  --md-sys-typescale-headline-large: 400 2rem/2.5rem var(--md-ref-typeface-brand);     /* 32px */
  --md-sys-typescale-headline-medium: 400 1.75rem/2.25rem var(--md-ref-typeface-brand); /* 28px */
  --md-sys-typescale-headline-small: 400 1.5rem/2rem var(--md-ref-typeface-brand);     /* 24px */

  --md-sys-typescale-title-large: 400 1.375rem/1.75rem var(--md-ref-typeface-brand);   /* 22px */
  --md-sys-typescale-title-medium: 500 1rem/1.5rem var(--md-ref-typeface-brand);       /* 16px */
  --md-sys-typescale-title-small: 500 0.875rem/1.25rem var(--md-ref-typeface-brand);   /* 14px */

  --md-sys-typescale-body-large: 400 1rem/1.5rem var(--md-ref-typeface-plain);         /* 16px */
  --md-sys-typescale-body-medium: 400 0.875rem/1.25rem var(--md-ref-typeface-plain);   /* 14px */
  --md-sys-typescale-body-small: 400 0.75rem/1rem var(--md-ref-typeface-plain);        /* 12px */

  --md-sys-typescale-label-large: 500 0.875rem/1.25rem var(--md-ref-typeface-plain);   /* 14px */
  --md-sys-typescale-label-medium: 500 0.75rem/1rem var(--md-ref-typeface-plain);      /* 12px */
  --md-sys-typescale-label-small: 500 0.6875rem/1rem var(--md-ref-typeface-plain);     /* 11px */

  --md-ref-typeface-brand: 'Roboto', sans-serif;
  --md-ref-typeface-plain: 'Roboto', sans-serif;
}
```

**Elevation System with CSS:**
```css
:root {
  /* M3 Elevation Levels (key shadow + ambient shadow) */
  --md-sys-elevation-0: none;
  --md-sys-elevation-1:
    0 1px 2px rgb(0 0 0 / 0.3),
    0 1px 3px 1px rgb(0 0 0 / 0.15);
  --md-sys-elevation-2:
    0 1px 2px rgb(0 0 0 / 0.3),
    0 2px 6px 2px rgb(0 0 0 / 0.15);
  --md-sys-elevation-3:
    0 4px 8px 3px rgb(0 0 0 / 0.15),
    0 1px 3px rgb(0 0 0 / 0.3);
  --md-sys-elevation-4:
    0 6px 10px 4px rgb(0 0 0 / 0.15),
    0 2px 3px rgb(0 0 0 / 0.3);
  --md-sys-elevation-5:
    0 8px 12px 6px rgb(0 0 0 / 0.15),
    0 4px 4px rgb(0 0 0 / 0.3);
}

/* Surface tint overlay for M3 (replaces shadow in some contexts) */
.surface-level-1 {
  background: color-mix(in srgb, var(--md-sys-color-primary) 5%, var(--md-sys-color-surface));
}
.surface-level-2 {
  background: color-mix(in srgb, var(--md-sys-color-primary) 8%, var(--md-sys-color-surface));
}
.surface-level-3 {
  background: color-mix(in srgb, var(--md-sys-color-primary) 11%, var(--md-sys-color-surface));
}
```

**Material Card Component (HTML + CSS):**
```html
<article class="md-card md-card--elevated">
  <div class="md-card__media">
    <img src="image.jpg" alt="Product photo" />
  </div>
  <div class="md-card__content">
    <h2 class="md-typescale-headline-small">Card Title</h2>
    <p class="md-typescale-body-medium">Supporting text that describes the content.</p>
  </div>
  <div class="md-card__actions">
    <button class="md-btn md-btn--text">Cancel</button>
    <button class="md-btn md-btn--filled">Confirm</button>
  </div>
</article>
```

```css
.md-card {
  border-radius: 12px;
  overflow: hidden;
  background: var(--md-sys-color-surface);
  color: var(--md-sys-color-on-surface);
}

.md-card--elevated {
  box-shadow: var(--md-sys-elevation-1);
}

.md-card--filled {
  background: var(--md-sys-color-surface-variant);
  box-shadow: var(--md-sys-elevation-0);
}

.md-card--outlined {
  border: 1px solid var(--md-sys-color-outline-variant);
  box-shadow: var(--md-sys-elevation-0);
}

.md-card__content {
  padding: 16px;
}

.md-card__actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  padding: 8px 16px 16px;
}

/* Typography utility classes */
.md-typescale-headline-small {
  font: 400 1.5rem/2rem var(--md-ref-typeface-brand);
  color: var(--md-sys-color-on-surface);
  margin: 0 0 8px;
}

.md-typescale-body-medium {
  font: 400 0.875rem/1.25rem var(--md-ref-typeface-plain);
  color: var(--md-sys-color-on-surface-variant);
  margin: 0;
}
```

**Responsive Layout Grid:**
```css
.md-layout-grid {
  display: grid;
  gap: 8px;
  margin-inline: 16px;
  grid-template-columns: repeat(4, 1fr);
}

/* Medium breakpoint (600dp+) */
@media (min-width: 600px) {
  .md-layout-grid {
    gap: 16px;
    margin-inline: 24px;
    grid-template-columns: repeat(8, 1fr);
  }
}

/* Expanded breakpoint (905dp+) */
@media (min-width: 905px) {
  .md-layout-grid {
    gap: 16px;
    margin-inline: 24px;
    grid-template-columns: repeat(12, 1fr);
    max-width: 1440px;
  }
}
```

**Comparison: Material Design vs. Apple HIG vs. Fluent Design:**

| Aspect | Material Design (Google) | Human Interface Guidelines (Apple) | Fluent Design (Microsoft) |
|--------|-------------------------|-----------------------------------|--------------------------|
| Metaphor | Physical material and ink | Clarity, deference, depth | Light, depth, motion, material, scale |
| Elevation | dp-based shadow levels + surface tint | Translucency and vibrancy layers | Acrylic (blur) and reveal effects |
| Motion | Shared axis, container transform | Spring physics animations | Connected animations, implicit transitions |
| Typography | Roboto, 5 categories x 3 sizes | SF Pro, Dynamic Type | Segoe UI, type ramp |
| Color | Tonal palettes, dynamic color | System colors, accent tinting | Accent colors, semantic palette |
| Grid | 4/8/12 column responsive | Auto Layout, HIG spacing guidelines | Responsive breakpoints |
| Primary platform | Android, Web | iOS, macOS | Windows, Web |
| Open source | Yes (material.io) | Partial (SwiftUI docs) | Yes (fluentui.dev) |

### Common Mistakes
1. **Using Material defaults without theming** — Out-of-the-box Material looks like a Google product. Always customize the color scheme and typography to match your brand.
2. **Mixing M2 and M3 components** — Material 2 and Material 3 have different elevation models, color roles, and component designs. Choose one version and be consistent.
3. **Ignoring elevation hierarchy** — Placing unrelated elements at the same elevation confuses the spatial model. Use elevation to communicate which elements are above others.
4. **Skipping the responsive grid** — Material's grid system is essential for layout consistency. Do not use arbitrary breakpoints or ad-hoc column counts.
5. **Over-animating** — Material motion should be functional. Avoid using transitions purely for visual flair; every animation should communicate a relationship or state change.

### Integration with Other Topics
- **Atomic Design (20)** — Material components (Button, Card, FAB) map to atoms and molecules; Material layout patterns map to templates.
- **Design Systems (21)** — Material Design is itself a design system. It can serve as a foundation for a custom design system with branded tokens layered on top.
- **Color Theory (26)** — Material's tonal palette system is a practical application of color theory principles, particularly hue, chroma, and tone relationships.

---

## Resources

### Essential Reading
- Google — [Material Design 3 Guidelines](https://m3.material.io/)
- Google — [Material Theme Builder](https://m3.material.io/theme-builder)
- Matias Duarte — Google I/O keynotes on Material Design evolution (2014, 2018, 2021)

### Online Resources
- [Material Web Components](https://github.com/nicoth-in/material-web) — Official web components for M3
- [MUI (Material UI for React)](https://mui.com/) — Popular React implementation of Material Design
- [Material Design Figma Kit](https://www.figma.com/community/file/1035203688168086460) — Official M3 design kit
- [Material Symbols and Icons](https://fonts.google.com/icons) — Official icon library

### Practice Exercises
1. **Theme Builder exercise:** Use the Material Theme Builder to generate a complete color scheme from your brand's primary color. Export the tokens and implement them as CSS custom properties with both light and dark modes.
2. **Card component trio:** Build all three card variants (elevated, filled, outlined) using only CSS custom properties and the Material elevation system. Add proper hover states with elevation changes.
3. **Responsive grid page:** Create a product listing page using the Material 4/8/12 column grid. Verify that the layout adapts correctly at all three breakpoints (compact, medium, expanded).
4. **Cross-system comparison:** Take a single UI pattern (e.g., a settings screen) and implement it three ways: Material Design, Apple HIG, and Fluent Design. Document the key differences in layout, spacing, and interaction.
