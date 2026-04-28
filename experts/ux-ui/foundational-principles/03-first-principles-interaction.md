# First Principles of Interaction Design

## Profile

### What It Is
Bruce Tognazzini's First Principles of Interaction Design is a comprehensive set of 19 foundational guidelines distilled from decades of work in human-computer interaction, originally developed during Tognazzini's tenure at Apple (employee #66) and refined through his consulting practice. Unlike heuristics that evaluate existing designs, these principles are generative: they guide designers in building interactions correctly from the start. They cover everything from perceptual concerns (color, readability) to structural concerns (navigation, consistency) to behavioral concerns (autonomy, latency).

### Origin and Evolution
- **1978-1992** — Tognazzini works at Apple, establishing the Apple Human Interface Guidelines and building the first HIG team
- **1992** — Publishes *Tog on Interface*, collecting principles from his "Ask Tog" column
- **1996** — Publishes *Tog on Software Design*, expanding principles to full software development
- **1999** — Co-founds Nielsen Norman Group with Don Norman and Jakob Nielsen
- **2000s-present** — Maintains and updates the First Principles list on AskTog.com, adapting for web, mobile, and emerging platforms

### Key Authors and References
- **Bruce Tognazzini** — *Tog on Interface* (1992)
- **Bruce Tognazzini** — *Tog on Software Design* (1996)
- **Bruce Tognazzini** — "First Principles of Interaction Design" (AskTog.com, updated regularly)
- **Paul Fitts** — Fitts's Law (1954) — the foundational pointing-time model referenced in the principles
- **Apple Computer** — *Macintosh Human Interface Guidelines* (1987) — co-authored by Tognazzini

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Anticipation | Provide information and tools before users need to ask for them |
| Autonomy | Give users control while keeping them informed of system state |
| Consistency | Maintain behavioral and visual consistency across the product |
| Defaults | Ship smart defaults that serve the majority without requiring configuration |
| Discoverability | Make all features and states findable through the interface itself |
| Efficiency | Optimize for the user's productivity, not the system's |
| Fitts's Law | Target acquisition time is a function of distance and size |
| Learnability | Reduce the learning curve through predictable patterns |
| Latency Reduction | Mask or eliminate perceived wait times |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Anticipate, do not interrogate** — The interface should supply users with the information and tools they will need at each step, rather than forcing them to request or search for resources
2. **Consistency enables transfer of knowledge** — When elements behave consistently, users can apply what they learned in one part of the system to every other part, dramatically reducing cognitive load
3. **Visible interfaces beat invisible ones** — Users should be able to see where they are, where they can go, and what is happening at all times. Invisible state and hidden navigation impose memory burdens that lead to errors
4. **Protect users' work above all** — No single user action should be able to destroy significant work. Auto-save, undo, and confirmation gates are not optional luxuries
5. **Perceived performance matters as much as actual performance** — A 2-second operation that shows immediate visual feedback feels faster than a 0.5-second operation that freezes the screen

### When to Use vs. When to Avoid

**Use when:**
- Designing any interactive system intended for repeated use by a broad audience
- Building component libraries or design systems that must be internally consistent
- Evaluating whether an interface respects fundamental human perceptual and cognitive limitations
- Training junior designers who need a comprehensive reference for interaction decisions
- Auditing existing products for systematic interaction problems

**Avoid over-application when:**
- Strict consistency would create a worse experience than a context-appropriate departure (consistency is a means, not an end)
- You are designing for a highly specialized expert audience that values density and speed over discoverability
- Rigid adherence to all 19 principles simultaneously creates contradictions (e.g., simplicity vs. discoverability); prioritize based on context

---

## Frameworks and Methodologies

### 1. Principle-Based Design Audit — step-by-step
1. Select a task flow (e.g., user registration, checkout, report creation)
2. Walk through the flow screen by screen
3. For each screen, evaluate against each of the 19 principles (use a checklist)
4. Score each principle as Supported, Partially Supported, or Violated
5. Prioritize violations by impact on the primary persona's goals
6. Propose design changes for the highest-impact violations first

### 2. The 19 Principles — complete reference
1. **Anticipation** — Supply all information and tools needed at each step
2. **Autonomy** — Let users make choices, but keep them informed of consequences
3. **Color Blindness** — Never use color as the sole differentiator; always pair with shape, text, or position
4. **Consistency** — Maintain visual, behavioral, and platform consistency; break only with strong justification
5. **Defaults** — Provide intelligent defaults based on the most common use case
6. **Discoverability** — Every feature must be findable by exploration of the interface
7. **Efficiency** — Optimize for the experienced user after providing a discoverable path for new users
8. **Explorable Interfaces** — Let users try things without irreversible consequences; support undo everywhere
9. **Fitts's Law** — Make frequently used targets large and close; make dangerous targets small and far
10. **Human Interface Objects** — Interactive objects should behave like real-world objects: graspable, movable, with visible affordances
11. **Latency Reduction** — Acknowledge actions within 50ms, show progress for anything over 500ms, allow interaction during long operations
12. **Learnability** — Minimize the learning curve; make the system predictable from the first interaction
13. **Metaphors** — Use metaphors that help users transfer existing knowledge, but do not let metaphors constrain the design
14. **Protect Users' Work** — Auto-save, confirm destructive actions, support undo with generous history
15. **Readability** — Use high-contrast text, readable font sizes, and appropriate line lengths
16. **Simplicity** — Simplify what you can, but never at the expense of removing necessary information or controls
17. **State Tracking** — Remember user state across sessions (scroll position, filters, preferences)
18. **Visible Navigation** — Users must always know where they are, where they can go, and how to get back
19. **Reduce Latency** — Use progressive loading, optimistic UI, and background processing to keep the interface responsive

---

## How to Apply

### Decision Checklist
- [ ] Does the interface anticipate what users need at each step?
- [ ] Can users always tell where they are and how to get back?
- [ ] Is color used only as a redundant cue, never as the sole differentiator?
- [ ] Do interactive elements behave consistently throughout the product?
- [ ] Are smart defaults set so most users never need to change settings?
- [ ] Can users undo every significant action?
- [ ] Are click/tap targets large enough and close enough for frequent actions (Fitts's Law)?
- [ ] Is text readable at default settings without zooming (minimum 16px body, 4.5:1 contrast)?
- [ ] Does the system remember user state across sessions?
- [ ] Is feedback shown within 50ms for every user action?

### Implementation Patterns

**Color blindness — never rely on color alone:**
```css
/* ANTI-PATTERN: Status indicated only by color */
.status-success { color: #16a34a; }
.status-error { color: #dc2626; }

/* CORRECT: Color paired with icon and text */
.status-success::before {
  content: "\2713 ";  /* checkmark */
}
.status-error::before {
  content: "\2717 ";  /* X mark */
}
```

```html
<!-- Accessible status indicator -->
<span class="status-success" role="status">
  <svg aria-hidden="true" class="icon"><use href="#icon-check" /></svg>
  Payment successful
</span>

<span class="status-error" role="alert">
  <svg aria-hidden="true" class="icon"><use href="#icon-x" /></svg>
  Payment failed — card declined
</span>
```

**Fitts's Law — target sizing and placement:**
```css
/* Primary action: large, prominent, easy to reach */
.action-primary {
  min-height: 48px;       /* large touch/click target */
  min-width: 120px;
  padding: 12px 32px;
  font-size: 1rem;
}

/* Destructive action: smaller, requires deliberate aim */
.action-destructive {
  min-height: 36px;
  padding: 6px 16px;
  font-size: 0.8125rem;
  /* Place away from primary action to prevent accidental clicks */
}

/* Navigation targets in mobile: full-width for easy tapping */
@media (max-width: 768px) {
  .nav-link {
    display: block;
    padding: 16px 20px;     /* generous tap area */
    min-height: 48px;
    border-bottom: 1px solid #e5e7eb;
  }
}
```

**Latency reduction — optimistic UI and skeleton screens:**
```html
<!-- Skeleton screen shown while content loads -->
<div class="card card--loading" aria-busy="true" aria-label="Loading content">
  <div class="skeleton skeleton--image"></div>
  <div class="skeleton skeleton--title"></div>
  <div class="skeleton skeleton--text"></div>
  <div class="skeleton skeleton--text skeleton--short"></div>
</div>
```

```css
.skeleton {
  background: linear-gradient(90deg, #e5e7eb 25%, #f3f4f6 50%, #e5e7eb 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: 4px;
}

.skeleton--image { height: 200px; margin-bottom: 16px; }
.skeleton--title { height: 24px; width: 70%; margin-bottom: 12px; }
.skeleton--text  { height: 16px; width: 100%; margin-bottom: 8px; }
.skeleton--short { width: 40%; }

@keyframes shimmer {
  0%   { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

**State tracking — remembering user preferences:**
```javascript
// Persist user state across sessions
const StateTracker = {
  save(key, value) {
    try {
      localStorage.setItem(`app_state_${key}`, JSON.stringify(value));
    } catch (e) {
      // Silently fail if storage is full or blocked
    }
  },

  load(key, defaultValue) {
    try {
      const stored = localStorage.getItem(`app_state_${key}`);
      return stored !== null ? JSON.parse(stored) : defaultValue;
    } catch (e) {
      return defaultValue;
    }
  }
};

// Restore scroll position, selected filters, sidebar state
window.addEventListener('DOMContentLoaded', () => {
  const scrollY = StateTracker.load('scrollY', 0);
  const sidebarOpen = StateTracker.load('sidebarOpen', true);
  const activeFilters = StateTracker.load('filters', []);

  window.scrollTo(0, scrollY);
  document.querySelector('.sidebar')
    .classList.toggle('collapsed', !sidebarOpen);
});
```

**Protect users' work — auto-save with visual indicator:**
```css
.save-indicator {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 0.8125rem;
  color: #6b7280;
  transition: opacity 0.3s ease;
}

.save-indicator[data-state="saving"] { color: #f59e0b; }
.save-indicator[data-state="saved"]  { color: #16a34a; }
.save-indicator[data-state="error"]  { color: #dc2626; }
```

### Common Mistakes
1. **Inconsistency disguised as innovation** — Changing standard interaction patterns (e.g., unconventional scrolling, non-standard icons) without user research showing it actually improves the experience
2. **Color-only status** — Using red/green alone to indicate error/success, which is invisible to the 8% of males with color vision deficiency
3. **Tiny touch targets** — Buttons and links smaller than 44x44px on mobile, violating Fitts's Law and causing frustration for anyone without fine motor precision
4. **Ignoring latency** — Showing no feedback between click and response, leaving users wondering whether their action registered
5. **Destroying user work** — Allowing a single accidental click, navigation, or session timeout to erase unsaved input without warning or recovery

### Integration with Other Topics
- **05-usability-heuristics** — Tognazzini's 19 principles and Nielsen's 10 heuristics overlap significantly but are independently derived. Using both for evaluation provides broader coverage
- **10-laws-of-ux** — Fitts's Law, Hick's Law, and the principles of perceived performance from Laws of UX are formalized versions of several Tog principles
- **01-design-of-everyday-things** — Norman's affordance/signifier framework provides the cognitive science foundation that Tognazzini's more pragmatic principles build upon
- **37-web-performance-ux** — The latency reduction principle connects directly to web performance optimization techniques (lazy loading, code splitting, service workers)

---

## Resources

### Essential Reading
- Tognazzini, B. (1992). *Tog on Interface*. Addison-Wesley.
- Tognazzini, B. (1996). *Tog on Software Design*. Addison-Wesley.
- Fitts, P. (1954). "The Information Capacity of the Human Motor System in Controlling the Amplitude of Movement." *Journal of Experimental Psychology*, 47(6), 381-391.

### Online Resources
- [AskTog: First Principles of Interaction Design](https://asktog.com/atc/principles-of-interaction-design/) — The canonical, regularly updated list
- [Nielsen Norman Group](https://www.nngroup.com/) — Co-founded by Tognazzini; research-based design articles
- [Apple Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/) — Descendant of the guidelines Tognazzini helped create

### Practice Exercises
1. **19-principle audit**: Select a web application you use regularly. Score it against all 19 principles using a three-point scale (Supported / Partial / Violated). Identify the three most impactful violations and propose fixes
2. **Color blindness test**: Use a color blindness simulator (e.g., Chrome DevTools "Emulate vision deficiencies") on your current project. Identify every place where color is the sole differentiator and add redundant cues
3. **Fitts's Law optimization**: Measure the distance (in pixels) between the most commonly used controls on a page. Identify pairs that should be closer together and targets that should be larger. Prototype the changes and compare task completion times
4. **Latency perception experiment**: Add skeleton screens or optimistic updates to a page that currently shows a spinner. Measure perceived speed improvement through a simple user test (ask users "which felt faster?")
