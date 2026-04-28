# TypeScript Monorepo

## Profile

### What It Is
A TypeScript monorepo houses multiple packages in a single repository — shared types, libraries, applications, and tools. Monorepo tooling (Turborepo, Nx, workspaces) provides dependency management, incremental builds, and task orchestration across packages.

### Origin and Evolution
- Lerna (2015) — Early monorepo tool for JavaScript
- Yarn Workspaces (2017) — Package manager workspace support
- npm Workspaces (2020) — Built-in npm workspace support
- Turborepo (2021, Vercel) — Fast, incremental builds with caching
- Nx (2018, Nrwl) — Extensible monorepo toolkit with plugins
- pnpm Workspaces — Strict dependency isolation
- Current: Turborepo or Nx for orchestration; pnpm for package management

### Key Authors and References
- **Jared Palmer** — Turborepo creator
- **Victor Savkin** — Nx creator
- **Vercel** — Turborepo documentation
- **pnpm** — Strict, efficient package management

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Workspace | Package manager feature for linking local packages |
| Internal Package | Package consumed only within the monorepo |
| Task Pipeline | Build order based on package dependencies |
| Remote Caching | Share build cache across CI and developers |
| Incremental Builds | Only rebuild changed packages |
| Shared Config | Shared tsconfig, eslint, and build configuration |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Internal packages for shared code** — Types, utilities, and UI components shared as internal packages. Import as `@myorg/shared` not `../../shared`.
2. **Incremental builds** — Only build what changed. Turborepo/Nx cache build outputs. Builds should be fast regardless of repo size.
3. **Shared configuration** — One tsconfig base, one ESLint config, one Prettier config shared across packages. DRY for tooling.
4. **Package boundaries enforce architecture** — Each package has a clear public API. Internal implementation is hidden. Packages can't reach into each other's internals.
5. **CI caching is essential** — Remote caching (Turborepo, Nx Cloud) ensures CI doesn't rebuild what hasn't changed. Builds stay fast as the repo grows.

### When to Use vs. When to Avoid

**Monorepo when:**
- Multiple related packages (app + shared library + API)
- Shared types between frontend and backend
- Consistent tooling and configuration across projects
- Atomic changes across multiple packages

**Separate repos when:**
- Unrelated projects with different teams
- Different deployment lifecycles
- Simple projects that don't share code

---

## Frameworks and Methodologies

### 1. Monorepo Setup — step-by-step
1. Choose package manager: pnpm (recommended), npm, or yarn
2. Configure workspaces in `package.json` or `pnpm-workspace.yaml`
3. Create shared config packages (tsconfig, eslint)
4. Create internal packages for shared code
5. Set up Turborepo or Nx for task orchestration
6. Configure CI with remote caching

### 2. Internal Package Design — step-by-step
1. Create package directory with `package.json`
2. Define `exports` field for public API
3. Add TypeScript source with `tsconfig.json` extending shared base
4. Choose build strategy: prebuild (tsc/tsup) or just-in-time (bundler resolves source)
5. Import in consuming packages as `@myorg/package-name`

---

## How to Apply

### Decision Checklist
- [ ] Are shared types/utilities in internal packages?
- [ ] Is incremental build caching configured (Turborepo/Nx)?
- [ ] Is configuration shared (tsconfig, eslint, prettier)?
- [ ] Are package boundaries enforced (clear exports)?
- [ ] Is remote caching enabled for CI?

### Implementation Patterns

**Monorepo Structure:**
```
my-monorepo/
├── package.json                  # Root: workspaces config
├── pnpm-workspace.yaml           # pnpm workspace definition
├── turbo.json                    # Turborepo pipeline config
├── tsconfig.base.json            # Shared TypeScript config
│
├── apps/
│   ├── web/                      # Next.js frontend
│   │   ├── package.json
│   │   ├── tsconfig.json         # Extends base
│   │   └── src/
│   │
│   └── api/                      # Node.js backend
│       ├── package.json
│       ├── tsconfig.json
│       └── src/
│
├── packages/
│   ├── shared/                   # Shared types and utils
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── src/
│   │       ├── types.ts
│   │       └── utils.ts
│   │
│   ├── ui/                       # Shared UI components
│   │   ├── package.json
│   │   └── src/
│   │
│   ├── tsconfig/                 # Shared tsconfig presets
│   │   ├── base.json
│   │   ├── react.json
│   │   └── node.json
│   │
│   └── eslint-config/            # Shared ESLint config
│       ├── package.json
│       └── index.js
```

**pnpm Workspace:**
```yaml
# pnpm-workspace.yaml
packages:
  - "apps/*"
  - "packages/*"
```

**Internal Package (packages/shared):**
```json
{
  "name": "@myorg/shared",
  "version": "0.0.0",
  "private": true,
  "exports": {
    ".": {
      "types": "./src/index.ts",
      "default": "./src/index.ts"
    }
  },
  "scripts": {
    "type-check": "tsc --noEmit",
    "lint": "eslint src/"
  },
  "devDependencies": {
    "@myorg/tsconfig": "workspace:*",
    "typescript": "^5.3"
  }
}
```

**Turborepo Configuration:**
```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**"]
    },
    "type-check": {
      "dependsOn": ["^build"]
    },
    "lint": {},
    "test": {
      "dependsOn": ["^build"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

**Shared tsconfig:**
```json
// packages/tsconfig/base.json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "target": "ES2022",
    "lib": ["ES2022"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "verbatimModuleSyntax": true
  }
}

// apps/web/tsconfig.json
{
  "extends": "@myorg/tsconfig/react.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"]
}
```

**Consuming Internal Packages:**
```json
// apps/web/package.json
{
  "dependencies": {
    "@myorg/shared": "workspace:*",
    "@myorg/ui": "workspace:*"
  }
}
```

```typescript
// apps/web/src/features/orders/OrderList.tsx
import { Order, formatCurrency } from "@myorg/shared";
import { Button, Card } from "@myorg/ui";

function OrderList({ orders }: { orders: Order[] }) {
  return orders.map((order) => (
    <Card key={order.id}>
      <span>{formatCurrency(order.total)}</span>
      <Button>View</Button>
    </Card>
  ));
}
```

### Common Mistakes
1. **No task caching** — Running all builds from scratch on every CI run; enable Turborepo/Nx caching
2. **Circular dependencies** — Package A imports B, B imports A; restructure shared code into a third package
3. **Huge shared package** — One `@myorg/shared` with everything; split into focused packages
4. **No remote caching in CI** — Local cache doesn't help CI; enable Turborepo Remote Caching
5. **Inconsistent tooling** — Different TypeScript or ESLint versions per package; share from config packages

### Integration with Other Topics
- **TypeScript Configuration** — Shared tsconfig base with project references
- **Module Systems** — Internal packages use workspace resolution
- **Build Tools** — Turborepo/Nx orchestrate builds across packages
- **CI/CD** — Incremental CI with remote caching
- **Design Systems** — UI package as internal design system library

---

## Resources

### Essential Reading
- Turborepo documentation (turbo.build)
- Nx documentation (nx.dev)
- pnpm workspaces documentation

### Online Resources
- Vercel monorepo examples
- Turborepo starter templates
- "Monorepo Handbook" (monorepo.tools)

### Practice Exercises
1. Set up a monorepo with pnpm + Turborepo + 2 apps + 1 shared package
2. Create a shared tsconfig package and extend it in all sub-packages
3. Configure Turborepo task pipeline with correct dependency ordering
4. Enable remote caching in CI (Turborepo)
5. Add a shared UI component library consumed by multiple apps
