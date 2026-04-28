# Dependency Injection in TypeScript

## Profile

### What It Is
Dependency Injection (DI) in TypeScript provides dependencies to classes and functions from the outside, enabling testability, loose coupling, and swappability. Approaches range from manual constructor injection to decorator-based containers (tsyringe, inversify) and framework-integrated DI (NestJS).

### Origin and Evolution
- Java/C# DI containers (Spring, .NET DI) — established patterns
- TypeScript manual DI — constructor injection, factory functions
- InversifyJS (2015) — First major TypeScript DI container
- tsyringe (2018, Microsoft) — Lightweight decorator-based DI
- NestJS (2017) — Built-in DI system (Angular-inspired)
- Current: manual DI for simple cases; NestJS DI for NestJS projects; tsyringe for standalone

### Key Authors and References
- **Martin Fowler** — DI and IoC patterns
- **NestJS documentation** — Comprehensive DI guide
- **InversifyJS documentation** — Container-based DI
- **Mark Seemann** — *Dependency Injection: Principles, Practices, Patterns*

### Key Concepts at a Glance
| Approach | Description |
|----------|-------------|
| Constructor Injection | Dependencies passed to constructor |
| Factory Function | Function that creates wired instances |
| tsyringe | Decorator-based container: `@injectable()`, `@inject()` |
| NestJS Providers | Framework-managed DI with `@Injectable()` |
| Manual Wiring | Composition root creates and connects all dependencies |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Manual DI first** — TypeScript doesn't need a container for most projects. Pass interfaces through constructors. Wire at startup.
2. **Interfaces as contracts** — Depend on interfaces (TypeScript interfaces or abstract classes), not concrete implementations. Swap in tests and across configurations.
3. **Composition root** — All dependency creation and wiring happens in one place (app entry point). Don't create dependencies deep in the call chain.
4. **Container for complexity** — Use tsyringe or NestJS DI when you have 20+ services with complex dependency graphs. Manual DI scales to ~10-15 services.
5. **Scoped dependencies** — Request-scoped services (DB connections, auth context) need proper scoping. Don't share mutable state across requests.

### When to Use vs. When to Avoid

