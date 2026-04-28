# WCAG — Web Content Accessibility Guidelines

## Profile

### What It Is
The Web Content Accessibility Guidelines (WCAG) are the internationally recognized standard for making web content accessible to people with disabilities. Developed by the W3C Web Accessibility Initiative (WAI), WCAG provides a shared, testable framework organized around four principles — Perceivable, Operable, Understandable, and Robust (POUR) — with three conformance levels (A, AA, AAA). WCAG 2.2, the current version, is referenced by accessibility legislation worldwide and serves as the foundation for inclusive web design.

### Origin and Evolution
- **1999** — WCAG 1.0 published as a W3C Recommendation, focusing on HTML-specific checkpoints
- **2008** — WCAG 2.0 released, shifting to technology-agnostic, testable success criteria organized under four principles
- **2012** — WCAG 2.0 adopted as ISO/IEC 40500 international standard
- **2018** — WCAG 2.1 adds 17 new success criteria addressing mobile, low vision, and cognitive disabilities
- **2023** — WCAG 2.2 published, adding 9 new success criteria including focus appearance, dragging movements, and target size enhancements
- **In progress** — WCAG 3.0 (Silver) being drafted with a fundamentally different scoring model and broader scope

### Key Authors and References
- **W3C Web Accessibility Initiative** — *WCAG 2.2* (2023)
- **Shawn Lawton Henry** — *Just Ask: Integrating Accessibility Throughout Design* (2007)
- **Karl Groves & Billy Gregory** — *Accessibility advocacy and WCAG education* (2010s-present)
- **Deque Systems** — *axe accessibility testing engine and WCAG documentation* (2015-present)
- **Steve Faulkner** — *HTML accessibility API mappings and WCAG technique contributions* (2010s-present)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| POUR | The four principles: Perceivable, Operable, Understandable, Robust |
| Success Criteria | Testable statements that define whether content meets a guideline |
| Conformance Level A | Minimum accessibility — removes the most severe barriers |
| Conformance Level AA | Standard requirement — the level referenced by most legislation |
| Conformance Level AAA | Enhanced accessibility — maximum inclusiveness for the broadest audience |
| Sufficient Techniques | Documented methods that satisfy a success criterion |
| Advisory Techniques | Recommended but not required methods that enhance accessibility |
| Failures | Documented patterns that violate a success criterion |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Perceivable content for all senses** — Information must be presentable in ways that every user can perceive, whether through sight, hearing, or touch; if one channel is unavailable, alternatives must exist
2. **Operable by any input method** — Every function must be usable via keyboard, voice, switch, pointer, and other input devices; no interaction should depend on a single modality
3. **Understandable language and behavior** — Content must be readable, interfaces must behave predictably, and users must receive help when they make errors
4. **Robust enough for all technologies** — Content must be compatible with current and future assistive technologies through proper semantics and standards compliance
5. **Progressive conformance over perfection** — Accessibility is a spectrum; start with Level A, achieve Level AA, and pursue Level AAA where feasible rather than treating it as all-or-nothing

### When to Use vs. When to Avoid

**Use when:**
- Building or redesigning any public-facing web content
- Developing internal tools used by diverse workforces
- Evaluating third-party components or design systems for compliance
- Responding to legal accessibility requirements (ADA, EAA, Section 508, EN 301 549)
- Setting acceptance criteria for user stories that involve UI changes

**Avoid over-application when:**
- Pursuing AAA conformance on every page would significantly degrade the experience for the majority of users (target AAA selectively)
- Automated testing alone is treated as sufficient — WCAG compliance requires manual and assistive-technology testing
- Rigid adherence to contrast ratios overrides brand requirements without exploring alternatives like adjustable themes

---

## Frameworks and Methodologies

### 1. POUR Audit Framework — step-by-step
1. **Perceivable** — Verify that every non-text element has a text alternative, video has captions and audio descriptions, content is adaptable to different presentations, and color contrast meets minimum ratios
2. **Operable** — Verify all functionality via keyboard alone, confirm no time limits without extension options, check for seizure-inducing content, validate navigation mechanisms and heading structure
3. **Understandable** — Check language declaration, verify consistent navigation and identification, confirm error prevention and input assistance on forms
4. **Robust** — Validate HTML, test with at least two screen readers, verify ARIA usage, check name/role/value of custom components

### 2. Contrast Ratio Requirements — quick reference
1. **Normal text (under 18pt / 14pt bold)** — 4.5:1 minimum for AA, 7:1 for AAA
2. **Large text (18pt+ / 14pt+ bold)** — 3:1 minimum for AA, 4.5:1 for AAA
3. **UI components and graphical objects** — 3:1 minimum against adjacent colors (AA)
4. **Focus indicators** — 3:1 contrast against the unfocused state and adjacent background (WCAG 2.2)

