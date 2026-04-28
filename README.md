# Experts Agents

A comprehensive knowledge base of software engineering principles organized into expert panels, designed as agent skills for Claude Code.

## What is this?

This repository contains **190 expert files** distilled from authoritative sources on software engineering and AI agent design. Each file covers a specific topic (~200-300 lines) with principles, patterns, code examples, trade-offs, and common mistakes.

The knowledge is structured as **five expert panels** that Claude Code can consult via slash commands, providing grounded, opinionated guidance on architectural decisions, language best practices, design choices, and AI agent engineering.

## Expert Panels

| Panel | Files | Subcategories |
|-------|-------|---------------|
| **Software Architect** | 66 | architecture & design patterns, system design, database, cloud & infrastructure, security, code quality, DevOps, engineering management, data engineering, frontend & mobile, AI & emerging |
| **Python Expert** | 20 | language fundamentals, patterns & architecture, testing & quality, ecosystem & tools |
| **TypeScript Expert** | 20 | language fundamentals, patterns & architecture, testing & quality, ecosystem & tools |
| **UX/UI Expert** | 37 | foundational principles, UX laws, visual design, design systems, accessibility, information architecture, usability research, responsive & mobile, modern web patterns, content strategy, emotional design & ethics, UX strategy |
| **Agent Architect** | 47 | experts & thought leaders, official AI company guides, open-source frameworks, architecture & design patterns, GitHub pattern repos, academic surveys |

## Usage with Claude Code

### As skills (slash commands)

From any project, invoke the expert panels directly:

```
/software-architect    # Architecture, patterns, system design, databases, cloud, security, DevOps
/python-expert         # Python best practices, frameworks, testing, tooling
/typescript-expert     # TypeScript type system, React, Node.js, testing, build tools
/ux-ui-expert          # UX strategy, visual design, accessibility, design systems
/agent-architect       # AI agent design, multi-agent orchestration, prompt/context engineering
```

### As an agent (proactive)

The repo also ships a Claude Code agent definition at `.claude/agents/agent-architect.md` so the AI agent expert can be invoked proactively (without an explicit slash command) when designing or reviewing agent architectures.

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
  agent-architect/          # 47 files across 6 subcategories
    experts/
    guides/
    frameworks/
    patterns/
    repos/
    surveys/
synthesis/                  # Panel summaries + decision matrices
.claude/skills/             # Claude Code skill definitions
.claude/agents/             # Claude Code agent definitions (agent-architect)
```

## License

Personal knowledge base.
