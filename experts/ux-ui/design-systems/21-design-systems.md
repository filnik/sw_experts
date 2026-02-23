# Design Systems

## Profile

### What It Is
A design system is a comprehensive set of standards, documentation, and reusable components that unify product design and development across teams and platforms. It encompasses design tokens, a component library, usage guidelines, governance processes, and distribution mechanisms. Beyond a style guide or pattern library, a design system is a living product that serves other products.

### Origin and Evolution
- **2013** — Lonely Planet releases its open-source style guide (Rizzo), one of the first public "living style guides"
- **2014** — Google launches Material Design; Salesforce launches Lightning Design System — enterprise design systems gain momentum
- **2016** — Alla Kholmatova publishes *Design Systems* (Smashing Magazine); Nathan Curtis begins his influential EightShapes articles
- **2017–2018** — Storybook becomes the dominant tool for component documentation; design tokens gain formal definition
- **2019–2020** — Figma tokens and style dictionaries mature; headless/unstyled component libraries (Radix, Headless UI) emerge
- **2022–present** — shadcn/ui popularizes a copy-paste component model; AI-assisted design system generation appears

### Key Authors and References
- **Nathan Curtis** — EightShapes articles on design system architecture, team models, and component specification
- **Alla Kholmatova** — *Design Systems: A Practical Guide to Creating Design Languages* (2016, Smashing Magazine)
- **Brad Frost** — *Atomic Design* (2016) — structural methodology that underpins many design systems
- **Jina Anne** — Design tokens pioneer, creator of the Design Tokens Community Group (W3C)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Design Tokens | Platform-agnostic variables for color, spacing, typography, elevation, motion |
| Component Library | Coded, reusable UI components with defined APIs and documentation |
| Documentation Site | A living reference (often Storybook) with usage guidelines, dos/don'ts, examples |
| Governance Model | Processes for proposing, reviewing, and releasing design system changes |
| Versioning | Semantic versioning (semver) applied to tokens and components via npm packages |
| Design-Dev Handoff | Bridging design tools (Figma) with code through tokens and shared naming |
| Contribution Model | Rules for how product teams contribute back to the system (federated, centralized, hybrid) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Single source of truth** — One canonical definition for every token, component, and pattern. Duplicates create drift.
2. **Consistency over novelty** — The system enables teams to ship faster by constraining choices to proven patterns. Custom solutions require justification.
3. **Accessibility by default** — Every component ships WCAG 2.1 AA compliant. Accessibility is not an afterthought or an optional addon.
4. **Composability** — Components are designed to be composed together. Avoid monolithic components with dozens of props; prefer smaller units that combine.
5. **The system is a product** — It has users (product teams), roadmaps, releases, and support channels. Treat it with the same rigor as a customer-facing product.

### When to Use vs. When to Avoid

**Use when:**
- Multiple teams or products share UI patterns and need visual/behavioral consistency
- Onboarding new developers and designers needs to be fast and self-service
- The product is scaling and ad-hoc patterns are causing inconsistency and tech debt
- You want to enforce accessibility, performance, and brand standards at the component level

**Avoid over-application when:**
- You are a solo developer on a single small project — a lightweight utility-class approach may suffice
- The product domain is highly experimental and patterns are not yet stable
- Governance overhead would slow down a team that needs maximum autonomy in early stages

---

## Frameworks and Methodologies

### 1. Design Token Architecture — step-by-step
1. Define **global tokens** (raw values): color palette hex codes, spacing pixel values, font stacks
2. Create **semantic tokens** (aliases): `color-primary` references a global token; `spacing-md` references `8px`
3. Add **component tokens** (scoped): `button-bg-color` references `color-primary`
4. Store tokens in a platform-agnostic format (JSON or YAML)
5. Use a token transformation tool (Style Dictionary, Tokens Studio) to generate CSS custom properties, iOS/Android values, and Tailwind config

### 2. Component Library Architecture — step-by-step
1. Choose a foundation: build from scratch, extend headless primitives (Radix, Headless UI), or adopt a full framework (Material UI, Chakra UI)
2. Define component API surface (props, variants, sizes, states) based on design specs
3. Implement components with accessibility baked in (ARIA attributes, keyboard navigation, focus management)
4. Write unit tests (Jest/Vitest + Testing Library) and visual regression tests (Chromatic, Percy)
5. Document each component in Storybook with stories for every variant, state, and edge case
6. Package as an npm library with tree-shaking support and TypeScript types
7. Publish with semantic versioning and a changelog

