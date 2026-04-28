# Generators, Iterators, Context Managers

## Profile

### What It Is
Generators, iterators, and context managers are core Python protocols for lazy evaluation, sequential data processing, and resource management. Generators produce values on demand (yield), iterators implement the iteration protocol (__iter__/__next__), and context managers handle setup/teardown via the `with` statement.

### Origin and Evolution
- Iterator protocol from Python 2 — `__iter__` and `__next__`
- PEP 255 (2001) — Generators with `yield`
- PEP 343 (2005) — `with` statement and context managers
- PEP 342 (2005) — Coroutines via enhanced generators (send, throw)
- PEP 380 (2012) — `yield from` for generator delegation
- `contextlib` module — utilities for creating context managers
- Current: generators underpin async/await; context managers everywhere

### Key Authors and References
- **David Beazley** — "Generator Tricks for Systems Programmers", "A Curious Course on Coroutines"
- **Luciano Ramalho** — Iterator and generator chapters in *Fluent Python*
- **Raymond Hettinger** — itertools patterns and talks

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Iterator Protocol | `__iter__()` returns self, `__next__()` returns next value or raises `StopIteration` |
| Generator Function | Function with `yield`; returns a generator iterator |
| Generator Expression | `(x for x in items)` — lazy comprehension |
| `yield from` | Delegates to a sub-generator |
| Context Manager | `__enter__`/`__exit__` protocol for `with` statement |
| `contextlib` | `@contextmanager`, `ExitStack`, `suppress`, `closing` |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Lazy is efficient** — Generate values on demand, not all at once. Generators process billion-row files with constant memory.
2. **`with` for every resource** — Files, connections, locks, transactions — anything that needs cleanup belongs in a `with` statement.
3. **`yield from` for composition** — Compose generators by delegating to sub-generators. Clean pipeline of transformations.
4. **itertools is your toolkit** — `chain`, `islice`, `groupby`, `product`, `combinations` — learn itertools before writing custom iteration.
5. **Generators replace temporary lists** — If you build a list just to iterate it once, use a generator instead.

### When to Use vs. When to Avoid

**Use generators when:**
- Processing large datasets that don't fit in memory
- Building data processing pipelines (transform → filter → aggregate)
- Producing sequences lazily (infinite sequences, expensive computations)

**Use context managers when:**
- Any resource needs deterministic cleanup (files, connections, locks)
- Setup/teardown pairs (enter/exit, begin/commit, acquire/release)
- Temporary state changes (change directory, set environment variable)

**Use regular lists/functions when:**
- The data is small and you need random access
- You need to iterate multiple times
- Simplicity matters more than memory efficiency

---

## Frameworks and Methodologies

### 1. Generator Pipeline Design — step-by-step
1. Identify the data source (file, database, API)
2. Create a generator for reading (yield each record)
3. Chain transformation generators (parse, filter, transform)
4. End with a consumer (aggregate, write, count)
5. Each stage processes one item at a time — constant memory
6. Use `yield from` to compose sub-pipelines

### 2. Context Manager Design — step-by-step
1. Identify the resource that needs cleanup
2. Decide: class-based (`__enter__`/`__exit__`) or `@contextmanager`
3. Use `@contextmanager` for simple cases (less boilerplate)
4. Use class-based for reusable, complex managers
5. Handle exceptions in `__exit__` (return True to suppress)
6. Use `ExitStack` for dynamic number of context managers

---

## How to Apply

### Decision Checklist
- [ ] Are generators used for large or streaming data processing?
- [ ] Are context managers used for all resource management?
- [ ] Is `itertools` used instead of reinventing iteration patterns?
- [ ] Are generator pipelines used instead of intermediate lists?
- [ ] Is `@contextmanager` used for simple setup/teardown?

### Implementation Patterns

