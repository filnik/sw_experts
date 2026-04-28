# TypeScript Design Patterns

## Profile

### What It Is
Design patterns adapted to TypeScript's type system — leveraging generics, discriminated unions, branded types, and the `satisfies` operator. TypeScript's structural typing and type inference make many patterns more concise than in Java/C# while adding type safety that plain JavaScript lacks.

### Origin and Evolution
- GoF *Design Patterns* (1994) — original catalog
- JavaScript patterns — prototype-based, closure-based approaches
- TypeScript brings nominal-like typing (branded types), generics, interfaces
- Current: patterns leverage TypeScript's advanced types for compile-time safety

### Key Authors and References
- **Addy Osmani** — *Learning JavaScript Design Patterns*
- **refactoring.guru** — Patterns with TypeScript examples
- **Matt Pocock** — TypeScript-specific patterns
- **Dan Vanderkam** — *Effective TypeScript* patterns

### Key Concepts at a Glance
| Pattern | TypeScript Approach |
|---------|---------------------|
| Strategy | Generic interface or function type |
| Builder | Chained methods with incremental type narrowing |
| Factory | Discriminated union + factory function |
| Observer | Generic EventEmitter with typed events |
| Repository | Generic interface with type parameter |
| State Machine | Discriminated unions + exhaustive switch |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Types encode the pattern** — Use the type system to enforce pattern rules. A Builder that requires all fields uses types to prevent incomplete construction.
2. **Discriminated unions for variant types** — Factory, State, Command — any pattern with variants uses discriminated unions.
3. **Generics for reusable patterns** — Repository<T>, EventEmitter<Events>, Result<T, E> — generics make patterns reusable.
4. **Branded types for domain safety** — `UserId` vs `OrderId` vs `string` — branded types prevent mixing up identifiers.
5. **Functions over classes when possible** — TypeScript supports both. Use functions for Strategy and Factory; classes when state management requires it.

### When to Use vs. When to Avoid

**Use typed patterns when:**
- Type safety catches real bugs (wrong ID type, missing fields)
- The pattern is used across multiple features
- Team benefits from enforced structure

**Keep it simple when:**
- A plain function or object suffices
- The pattern adds types without adding safety
- One-off usage doesn't justify the abstraction

---

## Frameworks and Methodologies

### 1. Pattern Selection for TypeScript — step-by-step
1. Identify the problem (creation, behavior, structure)
2. Consider type-level solutions (discriminated unions, branded types)
3. Use generics for reusable patterns
4. Implement with minimal class hierarchy
5. Test that types prevent misuse at compile time

### 2. Type-Safe Builder Pattern — step-by-step
1. Define the target type with required and optional fields
2. Create builder with generic state tracking
3. Each builder method narrows the type
4. `build()` is only available when all required fields are set
5. TypeScript enforces completeness at compile time

---

## How to Apply

### Decision Checklist
- [ ] Do types enforce pattern constraints at compile time?
- [ ] Are discriminated unions used for variant types?
- [ ] Are generics making the pattern reusable?
- [ ] Is the pattern simpler than the equivalent untyped code?

### Implementation Patterns

**Strategy Pattern:**
```typescript
// Strategy as generic function type
type SortStrategy<T> = (a: T, b: T) => number;

function sortItems<T>(items: T[], strategy: SortStrategy<T>): T[] {
  return [...items].sort(strategy);
}

// Strategies
const byPrice: SortStrategy<Product> = (a, b) => a.price - b.price;
const byName: SortStrategy<Product> = (a, b) => a.name.localeCompare(b.name);
const byRating: SortStrategy<Product> = (a, b) => b.rating - a.rating;

const sorted = sortItems(products, byPrice);

// Strategy with configuration
type DiscountStrategy = (total: number) => number;

const flatDiscount = (rate: number): DiscountStrategy =>
  (total) => total * rate;

const tieredDiscount: DiscountStrategy = (total) => {
  if (total > 1000) return total * 0.15;
  if (total > 500) return total * 0.10;
  return 0;
};
```

**Type-Safe Builder:**
```typescript
type RequiredFields = "name" | "email";

type BuilderState = {
  [K in RequiredFields]?: true;
};

class UserBuilder<State extends BuilderState = {}> {
  private data: Partial<User> = {};

  name(name: string): UserBuilder<State & { name: true }> {
    this.data.name = name;
    return this as any;
  }

  email(email: string): UserBuilder<State & { email: true }> {
    this.data.email = email;
    return this as any;
  }

  age(age: number): this {
    this.data.age = age;
    return this;
  }

  // build() only available when all required fields are set
  build(
    this: UserBuilder<{ [K in RequiredFields]: true }>,
  ): User {
    return this.data as User;
  }
}

// Usage
const user = new UserBuilder()
  .name("Alice")
  .email("alice@example.com")
  .age(30)
  .build(); // OK: all required fields set

// new UserBuilder().name("Alice").build(); // Error: email not set
```

