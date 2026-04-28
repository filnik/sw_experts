# Lean UX

## Profile

### What It Is
Lean UX is a design methodology that combines Lean Startup principles with user experience design, replacing heavy documentation and deliverables with rapid experimentation and cross-functional collaboration. It shifts the focus from producing design artifacts to achieving measurable outcomes through a continuous Think-Make-Check cycle.

### Origin and Evolution
- Grew out of Eric Ries's Lean Startup movement (2008-2011) and its emphasis on validated learning
- Formalized by Jeff Gothelf and Josh Seiden in *Lean UX* (2013)
- Influenced by Agile development's iterative approach and short feedback loops
- Second edition (2016) deepened integration with Agile and Scrum practices
- Third edition (2021) expanded focus on outcomes over outputs and organizational alignment
- Now widely adopted in product teams as the default approach to integrating UX with Agile delivery

### Key Authors and References
- **Jeff Gothelf & Josh Seiden** — *Lean UX: Designing Great Products with Agile Teams* (2013, 3rd ed. 2021)
- **Eric Ries** — *The Lean Startup* (2011)
- **Jeff Gothelf & Josh Seiden** — *Sense and Respond* (2017)
- **Marty Cagan** — *Inspired: How to Create Tech Products Customers Love* (2nd ed. 2018)
- **Jeff Patton** — *User Story Mapping* (2014)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Think-Make-Check | Core cycle: form hypothesis, build minimum experiment, measure results |
| Hypothesis-Driven Design | Every design decision is framed as a testable hypothesis |
| MVP (Minimum Viable Product) | The smallest thing you can build to learn whether your hypothesis is true |
| Outcomes over Outputs | Success is measured by user behavior change, not feature delivery |
| Collaborative Design | Cross-functional teams co-create designs together instead of handing off specs |
| Continuous Discovery | Research is ongoing, not a phase that ends before development begins |
| Experiment Map | Visual tool tracking hypotheses, experiments, and results |

---

## Core Philosophy

### The 5 Fundamental Principles

1. **Outcomes Over Outputs** — Shipping features is not success. Changing user behavior in a measurable way is. Every initiative should target a specific outcome: increased engagement, reduced task time, higher conversion, lower support tickets.

2. **Hypothesis-Driven Design** — Replace "requirements" with hypotheses. Instead of "Build a recommendation engine," say "We believe showing personalized recommendations will increase average order value by 15% for returning customers." This makes design decisions testable and reversible.

3. **Minimum Viable Experiments** — Build the smallest possible thing that will validate or invalidate your hypothesis. This might be a paper prototype, a fake door test, a landing page, or a Wizard of Oz setup. Minimize investment until learning justifies commitment.

4. **Cross-Functional Collaboration** — Designers, developers, and product managers work together throughout the process. There are no handoffs. Shared understanding replaces detailed documentation. Design studios and pair design replace isolated design work.

5. **Continuous Learning Loops** — Learning does not stop when development starts. Every sprint includes user feedback. Every release includes instrumentation. Data from production informs the next hypothesis.

### When to Use vs. When to Avoid

**Use when:**
- Working in Agile or Scrum teams with iterative delivery cycles
- Building products in uncertain markets where user needs are evolving
- Needing to validate concepts before investing engineering time
- Leadership is open to outcome-based success metrics rather than feature checklists
- Teams are cross-functional and co-located (or have strong remote collaboration)

**Avoid over-application when:**
- Regulatory environments require extensive upfront documentation (healthcare, finance)
- Building foundational infrastructure with no direct user-facing component
- The problem is well-understood and the solution is proven (just build it)
- Stakeholders require detailed design specifications before development can begin
- There is no mechanism for collecting user feedback or behavioral data

---

## Frameworks and Methodologies

### 1. Think-Make-Check Cycle — step-by-step
1. **Think** — State your assumptions. Who is the user? What problem do they have? What do you believe will solve it? Write a hypothesis.
2. **Make** — Build the minimum experiment to test the hypothesis. Could be a sketch, wireframe, clickable prototype, fake door test, or coded MVP.
3. **Check** — Put the experiment in front of real users. Measure against your success criteria. Did the hypothesis hold?
4. **Learn** — Synthesize what you learned. Update your assumptions. Feed learnings into the next cycle.
5. **Repeat** — Start the next Think-Make-Check cycle with refined understanding.