### 3. WCAG 2.2 New Success Criteria — what changed
1. **2.4.11 Focus Not Obscured (Minimum) (AA)** — The focused element must not be entirely hidden by other page content
2. **2.4.12 Focus Not Obscured (Enhanced) (AAA)** — No part of the focused element is hidden by author-created content
3. **2.4.13 Focus Appearance (AAA)** — Focus indicator must have sufficient area and contrast change
4. **2.5.7 Dragging Movements (AA)** — Any action achievable by dragging must also be achievable by a single pointer without dragging
5. **2.5.8 Target Size (Minimum) (AA)** — Interactive targets must be at least 24x24 CSS pixels, or have sufficient spacing
6. **3.2.6 Consistent Help (A)** — Help mechanisms appear in the same relative order across pages
7. **3.3.7 Redundant Entry (A)** — Information previously entered should be auto-populated or selectable
8. **3.3.8 Accessible Authentication (Minimum) (AA)** — No cognitive function test for authentication unless an alternative or mechanism is provided
9. **3.3.9 Accessible Authentication (Enhanced) (AAA)** — No cognitive function test at all for authentication

---

## How to Apply

### Decision Checklist
- [ ] Do all images have meaningful alt text (or alt="" for decorative images)?
- [ ] Do all videos have synchronized captions and audio descriptions where needed?
- [ ] Does normal text meet 4.5:1 contrast ratio and large text meet 3:1?
- [ ] Can every interactive element be reached and activated via keyboard?
- [ ] Is there a visible focus indicator on every focusable element?
- [ ] Does the page have a logical heading hierarchy (h1-h6)?
- [ ] Are forms fully labeled with associated error messages?
- [ ] Is the page language declared in the html element?
- [ ] Does the site work with at least two screen readers?
- [ ] Are all interactive targets at least 24x24 CSS pixels?
- [ ] Is drag functionality also available via click/tap alternatives?

### Implementation Patterns

**Text alternatives for images:**
```html
<!-- Informative image — describe its content -->
<img src="quarterly-chart.png"
     alt="Bar chart showing Q4 revenue up 23% compared to Q3, reaching $4.2M" />

<!-- Decorative image — empty alt to hide from assistive technology -->
<img src="decorative-swirl.svg" alt="" />

<!-- Complex image — link to full description -->
<figure>
  <img src="system-architecture.png"
       alt="System architecture diagram. Full description below." />
  <figcaption>
    <details>
      <summary>Full architecture description</summary>
      <p>The system consists of three tiers: a React frontend communicates
         with a Node.js API layer, which connects to a PostgreSQL database...</p>
    </details>
  </figcaption>
</figure>
```

**Color contrast and non-color indicators:**
```css
/* Meeting AA contrast for normal text: 4.5:1 ratio */
.body-text {
  color: #1f2937;           /* gray-800 */
  background-color: #ffffff; /* ratio: 15.4:1 — passes AA and AAA */
}

/* Meeting AA contrast for large text: 3:1 ratio */
.heading-large {
  font-size: 1.5rem;
  font-weight: 700;
  color: #4b5563;           /* gray-600 on white — ratio: 5.9:1 */
}

/* Error state — never rely on color alone */
.input-error {
  border: 2px solid #dc2626;
  border-left: 4px solid #dc2626; /* Visual weight indicator */
  background-color: #fef2f2;
}

/* Pair with an icon and text message */
.error-message {
  color: #991b1b;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.error-message::before {
  content: "\26A0"; /* Warning symbol — not relying on color alone */
  font-size: 1.2em;
}
```

**Target size — meeting WCAG 2.2 minimum:**
```css
/* 2.5.8 Target Size (Minimum): 24x24 CSS pixels */
.icon-button {
  min-width: 44px;    /* Recommended: 44px for comfortable touch */
  min-height: 44px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 10px;
}

/* If targets are smaller than 24px, ensure sufficient spacing */
.compact-action-list a {
  display: inline-block;
  padding: 4px 8px;
  margin: 4px;       /* Spacing creates non-overlapping 24px target area */
}
```

**Dragging alternative — meeting 2.5.7:**
```html
<!-- Sortable list with both drag and button controls -->
<ul role="list" aria-label="Task priority">
  <li draggable="true" class="sortable-item">
    <span class="item-label">Review pull request</span>
    <div class="reorder-controls">
      <button aria-label="Move Review pull request up"
              onclick="moveUp(this)">&#9650;</button>
      <button aria-label="Move Review pull request down"
              onclick="moveDown(this)">&#9660;</button>
    </div>
  </li>
</ul>
```

