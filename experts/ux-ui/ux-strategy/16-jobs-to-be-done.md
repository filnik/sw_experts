# Jobs to Be Done (JTBD)

## Profile

### What It Is
Jobs to Be Done is a framework for understanding why customers adopt, use, or abandon products by focusing on the underlying "job" they are trying to accomplish rather than on demographics or product features. It reframes product design around the progress a user is trying to make in a given circumstance, revealing innovation opportunities that traditional feature-driven thinking misses.

### Origin and Evolution
- Rooted in Theodore Levitt's marketing insight: "People don't want a quarter-inch drill, they want a quarter-inch hole"
- Pioneered by Clayton Christensen in his disruption theory work at Harvard Business School (1990s-2000s)
- Tony Ulwick developed the parallel Outcome-Driven Innovation (ODI) methodology starting in 1991
- Bob Moesta and Chris Spiek contributed the "Switch" interview technique for studying actual purchasing decisions
- Christensen formalized the framework in *Competing Against Luck* (2016)
- Ulwick published the comprehensive ODI methodology in *Jobs to Be Done: Theory to Practice* (2016)
- Now widely adopted in product management, UX design, and innovation strategy

### Key Authors and References
- **Clayton Christensen** — *Competing Against Luck: The Story of Innovation and Customer Choice* (2016)
- **Tony Ulwick** — *What Customers Want: Using Outcome-Driven Innovation to Create Breakthrough Products and Services* (2005)
- **Tony Ulwick** — *Jobs to Be Done: Theory to Practice* (2016)
- **Bob Moesta & Chris Spiek** — Switch interviews methodology; *Demand-Side Sales 101* (2020)
- **Alan Klement** — *When Coffee and Kale Compete* (2018)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Functional Job | The practical task the user is trying to accomplish |
| Emotional Job | How the user wants to feel (or avoid feeling) during and after the task |
| Social Job | How the user wants to be perceived by others |
| Job Statement | Structured format: "When [situation], I want to [motivation], so I can [outcome]" |
| Outcome-Driven Innovation | Ulwick's method of mapping desired outcomes to measure unmet needs |
| Hiring and Firing | Users "hire" a product to do a job and "fire" it when something better comes along |
| Forces of Progress | Push, pull, anxiety, and habit forces that drive or prevent product adoption |

---

## Core Philosophy

### The 5 Fundamental Principles

1. **People Buy Progress, Not Products** — Users do not want your app, your feature, or your widget. They want to make progress in their lives. A job is the progress a person is trying to make in a particular circumstance. Design for the progress, not the product.

2. **Jobs Are Stable, Solutions Change** — The job of "getting from point A to point B quickly" has existed for centuries. The solutions have changed from horses to cars to ride-sharing apps. Anchoring your design to the job rather than a current solution opens innovation possibilities.

3. **Context Determines the Job** — The same person has different jobs in different situations. A commuter listening to a podcast in the car has a different job than the same person listening at the gym. Circumstance (when, where, with whom, while doing what) is essential context.

4. **Competing Against Non-Consumption** — Your real competition is often not another product but the absence of a solution. Many users tolerate a bad situation rather than adopting a new tool. Understanding what holds them back reveals the true barriers to adoption.

5. **Three Dimensions of Every Job** — Every job has functional, emotional, and social dimensions. A project management tool must help users organize tasks (functional), feel in control (emotional), and appear competent to colleagues (social). Ignoring any dimension leaves the job partially undone.

### When to Use vs. When to Avoid

**Use when:**
- Defining product strategy and deciding what to build next
- Understanding why users adopt competitor products or abandon yours
- Prioritizing features based on unmet user needs rather than internal requests
- Designing onboarding flows that align with users' motivations for signing up
- Conducting competitive analysis beyond feature comparison

**Avoid over-application when:**
- Optimizing existing interfaces where the job is already well-served (focus on usability instead)
- Making visual design decisions (JTBD informs structure and features, not pixels)
- Working on technical infrastructure with no direct user-facing component
- The product space is mature and well-understood, and incremental improvement is the goal

---

## Frameworks and Methodologies

### 1. Job Statement Construction — step-by-step
1. Identify the situation or trigger: "When [circumstance]..."
2. Define the motivation: "I want to [action/goal]..."
3. State the expected outcome: "So I can [desired result]..."
4. Validate the three dimensions: is the functional, emotional, and social job addressed?
5. Test with users: does this statement resonate when read back to them?

### 2. Switch Interview Technique (Moesta) — step-by-step
1. Recruit users who recently adopted or abandoned a product
2. Timeline-map backward from purchase: first thought, passive looking, active looking, deciding
3. Identify the trigger event that moved them from passive to active evaluation
4. Map the four forces: push (current pain), pull (new appeal), anxiety (fear of change), habit (inertia)
5. Evaluate: do Push + Pull outweigh Anxiety + Habit? Design to amplify drivers and reduce resistors

