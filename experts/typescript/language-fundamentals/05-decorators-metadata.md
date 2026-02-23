# TypeScript Decorators and Metadata

## Profile

### What It Is
TypeScript decorators are functions that modify or annotate classes, methods, properties, and parameters at definition time. The TC39 Stage 3 decorators (TypeScript 5.0+) are the standard; experimental/legacy decorators (`experimentalDecorators`) remain for frameworks like NestJS and TypeORM that depend on them.

### Origin and Evolution
- Experimental decorators (TypeScript 1.5, 2015) — Based on early TC39 proposal
- `reflect-metadata` library — Runtime type metadata for experimental decorators
- Angular, NestJS, TypeORM — Built on experimental decorators
- TC39 Stage 3 decorators (2022) — Standard proposal, different API
- TypeScript 5.0 (2023) — TC39 decorators implemented
- Current: TC39 decorators are the standard; experimental decorators for legacy frameworks

### Key Authors and References
- **TC39 Decorators Proposal** — Daniel Ehrenberg, Chris Garrett
- **TypeScript team** — TC39 decorator implementation
- **NestJS / Angular** — Major users of experimental decorators
- **Ron Buckton** — `reflect-metadata` and TypeScript decorator design

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| TC39 Decorators | Standard decorators (TS 5.0+); `@decorator` modifies class elements |
| Experimental Decorators | Legacy: `experimentalDecorators` + `emitDecoratorMetadata` |
| Class Decorator | Modifies or replaces a class definition |
| Method Decorator | Wraps a method with additional behavior |
| `reflect-metadata` | Polyfill for runtime type metadata (experimental only) |
| Decorator Factory | Function that returns a decorator (for parameterized decorators) |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **TC39 for new code** — Use standard TC39 decorators for new projects. They're the future and don't require `experimentalDecorators`.
2. **Experimental for framework compatibility** — NestJS, Angular, TypeORM require experimental decorators. Use them when the framework requires it.
3. **Decorators for cross-cutting concerns** — Logging, validation, authorization, caching. Don't use decorators for core business logic.
4. **Don't mix decorator styles** — TC39 and experimental decorators have different APIs. A project should use one or the other.
5. **Metadata for DI frameworks** — `reflect-metadata` enables constructor parameter type reflection for dependency injection. This is experimental-only.

### When to Use vs. When to Avoid

**Use TC39 decorators when:**
- New projects without framework constraints
- Adding behavior to classes/methods (logging, memoization)
- Building custom frameworks

**Use experimental decorators when:**
- NestJS, Angular, TypeORM, or other frameworks require them
- Need `reflect-metadata` for DI

**Avoid decorators when:**
- Simple function composition suffices
- The decorator hides critical behavior
- The team isn't familiar with decorator patterns

---

## Frameworks and Methodologies

### 1. TC39 Decorator Design — step-by-step
1. Define the behavior to add (logging, timing, validation)
2. Write decorator function with correct signature (context parameter)
3. Return modified value or replacement
4. Use decorator factories for parameters
5. Test decorator independently

### 2. Experimental Decorator Migration — step-by-step
1. Identify experimental decorators in the project
2. Check if the framework supports TC39 decorators
3. If yes: rewrite decorators using TC39 API
4. If no: keep experimental decorators (NestJS, Angular)
5. Don't mix both styles in one project

---

## How to Apply

### Decision Checklist
- [ ] Is the decorator style (TC39 vs. experimental) consistent across the project?
- [ ] Are decorators used for cross-cutting concerns only?
- [ ] Are decorator factories used for parameterized behavior?
- [ ] Is `experimentalDecorators` only enabled when needed by framework?
- [ ] Are decorators tested independently?

### Implementation Patterns

**TC39 Decorators (TypeScript 5.0+):**
```typescript
// Method decorator: logging
function logged<T extends (...args: any[]) => any>(
  target: T,
  context: ClassMethodDecoratorContext,
): T {
  const methodName = String(context.name);

  function replacement(this: any, ...args: any[]) {
    console.log(`Calling ${methodName} with`, args);
    const result = target.call(this, ...args);
    console.log(`${methodName} returned`, result);
    return result;
  }

  return replacement as T;
}

class OrderService {
  @logged
  createOrder(items: CartItem[]): Order {
    return new Order(items);
  }
}

// Decorator factory with parameters
function retry(maxAttempts: number) {
  return function <T extends (...args: any[]) => any>(
    target: T,
    context: ClassMethodDecoratorContext,
  ): T {
    function replacement(this: any, ...args: any[]) {
      let lastError: Error;
      for (let i = 0; i < maxAttempts; i++) {
        try {
          return target.call(this, ...args);
        } catch (e) {
          lastError = e as Error;
        }
      }
      throw lastError!;
    }
    return replacement as T;
  };
}

class ApiClient {
  @retry(3)
  fetchData(url: string): Response {
    return fetch(url);
  }
}
```