**Accessible focus indicators:**
```css
/* Default visible focus — meets 2.4.7 (AA) */
:focus-visible {
  outline: 3px solid #2563eb;
  outline-offset: 2px;
}

/* Enhanced focus appearance — approaching 2.4.13 (AAA) */
:focus-visible {
  outline: 3px solid #1e40af;  /* 3:1 contrast against surrounding bg */
  outline-offset: 2px;
  box-shadow: 0 0 0 6px rgba(37, 99, 235, 0.25);
}

/* Remove default only when :focus-visible is supported */
:focus:not(:focus-visible) {
  outline: none;
}
```

### Common Mistakes
1. **Testing only with automated tools** — Automated scanners like axe and Lighthouse catch roughly 30-40% of WCAG issues; keyboard navigation, reading order, and meaningful alt text require manual testing
2. **Treating accessibility as a final QA step** — Retrofitting accessibility is 10x more expensive than building it in from the start; integrate checks into design reviews and CI pipelines
3. **Using color as the sole indicator** — Links distinguished only by color, errors shown only in red, and status conveyed only through color all fail Guideline 1.4.1 (Use of Color)
4. **Misunderstanding alt text** — Writing "image of" or "picture of" is redundant (screen readers already announce it as an image); over-describing decorative images adds noise
5. **Applying ARIA where native HTML suffices** — Using `role="button"` on a `<div>` instead of a `<button>` element; the first rule of ARIA is "do not use ARIA if you can use native HTML"
6. **Ignoring keyboard focus order** — Visual layout (CSS Grid, Flexbox order) can diverge from DOM order, creating a confusing tab sequence for keyboard users

### Integration with Other Topics
- **33-practical-accessibility** — WCAG provides the "what" (success criteria); practical accessibility covers the "how" (implementation patterns with semantic HTML and ARIA)
- **26-color-theory** — Color contrast ratios from WCAG directly constrain palette choices; accessible color systems must be designed with 4.5:1 and 3:1 ratios as inputs
- **29-form-design** — WCAG success criteria 1.3.1, 3.3.1, 3.3.2, and 3.3.3 define the accessibility requirements for labels, error identification, and instructions in forms

---

## Resources

### Essential Reading
- W3C WAI. (2023). *Web Content Accessibility Guidelines (WCAG) 2.2*. W3C Recommendation.
- W3C WAI. (2023). *How to Meet WCAG (Quick Reference)*. Customizable checklist.
- Henry, S.L. (2007). *Just Ask: Integrating Accessibility Throughout Design*. Lulu.com.
- Kalbag, L. (2017). *Accessibility for Everyone*. A Book Apart.

### Online Resources
- [WCAG 2.2 Specification](https://www.w3.org/TR/WCAG22/) — The complete normative standard
- [How to Meet WCAG (Quick Reference)](https://www.w3.org/WAI/WCAG22/quickref/) — Filterable checklist of all success criteria with techniques
- [Understanding WCAG 2.2](https://www.w3.org/WAI/WCAG22/Understanding/) — Detailed explanations and intent for each success criterion
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/) — Quick contrast ratio calculator
- [axe DevTools](https://www.deque.com/axe/) — Browser extension for automated accessibility testing
- [WAVE Evaluation Tool](https://wave.webaim.org/) — Visual accessibility evaluation overlay
- [Google Lighthouse](https://developer.chrome.com/docs/lighthouse/) — Built-in Chrome auditing with accessibility scoring

### Practice Exercises
1. **POUR audit**: Select a page on your current project. Walk through each POUR principle and document every failure with the specific WCAG success criterion it violates and the conformance level
2. **Contrast inventory**: Extract every text color and background color combination from your design system. Run each pair through a contrast checker and flag any that fail AA for their text size
3. **Keyboard walkthrough**: Unplug your mouse and navigate your application using only Tab, Shift+Tab, Enter, Space, Escape, and arrow keys. Document every point where you get stuck, lose focus, or cannot complete a task
4. **Screen reader session**: Enable VoiceOver (macOS) or NVDA (Windows) and navigate your site without looking at the screen. Note where announcements are confusing, missing, or in the wrong order
5. **WCAG 2.2 gap analysis**: Review your site against the nine new WCAG 2.2 success criteria specifically. Check target sizes, focus visibility, drag alternatives, and authentication flows
