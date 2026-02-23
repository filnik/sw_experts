# Contract Testing

## Profile

### What It Is
Contract testing verifies that the communication between services (API provider and consumer) conforms to an agreed contract. It ensures API changes don't break consumers without requiring full integration tests. Tools include Pact (consumer-driven contracts), MSW (Mock Service Worker), and OpenAPI codegen for type-safe API clients.

### Origin and Evolution
- Integration testing — Full stack tests, slow and brittle
- Pact (2013) — Consumer-driven contract testing framework
- OpenAPI/Swagger — API specification as contract
- MSW (2019) — Mock Service Worker for API mocking in tests and development
- OpenAPI codegen — Generate typed clients from API specs
- Current: MSW for mocking + OpenAPI codegen for type safety, Pact for microservices

### Key Authors and References
- **Pact Foundation** — Consumer-driven contracts
- **Artem Zakharchenko (kettanaito)** — MSW creator
- **OpenAPI Initiative** — API specification standard
- **Sam Newman** — Contract testing in *Building Microservices*

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Consumer-Driven Contract | Consumer defines expected API shape; provider verifies |
| MSW | Intercepts HTTP at network level; mock API for tests/dev |
| OpenAPI Codegen | Generate typed API clients from OpenAPI spec |
| Pact | Contract testing framework; generates and verifies contracts |
| Schema-First | Define API schema → generate code (types, validators) |
| Provider Verification | Provider runs consumer contracts to verify compatibility |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Contracts prevent integration bugs** — Agreeing on the API shape prevents "works on my machine" integration failures. Schema is the contract.
2. **MSW for consistent mocking** — Mock APIs at the network level, not at the function level. Same mocks work in tests, Storybook, and development.
3. **Generate, don't handwrite** — Generate TypeScript types and API clients from OpenAPI specs. Manual types drift from the actual API.
4. **Consumer-driven for microservices** — With Pact, the consumer says "I need these fields." The provider verifies it can deliver. Both teams have confidence.
5. **Test the contract, not the implementation** — Verify the shape and status codes, not business logic. Integration tests cover logic; contracts cover communication.

### When to Use vs. When to Avoid

**Contract testing when:**
- Frontend and backend developed by different teams
- Microservices communicating via APIs
- Breaking API changes would be costly
- Need to mock APIs consistently across tests/dev

**Skip contract testing when:**
- Monolith with no API boundaries
- Same team owns both sides
- tRPC (end-to-end type safety eliminates the need)

---

## Frameworks and Methodologies

### 1. MSW Setup — step-by-step
1. Install MSW: `npm install msw --save-dev`
2. Define request handlers (GET /api/users → mock response)
3. Set up MSW server for Node.js tests
4. Set up MSW worker for browser dev/Storybook
5. Share handlers between tests and development
6. Use `http.get()`, `http.post()` for typed handlers

### 2. OpenAPI Contract Workflow — step-by-step
1. Define or generate OpenAPI spec (YAML/JSON)
2. Generate TypeScript types: `openapi-typescript`
3. Generate API client: `openapi-fetch` or `orval`
4. Use generated types in frontend code
5. Validate API responses in tests against schema
6. Regenerate types when API spec changes

---

## How to Apply

### Decision Checklist
- [ ] Are API contracts defined (OpenAPI, Pact, or shared types)?
- [ ] Is MSW used for API mocking in tests?
- [ ] Are TypeScript types generated from API specs?
- [ ] Are contracts verified in CI (provider and consumer)?
- [ ] Is MSW used in Storybook for component development?

### Implementation Patterns

**MSW Handlers:**
```typescript
// mocks/handlers.ts
import { http, HttpResponse } from "msw";

export const handlers = [
  http.get("/api/users/:userId", ({ params }) => {
    const { userId } = params;
    return HttpResponse.json({
      id: userId,
      name: "Alice",
      email: "alice@example.com",
    });
  }),

  http.post("/api/orders", async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json(
      { id: "order-123", ...body, status: "created" },
      { status: 201 },
    );
  }),

  http.get("/api/orders", ({ request }) => {
    const url = new URL(request.url);
    const status = url.searchParams.get("status");
    return HttpResponse.json([
      { id: "1", status: status ?? "all", total: 99.99 },
    ]);
  }),

  // Error scenario
  http.get("/api/users/not-found", () => {
    return HttpResponse.json(
      { error: "User not found" },
      { status: 404 },
    );
  }),
];
```

