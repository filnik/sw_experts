# Caching Strategies

## Profile

### What It Is
Caching is the practice of storing copies of frequently accessed data in a faster storage layer to reduce latency, decrease load on primary data stores, and improve throughput. Caching strategies define how data enters the cache, when it's invalidated, and how consistency is maintained between cache and source of truth.

### Origin and Evolution
- CPU caches existed since the 1960s; application-level caching followed similar principles
- Memcached (2003) pioneered distributed in-memory caching
- Redis (2009) became the dominant cache with data structures, persistence, and pub/sub
- CDNs (Akamai, Cloudflare, Fastly) extended caching to the network edge
- Modern: multi-layer caching (browser → CDN → API gateway → application → database)

### Key Authors and References
- **Martin Kleppmann** — Caching in *Designing Data-Intensive Applications*
- **Salvatore Sanfilippo** — Redis creator
- **Alex Xu** — Caching patterns in *System Design Interview*
- **AWS Architecture Center** — Caching best practices

### Key Concepts at a Glance
| Strategy | Description | Consistency |
|----------|-------------|-------------|
| Cache-Aside | App checks cache first, loads from DB on miss | Eventual |
| Read-Through | Cache loads from DB transparently on miss | Eventual |
| Write-Through | Write goes to cache and DB synchronously | Strong |
| Write-Behind | Write goes to cache; DB updated asynchronously | Eventual |
| Write-Around | Write goes directly to DB; cache populated on read | Eventual |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Cache what's read often, changed rarely** — The highest-value cache targets are hot data with a high read-to-write ratio.
2. **Every cache needs an eviction strategy** — Caches have finite memory. LRU, LFU, TTL, or combinations determine what gets evicted.
3. **Cache invalidation is the hard problem** — "There are only two hard things in CS: cache invalidation and naming things." Plan invalidation carefully.
4. **Consistency tradeoff is explicit** — Cached data is inherently stale. Define acceptable staleness per use case.
5. **Multi-layer caching multiplies benefits** — Browser cache + CDN + application cache + database cache each reduce load on the next layer.

### When to Use vs. When to Avoid

**Use when:**
- Read-heavy workloads (read/write ratio > 10:1)
- Data computation is expensive (complex queries, aggregations)
- Latency requirements are strict (< 10ms)
- Data source is slow or has limited throughput
- Data doesn't change frequently

**Avoid when:**
- Write-heavy workloads (cache thrashing)
- Data must be always up-to-date (strong consistency required)
- Working set is too large to cache effectively
- Cache adds complexity without measurable benefit
- Data access patterns are uniform (no hot spots)

---

## Frameworks and Methodologies

### 1. Caching Strategy Selection — step-by-step
1. Analyze read/write ratio for each data type
2. Determine acceptable staleness (seconds, minutes, hours)
3. Identify hot data vs. cold data (Pareto: 20% data serves 80% reads)
4. Choose caching strategy: cache-aside for flexibility, read/write-through for consistency
5. Set TTL values based on data change frequency
6. Plan cache warming for cold-start scenarios
7. Monitor hit rate and adjust

### 2. Multi-Layer Cache Design — step-by-step
1. Layer 1: Browser/client cache (static assets, API responses with Cache-Control)
2. Layer 2: CDN cache (static content, cacheable API responses)
3. Layer 3: API gateway cache (common queries, rate-limited endpoints)
4. Layer 4: Application cache (Redis/Memcached for computed results, sessions)
5. Layer 5: Database cache (query cache, materialized views)
6. Set TTLs to decrease from outer to inner layers

---

## How to Apply

### Decision Checklist
- [ ] What is the read/write ratio for this data?
- [ ] What staleness is acceptable?
- [ ] What is the cache hit rate target (> 80%)?
- [ ] How will cache be invalidated on writes?
- [ ] What happens on cache failure (degrade gracefully)?
- [ ] Is cache warming needed on startup/deployment?

### Implementation Patterns

