# Infrastructure as Code

## Profile

### What It Is
Infrastructure as Code (IaC) is the practice of managing and provisioning computing infrastructure through machine-readable configuration files rather than manual processes. It applies software engineering practices — version control, testing, code review, CI/CD — to infrastructure management.

### Origin and Evolution
- CFEngine (1993) — first configuration management tool
- Puppet (2005), Chef (2009), Ansible (2012) — configuration management era
- Terraform (2014, HashiCorp) — declarative, cloud-agnostic infrastructure provisioning
- Pulumi (2018) — IaC using general-purpose programming languages
- AWS CDK (2019) — cloud-specific IaC with programming languages
- Current: GitOps (ArgoCD, Flux) applies IaC principles to Kubernetes

### Key Authors and References
- **Kief Morris** — *Infrastructure as Code* (O'Reilly, 2nd ed. 2020)
- **Mitchell Hashimoto** — Terraform creator (HashiCorp)
- **Yevgeniy Brikman** — *Terraform: Up & Running*

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Declarative | Define desired state; the tool figures out how to get there |
| Imperative | Define step-by-step instructions to reach desired state |
| Idempotent | Applying the same config multiple times produces the same result |
| State Management | Tracking what's actually provisioned vs. what's defined |
| Drift Detection | Identifying when actual infrastructure diverges from code |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Everything is code** — Infrastructure definitions, policies, configurations are version-controlled code. No manual changes.
2. **Declarative over imperative** — Define what the infrastructure should look like, not the steps to create it.
3. **Idempotency** — Running the same code produces the same result regardless of current state.
4. **Immutable infrastructure** — Don't patch running servers. Build new ones from code and replace the old ones.
5. **Test infrastructure code** — Lint, validate, plan, and test infrastructure changes before applying them.

### When to Use vs. When to Avoid

**Use when:**
- Managing cloud resources (AWS, GCP, Azure)
- Multiple environments (dev, staging, production)
- Team collaboration on infrastructure
- Compliance requires audit trails for infrastructure changes
- Infrastructure needs to be reproducible

**Avoid when:**
- Single developer, single server, infrequent changes
- Learning/prototyping where manual setup is faster
- The overhead of IaC tooling exceeds the benefit

---

## Frameworks and Methodologies

### 1. IaC Workflow — step-by-step
1. Write infrastructure code (Terraform, Pulumi, CDK)
2. Version control the code (Git)
3. Review changes (pull request with plan output)
4. Validate: lint, format, plan/preview changes
5. Apply to staging environment first
6. Apply to production after validation
7. Monitor for drift

### 2. Module Design — step-by-step
1. Identify reusable infrastructure patterns (VPC, ECS cluster, database)
2. Extract each pattern into a module with inputs/outputs
3. Version modules independently (semantic versioning)
4. Compose modules to build environments
5. Test modules with automated infrastructure tests
6. Publish modules to a registry for team use

---

## How to Apply

### Decision Checklist
- [ ] Is all infrastructure defined in code (no manual changes)?
- [ ] Is the code version-controlled with code review?
- [ ] Are there separate state files per environment?
- [ ] Is there a CI/CD pipeline for infrastructure changes?
- [ ] Is the state backend secure and locked?

### Implementation Patterns

**Terraform Example:**
```hcl
# modules/api_service/main.tf
resource "aws_ecs_service" "api" {
  name            = var.service_name
  cluster         = var.cluster_id
  task_definition = aws_ecs_task_definition.api.arn
  desired_count   = var.desired_count

  load_balancer {
    target_group_arn = aws_lb_target_group.api.arn
    container_name   = var.service_name
    container_port   = var.container_port
  }
}

resource "aws_lb_target_group" "api" {
  name     = "${var.service_name}-tg"
  port     = var.container_port
  protocol = "HTTP"
  vpc_id   = var.vpc_id

  health_check {
    path                = "/health"
    healthy_threshold   = 2
    unhealthy_threshold = 3
  }
}

# environments/production/main.tf
module "order_service" {
  source        = "../../modules/api_service"
  service_name  = "order-service"
  cluster_id    = module.ecs_cluster.id
  desired_count = 3
  container_port = 8080
  vpc_id        = module.vpc.id
}
```

**Pulumi (Python):**
```python
import pulumi
import pulumi_aws as aws

# Reusable component
class ApiService(pulumi.ComponentResource):
    def __init__(self, name: str, cluster_id: str, image: str,
                 desired_count: int = 2, **kwargs):
        super().__init__("custom:ApiService", name, {}, **kwargs)

        self.task_def = aws.ecs.TaskDefinition(f"{name}-task",
            family=name,
            container_definitions=pulumi.Output.json_dumps([{
                "name": name,
                "image": image,
                "portMappings": [{"containerPort": 8080}],
                "essential": True,
            }]),
            opts=pulumi.ResourceOptions(parent=self),
        )

        self.service = aws.ecs.Service(f"{name}-service",
            cluster=cluster_id,
            task_definition=self.task_def.arn,
            desired_count=desired_count,
            opts=pulumi.ResourceOptions(parent=self),
        )

# Usage
order_service = ApiService("order-service",
    cluster_id=cluster.id,
    image="order-service:latest",
    desired_count=3,
)
```

### Common Mistakes
1. **Manual changes after IaC** — "Just this once" manual change creates drift and breaks future applies
2. **Monolithic state** — One state file for entire infrastructure; one change risks everything
3. **No state locking** — Two people applying simultaneously corrupt the state
4. **Secrets in code** — Database passwords in Terraform files instead of secret management
5. **No plan review** — Applying without reviewing the plan leads to accidental deletions

### Integration with Other Topics
- **Cloud-Native Architecture** — IaC provisions the cloud infrastructure
- **CI/CD Pipelines** — Infrastructure changes flow through CI/CD
- **GitOps** — IaC + Git as single source of truth for infrastructure
- **Containerization** — Docker images + IaC-managed infrastructure
- **Observability** — IaC provisions monitoring infrastructure
- **Security** — Policy-as-code enforces security guardrails

---

## Resources

### Essential Reading
- *Infrastructure as Code* (2nd ed.) — Kief Morris
- *Terraform: Up & Running* (3rd ed.) — Yevgeniy Brikman
- Pulumi documentation (pulumi.com/docs)

### Online Resources
- Terraform Registry (registry.terraform.io)
- AWS CDK documentation
- HashiCorp Learn tutorials

### Practice Exercises
1. Define a VPC with subnets using Terraform
2. Create a reusable module for a web service (ALB + ECS + RDS)
3. Set up remote state with locking (S3 + DynamoDB)
4. Implement a CI/CD pipeline for Terraform changes
5. Detect and fix infrastructure drift
