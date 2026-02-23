# Type Hints and Static Typing

## Profile

### What It Is
Python's type hint system (PEP 484+) adds optional static type annotations to Python code. Combined with type checkers like mypy or pyright, it catches bugs before runtime, improves IDE support, and serves as executable documentation — all while keeping Python's dynamic nature intact.

### Origin and Evolution
- PEP 3107 (2006) — Function annotations syntax
- PEP 484 (2014) — Type hints specification, mypy
- PEP 526 (2016) — Variable annotations
- PEP 544 (2017) — Structural subtyping (Protocol)
- PEP 604 (2020) — `X | Y` union syntax
- PEP 612 (2020) — ParamSpec for decorator typing
- PEP 695 (2023, Python 3.12) — `type` statement, simplified generics syntax
- Current: pyright (Microsoft) gaining adoption, full generics in 3.12+

### Key Authors and References
- **Guido van Rossum** — PEP 484, mypy origins
- **Jukka Lehtosalo** — mypy creator
- **Eric Traut** — pyright/Pylance creator (Microsoft)
- **Luciano Ramalho** — Type hints chapters in *Fluent Python* (2nd ed.)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Type Annotations | Syntax: `def foo(x: int) -> str` |
| Generic Types | `list[int]`, `dict[str, Any]`, custom `T = TypeVar("T")` |
| Protocol | Structural subtyping (duck typing with type safety) |
| Union / Optional | `int \| str`, `Optional[int]` = `int \| None` |
| TypeVar | Generic type variable for polymorphic functions |
| TypeGuard | Narrow types in conditional branches |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Gradual typing** — Add types incrementally. You don't need 100% coverage. Start with public APIs and critical paths.
2. **Types as documentation** — Type annotations are documentation that the type checker validates. They never go stale.
3. **Protocols over inheritance** — Use `Protocol` for structural subtyping (duck typing). If it has `.read()`, it's a file-like object.
4. **Strict mode for new code** — Enable strict mode in mypy/pyright for new projects. Retrofit gradually for existing code.
5. **Runtime types for boundaries** — Type hints are not enforced at runtime. Use Pydantic or similar for runtime validation at system boundaries (APIs, config).

### When to Use vs. When to Avoid

**Add type hints when:**
- Public APIs of libraries and packages
- Complex data transformations or business logic
- Team projects where multiple developers touch the code
- Code that handles multiple types (unions, generics)

**Skip or simplify when:**
- Quick scripts or one-off automation
- Prototyping where types are changing rapidly
- When type annotations make the code less readable (rare)

---

## Frameworks and Methodologies

### 1. Type Checking Setup — step-by-step
1. Choose type checker: mypy (mature) or pyright (fast, strict)
2. Add to `pyproject.toml` configuration
3. Start with permissive settings, gradually increase strictness
4. Add to CI pipeline (fail on type errors)
5. Configure editor integration (VSCode + Pylance, PyCharm built-in)
6. Add `py.typed` marker for library packages

### 2. Gradual Typing Adoption — step-by-step
1. Type all public function signatures first
2. Add return type annotations
3. Type class attributes and dataclass fields
4. Add generic types where needed
5. Enable `--strict` for new modules
6. Use `# type: ignore` sparingly with comments explaining why

---

## How to Apply

### Decision Checklist
- [ ] Is a type checker configured and running in CI?
- [ ] Are public APIs fully typed?
- [ ] Are Protocol classes used for duck typing patterns?
- [ ] Are generic types used where functions work with multiple types?
- [ ] Is strict mode enabled for new code?

### Implementation Patterns

**Basic Type Annotations (Python 3.10+):**
```python
from datetime import datetime
from pathlib import Path

# Function signatures
def process_order(order_id: str, amount: float) -> bool:
    ...

# Variables (usually inferred, annotate when not obvious)
items: list[str] = []
config: dict[str, int] = {"timeout": 30}

# Union types (3.10+ syntax)
def parse_input(value: str | int) -> str:
    return str(value)

# Optional (value or None)
def find_user(user_id: str) -> User | None:
    ...
```

**Generics:**
```python
from typing import TypeVar, Generic
from collections.abc import Sequence, Callable

T = TypeVar("T")
K = TypeVar("K")
V = TypeVar("V")

# Generic function
def first(items: Sequence[T]) -> T | None:
    return items[0] if items else None

# Python 3.12 syntax
def first[T](items: Sequence[T]) -> T | None:
    return items[0] if items else None

# Generic class
class Repository(Generic[T]):
    def __init__(self, model_class: type[T]) -> None:
        self._model = model_class
        self._items: dict[str, T] = {}

    def get(self, id: str) -> T | None:
        return self._items.get(id)

    def save(self, id: str, item: T) -> None:
        self._items[id] = item

# Usage: Repository[User] — type checker knows get() returns User | None
user_repo = Repository(User)
user: User | None = user_repo.get("123")
```

**Protocol (Structural Subtyping):**
```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Renderable(Protocol):
    def render(self) -> str: ...

class MarkdownDoc:
    def render(self) -> str:
        return "# Hello"

class HTMLDoc:
    def render(self) -> str:
        return "<h1>Hello</h1>"

# Both work — no inheritance needed, just matching interface
def display(item: Renderable) -> None:
    print(item.render())

display(MarkdownDoc())  # OK
display(HTMLDoc())      # OK
```

**Callable and ParamSpec:**
```python
from typing import ParamSpec, Callable, TypeVar
from functools import wraps
import time

P = ParamSpec("P")
R = TypeVar("R")

def timed(func: Callable[P, R]) -> Callable[P, R]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.2f}s")
        return result
    return wrapper

@timed
def fetch_data(url: str, timeout: int = 30) -> dict:
    ...

# Type checker knows fetch_data signature is preserved
```

### Common Mistakes
1. **Using `Any` everywhere** — Defeats the purpose of typing; use specific types or generics
2. **Not running the type checker** — Annotations without checking are just comments that can lie
3. **Inheritance for protocols** — Using ABC when Protocol is more Pythonic (structural vs. nominal)
4. **Ignoring `None` returns** — Forgetting to annotate `-> X | None` for functions that can return None
5. **Confusing runtime and static** — Type hints are not enforced at runtime; use Pydantic for validation

### Integration with Other Topics
- **Pythonic Code** — Type hints are Pythonic when they improve readability
- **Pydantic** — Runtime validation using type annotations
- **Testing** — Typed code reduces certain test categories (type error tests)
- **Design Patterns** — Protocol enables clean Strategy, Observer patterns
- **Clean Architecture** — Typed interfaces define boundaries between layers

---

## Resources

### Essential Reading
- *Fluent Python* (2nd ed.) — Chapters on type hints
- *Robust Python* — Patrick Viafore
- mypy documentation (mypy.readthedocs.io)

### Online Resources
- Python typing documentation (docs.python.org/3/library/typing.html)
- pyright documentation (github.com/microsoft/pyright)
- typing PEPs index (peps.python.org)

### Practice Exercises
1. Add type annotations to an existing module and run mypy in strict mode
2. Create a generic Repository class with proper TypeVar usage
3. Replace an ABC with a Protocol and verify structural subtyping
4. Type a decorator that preserves function signatures using ParamSpec
5. Configure pyright in strict mode and resolve all type errors
