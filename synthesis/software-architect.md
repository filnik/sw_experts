# Software Architect — Synthesis

## Overview

This panel covers 66 topics across 11 categories, providing a comprehensive reference for software architecture decisions. Each topic follows a consistent structure: 5 fundamental principles, frameworks/methodologies, implementation patterns with code, common mistakes, and integration points.

## Category Map

### 1. Architecture and Design Patterns (12 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 01 | SOLID Principles | Foundation for all design decisions |
| 02 | Design Patterns (GoF+) | Which pattern for which problem |
| 03 | Clean Architecture | Layer separation and dependency direction |
| 04 | Hexagonal Architecture | Ports & Adapters for testability |
| 05 | Domain-Driven Design | Strategic and tactical DDD |
| 06 | CQRS | Separate read/write models |
| 07 | Event Sourcing | Store events, derive state |
| 08 | Microservices | Service decomposition |
| 09 | Modular Monolith | Modules with clear boundaries |
| 10 | Event-Driven Architecture | Async event communication |
| 11 | Saga Pattern | Distributed transaction coordination |
| 12 | Vertical Slice Architecture | Feature-based organization |

**Start here**: 01-SOLID → 03-Clean Architecture → 05-DDD → then choose based on system needs.

### 2. System Design and Distributed Systems (9 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 13 | CAP Theorem | Consistency vs. Availability tradeoffs |
| 14 | System Design Fundamentals | Load balancing, scaling, CDN |
| 15 | API Design | REST vs. GraphQL vs. gRPC |
| 16 | API Gateway and BFF | Client-specific API layers |
| 17 | Messaging | Sync vs. async communication |
| 18 | Caching Strategies | Where and how to cache |
| 19 | Resilience Patterns | Circuit breaker, retry, bulkhead |
| 20 | Service Discovery and Mesh | Service communication infrastructure |
| 21 | Concurrency and Parallelism | Threading, async, parallel models |

### 3. Database and Data Management (6 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 22 | Relational Database Design | Normalization, indexes, constraints |
| 23 | NoSQL and Polyglot Persistence | Which database for which data |
| 24 | Data Modeling | Entity relationships, schema design |
| 25 | Database Scaling | Sharding, replication, partitioning |
| 26 | Transactions and Isolation | ACID, isolation levels, consistency |
| 27 | Search Engine Architecture | Full-text search, Elasticsearch |

### 4. Cloud and Infrastructure (6 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 28 | Cloud-Native (12-Factor) | Cloud-ready application design |
| 29 | Infrastructure as Code | Terraform, Pulumi, CloudFormation |
| 30 | Containerization | Docker, Kubernetes orchestration |
| 31 | Serverless | Functions vs. containers |
| 32 | Multi-Tenancy | Shared vs. isolated tenant models |
| 33 | Edge Computing | Processing at the edge |

### 5. Security (5 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 34 | Application Security (OWASP) | OWASP Top 10, secure coding |
| 35 | Authentication and Authorization | AuthN/AuthZ patterns |
| 36 | Zero Trust Architecture | Never trust, always verify |
| 37 | Data Privacy and Compliance | GDPR, data protection |
| 38 | Supply Chain Security | Dependency and build security |

### 6. Code Quality and Practices (8 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 39 | Clean Code | Readability and maintainability |
| 40 | Test-Driven Development | Red-Green-Refactor cycle |
| 41 | Refactoring | Safe transformation techniques |
| 42 | Code Review Practices | Effective review process |
| 43 | Technical Debt Management | Identifying and paying down debt |
| 44 | Continuous Testing Strategy | Test pyramid and automation |
| 45 | Pair Programming | Collaborative development |
| 45b | Working with Legacy Code | Safely changing existing code |

### 7. DevOps and Operations (6 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 46 | CI/CD Pipelines | Build, test, deploy automation |
| 47 | Deployment Strategies | Blue-green, canary, rolling |
| 48 | Observability and Monitoring | Metrics, logs, traces |
| 49 | Site Reliability Engineering | SLOs, error budgets, toil |
| 50 | GitOps | Git as single source of truth |
| 51 | Incident Management | Response, communication, postmortem |

### 8. Engineering Management (6 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 52 | Agile and Lean | Iterative delivery methodology |
| 53 | Architecture Decision Records | Documenting decisions |
| 54 | Team Topologies | Team structure and interaction |
| 55 | Evolutionary Architecture | Architecture that supports change |
| 56 | Technical Leadership | Leading without authority |
| 57 | Documentation as Code | Living documentation |

### 9. Data Engineering (3 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 58 | Data Pipelines and ETL | Batch vs. stream processing |
| 59 | Event Streaming Platforms | Kafka, Pulsar, event log |
| 60 | Integration Patterns | System connectivity patterns |

### 10. Frontend and Mobile (3 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 61 | Frontend Architecture | SPA vs. SSR vs. SSG vs. Islands |
| 62 | Mobile Architecture | Native vs. cross-platform |
| 63 | Design Systems | Component libraries and tokens |

### 11. AI and Emerging (2 topics)
| # | Topic | Key Decision |
|---|-------|-------------|
| 64 | AI/ML Architecture | ML pipelines, serving, MLOps |
| 65 | Platform Engineering | Internal developer platforms |

## Cross-Cutting Themes

1. **Start simple, evolve** — Modular monolith → microservices. Simple caching → distributed cache. The simplest solution that works is the right one.
2. **Boundaries matter most** — Clean Architecture, Hexagonal, DDD bounded contexts — all about defining clear boundaries between concerns.
3. **Observability enables everything** — You can't improve what you can't measure. Monitoring, logging, and tracing are prerequisites for reliability.
4. **Automate quality** — CI/CD, testing strategy, code quality tools — humans review logic, machines check style and correctness.
5. **Architecture is about tradeoffs** — CAP theorem, consistency vs. availability, complexity vs. flexibility — document decisions with ADRs.

## File Paths

All files are in `experts/architect/` organized by subcategory. Each file is ~200-300 lines following the standard template.
