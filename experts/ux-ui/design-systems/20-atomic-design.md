# Atomic Design

## Profile

### What It Is
Atomic Design is a methodology for creating design systems composed of five distinct hierarchical levels: atoms, molecules, organisms, templates, and pages. Inspired by chemistry, it provides a mental model for building user interfaces from the smallest indivisible elements up to complete, content-rich screens. It bridges the gap between abstract design components and the final user-facing product.

### Origin and Evolution
- **2013** — Brad Frost introduces the Atomic Design concept in a blog post, borrowing the chemistry metaphor to describe UI composition
- **2015** — Pattern Lab, an open-source tool for building Atomic Design systems, gains traction in front-end communities
- **2016** — Brad Frost publishes *Atomic Design* (book), available free at atomicdesign.bradfrost.com, cementing the methodology
- **2018–present** — Atomic Design principles become the de facto mental model behind component-based frameworks (React, Vue, Svelte) and enterprise design systems (IBM Carbon, Shopify Polaris)

### Key Authors and References
- **Brad Frost** — *Atomic Design* (2016, atomicdesign.bradfrost.com)
- **Brad Frost & Dave Olsen** — Pattern Lab (open-source pattern-driven UI tool)
- **Dan Mall** — *Design System Handbook* contributions; frequent collaborator with Frost on design system consulting

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Atoms | The smallest UI building blocks: HTML tags like buttons, inputs, labels, icons |
| Molecules | Simple groups of atoms functioning as a unit (e.g., a search form) |
| Organisms | Complex, distinct sections of an interface composed of molecules and atoms |
| Templates | Page-level wireframes that define content structure and layout without real data |
| Pages | Specific template instances populated with real representative content |
| Interface Inventory | An audit cataloging every unique UI element across an existing product |
| Pattern Lab | A static site generator for building and documenting Atomic Design systems |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Build from the bottom up** — Start with the smallest elements and compose upward, ensuring every piece is well-defined before combining.
2. **Everything is a component** — Every visual element on screen maps to a reusable, testable unit at one of the five levels.
3. **Context independence** — Atoms and molecules should work in any context; organisms begin to assume layout context.
4. **Simultaneous top-down and bottom-up thinking** — While you build bottom-up, you design top-down: pages inform which organisms, molecules, and atoms are needed.
5. **Living documentation** — The component library is the deliverable, not static mockups. It evolves with the product.

### When to Use vs. When to Avoid

**Use when:**
- Building a component library or design system for a medium-to-large product
- Working across multiple teams that need a shared UI vocabulary
- Migrating a legacy interface to a modern component-based framework
- Conducting an interface inventory to reduce design inconsistency

**Avoid over-application when:**
- The project is a quick prototype or throwaway proof-of-concept
- The team is very small (1-2 people) and a lighter naming convention suffices
- You are forcing every element into the chemistry metaphor at the expense of clarity

---

## Frameworks and Methodologies

### 1. The Interface Inventory Process — step-by-step
1. Take screenshots of every unique UI element across all product screens
2. Group screenshots by category: typography, buttons, forms, navigation, images, icons, media, lists, interactive widgets
3. Identify duplicates and near-duplicates within each category
4. Consolidate variations — decide which to keep, merge, or remove
5. Name each surviving element and assign it to an Atomic Design level
6. Document findings as the seed for the component library

### 2. Mapping Atomic Design to a Component Library — step-by-step
1. Define atoms as the lowest-level presentational components (no business logic)
2. Compose molecules by combining 2-3 atoms into small functional groups
3. Build organisms by arranging molecules and atoms into recognizable UI sections
4. Create templates as layout components that accept organisms as children
5. Assemble pages by passing real data into templates for QA and stakeholder review

---

## How to Apply

### Decision Checklist
- [ ] Have I cataloged all existing UI elements (interface inventory)?
- [ ] Can I identify the atoms — elements that cannot be broken down further?
- [ ] Are my molecules truly simple groupings of 2-4 atoms with a single purpose?
- [ ] Do my organisms represent distinct, recognizable sections of the UI?
- [ ] Do templates define layout without coupling to specific data?
- [ ] Are pages populated with realistic content for stakeholder review?
- [ ] Is each component documented with usage guidelines and prop definitions?
- [ ] Does the naming convention communicate hierarchy clearly to the team?

### Implementation Patterns

**Atom — Button Component:**
```jsx
// atoms/Button.jsx
export function Button({ variant = 'primary', size = 'md', children, ...props }) {
  return (
    <button
      className={`btn btn--${variant} btn--${size}`}
      {...props}
    >
      {children}
    </button>
  );
}
```

**Atom — Input Component:**
```jsx
// atoms/Input.jsx
export function Input({ label, id, type = 'text', ...props }) {
  return (
    <div className="input-field">
      <label htmlFor={id}>{label}</label>
      <input id={id} type={type} {...props} />
    </div>
  );
}
```

**Molecule — Search Form (atoms combined):**
```jsx
// molecules/SearchForm.jsx
import { Input } from '../atoms/Input';
import { Button } from '../atoms/Button';

export function SearchForm({ onSubmit }) {
  return (
    <form className="search-form" onSubmit={onSubmit} role="search">
      <Input id="search" label="Search" type="search" placeholder="Search..." />
      <Button type="submit" variant="primary">Search</Button>
    </form>
  );
}
```

