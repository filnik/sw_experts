# Cognitive Load Theory

## Profile

### What It Is
Cognitive Load Theory (CLT) is an instructional design framework that describes the limited capacity of human working memory and its implications for learning and interface design. Applied to UX, it provides a scientific basis for simplifying interfaces, structuring information progressively, and reducing the mental effort required to complete tasks.

### Origin and Evolution
- George A. Miller's landmark paper *The Magical Number Seven, Plus or Minus Two* (1956) established working memory limits
- John Sweller formalized Cognitive Load Theory in 1988, originally for educational contexts
- Richard Mayer extended CLT into multimedia learning principles (1990s-2000s)
- Alan Baddeley's working memory model (1974, revised 2000) provided the cognitive architecture underpinning CLT
- UX community adopted CLT principles from the 2010s onward, particularly for form design, onboarding, and information architecture
- Modern applications include progressive disclosure patterns, skeleton screens, and conversational UI design

### Key Authors and References
- **John Sweller** — *Cognitive Load Theory* (Springer, 2011); original CLT paper (1988)
- **George A. Miller** — *The Magical Number Seven, Plus or Minus Two* (Psychological Review, 1956)
- **Richard Mayer** — *Multimedia Learning* (Cambridge University Press, 2001)
- **Alan Baddeley** — Working Memory Model (1974, 2000)
- **Steve Krug** — *Don't Make Me Think* (New Riders, 2000) — practical CLT application to web usability
- **Susan Weinschenk** — *100 Things Every Designer Needs to Know About People* (2011)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Intrinsic Load | Inherent complexity of the task itself; cannot be eliminated, only managed |
| Extraneous Load | Unnecessary load caused by poor design, layout, or instruction |
| Germane Load | Productive load that contributes to learning and schema building |
| Working Memory | Short-term store holding ~4-7 items for active processing |
| Chunking | Grouping individual items into meaningful units to extend effective capacity |
| Schema | Mental model that organizes knowledge, reducing load for familiar patterns |
| Progressive Disclosure | Revealing information only when needed to avoid overwhelming the user |
| Cognitive Tunneling | Fixation on one element under high load, missing critical information |

---

## Core Philosophy

### The 5 Fundamental Principles

1. **Minimize Extraneous Load** — Poor design is the primary source of unnecessary cognitive burden. Every visual element, instruction, or interaction that does not directly serve the user's goal adds extraneous load. The designer's first job is to eliminate it through clear hierarchy, consistent patterns, and removal of irrelevant content.

2. **Manage Intrinsic Load Through Sequencing** — Some tasks are inherently complex (filing taxes, configuring a server). You cannot remove intrinsic load, but you can manage it by breaking complex tasks into sequential steps, providing scaffolding, and allowing users to build understanding incrementally.

3. **Promote Germane Load** — Not all cognitive effort is bad. When users are learning a new tool, germane load helps them build lasting mental models. Support this with consistent metaphors, meaningful feedback, and clear conceptual frameworks that reward engagement.

4. **Respect Working Memory Limits** — Working memory holds approximately 4-7 discrete items and fades within 15-30 seconds without rehearsal. Any interface that requires users to hold more than this in mind simultaneously will produce errors, frustration, and abandonment.

5. **Use Recognition Over Recall** — Showing options is cheaper (cognitively) than requiring users to remember them. Dropdown menus, auto-complete, visual cues, and contextual help all shift the burden from recall (high load) to recognition (low load).

### When to Use vs. When to Avoid

**Use when:**
- Designing forms with more than 3-4 fields
- Building onboarding flows or first-run experiences
- Creating dashboards that display multiple data dimensions
- Structuring navigation for complex information architectures
- Writing microcopy, error messages, and instructional text
- Optimizing any flow where users abandon mid-task

**Avoid over-application when:**
- Expert users need power and density (trading platforms, code editors) — oversimplification slows experts down
- Progressive disclosure hides critical information that users need immediately
- Reducing content removes necessary context for decision-making
- The audience has strong existing schemas (domain experts) and chunking is automatic

