# Code Review Practices

## Profile

### What It Is
Code review is the systematic examination of source code by peers to identify bugs, improve quality, share knowledge, and enforce standards. It is one of the most effective quality assurance practices, catching defects early and spreading architectural understanding across the team.

### Origin and Evolution
- Formal inspections (Fagan, 1976) — structured, heavyweight process
- Pair programming (XP, 1999) — real-time continuous review
- Tool-assisted reviews (Gerrit, GitHub PRs, GitLab MRs) — asynchronous, lightweight
- Modern: automated code review (linters, SAST), AI-assisted review, stacked PRs
- Current best practice: small PRs, fast feedback, automated checks for style/formatting

### Key Authors and References
- **Michael Fagan** — Formal code inspections (1976)
- **Karl Wiegers** — *Peer Reviews in Software* (2001)
- **Google** — Engineering Practices documentation on code review
- **SmartBear** — *Best Kept Secrets of Peer Code Review*

### Key Concepts at a Glance
| Aspect | Best Practice |
|--------|--------------|
| PR Size | Small (< 400 lines changed) |
| Review Time | Respond within 1 business day |
| Automation | Lint, format, test in CI before review |
| Focus | Logic, design, security > style (automate style) |
| Tone | Constructive, specific, suggest alternatives |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Review for understanding, not just correctness** — The reviewer should understand the change well enough to maintain it. If you can't understand it, the code needs simplification.
2. **Small PRs get better reviews** — Large PRs get rubber-stamped. Keep PRs focused on one concern, under 400 lines. Stack PRs for large features.
3. **Automate what can be automated** — Don't argue about formatting, imports, or style in reviews. Automate with formatters, linters, and CI checks.
4. **Be kind, be specific** — "This is wrong" is unhelpful. "Consider using X because Y" is constructive. Assume good intent.
5. **Speed matters** — Slow reviews block teammates and encourage larger batches. Review within hours, not days.

### When to Use vs. When to Avoid

**Always review:**
- Production code changes
- Infrastructure and configuration changes
- Security-sensitive changes (double review)

**Lighter review for:**
- Documentation-only changes
- Automated dependency updates (with passing CI)
- Trivial fixes (typos, formatting)

---

## Frameworks and Methodologies

### 1. Effective Code Review Process — step-by-step
1. Author: create small, focused PR with clear description
2. CI: automated checks run (lint, format, test, build)
3. Reviewer: read the PR description and context first
4. Reviewer: review for design, logic, security, readability
5. Reviewer: leave specific, constructive comments
6. Author: address feedback, discuss disagreements
7. Reviewer: approve when satisfactory; merge

### 2. Review Checklist — step-by-step
1. **Design**: Does the change fit the architecture? Is there a simpler approach?
2. **Logic**: Are edge cases handled? Are assumptions documented?
3. **Security**: Input validation? Authentication checks? No secrets in code?
4. **Testing**: Are new behaviors tested? Are edge cases covered?
5. **Readability**: Can I understand this without explanation? Clear names?
6. **Performance**: Any obvious performance issues at scale?

---

## How to Apply

### Decision Checklist
- [ ] Is the PR small and focused (< 400 lines)?
- [ ] Does the PR description explain what and why?
- [ ] Are CI checks passing before review?
- [ ] Has the reviewer checked design, not just syntax?
- [ ] Is feedback specific and constructive?

### Implementation Patterns

**PR Description Template:**
```markdown
## What
Brief description of the change.

## Why
Problem being solved or feature being added.

## How
Approach taken and key design decisions.

## Testing
How this was tested. New tests added?

## Screenshots (if UI change)
Before/after if applicable.
```

**Constructive Feedback Examples:**
```
Bad:  "This is wrong."
Good: "This query runs N+1 times. Consider using a JOIN or prefetch."

Bad:  "Don't do this."
Good: "Using `eval()` here creates an injection risk. Consider `ast.literal_eval()`
       or a whitelist approach."

Bad:  "Rename this."
Good: "Consider renaming `data` to `order_summary` to clarify what it contains."

Nit (optional): "Nit: this could be a list comprehension, but fine as-is."
```

### Common Mistakes
1. **Reviewing after CI fails** — Don't review code that doesn't build or pass tests
2. **Giant PRs** — 2000-line PRs get superficial reviews; split them
3. **Gatekeeping** — Using review to enforce personal preferences, not team standards
4. **No review of tests** — Tests are code too; review them for correctness and coverage
5. **Review as bottleneck** — One reviewer for the entire team; distribute review responsibility

### Integration with Other Topics
- **Clean Code** — Reviews enforce and teach clean code practices
- **Technical Debt** — Reviews prevent new debt and identify existing debt
- **Pair Programming** — Real-time alternative or complement to async review
- **CI/CD Pipelines** — Automated checks run before human review
- **Code Quality Tools** — Linters and formatters reduce review burden
- **Team Topologies** — Review practices reflect team structure and ownership

---

## Resources

### Essential Reading
- Google Engineering Practices: Code Review guide
- *Peer Reviews in Software* — Karl Wiegers
- *Best Kept Secrets of Peer Code Review* — SmartBear

### Online Resources
- Google's "How to do a code review" documentation
- Conventional Comments (conventionalcomments.org)
- GitHub PR review features documentation

### Practice Exercises
1. Review 3 PRs focusing on design, not just style
2. Split a large PR into 3 stacked PRs
3. Write a PR description that fully explains context without the code
4. Practice giving constructive feedback on code you disagree with
5. Set up automated formatting/linting to eliminate style debates
