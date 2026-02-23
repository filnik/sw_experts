# Testing with pytest

## Profile

### What It Is
pytest is Python's dominant testing framework. It uses plain functions and `assert` statements (no boilerplate classes), powerful fixtures for setup/teardown, parametrize for data-driven tests, markers for test categorization, and a rich plugin ecosystem. It replaces unittest for virtually all Python testing needs.

### Origin and Evolution
- unittest (2001) — Python's built-in, JUnit-style testing
- nose (2005) — Extended unittest with discovery and plugins
- pytest (2007, Holger Krekel) — Modern testing with minimal boilerplate
- pytest-asyncio — Async test support
- pytest-cov, pytest-xdist — Coverage and parallel execution
- Current: pytest is the de facto standard; unittest mainly for legacy code

### Key Authors and References
- **Holger Krekel** — pytest creator
- **Brian Okken** — *Python Testing with pytest* (2nd ed., 2022)
- **Harry Percival** — *Test-Driven Development with Python*
- **pytest documentation** — comprehensive reference

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Fixtures | Reusable setup/teardown via dependency injection |
| Parametrize | Run same test with multiple inputs |
| Markers | Categorize tests (slow, integration, skip) |
| conftest.py | Shared fixtures and hooks, auto-discovered |
| Plugins | pytest-cov, pytest-xdist, pytest-mock, pytest-asyncio |
| assert rewriting | Plain `assert` with detailed failure messages |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Plain assert, no boilerplate** — `assert x == y` with automatic detailed failure messages. No `self.assertEqual`, no test classes required.
2. **Fixtures for dependency injection** — Fixtures provide test dependencies. They compose, have scopes, and clean up automatically.
3. **Parametrize for data-driven tests** — One test function, multiple inputs. Avoid copy-pasting test logic for different data.
4. **Conftest for shared setup** — `conftest.py` provides fixtures to all tests in its directory. Layer conftest files for different scopes.
5. **Fast feedback with markers** — Mark tests as `slow`, `integration`, `e2e`. Run `pytest -m "not slow"` for fast feedback during development.

### When to Use vs. When to Avoid

**Use pytest when:**
- Any Python testing (unit, integration, functional, E2E)
- Starting a new project (always choose pytest over unittest)
- Need powerful fixtures and parametrize

**Use unittest when:**
- Contributing to a project that already uses unittest
- Python standard library development (unittest is built-in)

---

## Frameworks and Methodologies

### 1. Test Suite Organization — step-by-step
1. Mirror source structure in `tests/` directory
2. Create `conftest.py` at test root for shared fixtures
3. Separate `tests/unit/`, `tests/integration/`, `tests/e2e/`
4. Use markers to categorize tests
5. Configure pytest in `pyproject.toml`
6. Add coverage with `pytest-cov`

### 2. Fixture Design — step-by-step
1. Identify shared setup (DB connection, test client, sample data)
2. Create fixtures in `conftest.py` at the appropriate level
3. Use `yield` fixtures for setup/teardown
4. Choose scope: `function` (default), `class`, `module`, `session`
5. Compose fixtures (fixture depends on other fixture)
6. Use fixture factories for configurable test data

---

## How to Apply

### Decision Checklist
- [ ] Are tests using plain `assert` (not unittest assertions)?
- [ ] Are shared fixtures in `conftest.py`?
- [ ] Is `parametrize` used for data-driven tests?
- [ ] Are markers configured for test categorization?
- [ ] Is coverage configured and measured in CI?

### Implementation Patterns

**Basic Tests:**
```python
# tests/test_order.py
from my_app.domain.order import Order, OrderItem

def test_order_total():
    order = Order(
        items=[
            OrderItem(product="A", price=10.0, quantity=2),
            OrderItem(product="B", price=5.0, quantity=1),
        ]
    )
    assert order.total == 25.0

def test_order_requires_items():
    with pytest.raises(ValueError, match="at least one item"):
        Order(items=[])

def test_order_discount():
    order = Order(items=[OrderItem(product="A", price=100.0, quantity=1)])
    order.apply_discount(0.10)
    assert order.total == pytest.approx(90.0)
```

