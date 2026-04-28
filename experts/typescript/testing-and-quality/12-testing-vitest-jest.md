# Testing with Vitest/Jest

## Profile

### What It Is
Testing in TypeScript uses Vitest (modern, Vite-native) or Jest (established, widely used) for unit and integration tests. Both provide typed test APIs, mocking capabilities, snapshot testing, and coverage reporting. Vitest is gaining dominance for new projects due to speed and ESM-native support.

### Origin and Evolution
- Mocha + Chai (2012) — Early JS test framework
- Jest (2014, Facebook) — All-in-one testing framework
- ts-jest (2016) — TypeScript support for Jest
- Vitest (2022) — Vite-powered, ESM-native, Jest-compatible API
- Current: Vitest for new projects; Jest for existing projects and React Native

### Key Authors and References
- **Christoph Nakazawa** — Jest creator
- **Vladimir Sheremet** — Vitest maintainer
- **Vitest documentation** — Comprehensive reference
- **Kent C. Dodds** — Testing philosophy (Testing Library)

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Vitest | Vite-native, ESM support, fast HMR, Jest-compatible API |
| Jest | Established, huge ecosystem, CJS-first |
| describe/it/expect | BDD-style test structure |
| Mocking | `vi.fn()` / `jest.fn()`, module mocking |
| Snapshot | `expect(x).toMatchSnapshot()` — serialize and compare |
| Coverage | c8 (Vitest) or istanbul (Jest) for code coverage |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Test behavior, not implementation** — Assert on outputs and observable effects. Don't assert on internal method calls or private state.
2. **Vitest for new projects** — ESM-native, fast, Vite-integrated, Jest-compatible API. Switching from Jest is low-cost.
3. **Typed test utilities** — Use TypeScript in tests. Type your mocks, fixtures, and helpers. Tests are code too.
4. **Arrange-Act-Assert** — Structure tests clearly: set up data, execute the action, verify the result. One assertion concept per test.
5. **Fast tests enable TDD** — Vitest's watch mode with HMR makes test-driven development practical. Tests should run in milliseconds.

### When to Use vs. When to Avoid

**Vitest**: New TypeScript projects, Vite-based apps, ESM projects
**Jest**: Existing Jest projects, React Native, projects with heavy Jest plugin usage
**Neither**: E2E tests (use Playwright), component visual tests (use Storybook/Chromatic)

---

## Frameworks and Methodologies

### 1. Test Suite Setup — step-by-step
1. Install: `vitest` (or `jest` + `ts-jest`)
2. Configure in `vitest.config.ts` (or `jest.config.ts`)
3. Create test files: `*.test.ts` or `*.spec.ts`
4. Set up shared fixtures and test utilities
5. Configure coverage reporting
6. Add to CI pipeline

### 2. Test Organization — step-by-step
1. Colocate tests with source: `order-service.test.ts` next to `order-service.ts`
2. Or separate: `tests/unit/`, `tests/integration/`
3. Use `describe` blocks for grouping related tests
4. Use `beforeEach`/`afterEach` for setup/teardown
5. Use test factories for creating test data

---

## How to Apply

### Decision Checklist
- [ ] Are tests colocated or in a consistent test directory?
- [ ] Is AAA (Arrange-Act-Assert) structure followed?
- [ ] Are mocks typed and minimal?
- [ ] Is coverage configured and tracked?
- [ ] Do tests run in CI on every push?

### Implementation Patterns

**Basic Vitest Tests:**
```typescript
// order-service.test.ts
import { describe, it, expect, beforeEach } from "vitest";
import { OrderService } from "./order-service";
import { FakeOrderRepository } from "../test-utils/fakes";

describe("OrderService", () => {
  let service: OrderService;
  let repo: FakeOrderRepository;

  beforeEach(() => {
    repo = new FakeOrderRepository();
    service = new OrderService(repo);
  });

  describe("createOrder", () => {
    it("creates order with correct total", () => {
      const order = service.createOrder({
        items: [
          { productId: "p1", quantity: 2, price: 10 },
          { productId: "p2", quantity: 1, price: 5 },
        ],
      });

      expect(order.total).toBe(25);
      expect(order.status).toBe("draft");
    });

    it("requires at least one item", () => {
      expect(() => service.createOrder({ items: [] })).toThrow("at least one item");
    });
  });

  describe("cancelOrder", () => {
    it("sets status to cancelled", () => {
      const order = service.createOrder({ items: [{ productId: "p1", quantity: 1, price: 10 }] });

      service.cancelOrder(order.id);

      expect(repo.findById(order.id)?.status).toBe("cancelled");
    });
  });
});
```

