# Python Performance

## Profile

### What It Is
Python performance optimization covers profiling, benchmarking, caching, algorithmic improvements, C extensions, and alternative runtimes. Python is often "fast enough," but when it isn't, understanding where time is spent and choosing the right optimization strategy is critical.

### Origin and Evolution
- CPython (1991) — Reference implementation, interpreted
- Cython (2007) — Python-to-C compiler for performance-critical code
- PyPy (2007) — JIT-compiled Python implementation
- `cProfile` and `timeit` — Standard library profiling tools
- `scalene` (2020) — CPU + memory + GPU profiler
- Cython 3.0 (2023) — Modern Cython with Python 3 features
- Current: profiling-first approach, caching, async I/O, Rust extensions (PyO3)

### Key Authors and References
- **Micha Gorelick & Ian Ozsvald** — *High Performance Python* (2nd ed.)
- **Raymond Hettinger** — Performance-focused PyCon talks
- **Carl Friedrich Bolz-Tereick** — PyPy developer
- **David Beazley** — Low-level Python performance analysis

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Profiling | Measuring where time is spent (cProfile, scalene) |
| Caching | `@functools.cache`, `@lru_cache`, Redis for computed results |
| Algorithmic | O(n²) → O(n log n) often beats micro-optimization |
| C Extensions | Cython, PyO3 (Rust), ctypes for performance-critical code |
| Alternative Runtimes | PyPy (JIT), GraalPy, Cinder (Meta's CPython fork) |
| Data Structures | `deque`, `defaultdict`, `set` lookups, `array.array` |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Profile first, optimize second** — Never optimize without profiling. Developers are bad at guessing bottlenecks. Use cProfile or scalene to find the actual hot spots.
2. **Algorithm beats micro-optimization** — Changing from O(n²) to O(n log n) gives more than any micro-optimization. Fix the algorithm before reaching for Cython.
3. **Cache expensive operations** — `@functools.cache` for pure functions, Redis for shared cache. The fastest computation is the one you don't repeat.
4. **Use the right data structure** — `set` for membership testing (O(1) vs O(n) for list), `deque` for queue operations, `dict` for lookups.
5. **Python-level optimization first** — Optimize Python code before rewriting in C/Rust. Comprehensions over loops, built-in functions, avoiding unnecessary allocations.

### When to Use vs. When to Avoid

**Optimize when:**
- Profiling shows a clear bottleneck
- Response times or throughput don't meet requirements
- Processing time is growing faster than expected (algorithmic)

**Don't optimize when:**
- No measured performance problem exists
- Code clarity would be significantly reduced
- The bottleneck is I/O (optimize I/O, not CPU code)

---

## Frameworks and Methodologies

### 1. Performance Investigation — step-by-step
1. Define the performance requirement (latency, throughput, memory)
2. Create a benchmark that measures the metric
3. Profile with scalene or cProfile to find hot spots
4. Identify the bottleneck type: CPU, I/O, memory, or algorithm
5. Apply appropriate optimization
6. Re-measure to verify improvement
7. Document the optimization and its trade-offs

### 2. Optimization Strategy — step-by-step
1. Fix algorithmic complexity first (O(n²) → O(n))
2. Use appropriate data structures (set, dict, deque)
3. Apply caching for repeated computations
4. Use built-in functions and comprehensions
5. Consider async for I/O-bound bottlenecks
6. Consider multiprocessing for CPU-bound bottlenecks
7. Last resort: C extension (Cython, Rust via PyO3)

---

## How to Apply

### Decision Checklist
- [ ] Is the bottleneck identified through profiling (not guessing)?
- [ ] Is algorithmic complexity appropriate for the data size?
- [ ] Are caching opportunities exploited?
- [ ] Are appropriate data structures used (set, dict, deque)?
- [ ] Is the optimization measured with benchmarks?

### Implementation Patterns

**Profiling:**
```python
# cProfile — function-level profiling
import cProfile

cProfile.run("process_orders(orders)", sort="cumulative")

# As context manager
from contextlib import contextmanager
import time

@contextmanager
def timer(label: str):
    start = time.perf_counter()
    yield
    elapsed = time.perf_counter() - start
    print(f"{label}: {elapsed:.3f}s")

with timer("order processing"):
    process_orders(orders)

# scalene (install: pip install scalene)
# Run: scalene my_script.py
# Shows CPU time, memory allocation, and GPU usage per line

# timeit for micro-benchmarks
import timeit
result = timeit.timeit(
    "sum(range(1000))",
    number=10000,
)
print(f"{result:.3f}s for 10000 iterations")
```

**Caching:**
```python
from functools import cache, lru_cache

# Unbounded cache (all results kept)
@cache
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# LRU cache (bounded, evicts least recently used)
@lru_cache(maxsize=256)
def get_user_permissions(user_id: str) -> set[str]:
    return db.query_permissions(user_id)

# Manual cache with TTL
from time import time

class TTLCache:
    def __init__(self, ttl: float = 300):
        self._cache: dict[str, tuple[float, any]] = {}
        self._ttl = ttl

    def get(self, key: str):
        if key in self._cache:
            expires, value = self._cache[key]
            if time() < expires:
                return value
            del self._cache[key]
        return None

    def set(self, key: str, value):
        self._cache[key] = (time() + self._ttl, value)
```

**Data Structure Optimization:**
```python
# List membership: O(n) → Set membership: O(1)
# BAD
valid_ids = [1, 2, 3, ..., 10000]
if item_id in valid_ids:  # O(n) scan
    ...

# GOOD
valid_ids = {1, 2, 3, ..., 10000}
if item_id in valid_ids:  # O(1) hash lookup
    ...

# String concatenation: O(n²) → join: O(n)
# BAD
result = ""
for item in items:
    result += str(item)  # Creates new string each time

# GOOD
result = "".join(str(item) for item in items)

# collections for specialized needs
from collections import deque, defaultdict, Counter

# deque for O(1) append/pop from both ends
queue = deque(maxlen=1000)
queue.append(item)      # O(1)
queue.popleft()          # O(1) — list.pop(0) is O(n)

# defaultdict to avoid key checking
word_count = defaultdict(int)
for word in words:
    word_count[word] += 1  # No KeyError check needed

# Counter for counting
top_10 = Counter(words).most_common(10)
```

**Algorithmic Improvements:**
```python
# Finding duplicates: O(n²) → O(n)
# BAD: nested loops
duplicates = []
for i in range(len(items)):
    for j in range(i + 1, len(items)):
        if items[i] == items[j]:
            duplicates.append(items[i])

# GOOD: set-based
seen = set()
duplicates = set()
for item in items:
    if item in seen:
        duplicates.add(item)
    seen.add(item)

# Sorted search: O(n) → O(log n)
from bisect import bisect_left

def binary_search(sorted_list: list, target) -> int | None:
    i = bisect_left(sorted_list, target)
    if i < len(sorted_list) and sorted_list[i] == target:
        return i
    return None
```

**C/Rust Extensions:**
```python
# Cython — compile Python to C
# fibonacci.pyx
def fibonacci_cy(int n):
    cdef int a = 0, b = 1, i
    for i in range(n):
        a, b = b, a + b
    return a

# PyO3 — Rust extension for Python
# Cargo.toml + lib.rs
# use pyo3::prelude::*;
#
# #[pyfunction]
# fn fibonacci(n: u64) -> u64 {
#     let (mut a, mut b) = (0u64, 1u64);
#     for _ in 0..n {
#         let temp = b;
#         b = a + b;
#         a = temp;
#     }
#     a
# }

# Using C extensions from Python
import my_rust_module
result = my_rust_module.fibonacci(1000)
```

### Common Mistakes
1. **Premature optimization** — Optimizing code that isn't a bottleneck; profile first
2. **Wrong data structure** — Using list for membership checks instead of set
3. **String concatenation in loops** — `+=` creates new string each time; use `join()`
4. **Not caching repeated computations** — Computing the same result multiple times
5. **Micro-optimizing instead of algorithm** — Shaving microseconds off O(n²) instead of making it O(n)

### Integration with Other Topics
- **Concurrency** — Multiprocessing for CPU-bound, async for I/O-bound bottlenecks
- **Caching Strategies** — Application-level caching (functools, Redis)
- **Data Structures** — Choosing the right structure for the access pattern
- **Testing** — Benchmark tests to prevent performance regressions
- **Profiling** — scalene, cProfile for identifying bottlenecks

---

## Resources

### Essential Reading
- *High Performance Python* (2nd ed.) — Gorelick & Ozsvald
- *Fluent Python* (2nd ed.) — Performance-related chapters
- Raymond Hettinger — "Beyond PEP 8" (YouTube)

### Online Resources
- scalene documentation (github.com/plasma-umass/scalene)
- PyO3 documentation (pyo3.rs) — Rust extensions for Python
- Python Speed tips (wiki.python.org/moin/PythonSpeed)

### Practice Exercises
1. Profile a slow function with scalene and identify the bottleneck
2. Replace list membership checks with set and measure improvement
3. Add `@lru_cache` to a recursive function and benchmark
4. Refactor O(n²) code to O(n) with appropriate data structures
5. Write a Cython extension for a compute-intensive function and benchmark
