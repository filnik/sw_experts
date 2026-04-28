# Node.js + TypeScript

## Profile

### What It Is
Node.js + TypeScript covers building type-safe server-side applications — typed Express/Fastify routes, middleware typing, tRPC for end-to-end type safety, and best practices for structuring Node.js backends with TypeScript.

### Origin and Evolution
- Node.js (2009) — JavaScript on the server
- Express.js (2010) — Minimal web framework
- ts-node (2015) — TypeScript execution for Node.js
- Fastify (2017) — Performance-focused, TypeScript-friendly
- tRPC (2021) — End-to-end type-safe APIs without code generation
- tsx (2022) — Fast TypeScript execution (esbuild-based)
- Current: Fastify or Express with typed routes; tRPC for full-stack TypeScript

### Key Authors and References
- **Matteo Collina** — Fastify creator
- **Alex / KATT** — tRPC creator
- **Colin McDonnell** — Zod (used with tRPC)
- **Fastify documentation** — Type system integration

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Typed Routes | Route handlers with typed request/response |
| Middleware Typing | Typed middleware chain adding to request context |
| tRPC | End-to-end type safety between client and server |
| Fastify | Schema-based validation with type inference |
| tsx | Fast TypeScript execution without precompilation |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Type the entire request/response cycle** — From route params and query strings to request body and response shape. No `any` in HTTP handlers.
2. **Fastify for type safety** — Fastify's schema system generates TypeScript types from JSON Schema. Express requires manual typing.
3. **tRPC for full-stack TypeScript** — When client and server are both TypeScript, tRPC provides end-to-end type safety without OpenAPI/codegen.
4. **tsx for development** — Use `tsx` (esbuild-based) for development. It's fast and handles TypeScript directly. Use `tsc` for type checking separately.
5. **Validate inputs, type everything else** — Use Zod at the API boundary to validate external input. Internal code uses TypeScript types directly.

### When to Use vs. When to Avoid

**tRPC**: Full-stack TypeScript (Next.js, monorepo with shared types)
**Fastify**: High-performance APIs, schema-driven development
**Express**: Existing projects, large ecosystem of middleware
**Hono**: Edge/serverless, lightweight, multi-runtime

---

## Frameworks and Methodologies

### 1. Typed Express Setup — step-by-step
1. Install Express + `@types/express`
2. Define typed request/response interfaces
3. Create typed route handlers
4. Add typed middleware
5. Use Zod for request body validation
6. Use tsx for development, tsc for type checking

### 2. tRPC Setup — step-by-step
1. Install tRPC server + Zod
2. Define router with typed procedures
3. Create context (auth, DB session)
4. Export router type
5. Set up tRPC client with router type
6. Call procedures with full type safety

---

## How to Apply

### Decision Checklist
- [ ] Are route handlers fully typed (params, body, response)?
- [ ] Is input validation done at the API boundary (Zod, JSON Schema)?
- [ ] Is middleware typed and composable?
- [ ] Is tRPC considered for full-stack TypeScript projects?
- [ ] Is tsx used for development, tsc for type checking?

### Implementation Patterns

**Typed Express Routes:**
```typescript
import express, { Request, Response, NextFunction } from "express";
import { z } from "zod";

// Zod schemas
const CreateOrderSchema = z.object({
  customerId: z.string().uuid(),
  items: z.array(z.object({
    productId: z.string().uuid(),
    quantity: z.number().int().positive(),
  })).min(1),
});

type CreateOrderInput = z.infer<typeof CreateOrderSchema>;

// Typed route handler
const createOrder = async (
  req: Request<{}, {}, CreateOrderInput>,
  res: Response<{ id: string; status: string }>,
) => {
  const input = CreateOrderSchema.parse(req.body);
  const order = await orderService.create(input);
  res.status(201).json({ id: order.id, status: order.status });
};

// Validation middleware
function validate<T extends z.ZodType>(schema: T) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      res.status(400).json({ errors: result.error.flatten().fieldErrors });
      return;
    }
    req.body = result.data;
    next();
  };
}

app.post("/orders", validate(CreateOrderSchema), createOrder);
```

