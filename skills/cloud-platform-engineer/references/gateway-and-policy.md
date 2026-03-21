Reference for cloud-platform-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.

**Note:** Verify CRD versions, Helm chart versions, and install commands against current upstream docs before applying in production.

---

## Gateway API vs Ingress

**Ingress** (`networking.k8s.io/v1 Ingress`): Mature, widely supported. One implementation per cluster (nginx, Traefik, etc.). Good default for HTTP routing when the team already uses classic Ingress and does not need Gateway API features.

**Gateway API** (`gateway.networking.k8s.io`): Successor model: separate **Gateway** (infrastructure: listeners, addresses) and **HTTPRoute** (application: host/path rules). Enables clearer roles (platform owns Gateway; teams own routes), multiple implementations (controllers), and richer traffic splitting. Prefer Gateway API for **new** platform designs where you want standard APIs and multi-team ownership; keep Ingress if the cluster standard is unchanged and requirements are simple.

**When choosing:** If the user asks for canary/weight-based routing, role separation, or portable L4/L7 APIs across implementations, start with Gateway API. If they only need basic TLS + path routing on an existing ingress controller, Ingress may be enough.

Official overview: [Gateway API](https://gateway-api.sigs.k8s.io/) (SIG Network).

---

## Admission policy (Kyverno, OPA Gatekeeper)

**Role:** Validate and mutate API requests at admission time—enforce labels, block privileged pods, require resources, mutate images, deny non-compliant namespaces. Complements **Pod Security Standards** (which enforce pod-level profiles) with **organizational** policy as code.

**Kyverno:** Kubernetes-native policies as YAML (`ClusterPolicy`, `Policy`). No Rego; good fit for teams that want policies in familiar K8s-style resources.

**OPA Gatekeeper:** Rego policies + `ConstraintTemplate` / `Constraint` model; often used when OPA is already standard elsewhere or policies are complex.

**Typical split:** PSS for baseline/restricted pod security; Kyverno or Gatekeeper for naming, labels, image sources, required annotations, cross-field checks, and multi-resource rules.

Use each project’s installation docs for current CRDs and Helm charts rather than pinning versions here.

---
