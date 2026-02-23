# Concurrency and Parallelism

## Profile

### What It Is
Concurrency is the ability to deal with multiple tasks at once (structuring a program to handle multiple tasks that may overlap in time). Parallelism is the ability to execute multiple tasks simultaneously (using multiple CPU cores). Understanding the distinction and the models available — threads, processes, async/await, actors, CSP — is essential for building responsive, high-throughput systems.

### Origin and Evolution
- Concurrency primitives (threads, mutexes, semaphores) from operating systems theory (Dijkstra, 1960s)
- Actor model by Carl Hewitt (1973), popularized by Erlang/OTP
- Communicating Sequential Processes (CSP) by Tony Hoare (1978), implemented in Go
- Async/await pattern emerged in C# (2012), adopted by Python, JavaScript, Rust, Kotlin
- Modern trend: structured concurrency (Java 21 Virtual Threads, Python trio, Kotlin coroutines)

### Key Authors and References
- **Edsger Dijkstra** — Semaphores, mutual exclusion
- **Tony Hoare** — CSP, *Communicating Sequential Processes* (1985)
- **Carl Hewitt** — Actor model (1973)
- **Joe Armstrong** — Erlang/OTP, "let it crash" philosophy
- **Martin Kleppmann** — Concurrency in *Designing Data-Intensive Applications*

### Key Concepts at a Glance
| Model | Mechanism | Communication | Best For |
|-------|-----------|---------------|----------|
| Threads | OS threads, shared memory | Locks, mutexes | CPU-bound (parallel) |
| Async/Await | Event loop, coroutines | Futures, promises | I/O-bound (concurrent) |
| Actors | Isolated processes, mailboxes | Message passing | Distributed, fault-tolerant |
| CSP | Goroutines, channels | Channel send/receive | Concurrent pipelines |
| Processes | OS processes, separate memory | IPC, pipes, shared memory | CPU-bound, isolation |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Concurrency is not parallelism** — Concurrency is about structure (managing multiple tasks); parallelism is about execution (running tasks simultaneously). You can have concurrency without parallelism.
2. **Share nothing when possible** — Shared mutable state is the root of concurrency bugs. Prefer message passing, immutability, or process isolation.
3. **Choose the right model for the workload** — I/O-bound work benefits from async; CPU-bound work benefits from true parallelism (processes or threads).
4. **Structured concurrency** — Concurrent tasks should have clear ownership and lifecycle. No fire-and-forget tasks that leak resources.
5. **Make concurrency explicit** — Concurrency should be visible in the code. Hidden shared state and implicit threading cause bugs that are hard to reproduce and debug.

### When to Use vs. When to Avoid

**Threads/processes (parallelism):**
- CPU-bound computation (data processing, ML inference, image processing)
- Need true parallel execution on multiple cores

**Async/await (concurrency):**
- I/O-bound operations (HTTP requests, database queries, file I/O)
- High-concurrency servers handling thousands of connections
- Event-driven applications

**Actors:**
- Distributed systems requiring fault tolerance
- Stateful services with message-driven interactions
- Systems requiring supervision and restart strategies

---

## Frameworks and Methodologies

### 1. Concurrency Model Selection — step-by-step
1. Profile the workload: I/O-bound or CPU-bound?
2. Determine scale: tens, hundreds, or thousands of concurrent tasks?
3. Assess state sharing requirements: shared state or independent?
4. Choose model: async for I/O, threads/processes for CPU, actors for distributed
5. Implement with structured concurrency (task groups, supervision trees)
6. Test under load with race condition detectors

### 2. Shared State Management — step-by-step
1. Identify all shared mutable state
2. Eliminate sharing where possible (copy data, use immutable structures)
3. For remaining shared state, choose a synchronization primitive (lock, atomic, channel)
4. Minimize critical section size (hold locks as briefly as possible)
5. Use lock-free data structures for high-contention paths
6. Test with thread sanitizer and stress tests

---

## How to Apply

### Decision Checklist
- [ ] Is this workload I/O-bound or CPU-bound?
- [ ] Is shared mutable state minimized?
- [ ] Are concurrent tasks structured (owned, scoped, cancellable)?
- [ ] Are error handling and cancellation handled for concurrent tasks?
- [ ] Has the solution been tested under concurrent load?

### Implementation Patterns

