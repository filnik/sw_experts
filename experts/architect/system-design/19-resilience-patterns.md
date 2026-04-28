# Resilience Patterns

## Profile

### What It Is
Resilience Patterns are design strategies that enable distributed systems to handle failures gracefully — maintaining functionality (possibly degraded) rather than failing completely. Key patterns include circuit breaker, retry with backoff, bulkhead, timeout, fallback, and rate limiting.

### Origin and Evolution
- Inspired by electrical engineering (circuit breakers) and naval architecture (bulkheads)
- Michael Nygard's *Release It!* (2007) brought resilience patterns to mainstream software
- Netflix Hystrix (2012) popularized circuit breaker and bulkhead in Java
- Modern implementations: Resilience4j (Java), Polly (.NET), tenacity (Python)
- Cloud providers offer managed resilience features (AWS App Mesh, Istio)

### Key Authors and References
- **Michael Nygard** — *Release It!* (2007, 2nd ed. 2018)
- **Netflix Engineering** — Hystrix library and chaos engineering
- **Adrian Cockcroft** — Netflix architecture and resilience
- **Uwe Friedrichsen** — Resilience patterns talks and articles

### Key Concepts at a Glance
| Pattern | Purpose |
|---------|---------|
| Circuit Breaker | Stop calling a failing service; allow recovery |
| Retry | Re-attempt failed operations with backoff |
| Bulkhead | Isolate failures to prevent cascade |
| Timeout | Bound the time waiting for a response |
| Fallback | Provide degraded response when primary fails |
| Rate Limiting | Control request rate to prevent overload |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Fail fast** — Detect failures quickly and stop propagating them. A slow failure is worse than a fast failure.
2. **Design for failure** — Every remote call can fail. Every dependency can be unavailable. Build systems that work despite failures.
3. **Isolate failure domains** — A failure in one component should not cascade to others. Use bulkheads and circuit breakers.
4. **Degrade gracefully** — When a dependency fails, provide a degraded experience rather than a complete failure.
5. **Recover automatically** — Systems should self-heal. Circuit breakers should re-test, retries should eventually succeed, and health checks should drive recovery.

### When to Use vs. When to Avoid

**Use when:**
- The system has multiple external dependencies (services, databases, APIs)
- Availability requirements are high (99.9%+)
- Failures in one component should not bring down the whole system
- The system operates at scale where failures are statistically inevitable

**Avoid when:**
- Single-process, single-dependency applications
- The overhead of resilience patterns exceeds the cost of occasional failures
- Retry and fallback mask bugs that should be fixed

---

## Frameworks and Methodologies

### 1. Resilience Stack Design — step-by-step
1. Wrap every external call with a timeout (always, no exceptions)
2. Add retry with exponential backoff for transient failures
3. Wrap retried calls in a circuit breaker to stop hammering failed services
4. Add bulkheads to isolate critical paths from non-critical paths
5. Define fallback behavior for every dependency failure
6. Monitor circuit breaker state, retry counts, and error rates

### 2. Chaos Engineering — step-by-step
1. Define steady state: normal system behavior metrics
2. Hypothesize: the system should tolerate failure of service X
3. Inject failure: kill instances, add latency, corrupt responses
4. Observe: did the system maintain steady state?
5. Fix: address any cascading failures discovered
6. Automate: run chaos experiments regularly in staging/production

---

## How to Apply

### Decision Checklist
- [ ] Does every external call have a timeout?
- [ ] Are retries configured with backoff and jitter?
- [ ] Are circuit breakers in place for unreliable dependencies?
- [ ] Are critical and non-critical paths isolated (bulkhead)?
- [ ] Is there a fallback for every external dependency?
- [ ] Are resilience metrics monitored and alerted?

### Implementation Patterns

