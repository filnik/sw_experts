# Practical Web Accessibility

## Profile

### What It Is
A hands-on approach to implementing web accessibility through semantic HTML, purposeful ARIA usage, keyboard interaction patterns, and screen reader compatibility. Rather than focusing on specification compliance alone, practical accessibility emphasizes building inclusive interfaces from the ground up using native platform features, progressive enhancement, and real-world testing with assistive technologies. This methodology bridges the gap between WCAG success criteria and production-ready code.

### Origin and Evolution
- **1999** — WAI-ARIA first conceptualized to bridge gaps in HTML's ability to express widget semantics
- **2008** — Screen readers begin supporting ARIA landmarks, making single-page applications more navigable
- **2014** — WAI-ARIA 1.0 becomes a W3C Recommendation, standardizing roles, states, and properties
- **2017** — WAI-ARIA 1.1 published, adding new roles for feeds, figures, and search suggestions
- **2019** — ARIA Authoring Practices Guide (APG) restructured with pattern-based examples
- **2023** — WAI-ARIA 1.2 and updated APG patterns align with WCAG 2.2
- **2024** — Sara Soueidan publishes comprehensive practical accessibility course

### Key Authors and References
- **Sara Soueidan** — *Practical Accessibility* course and blog (2018-present)
- **Leonie Watson** — *Screen reader user and W3C accessibility standards editor* (2010s-present)
- **Heydon Pickering** — *Inclusive Components* (2018)
- **Scott O'Hara** — *Accessible component patterns and ARIA research* (2016-present)
- **Adrian Roselli** — *Accessibility testing and ARIA implementation blog* (2014-present)
- **W3C WAI** — *ARIA Authoring Practices Guide (APG)* (ongoing)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Semantic HTML | Using elements that convey meaning and behavior natively, reducing the need for ARIA |
| ARIA Roles | Defining the type of UI element (e.g., dialog, tabpanel, alert) for assistive technology |
| ARIA States | Dynamic attributes reflecting the current condition of an element (e.g., aria-expanded, aria-checked) |
| ARIA Properties | Attributes that define relationships or characteristics (e.g., aria-labelledby, aria-describedby) |
| Focus Management | Controlling which element receives keyboard focus and in what order |
| Live Regions | Areas of the page that announce dynamic updates to screen readers |
| Landmark Regions | Semantic sections (nav, main, aside, header, footer) that enable navigation by region |
| Skip Links | Hidden links that let keyboard users jump past repetitive navigation to main content |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Semantic HTML first, ARIA second** — Native HTML elements carry built-in accessibility semantics, keyboard behavior, and cross-browser support; ARIA should only supplement what HTML cannot express
2. **No ARIA is better than bad ARIA** — Incorrect ARIA actively harms the experience; a `<div>` with no role is ignored, but a `<div role="button">` without keyboard handling creates a broken control
3. **Keyboard is the universal input** — If it works with a keyboard, it works with switch devices, voice control, and most other alternative inputs; keyboard operability is the single highest-impact accessibility improvement
4. **Test with real assistive technology** — Automated tools catch syntax errors, but only real screen reader testing reveals whether the experience is actually usable; VoiceOver, NVDA, and JAWS each interpret ARIA differently
5. **Accessibility is a design constraint, not a feature** — Like responsive design or performance, accessibility should be a baseline requirement in every component, not a separate ticket or afterthought

### When to Use vs. When to Avoid

**Use when:**
- Building any custom interactive component (dropdown, modal, tabs, accordion, carousel)
- Implementing single-page applications where content updates dynamically
- Creating forms that require validation, error handling, or multi-step workflows
- Designing navigation systems, menus, and wayfinding patterns
- Developing components for a shared design system or component library

**Avoid over-application when:**
- Native HTML already provides the needed semantics — do not add `role="navigation"` to a `<nav>` element
- ARIA is used to fix what should be solved by restructuring the DOM
- Complex ARIA patterns are added to simple content that needs no widget behavior
- Screen reader hacks are applied without understanding that they break other assistive technologies

