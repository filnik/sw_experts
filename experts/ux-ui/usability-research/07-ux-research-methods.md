# UX Research Methods

## Profile

### What It Is
UX research methods are systematic approaches for understanding user behaviors, needs, and motivations through observation, task analysis, and feedback. Erika Hall's "Just Enough Research" philosophy emphasizes doing the right amount of research at the right time — not too little (guessing) and not too much (analysis paralysis). Effective UX research combines qualitative and quantitative methods to build confidence in design decisions while minimizing time and cost.

### Origin and Evolution
- **1980s** — Usability testing emerges from human-computer interaction (HCI) laboratories
- **1990s** — Contextual inquiry and ethnographic methods gain traction in software development
- **2001** — Agile Manifesto shifts research toward leaner, more iterative approaches
- **2008** — Lean Startup methodology introduces rapid experimentation and validated learning
- **2013** — Erika Hall publishes *Just Enough Research*, advocating minimal-viable research approaches
- **2016-present** — Remote and unmoderated research tools (UserTesting, Maze, Optimal Workshop) democratize access to user data

### Key Authors and References
- **Erika Hall** — *Just Enough Research* (2013, 2nd edition 2019)
- **Steve Portigal** — *Interviewing Users: How to Uncover Compelling Insights* (2013)
- **Jeff Sauro & James Lewis** — *Quantifying the User Experience* (2012, 2nd edition 2016)
- **Tomer Sharon** — *It's Our Research* (2012)
- **Erika Hall** — *Conversational Design* (2018)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Generative Research | Explores the problem space to define opportunities before designing |
| Evaluative Research | Tests existing designs or prototypes against user expectations |
| Qualitative | Rich, descriptive data from small samples (why and how) |
| Quantitative | Numerical, measurable data from larger samples (what and how many) |
| Triangulation | Combining multiple methods to increase confidence in findings |
| Research Questions | Specific, answerable questions that drive study design |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Research is not optional** — Every design decision is based on assumptions; research converts assumptions into evidence before you invest in building the wrong thing
2. **Just enough, not exhaustive** — Match research scope to project risk; a 30-minute interview can be more valuable than a 3-month study if timed correctly
3. **Start with questions, not methods** — Define what you need to learn first, then choose the method; picking a method first leads to wasted effort
4. **Involve the whole team** — Research findings have more impact when stakeholders observe sessions directly rather than reading reports weeks later
5. **Bias is unavoidable but manageable** — Every method has limitations; acknowledge them, use triangulation, and separate observation from interpretation

### When to Use vs. When to Avoid

**Use when:**
- Starting a new product or feature and the target audience is not well understood
- Redesigning an existing experience with high bounce rates, low conversion, or user complaints
- Resolving internal disagreements about user needs ("we think users want X" vs "no, they want Y")
- Validating prototypes before committing engineering resources to implementation
- Measuring the impact of design changes post-launch

**Avoid over-application when:**
- The decision is easily reversible and low-risk (test in production instead)
- Research is being used to delay decision-making or avoid accountability
- The team lacks capacity to act on findings (research without follow-through wastes participant time)
- Industry best practices and established patterns already provide a clear answer

---

## Frameworks and Methodologies

### 1. The Four Types of UX Research — when to use each

1. **Generative (Exploratory)** — Discover what to build
   - Methods: stakeholder interviews, field studies, diary studies, contextual inquiry
   - Timing: before defining the problem
   - Output: problem statements, opportunity areas, personas

2. **Descriptive** — Understand the current state
   - Methods: analytics review, surveys, competitive analysis, heuristic evaluation
   - Timing: when assessing an existing product or market
   - Output: benchmarks, user segments, journey maps

3. **Evaluative** — Test if the design works
   - Methods: usability testing, A/B testing, tree testing, first-click testing
   - Timing: during design and development iterations
   - Output: task success rates, severity-rated issues, preference data

4. **Causal** — Prove why something works or does not
   - Methods: controlled experiments, A/B tests with statistical rigor, multivariate testing
   - Timing: when you need to isolate variables and prove causation
   - Output: statistically significant results, confidence intervals

### 2. Research Planning Template — step-by-step
1. **Define the research question** — What specific decision will this research inform?
2. **Choose the method** — Match method to question type (generative, evaluative, etc.)
3. **Define participants** — Who, how many, and recruitment criteria
4. **Write the protocol** — Tasks, questions, or scenarios participants will encounter
5. **Pilot test** — Run 1 session to catch issues with the protocol
6. **Conduct sessions** — Execute research with consistent protocol
7. **Analyze** — Code qualitative data for themes; calculate quantitative metrics
8. **Synthesize and share** — Create actionable findings tied to design recommendations
9. **Track impact** — Measure whether acting on findings improved outcomes

### 3. Core Methods Reference

**Interviews (Generative/Descriptive):**
- 5-8 participants for thematic saturation
- 45-60 minutes each
- Semi-structured: prepared questions with flexibility to follow interesting threads
- Record and transcribe for analysis

