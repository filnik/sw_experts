# Design Thinking

## Profile

### What It Is
Design Thinking is a human-centered, iterative problem-solving methodology that uses empathy, creativity, and experimentation to arrive at innovative solutions. It structures the design process into five non-linear stages that move from deep user understanding to rapid prototyping and testing, ensuring that solutions are desirable, feasible, and viable.

### Origin and Evolution
- Rooted in Herbert Simon's *Sciences of the Artificial* (1969) and Horst Rittel's work on wicked problems
- Formalized as a methodology by IDEO and its founder David Kelley in the 1990s
- Popularized by Tim Brown (IDEO CEO) through his 2008 Harvard Business Review article and book *Change by Design* (2009)
- The Hasso Plattner Institute of Design at Stanford (d.school) codified the five-stage model used globally
- The British Design Council introduced the Double Diamond model in 2005
- Has expanded beyond product design into business strategy, education, healthcare, and public policy

### Key Authors and References
- **Tim Brown** — *Change by Design: How Design Thinking Transforms Organizations and Inspires Innovation* (2009)
- **IDEO.org** — *The Field Guide to Human-Centered Design* (2015)
- **Stanford d.school** — bootcamp bootleg / Design Thinking process guide
- **Nigel Cross** — *Design Thinking: Understanding How Designers Think and Work* (2011)
- **British Design Council** — Double Diamond framework (2005, updated 2019)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Empathize | Deeply understand users through observation, interviews, and immersion |
| Define | Synthesize research into a clear problem statement (Point of View) |
| Ideate | Generate a broad range of creative solutions without judgment |
| Prototype | Build quick, low-cost representations of ideas to learn from |
| Test | Gather feedback by putting prototypes in front of real users |
| Double Diamond | Diverge-converge twice: discover/define the problem, then develop/deliver the solution |
| HMW Questions | "How Might We..." reframes problems as opportunities for design |

---

## Core Philosophy

### The 5 Fundamental Principles

1. **Human-Centricity** — Every decision starts with deep empathy for the people who will use the product. Solutions must address real human needs, not assumed ones. Observation and conversation replace assumption.

2. **Bias Toward Action** — Design Thinking favors making and testing over analyzing and debating. A rough prototype tested with five users yields more insight than months of theoretical planning.

3. **Radical Collaboration** — Cross-functional teams bring diverse perspectives. Designers, developers, product managers, and end users co-create solutions. The best ideas emerge from combining different viewpoints.

4. **Embrace Ambiguity** — The process intentionally moves through periods of uncertainty. Divergent phases (Empathize, Ideate) generate breadth; convergent phases (Define, Test) create focus. Comfort with not-knowing is essential.

5. **Iterative Learning** — Design Thinking is non-linear. Testing may send you back to Empathize. Prototyping may redefine the problem. Each cycle deepens understanding and improves the solution.

### When to Use vs. When to Avoid

**Use when:**
- Facing complex, ambiguous problems where the right solution is unclear
- Building new products or features where user needs are not well understood
- Redesigning experiences that have known usability issues
- Aligning cross-functional teams around a shared vision
- Innovating beyond incremental improvements

**Avoid over-application when:**
- The problem is well-defined and the solution is known (e.g., fixing a broken form validation)
- Strict regulatory requirements dictate the exact interface (e.g., medical device displays)
- The team lacks access to real users for testing
- Time constraints make a full Design Thinking cycle impractical (consider Lean UX sprints instead)

---

## Frameworks and Methodologies

### 1. The 5-Stage Process (d.school Model) — step-by-step

**Stage 1: Empathize**
1. Conduct contextual inquiry: observe users in their natural environment
2. Perform in-depth interviews using open-ended questions
3. Create empathy maps: what users Say, Think, Do, and Feel
4. Shadow users through their current workflow
5. Collect artifacts: screenshots, photos, quotes, journey snippets

**Stage 2: Define**
1. Synthesize research findings on a shared wall or board
2. Identify patterns, pain points, and unmet needs
3. Create user personas grounded in research data
4. Write a Point of View (POV) statement: "[User] needs [need] because [insight]"
5. Generate "How Might We" (HMW) questions from the POV

