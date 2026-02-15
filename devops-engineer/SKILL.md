---
name: devops-engineer
description: Designs and implements CI/CD pipelines, GitOps workflows, and deployment automation with expertise in GitHub Actions, Kubernetes, Helm, ArgoCD, and containerization. Use when automating builds, releases, deployments, or setting up GitOps workflows. Focuses on pipeline optimization, container orchestration, and declarative infrastructure.
---

# DevOps Engineer

Automates software delivery through robust CI/CD pipelines, GitOps practices, and container orchestration. Combines development and operations expertise to enable fast, reliable, and repeatable deployments.

## When to Use This Skill

Use this skill when the user:

- Wants to **set up CI/CD pipelines** or **automate deployments**
- Needs help with **GitHub Actions** workflows or optimization
- Asks about **Kubernetes** deployments, manifests, or configurations
- Wants to create or modify **Helm charts**
- Needs guidance on **GitOps** with **ArgoCD**
- Wants to **containerize applications** or optimize Docker builds
- Asks about **deployment strategies** (rolling, blue-green, canary)
- Needs help with **secrets management** in pipelines or Kubernetes
- Wants to set up **local Kubernetes** development with k3d
- Asks about **container registries**, image tagging, or versioning
- Needs to **automate testing** in CI/CD pipelines
- Wants to implement **release workflows** or versioning strategies
- Asks "how do I deploy..." or "how do I automate..."

## Scope

**In Scope** (DevOps/CI/CD):

- CI/CD pipeline design and implementation
- GitHub Actions workflows
- Container building and optimization
- Kubernetes deployments and configurations
- Helm charts and templating
- ArgoCD and GitOps workflows
- Deployment strategies and rollbacks
- Secrets management (GitHub Secrets, Kubernetes Secrets, Sealed Secrets)
- Container registry management
- Build optimization and caching
- Pipeline security and best practices

**Out of Scope** (Cloud Platform Engineering):

- Cloud provider infrastructure (AWS, GCP, Azure)
- Service mesh (Istio, Linkerd)
- Operators and CRDs
- SLA/SLO monitoring and reliability engineering
- Multi-cluster management
- Advanced observability and tracing
- Network policies and CNI
- Storage classes and PV management

**Note**: This skill focuses on application delivery automation. For infrastructure and platform concerns, use the **cloud-platform-engineer** skill.

---

## Core DevOps Principles

### The Three Ways

#### 1. Flow (Left to Right)

**Optimize the flow from development to production.**

```yaml
# Fast, automated pipeline
Development → Build → Test → Stage → Production
↓          ↓       ↓       ↓         ↓
< 5min    < 10min  < 15min < 5min   < 10min
```

#### 2. Feedback (Right to Left)

**Fast feedback from production to development.**

```yaml
# Monitoring and alerts flow back
Production → Monitoring → Alerts → Development
↓           ↓          ↓           ↓
Metrics    Dashboards  Incidents   Fixes
```

#### 3. Continuous Learning

**Culture of experimentation and learning.**

### GitOps Principles

#### 1. Declarative Configuration

**Entire system described declaratively.**

```yaml
# Git repository contains desired state
app-repo/
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── helm/
├── Chart.yaml
├── values.yaml
└── templates/
```

#### 2. Git as Single Source of Truth

**Git is the single source of truth for system state.**

```bash
# Any change requires Git commit
git commit -m "Update app to v2.0.0"
git push origin main

# ArgoCD automatically syncs
```

#### 3. Automated Deployment

**Changes are automatically applied to system.**

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  syncPolicy:
    automated:
      prune: true # Remove resources not in Git
      selfHeal: true # Sync if manual changes detected
```

#### 4. Continuous Reconciliation

**Agents continuously reconcile desired and actual state.**

---


---

## Additional Resources

For detailed patterns and examples, see these reference files:

- **GitHub Actions**: [github-actions.md](references/github-actions.md) — Workflows, jobs, caching
- **Docker and Kubernetes**: [docker-and-kubernetes.md](references/docker-and-kubernetes.md) — Containerization, deployments
- **Helm and ArgoCD**: [helm-and-argocd.md](references/helm-and-argocd.md) — Helm charts, GitOps
- **k3d and secrets**: [k3d-and-secrets.md](references/k3d-and-secrets.md) — Local Kubernetes, secrets management
- **Deployment and monitoring**: [deployment-and-monitoring.md](references/deployment-and-monitoring.md) — Strategies, CI/CD monitoring
- **Best practices and anti-patterns**: [best-practices-and-anti-patterns.md](references/best-practices-and-anti-patterns.md)
- **Checklist**: [checklist.md](references/checklist.md) — Quick reference

---

## Summary

**Key Principles:**

- Automate everything
- Git as single source of truth
- Fail fast with quick feedback
- Security throughout pipeline
- Infrastructure as code
- Declarative over imperative
- Monitor and measure

**Remember**: Good DevOps enables developers to ship code confidently and quickly while maintaining stability and security.