**Generator Pipeline:**
```python
from pathlib import Path
import csv

# Stage 1: Read
def read_csv(path: Path):
    with open(path) as f:
        reader = csv.DictReader(f)
        yield from reader

# Stage 2: Filter
def active_users(records):
    for record in records:
        if record["status"] == "active":
            yield record

# Stage 3: Transform
def normalize(records):
    for record in records:
        yield {
            "name": record["name"].strip().title(),
            "email": record["email"].strip().lower(),
            "age": int(record["age"]),
        }

# Pipeline: constant memory regardless of file size
pipeline = normalize(active_users(read_csv(Path("users.csv"))))
for user in pipeline:
    process(user)
```

**Context Managers:**
```python
from contextlib import contextmanager, ExitStack
import time

# Simple: @contextmanager
@contextmanager
def timer(label: str):
    start = time.perf_counter()
    try:
        yield
    finally:
        elapsed = time.perf_counter() - start
        print(f"{label}: {elapsed:.3f}s")

with timer("database query"):
    results = db.query("SELECT ...")

# Temporary state change
@contextmanager
def temporary_env(key: str, value: str):
    old = os.environ.get(key)
    os.environ[key] = value
    try:
        yield
    finally:
        if old is None:
            del os.environ[key]
        else:
            os.environ[key] = old

# Class-based for complex logic
class DatabaseTransaction:
    def __init__(self, connection):
        self.connection = connection

    def __enter__(self):
        self.connection.begin()
        return self.connection

    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type is None:
            self.connection.commit()
        else:
            self.connection.rollback()
        return False  # Don't suppress exceptions

# ExitStack for dynamic cleanup
def process_files(paths: list[Path]):
    with ExitStack() as stack:
        files = [stack.enter_context(open(p)) for p in paths]
        for f in files:
            process(f)
```

**itertools Patterns:**
```python
from itertools import chain, islice, groupby, batched

# Chain multiple iterables
all_items = chain(list_a, list_b, generator_c)

# Take first N items from any iterable
first_10 = list(islice(huge_generator(), 10))

# Group sorted data
data = sorted(records, key=lambda r: r["category"])
for category, group in groupby(data, key=lambda r: r["category"]):
    items = list(group)
    print(f"{category}: {len(items)} items")

# Batch processing (Python 3.12+)
for batch in batched(records, 100):
    db.insert_many(batch)

# Pre-3.12 batching
def batched_legacy(iterable, n):
    it = iter(iterable)
    while batch := list(islice(it, n)):
        yield batch
```

### Common Mistakes
1. **Consuming a generator twice** — Generators are single-use; iterating again yields nothing
2. **Not closing generator resources** — Generator with `open()` inside needs `.close()` or `with` statement
3. **Forgetting `yield from`** — Writing a loop to yield from sub-generator instead of `yield from`
4. **Exception handling in `__exit__`** — Returning `True` accidentally suppresses exceptions
5. **Building lists from generators prematurely** — `list(generator)` defeats the purpose of lazy evaluation

### Integration with Other Topics
- **Async Programming** — `async for`, `async with` are async versions of these protocols
- **Data Pipelines** — Generator pipelines are in-process data pipelines
- **Testing** — Generators as test data factories; context managers for test fixtures
- **Pythonic Code** — Generators and context managers are core Python idioms
- **Performance** — Generators enable constant-memory processing of large data

---

## Resources

### Essential Reading
- *Fluent Python* (2nd ed.) — Chapters on iterables, generators, context managers
- David Beazley — "Generator Tricks for Systems Programmers" (dabeaz.com)
- Python docs: itertools module

### Online Resources
- Python docs: contextlib module
- Real Python: generators and iterators tutorials
- itertools recipes in Python documentation

### Practice Exercises
1. Build a log file processor that handles GB-sized files with generators
2. Create a context manager for database transactions with commit/rollback
3. Implement a pipeline: read CSV → filter → transform → batch insert
4. Use `ExitStack` to manage a dynamic number of file handles
5. Replace 3 intermediate lists with generator pipeline in existing code
