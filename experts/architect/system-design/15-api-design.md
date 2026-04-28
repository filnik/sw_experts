# API Design (REST, GraphQL, gRPC)

## Profile

### What It Is
API Design is the practice of defining how software components communicate with each other. The three dominant paradigms — REST, GraphQL, and gRPC — each offer distinct approaches to structuring requests, responses, and contracts between clients and servers.

### Origin and Evolution
- **REST**: Roy Fielding's doctoral dissertation (2000); became the default for web APIs
- **GraphQL**: Created by Facebook (2012), open-sourced (2015); solves over/under-fetching
- **gRPC**: Google's open-source RPC framework (2015); Protocol Buffers for efficient binary serialization
- Modern trend: coexistence — REST for public APIs, gRPC for internal services, GraphQL for frontend-driven queries

### Key Authors and References
- **Roy Fielding** — REST architectural style (dissertation, 2000)
- **Lee Byron, Dan Schafer** — GraphQL specification (Facebook)
- **Google** — gRPC and Protocol Buffers
- **Leonard Richardson** — Richardson Maturity Model for REST

### Key Concepts at a Glance
| Aspect | REST | GraphQL | gRPC |
|--------|------|---------|------|
| Protocol | HTTP | HTTP | HTTP/2 |
| Data Format | JSON/XML | JSON | Protobuf (binary) |
| Contract | OpenAPI/Swagger | Schema/SDL | .proto files |
| Typing | Weak (OpenAPI adds types) | Strong schema | Strong (proto) |
| Use Case | Public APIs, CRUD | Flexible queries, BFF | Internal services, streaming |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Design for the consumer** — APIs exist for their consumers. Design from the consumer's perspective, not the server's internal model.
2. **Contract-first design** — Define the API contract (OpenAPI, GraphQL schema, .proto) before implementation. The contract is the product.
3. **Consistency above all** — Naming conventions, error formats, pagination, versioning should be uniform across all endpoints.
4. **Versioning strategy upfront** — APIs will evolve. Plan for backward-compatible changes and have a versioning strategy from day one.
5. **Choose the right tool** — REST, GraphQL, and gRPC excel in different contexts. Choose based on requirements, not hype.

### When to Use vs. When to Avoid

**REST**: Public APIs, simple CRUD, broad ecosystem support, cacheable resources
**GraphQL**: Complex frontends needing flexible queries, mobile apps minimizing requests, aggregating multiple services
**gRPC**: High-performance internal services, streaming, polyglot microservices, low-latency requirements

---

## Frameworks and Methodologies

### 1. REST API Design — step-by-step
1. Identify resources (nouns): /users, /orders, /products
2. Map CRUD operations to HTTP methods (GET, POST, PUT, PATCH, DELETE)
3. Design URL hierarchy reflecting resource relationships
4. Define consistent error response format
5. Add pagination, filtering, sorting for collections
6. Write OpenAPI specification
7. Plan versioning (URL path, header, or query param)

### 2. GraphQL Schema Design — step-by-step
1. Model the domain as types (entities, value objects)
2. Define queries (read operations) and mutations (write operations)
3. Design input types for mutations
4. Add connections/edges for paginated relationships
5. Implement resolvers with DataLoader for N+1 prevention
6. Add field-level authorization
7. Generate client types from schema

---

## How to Apply

### Decision Checklist
- [ ] Is the API contract defined before implementation?
- [ ] Are naming conventions consistent (plural nouns for REST, camelCase for GraphQL)?
- [ ] Is there a versioning strategy?
- [ ] Are error responses structured and consistent?
- [ ] Is authentication/authorization documented?
- [ ] Is pagination implemented for collections?

### Implementation Patterns

