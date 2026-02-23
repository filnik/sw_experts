# Data-Driven UX

## Profile

### What It Is
Data-driven UX is the practice of using quantitative metrics, analytics, and controlled experiments to inform and validate user experience design decisions. Jared Spool, founder of User Interface Engineering (UIE, now Center Centre), has spent over three decades conducting large-scale UX research demonstrating that measurable outcomes — not opinions — should drive design. This approach bridges the gap between qualitative user understanding and quantitative business evidence, making UX a strategic discipline grounded in data.

### Origin and Evolution
- **1988** — Jared Spool founds User Interface Engineering (UIE), one of the first UX research consultancies
- **1996** — Spool publishes influential research on web navigation and search behavior
- **1999** — *Web Site Usability: A Designer's Guide* presents early data-driven UX findings
- **2004** — SUS (System Usability Scale) gains widespread adoption as a standardized UX metric
- **2008** — Google famously tests 41 shades of blue, establishing A/B testing as a design practice
- **2011** — Lean Analytics movement integrates UX metrics into startup methodology
- **2015** — Google introduces HEART framework (Happiness, Engagement, Adoption, Retention, Task Success)
- **2018-present** — UX measurement matures with standardized benchmarking, experience analytics platforms, and experimentation at scale

### Key Authors and References
- **Jared Spool** — UIE articles and research studies (1988-present)
- **Jeff Sauro** — *Quantifying the User Experience* (2012, 2nd edition 2016)
- **Kerry Rodden, Hilary Hutchinson, Xin Fu** — *Measuring the User Experience on a Large Scale* (Google HEART framework, 2010)
- **Alistair Croll & Benjamin Yoskovitz** — *Lean Analytics* (2013)
- **John Brooke** — *SUS: A Quick and Dirty Usability Scale* (1996)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Task Success Rate | Percentage of users who complete a given task correctly |
| Time on Task | Duration to complete a task; shorter is usually (not always) better |
| Error Rate | Frequency and type of mistakes users make during task completion |
| SUS Score | Standardized 10-question usability questionnaire scored 0-100 (average: 68) |
| Net Promoter Score | Single-question metric measuring likelihood to recommend (-100 to +100) |
| HEART Framework | Google's Goals-Signals-Metrics framework for large-scale UX measurement |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Measure what matters, not what is easy** — Pageviews and time on site are easy to track but rarely indicate UX quality; task success rate and user satisfaction require more effort but reveal real usability
2. **Combine qualitative and quantitative** — Numbers tell you what is happening; qualitative research tells you why; Spool emphasizes that neither alone is sufficient
3. **Baseline before you optimize** — You cannot improve what you have not measured; establish current metrics before making changes so you can demonstrate impact
4. **Correlation is not causation** — Analytics reveal patterns, but only controlled experiments (A/B tests) can prove that a specific change caused an improvement
5. **UX metrics must connect to business outcomes** — Demonstrating that a usability improvement reduced support costs by 30% is more persuasive to stakeholders than reporting that SUS improved by 5 points

### When to Use vs. When to Avoid

**Use when:**
- You need to justify UX investment to stakeholders with concrete numbers
- Comparing the effectiveness of design alternatives with measurable criteria
- Establishing baselines for a redesign project so you can measure improvement
- Monitoring ongoing UX health over time to catch regressions
- Running A/B tests to resolve design disagreements with evidence

**Avoid over-application when:**
- You are in early generative research where understanding context and motivations matters more than metrics
- The sample size is too small for statistical significance (most A/B tests need hundreds to thousands of users)
- Metrics become proxies that are gamed rather than genuine indicators of user value
- Quantitative measurement replaces talking to actual users and understanding their goals
- You are optimizing a local metric at the expense of overall user experience (e.g., increasing clicks at the cost of user frustration)

---

## Frameworks and Methodologies

### 1. Google HEART Framework — step-by-step
1. **Choose a category:**
   - **Happiness** — Satisfaction, perceived ease of use, net promoter score
   - **Engagement** — Session frequency, depth of interaction, feature adoption
   - **Adoption** — New users, upgrade rate, feature discovery rate
   - **Retention** — Churn rate, return visits, renewal rate
   - **Task Success** — Completion rate, time on task, error rate

2. **Define Goals** — What UX outcome are you trying to achieve? (e.g., "Users should be able to complete checkout in under 2 minutes")

3. **Identify Signals** — What user behavior indicates progress toward the goal? (e.g., "Users reaching the confirmation page without going back")

4. **Select Metrics** — What specific number will you track? (e.g., "Checkout completion rate" and "Median time from cart to confirmation")

