# Design Systems

## Profile

### What It Is
A Design System is a collection of reusable components, patterns, guidelines, and principles that ensure consistency and efficiency across product development. It bridges design and engineering with a shared language, component library, design tokens, and documentation.

### Origin and Evolution
- Style guides and pattern libraries (2000s-2010s)
- Atomic Design by Brad Frost (2013) — atoms, molecules, organisms hierarchy
- Major design systems: Material Design (Google), Carbon (IBM), Polaris (Shopify)
- Component-driven development with Storybook (2016)
- Current: design tokens, multi-platform systems, AI-generated components

### Key Authors and References
- **Brad Frost** — *Atomic Design* (2016)
- **Nathan Curtis** — Design systems at scale
- **Alla Kholmatova** — *Design Systems* (Smashing Magazine)
- **Google** — Material Design system

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Design Tokens | Atomic design decisions (colors, spacing, typography) as variables |
| Component Library | Reusable UI components with consistent API |
| Pattern Library | Common UI patterns (forms, navigation, data display) |
| Storybook | Component development and documentation tool |
| Atomic Design | Hierarchy: atoms → molecules → organisms → templates → pages |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Consistency across products** — Every product, platform, and team uses the same components and patterns. Users get a unified experience.
2. **Components as the API** — Components are the contract between design and engineering. Clear props, documented behavior, tested interactions.
3. **Design tokens for theming** — Colors, spacing, typography defined as tokens, not hardcoded values. Enables theming and multi-brand support.
4. **Documentation is the product** — A design system without documentation is a code library. Documentation makes it usable by the whole organization.
5. **Treat it as a product, not a project** — Design systems need a dedicated team, versioning, release process, and user support.

### When to Use vs. When to Avoid

**Invest in a design system when:**
- Multiple products or teams share UI patterns
- Consistency across platforms is important
- New products/features are built frequently
- Design and engineering need a shared language

**Keep it simple when:**
- Single small team, one product
- Early-stage startup where UI is changing rapidly
- The overhead of maintaining a system exceeds the consistency benefit

---

## Frameworks and Methodologies

### 1. Design System Creation — step-by-step
1. Audit existing UI: inventory all components, patterns, and inconsistencies
2. Define design tokens (colors, spacing, typography, shadows, breakpoints)
3. Build foundational components (Button, Input, Card, Modal, Typography)
4. Document component API, usage guidelines, and accessibility requirements
5. Set up Storybook for component development and showcase
6. Version and publish as a package (npm, internal registry)
7. Create contribution guidelines for the team

### 2. Multi-Platform Tokens — step-by-step
1. Define tokens in a platform-agnostic format (JSON, YAML)
2. Use Style Dictionary or Figma Tokens to transform tokens per platform
3. Generate: CSS custom properties (web), Swift constants (iOS), XML resources (Android)
4. Sync tokens between Figma and code
5. Automate token generation in CI

---

## How to Apply

### Decision Checklist
- [ ] Are design tokens defined and used consistently?
- [ ] Are components documented with props, usage, and accessibility?
- [ ] Is there a Storybook or similar showcase?
- [ ] Is the design system versioned and published as a package?
- [ ] Is there a contribution process for new components?

### Implementation Patterns

**Design Tokens:**
```json
{
  "color": {
    "primary": { "value": "#2563EB" },
    "secondary": { "value": "#7C3AED" },
    "error": { "value": "#DC2626" },
    "text": { "value": "#1F2937" },
    "background": { "value": "#FFFFFF" }
  },
  "spacing": {
    "xs": { "value": "4px" },
    "sm": { "value": "8px" },
    "md": { "value": "16px" },
    "lg": { "value": "24px" },
    "xl": { "value": "32px" }
  },
  "font": {
    "family": { "value": "'Inter', sans-serif" },
    "size": {
      "sm": { "value": "14px" },
      "base": { "value": "16px" },
      "lg": { "value": "18px" },
      "xl": { "value": "24px" }
    }
  }
}
```

**Component with TypeScript:**
```typescript
interface ButtonProps {
  variant: 'primary' | 'secondary' | 'ghost';
  size: 'sm' | 'md' | 'lg';
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
  loading?: boolean;
}

export function Button({ variant, size, children, onClick, disabled, loading }: ButtonProps) {
  return (
    <button
      className={cn(styles.button, styles[variant], styles[size])}
      onClick={onClick}
      disabled={disabled || loading}
      aria-busy={loading}
    >
      {loading ? <Spinner size={size} /> : children}
    </button>
  );
}
```

### Common Mistakes
1. **Building too much upfront** — Start with 5-10 core components, grow based on demand
2. **No documentation** — Components without docs get misused or duplicated
3. **Design-only system** — Figma components without matching code components
4. **Rigid system** — No escape hatches for edge cases; teams work around it
5. **No team ownership** — Design system without a dedicated maintainer deteriorates

### Integration with Other Topics
- **Frontend Architecture** — Design system provides the component foundation
- **TypeScript** — Typed component props ensure correct usage
- **Testing** — Visual regression testing for components (Chromatic, Percy)
- **Mobile Architecture** — Multi-platform design tokens and components
- **Documentation as Code** — Storybook as living documentation
- **Code Quality** — Consistent components reduce UI bugs

---

## Resources

### Essential Reading
- *Atomic Design* — Brad Frost (atomicdesign.bradfrost.com)
- *Design Systems* — Alla Kholmatova
- Material Design documentation (material.io)

### Online Resources
- Storybook documentation (storybook.js.org)
- Style Dictionary documentation (amzn.github.io/style-dictionary)
- designsystems.com — Community resources

### Practice Exercises
1. Define design tokens for a project (colors, spacing, typography)
2. Build 5 core components with TypeScript and Storybook
3. Set up Style Dictionary to generate tokens for web and mobile
4. Create a contribution guide for the design system
5. Implement visual regression testing with Chromatic
