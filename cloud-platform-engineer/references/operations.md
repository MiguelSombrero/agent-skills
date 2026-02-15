Reference for cloud-platform-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.

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