**Fastify with Type Providers:**
```typescript
import Fastify from "fastify";
import { TypeBoxTypeProvider } from "@fastify/type-provider-typebox";
import { Type, Static } from "@sinclair/typebox";

const app = Fastify().withTypeProvider<TypeBoxTypeProvider>();

const OrderSchema = Type.Object({
  customerId: Type.String({ format: "uuid" }),
  items: Type.Array(Type.Object({
    productId: Type.String({ format: "uuid" }),
    quantity: Type.Integer({ minimum: 1 }),
  }), { minItems: 1 }),
});

type OrderInput = Static<typeof OrderSchema>;

app.post("/orders", {
  schema: {
    body: OrderSchema,
    response: {
      201: Type.Object({
        id: Type.String(),
        status: Type.String(),
      }),
    },
  },
}, async (request, reply) => {
  // request.body is typed as OrderInput
  const order = await orderService.create(request.body);
  reply.status(201).send({ id: order.id, status: order.status });
});
```

**tRPC End-to-End Type Safety:**
```typescript
// server/trpc.ts
import { initTRPC, TRPCError } from "@trpc/server";
import { z } from "zod";

const t = initTRPC.context<Context>().create();

const router = t.router;
const publicProcedure = t.procedure;
const protectedProcedure = t.procedure.use(authMiddleware);

// server/routers/orders.ts
export const orderRouter = router({
  list: protectedProcedure
    .input(z.object({
      status: z.enum(["draft", "confirmed", "shipped"]).optional(),
      limit: z.number().min(1).max(100).default(20),
    }))
    .query(async ({ ctx, input }) => {
      return ctx.db.orders.findMany({
        where: { userId: ctx.user.id, status: input.status },
        take: input.limit,
      });
    }),

  create: protectedProcedure
    .input(z.object({
      items: z.array(z.object({
        productId: z.string().uuid(),
        quantity: z.number().int().positive(),
      })).min(1),
    }))
    .mutation(async ({ ctx, input }) => {
      return ctx.db.orders.create({
        data: { userId: ctx.user.id, items: input.items },
      });
    }),
});

// server/root.ts
export const appRouter = router({
  orders: orderRouter,
  users: userRouter,
});
export type AppRouter = typeof appRouter;

// client/trpc.ts
import { createTRPCReact } from "@trpc/react-query";
import type { AppRouter } from "../server/root";

export const trpc = createTRPCReact<AppRouter>();

// client/components/OrderList.tsx
function OrderList() {
  const { data: orders } = trpc.orders.list.useQuery({ status: "confirmed" });
  // orders is fully typed: Order[] — no codegen needed!

  const createOrder = trpc.orders.create.useMutation();
  // createOrder.mutate({ items: [...] }) — typed input
}
```

**Typed Middleware:**
```typescript
// Express middleware that adds user to request
interface AuthenticatedRequest extends Request {
  user: User;
}

function authMiddleware(req: Request, res: Response, next: NextFunction): void {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) {
    res.status(401).json({ error: "Unauthorized" });
    return;
  }

  try {
    const user = verifyToken(token);
    (req as AuthenticatedRequest).user = user;
    next();
  } catch {
    res.status(401).json({ error: "Invalid token" });
  }
}

// Typed handler using authenticated request
const getProfile = async (req: AuthenticatedRequest, res: Response) => {
  res.json(req.user);
};

app.get("/profile", authMiddleware, getProfile as any); // Type assertion needed for Express
// Fastify handles this more elegantly with decorators
```

### Common Mistakes
1. **Untyped req.body** — Using `req.body` as `any`; validate and type with Zod
2. **Manual API types** — Handwriting client types instead of using tRPC or OpenAPI codegen
3. **tsc for running code** — Using `tsc && node dist/` in development; use `tsx` for speed
4. **No validation middleware** — Trusting client input without server-side validation
5. **Express type limitations** — Express types are loose; consider Fastify for better type integration

### Integration with Other Topics
- **Runtime Validation** — Zod at the API boundary
- **TypeScript Monorepo** — Shared types between client and server
- **API Design** — REST (Express/Fastify) or tRPC for type-safe APIs
- **Testing** — Supertest for HTTP testing, tRPC testing utilities
- **ORM** — Prisma, Drizzle for type-safe database access

---

## Resources

### Essential Reading
- tRPC documentation (trpc.io)
- Fastify documentation (fastify.dev)
- Express + TypeScript guide

### Online Resources
- Hono documentation (hono.dev) — Multi-runtime framework
- tsx documentation (github.com/privatenumber/tsx)
- Total TypeScript — Node.js patterns

### Practice Exercises
1. Build a typed Express API with Zod validation middleware
2. Set up Fastify with TypeBox type provider
3. Create a full-stack app with tRPC (Next.js or React + Express)
4. Implement typed authentication middleware
5. Compare Express vs Fastify type safety for the same API
