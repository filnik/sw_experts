# Test-Driven Development

## Profile

### What It Is
Test-Driven Development (TDD) is a software development practice where tests are written before the production code. The cycle — Red (write a failing test), Green (write minimal code to pass), Refactor (improve the code) — drives design decisions and ensures every line of code is verified by a test.

### Origin and Evolution
- Originated from Extreme Programming (Kent Beck, 1999)
- Formalized by Kent Beck in *Test-Driven Development: By Example* (2002)
- Extended to BDD (Behavior-Driven Development) by Dan North (2006)
- ATDD (Acceptance Test-Driven Development) for requirements validation
- Current: TDD in functional programming, property-based TDD, AI-assisted testing

### Key Authors and References
- **Kent Beck** — *Test-Driven Development: By Example* (2002)
- **Robert C. Martin** — *Clean Code* (TDD chapters)
- **Michael Feathers** — *Working Effectively with Legacy Code* (testing legacy)
- **Dan North** — BDD (Behavior-Driven Development)

### Key Concepts at a Glance
| Phase | Action | Purpose |
|-------|--------|---------|
| Red | Write a failing test | Define expected behavior |
| Green | Write minimal code to pass | Implement behavior |
| Refactor | Improve code structure | Maintain quality |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Test first, code second** — Writing tests first forces you to think about the API and behavior before implementation.
2. **Small steps** — Write one failing test, make it pass, refactor. Don't write 10 tests then 10 implementations.
3. **Only write code to pass a failing test** — No speculative code. Every line exists because a test requires it.
4. **Refactoring is non-negotiable** — The Green phase gets it working; the Refactor phase keeps it clean. Skip neither.
5. **Tests are design tools** — Hard-to-test code indicates design problems. TDD provides immediate design feedback.

### When to Use vs. When to Avoid

**Use when:**
- Building business logic with clear rules
- Developing APIs and libraries
- Working on complex algorithms
- Codebase health and long-term maintenance matter
- Learning a new domain or technology

**Be pragmatic about:**
- UI code (test behavior, not rendering details)
- Prototyping/exploration (test after when the design stabilizes)
- Infrastructure code (integration tests may be more valuable than unit TDD)
- Glue code with no logic

---

## Frameworks and Methodologies

### 1. Classic TDD (Red-Green-Refactor) — step-by-step
1. Write a test that describes the expected behavior (Red — it fails)
2. Write the simplest code that makes the test pass (Green)
3. Refactor: remove duplication, improve names, extract methods
4. Run all tests to verify nothing is broken
5. Pick the next behavior and repeat
6. Commit after each cycle

### 2. Outside-In TDD (London School) — step-by-step
1. Start with an acceptance test at the outer boundary (API, controller)
2. Write a unit test for the first collaborator
3. Implement with mocks for unimplemented dependencies
4. Move inward: implement each collaborator with its own TDD cycle
5. Replace mocks with real implementations as they're built
6. The acceptance test passes when all pieces are connected

---

## How to Apply

### Decision Checklist
- [ ] Am I writing the test before the code?
- [ ] Is each test testing one behavior?
- [ ] Does the test have a clear Arrange-Act-Assert structure?
- [ ] Am I refactoring after each green phase?
- [ ] Are tests fast enough to run continuously?

### Implementation Patterns

**TDD Cycle Example:**
```python
# RED: Write failing test
def test_calculate_discount_for_premium_customer():
    customer = Customer(tier="premium")
    order = Order(items=[Item(price=100)])
    discount = calculate_discount(customer, order)
    assert discount == Decimal("10.00")  # 10% for premium

# GREEN: Minimal implementation
def calculate_discount(customer: Customer, order: Order) -> Decimal:
    if customer.tier == "premium":
        return order.total * Decimal("0.10")
    return Decimal("0")

# RED: Next test
def test_no_discount_for_standard_customer():
    customer = Customer(tier="standard")
    order = Order(items=[Item(price=100)])
    discount = calculate_discount(customer, order)
    assert discount == Decimal("0")

# GREEN: Already passes! Move to next behavior.

# RED: Discount cap
def test_discount_capped_at_50():
    customer = Customer(tier="premium")
    order = Order(items=[Item(price=1000)])
    discount = calculate_discount(customer, order)
    assert discount == Decimal("50.00")

# GREEN: Add cap
def calculate_discount(customer: Customer, order: Order) -> Decimal:
    if customer.tier == "premium":
        return min(order.total * Decimal("0.10"), Decimal("50"))
    return Decimal("0")

# REFACTOR: Extract constants
MAX_DISCOUNT = Decimal("50")
PREMIUM_RATE = Decimal("0.10")
```

### Common Mistakes
1. **Writing too many tests before any code** — TDD is one test at a time
2. **Testing implementation, not behavior** — Tests that break on refactoring are testing the wrong thing
3. **Skipping the refactor step** — Code works but accumulates mess
4. **Slow tests** — TDD requires fast feedback; keep unit tests under 1 second total
5. **TDD dogma** — Not every line of code needs TDD; use judgment for the best testing strategy

### Integration with Other Topics
- **Clean Code** — TDD naturally produces cleaner code through design pressure
- **Refactoring** — Refactoring is built into the TDD cycle
- **SOLID Principles** — TDD pushes toward SOLID design (hard-to-test = SOLID violation)
- **Continuous Testing** — TDD creates the test suite that enables continuous testing
- **Code Quality Tools** — TDD tests feed code coverage metrics
- **Pair Programming** — Ping-pong pairing (one writes test, other implements)

---

## Resources

### Essential Reading
- *Test-Driven Development: By Example* — Kent Beck
- *Growing Object-Oriented Software, Guided by Tests* — Freeman & Pryce
- *Working Effectively with Legacy Code* — Michael Feathers

### Online Resources
- Kent Beck's TDD talks and articles
- Uncle Bob's "TDD Harms Architecture" rebuttal
- Exercism.org — TDD practice exercises

### Practice Exercises
1. Implement a string calculator using TDD (classic kata)
2. Build a bowling game scorer using strict Red-Green-Refactor
3. Use outside-in TDD to build a simple REST endpoint
4. Practice the Roman numerals kata in TDD style
5. Add tests to a legacy function before refactoring it
