# Decision Matrix — Cross-Panel Reference

## Architecture Decisions

### Monolith vs. Microservices
| Factor | Monolith | Modular Monolith | Microservices |
|--------|----------|------------------|---------------|
| Team size | 1-10 | 5-20 | 10+ |
| Deployment | Simple | Simple | Complex (K8s) |
| Latency | Low (in-process) | Low | Higher (network) |
| Data consistency | ACID transactions | Module boundaries | Eventual consistency |
| Complexity | Low | Medium | High |
| When | Start here | Growing team | Proven need |

**References**: `architect/08-microservices`, `architect/09-modular-monolith`

### API Style
| Factor | REST | GraphQL | gRPC | tRPC |
|--------|------|---------|------|------|
| Best for | Public APIs | Complex data needs | Service-to-service | Full-stack TS |
| Type safety | OpenAPI codegen | Schema + codegen | Protobuf | End-to-end |
| Caching | HTTP caching | Complex | N/A | React Query |
| Learning curve | Low | Medium | Medium | Low |
| Browser support | Native | Library needed | grpc-web | Native |

**References**: `architect/15-api-design`, `typescript/20-typescript-api-patterns`

### Database Selection
| Need | Recommended | Alternative |
|------|-------------|-------------|
| Relational data | PostgreSQL | MySQL |
| Document store | MongoDB | DynamoDB |
| Key-value cache | Redis | Memcached |
| Full-text search | Elasticsearch | PostgreSQL FTS |
| Time series | TimescaleDB | InfluxDB |
| Graph relationships | Neo4j | PostgreSQL (recursive) |

**References**: `architect/22-relational-database`, `architect/23-nosql-polyglot`

### Rendering Strategy
| Strategy | Best For | SEO | Performance |
|----------|----------|-----|-------------|
| SSR | Dynamic + SEO | Excellent | Good (TTFB) |
| SSG | Static content | Excellent | Best |
| SPA/CSR | Dashboards, apps | Poor | Good (after load) |
| Islands | Mostly static + interactive | Excellent | Best |

**References**: `architect/61-frontend-architecture`

## Python Decisions

### Web Framework
| Factor | FastAPI | Django | Flask |
|--------|---------|--------|-------|
| Best for | APIs, microservices | Full-stack web apps | Simple apps |
| Async | Native | Partial (views) | No |
| ORM | SQLAlchemy | Django ORM | SQLAlchemy |
| Admin panel | No (build custom) | Built-in | No |
| Type safety | Pydantic | Serializers | Manual |
| Performance | High | Medium | Medium |

**References**: `python/16-web-frameworks`

### Python ORM
| Factor | SQLAlchemy 2.0 | Django ORM | Raw SQL |
|--------|----------------|------------|---------|
| Type safety | Good (mapped_column) | Moderate | None |
| Query flexibility | High (Core + ORM) | Medium | Full |
| Async support | Yes (asyncpg) | Limited | Yes |
| Migration tool | Alembic | Built-in | Manual |
| Use when | FastAPI/Flask | Django | Complex analytics |

**References**: `python/17-orm-patterns`

### Python Concurrency
| Workload | Use | Avoid |
|----------|-----|-------|
| I/O-bound (many) | asyncio | multiprocessing |
| I/O-bound (few, sync libs) | ThreadPoolExecutor | ProcessPoolExecutor |
| CPU-bound | ProcessPoolExecutor | threading |
| Mixed I/O + CPU | asyncio + ProcessPool | Single approach |

**References**: `python/05-async-programming`, `python/11-python-concurrency`

## TypeScript Decisions

### State Management (React)
| Factor | Zustand | Jotai | Redux Toolkit | React Context |
|--------|---------|-------|---------------|---------------|
| Complexity | Low | Low | Medium | Low |
| DevTools | Yes | Yes | Excellent | React DevTools |
| Best for | Most apps | Atomic state | Large apps | Simple shared state |
| Bundle size | Small | Small | Medium | Zero |
| Learning curve | Low | Low | Medium | Low |

