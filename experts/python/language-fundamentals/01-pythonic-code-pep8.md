# Pythonic Code and PEP 8

## Profile

### What It Is
Pythonic code is idiomatic Python — code that leverages the language's unique features and follows community conventions. PEP 8 is the official style guide, and the Zen of Python (PEP 20) defines the philosophical underpinnings. Writing Pythonic code means using the right tool for the job: comprehensions over loops, context managers over try/finally, unpacking over indexing.

### Origin and Evolution
- Guido van Rossum's BDFL era — Python's design philosophy from the start
- PEP 8 (2001) — Style Guide for Python Code
- PEP 20 (2004) — The Zen of Python (`import this`)
- Black formatter (2018) — "uncompromising" auto-formatting
- Ruff (2023) — fast linter and formatter replacing flake8, isort, black
- Current: ruff as the de facto standard for linting and formatting

### Key Authors and References
- **Guido van Rossum** — Python creator, PEP 8 author
- **Raymond Hettinger** — "Transforming Code into Beautiful, Idiomatic Python" talk
- **Luciano Ramalho** — *Fluent Python* (2nd ed., 2022)
- **Brett Slatkin** — *Effective Python* (2nd ed., 2019)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| PEP 8 | Official style guide: naming, indentation, imports, whitespace |
| Zen of Python | 19 aphorisms guiding Python design philosophy |
| EAFP | Easier to Ask Forgiveness than Permission (try/except over check-first) |
| Comprehensions | List/dict/set comprehensions for concise collection creation |
| Unpacking | Tuple/iterable unpacking for clean variable assignment |
| Dunder Methods | Special methods (__init__, __repr__, __eq__) for Pythonic classes |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Readability counts** — Code is read far more than written. Clear, explicit code wins over clever, terse code. If it needs a comment to explain what it does, rewrite it.
2. **There should be one obvious way to do it** — Python favors one idiomatic way per pattern. Learn the idioms: comprehensions, context managers, f-strings, unpacking.
3. **EAFP over LBYL** — Try the operation and handle failure (try/except), rather than checking preconditions (if/else). This is the Pythonic way.
4. **Use the standard library** — Python's batteries-included stdlib covers most needs. Know `collections`, `itertools`, `pathlib`, `dataclasses`, `contextlib`.
5. **Flat is better than nested** — Avoid deep nesting. Use early returns, guard clauses, and decomposition to keep code flat and readable.

### When to Use vs. When to Avoid

**Be Pythonic when:**
- Writing Python professionally or contributing to Python projects
- Working with a team that values readability and maintainability
- Building libraries or tools others will consume

**Don't over-optimize for Pythonic-ness when:**
- Performance-critical inner loops where clarity matters more than idiom
- Porting code from another language temporarily (refactor later)
- The Pythonic way is less readable for your specific case (rare, but possible)

---

## Frameworks and Methodologies

### 1. Code Style Setup — step-by-step
1. Install ruff: `pip install ruff` (replaces flake8 + isort + black)
2. Configure in `pyproject.toml`: line length, rules, target Python version
3. Set up pre-commit hook for automatic formatting on commit
4. Configure editor integration (VSCode, PyCharm) for format-on-save
5. Run `ruff check --fix .` and `ruff format .` on existing codebase
6. Gradually enable stricter rules as team adapts

### 2. Pythonic Refactoring — step-by-step
1. Replace manual loops with comprehensions where appropriate
2. Replace `try/finally` with context managers
3. Replace index access with unpacking
4. Replace string concatenation with f-strings
5. Replace manual `__init__` boilerplate with `@dataclass`
6. Replace `type(x) == Foo` with `isinstance(x, Foo)`

---

## How to Apply

### Decision Checklist
- [ ] Is ruff (or black + flake8) configured and running in CI?
- [ ] Are naming conventions consistent (snake_case functions, PascalCase classes)?
- [ ] Are comprehensions used where they improve readability?
- [ ] Are context managers used for resource management?
- [ ] Is the code flat (minimal nesting)?

