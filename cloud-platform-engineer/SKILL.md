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

## Kubernetes Cluster Setup

### k3d for Local Development

```bash
# Install k3d
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

# Create production-like cluster
k3d cluster create platform \
  --servers 3 \
  --agents 3 \
  --registry-create registry.localhost:5000 \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer \
  --k3s-arg "--disable=traefik@server:*" \
  --k3s-arg "--disable=servicelb@server:*" \
  --volume /tmp/k3d-storage:/storage@all \
  --api-port 6550

# Install MetalLB for LoadBalancer support
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml

# Wait for MetalLB to be ready
kubectl wait --namespace metallb-system \
  --for=condition=ready pod \
  --selector=app=metallb \
  --timeout=90s

# Configure MetalLB IP address pool
docker network inspect k3d-platform | jq '.[0].IPAM.Config[0].Subnet' -r
# Use a range from the Docker network (e.g., 172.20.0.100-172.20.0.150)

cat <<EOF | kubectl apply -f -
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: default
  namespace: metallb-system
spec:
  addresses:
  - 172.20.0.100-172.20.0.150
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: default
  namespace: metallb-system
spec:
  ipAddressPools:
  - default
EOF
```

### Cluster Configuration as Code

```yaml
# cluster-config.yaml - Applied with kubectl or gitops
---
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-nginx
  labels:
    name: ingress-nginx
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted

---
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager
  labels:
    name: cert-manager
    pod-security.kubernetes.io/enforce: baseline

---
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
  labels:
    name: monitoring
    pod-security.kubernetes.io/enforce: baseline

---
apiVersion: v1
kind: Namespace
metadata:
  name: istio-system
  labels:
    name: istio-system
    istio-injection: disabled

---
# Resource quotas for tenant namespaces
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-quota
  namespace: tenant-apps
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    persistentvolumeclaims: "10"
    services.loadbalancers: "2"

---
# Limit ranges
apiVersion: v1
kind: LimitRange
metadata:
  name: tenant-limits
  namespace: tenant-apps
spec:
  limits:
    - max:
        cpu: "4"
        memory: 8Gi
      min:
        cpu: "100m"
        memory: 128Mi
      default:
        cpu: "500m"
        memory: 512Mi
      defaultRequest:
        cpu: "200m"
        memory: 256Mi
      type: Container
```

---

## RBAC and Security

### Pod Security Standards

```yaml
# Enforce Pod Security Standards at namespace level
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    # Privileged: Unrestricted (for system components)
    # Baseline: Minimally restrictive (default)
    # Restricted: Heavily restricted (production apps)
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted

---
# Example: Restricted workload
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secure-app
  namespace: production
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault

      containers:
        - name: app
          image: app:v1.0.0
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: cache
              mountPath: /app/cache

      volumes:
        - name: tmp
          emptyDir: {}
        - name: cache
          emptyDir: {}
```

### RBAC Policies

```yaml
# ServiceAccount for application
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: production

---
# Role - namespace-scoped permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["configmaps", "secrets"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]

---
# RoleBinding - bind role to service account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-role-binding
  namespace: production
subjects:
  - kind: ServiceAccount
    name: app-service-account
    namespace: production
roleRef:
  kind: Role
  name: app-role
  apiGroup: rbac.authorization.k8s.io

---
# ClusterRole - cluster-wide permissions
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-monitoring-view
rules:
  - apiGroups: [""]
    resources: ["nodes", "nodes/metrics", "services", "endpoints", "pods"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get"]
  - nonResourceURLs: ["/metrics"]
    verbs: ["get"]

---
# ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-cluster-view
subjects:
  - kind: ServiceAccount
    name: prometheus
    namespace: monitoring
roleRef:
  kind: ClusterRole
  name: cluster-monitoring-view
  apiGroup: rbac.authorization.k8s.io

---
# User access via ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: developers-view
subjects:
  - kind: Group
    name: developers
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view # Built-in role
  apiGroup: rbac.authorization.k8s.io
```

### Network Policies

