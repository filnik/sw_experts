# Decorators and Metaclasses

## Profile

### What It Is
Decorators and metaclasses are Python's metaprogramming tools. Decorators modify or extend functions and classes without changing their source code. Metaclasses control class creation itself — defining how classes are constructed. Together they enable cross-cutting concerns, registration patterns, validation, and framework-level abstractions.

### Origin and Evolution
- Metaclasses from Python 2 (type as metaclass)
- PEP 318 (2004) — Function decorator syntax `@decorator`
- PEP 3129 (2007) — Class decorators
- Descriptor protocol — underlying mechanism for properties, classmethod, staticmethod
- Current: decorators widely used; metaclasses rare (dataclasses, Pydantic, ORMs use them internally)

### Key Authors and References
- **Luciano Ramalho** — Decorator and metaclass chapters in *Fluent Python*
- **David Beazley** — "Python 3 Metaprogramming" talk
- **Brett Slatkin** — Metaclass patterns in *Effective Python*

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Function Decorator | Wraps a function to add behavior (logging, caching, auth) |
| Class Decorator | Modifies a class after creation (add methods, register) |
| Metaclass | Controls class creation; `class Foo(metaclass=Meta)` |
| Descriptor | Object with `__get__`/`__set__`/`__delete__` for attribute access |
| `functools.wraps` | Preserves original function metadata when wrapping |
| `__init_subclass__` | Hook called when a class is subclassed (simpler than metaclass) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Decorators for cross-cutting concerns** — Use decorators for logging, timing, caching, authentication, retry logic — anything that applies across multiple functions.
2. **Preserve the wrapped function** — Always use `@functools.wraps` to preserve the original function's name, docstring, and signature.
3. **Prefer `__init_subclass__` over metaclasses** — Most metaclass use cases (registration, validation) can be solved with `__init_subclass__` (Python 3.6+), which is simpler.
4. **Decorators should be transparent** — A decorated function should behave like the original, plus the added behavior. Don't change the function's contract.
5. **Metaclasses are for frameworks** — If you're not building a framework (ORM, serialization library), you probably don't need metaclasses. They add complexity.

### When to Use vs. When to Avoid

**Use decorators when:**
- Adding cross-cutting behavior (logging, timing, retries, auth)
- Registering functions/classes in a registry
- Modifying function behavior without changing its code
- Building APIs (Flask routes, FastAPI endpoints)

**Use metaclasses when:**
- Building a framework that needs to control class creation
- Implementing ORMs, serialization libraries, or validation frameworks
- `__init_subclass__` and class decorators are insufficient

**Avoid metaclasses when:**
- A class decorator solves the problem
- `__init_subclass__` solves the problem
- The team isn't familiar with metaprogramming

---

## Frameworks and Methodologies

### 1. Decorator Design — step-by-step
1. Identify the cross-cutting concern (what behavior to add)
2. Write the decorator function with `@functools.wraps`
3. Handle both `*args` and `**kwargs` for generality
4. Decide if the decorator needs parameters (decorator factory)
5. Add type hints using `ParamSpec` and `TypeVar` for type safety
6. Test the decorator independently from the wrapped function

### 2. Registration Pattern — step-by-step
1. Create a registry (dict or list) at module level
2. Create a decorator that adds to the registry
3. Decorate classes/functions to register them
4. Use the registry for dispatch, discovery, or plugin loading
5. Alternative: use `__init_subclass__` for class registration

---

## How to Apply

### Decision Checklist
- [ ] Is `@functools.wraps` used in every function decorator?
- [ ] Are decorator factories used when parameters are needed?
- [ ] Is `__init_subclass__` considered before metaclasses?
- [ ] Are decorators tested independently?
- [ ] Is metaprogramming complexity justified by the use case?

### Implementation Patterns

