# Async Programming

## Profile

### What It Is
Python's async programming model uses `async`/`await` syntax on top of the `asyncio` event loop to handle concurrent I/O-bound operations efficiently. A single thread manages thousands of concurrent connections, API calls, or file operations by cooperatively switching between coroutines when they await I/O.

### Origin and Evolution
- Twisted (2002) — Callback-based async framework
- Tornado (2009) — Async web framework
- PEP 3156 (2012) — asyncio module introduced
- PEP 492 (2015) — `async`/`await` syntax (Python 3.5)
- PEP 525 (2016) — Async generators
- PEP 530 (2016) — Async comprehensions
- uvloop (2016) — Drop-in faster event loop
- Current: `asyncio.TaskGroup` (3.11), exception groups, anyio for framework-agnostic async

### Key Authors and References
- **Guido van Rossum** — asyncio design
- **Yury Selivanov** — async/await PEPs, uvloop, EdgeDB
- **Nathaniel Smith** — Trio framework, structured concurrency advocacy
- **David Beazley** — "Build Your Own Async" live coding talks

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Coroutine | `async def` function; must be awaited |
| Event Loop | Scheduler that runs coroutines and manages I/O |
| `await` | Suspends coroutine until the awaited operation completes |
| `asyncio.gather` | Run multiple coroutines concurrently |
| `TaskGroup` | Structured concurrency (Python 3.11+); cancels on failure |
| `async with` / `async for` | Async context managers and iterators |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Async is for I/O, not CPU** — Use async for network calls, file I/O, database queries. For CPU-bound work, use `multiprocessing` or `concurrent.futures`.
2. **Structured concurrency** — Use `TaskGroup` (3.11+) instead of bare `create_task`. Tasks have a clear lifetime and errors propagate properly.
3. **Don't block the event loop** — Never call synchronous blocking functions (requests, time.sleep) in async code. Use async libraries (httpx, asyncio.sleep).
4. **Async all the way** — Once you go async, the entire call chain must be async. You can't easily mix sync and async without `asyncio.to_thread`.
5. **Concurrency is not parallelism** — Async runs concurrent tasks on one thread. It interleaves I/O waits, not CPU computation.

### When to Use vs. When to Avoid

**Use async when:**
- Many concurrent I/O operations (HTTP clients, WebSockets, DB queries)
- Building web servers handling many connections (FastAPI, aiohttp)
- Event-driven systems (chat servers, real-time feeds)
- Orchestrating multiple API calls concurrently

**Use sync/threading when:**
- CPU-bound computation (use multiprocessing)
- Simple scripts with sequential I/O
- Library ecosystem is sync-only
- Team is not comfortable with async patterns

---

## Frameworks and Methodologies

### 1. Async Application Design — step-by-step
1. Identify I/O-bound bottlenecks (HTTP calls, DB queries, file reads)
2. Choose async framework: FastAPI (web), aiohttp (HTTP client), asyncpg (PostgreSQL)
3. Make entrypoint async: `asyncio.run(main())`
4. Convert I/O operations to async equivalents
5. Use `TaskGroup` for concurrent operations
6. Use `asyncio.to_thread` for unavoidable sync calls
7. Add proper error handling and cancellation support

### 2. Async Testing Setup — step-by-step
1. Install `pytest-asyncio`
2. Mark async tests with `@pytest.mark.asyncio`
3. Create async fixtures with `@pytest_asyncio.fixture`
4. Mock async functions with `AsyncMock`
5. Use `anyio` for framework-agnostic tests

---

## How to Apply

### Decision Checklist
- [ ] Are all I/O operations using async libraries (not blocking)?
- [ ] Is `TaskGroup` used for concurrent task management?
- [ ] Is `asyncio.to_thread` used for unavoidable sync operations?
- [ ] Are async context managers used for connections/sessions?
- [ ] Is error handling and cancellation properly implemented?

### Implementation Patterns

