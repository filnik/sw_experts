# API Gateway and BFF Patterns

## Profile

### What It Is
An API Gateway is a single entry point for client requests that routes, aggregates, and transforms calls to backend services. The Backend for Frontend (BFF) pattern extends this by creating dedicated backend services tailored to each client type (web, mobile, third-party), optimizing the API surface for each consumer's specific needs.

### Origin and Evolution
- API Gateway emerged from the need to manage microservices complexity at the edge
- Netflix Zuul (2013) was among the first widely-known API gateways for microservices
- BFF pattern articulated by Sam Newman and Phil Calçado (~2015)
- Modern gateways: Kong, AWS API Gateway, Envoy, Traefik
- GraphQL is sometimes used as a BFF layer

### Key Authors and References
- **Sam Newman** — BFF pattern in *Building Microservices*
- **Phil Calçado** — "The Back-end for Front-end Pattern (BFF)" article
- **Chris Richardson** — API Gateway pattern in *Microservices Patterns*
- **Netflix Engineering** — Zuul gateway architecture

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| API Gateway | Edge service that handles routing, auth, rate limiting, protocol translation |
| BFF | Dedicated backend tailored to a specific frontend client |
| Request Aggregation | Combining multiple service calls into one client response |
| Protocol Translation | Converting between protocols (e.g., REST to gRPC) |
| Edge Functions | Cross-cutting concerns handled at the gateway (auth, logging, CORS) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Single entry point** — Clients interact with one URL/endpoint. The gateway handles routing to appropriate backend services.
2. **Cross-cutting concerns at the edge** — Authentication, rate limiting, logging, CORS, and TLS termination belong at the gateway, not in every service.
3. **Client-specific optimization** — Different clients (web, mobile, IoT) need different data shapes and protocols. BFFs optimize for each.
4. **Backend insulation** — Backend services can change, split, or merge without client impact. The gateway/BFF absorbs the change.
5. **Avoid the "god gateway"** — The gateway should route and apply policies, not contain business logic. Keep it thin.

### When to Use vs. When to Avoid

**API Gateway — use when:**
- Multiple backend services need a unified entry point
- Cross-cutting concerns (auth, rate limiting) need centralization
- Clients need response aggregation from multiple services

**BFF — use when:**
- Web and mobile clients need significantly different API shapes
- Frontend teams want autonomy over their backend API
- Different clients have different performance/payload requirements

**Avoid when:**
- Single backend, single client — a gateway is unnecessary overhead
- The gateway becomes a bottleneck or single point of failure
- Business logic creeps into the gateway layer

---

## Frameworks and Methodologies

### 1. API Gateway Design — step-by-step
1. Identify all client types and their entry points
2. Define routing rules: URL patterns to backend services
3. Implement cross-cutting concerns: auth, rate limiting, CORS, logging
4. Add request/response transformation where needed
5. Configure health checks and circuit breakers for backend services
6. Set up monitoring and alerting for gateway metrics

### 2. BFF Implementation — step-by-step
1. Identify distinct client types (web SPA, mobile app, third-party)
2. Create a dedicated BFF service for each client type
3. Each BFF aggregates and shapes data from backend services
4. Frontend team owns their BFF (aligned team ownership)
5. BFFs can use different protocols (REST for web, GraphQL for mobile)
6. Share nothing between BFFs except the underlying service APIs

---

## How to Apply

### Decision Checklist
- [ ] Is the gateway handling only cross-cutting concerns (no business logic)?
- [ ] Can the gateway handle the traffic load (not a bottleneck)?
- [ ] Is there a clear BFF per client type (not one BFF for all)?
- [ ] Does each BFF have clear team ownership?
- [ ] Are gateway configurations version-controlled and testable?

### Implementation Patterns