**Function Decorators:**
```python
import functools
import time
import logging

logger = logging.getLogger(__name__)

# Simple decorator
def timed(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        logger.info(f"{func.__name__} took {elapsed:.3f}s")
        return result
    return wrapper

# Decorator with parameters (decorator factory)
def retry(max_attempts: int = 3, delay: float = 1.0):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    logger.warning(f"{func.__name__} attempt {attempt + 1} failed: {e}")
                    time.sleep(delay)
            raise last_exception
        return wrapper
    return decorator

@timed
@retry(max_attempts=3, delay=0.5)
def fetch_data(url: str) -> dict:
    ...
```

**Class Decorators and Registration:**
```python
# Plugin registry pattern
_handlers: dict[str, type] = {}

def handler(event_type: str):
    def decorator(cls):
        _handlers[event_type] = cls
        return cls
    return decorator

@handler("order.created")
class OrderCreatedHandler:
    def handle(self, event: dict) -> None:
        ...

@handler("order.cancelled")
class OrderCancelledHandler:
    def handle(self, event: dict) -> None:
        ...

# Dispatch
def dispatch(event_type: str, event: dict):
    handler_cls = _handlers.get(event_type)
    if handler_cls:
        handler_cls().handle(event)
```

**`__init_subclass__` (metaclass alternative):**
```python
class Serializable:
    _registry: dict[str, type] = {}

    def __init_subclass__(cls, type_name: str | None = None, **kwargs):
        super().__init_subclass__(**kwargs)
        name = type_name or cls.__name__.lower()
        Serializable._registry[name] = cls

    @classmethod
    def deserialize(cls, type_name: str, data: dict):
        klass = cls._registry[type_name]
        return klass(**data)

class User(Serializable, type_name="user"):
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email

class Product(Serializable, type_name="product"):
    def __init__(self, title: str, price: float):
        self.title = title
        self.price = price

# Auto-registered, no metaclass needed
user = Serializable.deserialize("user", {"name": "Alice", "email": "a@b.com"})
```

**Descriptors:**
```python
class Validated:
    """Descriptor that validates on set."""
    def __init__(self, min_value: float = 0):
        self.min_value = min_value

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, f"_{self.name}", None)

    def __set__(self, obj, value):
        if value < self.min_value:
            raise ValueError(f"{self.name} must be >= {self.min_value}")
        setattr(obj, f"_{self.name}", value)

class Order:
    quantity = Validated(min_value=1)
    price = Validated(min_value=0.01)

    def __init__(self, quantity: int, price: float):
        self.quantity = quantity  # Goes through descriptor __set__
        self.price = price
```

### Common Mistakes
1. **Forgetting `@functools.wraps`** — Losing original function name, docstring, and module
2. **Decorator changes function signature** — Caller expects original args but decorator changes behavior
3. **Metaclass when `__init_subclass__` suffices** — Over-engineering class creation
4. **Mutable state in decorators** — Shared state between decorated function calls without thread safety
5. **Too many stacked decorators** — 5+ decorators on a function; hard to debug, unclear execution order

### Integration with Other Topics
- **Type Hints** — `ParamSpec` and `TypeVar` for typed decorators
- **Design Patterns** — Decorator pattern directly implemented as Python decorators
- **Testing** — Decorators must be testable independently; mock the wrapped function
- **Web Frameworks** — Flask `@app.route`, FastAPI `@app.get` are decorator-based
- **Clean Code** — Decorators enable separation of concerns

---

## Resources

### Essential Reading
- *Fluent Python* (2nd ed.) — Chapters 9 (Decorators), 24 (Metaclasses)
- *Effective Python* — Items on decorators and metaclasses
- David Beazley — "Python 3 Metaprogramming" (YouTube)

### Online Resources
- Real Python decorator tutorials
- Python docs: Descriptor HowTo (docs.python.org)
- PEP 487 — `__init_subclass__` specification

### Practice Exercises
1. Write a `@cache_with_ttl(seconds=60)` decorator with expiration
2. Build a plugin system using `__init_subclass__` registration
3. Create a descriptor that validates type and range
4. Type a decorator with `ParamSpec` to preserve function signature
5. Replace a metaclass-based pattern with `__init_subclass__`
