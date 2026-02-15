---
name: cloud-platform-engineer
description: Senior Kubernetes platform specialist with expertise in cluster operations, service mesh, operators, security, and infrastructure as code. Use when managing Kubernetes clusters, implementing service mesh, creating operators, configuring network policies, or building cloud-native platforms. Focuses on platform reliability, security, and observability. Uses k3d for local development.
---

# Cloud Platform Engineer

Builds and operates production-grade Kubernetes platforms with focus on reliability, security, observability, and infrastructure as code. Combines deep Kubernetes knowledge with platform engineering best practices.

## When to Use This Skill

Use this skill when the user:

- Wants to **manage Kubernetes clusters** or platform infrastructure
- Needs help with **service mesh** (Istio, Linkerd)
- Asks about **operators** or **Custom Resource Definitions (CRDs)**
- Wants to implement **network policies** or **CNI**
- Needs guidance on **storage classes** or **persistent volumes**
- Asks about **RBAC** or **security policies**
- Wants to set up **certificate management** (cert-manager)
- Needs help with **observability** (Prometheus, Grafana, Jaeger)
- Asks about **SLO/SLA** or **reliability engineering**
- Wants to implement **infrastructure as code** for clusters
- Needs help with **multi-cluster** management
- Wants to configure **ingress controllers** or **load balancers**
- Needs help with **local Kubernetes** setup using **k3d**
- Asks "how do I configure the cluster..." or "how do I secure..."

## Scope

**In Scope** (Platform Engineering):

- Kubernetes cluster management and operations
- Service mesh (Istio, Linkerd, Consul)
- Operators and Custom Resource Definitions
- Network policies and CNI plugins
- RBAC and Pod Security Standards
- Certificate management (cert-manager, mTLS)
- Storage classes and CSI drivers
- Observability (Prometheus, Grafana, Jaeger, Loki)
- SLO/SLA and reliability engineering
- Infrastructure as Code (Terraform, Pulumi)
- Multi-cluster federation
- k3d for local development

**Out of Scope** (DevOps Engineering):

- CI/CD pipelines (GitHub Actions, etc.)
- Helm chart creation and management
- ArgoCD and GitOps workflows
- Application deployment automation
- Container image building
- Release management

**Note**: This skill focuses on platform infrastructure and cluster operations, not application delivery pipelines.

---

## Core Platform Principles

### Infrastructure as Code

**All infrastructure should be declarative and version-controlled.**

```hcl
# Terraform for cluster provisioning
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

resource "google_container_cluster" "primary" {
  name     = "production-cluster"
  location = "us-central1"

  # We can't create a cluster with no node pool defined, but we want to only use
  # separately managed node pools. So we create the smallest possible default
  # node pool and immediately delete it.
  remove_default_node_pool = true
  initial_node_count       = 1

  network    = google_compute_network.vpc.name
  subnetwork = google_compute_subnetwork.subnet.name

  # Enable workload identity
  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }

  # Enable network policy
  network_policy {
    enabled = true
  }

  # Security hardening
  master_auth {
    client_certificate_config {
      issue_client_certificate = false
    }
  }

  # Private cluster
  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false
    master_ipv4_cidr_block  = "172.16.0.0/28"
  }

  # Enable security features
  security_posture_config {
    mode               = "BASIC"
    vulnerability_mode = "VULNERABILITY_ENTERPRISE"
  }

  # Enable monitoring
  monitoring_config {
    enable_components = ["SYSTEM_COMPONENTS", "WORKLOADS"]
    managed_prometheus {
      enabled = true
    }
  }

  # Enable logging
  logging_config {
    enable_components = ["SYSTEM_COMPONENTS", "WORKLOADS"]
  }
}

resource "google_container_node_pool" "primary_nodes" {
  name       = "production-node-pool"
  location   = "us-central1"
  cluster    = google_container_cluster.primary.name
  node_count = 3

  autoscaling {
    min_node_count = 3
    max_node_count = 10
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }

  node_config {
    machine_type = "e2-standard-4"
    disk_size_gb = 100
    disk_type    = "pd-standard"

    # Google recommends custom service accounts with minimal permissions
    service_account = google_service_account.gke_node.email
    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    # Security hardening
    shielded_instance_config {
      enable_secure_boot          = true
      enable_integrity_monitoring = true
    }

    workload_metadata_config {
      mode = "GKE_METADATA"
    }

    labels = {
      environment = "production"
      managed_by  = "terraform"
    }

    tags = ["gke-node", "production"]
  }
}
```

### Platform Reliability

**Design for failure, measure everything, continuous improvement.**

```yaml
# SLO Definition
apiVersion: v1
kind: ConfigMap
metadata:
  name: slo-definitions
  namespace: monitoring
data:
  api-availability-slo.yaml: |
    slo:
      name: api-availability
      description: API endpoints should be available 99.9% of the time
      service: api-gateway
      sli:
        name: availability
        type: availability
        query: |
          sum(rate(http_requests_total{job="api-gateway",code!~"5.."}[5m]))
          /
          sum(rate(http_requests_total{job="api-gateway"}[5m]))
      objective:
        target: 0.999  # 99.9%
        window: 30d
      alerting:
        burn_rate: 
          - threshold: 14.4  # 1h window
            severity: critical
          - threshold: 6     # 6h window
            severity: warning

  api-latency-slo.yaml: |
    slo:
      name: api-latency
      description: 95% of API requests should complete within 200ms
      service: api-gateway
      sli:
        name: latency
        type: latency
        query: |
          histogram_quantile(0.95,
            sum(rate(http_request_duration_seconds_bucket{job="api-gateway"}[5m])) by (le)
          )
      objective:
        target: 0.2  # 200ms
        window: 30d
```

---


---

## Additional Resources

For detailed patterns and examples, see these reference files:

- **Kubernetes and RBAC**: [kubernetes.md](references/kubernetes.md) — Cluster setup, k3d, RBAC, Pod Security
- **Certificates and service mesh**: [certificates-and-service-mesh.md](references/certificates-and-service-mesh.md) — cert-manager, Istio, Linkerd
- **Operators and storage**: [operators-and-storage.md](references/operators-and-storage.md) — CRDs, operators, storage classes
- **Observability**: [observability.md](references/observability.md) — Prometheus, Grafana, Jaeger, Loki
- **Operations**: [operations.md](references/operations.md) — Multi-cluster, best practices, disaster recovery
- **Checklist**: [checklist.md](references/checklist.md) — Quick reference

---

## Summary

**Key Principles:**

- Infrastructure as Code
- Security by default
- Observe everything
- Design for failure
- Automate operations
- Multi-tenancy support

**Remember**: Platform engineering is about building reliable, secure, and observable infrastructure that enables developers to ship software quickly and safely.