**REST Resource Design:**
```
GET    /api/v1/orders              → List orders (paginated)
GET    /api/v1/orders/123          → Get order by ID
POST   /api/v1/orders              → Create order
PATCH  /api/v1/orders/123          → Partial update
DELETE /api/v1/orders/123          → Cancel/delete order
GET    /api/v1/orders/123/items    → List items of an order

Query params for filtering/sorting:
GET /api/v1/orders?status=active&sort=-created_at&page=2&size=20

Consistent error format:
{
  "error": {
    "code": "ORDER_NOT_FOUND",
    "message": "Order with ID 123 was not found",
    "details": []
  }
}
```

**GraphQL Schema:**
```graphql
type Query {
  order(id: ID!): Order
  orders(filter: OrderFilter, first: Int, after: String): OrderConnection!
}

type Mutation {
  placeOrder(input: PlaceOrderInput!): PlaceOrderPayload!
  cancelOrder(id: ID!): CancelOrderPayload!
}

type Order {
  id: ID!
  customer: Customer!
  items: [OrderItem!]!
  total: Money!
  status: OrderStatus!
  createdAt: DateTime!
}

type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
}

input PlaceOrderInput {
  customerId: ID!
  items: [OrderItemInput!]!
}

type PlaceOrderPayload {
  order: Order
  errors: [UserError!]!
}
```

**gRPC Service Definition:**
```protobuf
syntax = "proto3";
package orders.v1;

service OrderService {
  rpc PlaceOrder(PlaceOrderRequest) returns (PlaceOrderResponse);
  rpc GetOrder(GetOrderRequest) returns (Order);
  rpc ListOrders(ListOrdersRequest) returns (ListOrdersResponse);
  rpc StreamOrderUpdates(StreamRequest) returns (stream OrderUpdate);
}

message PlaceOrderRequest {
  string customer_id = 1;
  repeated OrderItem items = 2;
}

message Order {
  string id = 1;
  string customer_id = 2;
  repeated OrderItem items = 3;
  Money total = 4;
  OrderStatus status = 5;
}
```

**API Versioning Strategies:**
```
URL Path:    /api/v1/orders, /api/v2/orders
  + Explicit, easy to route
  - URL pollution, hard to sunset

Header:      Accept: application/vnd.myapp.v2+json
  + Clean URLs
  - Less discoverable

Query Param: /api/orders?version=2
  + Simple
  - Caching complications

Best practice: URL path for major versions, backward-compatible changes within a version
```

### Common Mistakes
1. **Verbs in REST URLs** — `/api/getOrders` instead of `GET /api/orders`; resources are nouns
2. **N+1 in GraphQL** — Not using DataLoader; each nested field triggers a database query
3. **Ignoring backward compatibility** — Breaking changes without versioning destroy client trust
4. **Over-fetching in REST** — Returning 50 fields when the client needs 3; consider sparse fieldsets or GraphQL
5. **No rate limiting** — Public APIs without rate limiting invite abuse
6. **Inconsistent error formats** — Different error structures across endpoints confuse consumers

### Integration with Other Topics
- **API Gateway and BFF** — Gateway routes, aggregates, and secures API traffic
- **Microservices** — APIs define service boundaries and contracts
- **Authentication and Authorization** — OAuth2, JWT, API keys protect API access
- **Caching** — REST resources are naturally cacheable; GraphQL requires different caching strategies
- **Contract Testing** — Pact, OpenAPI validation ensure API contracts aren't broken
- **System Design** — API design is a core system design decision

---

## Resources

### Essential Reading
- *RESTful Web APIs* — Richardson & Amundsen
- *Learning GraphQL* — Eve Porcello & Alex Banks
- *gRPC: Up and Running* — Kasun Indrasiri

### Online Resources
- OpenAPI Specification (swagger.io/specification)
- GraphQL official documentation (graphql.org)
- gRPC official documentation (grpc.io)
- Google API Design Guide

### Practice Exercises
1. Design a REST API for a library management system with OpenAPI spec
2. Convert a REST API with over-fetching problems to GraphQL
3. Define a gRPC service for a real-time chat system with bidirectional streaming
4. Implement API versioning for a breaking change in a REST API
5. Compare the same operation implemented in REST, GraphQL, and gRPC