### 2. Hypothesis Statement Framework — step-by-step
1. Start with the business outcome: "We will achieve [business outcome]"
2. Identify the target persona: "if [persona]"
3. Define the user goal: "achieves [user goal]"
4. Specify the feature or change: "with [feature/change]"
5. Combine into the template: "We believe [outcome] will happen if [persona] achieves [goal] with [feature]"
6. Define measurable success criteria: "We will know this is true when we see [metric change]"

### 3. Design Studio Method — step-by-step
1. Frame the problem: present the hypothesis and constraints (5 minutes)
2. Individual sketching: each team member sketches 6-8 ideas silently (10 minutes)
3. Present and critique: each person shows sketches, team asks questions and gives feedback (3 minutes each)
4. Iterate individually: refine based on feedback, converge on strongest concepts (10 minutes)
5. Converge as a team: combine the best elements into 1-2 directions
6. Decide: vote on the direction to prototype first

### 4. Lean UX Canvas — step-by-step
1. Define the business problem in one sentence
2. State the business outcome you want to achieve
3. List your target users and their characteristics
4. Document user outcomes and benefits
5. Write your solution ideas
6. Formulate hypotheses connecting solutions to outcomes
7. Identify the riskiest assumption to test first
8. Design the minimum experiment to test it

---

## How to Apply

### Decision Checklist
- [ ] Is our design initiative framed as a hypothesis, not a requirement?
- [ ] Have we defined measurable success criteria before building?
- [ ] Is the experiment we are building the minimum needed to learn?
- [ ] Are designers, developers, and product managers collaborating directly?
- [ ] Do we have a mechanism to measure the outcome after release?
- [ ] Are we prepared to pivot if the hypothesis is invalidated?
- [ ] Is learning documented and shared with the broader team?

### Implementation Patterns

**Hypothesis Board (HTML Structure):**
```html
<!-- Lean UX hypothesis tracking board -->
<div class="hypothesis-board">
  <div class="hypothesis-board__column hypothesis-board__column--backlog">
    <h3>Hypotheses Backlog</h3>
    <div class="hypothesis-card" data-status="backlog">
      <p class="hypothesis-card__statement">
        We believe <strong>reducing checkout steps from 5 to 3</strong>
        will <strong>increase completion rate by 20%</strong>
        for <strong>mobile users</strong>.
      </p>
      <div class="hypothesis-card__risk">Risk: High</div>
      <div class="hypothesis-card__metric">
        Metric: Checkout completion rate (mobile)
      </div>
    </div>
  </div>

  <div class="hypothesis-board__column hypothesis-board__column--testing">
    <h3>Currently Testing</h3>
    <!-- Active experiment cards go here -->
  </div>

  <div class="hypothesis-board__column hypothesis-board__column--validated">
    <h3>Validated</h3>
    <!-- Confirmed hypotheses go here -->
  </div>

  <div class="hypothesis-board__column hypothesis-board__column--invalidated">
    <h3>Invalidated</h3>
    <!-- Disproven hypotheses go here -->
  </div>
</div>
```

```css
/* Hypothesis board layout */
.hypothesis-board {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1rem;
  padding: 1rem;
}

.hypothesis-board__column {
  background: #f9fafb;
  border-radius: 8px;
  padding: 1rem;
  min-height: 400px;
}

.hypothesis-card {
  background: #ffffff;
  border-left: 4px solid #2563eb;
  border-radius: 4px;
  padding: 1rem;
  margin-bottom: 0.75rem;
}

.hypothesis-board__column--validated .hypothesis-card { border-left-color: #16a34a; }
.hypothesis-board__column--invalidated .hypothesis-card { border-left-color: #dc2626; }
```

**Fake Door Test Implementation:**
```html
<!-- Fake door: test demand for a feature before building it -->
<section class="feature-teaser">
  <h2>New: Smart Recommendations</h2>
  <p>Get personalized product suggestions based on your browsing history.</p>
  <button class="feature-teaser__cta" id="fake-door-btn">
    Try It Now
  </button>
</section>

<!-- Modal shown when the fake door is clicked -->
<dialog class="fake-door-modal" id="fake-door-modal">
  <h3>Coming Soon!</h3>
  <p>We are working on this feature. Want to be notified when it launches?</p>
  <form class="fake-door-modal__form">
    <input type="email" placeholder="your@email.com" required />
    <button type="submit">Notify Me</button>
  </form>
  <button class="fake-door-modal__close" aria-label="Close">No thanks</button>
</dialog>
```