### 3. Outcome-Driven Innovation (Ulwick) — step-by-step
1. Define the core functional job (e.g., "manage personal finances")
2. Create a job map: break the job into steps (define, locate, prepare, execute, monitor, conclude)
3. For each step, identify desired outcomes: "Minimize the time it takes to [outcome]"
4. Survey users to rate each outcome on importance and current satisfaction
5. Calculate opportunity score: Importance + max(Importance - Satisfaction, 0)
6. Prioritize features that address high-opportunity (high importance, low satisfaction) outcomes

---

## How to Apply

### Decision Checklist
- [ ] Have we identified the core job our users are hiring our product to do?
- [ ] Is the job stated from the user's perspective, not the product's perspective?
- [ ] Have we addressed functional, emotional, and social dimensions?
- [ ] Do we understand the context (circumstance) in which the job arises?
- [ ] Have we mapped the forces of progress: push, pull, anxiety, habit?
- [ ] Are features prioritized by unmet need (high importance, low satisfaction)?
- [ ] Does our onboarding flow connect to the moment the job was "hired"?

### Implementation Patterns

**Job-Driven Navigation Structure:**
```html
<!-- Bad: feature-oriented navigation -->
<nav class="nav--feature-driven">
  <a href="/reports">Reports</a>
  <a href="/analytics">Analytics</a>
  <a href="/settings">Settings</a>
</nav>

<!-- Good: job-oriented navigation (labels reflect user goals) -->
<nav class="nav--job-driven" aria-label="Main navigation">
  <a href="/track-progress">Track My Progress</a>
  <a href="/find-insights">Find Insights</a>
  <a href="/share-results">Share Results</a>
</nav>
```

```css
/* Job-driven navigation emphasizes user goals over feature names */
.nav--job-driven {
  display: flex;
  gap: 0.5rem;
  padding: 0.75rem 1.5rem;
  background: #f8fafc;
  border-bottom: 1px solid #e2e8f0;
}

.nav--job-driven a {
  padding: 0.5rem 1rem;
  color: #334155;
  text-decoration: none;
  border-radius: 6px;
  font-weight: 500;
}

.nav--job-driven a[aria-current="page"] {
  background: #2563eb;
  color: #ffffff;
}
```

**Job Story Cards for Feature Prioritization:**
```html
<!-- Job story card: maps job statement to opportunity score -->
<div class="job-card" data-opportunity-score="14">
  <div class="job-card__header">
    <span class="job-card__type job-card__type--functional">Functional</span>
    <span class="job-card__score">Opportunity: 14/20</span>
  </div>
  <div class="job-card__statement">
    <p><strong>When</strong> I need to find a product quickly,<br>
       <strong>I want</strong> an effective search with relevant filters,<br>
       <strong>So I can</strong> complete my purchase without frustration.</p>
  </div>
  <div class="job-card__metric">
    <span>Importance</span>
    <div class="job-card__bar" style="--value: 90%"></div>
    <span>9/10</span>
  </div>
  <div class="job-card__metric">
    <span>Satisfaction</span>
    <div class="job-card__bar job-card__bar--low" style="--value: 40%"></div>
    <span>4/10</span>
  </div>
</div>
```

```css
/* Job story card styling */
.job-card {
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1.25rem;
  background: #ffffff;
  max-width: 420px;
}

.job-card__header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.job-card__type {
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  padding: 0.2rem 0.5rem;
  border-radius: 4px;
}

.job-card__type--functional { background: #dbeafe; color: #1e40af; }
.job-card__type--emotional  { background: #fce7f3; color: #9d174d; }
.job-card__type--social     { background: #d1fae5; color: #065f46; }

.job-card__statement {
  background: #f8fafc;
  padding: 1rem;
  border-radius: 6px;
  line-height: 1.6;
}

.job-card__metric {
  display: grid;
  grid-template-columns: 100px 1fr 40px;
  align-items: center;
  gap: 0.5rem;
}

/* Progress bar uses CSS custom property for value */
.job-card__bar {
  height: 8px;
  background: #e2e8f0;
  border-radius: 4px;
  position: relative;
}

.job-card__bar::after {
  content: '';
  position: absolute;
  inset: 0;
  width: var(--value);
  background: #2563eb;
  border-radius: 4px;
}

.job-card__bar--low::after { background: #dc2626; }
```

**Opportunity Scoring Calculator:**
```javascript
// Outcome-Driven Innovation: calculate opportunity scores
// Ulwick's formula: Score = Importance + max(Importance - Satisfaction, 0)
function calculateOpportunityScore(importance, satisfaction) {
  return importance + Math.max(importance - satisfaction, 0);
}

// Example: e-commerce search job outcomes
const outcomes = [
  { name: 'Find relevant results on first query', importance: 9.2, satisfaction: 4.1 },
  { name: 'Filter results by specific criteria',  importance: 8.5, satisfaction: 5.3 },
  { name: 'See product availability instantly',    importance: 8.8, satisfaction: 7.2 },
  { name: 'Compare results side by side',          importance: 7.1, satisfaction: 3.0 },
  { name: 'Save search for later',                 importance: 5.2, satisfaction: 6.8 }
];

const prioritized = outcomes
  .map(o => ({ ...o, score: calculateOpportunityScore(o.importance, o.satisfaction) }))
  .sort((a, b) => b.score - a.score);

// Result: "Find relevant results" scores highest (14.3) = underserved
// Segments: >12 = underserved (invest here), 8-12 = served, <8 = overserved
```