**Organism — Site Header (molecules + atoms):**
```jsx
// organisms/SiteHeader.jsx
import { Logo } from '../atoms/Logo';
import { SearchForm } from '../molecules/SearchForm';
import { NavMenu } from '../molecules/NavMenu';

export function SiteHeader({ navItems, onSearch }) {
  return (
    <header className="site-header">
      <Logo />
      <NavMenu items={navItems} />
      <SearchForm onSubmit={onSearch} />
    </header>
  );
}
```

**Organism — Product Card Grid:**
```jsx
// organisms/ProductCardGrid.jsx
import { ProductCard } from '../molecules/ProductCard';

export function ProductCardGrid({ products }) {
  return (
    <section className="product-grid">
      {products.map(product => (
        <ProductCard key={product.id} {...product} />
      ))}
    </section>
  );
}
```

**Template — Catalog Page Layout:**
```jsx
// templates/CatalogTemplate.jsx
import { SiteHeader } from '../organisms/SiteHeader';
import { ProductCardGrid } from '../organisms/ProductCardGrid';
import { SiteFooter } from '../organisms/SiteFooter';
import { Sidebar } from '../organisms/Sidebar';

export function CatalogTemplate({ header, sidebar, products, footer }) {
  return (
    <div className="catalog-layout">
      <SiteHeader {...header} />
      <div className="catalog-layout__body">
        <Sidebar {...sidebar} />
        <main className="catalog-layout__content">
          <ProductCardGrid products={products} />
        </main>
      </div>
      <SiteFooter {...footer} />
    </div>
  );
}
```

**Page — Specific Instance with Real Content:**
```jsx
// pages/SummerCatalogPage.jsx
import { CatalogTemplate } from '../templates/CatalogTemplate';
import { useSummerProducts } from '../hooks/useSummerProducts';

export function SummerCatalogPage() {
  const products = useSummerProducts();
  return (
    <CatalogTemplate
      header={{ navItems: ['Home', 'Shop', 'About'], onSearch: handleSearch }}
      sidebar={{ filters: ['T-Shirts', 'Shorts', 'Sandals'] }}
      products={products}
      footer={{ year: 2026, links: ['Privacy', 'Terms'] }}
    />
  );
}
```

**Corresponding HTML structure (atoms to organisms):**
```html
<!-- Atom -->
<button class="btn btn--primary btn--md">Add to Cart</button>

<!-- Molecule -->
<form class="search-form" role="search">
  <div class="input-field">
    <label for="search">Search</label>
    <input id="search" type="search" placeholder="Search..." />
  </div>
  <button class="btn btn--primary" type="submit">Search</button>
</form>

<!-- Organism -->
<header class="site-header">
  <a href="/" class="logo">Brand</a>
  <nav class="nav-menu">
    <a href="/shop">Shop</a>
    <a href="/about">About</a>
  </nav>
  <form class="search-form" role="search">
    <!-- molecule content -->
  </form>
</header>
```

### Common Mistakes
1. **Over-classifying — forcing the metaphor** — Not every element neatly fits atoms/molecules/organisms. Use the model as a guide, not a rigid taxonomy.
2. **Skipping the interface inventory** — Jumping straight to building atoms without auditing existing UI leads to incomplete or redundant components.
3. **Making atoms too smart** — Atoms should be presentational only. Business logic belongs at the organism or page level.
4. **Ignoring templates** — Teams often jump from organisms to pages. Templates are crucial for separating layout concerns from content.
5. **Flat file structures** — Dumping all components in one folder defeats the hierarchy. Mirror the five levels in your directory structure.

### Integration with Other Topics
- **Design Systems (21)** — Atomic Design provides the structural methodology; a design system adds tokens, documentation, governance, and distribution on top.
- **Material Design (22)** — Material components map to atoms and molecules in the Atomic Design hierarchy; Material layout grids inform templates.
- **Visual Hierarchy (25)** — Atomic Design structures *what* components exist; visual hierarchy principles determine *how* those components guide the user's eye.

---

## Resources

### Essential Reading
- Brad Frost — *Atomic Design* (2016), free at [atomicdesign.bradfrost.com](https://atomicdesign.bradfrost.com)
- Dan Mall — *Design System Handbook* (InVision)
- Brad Frost — "Interface Inventory" blog post (bradfrost.com/blog)

### Online Resources
- [Pattern Lab](https://patternlab.io/) — Official Atomic Design tooling
- [Storybook](https://storybook.js.org/) — Modern alternative for documenting component hierarchies
- [Component Gallery](https://component.gallery/) — Real-world design system component examples

### Practice Exercises
1. **Interface inventory audit:** Pick an existing website (your own or a public one), screenshot every unique UI element, and classify each as atom, molecule, or organism.
2. **Bottom-up build:** Using only HTML and CSS, build a set of 5 atoms, combine them into 3 molecules, and assemble 2 organisms. Document each level.
3. **React component tree:** Implement a blog homepage using the full five-level hierarchy in React. Start from atoms (`<Heading>`, `<Avatar>`, `<Tag>`) and build up to a `<BlogHomePage>` page component.
4. **Pattern Lab workshop:** Set up Pattern Lab and recreate a simple product page, using the mustache/twig templates to separate data from structure at the template and page levels.