---

## Frameworks and Methodologies

### 1. The Three-Load Audit — step-by-step
1. **Identify the Task** — Define exactly what the user is trying to accomplish on this screen or flow
2. **Classify Intrinsic Load** — List the inherent elements the user must process (data fields, decisions, concepts). This is the irreducible minimum
3. **Identify Extraneous Load** — Catalog everything on the screen not directly serving the task: decorative elements, ambiguous labels, inconsistent patterns, unnecessary choices, visual clutter
4. **Assess Germane Load** — Determine what the user is learning or building mental models for. Is the design supporting schema formation through consistent patterns and clear feedback?
5. **Reduce Extraneous** — Remove, simplify, or reorganize identified extraneous elements
6. **Sequence Intrinsic** — Break complex tasks into manageable steps with clear progress indication
7. **Support Germane** — Add pattern consistency, contextual help, and meaningful feedback

### 2. Progressive Disclosure Framework — step-by-step
1. **Map all information** — List every piece of data, option, and action on a screen
2. **Tier by necessity** — Classify each element as essential (always visible), secondary (available on demand), or tertiary (advanced/rare)
3. **Design the default** — Show only essential tier on first render
4. **Create expansion paths** — Add "Show more", accordion, or drill-down patterns for secondary tier
5. **Gate tertiary content** — Place advanced options in settings, modal dialogs, or separate views
6. **Validate** — Test whether users can complete the primary task using only the default view

---

## How to Apply

### Decision Checklist
- [ ] Each screen has a single primary task or purpose
- [ ] Form fields are grouped logically with clear section headings
- [ ] No more than 5-7 items require simultaneous comparison or retention
- [ ] Visual hierarchy guides the eye from most to least important
- [ ] Labels and instructions use plain, unambiguous language
- [ ] Complex tasks are broken into steps with progress indication
- [ ] Default states and smart suggestions reduce decision burden
- [ ] Consistent patterns are used across similar interactions
- [ ] Information is presented at the point of need, not all at once
- [ ] Error messages explain the problem and suggest a fix

### Implementation Patterns

**High vs. Low Cognitive Load — Form Design:**
```html
<!-- HIGH COGNITIVE LOAD: All fields at once, no grouping -->
<form class="form--high-load">
  <input type="text" placeholder="First name" />
  <input type="text" placeholder="Last name" />
  <input type="email" placeholder="Email" />
  <input type="tel" placeholder="Phone" />
  <input type="text" placeholder="Street address" />
  <input type="text" placeholder="City" />
  <input type="text" placeholder="State" />
  <input type="text" placeholder="ZIP" />
  <input type="text" placeholder="Card number" />
  <input type="text" placeholder="Expiration" />
  <input type="text" placeholder="CVV" />
  <button type="submit">Submit</button>
</form>

<!-- LOW COGNITIVE LOAD: Chunked, labeled, multi-step -->
<form class="form--low-load">
  <fieldset>
    <legend>1. Personal Information</legend>
    <label for="fname">First name</label>
    <input type="text" id="fname" name="fname" autocomplete="given-name" />
    <label for="lname">Last name</label>
    <input type="text" id="lname" name="lname" autocomplete="family-name" />
    <label for="email">Email address</label>
    <input type="email" id="email" name="email" autocomplete="email" />
  </fieldset>
  <div class="step-controls">
    <span class="progress">Step 1 of 3</span>
    <button type="button" class="btn-next">Continue to Address</button>
  </div>
</form>
```

