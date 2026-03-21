Reference for cloud-platform-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.

**Note:** Verify pinned versions and install URLs against current upstream releases or official docs before use.

---

## Quick Reference Checklist

**Cluster Setup:**

- [ ] Declarative provisioning or managed control plane (IaC, APIs, or cloud console—per your standards)
- [ ] Multi-zone/region for HA
- [ ] Workload identity configured
- [ ] Private cluster endpoints
- [ ] Network policies enabled
- [ ] Auto-scaling configured

**Gateway and policy:**

- [ ] Ingress or Gateway API approach agreed (see [gateway-and-policy.md](gateway-and-policy.md))
- [ ] Admission policies (Kyverno or Gatekeeper) where organizational rules need more than Pod Security Standards

**Security:**

- [ ] RBAC policies defined
- [ ] Pod Security Standards enforced
- [ ] Network policies implemented
- [ ] Secrets encrypted at rest
- [ ] Certificate management (cert-manager)
- [ ] Service mesh with mTLS

**Observability:**

- [ ] Prometheus metrics collection
- [ ] Grafana dashboards
- [ ] Distributed tracing (Jaeger)
- [ ] Centralized logging (Loki)
- [ ] SLO/SLA monitoring
- [ ] Alert manager configured

**Reliability:**

- [ ] Pod disruption budgets
- [ ] Resource quotas and limits
- [ ] Health checks configured
- [ ] Auto-scaling (HPA/VPA)
- [ ] Backup strategy (Velero)
- [ ] Disaster recovery plan

**Storage:**

- [ ] Storage classes defined
- [ ] Persistent volumes provisioned
- [ ] Snapshot classes configured
- [ ] Backup procedures tested

**Service Mesh:**

- [ ] mTLS between services
- [ ] Traffic management configured
- [ ] Circuit breakers implemented
- [ ] Observability integrated

---
