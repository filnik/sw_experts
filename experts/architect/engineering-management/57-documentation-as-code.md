# Documentation as Code

## Profile

### What It Is
Documentation as Code treats documentation with the same rigor as source code — version-controlled, reviewed via pull requests, tested for accuracy, and deployed automatically. It encompasses API docs, architecture docs, runbooks, and developer guides stored alongside code in the repository.

### Origin and Evolution
- Traditionally: separate wikis (Confluence), Word docs, out-of-date READMEs
- Docs-as-code movement (2010s): Markdown in repos, static site generators
- Tools: MkDocs, Docusaurus, Sphinx, AsciiDoc, GitBook
- Modern: auto-generated API docs (OpenAPI, TypeDoc), living documentation from tests
- Current: AI-assisted documentation, documentation linting, docs CI/CD

### Key Authors and References
- **Anne Gentle** — *Docs Like Code* (2017)
- **Tom Johnson** — idratherbewriting.com (API documentation)
- **Cyrille Martraire** — *Living Documentation* (2019)
- **Divio** — Documentation system framework (tutorials, how-tos, reference, explanation)

### Key Concepts at a Glance
| Type (Divio) | Purpose | Example |
|-------------|---------|---------|
| Tutorial | Learning-oriented, step-by-step | "Getting started with the API" |
| How-To | Goal-oriented, practical | "How to deploy to production" |
| Reference | Information-oriented, accurate | API reference, config options |
| Explanation | Understanding-oriented, conceptual | "Why we chose microservices" |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Docs live with code** — Documentation stored in the same repository as the code it describes. Same PR, same review.
2. **Single source of truth** — Auto-generate what you can (API docs from OpenAPI, type docs from code). Don't maintain parallel documents.
3. **Test documentation** — Broken links, invalid code examples, outdated screenshots are bugs. Test them in CI.
4. **Four types serve four needs** — Tutorials, how-tos, reference, and explanations serve different audiences and purposes. Don't mix them.
5. **Good enough > perfect** — Some documentation is infinitely better than no documentation. Start with READMEs and grow.

### When to Use vs. When to Avoid

**Invest in documentation when:**
- Onboarding new team members frequently
- The system is consumed by other teams (APIs, libraries, platforms)
- Operations require runbooks for incident response
- Architecture decisions need to be understood by future teams

**Keep it minimal when:**
- Solo developer on a personal project
- Code is self-documenting (clean code, clear naming, simple flow)
- Documentation will be outdated before it's useful

---

## Frameworks and Methodologies

### 1. Docs-as-Code Workflow — step-by-step
1. Store docs in `docs/` directory in the repository
2. Use Markdown or AsciiDoc for writing
3. Set up a static site generator (MkDocs, Docusaurus)
4. Configure CI to build and deploy docs on merge
5. Add documentation linting (markdownlint, vale)
6. Review docs changes in the same PR as code changes

### 2. API Documentation — step-by-step
1. Write OpenAPI/Swagger spec for REST APIs (or GraphQL schema)
2. Auto-generate reference documentation from the spec
3. Add code examples for each endpoint
4. Include authentication and error handling guides
5. Publish to developer portal with "try it" functionality
6. Validate spec in CI (spectral, openapi-lint)

---

## How to Apply

### Decision Checklist
- [ ] Are docs stored in the repository alongside code?
- [ ] Is there a CI pipeline for building and deploying docs?
- [ ] Is API documentation auto-generated from the spec?
- [ ] Are code examples tested or validated?
- [ ] Does the project have at minimum a useful README?

### Implementation Patterns

**MkDocs Configuration:**
```yaml
# mkdocs.yml
site_name: Order Service
theme:
  name: material
nav:
  - Home: index.md
  - Getting Started:
    - Quick Start: getting-started/quickstart.md
    - Configuration: getting-started/configuration.md
  - How-To:
    - Deploy: how-to/deploy.md
    - Add a New Endpoint: how-to/new-endpoint.md
  - Reference:
    - API: reference/api.md
    - Configuration Options: reference/config.md
  - Architecture:
    - Decisions: architecture/adr/index.md
    - System Overview: architecture/overview.md
plugins:
  - search
  - mkdocstrings  # Auto-generate from docstrings
```

**README Template:**
```markdown
# Service Name

Brief description of what this service does.

## Quick Start
\`\`\`bash
git clone ...
make setup
make run
\`\`\`

## Architecture
Brief overview with link to detailed docs.

## API
Link to API documentation or OpenAPI spec.

## Development
How to run tests, lint, and contribute.

## Deployment
How to deploy. Link to runbook.
```

### Common Mistakes
1. **Docs in a wiki** — Separate from code, quickly becomes outdated, no review process
2. **No code examples** — Documentation without runnable examples is incomplete
3. **Write once, never update** — Docs that aren't maintained are worse than no docs (misleading)
4. **Documenting everything** — Document what's needed; don't document obvious code
5. **No README** — The most basic documentation is missing from many projects

### Integration with Other Topics
- **ADRs** — Architecture decisions are documentation as code
- **API Design** — API docs generated from OpenAPI/GraphQL schemas
- **CI/CD** — Documentation built and deployed in the pipeline
- **Code Quality** — Documentation is a quality dimension
- **Clean Code** — Clean code reduces documentation needs
- **Platform Engineering** — Platform docs are essential for self-service

---

## Resources

### Essential Reading
- *Docs Like Code* — Anne Gentle
- *Living Documentation* — Cyrille Martraire
- Divio Documentation System (documentation.divio.com)

### Online Resources
- Write the Docs community (writethedocs.org)
- Tom Johnson's API documentation blog
- MkDocs Material theme documentation

### Practice Exercises
1. Set up MkDocs or Docusaurus for a project
2. Write a README that enables a new developer to start in 5 minutes
3. Auto-generate API documentation from an OpenAPI spec
4. Add documentation linting to CI
5. Organize existing docs into the Divio four-type framework