**Circuit Breaker:**
```python
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"         # Normal operation
    OPEN = "open"             # Failing, reject calls
    HALF_OPEN = "half_open"   # Testing if service recovered

class CircuitBreaker:
    def __init__(self, failure_threshold: int = 5,
                 recovery_timeout: float = 30.0):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.last_failure_time = 0.0

    def call(self, func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            if time.time() - self.last_failure_time > self.recovery_timeout:
                self.state = CircuitState.HALF_OPEN
            else:
                raise CircuitOpenError("Circuit is open")

        try:
            result = func(*args, **kwargs)
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise

    def _on_success(self):
        self.failure_count = 0
        self.state = CircuitState.CLOSED

    def _on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
```

**Retry with Exponential Backoff and Jitter:**
```python
import random
import time

def retry_with_backoff(func, max_retries: int = 3,
                       base_delay: float = 1.0,
                       max_delay: float = 30.0):
    for attempt in range(max_retries + 1):
        try:
            return func()
        except TransientError:
            if attempt == max_retries:
                raise
            delay = min(base_delay * (2 ** attempt), max_delay)
            jitter = random.uniform(0, delay * 0.1)  # 10% jitter
            time.sleep(delay + jitter)
```

**Bulkhead (Thread Pool Isolation):**
```python
from concurrent.futures import ThreadPoolExecutor, TimeoutError

class BulkheadExecutor:
    """Isolate different dependency calls into separate thread pools."""

    def __init__(self):
        self.pools = {
            "payment": ThreadPoolExecutor(max_workers=10),
            "inventory": ThreadPoolExecutor(max_workers=5),
            "notification": ThreadPoolExecutor(max_workers=3),
        }

    def call(self, pool_name: str, func, *args, timeout: float = 5.0):
        pool = self.pools[pool_name]
        future = pool.submit(func, *args)
        try:
            return future.result(timeout=timeout)
        except TimeoutError:
            future.cancel()
            raise ServiceTimeoutError(f"{pool_name} timed out")
```

**Fallback Strategy:**
```python
class ProductService:
    def __init__(self, primary: ProductAPI, cache: Redis):
        self.primary = primary
        self.cache = cache
        self.breaker = CircuitBreaker()

    def get_product(self, product_id: str) -> Product:
        try:
            product = self.breaker.call(self.primary.get, product_id)
            self.cache.setex(f"product:{product_id}", 3600, product.to_json())
            return product
        except (CircuitOpenError, ServiceError):
            # Fallback: serve from cache (stale but available)
            cached = self.cache.get(f"product:{product_id}")
            if cached:
                return Product.from_json(cached)
            # Final fallback: default product placeholder
            return Product.unavailable(product_id)
```

### Common Mistakes
1. **No timeout** — The most basic resilience pattern, yet often missing. Without timeouts, a slow dependency blocks resources indefinitely.
2. **Retry storms** — Retrying without backoff or jitter causes all clients to retry simultaneously, overwhelming the recovering service.
3. **Circuit breaker too sensitive** — Opening on the first error; should require a pattern of failures.
4. **Fallback hides bugs** — If fallback is always triggered, the primary path has a bug. Monitor fallback activation.
5. **Missing bulkheads** — All dependencies sharing one thread pool; a slow dependency starves others.

### Integration with Other Topics
- **Microservices** — Each inter-service call needs resilience patterns
- **API Gateway** — Gateway applies resilience patterns at the edge
- **Observability** — Resilience metrics (circuit state, retry count, error rate) must be monitored
- **Site Reliability Engineering** — Resilience patterns are SRE's implementation tools
- **Caching** — Cache serves as a resilience fallback for database/API failures
- **Event-Driven Architecture** — Async messaging inherently provides resilience through buffering

---

## Resources

### Essential Reading
- *Release It!* (2nd ed.) — Michael Nygard
- *Chaos Engineering* — Casey Rosenthal & Nora Jones
- Netflix Hystrix documentation (archived but educational)

### Online Resources
- Resilience4j documentation (resilience4j.readme.io)
- Polly .NET documentation
- Netflix Tech Blog on resilience engineering

### Practice Exercises
1. Implement a circuit breaker with closed/open/half-open states
2. Add retry with exponential backoff and jitter to an HTTP client
3. Design a bulkhead isolating payment processing from notifications
4. Implement a fallback chain: primary → cache → default
5. Run a chaos experiment: kill a dependency and verify graceful degradation