**References**: `architect/61-frontend-architecture`, `typescript/16-react-typescript`

### TypeScript ORM
| Factor | Prisma | Drizzle | TypeORM |
|--------|--------|---------|---------|
| Type safety | Excellent (generated) | Excellent (schema) | Good (decorators) |
| Query style | Custom API | SQL-like | Query builder |
| Performance | Good | Excellent | Good |
| Migration | Prisma Migrate | Drizzle Kit | TypeORM CLI |
| Best for | Rapid dev | SQL control | Java-like teams |

**References**: `typescript/18-typescript-orm-patterns`

### Build Tool
| Factor | Vite | Next.js | esbuild | tsc |
|--------|------|---------|---------|-----|
| Type | Dev + build | Framework | Compiler | Compiler + checker |
| Speed | Fast (esbuild) | Medium | Fastest | Slow |
| Type checking | No (use tsc) | No (use tsc) | No | Yes |
| Use for | Apps (React, Vue) | Full-stack React | Custom builds | Type checking only |

**References**: `typescript/19-build-tools-bundlers`

## UX/UI Decisions

### Design System vs Custom CSS vs UI Framework
| Factor | Custom Design System | Tailwind + shadcn/ui | Material UI | Chakra UI | CSS Modules |
|--------|---------------------|----------------------|-------------|-----------|-------------|
| Consistency | Full control | Good (with tokens) | Excellent | Good | Manual |
| Customization | Unlimited | High | Medium (theming) | High | Unlimited |
| Development speed | Slow initially | Fast | Fast | Fast | Medium |
| Bundle size | Minimal | Small (purged) | Large | Medium | Minimal |
| Learning curve | Team-specific | Low | Medium | Low | Low |
| Best for | Large orgs, multi-product | Most projects | Google-style apps | Rapid prototyping | Small teams |
| Maintenance | High (dedicated team) | Low | Low (community) | Low | Medium |

**References**: `ux-ui/20-atomic-design`, `ux-ui/21-design-systems`, `ux-ui/22-material-design`

### Rendering Strategy — UX Perspective
| Strategy | UX Strengths | UX Weaknesses | Best UX For |
|----------|-------------|---------------|-------------|
| SSR | Fast first paint, SEO | Full page reloads (without SPA hydration) | Content sites, e-commerce |
| SSG | Instant loads, reliability | Stale content | Blogs, docs, marketing |
| SPA/CSR | App-like transitions, rich interactions | Slow initial load, poor SEO | Dashboards, admin panels |
| Islands | Best of SSG + selective interactivity | Complexity | Mostly static + interactive sections |

**References**: `ux-ui/37-web-performance-ux`, `architect/61-frontend-architecture`

### Mobile-First vs Desktop-First
| Factor | Mobile-First | Desktop-First |
|--------|-------------|---------------|
| Approach | min-width media queries | max-width media queries |
| Content strategy | Progressive enhancement | Graceful degradation |
| Performance | Lighter base CSS | Heavier base, override for mobile |
| Focus | Core content first | Full experience first |
| Best when | Mobile traffic > 50%, new projects | Legacy desktop apps, data-heavy dashboards |
| Risk | Desktop may feel sparse | Mobile may feel cramped |

**References**: `ux-ui/27-responsive-web-design`, `ux-ui/28-mobile-first`

### Form Design Patterns
| Pattern | Best For | Avoid When |
|---------|----------|------------|
| Single-page form | < 6 fields, simple data | Complex multi-section data |
| Multi-step wizard | Complex processes (checkout, onboarding) | Simple forms (login, contact) |
| Inline validation | Real-time feedback, field-level errors | Expensive server validation |
| Floating labels | Space-constrained layouts | Long label text, accessibility concerns |
| Progressive disclosure | Optional/conditional fields | All fields are required |
| Accordion form | Long forms with logical sections | Short forms |