```yaml
# Default deny all ingress traffic
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress

---
# Allow ingress from ingress controller
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-ingress
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: web
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8080

---
# Allow communication within namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector: {}

---
# Allow specific pod-to-pod communication
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-to-database
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: postgres
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: api
      ports:
        - protocol: TCP
          port: 5432

---
# Egress policy - allow DNS and external HTTPS
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-and-https
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              name: kube-system
      ports:
        - protocol: UDP
          port: 53
    - to:
        - podSelector: {} # Same namespace
    - ports:
        - protocol: TCP
          port: 443
```

---

## Certificate Management

### cert-manager Installation

```yaml
# Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.yaml

# Verify installation
kubectl get pods -n cert-manager
kubectl get crd | grep cert-manager

---
# ClusterIssuer for Let's Encrypt (staging)
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: nginx

---
# ClusterIssuer for Let's Encrypt (production)
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
    - dns01:
        cloudflare:
          email: admin@example.com
          apiTokenSecretRef:
            name: cloudflare-api-token
            key: api-token

---
# Certificate resource
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: app-tls
  namespace: production
spec:
  secretName: app-tls-secret
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - app.example.com
  - api.example.com

---
# Ingress with automatic certificate
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls-secret
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

### Internal CA for mTLS

```yaml
# Self-signed CA
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}

---
# Root CA Certificate
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: root-ca
  namespace: cert-manager
spec:
  isCA: true
  commonName: internal-ca
  secretName: root-ca-secret
  privateKey:
    algorithm: ECDSA
    size: 256
  issuerRef:
    name: selfsigned-issuer
    kind: ClusterIssuer
    group: cert-manager.io

---
# CA Issuer
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: internal-ca
spec:
  ca:
    secretName: root-ca-secret

---
# Service certificate for mTLS
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: service-cert
  namespace: production
spec:
  secretName: service-tls
  duration: 2160h # 90 days
  renewBefore: 360h # 15 days
  subject:
    organizations:
      - example-org
  commonName: api-service.production.svc.cluster.local
  dnsNames:
    - api-service.production.svc.cluster.local
    - api-service.production.svc
    - api-service
  issuerRef:
    name: internal-ca
    kind: ClusterIssuer
  usages:
    - server auth
    - client auth
```

---

## Service Mesh

### Istio Installation and Configuration

```bash
# Install Istio
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH

# Install with production profile
istioctl install --set profile=production -y

# Verify installation
kubectl get pods -n istio-system
```

```yaml
# Enable sidecar injection for namespace
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    istio-injection: enabled

---
# Gateway - ingress for mesh
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: app-gateway
  namespace: production
spec:
  selector:
    istio: ingressgateway
  servers:
    - port:
        number: 443
        name: https
        protocol: HTTPS
      tls:
        mode: SIMPLE
        credentialName: app-tls-secret
      hosts:
        - "app.example.com"

---
# VirtualService - routing rules
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: app-routes
  namespace: production
spec:
  hosts:
    - "app.example.com"
  gateways:
    - app-gateway
  http:
    - match:
        - uri:
            prefix: "/api"
      route:
        - destination:
            host: api-service
            port:
              number: 8080
          weight: 90
        - destination:
            host: api-service-canary
            port:
              number: 8080
          weight: 10
      timeout: 30s
      retries:
        attempts: 3
        perTryTimeout: 10s
        retryOn: 5xx,reset,connect-failure,refused-stream

---
# DestinationRule - traffic policies
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: api-service
  namespace: production
spec:
  host: api-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 50
        http2MaxRequests: 100
        maxRequestsPerConnection: 2
    loadBalancer:
      simple: LEAST_REQUEST
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2

---
# PeerAuthentication - enforce mTLS
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: production
spec:
  mtls:
    mode: STRICT

---
# AuthorizationPolicy - service-to-service authz
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: api-service-authz
  namespace: production
spec:
  selector:
    matchLabels:
      app: api-service
  action: ALLOW
  rules:
    - from:
        - source:
            principals:
              - "cluster.local/ns/production/sa/web-app"
      to:
        - operation:
            methods: ["GET", "POST"]
            paths: ["/api/*"]