**Typed Event Emitter:**
```typescript
type EventMap = {
  "order:created": { orderId: string; total: number };
  "order:cancelled": { orderId: string; reason: string };
  "user:login": { userId: string };
};

class TypedEventEmitter<Events extends Record<string, unknown>> {
  private listeners = new Map<string, Set<Function>>();

  on<K extends keyof Events>(
    event: K,
    handler: (payload: Events[K]) => void,
  ): void {
    const handlers = this.listeners.get(event as string) ?? new Set();
    handlers.add(handler);
    this.listeners.set(event as string, handlers);
  }

  emit<K extends keyof Events>(event: K, payload: Events[K]): void {
    const handlers = this.listeners.get(event as string);
    handlers?.forEach((handler) => handler(payload));
  }
}

const events = new TypedEventEmitter<EventMap>();

events.on("order:created", ({ orderId, total }) => {
  // orderId: string, total: number — fully typed
  console.log(`Order ${orderId}: $${total}`);
});

events.emit("order:created", { orderId: "123", total: 99.99 }); // OK
// events.emit("order:created", { orderId: "123" }); // Error: missing total
```

**State Machine with Discriminated Unions:**
```typescript
type OrderState =
  | { status: "draft"; items: CartItem[] }
  | { status: "confirmed"; items: CartItem[]; confirmedAt: Date }
  | { status: "shipped"; items: CartItem[]; confirmedAt: Date; trackingId: string }
  | { status: "delivered"; items: CartItem[]; confirmedAt: Date; trackingId: string; deliveredAt: Date };

// Type-safe transitions
function confirmOrder(order: Extract<OrderState, { status: "draft" }>): Extract<OrderState, { status: "confirmed" }> {
  return { ...order, status: "confirmed", confirmedAt: new Date() };
}

function shipOrder(
  order: Extract<OrderState, { status: "confirmed" }>,
  trackingId: string,
): Extract<OrderState, { status: "shipped" }> {
  return { ...order, status: "shipped", trackingId };
}
// Can't ship a draft order — type error at compile time
```

**Repository Pattern:**
```typescript
interface Repository<T extends { id: string }> {
  findById(id: string): Promise<T | null>;
  findAll(filter?: Partial<T>): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: string): Promise<void>;
}

class InMemoryRepository<T extends { id: string }> implements Repository<T> {
  private store = new Map<string, T>();

  async findById(id: string): Promise<T | null> {
    return this.store.get(id) ?? null;
  }

  async findAll(filter?: Partial<T>): Promise<T[]> {
    const items = [...this.store.values()];
    if (!filter) return items;
    return items.filter((item) =>
      Object.entries(filter).every(([key, value]) => item[key as keyof T] === value),
    );
  }

  async save(entity: T): Promise<T> {
    this.store.set(entity.id, entity);
    return entity;
  }

  async delete(id: string): Promise<void> {
    this.store.delete(id);
  }
}

// Usage: fully typed
const orderRepo: Repository<Order> = new InMemoryRepository<Order>();
const order = await orderRepo.findById("123"); // Order | null
```

### Common Mistakes
1. **Class hierarchies like Java** — Deep inheritance chains; use composition and interfaces
2. **Patterns without type enforcement** — Pattern that doesn't leverage types; just complexity without safety
3. **Overusing classes** — Class with no state; a function or object literal is simpler
4. **Ignoring discriminated unions** — Boolean flags for state instead of union types
5. **Not using generics** — Duplicating patterns for different types instead of parameterizing

### Integration with Other Topics
- **TypeScript Type System** — Patterns leverage discriminated unions, generics, branded types
- **Advanced Types** — Conditional and mapped types for pattern type safety
- **Error Handling** — Result/Either pattern uses discriminated unions
- **React** — Typed component patterns (render props, HOCs, hooks)
- **Testing** — Patterns enable testability through interfaces

---

## Resources

### Essential Reading
- *Effective TypeScript* — Dan Vanderkam
- *Learning JavaScript Design Patterns* — Addy Osmani
- refactoring.guru — TypeScript pattern examples

### Online Resources
- Total TypeScript — Pattern sections
- TypeScript design patterns (patterns.dev)
- type-challenges (github.com/type-challenges)

### Practice Exercises
1. Implement a type-safe Builder that requires all fields at compile time
2. Create a typed EventEmitter with an event map
3. Model an order lifecycle as a state machine with discriminated unions
4. Build a generic Repository interface with two implementations
5. Implement Strategy pattern with generics and factory function
