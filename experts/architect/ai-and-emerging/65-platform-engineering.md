# Platform Engineering

## Profile

### What It Is
Platform Engineering is the discipline of designing and building self-service internal developer platforms (IDPs) that abstract infrastructure complexity, standardize workflows, and enable product teams to deliver software independently. It applies product thinking to internal tooling — treating developers as customers.

### Origin and Evolution
- Operations teams managing servers manually (pre-2010)
- DevOps movement (2010s) — breaking down dev/ops silos
- SRE at Google (2016) — reliability engineering as a discipline
- Platform teams emerge (2018+) — dedicated teams building internal platforms
- Current: Internal Developer Platforms (IDPs), Backstage (Spotify), platform as a product

### Key Authors and References
- **Team Topologies** (Skelton & Pais) — Platform team as a team type
- **Spotify** — Backstage developer portal
- **Humanitec** — Platform engineering advocacy and reference architectures
- **Gregor Hohpe** — Platform strategy and enterprise architecture

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Internal Developer Platform | Self-service layer over infrastructure for developer autonomy |
| Developer Portal | Single pane of glass for services, docs, APIs (e.g., Backstage) |
| Golden Path | Opinionated, paved road for common workflows (deploy, create service) |
| Platform as a Product | Treating the platform with product management: roadmap, feedback, adoption |
| Self-Service | Developers provision resources without tickets or ops intervention |
| Developer Experience (DX) | Measuring and optimizing the developer's daily workflow |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Reduce cognitive load** — Developers should focus on business logic, not infrastructure. The platform abstracts Kubernetes, networking, observability setup.
2. **Self-service with guardrails** — Developers can provision what they need independently, but within safe, compliant boundaries defined by the platform.
3. **Golden paths, not golden cages** — Provide opinionated defaults that cover 80% of use cases. Allow escape hatches for the 20% that need customization.
4. **Platform as a product** — The platform has users (developers), a roadmap, feedback loops, adoption metrics, and iterative improvement. It is not a mandate.
5. **Measure developer experience** — Track DORA metrics, time-to-first-deploy, onboarding time, and developer satisfaction. What gets measured gets improved.

### When to Use vs. When to Avoid

**Invest in platform engineering when:**
- Multiple teams deploying independently (5+ teams)
- Infrastructure complexity is slowing down delivery
- Repeated manual processes for provisioning, deployment, monitoring
- Compliance and security requirements need standardized enforcement

**Keep it lightweight when:**
- Small team (< 3 teams) where shared scripts suffice
- Infrastructure is simple (single app, single environment)
- Team has strong DevOps skills and doesn't need abstraction
- Organization isn't ready for platform product thinking

---

## Frameworks and Methodologies

### 1. Platform Design — step-by-step
1. **Discover**: Interview developer teams — what are their pain points? What takes too long?
2. **Map capabilities**: List platform capabilities needed (deploy, provision DB, create service, view logs)
3. **Define golden paths**: Design the default workflow for top use cases (new service, deploy to prod)
4. **Choose build vs. buy**: Backstage for portal, ArgoCD for GitOps, Crossplane for infra, or custom
5. **Build MVP**: Start with the highest-impact capability (usually deployment pipeline)
6. **Measure adoption**: Track usage, satisfaction, and time savings
7. **Iterate**: Add capabilities based on developer feedback and adoption data

### 2. Developer Portal Implementation — step-by-step
1. Deploy Backstage (or alternative: Port, Cortex, custom)
2. Create software catalog: register all services with ownership, dependencies, docs
3. Add software templates: scaffolding for new services, libraries, pipelines
4. Integrate with CI/CD: show build status, deployment history
5. Add TechDocs: documentation rendered from markdown in repos
6. Integrate with monitoring: show service health, SLOs, on-call
7. Add search and discovery: developers find services, APIs, documentation

---

## How to Apply

### Decision Checklist
- [ ] Are the top developer pain points identified and prioritized?
- [ ] Is there a golden path for the most common workflow (deploy a service)?
- [ ] Is the platform self-service (no tickets required)?
- [ ] Are guardrails in place for security and compliance?
- [ ] Is platform adoption measured and feedback collected?

### Implementation Patterns

