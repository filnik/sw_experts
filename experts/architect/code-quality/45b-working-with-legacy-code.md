# Working Effectively with Legacy Code

## Profile

### What It Is
Working with Legacy Code is the discipline of safely modifying, testing, and improving codebases that lack adequate test coverage, documentation, or clear structure. It provides techniques for getting untested code under test, breaking dependencies, and making changes with confidence in systems you didn't build and don't fully understand.

### Origin and Evolution
- Michael Feathers defined legacy code as "code without tests" (*Working Effectively with Legacy Code*, 2004)
- Built on refactoring principles (Fowler) and TDD practices (Beck)
- Techniques developed for real-world constraints: no time for rewrite, no documentation, no original developers
- Modern: AI-assisted code understanding, automated test generation, strangler fig migration
- Current: legacy code is the norm, not the exception — most developers work with it daily

### Key Authors and References
- **Michael Feathers** — *Working Effectively with Legacy Code* (2004)
- **Martin Fowler** — *Refactoring* (complementary techniques)
- **Kent Beck** — TDD as the safety net for legacy changes
- **Adam Tornhill** — *Your Code as a Crime Scene* (behavioral analysis of legacy code)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Legacy Code | Code without tests (Feathers' definition) |
| Seam | A place where behavior can be changed without editing the code |
| Characterization Test | Test that captures existing behavior (not desired behavior) |
| Sprout Method/Class | New functionality added in a new method/class, tested separately |
| Wrap Method/Class | Wrapping existing code to add behavior before/after |
| Scratch Refactoring | Refactor to understand, then throw away and start clean |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Legacy code = code without tests** — The defining characteristic of legacy code is not age or technology but the absence of tests. Untested code can't be safely changed.
2. **Get it under test before changing it** — The first step is always to add characterization tests that capture current behavior, even if that behavior has bugs.
3. **Find seams, don't rewrite** — Seams are points where you can change behavior without editing the source. Use them to break dependencies for testing.
4. **Preserve behavior first** — Understand and lock down existing behavior before improving it. Don't mix behavior changes with structural improvements.
5. **Small, safe steps** — Every change should be the smallest possible step. Commit frequently. If something breaks, revert one step, not a day's work.

### When to Use vs. When to Avoid

**Use these techniques when:**
- You need to modify code that has no tests
- The codebase is too large or critical to rewrite
- Dependencies are tangled and hard to isolate
- You're working with unfamiliar code
- Changes keep introducing regressions

**Consider rewriting when:**
- The code is small enough to rewrite in < 1 week
- The technology is truly obsolete and unsupportable
- The code is isolated and replaceable (strangler fig)
- Understanding the code is harder than rebuilding it

---

## Frameworks and Methodologies

### 1. The Legacy Code Change Algorithm — step-by-step
1. **Identify change points** — Where in the code do you need to make changes?
2. **Find test points** — Where can you observe the effects of your change?
3. **Break dependencies** — Use seams to isolate the code you need to test
4. **Write characterization tests** — Tests that describe what the code does now
5. **Make changes and refactor** — With tests in place, safely modify and improve
6. **Run all tests** — Verify nothing is broken

### 2. Sprout and Wrap Techniques — step-by-step
1. **Sprout Method**: Write new functionality in a new, tested method. Call it from the existing code.
2. **Sprout Class**: When the existing class is too tangled, create a new tested class for new functionality.
3. **Wrap Method**: Rename the existing method, create a new method with the old name that calls the original plus new behavior.
4. **Wrap Class**: Create a decorator that wraps the original class and adds behavior.
5. Choose based on how tangled the existing code is and how much you can safely modify.

---

## How to Apply

### Decision Checklist
- [ ] Have I written characterization tests before making changes?
- [ ] Have I identified seams for breaking dependencies?
- [ ] Am I making the smallest possible change at each step?
- [ ] Am I separating structural changes from behavior changes?
- [ ] Am I committing after each successful step?

### Implementation Patterns

**Characterization Test:**
```python
# You don't know what this function does exactly.
# Write a test that captures its CURRENT behavior.

def test_calculate_shipping_characterization():
    # Run the function with known inputs
    result = calculate_shipping(weight=5.0, destination="US", priority=True)
    # Whatever it returns, THAT is the expected value
    assert result == 12.50  # Captured from actual execution

    result = calculate_shipping(weight=0, destination="EU", priority=False)
    assert result == 5.00   # Captured from actual execution

    # Now you have a safety net for refactoring
```

**Finding and Using Seams:**
```python
# Original code — hard to test because it calls the database directly
class InvoiceProcessor:
    def process(self, invoice_id):
        invoice = db.query("SELECT * FROM invoices WHERE id = %s", invoice_id)
        if invoice.amount > 1000:
            send_email(invoice.customer_email, "Large invoice alert")
        invoice.status = "processed"
        db.update(invoice)

# Object Seam — extract dependency into a method you can override
class InvoiceProcessor:
    def process(self, invoice_id):
        invoice = self._get_invoice(invoice_id)
        if invoice.amount > 1000:
            self._send_alert(invoice.customer_email)
        invoice.status = "processed"
        self._save_invoice(invoice)

    def _get_invoice(self, invoice_id):  # Seam
        return db.query("SELECT * FROM invoices WHERE id = %s", invoice_id)

    def _send_alert(self, email):  # Seam
        send_email(email, "Large invoice alert")

    def _save_invoice(self, invoice):  # Seam
        db.update(invoice)

# Test with overridden seams
class TestableInvoiceProcessor(InvoiceProcessor):
    def __init__(self):
        self.saved_invoices = []
        self.sent_alerts = []

    def _get_invoice(self, invoice_id):
        return Invoice(id=invoice_id, amount=1500, customer_email="test@test.com")

    def _send_alert(self, email):
        self.sent_alerts.append(email)

    def _save_invoice(self, invoice):
        self.saved_invoices.append(invoice)

def test_large_invoice_sends_alert():
    processor = TestableInvoiceProcessor()
    processor.process("inv_123")
    assert len(processor.sent_alerts) == 1
    assert processor.saved_invoices[0].status == "processed"
```

**Sprout Method:**
```python
# Existing code you can't easily test
def process_order(order):
    # ... 50 lines of tangled logic ...
    order.total = calculate_complex_total(order)
    # ... 30 more lines ...

# Sprout: add new functionality in a NEW, tested method
def apply_loyalty_discount(order: Order, customer: Customer) -> Decimal:
    """New functionality — fully tested independently."""
    if customer.loyalty_years >= 5:
        return order.total * Decimal("0.15")
    elif customer.loyalty_years >= 2:
        return order.total * Decimal("0.05")
    return Decimal("0")

# Call sprout from existing code (minimal edit)
def process_order(order):
    # ... 50 lines of tangled logic ...
    order.total = calculate_complex_total(order)
    order.discount = apply_loyalty_discount(order, order.customer)  # New line
    # ... 30 more lines ...
```

**Scratch Refactoring:**
```
1. Create a branch for exploration (will NOT be merged)
2. Aggressively refactor to understand the code
3. Extract methods, rename variables, reorganize
4. Build a mental model of what the code does
5. Throw away the branch
6. Now write characterization tests with your new understanding
7. Do the real refactoring properly, with tests
```

### Common Mistakes
1. **Changing behavior while refactoring** — Mixing structural changes with behavior changes makes bugs invisible
2. **Rewriting instead of refactoring** — "Let's just rewrite this" fails more often than incremental improvement
3. **No characterization tests** — Modifying legacy code without capturing its current behavior
4. **Too-big changes** — Making large changes without intermediate commits; when it breaks, everything is lost
5. **Gold-plating legacy code** — Spending weeks making legacy code perfect when only a small area needs changes
6. **Giving up** — Legacy code is hard but manageable with the right techniques

### Integration with Other Topics
- **Refactoring** — Refactoring techniques applied to untested code (with extra safety steps)
- **TDD** — Once under test, use TDD for new changes
- **Technical Debt** — Legacy code is often the largest source of technical debt
- **Clean Code** — The goal after getting legacy code under test
- **Microservices (Strangler Fig)** — Incrementally replacing legacy modules with new services
- **Code Review** — Reviews catch regressions when modifying legacy code

---

## Resources

### Essential Reading
- *Working Effectively with Legacy Code* — Michael Feathers
- *Refactoring* (2nd ed.) — Martin Fowler
- *Your Code as a Crime Scene* — Adam Tornhill

### Online Resources
- Michael Feathers' talks on legacy code techniques
- Emily Bache's "Approval Testing" for legacy code
- Coding Dojo katas for practicing legacy code techniques (Gilded Rose)

### Practice Exercises
1. Write characterization tests for the Gilded Rose kata
2. Identify seams in a legacy class and extract testable methods
3. Use Sprout Method to add new functionality to untested code
4. Do a scratch refactoring session and then discard it
5. Apply the Legacy Code Change Algorithm to a real codebase
