# Site Reliability Engineering

## Profile

### What It Is
Site Reliability Engineering (SRE) is a discipline that applies software engineering practices to operations problems. Originated at Google, SRE defines reliability as the most important feature and uses error budgets, SLOs, and automation to balance reliability with feature velocity.

### Origin and Evolution
- Created at Google by Ben Treynor Sloss (~2003)
- Formalized in *Site Reliability Engineering* book (2016)
- SRE as a practice spread beyond Google to industry standard
- Modern: platform engineering evolved from SRE, SRE as embedded practice in dev teams

### Key Authors and References
- **Ben Treynor Sloss** — "SRE is what happens when you ask a software engineer to design an operations function"
- **Betsy Beyer et al.** — *Site Reliability Engineering* (2016), *The Site Reliability Workbook* (2018)
- **Niall Murphy** — SRE leadership
- **Google** — SRE books and practices

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| SLI | Service Level Indicator — measurable metric (latency, error rate) |
| SLO | Service Level Objective — target for an SLI (99.9% availability) |
| SLA | Service Level Agreement — contractual SLO with consequences |
| Error Budget | Allowed unreliability (100% - SLO). Spend it on features or save for reliability |
| Toil | Manual, repetitive operational work that should be automated |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Reliability is the most important feature** — If users can't reach your product, no other feature matters.
2. **Error budgets balance velocity and reliability** — 99.9% SLO means 0.1% error budget. If the budget is healthy, ship fast. If it's spent, slow down and fix reliability.
3. **Eliminate toil** — Toil is manual, repetitive, automatable work. SREs should spend <50% time on toil, rest on engineering.
4. **Measure everything** — SLIs, SLOs, error budgets, toil percentage, incident metrics. You can't improve what you don't measure.
5. **Blame-free postmortems** — Incidents happen. Learn from them without blaming individuals. Fix systems, not people.

### When to Use vs. When to Avoid

**Adopt SRE practices when:**
- Service reliability is critical to business success
- The team is large enough to dedicate effort to reliability
- There's a tension between feature velocity and operational stability
- Incidents are frequent or impactful

**Start simple when:**
- Small team with few services
- Low traffic / low criticality
- Begin with SLOs and error budgets; add more as needed

---

## Frameworks and Methodologies

### 1. SLO Implementation — step-by-step
1. Identify critical user journeys
2. Define SLIs for each journey (latency, availability, correctness)
3. Set SLOs based on user expectations and business needs (start conservative)
4. Calculate error budget: (1 - SLO) × time period
5. Track error budget burn rate
6. Establish policies: what happens when budget is exhausted?

### 2. Toil Reduction — step-by-step
1. Catalog all operational tasks
2. Classify each as toil or engineering
3. Measure time spent on toil (target: <50%)
4. Prioritize automation for the most time-consuming toil
5. Automate and validate
6. Repeat — toil is a continuous battle

---

## How to Apply

### Decision Checklist
- [ ] Are SLOs defined and measured for critical services?
- [ ] Is the error budget tracked and used for decision-making?
- [ ] Is toil measured and a reduction plan in place?
- [ ] Are postmortems conducted blame-free after incidents?
- [ ] Is on-call structured with proper escalation and compensation?

### Implementation Patterns

**SLO Definition:**
```yaml
service: order-service
slos:
  - name: availability
    sli: "successful_requests / total_requests"
    target: 99.9%  # 43.2 min downtime/month budget
    window: 30d

  - name: latency
    sli: "requests_under_200ms / total_requests"
    target: 99.0%  # 1% of requests can exceed 200ms
    window: 30d

error_budget_policy:
  budget_remaining > 50%: "Ship features freely"
  budget_remaining 25-50%: "Increase testing, slow deployment pace"
  budget_remaining < 25%: "Freeze features, focus on reliability"
  budget_exhausted: "Stop all feature work until reliability improves"
```

**Postmortem Template:**
```markdown
## Incident: [Title]
**Date**: YYYY-MM-DD
**Duration**: X hours
**Impact**: Y% of users affected, Z errors

## Timeline
- HH:MM — First alert fired
- HH:MM — Engineer paged
- HH:MM — Root cause identified
- HH:MM — Mitigation applied
- HH:MM — Full recovery

## Root Cause
[Blameless technical explanation]

## Contributing Factors
1. [Factor 1]
2. [Factor 2]

## Action Items
| Action | Owner | Priority | Deadline |
|--------|-------|----------|----------|
| Add alerting for X | @eng1 | P1 | 1 week |
| Automate recovery for Y | @eng2 | P2 | 2 weeks |

## Lessons Learned
- What went well
- What could be improved
```

### Common Mistakes
1. **SLOs without error budgets** — SLOs without error budget policies are just aspirational targets
2. **100% SLO** — No system is 100% reliable; an SLO of 100% means infinite reliability investment
3. **Alert on everything** — SRE alerts on SLO burn, not every metric spike
4. **Blame culture** — Blaming individuals in postmortems prevents honest analysis and learning
5. **SRE as ops** — SRE should be engineering-focused, not a renamed operations team

### Integration with Other Topics
- **Observability** — SLIs are measured through observability instrumentation
- **CI/CD** — Deployment velocity is balanced against error budget
- **Incident Management** — SRE defines incident response processes
- **Deployment Strategies** — Canary/progressive delivery reduces blast radius
- **Cloud-Native** — SRE practices apply to cloud-native architectures
- **Team Topologies** — SRE as enabling team or embedded practice

---

## Resources

### Essential Reading
- *Site Reliability Engineering* — Beyer, Jones, Petoff, Murphy (Google)
- *The Site Reliability Workbook* — Beyer et al.
- *Implementing Service Level Objectives* — Alex Hidalgo

### Online Resources
- sre.google — Google SRE resources
- SLO Generator tools (Google, Datadog, Nobl9)
- PagerDuty incident response documentation

### Practice Exercises
1. Define SLIs and SLOs for a web application
2. Calculate error budget and set up burn rate alerting
3. Conduct a blameless postmortem for a past incident
4. Catalog and measure toil for a team over one week
5. Design an on-call rotation with proper escalation policies
