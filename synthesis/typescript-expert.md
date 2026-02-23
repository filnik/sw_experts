# TypeScript Expert — Synthesis

## Overview

This panel covers 20 topics across 4 categories, providing a comprehensive reference for TypeScript best practices. From the type system and advanced types to testing, React patterns, and API design.

## Category Map

### 1. Language Fundamentals (5 topics)
| # | Topic | Key Takeaway |
|---|-------|-------------|
| 01 | TypeScript Type System | strict mode, discriminated unions, type narrowing, `satisfies` |
| 02 | Advanced Types | Generics, utility types, conditional types, template literals, branded types |
| 03 | TypeScript Configuration | strict: true, moduleResolution, path aliases, project references |
| 04 | Module Systems | ESM standard, named exports, tree shaking, dynamic imports |
| 05 | Decorators and Metadata | TC39 decorators (5.0+), experimental for NestJS/Angular |

**Learning path**: 01 → 02 → 03 → 04 → 05

### 2. Patterns and Architecture (6 topics)
| # | Topic | Key Takeaway |
|---|-------|-------------|
| 06 | TypeScript Design Patterns | Discriminated unions, generic patterns, typed event emitter |
| 07 | Error Handling | Result/Either with discriminated unions; neverthrow |
| 08 | Dependency Injection | Manual DI first; NestJS for NestJS; tsyringe for large projects |
| 09 | Runtime Validation | Zod: schema → type; validate at boundaries |
| 10 | Functional Programming | Immutability, pipe, Option/Result, pragmatic FP |
| 11 | TypeScript Monorepo | Turborepo/Nx, internal packages, shared config |

### 3. Testing and Quality (4 topics)
| # | Topic | Key Takeaway |
|---|-------|-------------|
| 12 | Testing with Vitest/Jest | Vitest for new projects; typed mocks, test factories |
| 13 | E2E Testing | Playwright: Page Objects, auth fixtures, stable selectors |
| 14 | Code Quality Tools | ESLint flat config + typescript-eslint + Prettier (or Biome) |
| 15 | Contract Testing | MSW for API mocking; OpenAPI codegen; Pact for microservices |

### 4. Ecosystem and Tools (5 topics)
| # | Topic | Key Takeaway |
|---|-------|-------------|
| 16 | React + TypeScript | Typed props, generic components, typed hooks/context |
| 17 | Node.js + TypeScript | Typed routes, tRPC for full-stack, Fastify for performance |
| 18 | TypeScript ORM Patterns | Prisma for DX, Drizzle for SQL control |
| 19 | Build Tools and Bundlers | Vite for apps, tsup for libraries, tsc for type checking |
| 20 | TypeScript API Patterns | tRPC, OpenAPI codegen, GraphQL codegen |

## Core TypeScript Principles (Cross-Cutting)

1. **strict: true always** — Non-negotiable. Enable from day one. Catches entire categories of bugs.
2. **Discriminated unions for state** — Model state machines, API responses, error types with discriminated unions. Not boolean flags.
3. **Generate, don't handwrite** — API types from OpenAPI/tRPC. ORM types from Prisma/Drizzle. Schema types from Zod.
4. **Validate at boundaries** — Zod at API inputs. TypeScript types for internal code. Never trust external data.
5. **Let inference work** — Don't over-annotate. Type function signatures, not every variable.

## Decision Quick Reference

| Question | Answer |
|----------|--------|
| Linter? | ESLint flat config + typescript-eslint |
| Formatter? | Prettier (or Biome for both) |
| Test framework? | Vitest |
| E2E framework? | Playwright |
| React state? | Zustand or Jotai |
| API (full-stack TS)? | tRPC |
| API (public/multi-lang)? | REST + OpenAPI codegen |
| ORM? | Prisma (DX) or Drizzle (control) |
| Runtime validation? | Zod |
| Build (app)? | Vite |
| Build (library)? | tsup |
| Monorepo? | Turborepo + pnpm |
| Dev execution? | tsx |

## File Paths

All files are in `experts/typescript/` organized by subcategory.
