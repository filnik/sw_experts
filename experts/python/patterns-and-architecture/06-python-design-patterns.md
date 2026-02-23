# Python Design Patterns

## Profile

### What It Is
Design patterns adapted to Python's dynamic nature — leveraging first-class functions, duck typing, protocols, and the descriptor protocol. Many GoF patterns that require complex class hierarchies in Java/C++ collapse into simpler, more Pythonic implementations using closures, decorators, or module-level functions.

### Origin and Evolution
- GoF *Design Patterns* (1994) — original 23 patterns in C++/Smalltalk
- Python adaptations — many patterns simplified by first-class functions
- Peter Norvig (1998) — "Design Patterns in Dynamic Languages" (16 of 23 simplified)
- Brandon Rhodes — pythonpatterns.guide — Python-specific pattern reference
- Current: Protocol-based patterns, dataclass-based value objects

### Key Authors and References
- **Luciano Ramalho** — *Fluent Python* pattern chapters
- **Brandon Rhodes** — python-patterns.guide
- **Peter Norvig** — Dynamic language pattern simplification
- **Harry Percival & Bob Gregory** — *Architecture Patterns with Python*

### Key Concepts at a Glance
| Pattern | Python Approach |
|---------|-----------------|
| Strategy | First-class functions or callables |
| Observer | Callbacks, signals, or weakref |
| Factory | Functions, `__init_subclass__`, or class methods |
| Decorator | `@decorator` syntax (language feature) |
| Iterator | Generator functions (`yield`) |
| Template Method | Functions with callable parameters |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Functions replace simple classes** — In Python, a function is often a better Strategy, Command, or Factory than a class hierarchy.
2. **Duck typing is the default** — Protocol (structural subtyping) replaces interface-heavy patterns. If it quacks, it's a duck.
3. **Modules are singletons** — Python modules are loaded once. Module-level state is the Singleton pattern without the pattern.
4. **Closures replace object state** — A closure captures state. Many patterns that use objects for state work better as closures in Python.
5. **Don't pattern-match blindly** — Apply patterns when they solve a real problem. Python's flexibility means you often don't need the pattern at all.

### When to Use vs. When to Avoid

**Use patterns when:**
- The problem genuinely benefits from the structure
- The codebase is large enough that patterns improve navigation
- Multiple developers need a shared vocabulary

**Keep it simple when:**
- A plain function solves the problem
- The pattern adds more code than it saves
- You're naming a pattern you don't need

---

## Frameworks and Methodologies

### 1. Pattern Selection — step-by-step
1. Identify the problem: creation, behavior, or structure?
2. Check if Python has a built-in solution (generators for Iterator, decorators for Decorator)
3. Consider function-based approach first
4. Use class-based only if state management requires it
5. Use Protocol for type-safe duck typing

### 2. Refactoring to Patterns — step-by-step
1. Spot the code smell (switch/if chains, duplicated setup, tight coupling)
2. Identify the pattern that addresses it
3. Implement the Pythonic version (functions, protocols, closures)
4. Verify the refactoring with tests
5. Document why the pattern was chosen (not just what)

---

## How to Apply

### Decision Checklist
- [ ] Is the pattern simpler than the code it replaces?
- [ ] Is a function-based approach considered before class-based?
- [ ] Is Protocol used instead of ABC where structural typing suffices?
- [ ] Is the pattern solving a real problem, not theoretical?

### Implementation Patterns

**Strategy Pattern (function-based):**
```python
from collections.abc import Callable
from decimal import Decimal

# Strategies are just functions
def flat_discount(total: Decimal) -> Decimal:
    return total * Decimal("0.10")

def tiered_discount(total: Decimal) -> Decimal:
    if total > Decimal("1000"):
        return total * Decimal("0.15")
    elif total > Decimal("500"):
        return total * Decimal("0.10")
    return Decimal("0")

def loyalty_discount(loyalty_years: int):
    def discount(total: Decimal) -> Decimal:
        rate = min(loyalty_years * 2, 20) / 100
        return total * Decimal(str(rate))
    return discount

# Usage: pass function as strategy
def calculate_total(
    subtotal: Decimal,
    discount_strategy: Callable[[Decimal], Decimal],
) -> Decimal:
    discount = discount_strategy(subtotal)
    return subtotal - discount

total = calculate_total(Decimal("800"), tiered_discount)
total = calculate_total(Decimal("800"), loyalty_discount(5))
```