**Usability Testing (Evaluative):**
- 5 participants find 80% of usability issues (Nielsen, 2000)
- 15-30 minutes per session
- Think-aloud protocol with task-based scenarios
- Measure: task success, time on task, errors, satisfaction

**Surveys (Descriptive/Evaluative):**
- 100+ responses for quantitative reliability
- Mix closed-ended (Likert scales) and open-ended questions
- Keep under 5 minutes to maintain completion rates
- Use validated instruments (SUS, UMUX-Lite) when possible

**Card Sorting (Generative):**
- 15-30 participants for stable categories
- Open sort: users create and name categories
- Closed sort: users place items into predefined categories
- Reveals user mental models for information architecture

**Tree Testing (Evaluative):**
- 50+ participants for reliable data
- Tests findability within a navigation structure without visual design
- Measures: success rate, directness, time to complete

**A/B Testing (Causal):**
- Hundreds to thousands of users for statistical significance
- Tests one variable at a time between two groups
- Requires clear success metric defined before test begins

**Diary Studies (Generative):**
- 10-15 participants over 1-4 weeks
- Participants log experiences in context over time
- Captures behaviors that do not appear in lab settings

---

## How to Apply

### Decision Checklist
- [ ] Have we defined specific research questions tied to business or design decisions?
- [ ] Is the chosen method appropriate for the type of question (generative vs evaluative)?
- [ ] Is the sample size appropriate for the method (5 for usability, 100+ for surveys)?
- [ ] Has the protocol been pilot tested before full execution?
- [ ] Are stakeholders invited to observe sessions directly?
- [ ] Are findings presented as actionable recommendations, not just observations?
- [ ] Is there a plan to measure whether acting on findings improved outcomes?

### Implementation Patterns

**Usability Test Script Template:**
```javascript
// Usability test session runner — structure for a 30-minute session
const usabilityTestProtocol = {
  introduction: {
    duration: "3 minutes",
    script: `Thanks for coming today. We're testing the [product], not you.
             There are no wrong answers. Please think aloud as you work —
             tell us what you're looking at, what you're trying to do, and
             what you expect to happen. We may ask follow-up questions.
             You can stop at any time. Any questions before we begin?`
  },

  backgroundQuestions: {
    duration: "3 minutes",
    questions: [
      "How often do you [relevant activity]?",
      "What tools or sites do you typically use for [relevant activity]?",
      "Can you describe the last time you [relevant activity]?"
    ]
  },

  tasks: [
    {
      id: 1,
      scenario: "You want to [goal]. Starting from this page, show me how you would do that.",
      successCriteria: "User reaches [target page/state]",
      maxTime: "5 minutes",
      metrics: ["success", "timeOnTask", "errors", "pathTaken"]
    },
    {
      id: 2,
      scenario: "Now imagine you need to [goal]. Please try to do that.",
      successCriteria: "User completes [action]",
      maxTime: "5 minutes",
      metrics: ["success", "timeOnTask", "errors", "pathTaken"]
    },
    {
      id: 3,
      scenario: "You realize you made a mistake. How would you fix it?",
      successCriteria: "User finds undo or correction mechanism",
      maxTime: "3 minutes",
      metrics: ["success", "timeOnTask", "errors"]
    }
  ],

  postTaskQuestions: {
    duration: "5 minutes",
    questions: [
      "Overall, how easy or difficult was this to use? (1-7 scale)",
      "What was the most confusing part?",
      "What, if anything, would you change?",
      "How does this compare to [competitor/current solution]?"
    ]
  },

  sus: {
    // System Usability Scale — 10 standardized questions
    description: "Administer the SUS questionnaire for a benchmark score",
    interpretation: "Average SUS score is 68. Above 80 is excellent."
  }
};
```

**Research Repository — Tracking Findings:**
```html
<!-- Research findings database structure -->
<div class="research-finding" data-method="usability-test" data-date="2024-03-15">
  <div class="finding-header">
    <span class="finding-tag finding-tag--evaluative">Evaluative</span>
    <span class="finding-tag finding-tag--severity-3">Severity: Major</span>
    <span class="finding-confidence">Confidence: High (5/5 participants)</span>
  </div>
  <h3 class="finding-title">Users cannot find the account settings page</h3>
  <p class="finding-observation">
    <strong>Observation:</strong> 5 of 5 participants looked under "Profile"
    for account settings. The actual location is under the gear icon in the
    top-right corner with no text label.
  </p>
  <p class="finding-recommendation">
    <strong>Recommendation:</strong> Add "Settings" as a labeled item in the
    Profile dropdown menu. Consider removing the standalone gear icon.
  </p>
  <p class="finding-impact">
    <strong>Expected impact:</strong> Reduce support tickets for "how do I
    change my password" (currently 15% of all tickets).
  </p>
</div>

<style>
.research-finding {
  border: 1px solid #e2e8f0;
  border-radius: 0.75rem;
  padding: 1.5rem;
  margin-bottom: 1rem;
  background: white;
}
.finding-header {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  margin-bottom: 0.75rem;
}
.finding-tag {
  font-size: 0.75rem;
  padding: 0.2rem 0.6rem;
  border-radius: 1rem;
  font-weight: 600;
}
.finding-tag--evaluative { background: #dbeafe; color: #1d4ed8; }
.finding-tag--severity-3 { background: #fee2e2; color: #dc2626; }
.finding-confidence {
  font-size: 0.75rem;
  color: #64748b;
  margin-left: auto;
}
.finding-title {
  font-size: 1.125rem;
  margin: 0 0 0.75rem;
  color: #0f172a;
}
.finding-observation,
.finding-recommendation,
.finding-impact {
  font-size: 0.9375rem;
  line-height: 1.6;
  color: #334155;
  margin-bottom: 0.5rem;
}
</style>
```

