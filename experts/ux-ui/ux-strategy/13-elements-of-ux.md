# The Elements of User Experience

## Profile

### What It Is
The Elements of User Experience is a conceptual framework that breaks UX design into five interdependent planes, each building on the one below. Created by Jesse James Garrett, it provides a structured, bottom-up approach to designing digital products, ensuring that strategic decisions at the foundation inform every visual detail at the surface.

### Origin and Evolution
- Originated as a diagram published by Jesse James Garrett in 2000
- Formalized in the book *The Elements of User Experience* (2002, 2nd edition 2010)
- Initially conceived for web design, now applicable to all digital product design
- Bridged the gap between information architecture and visual design communities
- Influenced the professionalization of UX as a discipline separate from graphic design

### Key Authors and References
- **Jesse James Garrett** — *The Elements of User Experience: User-Centered Design for the Web and Beyond* (2002, 2nd ed. 2010)
- **Peter Morville & Louis Rosenfeld** — *Information Architecture for the World Wide Web* (2006)
- **Alan Cooper** — *About Face: The Essentials of Interaction Design* (2014)
- **Don Norman** — *The Design of Everyday Things* (1988, revised 2013)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Strategy Plane | Defines user needs and business objectives |
| Scope Plane | Translates strategy into functional specifications and content requirements |
| Structure Plane | Organizes interaction design and information architecture |
| Skeleton Plane | Arranges interface design, navigation design, information design |
| Surface Plane | Defines the final visual design users see and interact with |
| Bottom-Up Approach | Decisions on lower planes constrain and inform higher planes |
| Duality | Each plane has a task-oriented side and an information-oriented side |

---

## Core Philosophy

### The 5 Fundamental Principles

1. **Bottom-Up Dependency** — Each plane depends on the decisions made in the plane below it. You cannot design effective navigation (Skeleton) without first establishing information architecture (Structure), which itself relies on clear scope and strategy.

2. **Concrete to Abstract Continuum** — The framework moves from abstract concerns (strategy, user needs) to concrete outputs (pixels, colors, typography). This progression mirrors real project workflow and prevents surface-level decisions from overriding foundational thinking.

3. **Dual Nature of Experience** — Every plane operates on two dimensions: task-oriented (what users do) and information-oriented (what users consume). A complete UX addresses both interaction and content at every level.

4. **Finish Before Progressing** — While overlap between planes is natural, work on a higher plane should not begin until decisions on the plane below are substantially resolved. This prevents rework and misalignment.

5. **User-Centered at Every Layer** — User needs are not just a strategy concern. Each plane must be validated against real user behavior, from strategic research to usability testing of the final surface.

### When to Use vs. When to Avoid

**Use when:**
- Designing a new digital product from scratch and needing a structured process
- Auditing an existing product to identify which layer is causing UX problems
- Communicating UX decisions to stakeholders who need to see how strategy maps to design
- Training junior designers to think systematically about UX
- Planning sprints or phases around clear deliverables per plane

**Avoid over-application when:**
- Working on micro-interactions or single-component updates that do not require full-stack thinking
- Rapid prototyping where speed matters more than layered documentation
- Projects with extremely tight timelines where a lighter framework like Lean UX is more appropriate
- The team is already mature and works iteratively without needing rigid layering

---

## Frameworks and Methodologies

### 1. Strategy Plane Analysis — step-by-step
1. Conduct stakeholder interviews to define business objectives
2. Perform user research (interviews, surveys, analytics) to identify user needs
3. Create user personas that capture goals, frustrations, and contexts
4. Map business objectives against user needs to find overlap and tension
5. Document the strategy as a brief: "We help [users] achieve [goal] while driving [business metric]"

