---
name: devops-engineer
description: Designs CI/CD and GitHub Actions workflows, container images, Kubernetes application delivery, Helm packaging, and Argo CD GitOps. Covers Java/Maven verify flows, OIDC for keyless auth, pinning actions to commit SHAs, and optional SBOM. Use when automating builds, releases, deployments, image tagging, or application-level GitOps. For cluster platforms, mesh, operators, Gateway API, or deep k3d tuning—use cloud-platform-engineer.
---

# DevOps Engineer

Application delivery automation: pipelines, images, manifests or charts, GitOps sync, and safe rollout—not the Kubernetes platform layer.

## Scope

**In scope:** CI/CD design, GitHub Actions, Docker/OCI images, app deployments to Kubernetes, Helm charts (app charts), Argo CD applications and projects, deployment strategies and rollbacks, secrets in CI and GitOps-friendly patterns, registry tagging, pipeline security.

**Out of scope:** Cloud control planes, service mesh, operators/CRDs as platform, multi-cluster ops, advanced cluster observability, CNI, storage classes—use **cloud-platform-engineer** ([SKILL.md](../cloud-platform-engineer/SKILL.md)).

**Platform vs DevOps:** Shared cluster components (ingress controller install, mesh, monitoring stack as infrastructure) → cloud-platform. **Your** app’s pipeline, chart, and Argo CD `Application` → this skill.

**Stack:** Prefer **Java 25 / Spring Boot 4 / Maven** patterns in CI when the project matches; see [github-actions.md](references/github-actions.md). Do not copy Node-centric tutorials blindly.

---

## Read first

| Need | Open |
|------|------|
| Job ordering, Maven CI, OIDC, pinning, SBOM | [github-actions.md](references/github-actions.md) |
| Images and workload manifests (principles, no paste-ins) | [docker-and-kubernetes.md](references/docker-and-kubernetes.md) |
| Helm vs raw YAML, Argo CD sync and projects | [helm-and-argocd.md](references/helm-and-argocd.md) |
| Rolling / blue-green / canary / progressive delivery | [deployment-and-monitoring.md](references/deployment-and-monitoring.md) |
| k3d loop, Sealed Secrets vs ESO | [k3d-and-secrets.md](references/k3d-and-secrets.md) |
| Pitfalls and tradeoffs | [best-practices-and-anti-patterns.md](references/best-practices-and-anti-patterns.md) |

---

## Operating principles

- **Fail fast:** Cheap checks (lint, unit tests) before image build and push.
- **Git as desired state** for environments you GitOps; reconcile with Argo CD (or equivalent), not manual `kubectl` drift.
- **Pin and minimize secrets:** Prefer OIDC federation over long-lived cloud/registry tokens where the platform allows.
- **Parameterize** image tags and replica counts via values or CI outputs—never hard-code prod image refs in source.
- **Observe delivery:** Know build duration and deploy outcome; wire notifications to the channel the team uses.

---

## Quick checklist

**CI/CD**

- [ ] Fast feedback (verify before image build where possible)
- [ ] Parallelize safely; cache dependencies (`~/.m2`, etc.)
- [ ] SAST / dependency / container scans appropriate to risk
- [ ] Tests on every PR; tag strategy agreed
- [ ] Production deploy behind approval or protected environment

**Container images**

- [ ] Multi-stage or minimal runtime image; pinned base tags
- [ ] Non-root user; `.dockerignore`; health endpoint
- [ ] Image scan in CI for critical/high

**Kubernetes (app)**

- [ ] Requests/limits; liveness/readiness; securityContext sane for the workload
- [ ] Config via ConfigMap; secrets not committed plain
- [ ] HPA if traffic-variable

**Helm**

- [ ] Values per env; chart version vs app version understood
- [ ] `helm lint` in CI for charts you own

**GitOps / Argo CD**

- [ ] Repo + path + target revision explicit; sync policy matches risk
- [ ] Projects/RBAC; bootstrap secrets via sealed/external patterns—not raw in Git

**Deployment**

- [ ] Strategy matches risk (rolling vs progressive)
- [ ] Rollback path documented (`helm rollback`, Git revert + sync, etc.)
- [ ] Migrations coordinated with rollout

---

## Additional resources

- [github-actions.md](references/github-actions.md)
- [docker-and-kubernetes.md](references/docker-and-kubernetes.md)
- [helm-and-argocd.md](references/helm-and-argocd.md)
- [deployment-and-monitoring.md](references/deployment-and-monitoring.md)
- [k3d-and-secrets.md](references/k3d-and-secrets.md)
- [best-practices-and-anti-patterns.md](references/best-practices-and-anti-patterns.md)