```css
/* Chunked form styling that reduces cognitive load */
.form--low-load fieldset {
  border: none;
  padding: 0;
  margin-bottom: 2rem;
}

.form--low-load legend {
  font-size: 1.25rem;
  font-weight: 600;
  color: #1f2937;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #e5e7eb;
}

.form--low-load label {
  display: block;
  font-size: 0.875rem;
  font-weight: 500;
  color: #374151;
  margin-bottom: 0.25rem;
}

.form--low-load input {
  display: block;
  width: 100%;
  padding: 0.75rem;
  margin-bottom: 1rem;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 1rem;
}

/* Progress indicator reduces Zeigarnik anxiety */
.progress {
  font-size: 0.875rem;
  color: #6b7280;
}
```

**Progressive Disclosure — Dashboard Pattern:**
```html
<!-- Summary view (low load) with drill-down capability -->
<div class="dashboard">
  <section class="metric-card">
    <h3>Revenue</h3>
    <p class="metric-value">$142,384</p>
    <p class="metric-trend metric-trend--up">+12.5% vs last month</p>
    <button class="btn-details" aria-expanded="false" aria-controls="revenue-details">
      View breakdown
    </button>
    <div id="revenue-details" class="details-panel" hidden>
      <!-- Secondary information revealed on demand -->
      <table>
        <tr><td>Subscriptions</td><td>$98,200</td></tr>
        <tr><td>One-time sales</td><td>$31,584</td></tr>
        <tr><td>Services</td><td>$12,600</td></tr>
      </table>
    </div>
  </section>
</div>
```

```javascript
// Progressive disclosure interaction
document.querySelectorAll('.btn-details').forEach(button => {
  button.addEventListener('click', () => {
    const panel = document.getElementById(button.getAttribute('aria-controls'));
    const isExpanded = button.getAttribute('aria-expanded') === 'true';

    button.setAttribute('aria-expanded', !isExpanded);
    panel.hidden = isExpanded;
    button.textContent = isExpanded ? 'View breakdown' : 'Hide breakdown';
  });
});
```

**Reducing Extraneous Load — Clear Visual Hierarchy:**
```css
/* HIGH EXTRANEOUS LOAD: Everything competes for attention */
.card--cluttered {
  border: 3px solid #ff6b00;
  background: linear-gradient(135deg, #ffecd2, #fcb69f);
  box-shadow: 0 4px 20px rgba(0,0,0,0.3);
  padding: 1rem;
}
.card--cluttered h3 { font-size: 1.5rem; color: #e74c3c; text-transform: uppercase; }
.card--cluttered p { font-size: 1.1rem; font-weight: bold; }
.card--cluttered .badge { animation: pulse 1s infinite; }

/* LOW EXTRANEOUS LOAD: Clear hierarchy, calm design */
.card--clean {
  border: 1px solid #e5e7eb;
  background: #ffffff;
  box-shadow: 0 1px 3px rgba(0,0,0,0.08);
  padding: 1.5rem;
  border-radius: 8px;
}
.card--clean h3 {
  font-size: 1.125rem;
  font-weight: 600;
  color: #111827;
  margin-bottom: 0.5rem;
}
.card--clean p {
  font-size: 0.9375rem;
  color: #4b5563;
  line-height: 1.6;
}
.card--clean .badge {
  font-size: 0.75rem;
  background: #dbeafe;
  color: #1e40af;
  padding: 0.125rem 0.5rem;
  border-radius: 9999px;
}
```

**Onboarding — Manage Intrinsic Load Through Steps:**
```html
<div class="onboarding" role="region" aria-label="Setup wizard">
  <!-- Step indicator manages expectations -->
  <nav class="stepper" aria-label="Progress">
    <ol>
      <li class="stepper__step stepper__step--active" aria-current="step">
        <span class="stepper__number">1</span> Create account
      </li>
      <li class="stepper__step">
        <span class="stepper__number">2</span> Choose plan
      </li>
      <li class="stepper__step">
        <span class="stepper__number">3</span> Set preferences
      </li>
    </ol>
  </nav>

  <!-- Only current step content is shown -->
  <div class="step-content">
    <h2>Create your account</h2>
    <p>Just your email and a password to get started.</p>
    <!-- Minimal fields reduce intrinsic load per step -->
    <label for="onb-email">Email</label>
    <input type="email" id="onb-email" autocomplete="email" />
    <label for="onb-pass">Password</label>
    <input type="password" id="onb-pass" autocomplete="new-password" />
  </div>
</div>
```