---
# ServiceEntry - external service
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: external-database
  namespace: production
spec:
  hosts:
    - database.external.com
  ports:
    - number: 5432
      name: postgres
      protocol: TCP
  location: MESH_EXTERNAL
  resolution: DNS

---
# Sidecar - optimize sidecar config
apiVersion: networking.istio.io/v1beta1
kind: Sidecar
metadata:
  name: default
  namespace: production
spec:
  egress:
    - hosts:
        - "./*" # Same namespace
        - "istio-system/*" # Istio components
```

### Circuit Breaking and Fault Injection

```yaml
# Circuit breaker
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: circuit-breaker
  namespace: production
spec:
  host: api-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 10
        http2MaxRequests: 100
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
      minHealthPercent: 50

---
# Fault injection for testing
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: fault-injection
  namespace: production
spec:
  hosts:
    - api-service
  http:
    - match:
        - headers:
            test-fault:
              exact: "true"
      fault:
        delay:
          percentage:
            value: 10
          fixedDelay: 5s
        abort:
          percentage:
            value: 5
          httpStatus: 503
      route:
        - destination:
            host: api-service
```

---

## Operators and Custom Resources

### Creating Custom Resource Definition

```yaml
# CRD for Application resource
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: applications.platform.example.com
spec:
  group: platform.example.com
  versions:
    - name: v1alpha1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                image:
                  type: string
                replicas:
                  type: integer
                  minimum: 1
                  maximum: 10
                resources:
                  type: object
                  properties:
                    cpu:
                      type: string
                    memory:
                      type: string
                ingress:
                  type: object
                  properties:
                    enabled:
                      type: boolean
                    host:
                      type: string
                    tls:
                      type: boolean
              required:
                - image
            status:
              type: object
              properties:
                state:
                  type: string
                deploymentStatus:
                  type: string
                serviceStatus:
                  type: string
      subresources:
        status: {}
      additionalPrinterColumns:
        - name: Image
          type: string
          jsonPath: .spec.image
        - name: Replicas
          type: integer
          jsonPath: .spec.replicas
        - name: State
          type: string
          jsonPath: .status.state
        - name: Age
          type: date
          jsonPath: .metadata.creationTimestamp
  scope: Namespaced
  names:
    plural: applications
    singular: application
    kind: Application
    shortNames:
      - app

---
# Example Application CR
apiVersion: platform.example.com/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: production
spec:
  image: my-app:v1.0.0
  replicas: 3
  resources:
    cpu: "500m"
    memory: "512Mi"
  ingress:
    enabled: true
    host: my-app.example.com
    tls: true
```

---

## Storage and Persistence

### Storage Classes

```yaml
# Local SSD storage class (high performance)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete

---
# Network storage (replicated, durable)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard-replicated
provisioner: pd.csi.storage.gke.io # GKE example
parameters:
  type: pd-standard
  replication-type: regional-pd
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Retain

---
# SSD network storage (high performance + durable)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-replicated
provisioner: pd.csi.storage.gke.io
parameters:
  type: pd-ssd
  replication-type: regional-pd
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true

---
# PersistentVolume with local storage
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv-node1
spec:
  capacity:
    storage: 100Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: fast-ssd
  local:
    path: /mnt/disks/ssd1
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - node1

---
# StatefulSet with persistent storage
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  namespace: database
spec:
  serviceName: postgres
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:15-alpine
          ports:
            - containerPort: 5432
              name: postgres
          volumeMounts:
            - name: data
              mountPath: /var/lib/postgresql/data
          env:
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-credentials
                  key: password
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: fast-replicated
        resources:
          requests:
            storage: 50Gi
```

### Volume Snapshots

```yaml
# VolumeSnapshotClass
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: csi-snapshot-class
driver: pd.csi.storage.gke.io
deletionPolicy: Delete

---
# Create snapshot
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: postgres-snapshot-20240115
  namespace: database