**Stage 3: Ideate**
1. Set divergent thinking rules: defer judgment, go for quantity, build on others' ideas
2. Run brainstorming sessions (timed, 15-20 minutes)
3. Use structured ideation techniques: Crazy 8s, SCAMPER, mind mapping
4. Cluster and vote on ideas using dot voting
5. Select 2-3 ideas to prototype based on feasibility, desirability, and novelty

**Stage 4: Prototype**
1. Choose fidelity level appropriate to the question being answered
2. Build paper prototypes for early concept validation
3. Create wireframes for layout and flow validation
4. Build clickable prototypes for interaction testing
5. Keep prototypes disposable: invest minimum effort for maximum learning

**Stage 5: Test**
1. Recruit 5-8 representative users
2. Set up tasks, not tours: "Find and purchase a blue widget"
3. Observe without intervening; note where users struggle
4. Capture feedback through think-aloud protocol
5. Synthesize findings and decide: iterate, pivot, or proceed

### 2. Double Diamond Model — step-by-step
1. **Discover** (Diverge) — Explore the problem space broadly through research
2. **Define** (Converge) — Narrow down to the core problem worth solving
3. **Develop** (Diverge) — Generate multiple potential solutions
4. **Deliver** (Converge) — Refine and test the best solution for launch

### 3. Design Sprint (adapted from Google Ventures) — step-by-step
1. Monday: Map the problem and pick a target
2. Tuesday: Sketch competing solutions individually
3. Wednesday: Decide on the best approach through structured voting
4. Thursday: Build a realistic prototype
5. Friday: Test with 5 users and capture insights

---

## How to Apply

### Decision Checklist
- [ ] Have we observed and interviewed real users (not just stakeholders)?
- [ ] Is our problem statement based on research, not assumptions?
- [ ] Did we generate at least 20 ideas before selecting one to prototype?
- [ ] Is our prototype the minimum needed to test our riskiest assumption?
- [ ] Did we test with representative users, not colleagues?
- [ ] Have we documented what we learned and what changed as a result?
- [ ] Are we prepared to iterate based on test results?

### Implementation Patterns

**Empathy Map in HTML:**
```html
<!-- Empathy map for synthesis workshops -->
<div class="empathy-map">
  <div class="empathy-map__center">
    <img src="persona-photo.jpg" alt="User persona" />
    <h3>Sarah, 34, Marketing Manager</h3>
  </div>
  <div class="empathy-map__quadrant empathy-map__says">
    <h4>Says</h4>
    <ul>
      <li>"I just want to find things quickly"</li>
      <li>"The dashboard is overwhelming"</li>
    </ul>
  </div>
  <div class="empathy-map__quadrant empathy-map__thinks">
    <h4>Thinks</h4>
    <ul>
      <li>Worries about missing deadlines</li>
      <li>Wonders if there is a faster way</li>
    </ul>
  </div>
  <div class="empathy-map__quadrant empathy-map__does">
    <h4>Does</h4>
    <ul>
      <li>Opens 5 tabs to cross-reference data</li>
      <li>Exports to spreadsheets for analysis</li>
    </ul>
  </div>
  <div class="empathy-map__quadrant empathy-map__feels">
    <h4>Feels</h4>
    <ul>
      <li>Frustrated by slow load times</li>
      <li>Anxious about making data errors</li>
    </ul>
  </div>
</div>
```

```css
/* Empathy map layout */
.empathy-map {
  display: grid;
  grid-template-areas:
    "says    center thinks"
    "does    center feels";
  grid-template-columns: 1fr 200px 1fr;
  grid-template-rows: 1fr 1fr;
  gap: 1rem;
  max-width: 900px;
  margin: 0 auto;
}

.empathy-map__center { grid-area: center; text-align: center; align-self: center; }
.empathy-map__says   { grid-area: says; background: #fef3c7; padding: 1rem; border-radius: 8px; }
.empathy-map__thinks { grid-area: thinks; background: #dbeafe; padding: 1rem; border-radius: 8px; }
.empathy-map__does   { grid-area: does; background: #d1fae5; padding: 1rem; border-radius: 8px; }
.empathy-map__feels  { grid-area: feels; background: #fce7f3; padding: 1rem; border-radius: 8px; }
```