**Fixtures:**
```python
# tests/conftest.py
import pytest
from my_app.database import Database
from my_app.domain.order import Order, OrderItem

@pytest.fixture
def sample_items() -> list[OrderItem]:
    return [
        OrderItem(product="Widget", price=29.99, quantity=2),
        OrderItem(product="Gadget", price=49.99, quantity=1),
    ]

@pytest.fixture
def sample_order(sample_items) -> Order:
    return Order(customer_id="C123", items=sample_items)

# Yield fixture for setup/teardown
@pytest.fixture
def db():
    database = Database(":memory:")
    database.create_tables()
    yield database
    database.close()

# Session-scoped fixture (created once per test session)
@pytest.fixture(scope="session")
def api_client():
    client = TestClient(app)
    yield client

# Fixture factory
@pytest.fixture
def make_order():
    def _make(customer_id: str = "C1", item_count: int = 1) -> Order:
        items = [OrderItem(product=f"P{i}", price=10.0, quantity=1) for i in range(item_count)]
        return Order(customer_id=customer_id, items=items)
    return _make

def test_large_order(make_order):
    order = make_order(item_count=100)
    assert len(order.items) == 100
```

**Parametrize:**
```python
import pytest

@pytest.mark.parametrize("input_str,expected", [
    ("hello@example.com", True),
    ("invalid-email", False),
    ("", False),
    ("user@domain.co.uk", True),
    ("user@.com", False),
])
def test_is_valid_email(input_str, expected):
    assert is_valid_email(input_str) == expected

# Multiple parameters combined
@pytest.mark.parametrize("amount", [100, 500, 1000])
@pytest.mark.parametrize("currency", ["USD", "EUR", "GBP"])
def test_currency_conversion(amount, currency):
    result = convert(amount, currency, "JPY")
    assert result > 0
```

**Markers and Configuration:**
```python
# pyproject.toml
# [tool.pytest.ini_options]
# markers = [
#     "slow: marks tests as slow",
#     "integration: marks integration tests",
#     "e2e: marks end-to-end tests",
# ]
# testpaths = ["tests"]
# addopts = "-v --tb=short"

import pytest

@pytest.mark.slow
def test_full_pipeline():
    ...

@pytest.mark.integration
def test_database_query(db):
    ...

# Run: pytest -m "not slow"
# Run: pytest -m integration
# Run: pytest -k "test_order"  (name matching)
```

**Async Tests:**
```python
import pytest
from httpx import AsyncClient

@pytest.fixture
async def async_client():
    async with AsyncClient(app=app, base_url="http://test") as client:
        yield client

@pytest.mark.asyncio
async def test_create_order(async_client):
    response = await async_client.post("/orders", json={
        "customer_id": "C1",
        "items": [{"product_id": "P1", "quantity": 1}],
    })
    assert response.status_code == 201
    assert response.json()["id"]
```

### Common Mistakes
1. **Test classes for everything** — pytest doesn't need classes; plain functions are simpler
2. **Fixture scope too broad** — Session-scoped fixtures with mutable state cause test pollution
3. **Parametrize with too many cases** — 50 parametrize entries; use property-based testing instead
4. **No conftest layers** — All fixtures in one huge conftest; layer by directory
5. **Testing implementation, not behavior** — Asserting internal state instead of observable behavior

### Integration with Other Topics
- **Mocking** — pytest-mock and monkeypatch for test doubles
- **TDD** — pytest is the tool for TDD in Python
- **Code Quality** — pytest-cov for coverage; run in CI
- **Async Programming** — pytest-asyncio for async tests
- **Property-Based Testing** — Hypothesis integrates with pytest

---

## Resources

### Essential Reading
- *Python Testing with pytest* (2nd ed.) — Brian Okken
- pytest documentation (docs.pytest.org)
- *Test-Driven Development with Python* — Harry Percival

### Online Resources
- Real Python — pytest tutorials
- pytest plugin directory (docs.pytest.org/en/latest/reference/plugin_list.html)
- pytest-dev GitHub organization

### Practice Exercises
1. Convert a unittest test suite to pytest (remove classes, use assert)
2. Design a fixture hierarchy with conftest.py layers
3. Write parametrized tests for a validation function with 10+ cases
4. Set up markers and run subsets of tests
5. Configure pytest-cov and achieve 80% coverage on a module
