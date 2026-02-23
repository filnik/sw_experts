# TypeScript API Patterns

## Profile

### What It Is
TypeScript API patterns cover type-safe approaches for building and consuming APIs — REST typing with OpenAPI codegen, GraphQL with codegen, tRPC for end-to-end safety, and patterns for typed HTTP clients, middleware, and API layers.

### Origin and Evolution
- REST + manual types — Handwritten request/response types
- Swagger/OpenAPI codegen — Generate types from API specification
- GraphQL Code Generator (2018) — Generate types from GraphQL schema
- tRPC (2021) — End-to-end type safety without code generation
- Hono RPC (2023) — Type-safe RPC for Hono framework
- Current: tRPC for full-stack TS, OpenAPI codegen for REST, GraphQL codegen for GraphQL

### Key Authors and References
- **Alex / KATT** — tRPC creator
- **Dotan Simha** — GraphQL Code Generator
- **OpenAPI Initiative** — API specification standard
- **Colin McDonnell** — Zod (used across API patterns)

### Key Concepts at a Glance
| Pattern | Type Safety | Code Generation |
|---------|-------------|-----------------|
| tRPC | End-to-end, compile-time | None needed |
| OpenAPI + codegen | Client types from spec | openapi-typescript, orval |
| GraphQL + codegen | Operations typed from schema | graphql-codegen |
| Manual REST types | Handwritten, can drift | None |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **End-to-end type safety** — The ideal: change a field on the server, the client gets a compile error. tRPC achieves this. OpenAPI codegen approximates it.
2. **Generate, don't handwrite** — API types should be generated from a single source of truth (schema, server code). Manual types drift.
3. **Validate at the boundary** — Whether tRPC (Zod), REST (Zod middleware), or GraphQL (schema), validate all input at the API boundary.
4. **Type the full request cycle** — URL params, query strings, headers, body, response — everything typed. No `any` in the API layer.
5. **Choose pattern by context** — tRPC for monorepo full-stack TS. OpenAPI for multi-language consumers. GraphQL for complex data requirements.

### When to Use vs. When to Avoid

**tRPC**: Full-stack TypeScript (Next.js, monorepo, single team)
**OpenAPI codegen**: Multi-language consumers, public APIs, microservices
**GraphQL codegen**: Complex data needs, multiple frontends with different data requirements
**Manual REST types**: Simple APIs, temporary solutions

---

## Frameworks and Methodologies

### 1. tRPC API Design — step-by-step
1. Define tRPC router with procedures
2. Add Zod input validation to each procedure
3. Create context with auth and dependencies
4. Export router type for client
5. Set up typed client (React Query integration)
6. Enjoy compile-time safety across the stack

### 2. OpenAPI-First Design — step-by-step
1. Define OpenAPI spec (YAML or generated from code)
2. Generate TypeScript types: `openapi-typescript`
3. Generate typed client: `openapi-fetch` or `orval`
4. Use generated types in frontend
5. Validate implementation matches spec in CI

---

## How to Apply

### Decision Checklist
- [ ] Is the API type-safe end-to-end (tRPC) or via codegen (OpenAPI/GraphQL)?
- [ ] Is input validation done at the API boundary?
- [ ] Are types generated from a single source of truth?
- [ ] Is the full request cycle typed (params, body, response)?
- [ ] Does the API approach match the project context?

### Implementation Patterns

**tRPC Full Pattern:**
```typescript
// Server: define procedures
import { initTRPC, TRPCError } from "@trpc/server";
import { z } from "zod";

const t = initTRPC.context<{ user: User | null; db: Database }>().create();

const isAuthed = t.middleware(({ ctx, next }) => {
  if (!ctx.user) throw new TRPCError({ code: "UNAUTHORIZED" });
  return next({ ctx: { ...ctx, user: ctx.user } });
});

export const appRouter = t.router({
  users: t.router({
    me: t.procedure.use(isAuthed).query(({ ctx }) => {
      return ctx.db.user.findUnique({ where: { id: ctx.user.id } });
    }),

    updateProfile: t.procedure
      .use(isAuthed)
      .input(z.object({
        name: z.string().min(1).max(100),
        bio: z.string().max(500).optional(),
      }))
      .mutation(({ ctx, input }) => {
        return ctx.db.user.update({
          where: { id: ctx.user.id },
          data: input,
        });
      }),
  }),

  posts: t.router({
    list: t.procedure
      .input(z.object({
        cursor: z.string().optional(),
        limit: z.number().min(1).max(50).default(20),
      }))
      .query(async ({ ctx, input }) => {
        const posts = await ctx.db.post.findMany({
          take: input.limit + 1,
          cursor: input.cursor ? { id: input.cursor } : undefined,
          orderBy: { createdAt: "desc" },
        });
        const hasMore = posts.length > input.limit;
        return {
          posts: posts.slice(0, input.limit),
          nextCursor: hasMore ? posts[input.limit - 1].id : null,
        };
      }),
  }),
});

export type AppRouter = typeof appRouter;

// Client: fully typed
import { trpc } from "@/utils/trpc";

function Profile() {
  const { data: user } = trpc.users.me.useQuery();
  const updateProfile = trpc.users.updateProfile.useMutation();

  // user is typed as User | null
  // updateProfile.mutate({ name: "..." }) is typed

  const { data } = trpc.posts.list.useQuery({ limit: 10 });
  // data.posts is Post[], data.nextCursor is string | null
}
```