**MSW in Tests:**
```typescript
// test-setup.ts
import { setupServer } from "msw/node";
import { handlers } from "./mocks/handlers";

export const server = setupServer(...handlers);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// In tests
import { server } from "../test-setup";
import { http, HttpResponse } from "msw";

test("displays user name", async () => {
  render(<UserProfile userId="123" />);

  await expect(screen.findByText("Alice")).resolves.toBeInTheDocument();
});

test("handles error state", async () => {
  // Override for this test
  server.use(
    http.get("/api/users/:userId", () => {
      return HttpResponse.json({ error: "Server error" }, { status: 500 });
    }),
  );

  render(<UserProfile userId="123" />);

  await expect(screen.findByText("Something went wrong")).resolves.toBeInTheDocument();
});
```

**OpenAPI Type Generation:**
```typescript
// Generate types from OpenAPI spec
// npx openapi-typescript ./api-spec.yaml -o ./src/api/schema.d.ts

// Use with openapi-fetch
import createClient from "openapi-fetch";
import type { paths } from "./api/schema";

const api = createClient<paths>({ baseUrl: "https://api.example.com" });

// Fully typed: params, body, response
const { data, error } = await api.GET("/users/{userId}", {
  params: { path: { userId: "123" } },
});

if (data) {
  // data is typed as the 200 response schema
  console.log(data.name);
}

// POST with typed body
const { data: order } = await api.POST("/orders", {
  body: {
    customerId: "c123",
    items: [{ productId: "p1", quantity: 2 }],
  },
});
```

**Pact Consumer Test:**
```typescript
import { PactV3 } from "@pact-foundation/pact";

const provider = new PactV3({
  consumer: "WebApp",
  provider: "OrderService",
});

describe("Order API contract", () => {
  it("returns order by ID", async () => {
    await provider
      .given("order 123 exists")
      .uponReceiving("a request for order 123")
      .withRequest({
        method: "GET",
        path: "/api/orders/123",
      })
      .willRespondWith({
        status: 200,
        body: {
          id: "123",
          status: "confirmed",
          total: 99.99,
        },
      });

    await provider.executeTest(async (mockServer) => {
      const client = createApiClient(mockServer.url);
      const order = await client.getOrder("123");

      expect(order.id).toBe("123");
      expect(order.status).toBe("confirmed");
    });
  });
});
// Generates a pact file; provider runs it to verify
```

### Common Mistakes
1. **Not using MSW** — Mocking at the function level (`jest.mock("axios")`); MSW mocks the network, more realistic
2. **Handwriting API types** — Manually defining TypeScript types for API responses; drift from actual API
3. **Testing logic in contracts** — Contract tests verify shape, not business logic; keep them focused
4. **No CI verification** — Contracts not checked in CI; contracts drift without automated verification
5. **MSW handlers too specific** — One handler per test; share base handlers, override for specific tests

### Integration with Other Topics
- **API Design** — OpenAPI spec defines the contract
- **Testing** — MSW integrates with Vitest/Jest for API mocking
- **React** — MSW mocks APIs for component tests
- **Microservices** — Pact for cross-service contract verification
- **TypeScript** — Generated types ensure type-safe API consumption

---

## Resources

### Essential Reading
- MSW documentation (mswjs.io)
- Pact documentation (docs.pact.io)
- openapi-typescript documentation

### Online Resources
- openapi-fetch documentation
- Orval — OpenAPI client generator (orval.dev)
- Kent C. Dodds — "Stop Mocking Fetch" (MSW advocacy)

### Practice Exercises
1. Set up MSW with handlers for 3 API endpoints and use in tests
2. Generate TypeScript types from an OpenAPI spec with openapi-typescript
3. Create a typed API client with openapi-fetch
4. Write Pact consumer tests for 2 API endpoints
5. Use MSW in Storybook for component development without backend
