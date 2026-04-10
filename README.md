# Experts Agents

A comprehensive knowledge base of software engineering principles organized into expert panels, designed as agent skills for Claude Code.

## What is this?

This repository contains **143 expert files** distilled from authoritative sources on software engineering. Each file covers a specific topic (~200-300 lines) with principles, patterns, code examples, trade-offs, and common mistakes.

The knowledge is structured as **four expert panels** that Claude Code can consult via slash commands, providing grounded, opinionated guidance on architectural decisions, language best practices, and design choices.

## Expert Panels

| Panel | Files | Subcategories |
|-------|-------|---------------|
| **Software Architect** | 66 | architecture & design patterns, system design, database, cloud & infrastructure, security, code quality, DevOps, engineering management, data engineering, frontend & mobile, AI & emerging |
| **Python Expert** | 20 | language fundamentals, patterns & architecture, testing & quality, ecosystem & tools |
| **TypeScript Expert** | 20 | language fundamentals, patterns & architecture, testing & quality, ecosystem & tools |
| **UX/UI Expert** | 37 | foundational principles, UX laws, visual design, design systems, accessibility, information architecture, usability research, responsive & mobile, modern web patterns, content strategy, emotional design & ethics, UX strategy |

## Usage with Claude Code

### As skills (slash commands)

From any project, invoke the expert panels directly:

```
/software-architect    # Architecture, patterns, system design, databases, cloud, security, DevOps
/python-expert         # Python best practices, frameworks, testing, tooling
/typescript-expert     # TypeScript type system, React, Node.js, testing, build tools
/ux-ui-expert          # UX strategy, visual design, accessibility, design systems
```

### Quick reference

- **Synthesis files** (`synthesis/`) — High-level overviews of each panel's topics
- **Decision matrix** (`synthesis/decision-matrix.md`) — Cross-panel comparison tables for common trade-offs (monolith vs microservices, database selection, API styles, etc.)
- **Expert files** (`experts/`) — Deep dives on individual topics with patterns, code, and anti-patterns

## Repository Structure

```
experts/                    # Individual expert files
  architect/                # 66 files across 11 subcategories
    architecture-and-design-patterns/
    system-design/
    database/
    cloud-and-infrastructure/
    security/
    code-quality/
    devops/
    engineering-management/
    data-engineering/
    frontend-and-mobile/
    ai-and-emerging/
  python/                   # 20 files across 4 subcategories
    language-fundamentals/
    patterns-and-architecture/
    testing-and-quality/
    ecosystem-and-tools/
  typescript/               # 20 files across 4 subcategories
    language-fundamentals/
    patterns-and-architecture/
    testing-and-quality/
    ecosystem-and-tools/
  ux-ui/                    # 37 files across 12 subcategories
    foundational-principles/
    ux-laws/
    visual-design/
    design-systems/
    accessibility/
    information-architecture/
    usability-research/
    responsive-mobile/
    modern-web-patterns/
    content-strategy/
    emotional-design-ethics/
    ux-strategy/
synthesis/                  # Panel summaries + decision matrix
.claude/skills/             # Claude Code skill definitions
```

## License

Personal knowledge base.
