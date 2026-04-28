# The Design of Everyday Things

## Profile

### What It Is
Don Norman's foundational framework for understanding how people interact with designed objects and interfaces. It introduces six core principles — affordance, signifier, constraint, mapping, feedback, and conceptual model — that explain why some designs feel intuitive while others frustrate users. These principles form the bedrock of modern interaction design, applicable from door handles to complex web applications.

### Origin and Evolution
- **1988** — Don Norman publishes *The Psychology of Everyday Things* (POET), introducing affordance and mental models to design discourse
- **1990** — The book is retitled *The Design of Everyday Things* (DOET) for broader appeal
- **1999** — Norman publishes his influential essay "Affordances, Conventions, and Design," clarifying the distinction between affordances and signifiers
- **2002** — Norman founds the Nielsen Norman Group with Jakob Nielsen, bridging cognitive science and usability
- **2013** — Revised and expanded edition of DOET published, adding signifiers as a distinct concept and updating examples for the digital age

### Key Authors and References
- **Don Norman** — *The Design of Everyday Things* (1988, revised 2013)
- **Don Norman** — *Emotional Design: Why We Love (or Hate) Everyday Things* (2004)
- **James J. Gibson** — *The Ecological Approach to Visual Perception* (1979) — originated the concept of affordance
- **Don Norman & Jakob Nielsen** — Nielsen Norman Group research articles (1998-present)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Affordance | The relationship between an object's properties and the capabilities of the agent interacting with it |
| Signifier | A perceivable cue that communicates what action is possible and how to perform it |
| Constraint | A limitation that guides the user toward correct actions by restricting possibilities |
| Mapping | The relationship between controls and their effects in the world |
| Feedback | Communicating the result of an action back to the user |
| Conceptual Model | The user's mental understanding of how a system works |
| Discoverability | How easily a user can figure out what actions are possible |
| Understanding | How well a user can build a correct mental model of the system |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Discoverability over hidden power** — Users should be able to determine what actions are possible just by looking at the interface; hidden functionality is useless functionality
2. **Feedback closes the loop** — Every user action must produce an immediate, informative response so users understand the system's state
3. **Conceptual models must match reality** — The system image (what the design communicates) must align with the designer's model so users form accurate mental models
4. **Constraints prevent errors** — Good design makes it hard or impossible to do the wrong thing, rather than punishing users after they err
5. **Human-centered over technology-centered** — Design must adapt to people, not force people to adapt to technology; when users make errors, blame the design, not the user

### When to Use vs. When to Avoid

**Use when:**
- Designing any interactive element (buttons, forms, navigation) that users must understand without instruction
- Evaluating why users struggle with an existing interface
- Building design systems or component libraries where consistent signifiers matter
- Onboarding new users who have no prior experience with your product
- Conducting heuristic evaluations or usability audits

**Avoid over-application when:**
- Expert interfaces where efficiency matters more than discoverability (e.g., professional video editing tools)
- Designing for power users who have already internalized the conceptual model
- Situations where over-signifying creates visual clutter that slows down experienced users
- Gamified experiences where deliberate mystery or exploration is the goal

---

## Frameworks and Methodologies

### 1. The Seven Stages of Action — diagnostic framework
1. **Goal** — Form the goal (what do I want to accomplish?)
2. **Plan** — Plan the action (what are the alternatives?)
3. **Specify** — Specify the action sequence (what can I do now?)
4. **Perform** — Perform the action (how do I do it?)
5. **Perceive** — Perceive the state of the world (what happened?)
6. **Interpret** — Interpret the perception (what does it mean?)
7. **Compare** — Compare the outcome with the goal (is this what I wanted?)

Use this to diagnose breakdowns: a Gulf of Execution exists between steps 1-4 (user cannot figure out how to act), and a Gulf of Evaluation exists between steps 5-7 (user cannot tell what happened).

### 2. Affordance Audit — for existing interfaces
1. List every interactive element on the page
2. For each element, identify the intended affordance (what it should let the user do)
3. Identify the signifier (what visual/auditory cue communicates the affordance)
4. Test: can a new user perceive and correctly interpret the signifier?
5. Document gaps where affordances exist but signifiers are missing or misleading

---

## How to Apply

### Decision Checklist
- [ ] Can users discover all available actions without instructions?
- [ ] Does every interactive element have a clear signifier distinguishing it from non-interactive content?
- [ ] Does the system provide immediate feedback for every user action?
- [ ] Do constraints prevent users from performing invalid or dangerous actions?
- [ ] Does the layout mapping reflect logical relationships between controls and outcomes?
- [ ] Can users form an accurate conceptual model from the system image alone?
- [ ] Are error messages constructive, explaining what went wrong and how to fix it?
- [ ] Have you tested with users who have no prior knowledge of the system?

### Implementation Patterns