### Common Mistakes
1. **Confusing Simplicity with Reduction** — Removing features is not the same as reducing cognitive load. The goal is to organize and sequence complexity, not eliminate necessary functionality. A tax form cannot have fewer fields, but it can present them in digestible groups.
2. **Ignoring Expert Users** — Applying aggressive progressive disclosure to power users who need information density. Expert users have built schemas that let them process high-density interfaces efficiently. Provide density toggles or expert modes.
3. **Overloading with Help Text** — Adding explanations for everything paradoxically increases extraneous load. Use contextual help (tooltips, inline hints) sparingly and only where data shows confusion.
4. **Inconsistent Patterns Across Screens** — Every new pattern forces schema rebuilding. When a form uses inline validation on one page and modal validation on another, users must relearn the interaction model, increasing extraneous load.
5. **Neglecting Germane Load** — Not all effort is bad. Stripping away every learning opportunity leaves users dependent on the interface rather than building competence. Thoughtful empty states, contextual tips, and progressive challenges build lasting understanding.

### Integration with Other Topics
- **08-psychology-for-designers** — CLT sits within broader cognitive psychology; dual-process theory explains why high load forces effortful System 2 thinking, and mental models describe the schemas that reduce load for experts
- **10-laws-of-ux** — Miller's Law (7 plus/minus 2) and Hick's Law (choice overload) are direct applications of cognitive load principles; Doherty Threshold relates to temporal load on working memory
- **25-visual-hierarchy** — Visual hierarchy is the primary tool for reducing extraneous load; clear hierarchy tells users what to process first, preventing cognitive overload
- **29-form-design** — Forms are the most common CLT battleground; chunking, progressive disclosure, inline validation, and smart defaults all derive from cognitive load management
- **12-gestalt-principles** — Gestalt grouping reduces cognitive load by allowing users to perceive related elements as single chunks rather than individual items

---

## Resources

### Essential Reading
- John Sweller, Paul Ayres, Slava Kalyuga, *Cognitive Load Theory* (Springer, 2011)
- George A. Miller, *The Magical Number Seven, Plus or Minus Two* (Psychological Review, 1956)
- Richard Mayer, *Multimedia Learning* (Cambridge University Press, 2nd edition, 2009)
- Steve Krug, *Don't Make Me Think, Revisited* (New Riders, 3rd edition, 2014)
- Susan Weinschenk, *100 Things Every Designer Needs to Know About People* (New Riders, 2011)

### Online Resources
- Nielsen Norman Group — Articles on cognitive load in UX: nngroup.com/topic/cognitive-load
- Interaction Design Foundation — Cognitive Load Theory course
- Laws of UX (lawsofux.com) — Miller's Law and related principles
- Web Content Accessibility Guidelines (WCAG) — Guideline 3.1-3.3 on understandable content

### Practice Exercises
1. **Three-Load Audit**: Take a complex form (insurance quote, tax filing, account setup) and classify every element as intrinsic, extraneous, or germane. Remove or reduce extraneous elements and redesign the flow.
2. **Chunking Challenge**: Take a 15-field form and reorganize it into chunked steps. Measure perceived complexity before and after with 5-second usability tests.
3. **Dashboard Simplification**: Take a data-heavy dashboard and apply progressive disclosure. Create a summary view showing 3-5 key metrics with drill-down paths to detail views.
4. **Onboarding Redesign**: Analyze an app onboarding flow. Count the decisions and new concepts introduced per screen. Redesign so each screen introduces at most one new concept.
5. **Expert vs. Novice Comparison**: Design two versions of the same interface — one optimized for novice users (maximum progressive disclosure) and one for experts (information density). Document the CLT rationale for each design decision.
