# Property-Based Testing

## Profile

### What It Is
Property-based testing (PBT) generates random inputs to verify that properties (invariants) hold for all inputs, not just hand-picked examples. Hypothesis is Python's PBT library — it generates test data, finds counterexamples, and shrinks them to minimal failing cases. PBT catches edge cases that example-based tests miss.

### Origin and Evolution
- QuickCheck (1999, Haskell) — original PBT library by Koen Claessen & John Hughes
- ScalaCheck, FsCheck — ports to other languages
- Hypothesis (2015, David MacIver) — Python PBT library
- Hypothesis integrated with pytest
- Schemathesis (2019) — PBT for API testing (OpenAPI schemas)
- Current: Hypothesis is mature and widely used; stateful testing for complex systems

### Key Authors and References
- **David MacIver** — Hypothesis creator
- **Koen Claessen & John Hughes** — QuickCheck inventors
- **Hillel Wayne** — Property-based testing advocacy
- **Hypothesis documentation** — comprehensive, well-written

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Property | An invariant that must hold for all valid inputs |
| Strategy | A generator of test data (`st.integers()`, `st.text()`) |
| Shrinking | Automatically reducing a failing input to the simplest counterexample |
| @given | Decorator that feeds generated data to a test |
| @example | Add specific edge cases alongside generated data |
| Stateful Testing | Generate sequences of operations to test state machines |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Test properties, not examples** — Instead of "sort([3,1,2]) == [1,2,3]", test "sorted output is same length, same elements, and each element ≤ next".
2. **Shrinking finds minimal bugs** — Hypothesis doesn't just find a failing input — it reduces it to the simplest case that still fails. Debug a 3-element list, not a 1000-element one.
3. **Combine with example tests** — PBT doesn't replace example tests. Use `@example` to pin important edge cases. Use `@given` to find unknown ones.
4. **Strategies compose** — Build complex data generators from simple ones. `st.lists(st.integers())`, `st.builds(Order, ...)`, `st.from_type(MyModel)`.
5. **Stateful testing for systems** — RuleBasedStateMachine tests sequences of operations against a model. Finds bugs that individual operations don't.

### When to Use vs. When to Avoid

**Use PBT when:**
- Pure functions with well-defined input/output contracts
- Serialization/deserialization (roundtrip property: decode(encode(x)) == x)
- Data structures (sorted, unique, contains invariants)
- Parsers and validators (never crash on valid input)
- APIs (Schemathesis for OpenAPI-based testing)

**Example-based tests suffice when:**
- Business logic with very specific expected outcomes
- UI behavior tests
- Integration tests with external systems
- Small, well-understood input space

---

## Frameworks and Methodologies

### 1. Property Identification — step-by-step
1. Identify the function under test
2. Think about invariants: what's ALWAYS true about the output?
3. Common properties: roundtrip, idempotency, commutativity, invariant preservation
4. Write the `@given` decorator with appropriate strategies
5. Add `@example` for known edge cases
6. Run and examine counterexamples

### 2. Stateful Testing — step-by-step
1. Identify the system under test (data structure, API, service)
2. Define rules: operations with preconditions
3. Define a model: simplified expected behavior
4. Create a `RuleBasedStateMachine` subclass
5. Hypothesis generates sequences of rules and checks invariants
6. Examine minimal failing sequences

---

## How to Apply

### Decision Checklist
- [ ] Are roundtrip properties tested for serialization/parsing?
- [ ] Are invariants identified for data structure operations?
- [ ] Are strategies composing to generate realistic domain data?
- [ ] Are `@example` decorators pinning known edge cases?
- [ ] Is stateful testing used for complex state-dependent systems?

### Implementation Patterns

**Basic Properties:**
```python
from hypothesis import given, example, assume, settings
from hypothesis import strategies as st

# Roundtrip property
@given(st.text())
def test_json_roundtrip(s):
    encoded = json.dumps(s)
    decoded = json.loads(encoded)
    assert decoded == s

# Sort properties
@given(st.lists(st.integers()))
def test_sort_preserves_length(xs):
    assert len(sorted(xs)) == len(xs)

@given(st.lists(st.integers()))
def test_sort_preserves_elements(xs):
    assert sorted(sorted(xs)) == sorted(xs)  # Idempotent

@given(st.lists(st.integers(), min_size=2))
def test_sort_is_ordered(xs):
    result = sorted(xs)
    for a, b in zip(result, result[1:]):
        assert a <= b

# Known edge case + generated
@example("")
@example("   ")
@example("a@b.com")
@given(st.text())
def test_email_validation_never_crashes(email):
    # Property: validation never raises (returns bool)
    result = is_valid_email(email)
    assert isinstance(result, bool)
```

