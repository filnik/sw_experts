# Frontend Architecture

## Profile

### What It Is
Frontend Architecture encompasses the patterns, principles, and decisions for structuring client-side applications — including state management, component architecture, rendering strategies (SPA, SSR, SSG, Islands), routing, data fetching, and performance optimization.

### Origin and Evolution
- jQuery era (2006): DOM manipulation without architecture
- MVC/MVVM frameworks: Backbone, Angular 1, Knockout
- Component-based: React (2013), Vue (2014), Angular 2+ (2016)
- Meta-frameworks: Next.js, Nuxt, SvelteKit provide full architecture
- Current: Server Components (React), Islands Architecture (Astro), signals-based reactivity

### Key Authors and References
- **Dan Abramov** — React, Redux, component patterns
- **Addy Osmani** — *Learning JavaScript Design Patterns*
- **Jason Miller** — Islands Architecture, Preact
- **Ryan Carniato** — SolidJS, fine-grained reactivity research

### Key Concepts at a Glance
| Pattern | Description |
|---------|-------------|
| SPA | Single Page Application — client-side routing, API-driven |
| SSR | Server-Side Rendering — HTML generated per request |
| SSG | Static Site Generation — HTML generated at build time |
| Islands | Interactive components in static HTML pages |
| Component | Reusable, encapsulated UI building block |
| State Management | Centralized (Redux/Zustand) or distributed (Context/signals) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Component composition** — Build UIs from small, reusable, composable components. Components are the atoms of frontend architecture.
2. **Unidirectional data flow** — Data flows down (props), events flow up. Predictable state management prevents bugs.
3. **Render what you need** — Choose the rendering strategy (SSR, SSG, CSR, Islands) based on the page's requirements, not a one-size-fits-all.
4. **Performance is a feature** — Core Web Vitals (LCP, FID, CLS) directly impact user experience and SEO. Performance must be designed in.
5. **Colocation** — Keep related code together: component, styles, tests, and types in the same directory.

### When to Use vs. When to Avoid

**SPA**: Highly interactive apps (dashboards, editors, internal tools)
**SSR**: SEO-critical pages with dynamic content, personalized pages
**SSG**: Content sites, blogs, documentation, marketing pages
**Islands**: Mostly static pages with pockets of interactivity

---

## Frameworks and Methodologies

### 1. Component Architecture — step-by-step
1. Identify UI patterns and decompose into reusable components
2. Separate presentational components (UI) from container components (logic)
3. Define component API (props, events, slots)
4. Implement state management: local state for components, shared state for cross-cutting
5. Create a component library/design system for consistency
6. Document and test components in isolation (Storybook)

### 2. Rendering Strategy Selection — step-by-step
1. Classify pages: static content, dynamic content, interactive app
2. Static content → SSG (build-time HTML)
3. SEO + dynamic data → SSR (per-request HTML)
4. Highly interactive → SPA/CSR (client-side rendering)
5. Mixed → Islands (static shell + interactive islands)
6. Use framework capabilities: Next.js, Nuxt, Astro support all strategies

---

## How to Apply

### Decision Checklist
- [ ] Is the rendering strategy appropriate for each page type?
- [ ] Is state management predictable (unidirectional data flow)?
- [ ] Are components small, reusable, and well-composed?
- [ ] Are Core Web Vitals monitored and optimized?
- [ ] Is code-splitting implemented for bundle size management?

### Implementation Patterns

**Feature-Based Folder Structure:**
```
src/
  features/
    orders/
      components/
        OrderList.tsx
        OrderDetail.tsx
      hooks/
        useOrders.ts
      api/
        orderApi.ts
      types.ts
      index.ts              # Public exports
    products/
      ...
  shared/
    components/             # Design system components
    hooks/                  # Shared hooks
    utils/                  # Shared utilities
  app/
    routes/                 # Page-level routing
    layout/                 # App shell, navigation
```

**State Management (Zustand — lightweight):**
```typescript
import { create } from 'zustand';

interface CartStore {
  items: CartItem[];
  addItem: (item: CartItem) => void;
  removeItem: (id: string) => void;
  total: () => number;
}

const useCartStore = create<CartStore>((set, get) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) => set((state) => ({
    items: state.items.filter((i) => i.id !== id),
  })),
  total: () => get().items.reduce((sum, item) => sum + item.price * item.qty, 0),
}));
```

### Common Mistakes
1. **SPA for everything** — Using SPA for content sites kills SEO and performance
2. **Prop drilling** — Passing props through 5 levels instead of using context/store
3. **Global state abuse** — Putting everything in Redux when local state suffices
4. **No code splitting** — Loading entire app upfront; use lazy loading for routes
5. **Ignoring accessibility** — No ARIA labels, keyboard navigation, or screen reader support

### Integration with Other Topics
- **Design Systems** — Component library enforces consistency
- **API Design** — Frontend consumes REST/GraphQL/tRPC APIs
- **TypeScript** — Type safety for components, props, and state
- **Testing** — Component testing (Testing Library), E2E testing (Playwright)
- **Performance** — Core Web Vitals optimization
- **BFF Pattern** — Backend optimized for frontend data needs

---

## Resources

### Essential Reading
- *Patterns.dev* — Addy Osmani & Lydia Hallie (free online)
- Next.js documentation (nextjs.org)
- React documentation (react.dev)

### Online Resources
- web.dev — Google's web performance guidance
- Astro documentation (Islands Architecture)
- Storybook documentation

### Practice Exercises
1. Structure a React/Next.js project with feature-based folder organization
2. Implement the same page with SSR, SSG, and CSR — compare performance
3. Set up Zustand/Jotai state management and compare with Redux
4. Optimize Core Web Vitals for a page (LCP, CLS, FID)
5. Build a component library with Storybook
