# Experts Agents — Claude Code Instructions

A curated knowledge base of software engineering expertise, organized as expert panels for Claude Code.

## How to Use

### Skills (slash commands)
Use these from any project that includes this knowledge base:
- `/software-architect` — architectural decisions, design patterns, system design, databases, cloud, security, DevOps
- `/python-expert` — Pythonic code, frameworks, testing, tooling
- `/typescript-expert` — type system, React, Node.js, testing, build tools
- `/ux-ui-expert` — UX strategy, visual design, accessibility, design systems
- `/agent-architect` — AI agent design, multi-agent orchestration, prompt/context engineering, agent frameworks

### Agent (proactive)
The repo also ships an agent definition at `.claude/agents/agent-architect.md` so the agent can be invoked proactively (not only via slash command). It points at the same knowledge base.

### Quick Reference
1. Read `synthesis/` files for high-level overviews of each panel
2. Use `synthesis/decision-matrix.md` for cross-panel trade-off comparisons (and `synthesis/agent-architect-decision-matrix.md` for AI-agent-specific ones)
3. Dive into `experts/` files for detailed guidance on specific topics

## Structure
```
experts/                # 190 expert files (~200-300 lines each)
  architect/            # 66 files — 11 subcategories
  python/               # 20 files — 4 subcategories
  typescript/           # 20 files — 4 subcategories
  ux-ui/                # 37 files — 12 subcategories
  agent-architect/      # 47 files — 6 subcategories
synthesis/              # Panel summaries + decision matrices
.claude/skills/         # Claude Code skill definitions
.claude/agents/         # Claude Code agent definitions (agent-architect)
```

## Language
All expert content is written in English.