**Manual DI**: Small to medium projects, clear dependency graphs, no framework requirement
**NestJS DI**: NestJS projects (always — it's built in)
**tsyringe/inversify**: Large projects outside NestJS that need automatic resolution

---

## Frameworks and Methodologies

### 1. Manual DI Setup — step-by-step
1. Define interfaces for dependencies
2. Accept interfaces in constructors
3. Create a composition root function
4. Wire all dependencies at app startup
5. Pass pre-wired instances to route handlers
6. In tests, provide fakes/mocks directly

### 2. NestJS DI — step-by-step
1. Mark classes with `@Injectable()` decorator
2. Register as providers in module `providers` array
3. Inject via constructor parameters
4. Use `@Inject()` for non-class tokens
5. Configure scopes (singleton, request, transient)
6. Override providers in tests with `Testing.createTestingModule()`

---

## How to Apply

### Decision Checklist
- [ ] Are dependencies injected (not created internally)?
- [ ] Is there a composition root / module registration?
- [ ] Can dependencies be swapped for testing?
- [ ] Are interfaces used instead of concrete types?
- [ ] Are scoped dependencies properly managed?

### Implementation Patterns

**Manual Constructor Injection:**
```typescript
// Interfaces
interface UserRepository {
  findById(id: string): Promise<User | null>;
  save(user: User): Promise<User>;
}

interface EmailService {
  send(to: string, subject: string, body: string): Promise<void>;
}

// Service depends on interfaces
class UserService {
  constructor(
    private readonly userRepo: UserRepository,
    private readonly emailService: EmailService,
  ) {}

  async register(input: RegisterInput): Promise<User> {
    const user = User.create(input);
    const saved = await this.userRepo.save(user);
    await this.emailService.send(saved.email, "Welcome!", "...");
    return saved;
  }
}

// Composition root
function createApp(config: AppConfig) {
  const db = new PostgresPool(config.databaseUrl);
  const userRepo = new PostgresUserRepository(db);
  const emailService = new SmtpEmailService(config.smtpHost);
  const userService = new UserService(userRepo, emailService);

  const app = express();
  app.post("/users", createUserHandler(userService));
  return app;
}

// Test: inject fakes
describe("UserService", () => {
  it("sends welcome email on registration", async () => {
    const fakeRepo = new InMemoryUserRepository();
    const fakeEmail = new FakeEmailService();
    const service = new UserService(fakeRepo, fakeEmail);

    await service.register({ name: "Alice", email: "alice@example.com" });

    expect(fakeEmail.sentEmails).toHaveLength(1);
    expect(fakeEmail.sentEmails[0].to).toBe("alice@example.com");
  });
});
```

**NestJS DI:**
```typescript
import { Injectable, Module, Inject } from "@nestjs/common";

// Injectable service
@Injectable()
class OrderService {
  constructor(
    private readonly orderRepo: OrderRepository,
    @Inject("PAYMENT_GATEWAY") private readonly payment: PaymentGateway,
  ) {}

  async createOrder(input: CreateOrderInput): Promise<Order> {
    const order = Order.create(input);
    await this.payment.charge(order.total);
    return this.orderRepo.save(order);
  }
}

// Module registration
@Module({
  providers: [
    OrderService,
    OrderRepository,
    {
      provide: "PAYMENT_GATEWAY",
      useClass: StripePaymentGateway,
    },
  ],
  controllers: [OrderController],
})
class OrderModule {}

// Testing: override providers
const module = await Test.createTestingModule({
  providers: [
    OrderService,
    { provide: OrderRepository, useClass: InMemoryOrderRepository },
    { provide: "PAYMENT_GATEWAY", useClass: FakePaymentGateway },
  ],
}).compile();

const service = module.get(OrderService);
```

**tsyringe:**
```typescript
import { container, injectable, inject, singleton } from "tsyringe";

interface Logger {
  log(message: string): void;
}

@singleton()
class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(`[LOG] ${message}`);
  }
}

@injectable()
class OrderService {
  constructor(
    @inject("Logger") private readonly logger: Logger,
    @inject("OrderRepository") private readonly repo: OrderRepository,
  ) {}

  async create(input: CreateOrderInput): Promise<Order> {
    this.logger.log(`Creating order for ${input.customerId}`);
    const order = Order.create(input);
    return this.repo.save(order);
  }
}

// Registration
container.register("Logger", { useClass: ConsoleLogger });
container.register("OrderRepository", { useClass: PostgresOrderRepository });

// Resolution
const service = container.resolve(OrderService);
```

**Factory Pattern for Scoped Dependencies:**
```typescript
// Factory creates scoped instances per request
type ServiceFactory = (requestContext: RequestContext) => {
  userService: UserService;
  orderService: OrderService;
};

function createServiceFactory(config: AppConfig): ServiceFactory {
  const db = new PostgresPool(config.databaseUrl);

  return (ctx: RequestContext) => {
    const session = db.createSession();
    const userRepo = new PostgresUserRepository(session);
    const orderRepo = new PostgresOrderRepository(session);

    return {
      userService: new UserService(userRepo, ctx.currentUser),
      orderService: new OrderService(orderRepo, ctx.currentUser),
    };
  };
}

// Middleware creates scoped services per request
app.use((req, res, next) => {
  req.services = serviceFactory({ currentUser: req.user });
  next();
});
```

### Common Mistakes
1. **Creating dependencies inside classes** — `new Database()` in constructor; impossible to test
2. **Container for 5 services** — DI container overhead for a small dependency graph; manual DI suffices
3. **No composition root** — Dependencies created everywhere; no single view of the wiring
4. **Depending on concrete classes** — `UserService` depends on `PostgresRepository` instead of `Repository` interface
5. **Circular dependencies** — A depends on B, B depends on A; restructure or use lazy injection

### Integration with Other Topics
- **Clean Architecture** — DI enforces dependency inversion at layer boundaries
- **Testing** — DI enables testing with fakes without mocking frameworks
- **Design Patterns** — Strategy, Repository patterns rely on DI
- **NestJS** — Framework DI is built on TypeScript decorators
- **SOLID** — Dependency Inversion Principle

---

## Resources

### Essential Reading
- *Dependency Injection: Principles, Practices, Patterns* — Mark Seemann
- NestJS Providers documentation
- tsyringe documentation (github.com/microsoft/tsyringe)

### Online Resources
- InversifyJS documentation
- Martin Fowler — "Inversion of Control Containers and DI"
- NestJS testing documentation

### Practice Exercises
1. Refactor a class with internal dependencies to constructor injection
2. Build a composition root for a 3-layer Express application
3. Set up NestJS DI with custom providers and testing overrides
4. Implement scoped services per HTTP request
5. Compare manual DI vs tsyringe for the same 10-service application
