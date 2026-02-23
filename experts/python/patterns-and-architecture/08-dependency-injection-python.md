# Dependency Injection in Python

## Profile

### What It Is
Dependency Injection (DI) in Python is the practice of providing dependencies to a function or class from the outside rather than creating them internally. Python's dynamic nature makes DI simpler than in static languages — constructor injection, function parameters, and module-level configuration often suffice without a framework.

### Origin and Evolution
- DI concept from Martin Fowler's IoC article (2004)
- Java/C# heavy DI frameworks (Spring, .NET DI)
- Python: manual DI preferred by most of the community
- `dependency-injector` library — container-based DI for Python
- `punq` — lightweight DI container
- FastAPI's `Depends()` — function-level DI for web endpoints
- Current: manual DI + FastAPI Depends is the dominant pattern

### Key Authors and References
- **Martin Fowler** — Inversion of Control Containers and the Dependency Injection pattern
- **Harry Percival & Bob Gregory** — *Architecture Patterns with Python* (DI chapters)
- **Robert C. Martin** — Dependency Inversion Principle (SOLID)
- **Sebastian Ramirez** — FastAPI Depends system

### Key Concepts at a Glance
| Approach | Description |
|----------|-------------|
| Constructor Injection | Pass dependencies to `__init__` |
| Function Parameter Injection | Dependencies as function arguments with defaults |
| FastAPI Depends | Declarative DI for route handlers |
| DI Container | Automated resolution (dependency-injector, punq) |
| Module-level Wiring | Configure at application startup, pass through layers |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Manual DI first** — In Python, pass dependencies as parameters. No framework needed for most cases.
2. **Depend on abstractions** — Accept Protocol or ABC, not concrete implementations. Enables testing and swapping.
3. **Wire at the composition root** — Create and connect all dependencies at the application entry point (main, app factory). Don't create dependencies deep in the call chain.
4. **Defaults for convenience** — Provide default implementations for common cases. Override in tests or configuration.
5. **Don't fight Python** — Heavy DI containers add Java-like complexity. Use Python's flexibility: functions, closures, module-level instances.

### When to Use vs. When to Avoid

**Use DI when:**
- Components need different implementations (test vs. production)
- External services need to be mocked in tests
- Multiple configurations of the same component exist
- Clean Architecture boundaries require explicit dependencies

**Keep it simple when:**
- Internal utilities with no external dependencies
- Small scripts where everything is wired linearly
- Dependencies that never change (e.g., standard library modules)

---

## Frameworks and Methodologies

### 1. Manual DI Setup — step-by-step
1. Define dependencies as Protocol or ABC (interfaces)
2. Accept dependencies in `__init__` or function parameters
3. Provide default implementations for convenience
4. Create a composition root (factory function or main)
5. Wire dependencies at startup
6. Override in tests with fakes or mocks

### 2. FastAPI Depends Pattern — step-by-step
1. Define dependency functions (sync or async)
2. Use `Depends()` in route handler signatures
3. Chain dependencies (a depends on b depends on c)
4. Override dependencies in tests with `app.dependency_overrides`
5. Use sub-dependencies for shared resources (DB session, auth)

---

## How to Apply

### Decision Checklist
- [ ] Are external dependencies injected, not created internally?
- [ ] Is there a composition root where dependencies are wired?
- [ ] Can dependencies be swapped for testing without monkey-patching?
- [ ] Are Protocol/ABC used for dependency interfaces?
- [ ] Is the DI approach proportional to the project's complexity?

### Implementation Patterns

**Manual Constructor Injection:**
```python
from typing import Protocol

class EmailSender(Protocol):
    def send(self, to: str, subject: str, body: str) -> None: ...

class NotificationService:
    def __init__(self, email_sender: EmailSender) -> None:
        self._email = email_sender

    def notify_user(self, user_id: str, message: str) -> None:
        user = self._get_user(user_id)
        self._email.send(
            to=user.email,
            subject="Notification",
            body=message,
        )

# Production wiring (composition root)
def create_app() -> NotificationService:
    smtp_sender = SmtpEmailSender(host="smtp.example.com", port=587)
    return NotificationService(email_sender=smtp_sender)

# Test wiring
def test_notify_user():
    fake_sender = FakeEmailSender()
    service = NotificationService(email_sender=fake_sender)
    service.notify_user("u1", "Hello")
    assert fake_sender.sent[0].to == "user@example.com"
```