### 2. Scope Definition — step-by-step
1. Translate strategy into a feature inventory (functional specifications)
2. Define content requirements: what information must the product provide
3. Prioritize features using MoSCoW (Must, Should, Could, Won't)
4. Write user stories or use cases for each functional requirement
5. Validate scope against strategy: every feature must serve a documented user need or business objective

### 3. Structure Design — step-by-step
1. Define the interaction design model: how does the system respond to user actions
2. Map the information architecture: create a sitemap or content hierarchy
3. Define navigation pathways: primary, secondary, contextual
4. Create task flows and user flows for core scenarios
5. Test the structure with card sorting or tree testing

### 4. Skeleton Layout — step-by-step
1. Create wireframes for key pages, defining placement of interface elements
2. Design the navigation system: menus, breadcrumbs, tabs, links
3. Arrange information design elements: labels, groupings, visual priority cues
4. Define conventions: where do buttons go, how are forms laid out, where is feedback shown
5. Test wireframes with low-fidelity usability testing

### 5. Surface Execution — step-by-step
1. Establish a visual language: color palette, typography, spacing, iconography
2. Apply visual hierarchy to guide the eye through content
3. Ensure contrast and accessibility (WCAG compliance)
4. Design micro-interactions and transitions
5. Conduct visual design reviews and high-fidelity usability tests

---

## How to Apply

### Decision Checklist
- [ ] Have we documented both user needs and business objectives (Strategy)?
- [ ] Do all features trace back to a strategic requirement (Scope)?
- [ ] Is there a clear sitemap and defined interaction patterns (Structure)?
- [ ] Have wireframes been tested with users before visual design begins (Skeleton)?
- [ ] Does the visual design reinforce the hierarchy established in the skeleton (Surface)?
- [ ] Have we validated each plane with appropriate research methods?
- [ ] Is there documentation linking decisions across planes?

### Implementation Patterns

**Strategy-to-Scope Mapping Table:**
```html
<!-- Strategy-to-Scope traceability matrix -->
<table class="ux-traceability">
  <thead>
    <tr>
      <th>User Need</th>
      <th>Business Objective</th>
      <th>Feature (Scope)</th>
      <th>Priority</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Find products quickly</td>
      <td>Increase conversion rate</td>
      <td>Search with autocomplete</td>
      <td>Must Have</td>
    </tr>
    <tr>
      <td>Compare options easily</td>
      <td>Reduce bounce rate</td>
      <td>Side-by-side comparison</td>
      <td>Should Have</td>
    </tr>
  </tbody>
</table>
```

**Structure: Semantic HTML Reflecting IA:**
```html
<!-- Structure plane: HTML reflects information architecture -->
<body>
  <header role="banner">
    <nav role="navigation" aria-label="Primary">
      <!-- Primary navigation mirrors IA top-level categories -->
      <ul>
        <li><a href="/products">Products</a></li>
        <li><a href="/solutions">Solutions</a></li>
        <li><a href="/resources">Resources</a></li>
      </ul>
    </nav>
  </header>

  <main role="main">
    <nav aria-label="Breadcrumb">
      <!-- Breadcrumb reveals the IA hierarchy -->
      <ol>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li aria-current="page">Widget Pro</li>
      </ol>
    </nav>

    <article>
      <!-- Content hierarchy maps to Structure plane decisions -->
      <h1>Widget Pro</h1>
      <section aria-labelledby="overview">
        <h2 id="overview">Overview</h2>
      </section>
      <section aria-labelledby="specs">
        <h2 id="specs">Specifications</h2>
      </section>
    </article>
  </main>
</body>
```

**Skeleton: Wireframe Grid System:**
```css
/* Skeleton plane: layout defines spatial arrangement */
.page-skeleton {
  display: grid;
  grid-template-areas:
    "header  header  header"
    "nav     content sidebar"
    "footer  footer  footer";
  grid-template-columns: 200px 1fr 280px;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

.page-skeleton__header  { grid-area: header; }
.page-skeleton__nav     { grid-area: nav; }
.page-skeleton__content { grid-area: content; }
.page-skeleton__sidebar { grid-area: sidebar; }
.page-skeleton__footer  { grid-area: footer; }

/* Wireframe mode: strip visual design to focus on layout */
.wireframe-mode * {
  border: 1px solid #999;
  background: #f0f0f0;
  color: #333;
  font-family: 'Courier New', monospace;
  border-radius: 0;
  box-shadow: none;
}
```

**Surface: Visual Design Tokens:**
```css
/* Surface plane: design tokens define the visual language */
:root {
  /* Color */
  --color-primary: #2563eb;
  --color-primary-hover: #1d4ed8;
  --color-surface: #ffffff;
  --color-text-primary: #111827;
  --color-text-secondary: #6b7280;

  /* Typography */
  --font-family-heading: 'Inter', system-ui, sans-serif;
  --font-family-body: 'Inter', system-ui, sans-serif;
  --font-size-h1: 2.25rem;
  --font-size-body: 1rem;
  --line-height-body: 1.625;

  /* Spacing (8px base unit) */
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 1.5rem;
  --space-xl: 2rem;
}
```

### Common Mistakes

1. **Jumping to Surface** — Designers skip to visual design (colors, fonts) before defining strategy or structure, resulting in beautiful but unusable interfaces.

2. **Treating Planes as Waterfalled Phases** — The planes describe dependencies, not a rigid waterfall. Some overlap and iteration is expected; the key is that lower planes should stabilize before upper planes finalize.

3. **Ignoring the Information Side** — Teams focus exclusively on features (task side) and neglect content strategy and information architecture, leading to feature-rich but content-poor products.

4. **No Traceability Between Planes** — When a feature cannot be traced back to a strategic user need or business objective, it becomes scope creep. Maintain a traceability matrix.

5. **Confusing Skeleton and Surface** — Wireframes (Skeleton) define placement and priority. Visual design (Surface) defines appearance. Mixing them leads to debates about color during layout reviews.

### Integration with Other Topics
- **Design Thinking (14)** — Design Thinking's Empathize and Define phases feed directly into the Strategy plane; Ideate and Prototype map to Scope and Skeleton
- **Information Architecture (17)** — IA is a core component of the Structure plane, defining how content is organized and labeled
- **Visual Hierarchy (25)** — Visual hierarchy principles are applied at the Surface plane to guide user attention through the interface
- **Lean UX (15)** — Lean UX's hypothesis-driven approach can validate assumptions at each plane with minimal deliverables

---

## Resources

### Essential Reading
- *The Elements of User Experience* by Jesse James Garrett (2nd edition, 2010)
- *Information Architecture: For the Web and Beyond* by Rosenfeld, Morville & Arango (4th edition, 2015)
- *About Face: The Essentials of Interaction Design* by Alan Cooper et al. (4th edition, 2014)

### Online Resources
- Jesse James Garrett's original diagram: jjg.net/elements
- Nielsen Norman Group: articles on UX strategy and IA
- UX Collective on Medium: modern applications of the 5 planes
- A List Apart: foundational articles on web UX design

### Practice Exercises
1. **Plane Audit** — Take an existing website and document what you observe at each plane. Identify where breakdowns occur between planes.
2. **Strategy Brief** — For a hypothetical product, write a one-page strategy document that clearly states user needs, business objectives, and how they align.
3. **Traceability Matrix** — Create a spreadsheet mapping user needs to features to wireframe sections to visual design elements. Verify every surface decision has a strategic rationale.
4. **Bottom-Up Redesign** — Choose a poorly designed page. Redesign it starting strictly from strategy, moving up through each plane, documenting decisions at each level.
