# Python Concurrency

## Profile

### What It Is
Python concurrency covers threading, multiprocessing, and concurrent.futures for parallel and concurrent execution. The GIL (Global Interpreter Lock) shapes Python's concurrency model: threads work for I/O-bound tasks, processes for CPU-bound tasks. Understanding when to use each — and when `asyncio` is the better fit — is essential for performant Python applications.

### Origin and Evolution
- `threading` module — OS-level threads with GIL limiting CPU parallelism
- `multiprocessing` module (2008) — Process-based parallelism, bypasses GIL
- `concurrent.futures` (PEP 3148, 2010) — High-level executor API
- `asyncio` (2014) — Cooperative async I/O (separate topic)
- PEP 703 (2023) — Free-threaded Python (no-GIL), experimental in 3.13
- Current: `concurrent.futures` for most cases, free-threaded Python emerging

### Key Authors and References
- **David Beazley** — "Understanding the Python GIL" talk
- **Luciano Ramalho** — Concurrency chapters in *Fluent Python*
- **Sam Gross** — PEP 703, free-threaded Python (nogil)
- **Python documentation** — threading, multiprocessing, concurrent.futures

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| GIL | Global Interpreter Lock; only one thread executes Python bytecode at a time |
| Threading | Concurrent (not parallel) execution; good for I/O-bound tasks |
| Multiprocessing | True parallelism via separate processes; good for CPU-bound |
| concurrent.futures | High-level API with ThreadPoolExecutor and ProcessPoolExecutor |
| Free-threaded Python | Experimental no-GIL mode in Python 3.13+ |
| Queue | Thread-safe communication between threads/processes |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **GIL determines the strategy** — Threads for I/O (network, disk), processes for CPU (computation). Mixing them up wastes resources.
2. **`concurrent.futures` first** — Use `ThreadPoolExecutor` or `ProcessPoolExecutor` before dropping to raw `threading` or `multiprocessing`. Simpler API, fewer bugs.
3. **Shared state is the enemy** — Avoid shared mutable state between threads/processes. Use queues, immutable data, or process isolation.
4. **Prefer async for I/O concurrency** — If the workload is mostly I/O (HTTP calls, DB queries), `asyncio` is more efficient than threading. Use threads when async isn't an option.
5. **Measure before parallelizing** — Profile first. Parallelism adds complexity. Only parallelize actual bottlenecks.

### When to Use vs. When to Avoid

**Use threading when:**
- I/O-bound tasks (file reads, HTTP calls) and async isn't available
- Integrating with sync libraries that do I/O
- Simple concurrent tasks (download 10 files simultaneously)

**Use multiprocessing when:**
- CPU-bound computation (data processing, image manipulation, ML inference)
- Need true parallelism across multiple cores
- GIL is the bottleneck (measured, not assumed)

**Use asyncio instead when:**
- High-concurrency I/O (thousands of connections)
- Building web servers or clients
- Async libraries are available for your I/O

---

## Frameworks and Methodologies

### 1. Concurrent.futures Pattern — step-by-step
1. Identify the bottleneck: I/O or CPU?
2. Choose executor: `ThreadPoolExecutor` (I/O) or `ProcessPoolExecutor` (CPU)
3. Set worker count: I/O = 5-20× cores, CPU = number of cores
4. Submit tasks and collect futures
5. Handle exceptions from futures
6. Use context manager for clean shutdown

### 2. Producer-Consumer Pattern — step-by-step
1. Create a `queue.Queue` (thread-safe) or `multiprocessing.Queue`
2. Start producer threads/processes that put items
3. Start consumer threads/processes that get and process items
4. Use sentinel values for shutdown signaling
5. Handle poison pill or `queue.Empty` for graceful shutdown

---

## How to Apply

### Decision Checklist
- [ ] Is the bottleneck identified as I/O-bound or CPU-bound?
- [ ] Is `concurrent.futures` used instead of raw threading/multiprocessing?
- [ ] Is shared mutable state eliminated or properly synchronized?
- [ ] Is the worker pool size tuned (not too many, not too few)?
- [ ] Are exceptions from futures handled properly?

### Implementation Patterns

**ThreadPoolExecutor for I/O:**
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import httpx

def fetch_url(url: str) -> dict:
    response = httpx.get(url, timeout=10)
    response.raise_for_status()
    return {"url": url, "status": response.status_code, "size": len(response.content)}

urls = ["https://example.com/api/1", "https://example.com/api/2", ...]