**Function-Level DI with Defaults:**
```python
from functools import partial

def fetch_orders(
    customer_id: str,
    *,
    db: Database = None,
    cache: Cache = None,
) -> list[Order]:
    db = db or get_default_database()
    cache = cache or get_default_cache()

    cached = cache.get(f"orders:{customer_id}")
    if cached:
        return cached

    orders = db.query(Order).filter_by(customer_id=customer_id).all()
    cache.set(f"orders:{customer_id}", orders, ttl=300)
    return orders

# Test: override dependencies
def test_fetch_orders():
    fake_db = FakeDatabase(orders=[Order(id="1")])
    fake_cache = FakeCache()
    orders = fetch_orders("c1", db=fake_db, cache=fake_cache)
    assert len(orders) == 1
```

**FastAPI Depends:**
```python
from fastapi import FastAPI, Depends
from sqlalchemy.ext.asyncio import AsyncSession

app = FastAPI()

# Dependency: database session
async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        yield session

# Dependency: current user (depends on db)
async def get_current_user(
    token: str = Header(),
    db: AsyncSession = Depends(get_db),
) -> User:
    user = await authenticate(token, db)
    if not user:
        raise HTTPException(status_code=401)
    return user

# Route: dependencies injected automatically
@app.get("/orders")
async def list_orders(
    user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
) -> list[OrderResponse]:
    orders = await db.execute(
        select(Order).where(Order.user_id == user.id)
    )
    return [OrderResponse.model_validate(o) for o in orders.scalars()]

# Test: override dependencies
@pytest.fixture
def client():
    app.dependency_overrides[get_db] = lambda: fake_db_session
    app.dependency_overrides[get_current_user] = lambda: fake_user
    with TestClient(app) as c:
        yield c
    app.dependency_overrides.clear()
```

**Composition Root Pattern:**
```python
# app/container.py — composition root
from dataclasses import dataclass

@dataclass
class Container:
    db: Database
    cache: Cache
    email: EmailSender
    order_service: OrderService
    notification_service: NotificationService

def create_container(config: Config) -> Container:
    db = PostgresDatabase(config.database_url)
    cache = RedisCache(config.redis_url)
    email = SmtpEmailSender(config.smtp_host)

    order_service = OrderService(db=db, cache=cache)
    notification_service = NotificationService(email=email)

    return Container(
        db=db,
        cache=cache,
        email=email,
        order_service=order_service,
        notification_service=notification_service,
    )

# main.py
container = create_container(Config())
app = create_fastapi_app(container)
```

### Common Mistakes
1. **Creating dependencies inside classes** — `self.db = Database()` inside `__init__`; impossible to test or swap
2. **Over-engineering with containers** — Using dependency-injector for a 3-class project; manual DI suffices
3. **No composition root** — Dependencies created everywhere; no single place to see the wiring
4. **Monkey-patching instead of DI** — Using `unittest.mock.patch` to replace modules instead of injecting dependencies
5. **Too many dependencies** — Class with 10 constructor parameters; indicates it does too much

### Integration with Other Topics
- **Clean Architecture** — DI enforces dependency inversion at layer boundaries
- **Testing** — DI enables testing with fakes/stubs without monkey-patching
- **Design Patterns** — Strategy pattern is DI of behavior
- **Web Frameworks** — FastAPI Depends, Django's middleware as DI
- **SOLID** — Dependency Inversion Principle (the D in SOLID)

---

## Resources

### Essential Reading
- *Architecture Patterns with Python* — Percival & Gregory
- Martin Fowler — "Inversion of Control Containers and DI" (martinfowler.com)
- FastAPI Depends documentation

### Online Resources
- dependency-injector docs (python-dependency-injector.ets-labs.org)
- punq documentation (punq.readthedocs.io)
- Real Python — DI patterns tutorials

### Practice Exercises
1. Refactor a class that creates its own dependencies to use constructor injection
2. Build a composition root for a 3-layer application
3. Implement FastAPI Depends chain with DB session and auth
4. Write tests using injected fakes (no mocking framework)
5. Compare manual DI vs. dependency-injector container for the same use case
