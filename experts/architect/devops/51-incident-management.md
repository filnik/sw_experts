# Incident Management and On-Call

## Profile

### What It Is
Incident Management is the structured process for detecting, responding to, resolving, and learning from production incidents. On-Call is the practice of having designated engineers available to respond to incidents outside business hours, with defined escalation paths and rotation schedules.

### Origin and Evolution
- ITIL incident management framework (1980s-90s)
- PagerDuty (2009) popularized modern on-call tooling
- Google SRE formalized incident response and blameless postmortems
- Incident.io, FireHydrant — modern incident management platforms
- Current: automated incident response, AIOps, incident learning culture

### Key Authors and References
- **Google SRE Team** — Incident response chapters in SRE books
- **PagerDuty** — Incident Response documentation
- **John Allspaw** — Resilience engineering and learning from incidents
- **Nora Jones** — Jeli.io, Learning from Incidents community

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Incident Commander | Single person coordinating the response |
| Severity Levels | Classification: SEV1 (critical) through SEV4 (minor) |
| Escalation | Defined path for involving additional responders |
| Blameless Postmortem | Analysis focused on systems, not individuals |
| On-Call Rotation | Scheduled availability for incident response |
| Runbook | Step-by-step guide for handling known issues |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Fast detection, fast mitigation** — Detect incidents through monitoring and alerting. Mitigate impact first, root cause later.
2. **Clear roles and communication** — Incident commander coordinates. Responders work. Communication happens in a single channel.
3. **Blameless learning** — Incidents are opportunities to learn and improve systems. Blame prevents learning.
4. **Sustainable on-call** — On-call should be compensated, rotated fairly, and not overwhelmingly noisy. Burnout helps nobody.
5. **Runbooks for known issues** — Document remediation steps for recurring issues. Automate where possible.

### When to Use vs. When to Avoid
**Incident management scales with system criticality.** Even small teams benefit from basic incident response structure.

---

## Frameworks and Methodologies

### 1. Incident Response Process — step-by-step
1. **Detect**: Alert fires or user reports issue
2. **Triage**: Assess severity (SEV1-4), assign incident commander
3. **Communicate**: Open incident channel, notify stakeholders
4. **Mitigate**: Apply immediate fix (rollback, failover, scaling)
5. **Resolve**: Full fix deployed and verified
6. **Postmortem**: Within 48 hours, blameless analysis
7. **Follow-up**: Track and complete action items

### 2. On-Call Design — step-by-step
1. Define on-call rotation (weekly, with primary and secondary)
2. Set alert routing: pages go to on-call, not everyone
3. Define escalation policy (primary → secondary → lead → all-hands)
4. Provide runbooks for common alerts
5. Review on-call load: if too noisy, fix the system, not the person
6. Compensate on-call (time off or financial compensation)

---

## How to Apply

### Decision Checklist
- [ ] Are severity levels defined with clear criteria?
- [ ] Is there a designated incident commander for each incident?
- [ ] Is communication centralized in one channel?
- [ ] Are postmortems conducted within 48 hours?
- [ ] Is on-call rotation fair and sustainable?

### Implementation Patterns

**Severity Levels:**
```
SEV1 (Critical): Service down for all users. Revenue loss. Data loss.
  Response: Immediate page. All-hands if needed. Customer communication.
  Target: Mitigate in 30 min.

SEV2 (Major): Significant degradation. Some users affected.
  Response: Page on-call. Escalate if not mitigated in 1 hour.
  Target: Mitigate in 1 hour.

SEV3 (Minor): Partial degradation. Workaround available.
  Response: Alert during business hours. Fix within 1 business day.

SEV4 (Low): Cosmetic or minor issue. No user impact.
  Response: Ticket in backlog. Fix in next sprint.
```

**Postmortem Action Items:**
```markdown
## Action Items (Prioritized)

| # | Action | Type | Owner | Priority | Status |
|---|--------|------|-------|----------|--------|
| 1 | Add alert for database connection pool exhaustion | Detect | @sre | P1 | Open |
| 2 | Implement automatic connection pool scaling | Mitigate | @backend | P1 | Open |
| 3 | Add runbook for connection pool issues | Process | @sre | P2 | Open |
| 4 | Improve staging load tests to catch pool limits | Prevent | @qa | P2 | Open |
```

### Common Mistakes
1. **No incident commander** — Everyone trying to fix simultaneously without coordination
2. **Blame in postmortems** — "Bob should have checked" prevents systemic improvements
3. **Alert fatigue** — Too many alerts; on-call ignores them. Fix signal-to-noise ratio
4. **No follow-through** — Postmortem action items never completed
5. **Hero culture** — Relying on one person to fix everything; unsustainable

### Integration with Other Topics
- **SRE** — Incident management is core SRE practice
- **Observability** — Observability data drives incident detection and diagnosis
- **Deployment Strategies** — Quick rollback is the fastest mitigation
- **Resilience Patterns** — Patterns reduce incident frequency and impact
- **Team Topologies** — On-call ownership aligns with team structure
- **Technical Leadership** — Leaders set blameless culture

---

## Resources

### Essential Reading
- *Site Reliability Engineering* — Google (incident management chapters)
- PagerDuty Incident Response documentation (response.pagerduty.com)
- *Incident Management for Operations* — Rob Schnepp

### Online Resources
- Jeli.io — Learning from Incidents community
- FireHydrant incident management blog
- Atlassian Incident Management handbook

### Practice Exercises
1. Define severity levels and response procedures for your team
2. Run a tabletop exercise simulating a SEV1 incident
3. Conduct a blameless postmortem for a recent incident
4. Set up an on-call rotation with escalation policies
5. Create runbooks for the 5 most common alerts