# Process results as they complete
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(fetch_url, url): url for url in urls}
    for future in as_completed(futures):
        url = futures[future]
        try:
            result = future.result()
            print(f"{url}: {result['size']} bytes")
        except Exception as e:
            print(f"{url}: failed - {e}")
```

**ProcessPoolExecutor for CPU:**
```python
from concurrent.futures import ProcessPoolExecutor
import math

def compute_heavy(n: int) -> tuple[int, bool]:
    """CPU-intensive computation."""
    if n < 2:
        return n, False
    for i in range(2, int(math.sqrt(n)) + 1):
        if n % i == 0:
            return n, False
    return n, True

numbers = range(1_000_000, 1_001_000)

# Parallel CPU computation
with ProcessPoolExecutor() as executor:
    results = executor.map(compute_heavy, numbers, chunksize=100)
    primes = [n for n, is_prime in results if is_prime]
    print(f"Found {len(primes)} primes")
```

**Producer-Consumer with Queue:**
```python
import queue
import threading
from dataclasses import dataclass

SENTINEL = object()  # Shutdown signal

@dataclass
class WorkItem:
    url: str
    priority: int

def producer(work_queue: queue.Queue, urls: list[str]) -> None:
    for i, url in enumerate(urls):
        work_queue.put(WorkItem(url=url, priority=i))
    work_queue.put(SENTINEL)

def consumer(work_queue: queue.Queue, results: list) -> None:
    while True:
        item = work_queue.get()
        if item is SENTINEL:
            work_queue.put(SENTINEL)  # Pass to next consumer
            break
        result = process(item)
        results.append(result)
        work_queue.put(None)  # Signal task done

work_queue = queue.Queue(maxsize=100)
results = []

producer_thread = threading.Thread(target=producer, args=(work_queue, urls))
consumers = [
    threading.Thread(target=consumer, args=(work_queue, results))
    for _ in range(4)
]

producer_thread.start()
for c in consumers:
    c.start()
producer_thread.join()
for c in consumers:
    c.join()
```

**Choosing the Right Approach:**
```python
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def benchmark_io_task(n: int) -> None:
    """Simulated I/O-bound task."""
    time.sleep(0.1)

def benchmark_cpu_task(n: int) -> float:
    """CPU-bound task."""
    return sum(i * i for i in range(n))

# I/O-bound: threads win
# 10 tasks × 0.1s each = 1s sequential, ~0.1s with 10 threads
with ThreadPoolExecutor(max_workers=10) as ex:
    list(ex.map(benchmark_io_task, range(10)))

# CPU-bound: processes win
# Threads: GIL prevents parallel execution
# Processes: true parallelism across cores
with ProcessPoolExecutor() as ex:
    list(ex.map(benchmark_cpu_task, [10_000_000] * 4))
```

### Common Mistakes
1. **Threading for CPU-bound work** — GIL prevents parallel execution; use multiprocessing instead
2. **Too many workers** — 1000 threads create overhead; tune to the workload (I/O: 10-50, CPU: core count)
3. **Shared mutable state without locks** — Race conditions; use queues or immutable data
4. **Ignoring future exceptions** — `future.result()` may raise; must handle or the error is silent
5. **Not shutting down executors** — Leaked threads/processes; always use context manager

### Integration with Other Topics
- **Async Programming** — asyncio for high-concurrency I/O; threads for sync library integration
- **Performance** — Profiling determines if concurrency helps and which type
- **Testing** — Concurrent code needs race condition testing; use `ThreadPoolExecutor` with `max_workers=1` for deterministic tests
- **Data Pipelines** — Parallel processing stages for ETL
- **Web Frameworks** — WSGI servers use threading; ASGI servers use asyncio

---

## Resources

### Essential Reading
- *Fluent Python* (2nd ed.) — Concurrency chapters
- David Beazley — "Understanding the GIL" (dabeaz.com)
- Python docs — concurrent.futures module

### Online Resources
- Real Python — Threading and multiprocessing tutorials
- PEP 703 — Free-threaded CPython
- Python docs — threading and multiprocessing modules

### Practice Exercises
1. Download 50 URLs concurrently with ThreadPoolExecutor
2. Process 1000 images in parallel with ProcessPoolExecutor
3. Build a producer-consumer pipeline with Queue
4. Benchmark threading vs. multiprocessing for CPU and I/O tasks
5. Implement a rate-limited concurrent HTTP client with Semaphore
