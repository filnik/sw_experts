# Clean Code

## Profile

### What It Is
Clean Code is the practice of writing software that is readable, understandable, and maintainable. It emphasizes clarity of intent, simplicity, and craftsmanship — code that communicates its purpose to other developers (including future you) without requiring extensive comments or documentation.

### Origin and Evolution
- Rooted in structured programming (Dijkstra) and software craftsmanship movement
- Popularized by Robert C. Martin's *Clean Code* (2008)
- Influenced by Kent Beck's *Implementation Patterns* and Martin Fowler's *Refactoring*
- Modern evolution: language-specific idioms, automated formatting, AI-assisted code review
- Debate: some principles are universal, others are context-dependent

### Key Authors and References
- **Robert C. Martin** — *Clean Code* (2008), *The Clean Coder* (2011)
- **Martin Fowler** — *Refactoring* (1999, 2nd ed. 2018)
- **Kent Beck** — *Implementation Patterns*, *Smalltalk Best Practice Patterns*
- **Grady Booch** — "Clean code reads like well-written prose"

### Key Concepts at a Glance
| Principle | Description |
|-----------|-------------|
| Meaningful Names | Names reveal intent, not implementation |
| Small Functions | Functions do one thing, at one level of abstraction |
| DRY | Don't Repeat Yourself — but only for true duplication |
| KISS | Keep It Simple — simplest solution that works |
| Boy Scout Rule | Leave the code cleaner than you found it |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Code is read far more than it's written** — Optimize for readability. Every minute spent making code clearer saves hours of future comprehension.
2. **Names matter** — A well-chosen name eliminates the need for a comment. Names should reveal intent, context, and meaning.
3. **Small, focused functions** — Each function should do one thing, do it well, and do it only. Functions should operate at a single level of abstraction.
4. **Minimize surprise** — Code should do what a reader expects. Follow conventions, be consistent, and avoid clever tricks.
5. **Leave it better** — The Boy Scout Rule: whenever you touch code, improve it slightly. Small, continuous improvements compound over time.

### When to Use vs. When to Avoid

**Always apply:**
- Meaningful naming, consistent formatting, small functions
- In production code that will be maintained

**Be pragmatic about:**
- Prototypes and throwaway code (clean enough, not perfect)
- Performance-critical code where clarity must sometimes yield to optimization
- Legacy code (improve incrementally, don't rewrite)

---

## Frameworks and Methodologies

### 1. Function Cleanup — step-by-step
1. Identify functions longer than 20 lines
2. Name the "what" at each step (extract method with descriptive name)
3. Ensure each function operates at one level of abstraction
4. Reduce parameters (max 3; use objects for more)
5. Eliminate side effects or make them explicit
6. Replace comments with self-documenting code

### 2. Naming Improvement — step-by-step
1. Replace single-letter variables (except loop counters in small scopes)
2. Use verbs for functions, nouns for classes/variables
3. Be specific: `getUserById` not `getData`, `isEligibleForDiscount` not `check`
4. Use domain language (ubiquitous language from DDD)
5. Avoid encodings: no Hungarian notation, no type prefixes
6. Ensure names are pronounceable and searchable

---

## How to Apply

### Decision Checklist
- [ ] Can I understand this function without reading its implementation?
- [ ] Are names meaningful and intention-revealing?
- [ ] Is each function doing exactly one thing?
- [ ] Are there any comments that could be replaced by better naming?
- [ ] Is the code consistent in style and conventions?

### Implementation Patterns

**Before and After — Naming:**
```python
# Bad: What does this do?
def calc(lst, t):
    r = []
    for x in lst:
        if x.a > t:
            r.append(x)
    return r

# Good: Names reveal intent
def find_overdue_invoices(invoices: list[Invoice], threshold_days: int) -> list[Invoice]:
    return [inv for inv in invoices if inv.days_overdue > threshold_days]
```

**Before and After — Function Extraction:**
```python
# Bad: Function doing too many things
def process_order(order_data):
    # validate
    if not order_data.get("items"):
        raise ValueError("No items")
    if not order_data.get("customer_id"):
        raise ValueError("No customer")
    # calculate total
    total = 0
    for item in order_data["items"]:
        price = get_price(item["product_id"])
        total += price * item["quantity"]
    # apply discount
    if total > 100:
        total *= 0.9
    # save
    order = Order(customer_id=order_data["customer_id"], total=total)
    db.save(order)
    # notify
    send_email(order_data["customer_id"], f"Order confirmed: {total}")
    return order

# Good: Each function does one thing
def process_order(order_data: dict) -> Order:
    validate_order_data(order_data)
    total = calculate_order_total(order_data["items"])
    total = apply_discount(total)
    order = save_order(order_data["customer_id"], total)
    notify_customer(order)
    return order
```

### Common Mistakes
1. **Comments as deodorant** — Writing comments to explain bad code instead of rewriting the code
2. **Premature abstraction** — Creating helpers and utilities before duplication exists ("rule of three")
3. **Inconsistent style** — Mixing naming conventions, formatting styles within the same codebase
4. **Over-engineering for cleanliness** — 15 classes for a 30-line feature in the name of "clean code"
5. **Ignoring language idioms** — Writing Java-style code in Python; each language has its own "clean"

### Integration with Other Topics
- **Refactoring** — Clean code is achieved through continuous refactoring
- **Test-Driven Development** — TDD naturally produces clean, well-structured code
- **Code Review** — Code review enforces and teaches clean code practices
- **SOLID Principles** — SOLID at the class/module level, clean code at the statement/function level
- **Technical Debt** — Unclean code is the primary source of technical debt
- **Pair Programming** — Real-time code quality improvement through collaboration

---

## Resources

### Essential Reading
- *Clean Code* — Robert C. Martin
- *Refactoring* (2nd ed.) — Martin Fowler
- *A Philosophy of Software Design* — John Ousterhout

### Online Resources
- Clean Code blog (blog.cleancoder.com)
- Refactoring Guru (refactoring.guru)
- Google Engineering Practices documentation

### Practice Exercises
1. Refactor a 100-line function into clean, small functions
2. Rename 10 poorly named variables/functions in a codebase
3. Remove all comments that restate what the code does
4. Apply the Boy Scout Rule to 5 files you touch this week
5. Compare the same logic before and after clean code refactoring