---

## Frameworks and Methodologies

### 1. The ARIA Decision Tree — choosing the right approach
1. **Can you use a native HTML element?** If yes, use it and stop. `<button>`, `<a>`, `<input>`, `<select>`, `<dialog>` all carry built-in semantics and keyboard behavior
2. **Does the native element need supplemental labeling?** Use `aria-label`, `aria-labelledby`, or `aria-describedby` to provide additional context
3. **Are you building a custom widget with no HTML equivalent?** Implement the full ARIA pattern from the APG: role, keyboard interaction, states, and properties
4. **Does content update dynamically?** Use live regions (`aria-live`, `role="status"`, `role="alert"`) to announce changes
5. **Test with a screen reader.** If the announcement does not match the visual experience, revise

### 2. Component Accessibility Audit — for existing components
1. **Semantic check** — Is the component using the correct HTML elements for its purpose?
2. **Name check** — Does every interactive element have an accessible name (visible label, aria-label, or aria-labelledby)?
3. **Role check** — Is the component's role correctly communicated (native or via ARIA)?
4. **State check** — Are dynamic states (expanded, selected, checked, disabled) communicated to assistive tech?
5. **Keyboard check** — Can the component be fully operated by keyboard following expected conventions?
6. **Focus check** — Is focus managed correctly when the component opens, closes, or updates?

---

## How to Apply