**Repository Pattern:**
```python
from typing import Protocol, TypeVar, Generic

T = TypeVar("T")

class Repository(Protocol[T]):
    def get(self, id: str) -> T | None: ...
    def save(self, entity: T) -> None: ...
    def delete(self, id: str) -> None: ...
    def list(self) -> list[T]: ...

class InMemoryRepository(Generic[T]):
    def __init__(self) -> None:
        self._store: dict[str, T] = {}

    def get(self, id: str) -> T | None:
        return self._store.get(id)

    def save(self, entity: T) -> None:
        self._store[entity.id] = entity

    def delete(self, id: str) -> None:
        self._store.pop(id, None)

    def list(self) -> list[T]:
        return list(self._store.values())

class PostgresRepository:
    def __init__(self, session: Session) -> None:
        self._session = session

    def get(self, id: str) -> User | None:
        return self._session.query(User).get(id)
    # ... implements same interface, satisfies Protocol
```

**Observer Pattern (callback-based):**
```python
from collections import defaultdict
from collections.abc import Callable
from typing import Any

class EventEmitter:
    def __init__(self) -> None:
        self._listeners: dict[str, list[Callable]] = defaultdict(list)

    def on(self, event: str, callback: Callable) -> None:
        self._listeners[event].append(callback)

    def emit(self, event: str, **data: Any) -> None:
        for callback in self._listeners[event]:
            callback(**data)

# Usage
events = EventEmitter()
events.on("order.created", lambda order_id, **_: send_confirmation(order_id))
events.on("order.created", lambda order_id, **_: update_inventory(order_id))
events.emit("order.created", order_id="123", total=99.99)
```

**Factory with Registry:**
```python
from dataclasses import dataclass

class Notification:
    _registry: dict[str, type["Notification"]] = {}

    def __init_subclass__(cls, channel: str = "", **kwargs):
        super().__init_subclass__(**kwargs)
        if channel:
            Notification._registry[channel] = cls

    @classmethod
    def create(cls, channel: str, **kwargs) -> "Notification":
        klass = cls._registry[channel]
        return klass(**kwargs)

    def send(self, message: str) -> None:
        raise NotImplementedError

class EmailNotification(Notification, channel="email"):
    def __init__(self, to: str):
        self.to = to

    def send(self, message: str) -> None:
        print(f"Email to {self.to}: {message}")

class SlackNotification(Notification, channel="slack"):
    def __init__(self, channel_name: str):
        self.channel_name = channel_name

    def send(self, message: str) -> None:
        print(f"Slack #{self.channel_name}: {message}")

# Factory usage
notif = Notification.create("email", to="user@example.com")
notif.send("Hello!")
```

### Common Mistakes
1. **Class for everything** — Creating a Strategy class when a function works; Java patterns in Python
2. **Singleton pattern** — Using Singleton class instead of module-level instance or dependency injection
3. **Abstract base class overuse** — ABC with one implementation; use Protocol for duck typing
4. **Pattern naming without purpose** — Naming something "Factory" or "Strategy" when it's just a function call
5. **Ignoring Python built-ins** — Reimplementing Iterator (use generators), Decorator (use `@decorator`)

### Integration with Other Topics
- **Clean Architecture** — Patterns structure the layers of clean architecture
- **Decorators** — Python's decorator syntax is the Decorator pattern
- **Type Hints** — Protocol enables typed duck typing for pattern interfaces
- **Testing** — Patterns enable testability through substitution (Strategy, Repository)
- **DDD** — Repository, Factory, Value Object patterns from DDD

---

## Resources

### Essential Reading
- *Fluent Python* (2nd ed.) — Design pattern chapters
- python-patterns.guide — Brandon Rhodes
- *Architecture Patterns with Python* — Percival & Gregory

### Online Resources
- refactoring.guru — Patterns with Python examples
- Peter Norvig — "Design Patterns in Dynamic Languages" (slides)
- Real Python — Design patterns tutorials

### Practice Exercises
1. Replace an if/elif chain with Strategy pattern using functions
2. Implement a Repository with Protocol and two implementations
3. Build an event system with EventEmitter and typed callbacks
4. Convert a Singleton class to a module-level instance
5. Implement Factory with `__init_subclass__` registration