### 3. Governance and Contribution Model — step-by-step
1. Establish a **core team** responsible for maintaining the system
2. Define a **contribution process**: RFC (Request for Comments) for new components or breaking changes
3. Create **quality gates**: code review, design review, accessibility audit, visual regression check
4. Publish a **decision log** documenting why patterns were chosen or rejected
5. Set a **release cadence**: regular releases (e.g., bi-weekly) with a clear deprecation policy

---

## How to Apply

### Decision Checklist
- [ ] Have I defined design tokens for colors, spacing, typography, shadows, and breakpoints?
- [ ] Are tokens stored in a platform-agnostic format (JSON/YAML) with a build pipeline?
- [ ] Does the component library cover the most-used patterns (buttons, inputs, modals, navigation)?
- [ ] Is every component documented with props, usage guidelines, and accessibility notes?
- [ ] Are components versioned and distributed via a package manager (npm)?
- [ ] Is there a Storybook (or equivalent) instance deployed and accessible to all teams?
- [ ] Is there a governance model with clear ownership and contribution guidelines?
- [ ] Have I decided: build custom, adopt existing, or hybrid?

### Implementation Patterns

**Design Tokens as CSS Custom Properties:**
```css
/* tokens/global.css — raw values */
:root {
  /* Color palette */
  --color-blue-50: #eff6ff;
  --color-blue-500: #3b82f6;
  --color-blue-700: #1d4ed8;
  --color-gray-50: #f9fafb;
  --color-gray-900: #111827;

  /* Spacing scale (4px base) */
  --space-1: 0.25rem;  /* 4px */
  --space-2: 0.5rem;   /* 8px */
  --space-3: 0.75rem;  /* 12px */
  --space-4: 1rem;     /* 16px */
  --space-6: 1.5rem;   /* 24px */
  --space-8: 2rem;     /* 32px */

  /* Typography */
  --font-sans: 'Inter', system-ui, -apple-system, sans-serif;
  --font-mono: 'JetBrains Mono', ui-monospace, monospace;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --leading-tight: 1.25;
  --leading-normal: 1.5;

  /* Shadows */
  --shadow-sm: 0 1px 2px rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px rgb(0 0 0 / 0.07), 0 2px 4px rgb(0 0 0 / 0.06);
  --shadow-lg: 0 10px 15px rgb(0 0 0 / 0.1), 0 4px 6px rgb(0 0 0 / 0.05);

  /* Border radius */
  --radius-sm: 0.25rem;
  --radius-md: 0.375rem;
  --radius-lg: 0.5rem;
  --radius-full: 9999px;
}
```

**Semantic Tokens (light/dark theming):**
```css
/* tokens/semantic-light.css */
:root, [data-theme="light"] {
  --color-bg-primary: var(--color-gray-50);
  --color-bg-surface: #ffffff;
  --color-text-primary: var(--color-gray-900);
  --color-text-secondary: #6b7280;
  --color-border: #e5e7eb;
  --color-accent: var(--color-blue-500);
  --color-accent-hover: var(--color-blue-700);
}

/* tokens/semantic-dark.css */
[data-theme="dark"] {
  --color-bg-primary: var(--color-gray-900);
  --color-bg-surface: #1f2937;
  --color-text-primary: var(--color-gray-50);
  --color-text-secondary: #9ca3af;
  --color-border: #374151;
  --color-accent: var(--color-blue-500);
  --color-accent-hover: var(--color-blue-50);
}
```

**Component Token Example (scoped to component):**
```css
/* components/button.css */
.btn {
  --_btn-bg: var(--color-accent);
  --_btn-color: #ffffff;
  --_btn-padding-x: var(--space-4);
  --_btn-padding-y: var(--space-2);
  --_btn-radius: var(--radius-md);
  --_btn-font-size: var(--text-sm);

  background-color: var(--_btn-bg);
  color: var(--_btn-color);
  padding: var(--_btn-padding-y) var(--_btn-padding-x);
  border-radius: var(--_btn-radius);
  font-size: var(--_btn-font-size);
  font-family: var(--font-sans);
  font-weight: 500;
  border: none;
  cursor: pointer;
  transition: background-color 150ms ease;
}

.btn:hover {
  --_btn-bg: var(--color-accent-hover);
}

.btn--secondary {
  --_btn-bg: transparent;
  --_btn-color: var(--color-accent);
  border: 1px solid var(--color-accent);
}
```