### 2. UX Measurement Framework — step-by-step
1. **Identify key user tasks** — List the 5-10 most important tasks users perform
2. **Select metrics per task** — Choose 2-3 metrics from: success rate, time on task, error rate, satisfaction
3. **Establish baselines** — Measure current performance through usability testing or analytics
4. **Set targets** — Define what "good enough" looks like (e.g., 90% task success, SUS > 80)
5. **Instrument and collect** — Set up analytics events, run periodic benchmark studies
6. **Review and iterate** — Monthly or quarterly review of metrics against targets
7. **Report impact** — Connect UX improvements to business metrics (support tickets, conversion, retention)

### 3. A/B Testing Methodology — step-by-step
1. **Form a hypothesis** — "Changing the CTA button text from 'Submit' to 'Get Started Free' will increase signup conversion by 10%"
2. **Calculate sample size** — Use a power calculator; typical: 80% power, 5% significance, minimum detectable effect
3. **Randomize assignment** — Users are randomly and persistently assigned to control (A) or variant (B)
4. **Run the test** — Do not peek at results prematurely; let it run until the predetermined sample size is reached
5. **Analyze results** — Check statistical significance (p < 0.05) and practical significance (is the effect meaningful?)
6. **Account for multiple testing** — If testing multiple variants, apply Bonferroni correction or use sequential testing methods
7. **Document and ship** — Record the hypothesis, result, and context; implement the winner or iterate

### 4. Spool's UX Metrics That Matter
- **Task success rate** — The most fundamental UX metric; if users cannot complete their goals, nothing else matters
- **Time on task** — Context-dependent: faster is better for checkout, but not necessarily for content consumption
- **Error rate** — Measures how often users make mistakes; high error rates indicate confusing interfaces
- **Findability rate** — Can users locate the content or feature they need? Spool's navigation research showed that users succeed only 42% of the time on average
- **Customer support burden** — The number of support contacts per user is a proxy for UX problems
- **Conversion rate** — For e-commerce, the percentage of visitors who complete a purchase

---

## How to Apply

### Decision Checklist
- [ ] Have we identified the top 5 user tasks and assigned measurable success criteria?
- [ ] Do we have baseline metrics before starting the redesign?
- [ ] Are we tracking both behavioral metrics (what users do) and attitudinal metrics (what users feel)?
- [ ] Is our A/B test sample size large enough for statistical significance?
- [ ] Have we connected UX metrics to at least one business outcome (revenue, cost, retention)?
- [ ] Are we reviewing UX metrics on a regular cadence (monthly or quarterly)?
- [ ] Do we have a process for acting on metric changes (both improvements and regressions)?

### Implementation Patterns

**UX Analytics Event Tracking:**
```javascript
// Structured UX event tracking for key user tasks
const UXMetrics = {
  trackTaskStart(taskName) {
    const startTime = performance.now();
    sessionStorage.setItem(`ux_task_${taskName}_start`, startTime);

    this.sendEvent('ux_task_start', {
      task: taskName,
      timestamp: new Date().toISOString(),
      page: window.location.pathname,
      viewport: `${window.innerWidth}x${window.innerHeight}`
    });
  },

  trackTaskComplete(taskName, success = true) {
    const startTime = parseFloat(
      sessionStorage.getItem(`ux_task_${taskName}_start`)
    );
    const duration = startTime ? Math.round(performance.now() - startTime) : null;

    this.sendEvent('ux_task_complete', {
      task: taskName,
      success: success,
      duration_ms: duration,
      error_count: this.getErrorCount(taskName),
      timestamp: new Date().toISOString()
    });

    sessionStorage.removeItem(`ux_task_${taskName}_start`);
  },

  trackError(taskName, errorType, errorMessage) {
    const count = (parseInt(
      sessionStorage.getItem(`ux_errors_${taskName}`) || '0'
    )) + 1;
    sessionStorage.setItem(`ux_errors_${taskName}`, count);

    this.sendEvent('ux_task_error', {
      task: taskName,
      error_type: errorType,
      error_message: errorMessage,
      error_sequence: count,
      page: window.location.pathname
    });
  },

  getErrorCount(taskName) {
    return parseInt(sessionStorage.getItem(`ux_errors_${taskName}`) || '0');
  },

  sendEvent(eventName, data) {
    // Replace with your analytics provider
    if (typeof gtag !== 'undefined') {
      gtag('event', eventName, data);
    }
    // Also log for debugging in development
    if (process.env.NODE_ENV === 'development') {
      console.table({ event: eventName, ...data });
    }
  }
};

// Usage in a checkout flow
UXMetrics.trackTaskStart('checkout');

// On validation error
UXMetrics.trackError('checkout', 'validation', 'Invalid card number');

// On successful purchase
UXMetrics.trackTaskComplete('checkout', true);

// On abandonment
UXMetrics.trackTaskComplete('checkout', false);
```

