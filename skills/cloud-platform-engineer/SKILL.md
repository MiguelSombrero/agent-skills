---
name: cloud-platform-engineer
description: Senior Kubernetes platform specialist for cluster operations, CNI, Gateway API, admission policy (Kyverno, Gatekeeper), service mesh, operators, RBAC, cert-manager, storage, and observability. Use when managing EKS, AKS, GKE, or k3d clusters; configuring network policies, ingress or Gateway API; securing the platform; tuning reliability (SLOs, PDBs); or building cloud-native platforms. Distinguish from application delivery—see skill body for DevOps boundary. Uses k3d for local development.
---

# Cloud Platform Engineer

Builds and operates production-grade Kubernetes platforms: reliability, security, observability, and declarative configuration. Use the routing table below to open the right reference first.

## Scope

**In scope** (platform / cluster):

- Cluster operations, upgrades mindset, node scheduling (taints, affinity, PDBs)
- CNI, NetworkPolicy, Ingress and **Gateway API**
- Admission policy (Kyverno, OPA Gatekeeper) — see [gateway-and-policy.md](references/gateway-and-policy.md)
- Service mesh (Istio, Linkerd, Consul), cert-manager, mTLS
- Operators, CRDs, storage classes, CSI
- RBAC, Pod Security Standards, secrets at rest (cluster-level concerns)
- Observability stack for the platform (Prometheus, Grafana, Jaeger, Loki)
- SLO/SLA thinking, error budgets, capacity and quotas
- Declarative infra and GitOps **for platform components** (install/upgrade ingress, mesh, monitoring, etc.)
- Multi-cluster patterns (provider-specific features when relevant)
- **k3d** for local clusters

**Out of scope** (use [devops-engineer](../devops-engineer/SKILL.md) or app-focused skills):

- CI/CD pipeline design (GitHub Actions, etc.) and release automation
- **Application** Helm chart authoring, app promotion, and **application** GitOps workflows (Argo CD app-of-apps for microservices, etc.)
- Container image build optimization and registry tagging strategies
- “How do I deploy **my** app?” unless the question is clearly about a **platform** component (ingress controller, cert-manager, mesh)

**Platform vs DevOps:** Installing and upgrading **shared platform components** with Helm or GitOps (e.g. ingress-nginx, cert-manager, Prometheus stack) is **platform** work. Owning **application** manifests, app charts, and delivery pipelines is **DevOps**. When in doubt, cluster-wide infrastructure → this skill; shipping application changes → DevOps.

---

## Read first

| Need | Open |
|------|------|
| Local cluster (k3d), namespaces, quotas, **LimitRange**, RBAC, NetworkPolicy, Pod Security | [kubernetes.md](references/kubernetes.md) |
| Gateway API vs Ingress, Kyverno, Gatekeeper | [gateway-and-policy.md](references/gateway-and-policy.md) |
| cert-manager, Istio, Linkerd | [certificates-and-service-mesh.md](references/certificates-and-service-mesh.md) |
| CRDs, operators, storage classes | [operators-and-storage.md](references/operators-and-storage.md) |
| Prometheus, Grafana, Jaeger, Loki | [observability.md](references/observability.md) |
| Multi-cluster ingress (e.g. GKE), scheduling extras, Velero, hardening snippets | [operations.md](references/operations.md) |
| Quick checklist | [checklist.md](references/checklist.md) |

---

## Core principles

- **Declarative and versioned:** Cluster and platform config should be reproducible (Git, templates, GitOps for **platform** stacks)—without tying examples to a specific cloud provisioner in this file.
- **Security by default:** RBAC least privilege, Pod Security Standards, network policy default-deny where appropriate, TLS and identity for east-west traffic when using a mesh.
- **Observe then promise:** Metrics, logs, and traces for platform and workloads; define SLOs and error budgets from real signals, not from example YAML in the skill.
- **Design for failure:** PDBs, spreading, backups (e.g. Velero) where operations.md goes deeper.
- **Clear boundaries:** Platform enables teams; tenant isolation via namespaces, quotas, and policy.

---

## Additional resources

All references live under `references/` and link back to this file. Read only what you need for the task.

- [kubernetes.md](references/kubernetes.md)
- [gateway-and-policy.md](references/gateway-and-policy.md)
- [certificates-and-service-mesh.md](references/certificates-and-service-mesh.md)
- [operators-and-storage.md](references/operators-and-storage.md)
- [observability.md](references/observability.md)
- [operations.md](references/operations.md)
- [checklist.md](references/checklist.md)