spec:
  volumeSnapshotClassName: csi-snapshot-class
  source:
    persistentVolumeClaimName: data-postgres-0

---
# Restore from snapshot
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-restore
  namespace: database
spec:
  storageClassName: fast-replicated
  dataSource:
    name: postgres-snapshot-20240115
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

---

## Observability

### Prometheus Stack

```yaml
# Prometheus Operator CRDs
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/bundle.yaml
---
# ServiceMonitor for application metrics
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: api-service
  namespace: production
  labels:
    app: api-service
spec:
  selector:
    matchLabels:
      app: api-service
  endpoints:
    - port: metrics
      interval: 30s
      path: /metrics

---
# PodMonitor for pod metrics
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: istio-mesh
  namespace: istio-system
spec:
  selector:
    matchLabels:
      istio: sidecar
  podMetricsEndpoints:
    - port: http-envoy-prom
      interval: 30s
      path: /stats/prometheus

---
# PrometheusRule for alerts
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: api-alerts
  namespace: monitoring
spec:
  groups:
    - name: api-service
      interval: 30s
      rules:
        - alert: HighErrorRate
          expr: |
            sum(rate(http_requests_total{job="api-service",code=~"5.."}[5m]))
            /
            sum(rate(http_requests_total{job="api-service"}[5m]))
            > 0.05
          for: 5m
          labels:
            severity: critical
            service: api-service
          annotations:
            summary: "High error rate detected"
            description: "Error rate is {{ $value | humanizePercentage }} for the last 5 minutes"

        - alert: HighLatency
          expr: |
            histogram_quantile(0.99,
              sum(rate(http_request_duration_seconds_bucket{job="api-service"}[5m])) by (le)
            ) > 1
          for: 10m
          labels:
            severity: warning
            service: api-service
          annotations:
            summary: "High latency detected"
            description: "P99 latency is {{ $value }}s"

---
# Prometheus instance
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: main
  namespace: monitoring
spec:
  replicas: 2
  version: v2.48.0
  serviceAccountName: prometheus
  serviceMonitorSelector:
    matchLabels:
      monitoring: enabled
  podMonitorSelector:
    matchLabels:
      monitoring: enabled
  resources:
    requests:
      memory: 2Gi
      cpu: 1000m
    limits:
      memory: 4Gi
      cpu: 2000m
  retention: 30d
  storage:
    volumeClaimTemplate:
      spec:
        storageClassName: fast-replicated
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 100Gi
  alerting:
    alertmanagers:
      - namespace: monitoring
        name: alertmanager-main
        port: web
```

### Distributed Tracing (Jaeger)

```yaml
# Jaeger Operator
kubectl create namespace observability
kubectl apply -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.51.0/jaeger-operator.yaml -n observability
---
# Jaeger instance with production config
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: jaeger-production
  namespace: observability
spec:
  strategy: production

  storage:
    type: elasticsearch
    options:
      es:
        server-urls: https://elasticsearch:9200
        index-prefix: jaeger
        tls:
          ca: /tls/ca.crt
    secretName: jaeger-elasticsearch-secret

  collector:
    replicas: 3
    resources:
      requests:
        cpu: 500m
        memory: 512Mi
      limits:
        cpu: 1000m
        memory: 1Gi
    maxReplicas: 5

  query:
    replicas: 2
    resources:
      requests:
        cpu: 200m
        memory: 256Mi
      limits:
        cpu: 500m
        memory: 512Mi

  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
      cert-manager.io/cluster-issuer: letsencrypt-prod
    hosts:
      - jaeger.example.com
    tls:
      - secretName: jaeger-tls
        hosts:
          - jaeger.example.com

---
# Configure applications to send traces
apiVersion: v1
kind: ConfigMap
metadata:
  name: jaeger-config
  namespace: production
data:
  JAEGER_AGENT_HOST: jaeger-production-agent.observability
  JAEGER_AGENT_PORT: "6831"
  JAEGER_SAMPLER_TYPE: probabilistic
  JAEGER_SAMPLER_PARAM: "0.1" # Sample 10% of traces
```