### Implementation Patterns

**Pythonic vs. Non-Pythonic:**
```python
# --- Non-Pythonic ---
# Manual loop to build a list
result = []
for item in items:
    if item.is_active:
        result.append(item.name)

# Index-based access
for i in range(len(items)):
    print(items[i])

# String concatenation
msg = "Hello " + name + ", you have " + str(count) + " items"

# Check before act (LBYL)
if key in dictionary:
    value = dictionary[key]
else:
    value = default

# --- Pythonic ---
# List comprehension
result = [item.name for item in items if item.is_active]

# Direct iteration
for item in items:
    print(item)

# f-string
msg = f"Hello {name}, you have {count} items"

# EAFP / use .get()
value = dictionary.get(key, default)
```

**Pythonic Classes with dataclasses:**
```python
from dataclasses import dataclass, field
from datetime import datetime

# Non-Pythonic: manual boilerplate
class Order:
    def __init__(self, id, customer, items, created_at=None):
        self.id = id
        self.customer = customer
        self.items = items
        self.created_at = created_at or datetime.now()

    def __repr__(self):
        return f"Order(id={self.id}, customer={self.customer})"

    def __eq__(self, other):
        return isinstance(other, Order) and self.id == other.id

# Pythonic: dataclass
@dataclass
class Order:
    id: str
    customer: str
    items: list[str]
    created_at: datetime = field(default_factory=datetime.now)

    @property
    def total(self) -> float:
        return sum(item.price for item in self.items)
```

**Context Managers and Unpacking:**
```python
from contextlib import contextmanager
from pathlib import Path

# Context manager for resource cleanup
@contextmanager
def temporary_directory(prefix: str):
    path = Path(tempfile.mkdtemp(prefix=prefix))
    try:
        yield path
    finally:
        shutil.rmtree(path)

# Unpacking
point = (3, 4)
x, y = point

# Star unpacking
first, *rest, last = [1, 2, 3, 4, 5]
# first=1, rest=[2,3,4], last=5

# Dict unpacking
defaults = {"timeout": 30, "retries": 3}
overrides = {"timeout": 60}
config = {**defaults, **overrides}  # timeout=60, retries=3
```

### Common Mistakes
1. **Overusing comprehensions** — Nested comprehensions with side effects; if it's hard to read, use a loop
2. **Ignoring PEP 8 naming** — `camelCase` functions, `ALL_CAPS` for regular variables; follow conventions
3. **Bare except** — `except:` or `except Exception:` catching everything; catch specific exceptions
4. **Mutable default arguments** — `def foo(items=[])` shares the list across calls; use `None` and create inside
5. **Not using pathlib** — String manipulation for paths instead of `Path` objects

### Integration with Other Topics
- **Type Hints** — Type annotations complement Pythonic code with explicit contracts
- **Code Quality Tools** — Ruff enforces PEP 8 and Pythonic patterns automatically
- **Clean Code** — Pythonic code aligns with Clean Code principles (readability, simplicity)
- **Testing** — Pythonic test code uses pytest fixtures and parametrize idiomatically
- **Project Structure** — PEP 8 import ordering and module organization

---

## Resources

### Essential Reading
- *Fluent Python* (2nd ed.) — Luciano Ramalho
- *Effective Python* (2nd ed.) — Brett Slatkin
- PEP 8 (peps.python.org/pep-0008)

### Online Resources
- The Hitchhiker's Guide to Python (docs.python-guide.org)
- Raymond Hettinger's PyCon talks (YouTube)
- Real Python style guide articles

### Practice Exercises
1. Refactor a 50-line function to use comprehensions, unpacking, and f-strings
2. Convert a class with manual __init__/__repr__/__eq__ to a dataclass
3. Configure ruff with strict rules and fix all violations in a project
4. Replace all string path manipulation with pathlib
5. Identify and fix 5 common non-Pythonic patterns in existing code