**Backstage Software Template (new service scaffolding):**
```yaml
# template.yaml — Backstage scaffolder template
apiVersion: scaffolder.backstage.io/v1beta3
kind: Template
metadata:
  name: new-microservice
  title: Create a New Microservice
  description: Scaffold a production-ready microservice with CI/CD
spec:
  owner: platform-team
  type: service
  parameters:
    - title: Service Info
      required: [name, owner, description]
      properties:
        name:
          title: Service Name
          type: string
          pattern: "^[a-z][a-z0-9-]*$"
        owner:
          title: Owner Team
          type: string
          ui:field: OwnerPicker
        description:
          title: Description
          type: string
    - title: Infrastructure
      properties:
        database:
          title: Database
          type: string
          enum: [none, postgresql, redis]
          default: none
        expose:
          title: Public API
          type: boolean
          default: false
  steps:
    - id: fetch
      name: Fetch Template
      action: fetch:template
      input:
        url: ./skeleton
        values:
          name: ${{ parameters.name }}
          owner: ${{ parameters.owner }}
    - id: publish
      name: Publish to GitHub
      action: publish:github
      input:
        repoUrl: github.com?owner=myorg&repo=${{ parameters.name }}
    - id: register
      name: Register in Catalog
      action: catalog:register
      input:
        repoContentsUrl: ${{ steps.publish.output.repoContentsUrl }}
        catalogInfoPath: /catalog-info.yaml
```

**Self-Service Infrastructure with Crossplane:**
```yaml
# Developer requests a database via Kubernetes CR
apiVersion: database.platform.io/v1alpha1
kind: PostgreSQLInstance
metadata:
  name: orders-db
  namespace: orders-team
spec:
  size: small          # Abstracted: maps to db.t3.medium, 20GB
  version: "15"
  backup:
    enabled: true
    retentionDays: 7
  # Platform handles: VPC, security groups, encryption,
  # credentials (injected as K8s secret), monitoring
```

**Platform Metrics Dashboard:**
```python
# Track platform adoption and developer experience
platform_metrics = {
    # DORA Metrics
    "deployment_frequency": "deploys per team per week",
    "lead_time_for_changes": "commit to production (minutes)",
    "change_failure_rate": "% of deploys causing incidents",
    "mttr": "mean time to recovery (minutes)",

    # Platform-specific
    "time_to_first_deploy": "new dev → first production deploy",
    "self_service_ratio": "% of infra provisioned without tickets",
    "golden_path_adoption": "% of services using golden path",
    "developer_satisfaction": "quarterly NPS survey score",
    "onboarding_time": "new hire → productive contributor",
}
```

### Common Mistakes
1. **Building a platform nobody asked for** — Platform built by infra team without developer input; low adoption
2. **Mandating instead of attracting** — Forcing teams to use the platform instead of making it so good they want to
3. **Too much abstraction** — Hiding everything; developers can't debug when things go wrong
4. **No product management** — Platform without a roadmap, user research, or prioritization; becomes a dumping ground
5. **Premature platform** — Building a platform before understanding the problems; standardizing too early

### Integration with Other Topics
- **Team Topologies** — Platform team is a dedicated team type enabling stream-aligned teams
- **DevOps / CI/CD** — Platform standardizes and automates deployment pipelines
- **Cloud-Native** — Platform abstracts cloud infrastructure (Kubernetes, managed services)
- **Infrastructure as Code** — Platform uses IaC internally but abstracts it from developers
- **Observability** — Platform provides standardized monitoring, logging, and tracing
- **Site Reliability Engineering** — SRE practices embedded in platform capabilities (SLOs, error budgets)

---

## Resources

### Essential Reading
- *Team Topologies* — Skelton & Pais (platform team concept)
- *Platform Engineering on Kubernetes* — Mauricio Salatino
- Humanitec Platform Engineering guide (humanitec.com)

### Online Resources
- Backstage documentation (backstage.io)
- platformengineering.org — Community and resources
- CNCF Platforms Working Group

### Practice Exercises
1. Map developer pain points and design a golden path for deployment
2. Set up Backstage with a software catalog and one template
3. Implement self-service database provisioning with Crossplane
4. Define and measure DORA metrics for your teams
5. Create an onboarding golden path: new developer to first deploy
