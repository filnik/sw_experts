# Experts Agents — Claude Code Instructions

A curated knowledge base of software engineering expertise, organized as expert panels for Claude Code.

## How to Use

### Skills (slash commands)
Use these from any project that includes this knowledge base:
- `/software-architect` — architectural decisions, design patterns, system design, databases, cloud, security, DevOps
- `/python-expert` — Pythonic code, frameworks, testing, tooling
- `/typescript-expert` — type system, React, Node.js, testing, build tools
- `/ux-ui-expert` — UX strategy, visual design, accessibility, design systems

### Quick Reference
1. Read `synthesis/` files for high-level overviews of each panel
2. Use `synthesis/decision-matrix.md` for cross-panel trade-off comparisons
3. Dive into `experts/` files for detailed guidance on specific topics

## Structure
```
experts/           # 143 expert files (~200-300 lines each)
  architect/       # 66 files — 11 subcategories
  python/          # 20 files — 4 subcategories
  typescript/      # 20 files — 4 subcategories
  ux-ui/           # 37 files — 12 subcategories
synthesis/         # Panel summaries + decision matrix
.claude/skills/    # Claude Code skill definitions
```

## Language
All expert content is written in English.