**Custom Strategies:**
```python
from hypothesis import strategies as st
from decimal import Decimal

# Composite strategy for domain objects
@st.composite
def order_items(draw):
    return OrderItem(
        product_id=draw(st.text(min_size=1, max_size=10, alphabet=st.characters(whitelist_categories=("L",)))),
        quantity=draw(st.integers(min_value=1, max_value=100)),
        price=draw(st.decimals(min_value=Decimal("0.01"), max_value=Decimal("9999.99"), places=2)),
    )

@st.composite
def orders(draw):
    items = draw(st.lists(order_items(), min_size=1, max_size=10))
    return Order(
        customer_id=draw(st.uuids().map(str)),
        items=items,
    )

@given(orders())
def test_order_total_is_positive(order):
    assert order.total > 0

@given(orders())
def test_order_total_equals_sum_of_items(order):
    expected = sum(item.price * item.quantity for item in order.items)
    assert order.total == expected

# Strategy from Pydantic model
from hypothesis import strategies as st
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int
    email: str

# Hypothesis can generate from type annotations
@given(st.from_type(User))
def test_user_serialization_roundtrip(user):
    data = user.model_dump()
    restored = User.model_validate(data)
    assert restored == user
```

**Stateful Testing:**
```python
from hypothesis.stateful import RuleBasedStateMachine, rule, invariant, initialize

class SetModel(RuleBasedStateMachine):
    """Test a custom Set implementation against Python's built-in set."""

    def __init__(self):
        super().__init__()
        self.model = set()          # Expected behavior
        self.impl = MyCustomSet()   # Implementation under test

    @rule(value=st.integers())
    def add(self, value):
        self.model.add(value)
        self.impl.add(value)

    @rule(value=st.integers())
    def discard(self, value):
        self.model.discard(value)
        self.impl.discard(value)

    @rule(value=st.integers())
    def check_contains(self, value):
        assert (value in self.model) == self.impl.contains(value)

    @invariant()
    def size_matches(self):
        assert len(self.model) == self.impl.size()

# Run: Hypothesis generates sequences of add/discard/check operations
TestSetModel = SetModel.TestCase
```

**API Testing with Schemathesis:**
```python
# Test API against OpenAPI schema
import schemathesis

schema = schemathesis.from_url("http://localhost:8000/openapi.json")

@schema.parametrize()
def test_api(case):
    response = case.call()
    case.validate_response(response)
    # Properties: no 500 errors, response matches schema
```

### Common Mistakes
1. **Testing too specific** — `assert sort(xs) == expected` is an example test, not a property test; test invariants
2. **Unbounded strategies** — `st.text()` without limits generates huge strings; constrain for realistic data
3. **Ignoring `@example`** — Not pinning known edge cases; PBT might not find them every run
4. **Flaky tests from randomness** — Set `@settings(max_examples=200)` for CI stability; use database for reproducing
5. **Not reading counterexamples** — Hypothesis prints the minimal failing case; read it carefully

### Integration with Other Topics
- **Testing with pytest** — Hypothesis integrates natively with pytest
- **Pydantic** — `st.from_type()` generates valid Pydantic models
- **Error Handling** — PBT ensures error handling covers edge cases
- **Serialization** — Roundtrip properties for JSON, Protobuf, etc.
- **Data Validation** — Test validators with random inputs to find crashes

---

## Resources

### Essential Reading
- Hypothesis documentation (hypothesis.readthedocs.io)
- Hillel Wayne — "Property-Based Testing" (hillelwayne.com)
- *QuickCheck: A Lightweight Tool for Random Testing* — original paper

### Online Resources
- Hypothesis examples and strategies cookbook
- Schemathesis documentation
- Real Python — Hypothesis tutorials

### Practice Exercises
1. Write property tests for a sort function (length, elements, ordering)
2. Create a custom composite strategy for a domain model
3. Test serialization roundtrip for a Pydantic model with `st.from_type`
4. Build a stateful test for a shopping cart (add, remove, checkout)
5. Use Schemathesis to test a FastAPI endpoint against its OpenAPI schema