### Decision Checklist
- [ ] Is every interactive element built with the most semantic HTML element available?
- [ ] Does every form input have a programmatically associated `<label>`?
- [ ] Are landmark regions (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`) used correctly?
- [ ] Is ARIA used only where native HTML is insufficient?
- [ ] Can the entire interface be operated with keyboard alone?
- [ ] Is focus visible, logical, and managed correctly on dynamic content changes?
- [ ] Are dynamic updates announced via appropriate live regions?
- [ ] Does the page include a skip navigation link?
- [ ] Has the component been tested with VoiceOver and NVDA?
- [ ] Are error messages associated with their inputs via `aria-describedby`?

### Implementation Patterns

**Landmark regions and skip navigation:**
```html
<body>
  <!-- Skip link — first focusable element on the page -->
  <a href="#main-content" class="skip-link">Skip to main content</a>

  <header role="banner">
    <nav aria-label="Primary">
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/products">Products</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>
  </header>

  <main id="main-content">
    <!-- Main page content -->
  </main>

  <aside aria-label="Related articles">
    <!-- Sidebar content -->
  </aside>

  <footer role="contentinfo">
    <nav aria-label="Footer">
      <!-- Footer navigation, distinct from primary -->
    </nav>
  </footer>
</body>
```

```css
/* Skip link — visible only on focus */
.skip-link {
  position: absolute;
  top: -100%;
  left: 1rem;
  z-index: 10000;
  padding: 0.75rem 1.5rem;
  background-color: #1e40af;
  color: #ffffff;
  font-weight: 600;
  text-decoration: none;
  border-radius: 0 0 6px 6px;
}

.skip-link:focus {
  top: 0;
}
```

**Accessible form with error handling:**
```html
<form novalidate>
  <div class="form-group">
    <label for="user-email">Email address <span aria-hidden="true">*</span></label>
    <input
      type="email"
      id="user-email"
      name="email"
      required
      autocomplete="email"
      aria-required="true"
      aria-invalid="false"
      aria-describedby="email-hint email-error"
    />
    <p id="email-hint" class="hint">We will never share your email.</p>
    <p id="email-error" class="error-msg" role="alert" hidden>
      Please enter a valid email address, for example name@domain.com
    </p>
  </div>

  <fieldset>
    <legend>Notification preferences</legend>
    <div class="checkbox-group">
      <input type="checkbox" id="notify-email" name="notify" value="email" />
      <label for="notify-email">Email notifications</label>
    </div>
    <div class="checkbox-group">
      <input type="checkbox" id="notify-sms" name="notify" value="sms" />
      <label for="notify-sms">SMS notifications</label>
    </div>
  </fieldset>

  <button type="submit">Save preferences</button>
</form>
```

```javascript
// Form validation with accessible error announcement
function validateEmail(input) {
  const errorEl = document.getElementById('email-error');
  const isValid = input.validity.valid;

  input.setAttribute('aria-invalid', !isValid);

  if (!isValid) {
    errorEl.hidden = false;   // role="alert" triggers screen reader announcement
  } else {
    errorEl.hidden = true;
  }
}
```

**Accessible modal dialog:**
```html
<dialog id="confirm-dialog" aria-labelledby="dlg-title" aria-describedby="dlg-desc">
  <h2 id="dlg-title">Confirm deletion</h2>
  <p id="dlg-desc">This action cannot be undone. Delete this item?</p>
  <button type="button" id="cancel-btn">Cancel</button>
  <button type="button" id="confirm-btn" class="destructive">Delete</button>
</dialog>
```

```javascript
// Native <dialog> handles focus trapping, backdrop, and Escape key
const dialog = document.getElementById('confirm-dialog');
const triggerBtn = document.getElementById('open-dialog-btn');

triggerBtn.addEventListener('click', () => dialog.showModal());
document.getElementById('cancel-btn').addEventListener('click', () => {
  dialog.close();
  triggerBtn.focus();  // Always return focus to the triggering element
});
```

**Accessible tab component:**
```html
<!-- Tabs: arrow keys navigate tabs, Tab key moves into panel content -->
<div role="tablist" aria-label="Account settings">
  <button role="tab" id="tab-1" aria-controls="panel-1"
          aria-selected="true" tabindex="0">Profile</button>
  <button role="tab" id="tab-2" aria-controls="panel-2"
          aria-selected="false" tabindex="-1">Security</button>
</div>
<div role="tabpanel" id="panel-1" aria-labelledby="tab-1" tabindex="0">
  <!-- Active panel content -->
</div>
<div role="tabpanel" id="panel-2" aria-labelledby="tab-2" tabindex="0" hidden>
  <!-- Hidden panel content -->
</div>
```

```javascript
// Keyboard: ArrowRight/Left cycle tabs, Home/End jump to first/last
tablist.addEventListener('keydown', (e) => {
  const current = Array.from(tabs).indexOf(e.target);
  const moves = { ArrowRight: 1, ArrowLeft: -1, Home: -current, End: tabs.length - 1 - current };
  if (!(e.key in moves)) return;
  e.preventDefault();
  const next = (current + moves[e.key] + tabs.length) % tabs.length;
  tabs.forEach((t, i) => {
    t.setAttribute('aria-selected', i === next);
    t.tabIndex = i === next ? 0 : -1;
    document.getElementById(t.getAttribute('aria-controls')).hidden = i !== next;
  });
  tabs[next].focus();
});
```

**Live regions for dynamic updates:**
```html
<!-- Polite: waits for screen reader to finish current speech -->
<div aria-live="polite" aria-atomic="true" class="sr-only" id="status-msg"></div>
<!-- Assertive: interrupts current speech — use only for critical errors -->
<div role="alert" class="sr-only" id="error-alert"></div>
```

```javascript
// Update live region text to trigger screen reader announcement
document.getElementById('status-msg').textContent = `${count} results found.`;
document.getElementById('error-alert').textContent = 'Session expired. Please log in again.';
```

```css
/* Visually hidden but announced by screen readers */
.sr-only {
  position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px;
  overflow: hidden; clip: rect(0, 0, 0, 0); white-space: nowrap; border: 0;
}
```

**Focus-visible styling:**
```css
:focus:not(:focus-visible) { outline: none; } /* Hide outline for mouse clicks */
:focus-visible {
  outline: 3px solid #2563eb;
  outline-offset: 2px;
  border-radius: 2px;
}
```

### Common Mistakes
1. **Adding ARIA to native elements** — Writing `<nav role="navigation">` or `<button role="button">` is redundant; native semantics are already communicated. Worse, adding incorrect roles overrides native semantics
2. **Div and span as interactive elements** — Using `<div onclick="...">` instead of `<button>` loses keyboard support, focus management, and screen reader announcement. Replicating button behavior with ARIA requires role, tabindex, and keydown handling
3. **Missing accessible names** — Icon-only buttons without `aria-label`, inputs without associated labels, and landmarks without distinguishing labels when multiple of the same type exist
4. **Trapping focus incorrectly** — Focus traps are required for modals but must always provide an escape mechanism. Trapping focus without an exit path blocks keyboard users entirely
5. **Using aria-live too aggressively** — Marking large content areas as live regions or using `aria-live="assertive"` for non-critical updates creates an overwhelming experience; use `polite` by default and `assertive` only for errors
6. **Hiding content from only one audience** — Using `display: none` hides from everyone, `aria-hidden="true"` hides from assistive tech only, and `.sr-only` hides visually only. Confusing these three causes content to be invisible to the wrong audience

### Integration with Other Topics
- **32-wcag-accessibility** — WCAG defines the success criteria; this topic provides the implementation techniques that satisfy them
- **29-form-design** — Accessible forms require semantic structure (fieldset, legend, label) and ARIA-enhanced error handling covered here
- **05-usability-heuristics** — Nielsen's heuristics on error prevention, recognition over recall, and help/documentation align directly with accessible form design and navigation patterns

---

## Resources

### Essential Reading
- Soueidan, S. (2024). *Practical Accessibility*. practicalacccessibility.today.
- Pickering, H. (2018). *Inclusive Components*. Smashing Magazine.
- Watson, L. (2021). *Screen reader user perspective talks*. TetraLogical.
- W3C WAI. (2023). *ARIA Authoring Practices Guide (APG)*. W3C.

### Online Resources
- [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/) — Canonical reference for ARIA widget patterns with examples
- [Sara Soueidan's Blog](https://www.sarasoueidan.com/blog/) — Deep dives into practical accessibility implementation
- [TetraLogical Blog (Leonie Watson)](https://tetralogical.com/blog/) — Accessibility from a screen reader user perspective
- [Scott O'Hara's Accessible Components](https://www.scottohara.me/) — Tested accessible component patterns
- [Adrian Roselli's Blog](https://adrianroselli.com/) — Detailed ARIA testing and implementation analysis
- [A11y Project Checklist](https://www.a11yproject.com/checklist/) — Community-driven accessibility checklist
- [Deque University](https://dequeuniversity.com/) — Free and paid accessibility training resources

### Practice Exercises
1. **Semantic HTML refactor**: Take a page that relies on `<div>` and `<span>` for structure. Refactor it to use proper landmark elements, headings, lists, and semantic HTML. Test before and after with a screen reader and note the difference in navigation
2. **Build an accessible modal**: Implement a confirmation dialog using the native `<dialog>` element with proper focus trapping, Escape key handling, and focus restoration. Test with VoiceOver and NVDA
3. **Tab component from scratch**: Build a tabbed interface following the ARIA APG tab pattern. Implement arrow key navigation between tabs, proper roles and states, and hidden panel management. Verify that Tab moves from the active tab to its panel content
4. **Screen reader testing session**: Install NVDA (Windows) or use VoiceOver (macOS). Navigate a complex web application (e.g., GitHub, Gmail) for 15 minutes with the screen turned off. Document every moment of confusion and identify which ARIA patterns could improve the experience
5. **Live region implementation**: Build a search interface with dynamically filtered results. Implement a polite live region that announces the number of results after each filter change. Verify that the announcement timing does not overlap with user input
