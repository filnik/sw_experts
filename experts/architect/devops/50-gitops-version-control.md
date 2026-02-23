# GitOps and Version Control

## Profile

### What It Is
GitOps is an operational framework where Git is the single source of truth for declarative infrastructure and application configurations. Changes to production are made through Git commits and pull requests, with automated reconciliation ensuring the live system matches the desired state in Git.

### Origin and Evolution
- Git (Linus Torvalds, 2005) — distributed version control
- GitFlow (Vincent Driessen, 2010), trunk-based development emerged as branching strategies
- GitOps coined by Weaveworks (Alexis Richardson, 2017)
- ArgoCD and Flux became the standard GitOps tools for Kubernetes
- Current: GitOps beyond Kubernetes, multi-cluster GitOps, progressive delivery

### Key Authors and References
- **Alexis Richardson** — GitOps term and Weaveworks
- **Linus Torvalds** — Git creator
- **Vincent Driessen** — GitFlow branching model
- **CNCF** — GitOps working group and principles

### Key Concepts at a Glance
| Concept | Description |
|---------|-------------|
| Declarative | Desired state described in Git; system reconciles to match |
| Reconciliation | Automated process that compares Git state with live state |
| Drift Detection | Identifying when live state diverges from Git |
| Pull-based | Agents in the cluster pull changes from Git (vs. push from CI) |
| Trunk-based Development | Short-lived branches, frequent merges to main |

---

## Core Philosophy

### The 5 Fundamental Principles
1. **Git is the single source of truth** — Everything (app config, infra, policies) lives in Git. The live system is a reflection of Git state.
2. **Declarative desired state** — Describe what you want, not how to get there. Tools reconcile actual state to desired state.
3. **Automated reconciliation** — An agent continuously ensures the live system matches Git. Manual changes are overwritten.
4. **Pull, don't push** — The cluster pulls desired state from Git rather than CI pushing changes. This is more secure (no cluster credentials in CI).
5. **Audit trail for free** — Git provides a complete, immutable history of every change, who made it, when, and why.

### When to Use vs. When to Avoid

**Use GitOps when:**
- Managing Kubernetes deployments
- Multiple environments need consistent configuration
- Audit trail and compliance are required
- Team wants self-service deployment via PRs

**Use simpler approaches when:**
- No Kubernetes (though GitOps concepts apply elsewhere)
- Very simple infrastructure
- The team isn't ready for declarative configuration management

---

## Frameworks and Methodologies

### 1. GitOps Workflow — step-by-step
1. Define all configuration in Git (Kubernetes manifests, Helm charts, Kustomize)
2. Install GitOps agent (ArgoCD or Flux) in the cluster
3. Configure agent to watch the Git repository
4. Make changes via pull requests (reviewed and approved)
5. Agent detects changes and reconciles the cluster
6. Monitor sync status and drift alerts

### 2. Branching Strategy — step-by-step
1. **Trunk-based**: All developers commit to main (or short-lived branches <1 day)
2. Use feature flags for incomplete features (not long-lived branches)
3. PRs are small and reviewed quickly
4. Main is always deployable
5. Tags or release branches for production releases if needed
6. Avoid: long-lived feature branches, GitFlow for web services

---

## How to Apply

### Decision Checklist
- [ ] Is all configuration stored in Git (no manual cluster changes)?
- [ ] Is a GitOps agent managing reconciliation?
- [ ] Are changes made exclusively through pull requests?
- [ ] Is drift detection enabled and alerting?
- [ ] Is the branching strategy supporting continuous delivery?

### Implementation Patterns

**ArgoCD Application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: order-service
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/k8s-config.git
    targetRevision: main
    path: apps/order-service/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true          # Delete resources removed from Git
      selfHeal: true       # Revert manual changes in cluster
    syncOptions:
      - CreateNamespace=true
```

**Repository Structure:**
```
k8s-config/
  apps/
    order-service/
      base/                 # Shared configuration
        deployment.yaml
        service.yaml
        kustomization.yaml
      staging/              # Staging overrides
        kustomization.yaml
        replicas-patch.yaml
      production/           # Production overrides
        kustomization.yaml
        replicas-patch.yaml
        hpa.yaml
  infrastructure/
    monitoring/
    ingress/
```

### Common Mistakes
1. **Manual kubectl changes** — GitOps agent will revert them; all changes must go through Git
2. **Secrets in Git** — Use Sealed Secrets, SOPS, or External Secrets Operator
3. **One repo for everything** — Separate app config repo from source code repo for GitOps
4. **No drift alerting** — Someone makes a manual change and GitOps agent silently reverts it
5. **Long-lived branches** — Branches that live for weeks create merge conflicts and delay delivery

### Integration with Other Topics
- **CI/CD Pipelines** — CI builds artifacts; GitOps deploys them
- **Infrastructure as Code** — IaC configs managed via GitOps
- **Deployment Strategies** — ArgoCD Rollouts for progressive delivery
- **Containerization** — Kubernetes manifests managed by GitOps
- **Security** — Git provides audit trail; PRs enforce review
- **Observability** — Monitor GitOps sync status and drift

---

## Resources

### Essential Reading
- *GitOps and Kubernetes* — Billy Yuen et al.
- ArgoCD documentation (argo-cd.readthedocs.io)
- Flux documentation (fluxcd.io)

### Online Resources
- OpenGitOps principles (opengitops.dev)
- Weaveworks GitOps resources
- CNCF GitOps working group

### Practice Exercises
1. Set up ArgoCD and deploy an application from Git
2. Make a change via PR and observe automated sync
3. Introduce drift manually and verify self-healing
4. Implement multi-environment setup with Kustomize overlays
5. Manage secrets with Sealed Secrets in a GitOps workflow
