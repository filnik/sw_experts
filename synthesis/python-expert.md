# Python Expert — Synthesis

## Overview

This panel covers 20 topics across 4 categories, providing a comprehensive reference for Python best practices. From idiomatic code and type hints to testing, concurrency, and ecosystem tools.

## Category Map

### 1. Language Fundamentals (5 topics)
| # | Topic | Key Takeaway |
|---|-------|-------------|
| 01 | Pythonic Code and PEP 8 | Idiomatic Python: comprehensions, unpacking, EAFP, f-strings |
| 02 | Type Hints and Static Typing | Gradual typing with mypy/pyright; Protocol for duck typing |
| 03 | Decorators and Metaclasses | Cross-cutting concerns; prefer `__init_subclass__` over metaclasses |
| 04 | Generators, Iterators, Context Managers | Lazy evaluation, resource management, `yield` pipelines |
| 05 | Async Programming | asyncio for I/O concurrency; `TaskGroup` for structured concurrency |

**Learning path**: 01 → 02 → 03 → 04 → 05

### 2. Patterns and Architecture (6 topics)
| # | Topic | Key Takeaway |
|---|-------|-------------|
| 06 | Python Design Patterns | Functions replace simple classes; Protocol for duck typing |
| 07 | Python Project Structure | src layout, pyproject.toml, curated `__init__.py` |
| 08 | Dependency Injection | Manual DI first; FastAPI Depends for web; composition root |
| 09 | Error Handling Patterns | Custom exception hierarchies; Result pattern for expected failures |
| 10 | Data Validation with Pydantic | Schema as source of truth; validate at boundaries |
| 11 | Python Concurrency | GIL: threads for I/O, processes for CPU; concurrent.futures |

### 3. Testing and Quality (4 topics)
| # | Topic | Key Takeaway |
|---|-------|-------------|
| 12 | Testing with pytest | Fixtures, parametrize, markers; plain assert |
| 13 | Mocking and Test Doubles | Prefer fakes over mocks; mock at boundaries |
| 14 | Code Quality Tools | ruff (lint+format), mypy (types), pre-commit |
| 15 | Property-Based Testing | Hypothesis: test invariants, not examples |

### 4. Ecosystem and Tools (5 topics)
| # | Topic | Key Takeaway |
|---|-------|-------------|
| 16 | Web Frameworks | FastAPI for APIs, Django for full-stack, Flask declining |
| 17 | ORM Patterns | SQLAlchemy 2.0: typed models, N+1 prevention, Alembic migrations |
| 18 | Package Management | uv as all-in-one; pyproject.toml; lock files for apps |
| 19 | CLI Development | typer + rich for modern CLIs |
| 20 | Python Performance | Profile first; algorithm > micro-optimization; caching |

## Core Python Principles (Cross-Cutting)

1. **Readability counts** — PEP 8, comprehensions, flat code. If it needs a comment to explain what it does, rewrite it.
2. **Type everything public** — Type annotations on public APIs. Gradual typing: start with signatures, expand to strict mode.
3. **Validate at boundaries** — Pydantic at API boundaries. Internal code trusts validated types.
4. **Test with pytest** — Fixtures for DI in tests. Parametrize for data-driven. Fakes over mocks.
5. **Modern tooling** — ruff replaces flake8+black+isort. uv replaces pip+virtualenv. pyproject.toml replaces setup.py.

## Decision Quick Reference

| Question | Answer |
|----------|--------|
| Linter? | ruff |
| Formatter? | ruff format |
| Type checker? | mypy (strict) or pyright |
| Test framework? | pytest |
| Web framework (API)? | FastAPI |
| Web framework (full-stack)? | Django |
| ORM? | SQLAlchemy 2.0 |
| Package manager? | uv |
| CLI framework? | typer + rich |
| Async HTTP client? | httpx |
| Data validation? | Pydantic v2 |

## File Paths

All files are in `experts/python/` organized by subcategory.
