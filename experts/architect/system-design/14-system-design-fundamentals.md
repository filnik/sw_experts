# System Design Fundamentals

## Profile

### What It Is
System Design Fundamentals encompass the core building blocks and principles for designing scalable, reliable, and performant distributed systems. This includes understanding load balancing, horizontal and vertical scaling, DNS, CDNs, proxies, rate limiting, and capacity planning — the foundational infrastructure decisions every architect must make.

### Origin and Evolution
- Rooted in decades of internet infrastructure development (1990s-present)
- Accelerated by hyperscale companies (Google, Amazon, Facebook) sharing their architectures
- Popularized in interview culture but deeply practical for real-world system design
- Cloud providers (AWS, GCP, Azure) commoditized many of these building blocks
- Modern emphasis on managed services vs. self-hosted infrastructure

### Key Authors and References
- **Martin Kleppmann** — *Designing Data-Intensive Applications* (2017)
- **Alex Xu** — *System Design Interview* (Vol. 1 & 2)
- **Werner Vogels** — "Eventually Consistent" and Amazon architecture papers
- **Google SRE Team** — *Site Reliability Engineering* (O'Reilly)

### Key Concepts at a Glance
| Concept | Purpose |
|---------|---------|
| Load Balancing | Distribute traffic across servers |
| Horizontal Scaling | Add more machines to handle load |
| Vertical Scaling | Upgrade existing machine resources |
| CDN | Cache static content at edge locations |
| DNS | Translate domain names to IP addresses |
| Reverse Proxy | Intermediary for backend servers |
| Rate Limiting | Control request throughput |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Scale horizontally** — Prefer adding more machines over bigger machines. Horizontal scaling is near-unlimited; vertical scaling hits hardware ceilings.
2. **Eliminate single points of failure** — Every component should be redundant. If one instance fails, others handle the load seamlessly.
3. **Push work to the edge** — Serve static content from CDNs, cache aggressively, minimize round trips to origin servers.
4. **Design for the 99th percentile** — Optimize for tail latency, not just averages. The worst user experience defines your system's quality.
5. **Measure, then optimize** — Decisions should be data-driven. Profile bottlenecks before optimizing; premature optimization is wasteful.

### When to Use vs. When to Avoid

**Heavy infrastructure design when:**
- Traffic is high (>1000 RPS) or expected to grow significantly
- Availability requirements are strict (99.9%+)
- Global user base requires low-latency access
- Data volume exceeds single-machine capacity

**Keep it simple when:**
- Traffic is low and predictable
- Single-region, single-server suffices
- Speed of development matters more than scalability
- Managed services handle the complexity

---

## Frameworks and Methodologies

### 1. Back-of-the-Envelope Estimation — step-by-step
1. Estimate daily active users (DAU) and peak concurrent users
2. Estimate requests per second: DAU × actions/day / 86400
3. Estimate storage: records/day × record size × retention period
4. Estimate bandwidth: RPS × average response size
5. Determine read/write ratio to inform caching and replication strategy
6. Apply safety margin (2-5x) for peak traffic

### 2. System Design Process — step-by-step
1. Clarify requirements: functional, non-functional, constraints
2. Estimate scale: users, traffic, data, bandwidth
3. Design high-level architecture: clients, load balancers, services, databases
4. Deep dive into components: data model, API design, caching, messaging
5. Address bottlenecks: identify and resolve scaling limits
6. Discuss tradeoffs: consistency, availability, cost, complexity

---

## How to Apply

### Decision Checklist
- [ ] What is the expected QPS (queries per second) at peak?
- [ ] What is the read/write ratio?
- [ ] What are the latency requirements (p50, p99)?
- [ ] What is the availability target (99.9%, 99.99%)?
- [ ] Where are the users geographically?
- [ ] What is the data retention policy?

### Implementation Patterns

**Load Balancing Strategies:**
```
Layer 4 (TCP):     Fast, connection-level distribution
Layer 7 (HTTP):    Content-aware routing (URL, headers, cookies)
Algorithms:
  Round Robin:     Simple rotation across servers
  Weighted:        More traffic to stronger servers
  Least Connections: Route to server with fewest active connections
  IP Hash:         Consistent routing for session affinity

Example (nginx):
  upstream backend {
    least_conn;
    server backend1:8080 weight=3;
    server backend2:8080 weight=1;
    server backend3:8080 backup;
  }
```

**Horizontal Scaling Pattern:**
```
Stateless Services (easy to scale):
  - No server-side session storage
  - Session state in external store (Redis, JWT)
  - Any instance can handle any request

Stateful Services (harder to scale):
  - Partition state using consistent hashing
  - Use leader-follower for data replication
  - Shard by user ID, tenant ID, or geography
```

**Rate Limiting:**
```python
import time
from collections import defaultdict

class SlidingWindowRateLimiter:
    def __init__(self, max_requests: int, window_seconds: int):
        self.max_requests = max_requests
        self.window = window_seconds
        self.requests: dict[str, list[float]] = defaultdict(list)

    def allow(self, client_id: str) -> bool:
        now = time.time()
        window_start = now - self.window
        # Remove expired timestamps
        self.requests[client_id] = [
            t for t in self.requests[client_id] if t > window_start
        ]
        if len(self.requests[client_id]) >= self.max_requests:
            return False
        self.requests[client_id].append(now)
        return True

# 100 requests per minute per client
limiter = SlidingWindowRateLimiter(max_requests=100, window_seconds=60)
```

**CDN Integration:**
```
Origin Server (your backend)
    ↑ cache miss
Edge Server (CDN PoP closest to user)
    ↑ request
User

Cache-Control headers drive CDN behavior:
  Cache-Control: public, max-age=3600        → CDN caches for 1 hour
  Cache-Control: private, no-cache           → Never cached by CDN
  Cache-Control: public, s-maxage=86400      → CDN caches for 1 day
```

### Common Mistakes
1. **Premature scaling** — Adding complex infrastructure before measuring actual bottlenecks
2. **Vertical scaling dependency** — Relying on bigger machines instead of designing for horizontal scaling
3. **Ignoring tail latency** — Optimizing for p50 while p99 is 10x worse
4. **Session affinity everywhere** — Sticky sessions prevent effective load balancing and complicate failover
5. **No capacity planning** — Running at 90% capacity with no plan for traffic spikes
6. **Forgetting the network** — Assuming zero-latency between services in different availability zones

### Integration with Other Topics
- **Microservices** — Each service needs its own scaling strategy
- **Caching Strategies** — Caching is the most impactful scaling technique
- **Cloud-Native Architecture** — Cloud services provide managed versions of these building blocks
- **Observability** — Metrics and monitoring drive scaling decisions
- **Resilience Patterns** — Load balancing and rate limiting are resilience mechanisms
- **Database Scaling** — Database is often the first bottleneck

---

## Resources

### Essential Reading
- *Designing Data-Intensive Applications* — Martin Kleppmann
- *System Design Interview* (Vol. 1 & 2) — Alex Xu
- *The Art of Scalability* — Abbott & Fisher

### Online Resources
- High Scalability blog (highscalability.com)
- AWS Well-Architected Framework
- Google Cloud Architecture Center

### Practice Exercises
1. Estimate the infrastructure needed for a URL shortener serving 100M URLs/day
2. Design a load balancing strategy for a stateful WebSocket service
3. Calculate CDN cache hit ratio impact on origin server load
4. Implement a token bucket rate limiter
5. Design a system that handles 10x traffic spikes gracefully