**Typed Mocks:**
```typescript
import { vi, describe, it, expect } from "vitest";

// Mock function with type
const sendEmail = vi.fn<[to: string, subject: string, body: string], Promise<void>>();

// Mock implementation
sendEmail.mockResolvedValue(undefined);

// Use in test
await service.notify("user@example.com", "Hello");
expect(sendEmail).toHaveBeenCalledWith("user@example.com", "Notification", expect.any(String));

// Mock module
vi.mock("./email-service", () => ({
  sendEmail: vi.fn().mockResolvedValue(undefined),
}));

// Spy on existing method
const spy = vi.spyOn(service, "calculateTotal");
service.createOrder(input);
expect(spy).toHaveReturnedWith(25);
```

**Test Factories:**
```typescript
// test-utils/factories.ts
import { faker } from "@faker-js/faker";

function createUser(overrides?: Partial<User>): User {
  return {
    id: faker.string.uuid(),
    name: faker.person.fullName(),
    email: faker.internet.email(),
    role: "user",
    createdAt: new Date(),
    ...overrides,
  };
}

function createOrder(overrides?: Partial<Order>): Order {
  return {
    id: faker.string.uuid(),
    customerId: faker.string.uuid(),
    items: [createOrderItem()],
    status: "draft",
    total: 0,
    createdAt: new Date(),
    ...overrides,
  };
}

// Usage in tests
it("admin can delete any order", () => {
  const admin = createUser({ role: "admin" });
  const order = createOrder({ customerId: "other-user" });

  const result = service.deleteOrder(order.id, admin);

  expect(result.success).toBe(true);
});
```

**Vitest Configuration:**
```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";
import path from "path";

export default defineConfig({
  test: {
    globals: true,
    environment: "node",  // or "jsdom" for browser tests
    include: ["src/**/*.test.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      include: ["src/**/*.ts"],
      exclude: ["src/**/*.test.ts", "src/**/*.d.ts"],
      thresholds: {
        branches: 80,
        functions: 80,
        lines: 80,
      },
    },
    setupFiles: ["./src/test-setup.ts"],
  },
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "src"),
    },
  },
});
```

**Async Tests:**
```typescript
describe("API client", () => {
  it("fetches user data", async () => {
    const user = await apiClient.getUser("123");

    expect(user).toEqual(expect.objectContaining({
      id: "123",
      name: expect.any(String),
    }));
  });

  it("throws on network error", async () => {
    vi.spyOn(global, "fetch").mockRejectedValueOnce(new Error("Network error"));

    await expect(apiClient.getUser("123")).rejects.toThrow("Network error");
  });
});
```

### Common Mistakes
1. **Testing implementation** — Asserting mock was called 3 times instead of checking the result
2. **Snapshot overuse** — Snapshot everything; snapshots become noise; use for serialized output only
3. **Flaky tests** — Tests depending on order, time, or shared state; isolate with beforeEach
4. **No test factories** — Building test data inline in every test; extract to reusable factories
5. **Slow tests** — Not mocking external services in unit tests; unit tests should be milliseconds

### Integration with Other Topics
- **Mocking** — vi.fn/vi.mock for test doubles
- **E2E Testing** — Playwright for browser testing
- **CI/CD** — Tests run on every push/PR
- **TypeScript** — Typed tests, typed mocks, typed assertions
- **Code Quality** — Coverage thresholds in CI

---

## Resources

### Essential Reading
- Vitest documentation (vitest.dev)
- Jest documentation (jestjs.io)
- Kent C. Dodds — "Testing JavaScript" course

### Online Resources
- Testing Library documentation
- @faker-js/faker for test data
- Vitest examples repository

### Practice Exercises
1. Set up Vitest with coverage thresholds for a TypeScript project
2. Write tests with typed factories using @faker-js/faker
3. Mock an HTTP module and test error handling
4. Migrate a Jest test suite to Vitest
5. Configure CI to run tests and report coverage
