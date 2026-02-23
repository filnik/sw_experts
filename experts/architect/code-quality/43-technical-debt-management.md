# Technical Debt Management

## Profile

### What It Is
Technical Debt is the implied cost of future rework caused by choosing an expedient solution now instead of a better approach that would take longer. Managing technical debt involves identifying, quantifying, prioritizing, and systematically paying it down while preventing new debt from accumulating unchecked.

### Origin and Evolution
- Metaphor coined by Ward Cunningham (1992) — comparing design shortcuts to financial debt
- Extended by Martin Fowler's technical debt quadrant (deliberate/inadvertent × prudent/reckless)
- Integration with agile practices: debt as backlog items, debt sprints
- Modern: automated debt detection (SonarQube, CodeClimate), debt-aware architecture decisions
- Current: shift from "debt-free" goal to "sustainable debt" management

### Key Authors and References
- **Ward Cunningham** — Technical debt metaphor (1992)
- **Martin Fowler** — Technical Debt Quadrant
- **Adam Tornhill** — *Software Design X-Rays*, *Your Code as a Crime Scene*
- **Michael Feathers** — *Working Effectively with Legacy Code*

### Key Concepts at a Glance
| Type | Description | Example |
|------|-------------|---------|
| Deliberate-Prudent | Conscious shortcut with plan to fix | "Ship now, refactor next sprint" |
| Deliberate-Reckless | Conscious shortcut with no plan | "We don't have time for design" |
| Inadvertent-Prudent | Learned better after delivery | "Now we know how it should work" |
| Inadvertent-Reckless | Didn't know better | "What's a design pattern?" |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Debt is not inherently bad** — Like financial debt, technical debt can be strategic. The problem is unmanaged, invisible debt.
2. **Make debt visible** — Track debt explicitly in the backlog. Invisible debt can't be prioritized or managed.
3. **Pay interest or pay principal** — Every time you work around debt instead of fixing it, you're paying interest. Eventually, paying down the principal is cheaper.
4. **Prevention over cure** — Code reviews, automated checks, and architectural guardrails prevent reckless debt accumulation.
5. **Continuous paydown** — Allocate a percentage of each sprint (15-20%) to debt reduction. Don't batch it into "debt sprints" that never happen.

### When to Use vs. When to Avoid

**Take on deliberate debt when:**
- Time-to-market is critical and the debt is well-understood
- There's a concrete plan to pay it down
- The affected area is isolated and won't spread

**Prioritize paying down debt when:**
- A high-change area is slowed by debt (hotspot analysis)
- Debt blocks feature development
- Debt creates security or reliability risks
- Team velocity is declining

---

## Frameworks and Methodologies

### 1. Debt Identification and Prioritization — step-by-step
1. Collect debt from: code smells, TODO/HACK comments, team surveys, incident retros
2. Categorize: code debt, design debt, test debt, infrastructure debt, documentation debt
3. Assess impact: how much does this slow us down? What's the risk?
4. Identify hotspots: files with most changes + most complexity (Adam Tornhill's approach)
5. Prioritize: high-impact debt in high-change areas first
6. Create actionable backlog items with clear acceptance criteria

### 2. Continuous Debt Reduction — step-by-step
1. Reserve 15-20% of sprint capacity for debt paydown
2. Apply the Boy Scout Rule: improve code you touch for features
3. Include debt items in sprint planning alongside features
4. Track debt metrics over time (code quality scores, complexity trends)
5. Celebrate debt reduction as team achievements
6. Prevent new debt: enforce standards in code review and CI

---

## How to Apply

### Decision Checklist
- [ ] Is technical debt tracked in the backlog (not just in TODO comments)?
- [ ] Are high-change, high-complexity hotspots identified?
- [ ] Is there dedicated capacity for debt reduction each sprint?
- [ ] Are code quality metrics tracked over time?
- [ ] Do code reviews prevent new reckless debt?

### Implementation Patterns

**Hotspot Analysis:**
```bash
# Identify files that change most often (high churn)
git log --since="6 months ago" --name-only --pretty=format: | \
  sort | uniq -c | sort -rn | head -20

# Combine with complexity (high churn + high complexity = hotspot)
# Use tools like CodeScene, SonarQube, or custom scripts
```

**Debt Backlog Item Template:**
```markdown
## Technical Debt: [Title]

**Type**: Code / Design / Test / Infrastructure
**Location**: `src/services/order_service.py`
**Impact**: Slows feature development in orders module by ~30%
**Risk**: Medium (potential for bugs during order changes)

**Current State**: OrderService is 500 lines with 12 dependencies,
mixing validation, persistence, and notification logic.

**Target State**: Split into OrderValidator, OrderRepository,
OrderNotifier with clear interfaces.

**Payoff**: Faster feature delivery, easier testing, reduced bug rate.

**Effort**: Medium (2-3 days of focused refactoring)
```

### Common Mistakes
1. **Invisible debt** — Debt exists only in developers' heads, not in the backlog
2. **"We'll fix it later" → never** — Without a plan and timeline, debt is permanent
3. **Debt sprints** — Dedicating entire sprints to debt rarely gets approved; use continuous allocation
4. **Measuring debt by LOC or coverage alone** — These metrics miss design debt and complexity
5. **Paying down low-impact debt** — Fixing clean code in rarely-changed files instead of hotspots

### Integration with Other Topics
- **Refactoring** — The primary mechanism for paying down code debt
- **Clean Code** — Clean code practices prevent new debt
- **Code Review** — Reviews are the gatekeeping mechanism against new debt
- **Architecture Decision Records** — Document deliberate debt decisions
- **Evolutionary Architecture** — Architecture fitness functions detect architectural debt
- **Agile** — Debt management is integral to sustainable agile delivery

---

## Resources

### Essential Reading
- *Your Code as a Crime Scene* — Adam Tornhill
- *Software Design X-Rays* — Adam Tornhill
- *Managing Technical Debt* — Philippe Kruchten et al.

### Online Resources
- Martin Fowler's Technical Debt articles
- CodeScene (codescene.com) — Behavioral code analysis
- SonarQube documentation on code quality metrics

### Practice Exercises
1. Conduct a hotspot analysis on a codebase using git log
2. Create 5 debt backlog items with impact and effort estimates
3. Track code quality metrics over 3 sprints
4. Allocate 20% of sprint capacity to debt and measure velocity impact
5. Run a team survey to identify the most painful debt areas
