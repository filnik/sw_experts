# Multi-Tenancy Architecture

## Profile

### What It Is
Multi-Tenancy Architecture is a software design approach where a single instance of an application serves multiple tenants (customers/organizations), each with logical isolation of their data, configuration, and sometimes compute resources. It enables efficient resource sharing while maintaining security and customization boundaries.

### Origin and Evolution
- Originated in mainframe timesharing systems (1960s)
- SaaS model made multi-tenancy essential (Salesforce, 1999)
- Evolved from shared-nothing to flexible isolation models
- Cloud-native multi-tenancy with Kubernetes namespaces, row-level security
- Current: tenant-aware platforms, cell-based architectures (AWS)

### Key Authors and References
- **Tod Golding** — *Building Multi-Tenant SaaS Architectures* (AWS, 2024)
- **AWS SaaS Factory** — Multi-tenant reference architectures
- **Microsoft** — Azure multi-tenant architecture guides
- **Salesforce Engineering** — Pioneered multi-tenant at scale

### Key Concepts at a Glance
| Model | Isolation | Cost | Complexity |
|-------|-----------|------|------------|
| Shared Everything | Lowest (row-level) | Lowest | Medium |
| Shared DB, Separate Schema | Medium | Low-Medium | Medium |
| Separate Database | High | Higher | Higher |
| Separate Infrastructure | Highest (silo) | Highest | Highest |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Tenant isolation is non-negotiable** — Regardless of sharing model, one tenant must never access another tenant's data. This is a security invariant.
2. **Noisy neighbor prevention** — One tenant's workload should not degrade another tenant's performance. Rate limiting and resource quotas enforce fairness.
3. **Tenant-aware from the start** — Retrofitting multi-tenancy is extremely expensive. Design tenant context propagation into the architecture from day one.
4. **Right-size isolation per tier** — Different pricing tiers can have different isolation levels: shared for free/basic, dedicated for enterprise.
5. **Centralized tenant management** — Onboarding, offboarding, billing, and configuration should be managed centrally, even if compute is distributed.

### When to Use vs. When to Avoid

**Use when:**
- Building SaaS products serving multiple customers
- Cost efficiency through resource sharing is important
- Tenants have similar workloads and requirements
- Operational simplicity (manage one system, not N)

**Avoid when:**
- Regulatory requirements mandate complete infrastructure isolation
- Tenants have vastly different scale requirements
- Security/compliance requirements prevent any data co-location
- The system serves a single customer (self-hosted)

---

## Frameworks and Methodologies

### 1. Isolation Model Selection — step-by-step
1. Assess security/compliance requirements per tenant segment
2. Determine data sensitivity and isolation needs
3. Evaluate cost constraints (shared = cheaper, isolated = more expensive)
4. Define tenant tiers (basic = shared, enterprise = dedicated)
5. Choose isolation model per tier
6. Implement tenant context propagation throughout the stack

### 2. Tenant Onboarding Design — step-by-step
1. Define tenant provisioning workflow (create tenant record, configure resources)
2. Automate: no manual steps for tenant creation
3. For pool model: insert tenant record and configure row-level security
4. For silo model: provision dedicated infrastructure (database, compute)
5. Configure tenant-specific settings (branding, features, limits)
6. Verify isolation with automated tenant isolation tests

---

## How to Apply

### Decision Checklist
- [ ] Is tenant context propagated through every layer?
- [ ] Is data isolation enforced at the database level (not just application)?
- [ ] Are there resource quotas/rate limits per tenant?
- [ ] Is tenant onboarding/offboarding automated?
- [ ] Are there automated tests verifying tenant isolation?

### Implementation Patterns

**Row-Level Tenant Isolation (PostgreSQL):**
```sql
-- Row-level security
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON orders
    USING (tenant_id = current_setting('app.current_tenant')::uuid);

-- Application sets tenant context per request
SET app.current_tenant = 'tenant-uuid-here';
SELECT * FROM orders;  -- Only sees current tenant's orders
```

**Tenant Context Middleware:**
```python
class TenantMiddleware:
    async def __call__(self, request: Request, call_next):
        tenant_id = self._extract_tenant(request)
        if not tenant_id:
            return JSONResponse(status_code=401, content={"error": "Tenant not identified"})

        # Set tenant context for the request
        request.state.tenant_id = tenant_id

        # Set database-level isolation
        async with get_db_session() as session:
            await session.execute(text(f"SET app.current_tenant = '{tenant_id}'"))
            request.state.db = session
            response = await call_next(request)

        return response

    def _extract_tenant(self, request: Request) -> str | None:
        # From subdomain: tenant1.app.example.com
        host = request.headers.get("host", "")
        if ".app.example.com" in host:
            return host.split(".")[0]
        # From header
        return request.headers.get("X-Tenant-ID")
```

**Tenant-Aware Repository:**
```python
class OrderRepository:
    def __init__(self, db: Session, tenant_id: str):
        self.db = db
        self.tenant_id = tenant_id

    def find_all(self) -> list[Order]:
        # Even with RLS, always filter by tenant as defense in depth
        return self.db.query(Order).filter(
            Order.tenant_id == self.tenant_id
        ).all()

    def save(self, order: Order) -> None:
        order.tenant_id = self.tenant_id  # Always stamp tenant
        self.db.add(order)
```

**Noisy Neighbor Prevention:**
```python
class TenantRateLimiter:
    def __init__(self, redis: Redis):
        self.redis = redis

    async def check_limit(self, tenant_id: str, tier: str) -> bool:
        limits = {"free": 100, "basic": 1000, "enterprise": 10000}
        max_rpm = limits.get(tier, 100)

        key = f"rate:{tenant_id}:{current_minute()}"
        current = await self.redis.incr(key)
        if current == 1:
            await self.redis.expire(key, 60)
        return current <= max_rpm
```

### Common Mistakes
1. **Tenant ID not in every query** — Relying solely on application logic without database-level enforcement
2. **No noisy neighbor protection** — One tenant's bulk operation slows all tenants
3. **Tenant context leaking** — Request A's tenant context leaks to request B (especially with connection pooling)
4. **Manual tenant provisioning** — Not automating onboarding leads to errors and delays
5. **Testing only single-tenant** — Not testing cross-tenant isolation in automated tests

### Integration with Other Topics
- **Database Scaling** — Tenant-based sharding is a natural scaling strategy
- **Authentication and Authorization** — Tenant identification is part of auth flow
- **Cloud-Native Architecture** — Multi-tenant SaaS is a cloud-native pattern
- **Microservices** — Tenant context must propagate across service boundaries
- **Data Privacy** — GDPR/data residency requirements affect tenant isolation
- **API Gateway** — Gateway identifies and routes by tenant

---

## Resources

### Essential Reading
- *Building Multi-Tenant SaaS Architectures* — Tod Golding
- AWS SaaS Factory resources and reference architectures
- Microsoft Azure multi-tenant documentation

### Online Resources
- AWS SaaS Lens (Well-Architected Framework)
- Tod Golding's talks on multi-tenant SaaS
- Salesforce Engineering blog on multi-tenancy

### Practice Exercises
1. Implement row-level security for multi-tenant data in PostgreSQL
2. Build tenant context middleware that extracts tenant from subdomain
3. Design a tiered isolation model (shared for basic, dedicated for enterprise)
4. Implement per-tenant rate limiting
5. Write automated tests that verify cross-tenant isolation