**Qualitative vs Quantitative Decision Matrix:**
```javascript
// Selecting the right method based on research needs
function recommendMethod({ questionType, budget, timeline, sampleAccess }) {
  const methods = {
    "What do users need?": {
      primary: "User Interviews",
      participants: "5-8",
      time: "1-2 weeks",
      cost: "low",
      type: "qualitative"
    },
    "Can users complete the task?": {
      primary: "Usability Testing",
      participants: "5",
      time: "1 week",
      cost: "low-medium",
      type: "qualitative"
    },
    "How satisfied are users?": {
      primary: "Survey (SUS/CSAT)",
      participants: "100+",
      time: "1-2 weeks",
      cost: "low",
      type: "quantitative"
    },
    "Which design performs better?": {
      primary: "A/B Test",
      participants: "1000+",
      time: "2-4 weeks",
      cost: "medium",
      type: "quantitative"
    },
    "How do users organize information?": {
      primary: "Card Sort",
      participants: "15-30",
      time: "1-2 weeks",
      cost: "low",
      type: "mixed"
    },
    "Can users find content?": {
      primary: "Tree Test",
      participants: "50+",
      time: "1 week",
      cost: "low",
      type: "quantitative"
    }
  };

  return methods[questionType] || "Define a clearer research question first.";
}
```

### Common Mistakes
1. **Leading questions in interviews** — Asking "Don't you think the navigation is confusing?" biases the response; ask "Describe how you found [item]" instead
2. **Testing with colleagues instead of real users** — Internal users have too much product knowledge to represent actual users
3. **Confirmation bias in analysis** — Selectively highlighting findings that support a preferred design direction while ignoring contradictory evidence
4. **Too many methods, too little action** — Running every type of research but never implementing findings; one acted-on study beats five shelved reports
5. **Confusing correlation with causation** — Analytics can show that users who saw the tutorial had higher retention, but that does not prove the tutorial caused retention (self-selection bias)
6. **Skipping the pilot test** — Protocol issues found in the first session waste participant time and produce unreliable data

### Integration with Other Topics
- **[Design Thinking (14)](../design-process/14-design-thinking.md)** — Research methods feed directly into the Empathize and Test phases of the design thinking process
- **[Lean UX (15)](../design-process/15-lean-ux.md)** — Lean UX emphasizes rapid hypothesis testing; UX research methods provide the toolkit for validation
- **[Data-Driven UX (09)](../usability-research/09-data-driven-ux.md)** — Quantitative research methods connect directly to UX metrics and measurement frameworks

---

## Resources

### Essential Reading
- Hall, E. (2019). *Just Enough Research*. A Book Apart. 2nd edition.
- Portigal, S. (2013). *Interviewing Users: How to Uncover Compelling Insights*. Rosenfeld Media.
- Sauro, J. & Lewis, J. (2016). *Quantifying the User Experience*. Morgan Kaufmann. 2nd edition.
- Sharon, T. (2012). *It's Our Research: Getting Stakeholder Buy-in for User Experience Research Projects*. Morgan Kaufmann.

### Online Resources
- [Nielsen Norman Group — UX Research Methods](https://www.nngroup.com/articles/which-ux-research-methods/)
- [Mule Design — Research Resources (Erika Hall)](https://www.muledesign.com/)
- [UserTesting — UX Research Repository](https://www.usertesting.com/resources)
- [Optimal Workshop — Card Sorting and Tree Testing Tools](https://www.optimalworkshop.com/)
- [Maze — Rapid Remote Testing Platform](https://maze.co/)

### Practice Exercises
1. **Write 5 research questions** — For a product you use daily, write 5 specific research questions that could improve its UX; classify each as generative, descriptive, evaluative, or causal
2. **Conduct a 5-user usability test** — Recruit 5 people unfamiliar with your project, write 3 task scenarios, run think-aloud sessions, and synthesize the top 3 findings
3. **Run a card sort** — List 30 items from a website and have 15 people sort them into groups; compare the resulting categories to the current site navigation
4. **Create a research plan** — For a feature your team is considering, complete the full planning template: research question, method, participants, protocol, analysis plan, and expected timeline
5. **Triangulation exercise** — Pick one research question and design three different methods to answer it; discuss what each method reveals that the others miss