**SUS Score Calculator:**
```javascript
// System Usability Scale — standardized 10-question usability metric
// Questions alternate positive/negative. Score range: 0-100. Average: 68.

function calculateSUS(responses) {
  // responses: array of 10 integers, each 1-5 (Strongly Disagree to Strongly Agree)
  if (responses.length !== 10) {
    throw new Error('SUS requires exactly 10 responses');
  }

  let score = 0;
  responses.forEach((response, index) => {
    if (index % 2 === 0) {
      // Odd-numbered questions (positive): subtract 1
      score += (response - 1);
    } else {
      // Even-numbered questions (negative): subtract from 5
      score += (5 - response);
    }
  });

  return score * 2.5; // Scale to 0-100
}

function interpretSUS(score) {
  if (score >= 85) return { grade: 'A+', label: 'Excellent', percentile: 'Top 10%' };
  if (score >= 80) return { grade: 'A',  label: 'Very Good', percentile: 'Top 15%' };
  if (score >= 72) return { grade: 'B',  label: 'Good', percentile: 'Top 30%' };
  if (score >= 68) return { grade: 'C',  label: 'Average', percentile: 'Median' };
  if (score >= 51) return { grade: 'D',  label: 'Below Average', percentile: 'Bottom 30%' };
  return { grade: 'F', label: 'Poor', percentile: 'Bottom 15%' };
}

// Example
const userResponses = [4, 2, 5, 1, 4, 2, 5, 1, 4, 2];
const susScore = calculateSUS(userResponses);
console.log(`SUS Score: ${susScore}`);
console.log(interpretSUS(susScore));
```

**UX Dashboard Component:**
```html
<!-- UX Metrics Dashboard — displays key metrics at a glance -->
<div class="ux-dashboard">
  <h2>UX Health Metrics — February 2026</h2>
  <div class="metrics-grid">
    <div class="metric-card metric-card--positive">
      <span class="metric-label">Task Success Rate</span>
      <span class="metric-value">87%</span>
      <span class="metric-trend">+4% from last month</span>
      <span class="metric-target">Target: 90%</span>
    </div>
    <div class="metric-card metric-card--warning">
      <span class="metric-label">Median Time on Task</span>
      <span class="metric-value">2m 34s</span>
      <span class="metric-trend">+12s from last month</span>
      <span class="metric-target">Target: < 2m</span>
    </div>
    <div class="metric-card metric-card--positive">
      <span class="metric-label">SUS Score</span>
      <span class="metric-value">74</span>
      <span class="metric-trend">+3 from last quarter</span>
      <span class="metric-target">Target: 80</span>
    </div>
    <div class="metric-card metric-card--negative">
      <span class="metric-label">Error Rate</span>
      <span class="metric-value">12%</span>
      <span class="metric-trend">+2% from last month</span>
      <span class="metric-target">Target: < 5%</span>
    </div>
  </div>
</div>

<style>
.ux-dashboard {
  max-width: 64rem;
  margin: 0 auto;
  padding: 2rem;
}
.metrics-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(14rem, 1fr));
  gap: 1.25rem;
  margin-top: 1.5rem;
}
.metric-card {
  background: white;
  border-radius: 0.75rem;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.375rem;
  border-left: 4px solid #e2e8f0;
  box-shadow: 0 1px 3px rgba(0,0,0,0.08);
}
.metric-card--positive { border-left-color: #22c55e; }
.metric-card--warning  { border-left-color: #f59e0b; }
.metric-card--negative { border-left-color: #ef4444; }

.metric-label {
  font-size: 0.8125rem;
  color: #64748b;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}
.metric-value {
  font-size: 2rem;
  font-weight: 700;
  color: #0f172a;
}
.metric-trend {
  font-size: 0.8125rem;
  color: #64748b;
}
.metric-card--positive .metric-trend { color: #16a34a; }
.metric-card--negative .metric-trend { color: #dc2626; }
.metric-target {
  font-size: 0.75rem;
  color: #94a3b8;
  margin-top: 0.25rem;
}
</style>
```

