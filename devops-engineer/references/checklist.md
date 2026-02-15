Reference for devops-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.
---

## Quick Reference Checklist

**CI/CD Pipeline:**

- [ ] Fast feedback (lint/test before build)
- [ ] Parallel jobs where possible
- [ ] Caching for dependencies and builds
- [ ] Security scanning (SAST, dependency, container)
- [ ] Automated tests run on every PR
- [ ] Version tagging strategy
- [ ] Deployment approval for production

**Container Images:**

- [ ] Multi-stage builds for optimization
- [ ] Specific base image versions
- [ ] Non-root user
- [ ] Security scanning
- [ ] .dockerignore file
- [ ] Health check endpoint

**Kubernetes Manifests:**

- [ ] Resource requests and limits set
- [ ] Liveness and readiness probes
- [ ] Security context (non-root, read-only FS)
- [ ] Pod anti-affinity for HA
- [ ] ConfigMaps for configuration
- [ ] Secrets properly managed
- [ ] HPA configured

**Helm Charts:**

- [ ] Values parameterized
- [ ] Multiple environment value files
- [ ] Template helpers used
- [ ] Chart versioning
- [ ] Dependencies declared
- [ ] NOTES.txt with usage info

**GitOps/ArgoCD:**

- [ ] Git as single source of truth
- [ ] Automated sync enabled
- [ ] Sync waves for ordered deployment
- [ ] Health checks configured
- [ ] Proper RBAC and projects
- [ ] Secrets sealed/external

**Deployment:**

- [ ] Rolling update strategy
- [ ] Zero downtime deployments
- [ ] Rollback plan
- [ ] Monitoring and alerts
- [ ] Database migrations handled
- [ ] Progressive rollout for risky changes

---

