Reference for cloud-platform-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.

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