**Forces of Progress Visualization:**
```html
<!-- Forces of progress: 2x2 grid showing drivers and resistors of product switching -->
<div class="forces-diagram">
  <h3>Why Users Switch to Our Product</h3>
  <div class="forces-diagram__grid">
    <div class="force force--push">
      <h4>Push (Current Pain)</h4>
      <ul>
        <li>Current tool too slow for large datasets</li>
        <li>No real-time collaboration</li>
      </ul>
    </div>
    <div class="force force--pull">
      <h4>Pull (New Appeal)</h4>
      <ul>
        <li>Instant search across all data</li>
        <li>Real-time team dashboards</li>
      </ul>
    </div>
    <div class="force force--anxiety">
      <h4>Anxiety (Fears)</h4>
      <ul>
        <li>Will my data migrate safely?</li>
        <li>Learning curve for the team</li>
      </ul>
    </div>
    <div class="force force--habit">
      <h4>Habit (Inertia)</h4>
      <ul>
        <li>Team already knows the old tool</li>
        <li>"It works well enough"</li>
      </ul>
    </div>
  </div>
</div>
```

```css
/* Forces of progress: Push + Pull drive switching; Anxiety + Habit resist it */
.forces-diagram__grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.force { padding: 1rem; border-radius: 8px; }
.force--push    { background: #fef2f2; border-left: 4px solid #ef4444; }
.force--pull    { background: #f0fdf4; border-left: 4px solid #22c55e; }
.force--anxiety { background: #fefce8; border-left: 4px solid #eab308; }
.force--habit   { background: #f5f3ff; border-left: 4px solid #8b5cf6; }
```

### Common Mistakes

1. **Confusing Jobs with Tasks** — "Upload a CSV file" is a task. "Get data into the system quickly so I can start analyzing" is a job. Jobs describe desired progress; tasks describe interactions with the current solution.

2. **Ignoring Emotional and Social Jobs** — A project management tool that only addresses functional jobs (track tasks, set deadlines) but ignores emotional jobs (feel in control, reduce anxiety) and social jobs (appear organized to the team) misses critical adoption drivers.

3. **Writing Jobs from the Product's Perspective** — "Increase user engagement" is a business goal, not a job. Jobs are always written from the user's point of view: "Stay informed about my project status without checking multiple tools."

4. **Using JTBD for Visual Design Decisions** — JTBD informs what to build and why, not how it looks. Use the framework for strategy and feature prioritization, then hand off to visual design principles for the surface layer.

5. **Treating All Jobs as Equal** — Not all jobs are equally important or underserved. Without Ulwick's opportunity scoring or a similar prioritization method, teams spread effort across low-impact jobs and miss the high-opportunity ones.

### Integration with Other Topics
- **Goal-Directed Design (02)** — Cooper's personas define who has the job; JTBD defines what progress they are trying to make. Together they provide a complete picture of user motivation.
- **Elements of UX (13)** — JTBD directly informs the Strategy plane (user needs) and the Scope plane (which features address which jobs)
- **Lean UX (15)** — JTBD provides the "why" behind Lean UX hypotheses: "We believe [persona] will hire [feature] to do [job], which we will know when [outcome metric] improves"
- **Data-Driven UX (09)** — Opportunity scoring requires quantitative data on importance and satisfaction, bridging JTBD qualitative insights with data-driven prioritization

---

## Resources

### Essential Reading
- *Competing Against Luck* by Clayton Christensen et al. (2016)
- *What Customers Want* by Tony Ulwick (2005)
- *Jobs to Be Done: Theory to Practice* by Tony Ulwick (2016)
- *When Coffee and Kale Compete* by Alan Klement (2018)
- *Demand-Side Sales 101* by Bob Moesta (2020)

### Online Resources
- Tony Ulwick's Strategyn blog: strategyn.com/jobs-to-be-done
- JTBD Toolkit: jtbd.info
- Intercom on Jobs to Be Done: intercom.com/books/jobs-to-be-done
- Harvard Business School working knowledge: Christensen's JTBD articles

### Practice Exercises
1. **Job Statement Workshop** — For three features in your current product, write the underlying job statement using the "When... I want... So I can..." format. Verify with user interviews.
2. **Switch Interview** — Interview someone who recently switched to or from a product (any product). Map the timeline, identify the four forces, and find the "first thought" moment.
3. **Opportunity Scoring** — List 10 desired outcomes for a job your product serves. Survey 20 users on importance and satisfaction. Calculate opportunity scores and compare the ranking to your current roadmap priorities.
4. **Competitive Job Analysis** — Choose a competitor product. Identify the core job it is hired for. Map the functional, emotional, and social dimensions. Find a dimension your product could serve better.