**References**: `ux-ui/29-form-design`, `ux-ui/05-usability-heuristics`, `ux-ui/33-practical-accessibility`

### Navigation Pattern by Site Type
| Site Type | Primary Nav | Secondary Nav | Mobile Nav |
|-----------|------------|---------------|------------|
| E-commerce | Mega menu + categories | Breadcrumbs + filters | Bottom nav + hamburger |
| SaaS dashboard | Sidebar | Tabs + breadcrumbs | Hamburger + bottom nav |
| Blog/content | Top bar + categories | Related posts + tags | Hamburger |
| Marketing/landing | Top bar (minimal) | Footer links | Hamburger (minimal) |
| Documentation | Sidebar + search | Table of contents | Hamburger + search |
| Social platform | Bottom nav (mobile-first) | Tabs | Bottom nav |

**References**: `ux-ui/17-information-architecture`, `ux-ui/36-modern-web-patterns`, `ux-ui/06-web-usability`

### Color Scheme Selection
| Approach | Hues | Best For | Mood |
|----------|------|----------|------|
| Monochromatic | 1 hue, varied lightness | Elegant, minimal interfaces | Professional, calm |
| Analogous | 2-3 adjacent hues | Harmonious, cohesive feel | Warm/cool, natural |
| Complementary | 2 opposite hues | High contrast, emphasis | Bold, energetic |
| Split-complementary | 1 + 2 adjacent to complement | Balanced contrast | Dynamic, approachable |
| 60-30-10 rule | Primary 60%, secondary 30%, accent 10% | Most interfaces | Balanced |

**References**: `ux-ui/26-color-theory`, `ux-ui/21-design-systems`, `ux-ui/32-wcag-accessibility`

### Typography Strategy
| Context | Font Pairing | Type Scale | Line Height |
|---------|-------------|------------|-------------|
| Corporate/SaaS | Inter + system stack | 1.25 ratio (Major Third) | 1.5 body, 1.2 headings |
| Editorial/blog | Serif headings + sans body | 1.333 ratio (Perfect Fourth) | 1.6-1.75 body |
| Technical/docs | Monospace code + sans body | 1.2 ratio (Minor Third) | 1.5 body, 1.3 code |
| Marketing | Display headings + clean body | 1.5+ ratio (contrast) | 1.5 body, 1.1 display |
| Mobile-first | System fonts for performance | Smaller scale, fluid (clamp) | 1.5 minimum |

**References**: `ux-ui/24-web-typography`, `ux-ui/25-visual-hierarchy`, `ux-ui/37-web-performance-ux`

## Cross-Panel Decisions

### Testing Strategy
| Layer | Python Tool | TypeScript Tool | Scope |
|-------|-------------|-----------------|-------|
| Unit | pytest | Vitest | Functions, classes |
| Integration | pytest + fixtures | Vitest + MSW | Services, APIs |
| E2E | Playwright | Playwright | User flows |
| Contract | Schemathesis | Pact / MSW | API contracts |
| Property | Hypothesis | fast-check | Invariants |
| Visual | N/A | Chromatic / Percy | UI regression |

### Code Quality
| Concern | Python | TypeScript |
|---------|--------|------------|
| Linting | ruff | ESLint + typescript-eslint |
| Formatting | ruff format | Prettier (or Biome) |
| Type checking | mypy / pyright | tsc --noEmit |
| Security scanning | bandit (ruff S rules) | ESLint security plugins |
| Pre-commit | pre-commit | husky + lint-staged |

### Package Management
| Concern | Python | TypeScript |
|---------|--------|------------|
| Tool | uv | pnpm |
| Config file | pyproject.toml | package.json |
| Lock file | uv.lock | pnpm-lock.yaml |
| Monorepo | uv workspaces | pnpm workspaces + Turborepo |
| Registry | PyPI | npm |