**Cache-Aside (Lazy Loading):**
```python
class UserService:
    def __init__(self, cache: Redis, db: Database):
        self.cache = cache
        self.db = db

    def get_user(self, user_id: str) -> User:
        # 1. Check cache
        cached = self.cache.get(f"user:{user_id}")
        if cached:
            return User.from_json(cached)

        # 2. Cache miss: load from DB
        user = self.db.query("SELECT * FROM users WHERE id = %s", user_id)
        if user is None:
            raise UserNotFound(user_id)

        # 3. Populate cache with TTL
        self.cache.setex(f"user:{user_id}", 3600, user.to_json())
        return user

    def update_user(self, user_id: str, data: dict) -> User:
        user = self.db.update("users", user_id, data)
        # Invalidate cache on write
        self.cache.delete(f"user:{user_id}")
        return user
```

**Write-Through:**
```python
class WriteThrough:
    def __init__(self, cache: Redis, db: Database):
        self.cache = cache
        self.db = db

    def write(self, key: str, value: Any) -> None:
        # Write to both cache and DB synchronously
        self.db.upsert(key, value)
        self.cache.setex(key, 3600, serialize(value))

    def read(self, key: str) -> Any:
        cached = self.cache.get(key)
        if cached:
            return deserialize(cached)
        # Shouldn't happen often with write-through
        value = self.db.get(key)
        self.cache.setex(key, 3600, serialize(value))
        return value
```

**Cache Stampede Prevention:**
```python
import threading

class StampedeProtectedCache:
    def __init__(self, cache: Redis, db: Database):
        self.cache = cache
        self.db = db
        self.locks: dict[str, threading.Lock] = {}

    def get(self, key: str, loader: Callable, ttl: int = 3600) -> Any:
        # Try cache first
        value = self.cache.get(key)
        if value is not None:
            return deserialize(value)

        # Acquire per-key lock to prevent stampede
        lock = self.locks.setdefault(key, threading.Lock())
        with lock:
            # Double-check after acquiring lock
            value = self.cache.get(key)
            if value is not None:
                return deserialize(value)

            # Load from source and cache
            result = loader()
            self.cache.setex(key, ttl, serialize(result))
            return result
```

**HTTP Caching Headers:**
```
# Immutable static assets (hashed filenames)
Cache-Control: public, max-age=31536000, immutable

# API response cacheable for 5 minutes
Cache-Control: public, max-age=300, s-maxage=600
ETag: "abc123"

# Private user data, revalidate every request
Cache-Control: private, no-cache
ETag: "user-version-42"

# Never cache
Cache-Control: no-store
```

### Common Mistakes
1. **No TTL** — Cached data lives forever; stale data accumulates
2. **Cache stampede** — Cache expires and 1000 requests simultaneously hit the database
3. **Caching everything** — Low-value data fills the cache, evicting high-value data
4. **Delete-before-write race** — Invalidating cache, then another request re-caches stale data before the write completes
5. **No monitoring** — Not tracking hit rate, miss rate, and eviction rate means flying blind
6. **Cache as source of truth** — Cache is volatile; Redis can lose data. The database is the source of truth.

### Integration with Other Topics
- **System Design Fundamentals** — Caching is the most impactful scaling technique
- **Database Scaling** — Read replicas + caching reduce primary database load
- **CDN** — CDNs are geographically distributed caches for static and dynamic content
- **CQRS** — Read models can be cached aggressively since they're already denormalized
- **Microservices** — Each service manages its own cache; distributed caching adds complexity
- **Resilience Patterns** — Cache serves as fallback when the database is slow or unavailable

---

## Resources

### Essential Reading
- *Designing Data-Intensive Applications* — Martin Kleppmann (caching and replication)
- *System Design Interview* — Alex Xu (caching chapter)
- Redis documentation on caching patterns

### Online Resources
- AWS ElastiCache best practices
- Cloudflare caching documentation
- Facebook's Memcached scaling paper

### Practice Exercises
1. Implement cache-aside with TTL and invalidation on write
2. Measure the impact of caching on a read-heavy endpoint (before/after)
3. Implement cache stampede prevention with distributed locks
4. Design a multi-layer cache strategy (CDN + Redis + DB query cache)
5. Monitor cache hit rate and optimize TTL values