**API Gateway Configuration:**
```yaml
# Kong / similar gateway configuration
services:
  - name: order-service
    url: http://order-service:8080
    routes:
      - paths: ["/api/v1/orders"]
        methods: ["GET", "POST", "PATCH", "DELETE"]
    plugins:
      - name: rate-limiting
        config:
          minute: 100
      - name: jwt-auth
      - name: cors
        config:
          origins: ["https://app.example.com"]

  - name: product-service
    url: http://product-service:8080
    routes:
      - paths: ["/api/v1/products"]
    plugins:
      - name: rate-limiting
        config:
          minute: 500
      - name: proxy-cache
        config:
          content_type: ["application/json"]
          cache_ttl: 300
```

**BFF — Web vs. Mobile:**
```python
# Web BFF — returns rich, detailed data
class WebOrderBFF:
    def __init__(self, order_svc: OrderClient, product_svc: ProductClient,
                 user_svc: UserClient):
        self.order_svc = order_svc
        self.product_svc = product_svc
        self.user_svc = user_svc

    async def get_order_detail(self, order_id: str) -> WebOrderDetail:
        order, products, customer = await asyncio.gather(
            self.order_svc.get(order_id),
            self.product_svc.get_batch(order.product_ids),
            self.user_svc.get(order.customer_id),
        )
        return WebOrderDetail(
            order=order,
            products=products,          # Full product details
            customer=customer,          # Full customer profile
            recommendations=await self.product_svc.recommend(order.product_ids),
        )

# Mobile BFF — returns minimal, optimized data
class MobileOrderBFF:
    def __init__(self, order_svc: OrderClient, product_svc: ProductClient):
        self.order_svc = order_svc
        self.product_svc = product_svc

    async def get_order_summary(self, order_id: str) -> MobileOrderSummary:
        order = await self.order_svc.get(order_id)
        return MobileOrderSummary(
            order_id=order.id,
            status=order.status,
            total=order.total,
            item_count=len(order.items),
            thumbnail_url=order.items[0].thumbnail if order.items else None,
        )
```

**Gateway Aggregation:**
```python
# Gateway aggregates dashboard data from multiple services
class DashboardAggregator:
    async def get_dashboard(self, user_id: str) -> DashboardResponse:
        orders, notifications, recommendations = await asyncio.gather(
            self.order_service.recent_orders(user_id, limit=5),
            self.notification_service.unread(user_id),
            self.recommendation_service.for_user(user_id, limit=10),
        )
        return DashboardResponse(
            recent_orders=orders,
            unread_notifications=len(notifications),
            recommended_products=recommendations,
        )
```

### Common Mistakes
1. **Business logic in the gateway** — The gateway should route and apply policies, not compute discounts or validate domain rules
2. **Single BFF for all clients** — Defeats the purpose; each client type should have its own BFF
3. **Gateway as single point of failure** — Must be horizontally scaled and highly available
4. **Over-aggregation** — Aggregating too many backend calls in one gateway request increases latency and failure blast radius
5. **No circuit breakers** — If a backend service is down, the gateway should degrade gracefully, not cascade failures

### Integration with Other Topics
- **Microservices** — Gateway is the edge of the microservices architecture
- **API Design** — Gateway exposes the public API; backend services have internal APIs
- **Authentication and Authorization** — Gateway handles token validation and API key management
- **Caching Strategies** — Gateway can cache responses to reduce backend load
- **Resilience Patterns** — Circuit breakers, timeouts, and fallbacks at the gateway
- **Frontend Architecture** — BFF enables frontend teams to control their data needs

---

## Resources

### Essential Reading
- *Building Microservices* (2nd ed.) — Sam Newman (gateway and BFF chapters)
- *Microservices Patterns* — Chris Richardson (API gateway chapter)
- Phil Calçado — "The Back-end for Front-end Pattern (BFF)" article

### Online Resources
- Netflix Tech Blog on Zuul gateway
- Kong documentation and patterns
- AWS API Gateway developer guide

### Practice Exercises
1. Set up an API gateway routing to 3 backend services
2. Implement a BFF that aggregates data from 2 services for a mobile client
3. Add rate limiting and JWT authentication at the gateway level
4. Compare a single gateway vs. separate BFFs for web and mobile
5. Implement circuit breaker at the gateway for a failing backend service