**Async/Await (Python):**
```python
import asyncio
import aiohttp

async def fetch_all_users(user_ids: list[str]) -> list[User]:
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_user(session, uid) for uid in user_ids]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        return [r for r in results if not isinstance(r, Exception)]

async def fetch_user(session: aiohttp.ClientSession, user_id: str) -> User:
    async with session.get(f"https://api.example.com/users/{user_id}") as resp:
        data = await resp.json()
        return User(**data)

# Structured concurrency with TaskGroup (Python 3.11+)
async def process_orders(orders: list[Order]) -> list[Result]:
    results = []
    async with asyncio.TaskGroup() as tg:
        for order in orders:
            tg.create_task(process_single_order(order, results))
    return results
```

**Thread Pool for CPU-Bound (Python):**
```python
from concurrent.futures import ProcessPoolExecutor
import multiprocessing

def cpu_intensive_task(data: bytes) -> Result:
    # Runs in a separate process, bypasses GIL
    return heavy_computation(data)

def process_batch(items: list[bytes]) -> list[Result]:
    workers = multiprocessing.cpu_count()
    with ProcessPoolExecutor(max_workers=workers) as executor:
        futures = [executor.submit(cpu_intensive_task, item) for item in items]
        return [f.result() for f in futures]
```

**Actor-Like Pattern (Python):**
```python
import asyncio
from dataclasses import dataclass

@dataclass
class Message:
    type: str
    payload: dict

class Actor:
    def __init__(self):
        self.mailbox: asyncio.Queue[Message] = asyncio.Queue()

    async def run(self):
        while True:
            message = await self.mailbox.get()
            await self.handle(message)

    async def handle(self, message: Message):
        raise NotImplementedError

    async def send(self, message: Message):
        await self.mailbox.put(message)

class AccountActor(Actor):
    def __init__(self, balance: float = 0):
        super().__init__()
        self.balance = balance

    async def handle(self, message: Message):
        match message.type:
            case "deposit":
                self.balance += message.payload["amount"]
            case "withdraw":
                if self.balance >= message.payload["amount"]:
                    self.balance -= message.payload["amount"]
```

**Go-Style CSP (Python with channels):**
```python
import asyncio

async def producer(channel: asyncio.Queue, items: list):
    for item in items:
        await channel.put(item)
    await channel.put(None)  # Sentinel

async def consumer(channel: asyncio.Queue, process_fn):
    while True:
        item = await channel.get()
        if item is None:
            break
        await process_fn(item)

async def pipeline():
    raw_channel = asyncio.Queue(maxsize=10)
    processed_channel = asyncio.Queue(maxsize=10)

    await asyncio.gather(
        producer(raw_channel, data),
        transformer(raw_channel, processed_channel),
        consumer(processed_channel, save),
    )
```

### Common Mistakes
1. **Async for CPU-bound work** — Async/await doesn't parallelize CPU work; it only helps with I/O waiting
2. **Shared mutable state without locks** — Race conditions that appear intermittently and are nearly impossible to debug
3. **Deadlocks from lock ordering** — Multiple locks acquired in different orders by different threads
4. **Fire-and-forget tasks** — Unowned concurrent tasks that leak resources, swallow exceptions, or outlive their scope
5. **Blocking the event loop** — Calling synchronous I/O or CPU-heavy code inside an async context
6. **Over-threading** — Creating thousands of OS threads; use async or thread pools instead

### Integration with Other Topics
- **Microservices** — Each service handles concurrency independently
- **Event-Driven Architecture** — Event processing is inherently concurrent
- **Database Transactions** — Concurrency control at the data layer (locks, MVCC, isolation levels)
- **Resilience Patterns** — Timeouts and bulkheads manage concurrent resource usage
- **System Design** — Concurrency model choice affects scalability and throughput
- **Python Async Programming** — Python-specific async patterns and GIL considerations

---

## Resources

### Essential Reading
- *Designing Data-Intensive Applications* — Martin Kleppmann (concurrency chapters)
- *Concurrency in Go* — Katherine Cox-Buday
- *Programming Erlang* — Joe Armstrong (actor model, "let it crash")
- *Java Concurrency in Practice* — Brian Goetz

### Online Resources
- Rob Pike's "Concurrency is Not Parallelism" talk
- Trio (Python structured concurrency) documentation
- Go concurrency patterns (golang.org)

### Practice Exercises
1. Implement a web scraper using async/await for I/O-bound concurrency
2. Parallelize image processing using multiprocessing
3. Build a producer-consumer pipeline using channels/queues
4. Implement an actor that manages account balance with message passing
5. Find and fix a race condition using a thread sanitizer