**REST with OpenAPI Codegen:**
```typescript
// 1. Generate types from OpenAPI spec
// npx openapi-typescript api-spec.yaml -o src/api/types.ts

// 2. Create typed client
import createClient from "openapi-fetch";
import type { paths } from "./api/types";

const api = createClient<paths>({
  baseUrl: "https://api.example.com",
  headers: { Authorization: `Bearer ${token}` },
});

// 3. Fully typed API calls
async function getOrders() {
  const { data, error } = await api.GET("/orders", {
    params: {
      query: { status: "confirmed", limit: 20 },
    },
  });

  if (error) {
    // error is typed based on OpenAPI error responses
    console.error(error.message);
    return [];
  }

  return data; // Typed as Order[]
}

async function createOrder(input: CreateOrderInput) {
  const { data } = await api.POST("/orders", {
    body: input, // Typed: must match OpenAPI request body schema
  });
  return data; // Typed: matches OpenAPI 201 response
}
```

**GraphQL with Codegen:**
```typescript
// operations.graphql
// query GetOrders($status: OrderStatus, $limit: Int) {
//   orders(status: $status, limit: $limit) {
//     id
//     status
//     total
//     items {
//       productId
//       quantity
//     }
//   }
// }

// Generated types (graphql-codegen)
import { useGetOrdersQuery } from "./generated/graphql";

function OrderList() {
  const { data, loading, error } = useGetOrdersQuery({
    variables: { status: "CONFIRMED", limit: 20 },
  });

  // data.orders is typed: { id: string; status: string; total: number; items: ... }[]
}
```

**Typed REST Client (manual, lightweight):**
```typescript
// Type-safe fetch wrapper
type ApiRoutes = {
  "GET /users/:id": { params: { id: string }; response: User };
  "GET /orders": { query: { status?: string; limit?: number }; response: Order[] };
  "POST /orders": { body: CreateOrderInput; response: Order };
  "DELETE /orders/:id": { params: { id: string }; response: void };
};

class TypedApiClient {
  constructor(private baseUrl: string) {}

  async get<R extends keyof ApiRoutes & `GET ${string}`>(
    route: R,
    options?: Omit<ApiRoutes[R], "response">,
  ): Promise<ApiRoutes[R]["response"]> {
    // Implementation: build URL from route + params/query
    const response = await fetch(this.buildUrl(route, options));
    return response.json();
  }

  // Similar for post, put, delete...
}

const api = new TypedApiClient("https://api.example.com");
const user = await api.get("GET /users/:id", { params: { id: "123" } });
// user is typed as User
```

### Common Mistakes
1. **Manual API types** — Handwriting types that drift from the actual API; use codegen or tRPC
2. **No input validation** — Trusting client input on the server; always validate with Zod/schema
3. **tRPC in public APIs** — tRPC is for internal TypeScript clients; use REST/GraphQL for public APIs
4. **Ignoring error types** — Only typing success responses; type error responses too
5. **Not regenerating types** — API spec changes but generated types are stale; add to CI

### Integration with Other Topics
- **Runtime Validation** — Zod validates API inputs
- **Node.js + TypeScript** — Server-side API implementation
- **React + TypeScript** — Client-side API consumption
- **Contract Testing** — API contracts verified with MSW, Pact
- **TypeScript Monorepo** — Shared API types in monorepo packages

---

## Resources

### Essential Reading
- tRPC documentation (trpc.io)
- openapi-typescript documentation
- GraphQL Code Generator documentation (the-guild.dev/graphql/codegen)

### Online Resources
- openapi-fetch documentation
- orval — Generate typed API clients (orval.dev)
- Total TypeScript — API patterns

### Practice Exercises
1. Build a complete tRPC API with router, context, and React client
2. Generate types from an OpenAPI spec and create a typed client
3. Set up GraphQL codegen for a React app with typed hooks
4. Compare tRPC vs OpenAPI codegen for the same API
5. Build a lightweight typed REST client wrapper
