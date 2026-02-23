# Laws of UX

## Profile

### What It Is
Laws of UX is a collection of psychological principles and design maxims that describe how users perceive and interact with digital interfaces. Compiled and popularized by designer Jon Yablonski, these laws distill decades of cognitive science research into actionable guidelines for UX practitioners building websites, apps, and digital products.

### Origin and Evolution
- Rooted in foundational research spanning from the 1950s (Fitts, Miller, Hick) through modern interaction design
- Jon Yablonski launched lawsofux.com in 2018, curating and visualizing the key principles
- Published as the book *Laws of UX* in 2020, bridging psychology and digital design practice
- Continuously updated as new research and design patterns emerge in modern web and mobile contexts

### Key Authors and References
- **Jon Yablonski** — *Laws of UX: Using Psychology to Design Better Products & Services* (2020)
- **Paul Fitts** — Fitts's Law research on motor movement (1954)
- **William Edmund Hick & Ray Hyman** — Hick-Hyman Law on decision time (1952)
- **George A. Miller** — *The Magical Number Seven, Plus or Minus Two* (1956)
- **Jakob Nielsen** — Jakob's Law and usability heuristics (2000)
- **Hedwig von Restorff** — Isolation Effect research (1933)
- **Bluma Zeigarnik** — Zeigarnik Effect on incomplete tasks (1927)

### Key Concepts at a Glance
| Law | Core Insight |
|-----|-------------|
| Fitts's Law | Larger, closer targets are faster to reach |
| Hick's Law | More choices = longer decision time |
| Jakob's Law | Users prefer familiar patterns |
| Miller's Law | Working memory holds ~7 items |
| Von Restorff Effect | Distinct items are remembered |
| Zeigarnik Effect | Incomplete tasks stick in memory |
| Doherty Threshold | Keep response time under 400ms |
| Postel's Law | Accept broadly, output strictly |
| Serial Position Effect | First and last items recalled best |
| Peak-End Rule | Experiences judged by peak and ending |

---

## Core Philosophy

### The 5 Fundamental Principles

1. **Reduce Friction** — Every law converges on one goal: minimize the cognitive, motor, and emotional effort a user must expend. Fitts's Law optimizes physical targeting, Hick's Law reduces decision overhead, and the Doherty Threshold eliminates waiting friction.

2. **Leverage Familiarity** — Humans are pattern-matching creatures. Jakob's Law and the Serial Position Effect remind us that users arrive with expectations formed by every other product they have used. Working with those expectations, rather than against them, accelerates comprehension.

3. **Respect Cognitive Limits** — Miller's Law and cognitive load research make clear that working memory is a scarce resource. Chunk information, progressively disclose complexity, and never force users to hold more in their heads than necessary.

4. **Design for Attention and Memory** — The Von Restorff Effect, Serial Position Effect, and Peak-End Rule describe how memory works. Use visual emphasis strategically, place critical items first or last, and ensure the most emotional moments of an experience are positive.

5. **Be Tolerant and Responsive** — Postel's Law and the Doherty Threshold remind designers to accept imperfect input gracefully and respond quickly. Forgiveness in input handling and speed in feedback create a sense of competence and trust.

### When to Use vs. When to Avoid

**Use when:**
- Designing navigation, menus, and information architecture
- Building forms, checkout flows, or any multi-step process
- Evaluating designs via heuristic review or usability audit
- Prioritizing features and content on a page
- Optimizing perceived performance and interaction responsiveness