**Class Decorator (TC39):**
```typescript
// Class decorator: add metadata
function sealed(
  target: Function,
  context: ClassDecoratorContext,
) {
  Object.seal(target);
  Object.seal(target.prototype);
}

// Class decorator: add initialization logic
function withTimestamp<T extends new (...args: any[]) => any>(
  target: T,
  context: ClassDecoratorContext,
) {
  context.addInitializer(function (this: any) {
    this.createdAt = new Date();
  });
}

@sealed
@withTimestamp
class Order {
  id: string;
  createdAt!: Date;

  constructor(id: string) {
    this.id = id;
  }
}
```

**Experimental Decorators (NestJS style):**
```typescript
// Enable in tsconfig: "experimentalDecorators": true, "emitDecoratorMetadata": true

// NestJS Controller
import { Controller, Get, Post, Body, Param, UseGuards } from "@nestjs/common";

@Controller("orders")
@UseGuards(AuthGuard)
class OrderController {
  constructor(private readonly orderService: OrderService) {}

  @Get()
  async findAll(): Promise<Order[]> {
    return this.orderService.findAll();
  }

  @Post()
  async create(@Body() dto: CreateOrderDto): Promise<Order> {
    return this.orderService.create(dto);
  }

  @Get(":id")
  async findOne(@Param("id") id: string): Promise<Order> {
    return this.orderService.findOne(id);
  }
}

// Custom decorator (experimental)
function Log(): MethodDecorator {
  return function (target, propertyKey, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    descriptor.value = function (...args: any[]) {
      console.log(`${String(propertyKey)} called with`, args);
      return original.apply(this, args);
    };
  };
}
```

**Auto-accessor Decorator (TC39):**
```typescript
// TC39 auto-accessor decorator
function validate(min: number, max: number) {
  return function (
    target: ClassAccessorDecoratorTarget<any, number>,
    context: ClassAccessorDecoratorContext,
  ): ClassAccessorDecoratorResult<any, number> {
    return {
      set(value: number) {
        if (value < min || value > max) {
          throw new RangeError(`${String(context.name)} must be between ${min} and ${max}`);
        }
        target.set.call(this, value);
      },
      get() {
        return target.get.call(this);
      },
    };
  };
}

class Product {
  @validate(0, 10000)
  accessor price: number = 0;

  @validate(0, 999)
  accessor quantity: number = 0;
}
```

### Common Mistakes
1. **Mixing TC39 and experimental** — Different APIs, different behavior; choose one per project
2. **Decorators for business logic** — Hiding critical logic in decorators; use for cross-cutting only
3. **Relying on `emitDecoratorMetadata` with TC39** — Metadata emission is experimental-only
4. **Over-decorating** — 5 decorators on a method; unclear execution order and behavior
5. **Not testing decorators** — Decorators are functions; test them independently

### Integration with Other Topics
- **Dependency Injection** — NestJS, tsyringe use decorators for DI registration
- **Design Patterns** — Decorator pattern implemented as TypeScript decorators
- **TypeScript Configuration** — `experimentalDecorators` and `emitDecoratorMetadata` flags
- **Testing** — Decorators need independent testing; they're reusable functions
- **Web Frameworks** — NestJS, Angular heavily use experimental decorators

---

## Resources

### Essential Reading
- TC39 Decorators proposal (github.com/tc39/proposal-decorators)
- TypeScript 5.0 release notes (decorators section)
- NestJS documentation — Custom decorators

### Online Resources
- TypeScript Handbook — Decorators section
- Total TypeScript — Decorators guide
- Angular documentation — Decorators

### Practice Exercises
1. Write a TC39 `@logged` method decorator with timing
2. Create a `@retry(n)` decorator factory for error retry
3. Build a validation decorator for class accessor properties
4. Compare TC39 and experimental decorator implementations side by side
5. Write unit tests for a custom decorator function