### Logging with Loki

```yaml
# Loki deployment
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: loki
  namespace: monitoring
spec:
  serviceName: loki
  replicas: 3
  selector:
    matchLabels:
      app: loki
  template:
    metadata:
      labels:
        app: loki
    spec:
      containers:
        - name: loki
          image: grafana/loki:2.9.3
          args:
            - -config.file=/etc/loki/loki.yaml
            - -target=all
          ports:
            - containerPort: 3100
              name: http
            - containerPort: 9095
              name: grpc
          volumeMounts:
            - name: config
              mountPath: /etc/loki
            - name: storage
              mountPath: /loki
          resources:
            requests:
              cpu: 500m
              memory: 1Gi
            limits:
              cpu: 2000m
              memory: 4Gi
      volumes:
        - name: config
          configMap:
            name: loki-config
  volumeClaimTemplates:
    - metadata:
        name: storage
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: fast-replicated
        resources:
          requests:
            storage: 50Gi

---
# Promtail DaemonSet for log collection
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: promtail
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: promtail
  template:
    metadata:
      labels:
        app: promtail
    spec:
      serviceAccountName: promtail
      containers:
        - name: promtail
          image: grafana/promtail:2.9.3
          args:
            - -config.file=/etc/promtail/promtail.yaml
          volumeMounts:
            - name: config
              mountPath: /etc/promtail
            - name: varlog
              mountPath: /var/log
              readOnly: true
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 200m
              memory: 256Mi
      volumes:
        - name: config
          configMap:
            name: promtail-config
        - name: varlog
          hostPath:
            path: /var/log
        - name: varlibdockercontainers
          hostPath:
            path: /var/lib/docker/containers
```

---

## Multi-Cluster Management

### Cluster Federation

```yaml
# KubeFed installation
kubectl create namespace kube-federation-system
helm install kubefed kubefed-charts/kubefed \
--namespace kube-federation-system
---
# Register clusters
kubefedctl join cluster1 \
--cluster-context cluster1 \
--host-cluster-context cluster1

kubefedctl join cluster2 \
--cluster-context cluster2 \
--host-cluster-context cluster1
---
# Federated deployment
apiVersion: types.kubefed.io/v1beta1
kind: FederatedDeployment
metadata:
  name: my-app
  namespace: production
spec:
  template:
    metadata:
      labels:
        app: my-app
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: my-app
      template:
        metadata:
          labels:
            app: my-app
        spec:
          containers:
            - name: app
              image: my-app:v1.0.0
  placement:
    clusters:
      - name: cluster1
      - name: cluster2
  overrides:
    - clusterName: cluster1
      clusterOverrides:
        - path: "/spec/replicas"
          value: 5
    - clusterName: cluster2
      clusterOverrides:
        - path: "/spec/replicas"
          value: 3

---
# Federated service
apiVersion: types.kubefed.io/v1beta1
kind: FederatedService
metadata:
  name: my-app
  namespace: production
spec:
  template:
    spec:
      selector:
        app: my-app
      ports:
        - port: 80
          targetPort: 8080
  placement:
    clusters:
      - name: cluster1
      - name: cluster2
```

### Multi-Cluster Ingress

```yaml
# Global load balancer with multi-cluster backend
apiVersion: networking.gke.io/v1
kind: MultiClusterIngress
metadata:
  name: global-ingress
  namespace: production
spec:
  template:
    spec:
      backend:
        serviceName: my-app
        servicePort: 80
      rules:
        - host: app.example.com
          http:
            paths:
              - path: /*
                backend:
                  serviceName: my-app
                  servicePort: 80

---
# Multi-cluster service
apiVersion: networking.gke.io/v1
kind: MultiClusterService
metadata:
  name: my-app
  namespace: production
spec:
  template:
    spec:
      selector:
        app: my-app
      ports:
        - port: 80
          targetPort: 8080
  clusters:
    - link: us-central1/cluster1
    - link: europe-west1/cluster2
```

---

## Platform Best Practices

### Resource Management