**Basic Async Patterns:**
```python
import asyncio
import httpx

# Simple coroutine
async def fetch_user(client: httpx.AsyncClient, user_id: str) -> dict:
    response = await client.get(f"/users/{user_id}")
    response.raise_for_status()
    return response.json()

# Concurrent execution with TaskGroup (Python 3.11+)
async def fetch_all_users(user_ids: list[str]) -> list[dict]:
    results = []
    async with httpx.AsyncClient(base_url="https://api.example.com") as client:
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(fetch_user(client, uid)) for uid in user_ids]
    return [task.result() for task in tasks]

# Entry point
async def main():
    users = await fetch_all_users(["u1", "u2", "u3"])
    for user in users:
        print(user["name"])

asyncio.run(main())
```

**Async with Rate Limiting:**
```python
import asyncio

class RateLimiter:
    def __init__(self, max_concurrent: int):
        self._semaphore = asyncio.Semaphore(max_concurrent)

    async def __aenter__(self):
        await self._semaphore.acquire()
        return self

    async def __aexit__(self, *args):
        self._semaphore.release()

async def fetch_with_limit(
    urls: list[str],
    max_concurrent: int = 10,
) -> list[dict]:
    limiter = RateLimiter(max_concurrent)
    async with httpx.AsyncClient() as client:
        async def fetch_one(url: str) -> dict:
            async with limiter:
                response = await client.get(url)
                return response.json()

        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(fetch_one(url)) for url in urls]
    return [t.result() for t in tasks]
```

**Async Generator and Streaming:**
```python
from collections.abc import AsyncIterator

async def stream_records(query: str) -> AsyncIterator[dict]:
    """Async generator for streaming database results."""
    async with get_connection() as conn:
        async with conn.cursor() as cursor:
            await cursor.execute(query)
            async for row in cursor:
                yield dict(row)

# Consume with async for
async def process_all():
    async for record in stream_records("SELECT * FROM orders"):
        await process_record(record)

# Async comprehension
async def get_names():
    return [
        record["name"]
        async for record in stream_records("SELECT name FROM users")
        if record["name"]
    ]
```

**Mixing Sync and Async:**
```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

# Run sync blocking function in thread
async def read_file_async(path: str) -> str:
    return await asyncio.to_thread(Path(path).read_text)

# Run CPU-bound work in process pool
async def compute_heavy(data: list[float]) -> float:
    loop = asyncio.get_running_loop()
    with ProcessPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, heavy_computation, data)
    return result
```

### Common Mistakes
1. **Blocking the event loop** — Using `requests.get()` or `time.sleep()` in async code; use httpx and `asyncio.sleep`
2. **Fire-and-forget tasks** — `asyncio.create_task()` without tracking; exceptions are silently lost
3. **Not using async context managers** — Connections, sessions not properly closed in async code
4. **Async for CPU-bound work** — Using async for computation; it still runs on one thread
5. **Mixing sync and async incorrectly** — Calling `asyncio.run()` inside an already-running loop

### Integration with Other Topics
- **Web Frameworks** — FastAPI is async-first; Django has async views
- **Generators** — Async generators extend the generator protocol
- **Concurrency** — Async is cooperative concurrency; threading/multiprocessing for other patterns
- **Testing** — pytest-asyncio for testing coroutines
- **Performance** — Async dramatically improves throughput for I/O-bound applications

---

## Resources

### Essential Reading
- *Fluent Python* (2nd ed.) — Async chapters
- *Using Asyncio in Python* — Caleb Hattingh
- Python asyncio documentation (docs.python.org)

### Online Resources
- Real Python: Async IO tutorials
- Trio documentation (trio.readthedocs.io) — structured concurrency reference
- anyio documentation — framework-agnostic async

### Practice Exercises
1. Convert a sync HTTP client to async with httpx and TaskGroup
2. Build an async web scraper with rate limiting (semaphore)
3. Create an async generator that streams from a paginated API
4. Use `asyncio.to_thread` to integrate a sync library in async code
5. Write async tests with pytest-asyncio and AsyncMock
