# Refactoring

## Profile

### What It Is
Refactoring is the disciplined process of restructuring existing code without changing its external behavior. It improves the internal structure — readability, maintainability, and extensibility — while preserving functionality, verified by tests.

### Origin and Evolution
- Concept originated in Smalltalk community (1980s-90s)
- Formalized by Martin Fowler in *Refactoring* (1999, 2nd ed. 2018)
- Joshua Kerievsky extended with *Refactoring to Patterns* (2004)
- Modern: IDE-assisted refactoring, automated refactoring tools, AI-suggested refactorings
- Continuous refactoring is now standard practice in agile teams

### Key Authors and References
- **Martin Fowler** — *Refactoring: Improving the Design of Existing Code* (1999, 2018)
- **Joshua Kerievsky** — *Refactoring to Patterns* (2004)
- **Michael Feathers** — *Working Effectively with Legacy Code* (2004)
- **Kent Beck** — Refactoring as part of TDD and XP

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Code Smell | Indicator of a potential design problem |
| Extract Method | Pull a code block into a named function |
| Rename | Change name to better reveal intent |
| Move | Relocate code to where it belongs |
| Inline | Remove unnecessary indirection |
| Replace Conditional with Polymorphism | Strategy/State instead of switch |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Behavior preservation** — Refactoring must not change what the code does. Tests verify this. No tests = no safe refactoring.
2. **Small steps** — Each refactoring is a tiny, safe transformation. Commit after each step. If something breaks, revert just one step.
3. **Smell-driven** — Refactor in response to code smells (long method, feature envy, data clumps), not abstractly "to be clean."
4. **Opportunistic** — Refactor when you encounter code that needs it, not in separate "refactoring sprints." The Boy Scout Rule.
5. **Tests enable refactoring** — Without tests, refactoring is just changing code and hoping. Write tests first if they don't exist.

### When to Use vs. When to Avoid

**Refactor when:**
- Adding a feature requires understanding unclear code
- A code smell makes the code hard to change
- Tests are in place to verify behavior
- The code is actively maintained

**Avoid when:**
- No tests exist and adding them isn't feasible right now
- The code is being replaced soon
- Deadline pressure (schedule refactoring, don't skip it permanently)
- Refactoring would require changing public APIs without a migration plan

---

## Frameworks and Methodologies

### 1. Smell-Driven Refactoring — step-by-step
1. Identify a code smell (long method, duplicate code, feature envy)
2. Choose the appropriate refactoring from the catalog
3. Verify existing tests cover the affected code
4. Apply the refactoring in small, safe steps
5. Run tests after each step
6. Commit when green

### 2. Preparatory Refactoring — step-by-step
1. You need to add a feature but the code isn't structured for it
2. Before adding the feature, refactor to make the change easy
3. Apply refactorings: extract, move, rename to prepare the structure
4. Commit the refactoring separately from the feature
5. Now add the feature — it should be straightforward
6. Kent Beck: "Make the change easy, then make the easy change"

---

## How to Apply

### Decision Checklist
- [ ] Is there a code smell driving this refactoring?
- [ ] Do tests cover the code being refactored?
- [ ] Am I making small, reversible steps?
- [ ] Is the refactoring committed separately from feature changes?
- [ ] Does the code read better after refactoring?

### Implementation Patterns

**Common Code Smells and Refactorings:**
```
Smell                          → Refactoring
Long Method (>20 lines)        → Extract Method
Duplicate Code                 → Extract Method, Pull Up Method
Feature Envy                   → Move Method
Data Clumps                    → Extract Class, Introduce Parameter Object
Primitive Obsession            → Replace Primitive with Value Object
Switch Statements              → Replace Conditional with Polymorphism
Long Parameter List            → Introduce Parameter Object
Shotgun Surgery                → Move Method, Inline Class
```

**Extract Method Example:**
```python
# Before: long method with mixed abstraction levels
def generate_report(orders):
    # filter active orders
    active = [o for o in orders if o.status == "active"]
    # calculate totals
    total = sum(o.amount for o in active)
    tax = total * Decimal("0.20")
    # format output
    lines = [f"Order {o.id}: {o.amount}" for o in active]
    lines.append(f"Total: {total}")
    lines.append(f"Tax: {tax}")
    return "\n".join(lines)

# After: each step is a named function
def generate_report(orders):
    active = filter_active_orders(orders)
    total, tax = calculate_totals(active)
    return format_report(active, total, tax)

def filter_active_orders(orders):
    return [o for o in orders if o.status == "active"]

def calculate_totals(orders):
    total = sum(o.amount for o in orders)
    tax = total * Decimal("0.20")
    return total, tax

def format_report(orders, total, tax):
    lines = [f"Order {o.id}: {o.amount}" for o in orders]
    lines.append(f"Total: {total}")
    lines.append(f"Tax: {tax}")
    return "\n".join(lines)
```

### Common Mistakes
1. **Refactoring without tests** — Changing code with no safety net is not refactoring, it's gambling
2. **Big-bang refactoring** — Rewriting large sections at once instead of small steps
3. **Mixing refactoring with features** — Commit refactorings separately from behavior changes
4. **Refactoring for its own sake** — Without a smell or a need, refactoring is waste
5. **Not using IDE refactoring tools** — Manual rename/extract is error-prone; use automated tools

### Integration with Other Topics
- **Clean Code** — Refactoring is how you achieve and maintain clean code
- **TDD** — The refactor step in Red-Green-Refactor
- **Technical Debt** — Refactoring pays down technical debt incrementally
- **Code Review** — Reviews identify refactoring opportunities
- **Design Patterns** — Refactoring to patterns when complexity warrants
- **SOLID Principles** — Refactoring often improves SOLID compliance

---

## Resources

### Essential Reading
- *Refactoring: Improving the Design of Existing Code* (2nd ed.) — Martin Fowler
- *Refactoring to Patterns* — Joshua Kerievsky
- *Working Effectively with Legacy Code* — Michael Feathers

### Online Resources
- refactoring.guru — Catalog of refactorings and smells
- Refactoring catalog (refactoring.com)
- Sourcery.ai — Automated Python refactoring suggestions

### Practice Exercises
1. Identify 5 code smells in a legacy codebase and refactor each
2. Practice Extract Method on a 50+ line function
3. Replace a switch statement with Strategy pattern
4. Refactor a function with 6 parameters into a Parameter Object
5. Do a preparatory refactoring before adding a new feature