**Storybook Configuration (basic setup):**
```js
// .storybook/main.js
export default {
  stories: ['../src/components/**/*.stories.@(js|jsx|ts|tsx)'],
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-a11y',       // accessibility checks
    '@storybook/addon-interactions', // interaction testing
  ],
  framework: '@storybook/react-vite',
};
```

**Storybook Story Example:**
```jsx
// src/components/Button/Button.stories.jsx
import { Button } from './Button';

export default {
  title: 'Atoms/Button',
  component: Button,
  argTypes: {
    variant: { control: 'select', options: ['primary', 'secondary', 'ghost'] },
    size: { control: 'select', options: ['sm', 'md', 'lg'] },
  },
};

export const Primary = { args: { variant: 'primary', children: 'Click me' } };
export const Secondary = { args: { variant: 'secondary', children: 'Click me' } };
export const Disabled = { args: { variant: 'primary', children: 'Disabled', disabled: true } };
```

**Comparison: Build Custom vs. Adopt Existing:**

| Factor | Build Custom | Material UI / Chakra | Radix / Headless UI | shadcn/ui |
|--------|-------------|---------------------|---------------------|-----------|
| Brand alignment | Full control | Needs heavy theming | Full control (unstyled) | Full control (copy-paste) |
| Development speed | Slow initial | Fast initial | Medium | Fast |
| Bundle size | Optimized | Can be large | Minimal | Optimized (only what you use) |
| Maintenance burden | High (your team owns it) | Low (community) | Medium | Medium (you own the code) |
| Accessibility | You must build it | Built-in | Built-in | Built-in (via Radix) |
| Best for | Unique brand, large team | Rapid prototyping, MVPs | Custom look, a11y focus | Modern React projects |

### Common Mistakes
1. **Starting with components, not tokens** — Tokens are the foundation. Without them, components will have hardcoded values that resist theming and scaling.
2. **No governance process** — Without clear ownership and contribution rules, the system fragments as teams build parallel solutions.
3. **Documentation as an afterthought** — If a component is not documented, it does not exist. Undocumented components get reinvented.
4. **Over-engineering v1** — Ship a small, well-tested set of core components first. Expand based on measured demand, not speculation.
5. **Ignoring adoption metrics** — Track how many teams use the system and which components are most/least adopted. Data drives roadmap.

### Integration with Other Topics
- **Atomic Design (20)** — Provides the structural hierarchy (atoms through pages) that organizes a design system's component library.
- **Material Design (22)** — A complete reference design system; useful as inspiration or as a base to customize.
- **Color Theory (26)** — Informs the token definitions for color palettes, ensuring accessible contrast and intentional emotional impact.
- **Web Typography (24)** — Typography tokens (font families, sizes, line heights, weights) form a critical subset of the token system.

---

## Resources

### Essential Reading
- Alla Kholmatova — *Design Systems: A Practical Guide to Creating Design Languages* (Smashing Magazine, 2016)
- Nathan Curtis — EightShapes Design System articles series ([medium.com/eightshapes-llc](https://medium.com/eightshapes-llc))
- Brad Frost — *Atomic Design* (2016)
- Jina Anne — Design Tokens W3C Community Group specification drafts

### Online Resources
- [Storybook](https://storybook.js.org/) — Component documentation and visual testing
- [Style Dictionary](https://amzn.github.io/style-dictionary/) — Token transformation pipeline by Amazon
- [Tokens Studio](https://tokens.studio/) — Figma plugin for managing design tokens
- [shadcn/ui](https://ui.shadcn.com/) — Copy-paste component library built on Radix and Tailwind
- [Adele](https://adele.uxpin.com/) — Repository of publicly available design systems for inspiration

### Practice Exercises
1. **Token system from scratch:** Define a complete set of design tokens (colors, spacing, typography, shadows) as CSS custom properties. Implement light and dark themes using only semantic token swaps.
2. **Component library starter:** Build 5 core components (Button, Input, Card, Modal, Badge) with design tokens, full accessibility, and Storybook documentation.
3. **Governance simulation:** Write an RFC document proposing a new DatePicker component. Include API design, accessibility requirements, and acceptance criteria.
4. **Audit and adopt:** Take an existing project and replace all hardcoded color/spacing values with design tokens. Measure the reduction in unique values.