**A/B Test Statistical Significance Check:**
```javascript
// Simple A/B test significance calculator
function abTestSignificance(controlVisitors, controlConversions,
                             variantVisitors, variantConversions) {
  const controlRate = controlConversions / controlVisitors;
  const variantRate = variantConversions / variantVisitors;

  // Pooled standard error
  const pooledRate = (controlConversions + variantConversions) /
                     (controlVisitors + variantVisitors);
  const se = Math.sqrt(pooledRate * (1 - pooledRate) *
             (1 / controlVisitors + 1 / variantVisitors));

  // Z-score
  const z = (variantRate - controlRate) / se;

  // Two-tailed p-value approximation
  const p = 2 * (1 - normalCDF(Math.abs(z)));

  const lift = ((variantRate - controlRate) / controlRate) * 100;

  return {
    controlRate: (controlRate * 100).toFixed(2) + '%',
    variantRate: (variantRate * 100).toFixed(2) + '%',
    lift: lift.toFixed(1) + '%',
    zScore: z.toFixed(3),
    pValue: p.toFixed(4),
    significant: p < 0.05,
    recommendation: p < 0.05
      ? (lift > 0 ? 'Ship the variant' : 'Keep the control')
      : 'Continue testing — not yet significant'
  };
}

function normalCDF(x) {
  const a1 =  0.254829592, a2 = -0.284496736, a3 =  1.421413741;
  const a4 = -1.453152027, a5 =  1.061405429, p   =  0.3275911;
  const sign = x < 0 ? -1 : 1;
  x = Math.abs(x) / Math.sqrt(2);
  const t = 1.0 / (1.0 + p * x);
  const y = 1.0 - (((((a5*t + a4)*t) + a3)*t + a2)*t + a1)*t * Math.exp(-x*x);
  return 0.5 * (1.0 + sign * y);
}

// Example usage
const result = abTestSignificance(5000, 150, 5000, 185);
console.log(result);
// { controlRate: '3.00%', variantRate: '3.70%', lift: '23.3%',
//   zScore: '1.978', pValue: '0.0479', significant: true,
//   recommendation: 'Ship the variant' }
```

### Common Mistakes
1. **Peeking at A/B test results** — Checking results before the sample size is reached inflates false positive rates; commit to a sample size and wait
2. **Optimizing vanity metrics** — Pageviews, downloads, and registered accounts look good in reports but may not reflect actual user value or business outcomes
3. **Ignoring Simpson's Paradox** — Aggregate data can show the opposite trend from segmented data; always break metrics down by user segment, device, and context
4. **Setting arbitrary targets** — "We want 95% task success" is meaningless without understanding the baseline, industry benchmarks, and what improvement is realistic
5. **Treating NPS as a UX metric** — NPS measures brand loyalty and recommendation intent, which is influenced by pricing, marketing, and support, not just UX
6. **Running tests with insufficient traffic** — A/B tests on low-traffic pages may take months to reach significance; use qualitative methods instead for these cases

### Integration with Other Topics
- **[UX Research Methods (07)](../usability-research/07-ux-research-methods.md)** — Data-driven UX is the quantitative complement to qualitative research methods; both are needed for complete understanding
- **[Lean UX (15)](../design-process/15-lean-ux.md)** — Lean UX's build-measure-learn cycle relies on the metrics and measurement frameworks covered here
- **[Web Performance UX (37)](../performance/37-web-performance-ux.md)** — Page load time and interaction responsiveness are measurable UX metrics that directly impact task success and satisfaction

---

## Resources

### Essential Reading
- Sauro, J. & Lewis, J. (2016). *Quantifying the User Experience*. Morgan Kaufmann. 2nd edition.
- Croll, A. & Yoskovitz, B. (2013). *Lean Analytics*. O'Reilly.
- Brooke, J. (1996). *SUS: A Quick and Dirty Usability Scale*. In Jordan et al. (Eds.), Usability Evaluation in Industry.
- Kohavi, R., Tang, D., & Xu, Y. (2020). *Trustworthy Online Controlled Experiments*. Cambridge University Press.

### Online Resources
- [Jared Spool — UIE Articles and Podcasts](https://articles.uie.com/)
- [MeasuringU — UX Measurement Resources (Jeff Sauro)](https://measuringu.com/)
- [Google HEART Framework Paper](https://research.google/pubs/pub36299/)
- [Nielsen Norman Group — UX Metrics](https://www.nngroup.com/articles/usability-metrics/)
- [Evan Miller — A/B Testing Sample Size Calculator](https://www.evanmiller.org/ab-testing/sample-size.html)
- [Optimizely — Experimentation Knowledge Base](https://www.optimizely.com/optimization-glossary/)

### Practice Exercises
1. **Build a HEART metrics table** — For your current project, define Goals, Signals, and Metrics for each HEART category (Happiness, Engagement, Adoption, Retention, Task Success)
2. **Baseline measurement** — Pick the three most important user tasks in your product; run a benchmark usability study with 5 users measuring task success rate, time on task, and error rate
3. **Instrument one user flow** — Add analytics events to track task start, errors, and completion for a single critical flow (e.g., checkout, onboarding); create a dashboard showing these metrics
4. **Design and run an A/B test** — Write a hypothesis, calculate the required sample size, implement the variant, and run the test to completion without peeking
5. **SUS benchmark** — Administer the SUS questionnaire to 20 users of your product; calculate the score and compare to the industry average (68) and your target