```javascript
// Fake door test: measure interest before building
document.getElementById('fake-door-btn').addEventListener('click', () => {
  analytics.track('fake_door_clicked', {
    feature: 'smart_recommendations',
    page: window.location.pathname
  });
  document.getElementById('fake-door-modal').showModal();
});

// Decision criteria: if >5% of page visitors click,
// the feature has sufficient demand to justify building.
```

**Sprint Integration Pattern:**
```javascript
// Lean UX Sprint Structure
const leanUXSprint = {
  sprintLength: '2 weeks',
  structure: {
    day1_2: {
      phase: 'Think',
      activities: [
        'Review previous sprint learnings',
        'Refine or write new hypotheses',
        'Prioritize by riskiest assumption',
        'Define experiment and success criteria'
      ]
    },
    day3_7: {
      phase: 'Make',
      activities: [
        'Collaborative design studio (day 3)',
        'Build experiment: prototype or coded MVP',
        'Developers and designers pair on implementation',
        'Instrument analytics for measurement'
      ]
    },
    day8_9: {
      phase: 'Check',
      activities: [
        'Run usability tests (5 users)',
        'Analyze quantitative data if A/B test',
        'Synthesize qualitative feedback',
        'Document learnings'
      ]
    },
    day10: {
      phase: 'Learn & Plan',
      activities: [
        'Share findings with team and stakeholders',
        'Update hypothesis board: validated / invalidated',
        'Decide: iterate, pivot, or persevere',
        'Seed next sprint backlog with refined hypotheses'
      ]
    }
  }
};
```

### Common Mistakes

1. **Hypothesis Theater** — Writing hypotheses but never actually testing them. The hypothesis format is only valuable if you measure against the stated criteria and act on the results.

2. **MVP Means Low Quality** — A Minimum Viable Product must still be usable and trustworthy. "Minimum" refers to scope, not quality. A buggy, confusing experiment teaches you nothing except that users dislike bugs.

3. **Skipping Collaboration** — Lean UX requires designers and developers to work together, not in sequence. Throwing wireframes over the wall to developers is not Lean UX, it is waterfall with lighter documents.

4. **Measuring Outputs Instead of Outcomes** — Counting features shipped, pages designed, or story points completed misses the point. Lean UX measures whether user behavior changed as predicted.

5. **Abandoning All Documentation** — Lean UX reduces unnecessary documentation but does not eliminate all of it. Hypotheses, experiment results, and learnings must be recorded. Shared understanding fades without lightweight documentation.

### Integration with Other Topics
- **Design Thinking (14)** — Design Thinking provides the empathy and ideation toolkit; Lean UX adds the speed and measurement discipline for Agile contexts
- **UX Research Methods (07)** — Lean UX uses lightweight, rapid research methods: 5-user tests, guerrilla interviews, analytics-driven insights
- **Data-Driven UX (09)** — Lean UX relies on quantitative metrics to validate hypotheses; A/B testing and analytics are core Check-phase tools
- **Elements of UX (13)** — Lean UX can operate at any of Garrett's five planes, testing assumptions about strategy, scope, structure, skeleton, or surface
- **Jobs to Be Done (16)** — JTBD provides a powerful framework for writing Lean UX hypotheses: "We believe [persona] will hire [feature] to do [job]"

---

## Resources

### Essential Reading
- *Lean UX: Designing Great Products with Agile Teams* by Jeff Gothelf & Josh Seiden (3rd edition, 2021)
- *The Lean Startup* by Eric Ries (2011)
- *Sense and Respond* by Jeff Gothelf & Josh Seiden (2017)
- *Inspired* by Marty Cagan (2nd edition, 2018)

### Online Resources
- Jeff Gothelf's blog: jeffgothelf.com
- Lean UX Canvas template: jeffgothelf.com/blog/leanuxcanvas
- Nielsen Norman Group: Lean UX articles
- Mind the Product: articles on outcome-driven product management

### Practice Exercises
1. **Hypothesis Writing Workshop** — Take three features from your current backlog. Rewrite each as a Lean UX hypothesis with persona, outcome, feature, and measurable success criteria.
2. **Fake Door Experiment** — Add a button or link for a feature you are considering. Measure click-through rate over one week. Use the data to decide whether to build it.
3. **Design Studio Session** — Gather 3-5 team members. Frame a real design problem. Run a 45-minute design studio with individual sketching, critique, and convergence.
4. **Lean UX Canvas** — Complete a Lean UX Canvas for your current project. Identify your riskiest assumption. Design the cheapest experiment that could test it.
