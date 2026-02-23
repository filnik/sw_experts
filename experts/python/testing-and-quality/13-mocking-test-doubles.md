# Mocking and Test Doubles

## Profile

### What It Is
Mocking and test doubles enable testing code in isolation by replacing real dependencies with controlled substitutes. Python provides `unittest.mock` (Mock, MagicMock, patch, AsyncMock), pytest's `monkeypatch`, and community tools like `responses` (HTTP), `freezegun` (time), and `factory_boy` (test data).

### Origin and Evolution
- unittest.mock (2011) — Python's standard mocking library
- `mock` backport for Python 2
- pytest monkeypatch — attribute/environment replacement
- responses (2014) — Mock HTTP requests (requests library)
- respx (2020) — Mock HTTP for httpx (async)
- freezegun (2013) — Mock datetime.now()
- Current: unittest.mock + pytest fixtures + specialized mock libraries

### Key Authors and References
- **Michael Foord** — unittest.mock creator
- **Brian Okken** — Mocking in *Python Testing with pytest*
- **Harry Percival & Bob Gregory** — Test doubles in *Architecture Patterns with Python*
- **Martin Fowler** — "Mocks Aren't Stubs" article

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Mock | Object that records calls and returns configured values |
| MagicMock | Mock with default implementations of magic methods |
| AsyncMock | Mock for async functions |
| patch | Context manager/decorator that replaces an attribute temporarily |
| monkeypatch | pytest fixture for attribute/env replacement |
| Fake | Working implementation with shortcuts (in-memory DB) |
| Stub | Returns predefined data, no logic |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Prefer fakes over mocks** — A fake repository (in-memory dict) is more reliable than a mock with configured returns. Fakes have behavior; mocks just echo.
2. **Mock at the boundary, not everywhere** — Mock external systems (HTTP APIs, databases, file systems). Don't mock your own code — refactor it to be testable instead.
3. **Don't mock what you don't own** — Instead of mocking `requests.get`, create a wrapper and mock that. Or use `responses` to mock at the HTTP level.
4. **Assert behavior, not implementation** — Verify that the right result was produced, not that internal methods were called in a specific order.
5. **Inject instead of patch** — Dependency injection makes mocking unnecessary. Pass the dependency; in tests, pass the fake.

### When to Use vs. When to Avoid

**Use mocking when:**
- External APIs, databases, or file systems in unit tests
- Time-dependent logic (`datetime.now()`, `time.time()`)
- Non-deterministic behavior (random, UUIDs)
- Slow or unreliable dependencies

**Avoid mocking when:**
- You can use a fake implementation instead
- You can refactor to inject the dependency
- You're mocking your own code (sign of bad design)
- Integration tests where you want real behavior

---

## Frameworks and Methodologies

### 1. Test Double Strategy — step-by-step
1. Identify external dependencies in the unit under test
2. For each dependency, choose: fake, stub, or mock
3. Prefer fakes for repositories and services
4. Use mocks/stubs for external APIs and infrastructure
5. Use specialized libraries (responses, freezegun) over raw Mock
6. Verify behavior (output), not interactions (method calls)

### 2. From Mock-Heavy to DI — step-by-step
1. Identify tests with many `patch` decorators
2. Extract the patched dependency as a constructor parameter
3. Define a Protocol for the dependency
4. Create a fake implementation for tests
5. Pass the fake directly instead of patching
6. Remove `patch` decorators

---

## How to Apply

### Decision Checklist
- [ ] Are fakes preferred over mocks for domain dependencies?
- [ ] Is mocking limited to external system boundaries?
- [ ] Are specialized mock libraries used (responses, freezegun)?
- [ ] Is DI used to avoid excessive patching?
- [ ] Are assertions on behavior (output), not interactions?

### Implementation Patterns

**Fakes over Mocks:**
```python
# Protocol definition
from typing import Protocol

class OrderRepository(Protocol):
    def get(self, order_id: str) -> Order | None: ...
    def save(self, order: Order) -> None: ...

# Fake for testing (has real behavior)
class FakeOrderRepository:
    def __init__(self) -> None:
        self._orders: dict[str, Order] = {}

    def get(self, order_id: str) -> Order | None:
        return self._orders.get(order_id)

    def save(self, order: Order) -> None:
        self._orders[order.id] = order

# Test with fake — no mock framework needed
def test_cancel_order():
    repo = FakeOrderRepository()
    order = Order(id="1", status="active")
    repo.save(order)

    service = OrderService(repo=repo)
    service.cancel_order("1")

    assert repo.get("1").status == "cancelled"
```