**Avoid over-application when:**
- Blindly reducing options removes necessary complexity (Hick's Law taken to extreme)
- Making everything look the same eliminates meaningful visual emphasis (Von Restorff misapplied)
- Copying competitor patterns verbatim conflicts with brand or use-case needs (Jakob's Law misread)
- Oversimplifying to 7 items when the context supports scanning larger sets (Miller's Law misunderstood as a hard cap)

---

## Frameworks and Methodologies

### 1. UX Law Audit Framework — step-by-step
1. **Inventory** — List every screen, flow, and major interaction in the product
2. **Map Laws** — For each screen, identify which laws are most relevant (e.g., a navigation bar involves Jakob's Law, Serial Position Effect, Hick's Law)
3. **Evaluate** — Score each screen against the relevant laws (pass/partial/fail)
4. **Prioritize** — Rank issues by impact (high-traffic screens first) and effort
5. **Redesign** — Apply the law-based recommendations to create improved designs
6. **Validate** — A/B test or usability test the redesigned flow to confirm improvement

### 2. The 10-Law Checklist for Page Design — step-by-step
1. Are interactive targets large enough and close to related actions? (Fitts's Law)
2. Are choices limited to a manageable number per decision point? (Hick's Law)
3. Does the layout follow conventions users already know? (Jakob's Law)
4. Is information chunked into groups of 5-9 items or fewer? (Miller's Law)
5. Does the most important element visually stand out? (Von Restorff Effect)
6. Are progress indicators present for multi-step tasks? (Zeigarnik Effect)
7. Does the interface respond within 400ms? (Doherty Threshold)
8. Does the system accept flexible input formats? (Postel's Law)
9. Are the most critical items placed first or last? (Serial Position Effect)
10. Does the experience end on a positive note? (Peak-End Rule)

---

## How to Apply

### Decision Checklist
- [ ] Touch and click targets meet minimum 44x44px (Fitts's Law)
- [ ] Navigation presents no more than 7 primary items (Hick's + Miller's)
- [ ] Layout follows established conventions for the product category (Jakob's Law)
- [ ] Long lists are chunked or categorized into digestible groups (Miller's Law)
- [ ] The primary CTA is visually distinct from surrounding elements (Von Restorff)
- [ ] Multi-step flows show progress indication (Zeigarnik Effect)
- [ ] Page load and interaction feedback occur within 400ms (Doherty Threshold)
- [ ] Form inputs accept multiple valid formats (Postel's Law)
- [ ] Key actions are positioned at the start or end of lists/menus (Serial Position)
- [ ] The final step of a flow provides positive reinforcement (Peak-End Rule)

### Implementation Patterns

**Fitts's Law — Accessible Touch Targets:**
```css
/* Ensure minimum touch target size per WCAG and Apple HIG */
.button,
.nav-link,
.interactive-element {
  min-width: 44px;
  min-height: 44px;
  padding: 12px 16px;
  /* Increase target area without visual bloat */
  position: relative;
}

/* Invisible tap area extension for small icons */
.icon-button::after {
  content: '';
  position: absolute;
  top: -8px;
  right: -8px;
  bottom: -8px;
  left: -8px;
}
```

**Hick's Law — Reducing Navigation Choices:**
```html
<!-- BAD: 15 top-level items overwhelm users -->
<nav>
  <a href="/home">Home</a>
  <a href="/about">About</a>
  <a href="/services">Services</a>
  <a href="/portfolio">Portfolio</a>
  <a href="/team">Team</a>
  <a href="/blog">Blog</a>
  <a href="/news">News</a>
  <a href="/events">Events</a>
  <a href="/careers">Careers</a>
  <a href="/faq">FAQ</a>
  <a href="/support">Support</a>
  <a href="/contact">Contact</a>
  <!-- ... -->
</nav>

<!-- GOOD: 5 primary items, rest grouped under secondary navigation -->
<nav class="primary-nav">
  <a href="/home">Home</a>
  <a href="/services">Services</a>
  <a href="/portfolio">Work</a>
  <a href="/blog">Insights</a>
  <a href="/contact">Contact</a>
</nav>
```

**Von Restorff Effect — Visual Emphasis for CTAs:**
```css
/* Make the primary action visually distinct */
.btn-secondary {
  background-color: transparent;
  border: 1px solid #6b7280;
  color: #374151;
}

.btn-primary {
  background-color: #2563eb;
  border: 1px solid #2563eb;
  color: #ffffff;
  box-shadow: 0 2px 8px rgba(37, 99, 235, 0.3);
  font-weight: 600;
  transform: scale(1.02);
}

/* Pricing table: featured plan stands out */
.pricing-card { background: #ffffff; border: 1px solid #e5e7eb; }
.pricing-card--featured {
  background: #eff6ff;
  border: 2px solid #2563eb;
  transform: scale(1.05);
  box-shadow: 0 8px 24px rgba(37, 99, 235, 0.15);
}
```

**Doherty Threshold — Perceived Performance:**
```css
/* Skeleton screen to create instant response perception */
.skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: skeleton-pulse 1.5s ease-in-out infinite;
  border-radius: 4px;
}

@keyframes skeleton-pulse {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

```javascript
// Optimistic UI: update immediately, reconcile with server after
function handleLikeClick(postId) {
  // Instant visual feedback (< 400ms perceived)
  updateUIOptimistically(postId, { liked: true, count: count + 1 });

  // Actual API call in background
  api.likePost(postId).catch(() => {
    // Rollback on failure
    updateUIOptimistically(postId, { liked: false, count: count });
    showToast('Could not save your like. Please try again.');
  });
}
```

**Postel's Law — Flexible Form Inputs:**
```javascript
// Accept phone numbers in any format, normalize on output
function normalizePhone(input) {
  const digits = input.replace(/\D/g, '');
  if (digits.length === 10) {
    return `(${digits.slice(0,3)}) ${digits.slice(3,6)}-${digits.slice(6)}`;
  }
  return input; // Return as-is if unrecognized
}

// Accept dates in multiple formats
function parseFlexibleDate(input) {
  const formats = [
    /(\d{4})-(\d{2})-(\d{2})/,        // 2024-01-15
    /(\d{2})\/(\d{2})\/(\d{4})/,        // 01/15/2024
    /(\d{2})\.(\d{2})\.(\d{4})/,        // 15.01.2024
  ];
  for (const fmt of formats) {
    if (fmt.test(input)) return new Date(input);
  }
  return null;
}
```

**Serial Position Effect — Navigation Placement:**
```html
<!-- Place most important items first and last in navigation -->
<nav class="bottom-nav">
  <a href="/home" class="nav-item nav-item--primary">Home</a>  <!-- First: primacy -->
  <a href="/search" class="nav-item">Search</a>
  <a href="/favorites" class="nav-item">Favorites</a>
  <a href="/cart" class="nav-item">Cart</a>
  <a href="/profile" class="nav-item nav-item--primary">Profile</a>  <!-- Last: recency -->
</nav>
```

### Common Mistakes
1. **Treating Miller's 7 as a Hard Rule** — Miller described working memory capacity in a specific context. It is a guideline for chunking, not a maximum item count for all lists. Scannable lists can exceed 7 items if well-organized.
2. **Over-reducing Options (Hick's Law)** — Removing necessary choices increases task complexity by forcing users to search elsewhere. Reduce choices per decision point, not total available options.
3. **Copying Patterns Without Context (Jakob's Law)** — Users expect conventions from similar products, not all products. A medical dashboard and a social media feed have different conventions.
4. **Ignoring the Peak-End Rule in Error States** — If a user's last interaction is a confusing error page, that defines their memory of the experience regardless of earlier quality.
5. **Applying Laws in Isolation** — The laws interact. For example, making a CTA stand out (Von Restorff) but placing it in the middle of a list (against Serial Position) weakens both effects.

### Integration with Other Topics
- **05-usability-heuristics** — Nielsen's 10 heuristics overlap significantly; Laws of UX provides the psychological research underpinning heuristics like "recognition rather than recall" (Miller's Law) and "error prevention" (Postel's Law)
- **08-psychology-for-designers** — Broader cognitive psychology context for each law; dual-process theory, attention economics, and behavioral design patterns
- **11-cognitive-load-theory** — Miller's Law and Hick's Law are directly tied to cognitive load management; intrinsic vs. extraneous load maps to inherent complexity vs. poor design
- **12-gestalt-principles** — Gestalt principles of proximity and similarity support the chunking strategies prescribed by Miller's Law
- **25-visual-hierarchy** — Von Restorff Effect and Serial Position Effect are visual hierarchy tools; size, color, contrast, and position create the emphasis these laws prescribe

---

## Resources

### Essential Reading
- Jon Yablonski, *Laws of UX: Using Psychology to Design Better Products & Services* (O'Reilly, 2020)
- Susan Weinschenk, *100 Things Every Designer Needs to Know About People* (New Riders, 2011)
- Don Norman, *The Design of Everyday Things* (Basic Books, revised 2013)
- Daniel Kahneman, *Thinking, Fast and Slow* (Farrar, Straus and Giroux, 2011)

### Online Resources
- lawsofux.com — Jon Yablonski's curated collection with visual cards for each law
- Nielsen Norman Group (nngroup.com) — Research articles on Fitts's Law, Hick's Law, Jakob's Law
- Interaction Design Foundation — UX psychology courses and law-specific articles
- Google Material Design Guidelines — Practical application of many laws (touch targets, response time)

### Practice Exercises
1. **Navigation Audit**: Take a live website and evaluate its primary navigation against all 10 laws. Document which laws are satisfied and which are violated, then redesign the navigation.
2. **Touch Target Review**: Open a mobile app and measure (using developer tools or screenshots) whether all interactive elements meet the 44x44px minimum. Document violations and propose fixes.
3. **Checkout Flow Teardown**: Map a multi-step checkout process and identify where each law applies. Score the flow, then redesign the weakest steps focusing on Zeigarnik Effect (progress), Doherty Threshold (speed), and Peak-End Rule (confirmation).
4. **A/B Test Design**: Create two versions of a form — one with 12 fields on a single page, another chunked into 3 groups of 4. Predict which performs better using Hick's Law and Miller's Law, then test.
5. **Emphasis Audit**: Take a landing page and identify what visually stands out. Does the most prominent element match the primary user goal? Apply Von Restorff to fix misaligned emphasis.