**Good affordance — button that looks clickable:**
```css
/* Strong signifiers: raised appearance, cursor change, hover feedback */
.btn-primary {
  display: inline-block;
  padding: 12px 24px;
  background-color: #2563eb;
  color: #ffffff;
  border: none;
  border-radius: 6px;
  font-weight: 600;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.15);
  transition: all 0.15s ease;
}

.btn-primary:hover {
  background-color: #1d4ed8;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  transform: translateY(-1px);
}

.btn-primary:active {
  transform: translateY(0);
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.15);
}
```

**Bad affordance — clickable element that looks like plain text:**
```html
<!-- ANTI-PATTERN: No signifier that this is interactive -->
<span onclick="submitForm()" style="color: #333; font-size: 14px;">
  Submit your application
</span>

<!-- FIX: Use semantic HTML with proper signifiers -->
<button type="submit" class="btn-primary">
  Submit your application
</button>
```

**Feedback pattern — form submission states:**
```css
/* Visual feedback for each state of a form submission */
.submit-btn[data-state="idle"] {
  background-color: #2563eb;
}

.submit-btn[data-state="loading"] {
  background-color: #93c5fd;
  pointer-events: none;
}

.submit-btn[data-state="success"] {
  background-color: #16a34a;
}

.submit-btn[data-state="error"] {
  background-color: #dc2626;
  animation: shake 0.3s ease-in-out;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-4px); }
  75% { transform: translateX(4px); }
}
```

**Constraint pattern — preventing invalid input:**
```html
<!-- Constraints that guide users toward correct actions -->
<form>
  <label for="email">Email address</label>
  <input
    type="email"
    id="email"
    required
    pattern="[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}$"
    placeholder="you@example.com"
    autocomplete="email"
  />

  <label for="date">Appointment date</label>
  <input
    type="date"
    id="date"
    min="2026-02-23"
    max="2026-12-31"
  />
  <!-- min constraint prevents selecting past dates -->
</form>
```

**Mapping pattern — controls that reflect spatial layout:**
```html
<!-- Good mapping: controls mirror the layout they affect -->
<div class="text-alignment" role="radiogroup" aria-label="Text alignment">
  <button aria-label="Align left" class="align-btn">
    <span class="icon-align-left"></span>
  </button>
  <button aria-label="Align center" class="align-btn">
    <span class="icon-align-center"></span>
  </button>
  <button aria-label="Align right" class="align-btn">
    <span class="icon-align-right"></span>
  </button>
</div>
<!-- Buttons are ordered left-to-right matching the alignment they produce -->
```

### Common Mistakes
1. **Confusing affordance with signifier** — An affordance is the actual possibility for action; a signifier is the perceivable indicator. A flat text link has the affordance of being clickable but may lack a signifier if it is not underlined or colored differently
2. **Feedback that arrives too late** — Users need feedback within 100ms to feel the system responded instantly; delays beyond 1 second require a progress indicator
3. **False affordances** — Styling non-interactive elements to look interactive (underlined text that is not a link, card shadows on non-clickable cards) erodes user trust
4. **Over-constraining expert users** — Applying so many constraints that proficient users feel trapped; provide escape hatches for advanced workflows
5. **Invisible system state** — Failing to show users where they are, what mode the system is in, or whether their action had any effect

### Integration with Other Topics
- **05-usability-heuristics** — Norman's principles directly map to several of Nielsen's 10 heuristics, particularly visibility of system status, match between system and real world, and error prevention
- **03-first-principles-interaction** — Tognazzini's 19 principles expand on Norman's work with more specific, implementation-ready guidelines
- **10-laws-of-ux** — Laws like Fitts's Law and Hick's Law provide quantitative backing for Norman's qualitative principles
- **34-emotional-design** — Norman's later work on emotional design extends DOET by adding visceral, behavioral, and reflective levels of processing

---

## Resources

### Essential Reading
- Norman, D. (2013). *The Design of Everyday Things: Revised and Expanded Edition*. Basic Books.
- Norman, D. (1999). "Affordances, Conventions, and Design." *Interactions*, 6(3), 38-43.
- Gibson, J.J. (1979). *The Ecological Approach to Visual Perception*. Houghton Mifflin.

### Online Resources
- [Nielsen Norman Group Articles](https://www.nngroup.com/articles/) — Research-backed design guidance
- [Don Norman's jnd.org](https://jnd.org/) — Essays and talks on design
- [NNg: Affordances and Signifiers](https://www.nngroup.com/articles/affordances-signifiers/) — Clarification of the concepts for digital design

### Practice Exercises
1. **Affordance audit**: Pick any web application you use daily. Screenshot three pages and label every signifier. Identify at least two places where affordances exist without adequate signifiers
2. **Gulf analysis**: Observe someone using an unfamiliar interface. Map their actions to the Seven Stages of Action and identify where gulfs of execution or evaluation occur
3. **Constraint redesign**: Take a form that relies on error messages after submission. Redesign it using input constraints (type, min, max, pattern, required) so that errors are prevented rather than reported
4. **Feedback inventory**: Time the feedback delay for 10 common actions in a web app. Categorize each as instant (<100ms), acknowledged (100ms-1s), or requires-indicator (>1s). Fix any that fall in the wrong category