**unittest.mock Patterns:**
```python
from unittest.mock import Mock, MagicMock, patch, AsyncMock, call

# Basic mock
mock_sender = Mock()
mock_sender.send.return_value = {"status": "sent"}

service = NotificationService(sender=mock_sender)
service.notify("user@example.com", "Hello")

mock_sender.send.assert_called_once_with(
    to="user@example.com",
    body="Hello",
)

# patch as decorator
@patch("my_app.services.httpx.get")
def test_fetch_user(mock_get):
    mock_get.return_value = Mock(
        status_code=200,
        json=lambda: {"id": "1", "name": "Alice"},
    )
    user = fetch_user("1")
    assert user.name == "Alice"
    mock_get.assert_called_once()

# patch as context manager
def test_with_patch():
    with patch("my_app.services.get_current_time") as mock_time:
        mock_time.return_value = datetime(2024, 1, 1)
        result = create_order()
        assert result.created_at == datetime(2024, 1, 1)

# AsyncMock
@pytest.mark.asyncio
async def test_async_service():
    mock_client = AsyncMock()
    mock_client.get.return_value = Mock(json=lambda: {"data": "value"})

    result = await fetch_data(client=mock_client)
    assert result == {"data": "value"}
```

**Specialized Libraries:**
```python
# responses — mock HTTP for requests library
import responses

@responses.activate
def test_api_client():
    responses.add(
        responses.GET,
        "https://api.example.com/users/1",
        json={"id": "1", "name": "Alice"},
        status=200,
    )
    user = api_client.get_user("1")
    assert user.name == "Alice"

# respx — mock HTTP for httpx (async)
import respx

@respx.mock
@pytest.mark.asyncio
async def test_async_api():
    respx.get("https://api.example.com/orders").respond(
        json=[{"id": "1"}], status_code=200
    )
    orders = await fetch_orders()
    assert len(orders) == 1

# freezegun — mock time
from freezegun import freeze_time

@freeze_time("2024-06-15 10:30:00")
def test_scheduled_task():
    task = ScheduledTask(run_at="2024-06-15 10:30:00")
    assert task.is_due()

# time-machine — faster alternative to freezegun
import time_machine

@time_machine.travel("2024-06-15 10:30:00")
def test_scheduled():
    assert datetime.now().hour == 10
```

**monkeypatch (pytest):**
```python
def test_with_env_var(monkeypatch):
    monkeypatch.setenv("API_KEY", "test-key-123")
    config = load_config()
    assert config.api_key == "test-key-123"

def test_with_patched_attr(monkeypatch):
    monkeypatch.setattr("my_app.services.DEFAULT_TIMEOUT", 1)
    result = fetch_with_timeout("https://example.com")
    ...

def test_with_patched_function(monkeypatch):
    def fake_send(to, body):
        return {"status": "sent"}

    monkeypatch.setattr("my_app.notifications.send_email", fake_send)
    result = notify_user("u1", "Hello")
    assert result["status"] == "sent"
```

### Common Mistakes
1. **Mocking everything** — Tests that only test mocks; test behavior with fakes instead
2. **Patching wrong path** — `@patch("my_module.requests.get")` not `@patch("requests.get")`; patch where it's used
3. **Asserting call order** — Tests break when implementation changes order; assert on outputs
4. **Mock returning Mock** — Unconfigured mock returns Mock; test passes vacuously
5. **Not resetting mocks** — Shared mock accumulates calls across tests; use fresh mocks

### Integration with Other Topics
- **Testing with pytest** — Mocks are used within pytest fixtures and tests
- **Dependency Injection** — DI reduces need for patching; pass fakes directly
- **Async Programming** — AsyncMock for testing coroutines
- **Clean Architecture** — Mock at layer boundaries (ports); fake the adapters
- **Design Patterns** — Repository, Strategy patterns are naturally testable with fakes

---

## Resources

### Essential Reading
- *Python Testing with pytest* (2nd ed.) — Brian Okken
- Martin Fowler — "Mocks Aren't Stubs" (martinfowler.com)
- unittest.mock documentation (docs.python.org)

### Online Resources
- responses library documentation
- freezegun documentation
- Real Python — Mocking tutorials

### Practice Exercises
1. Replace 5 `@patch` decorators with dependency injection and fakes
2. Build a FakeRepository that implements the Repository Protocol
3. Mock an HTTP API with responses and test error handling
4. Use freezegun to test time-dependent scheduling logic
5. Write a test that uses AsyncMock for an async HTTP client