```yaml
# LimitRange for namespace defaults
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: production
spec:
  limits:
    - default:
        cpu: 500m
        memory: 512Mi
      defaultRequest:
        cpu: 100m
        memory: 128Mi
      max:
        cpu: 4000m
        memory: 8Gi
      min:
        cpu: 50m
        memory: 64Mi
      type: Container
    - max:
        cpu: 8000m
        memory: 16Gi
      min:
        cpu: 50m
        memory: 64Mi
      type: Pod

---
# PodDisruptionBudget
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
  namespace: production
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: api-service
  unhealthyPodEvictionPolicy: AlwaysAllow

---
# PriorityClass
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "High priority for critical services"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 100
globalDefault: true
description: "Low priority for batch jobs"
```

### Node Management

```yaml
# Node taints and tolerations
apiVersion: v1
kind: Node
metadata:
  name: node1
spec:
  taints:
    - key: workload-type
      value: database
      effect: NoSchedule

---
# Pod with toleration
apiVersion: v1
kind: Pod
metadata:
  name: postgres
spec:
  tolerations:
    - key: workload-type
      operator: Equal
      value: database
      effect: NoSchedule
  nodeSelector:
    workload-type: database
  containers:
    - name: postgres
      image: postgres:15

---
# Node affinity
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  template:
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: node-type
                    operator: In
                    values:
                      - compute-optimized
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              preference:
                matchExpressions:
                  - key: zone
                    operator: In
                    values:
                      - us-central1-a
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - api-service
              topologyKey: kubernetes.io/hostname
```

---

## Disaster Recovery

### Backup Strategy

```yaml
# Velero for cluster backups
velero install \
--provider gcp \
--plugins velero/velero-plugin-for-gcp:v1.9.0 \
--bucket velero-backups \
--secret-file ./credentials-velero
---
# Schedule regular backups
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-backup
  namespace: velero
spec:
  schedule: "0 2 * * *" # Daily at 2 AM
  template:
    includedNamespaces:
      - production
      - staging
    excludedResources:
      - events
      - events.events.k8s.io
    storageLocation: default
    volumeSnapshotLocations:
      - default
    ttl: 720h # 30 days

---
# Backup specific resources
apiVersion: velero.io/v1
kind: Backup
metadata:
  name: production-backup
  namespace: velero
spec:
  includedNamespaces:
    - production
  labelSelector:
    matchLabels:
      backup: enabled
  snapshotVolumes: true
  storageLocation: default
  ttl: 2160h # 90 days

---
# Restore from backup
apiVersion: velero.io/v1
kind: Restore
metadata:
  name: production-restore
  namespace: velero
spec:
  backupName: production-backup
  includedNamespaces:
    - production
  restorePVs: true
```

---

## Security Hardening

### Pod Security Policies (deprecated, use Pod Security Standards)

```yaml
# Restricted Pod Security Standard
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted

---
# Security scanning with Falco
apiVersion: v1
kind: ConfigMap
metadata:
  name: falco-rules
  namespace: falco
data:
  custom-rules.yaml: |
    - rule: Unauthorized Process in Container
      desc: Detect processes running that are not in allowed list
      condition: >
        spawned_process and
        container and
        not proc.name in (allowed_processes)
      output: >
        Unauthorized process started in container
        (user=%user.name command=%proc.cmdline container=%container.name)
      priority: WARNING

---
# Network scanning with Trivy Operator
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/trivy-operator/v0.18.0/deploy/static/trivy-operator.yaml
```

---

## Quick Reference Checklist

**Cluster Setup:**

- [ ] Infrastructure as Code (Terraform/Pulumi)
- [ ] Multi-zone/region for HA
- [ ] Workload identity configured
- [ ] Private cluster endpoints
- [ ] Network policies enabled
- [ ] Auto-scaling configured

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

## Summary

**Key Principles:**

- Infrastructure as Code
- Security by default
- Observe everything
- Design for failure
- Automate operations
- Multi-tenancy support

**Remember**: Platform engineering is about building reliable, secure, and observable infrastructure that enables developers to ship software quickly and safely.