**Rapid Wireframe in CSS (Low-Fidelity):**
```css
/* Wireframe styling: intentionally rough to keep focus on structure */
.wireframe * {
  font-family: 'Courier New', monospace;
  border: 2px solid #666;
  color: #333;
}

.wireframe__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0.75rem 1rem;
  background: #e5e5e5;
}

.wireframe__features {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
  padding: 2rem;
}

.wireframe__card {
  padding: 2rem;
  text-align: center;
  background: #f5f5f5;
  min-height: 120px;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

**Prototype Testing Script (JavaScript):**
```javascript
// Simple prototype click-tracking for usability testing
class PrototypeTracker {
  constructor() {
    this.events = [];
    this.taskStart = null;
  }

  startTask(taskName) {
    this.taskStart = Date.now();
    this.events.push({ type: 'task_start', task: taskName, timestamp: this.taskStart });
  }

  trackClick(element) {
    this.events.push({
      type: 'click', target: element.tagName, id: element.id,
      text: element.textContent?.slice(0, 50),
      elapsed: Date.now() - this.taskStart
    });
  }

  endTask(taskName, success) {
    const duration = Date.now() - this.taskStart;
    this.events.push({ type: 'task_end', task: taskName, success, duration });
  }

  getSummary() {
    const tasks = this.events.filter(e => e.type === 'task_end');
    return {
      successRate: tasks.filter(t => t.success).length / tasks.length,
      avgDuration: tasks.reduce((sum, t) => sum + t.duration, 0) / tasks.length
    };
  }
}
```

### Common Mistakes

1. **Skipping Empathize** — Teams jump to ideation based on assumptions rather than spending time with real users. Without empathy, solutions solve imagined problems.

2. **Falling in Love with the First Idea** — Ideation requires quantity and divergent thinking. Selecting one idea after generating only three defeats the purpose of the phase.

3. **Over-Polishing Prototypes** — High-fidelity prototypes discourage honest feedback. Users hesitate to criticize something that looks finished. Keep prototypes rough enough to invite critique.

4. **Testing for Validation, Not Learning** — Tests should seek to disprove hypotheses, not confirm them. Confirmation bias leads to "the users loved it" conclusions that mask real issues.

5. **Treating Design Thinking as Linear** — The five stages are iterative, not sequential. Test results should loop back to Empathize or Define when fundamental assumptions prove wrong.

### Integration with Other Topics
- **UX Research Methods (07)** — Research methods provide the toolkit for the Empathize and Test stages
- **Lean UX (15)** — Lean UX streamlines Design Thinking for Agile environments, replacing heavy deliverables with lightweight experiments
- **Goal-Directed Design (02)** — Cooper's persona methodology deeply informs the Define stage, giving structure to user characterization
- **Elements of UX (13)** — Design Thinking's stages map to Garrett's planes: Empathize/Define feed Strategy, Ideate feeds Scope, Prototype/Test span Structure through Surface

---

## Resources

### Essential Reading
- *Change by Design* by Tim Brown (2009)
- *The Field Guide to Human-Centered Design* by IDEO.org (2015, free PDF)
- *Sprint: How to Solve Big Problems and Test New Ideas in Just Five Days* by Jake Knapp (2016)
- *Creative Confidence* by Tom Kelley and David Kelley (2013)

### Online Resources
- Stanford d.school resources: dschool.stanford.edu
- IDEO Design Kit: designkit.org
- Google Design Sprint methodology: designsprintkit.withgoogle.com
- Interaction Design Foundation: Design Thinking course

### Practice Exercises
1. **Empathy Immersion** — Spend 30 minutes observing someone use a familiar app (email, calendar). Document pain points they do not articulate but demonstrate through behavior.
2. **Crazy 8s Sprint** — Fold a paper into 8 sections. Set a timer for 8 minutes. Sketch 8 different solutions to a single problem, one per section. Present and discuss.
3. **Paper Prototype Test** — Sketch a 3-screen flow on paper. Recruit a friend. Ask them to complete a task. Note every moment of confusion or hesitation.
4. **Double Diamond Mapping** — Take a recent project and map its activities onto the Double Diamond. Identify where you diverged, where you converged, and where you skipped a phase.
