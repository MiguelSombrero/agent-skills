Reference for devops-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.
---

## Helm

### Chart Structure

```
my-app/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
├── values-prod.yaml
├── charts/              # Dependency charts
├── templates/
│   ├── _helpers.tpl     # Template helpers
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   ├── serviceaccount.yaml
│   └── NOTES.txt
└── .helmignore
```

### Chart.yaml

```yaml
# Chart.yaml
apiVersion: v2
name: my-app
description: A Helm chart for my application
type: application
version: 1.0.0 # Chart version
appVersion: "2.0.0" # Application version

keywords:
  - web
  - nodejs

maintainers:
  - name: DevOps Team
    email: devops@example.com

dependencies:
  - name: postgresql
    version: 12.x.x
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled

  - name: redis
    version: 17.x.x
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

### values.yaml

```yaml
# values.yaml - Default values
replicaCount: 3

image:
  repository: ghcr.io/org/my-app
  pullPolicy: IfNotPresent
  tag: "" # Defaults to chart appVersion

imagePullSecrets: []

nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  annotations: {}
  name: ""

podAnnotations: {}

podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1001
  fsGroup: 1001

securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL

service:
  type: ClusterIP
  port: 80
  targetPort: http

ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: app.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: app-tls
      hosts:
        - app.example.com

resources:
  requests:
    memory: 256Mi
    cpu: 100m
  limits:
    memory: 512Mi
    cpu: 500m

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchLabels:
              app: my-app
          topologyKey: kubernetes.io/hostname

env:
  NODE_ENV: production
  LOG_LEVEL: info

envSecrets: {}

configMap:
  data:
    CACHE_TTL: "3600"

# Dependencies
postgresql:
  enabled: true
  auth:
    username: myapp
    database: myapp

redis:
  enabled: true
  architecture: standalone
```

### templates/deployment.yaml

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}

  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}

  template:
    metadata:
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}

    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}

      serviceAccountName: {{ include "my-app.serviceAccountName" . }}

      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}

      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}

          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}

          ports:
            - name: http
              containerPort: 3000
              protocol: TCP

          env:
            {{- range $key, $value := .Values.env }}
            - name: {{ $key }}
              value: {{ $value | quote }}
            {{- end }}
            {{- range $key, $value := .Values.envSecrets }}
            - name: {{ $key }}
              valueFrom:
                secretKeyRef:
                  name: {{ include "my-app.fullname" $ }}-secrets
                  key: {{ $key }}
            {{- end }}

          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10

          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 10
            periodSeconds: 5

          resources:
            {{- toYaml .Values.resources | nindent 12 }}

      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}

      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}

      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

### templates/\_helpers.tpl

```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "my-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "my-app.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "my-app.labels" -}}
helm.sh/chart: {{ include "my-app.chart" . }}
{{ include "my-app.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "my-app.selectorLabels" -}}
app.kubernetes.io/name: {{ include "my-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

### Helm Commands

```bash
# Create new chart
helm create my-app

# Validate chart
helm lint my-app

# Dry run to see rendered templates
helm install my-app ./my-app --dry-run --debug

# Install chart
helm install my-app ./my-app \
  --namespace production \
  --create-namespace \
  --values values-prod.yaml

# Upgrade release
helm upgrade my-app ./my-app \
  --namespace production \
  --values values-prod.yaml \
  --set image.tag=v2.0.0

# Rollback
helm rollback my-app 1 --namespace production

# View history
helm history my-app --namespace production

# Uninstall
helm uninstall my-app --namespace production

# Package chart
helm package my-app

# Update dependencies
helm dependency update my-app
```

---

## ArgoCD (GitOps)

### Application CRD

```yaml
# argocd/application.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app-production
  namespace: argocd

  # Finalizer to ensure cascading delete
  finalizers:
    - resources-finalizer.argocd.argoproj.io

spec:
  # Project (optional, defaults to 'default')
  project: production

  # Source repository
  source:
    repoURL: https://github.com/org/my-app.git
    targetRevision: main
    path: helm/my-app

    # Helm-specific configuration
    helm:
      releaseName: my-app

      # Value files
      valueFiles:
        - values-prod.yaml

      # Override values
      values: |
        replicaCount: 5
        image:
          tag: v2.0.0

      # Parameters (alternative to values)
      parameters:
        - name: image.tag
          value: v2.0.0

  # Destination cluster and namespace
  destination:
    server: https://kubernetes.default.svc
    namespace: production

  # Sync policy
  syncPolicy:
    automated:
      prune: true # Delete resources not in Git
      selfHeal: true # Sync if manual changes detected
      allowEmpty: false

    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
      - PruneLast=true

    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

  # Ignore differences
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas # Ignore if HPA manages replicas
```

### AppProject

```yaml
# argocd/project.yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: Production applications

  # Source repositories
  sourceRepos:
    - https://github.com/org/*
    - https://charts.bitnami.com/bitnami

  # Destination clusters/namespaces
  destinations:
    - server: https://kubernetes.default.svc
      namespace: production
    - server: https://kubernetes.default.svc
      namespace: production-*

  # Allowed resource kinds
  clusterResourceWhitelist:
    - group: ""
      kind: Namespace

  namespaceResourceWhitelist:
    - group: "*"
      kind: "*"

  # Deny certain resource kinds
  namespaceResourceBlacklist:
    - group: ""
      kind: ResourceQuota
    - group: ""
      kind: LimitRange

  # Roles for RBAC
  roles:
    - name: developer
      description: Developer role
      policies:
        - p, proj:production:developer, applications, get, production/*, allow
        - p, proj:production:developer, applications, sync, production/*, allow
      groups:
        - developers
```

### Multi-Environment with ApplicationSet

```yaml
# argocd/applicationset.yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: my-app
  namespace: argocd
spec:
  generators:
    # Git directory generator - one app per directory
    - git:
        repoURL: https://github.com/org/gitops-repo.git
        revision: main
        directories:
          - path: environments/*

  template:
    metadata:
      name: "my-app-{{path.basename}}"

    spec:
      project: default

      source:
        repoURL: https://github.com/org/gitops-repo.git
        targetRevision: main
        path: "{{path}}"

        helm:
          valueFiles:
            - values.yaml
            - values-{{path.basename}}.yaml

      destination:
        server: https://kubernetes.default.svc
        namespace: "{{path.basename}}"

      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

### GitOps Workflow

```bash
# Repository structure
gitops-repo/
├── environments/
│   ├── dev/
│   │   ├── values.yaml
│   │   └── values-dev.yaml
│   ├── staging/
│   │   ├── values.yaml
│   │   └── values-staging.yaml
│   └── production/
│       ├── values.yaml
│       └── values-prod.yaml
└── base/
    └── my-app/
        ├── Chart.yaml
        └── templates/

# Deployment process:
# 1. Developer merges to main
# 2. CI builds and pushes image with tag
# 3. CI updates gitops-repo with new image tag
# 4. ArgoCD detects change and syncs

# Update image tag in GitOps repo
git clone https://github.com/org/gitops-repo.git
cd gitops-repo

# Update production values
yq eval '.image.tag = "v2.0.0"' -i environments/production/values-prod.yaml

git add environments/production/values-prod.yaml
git commit -m "Update production to v2.0.0"
git push origin main

# ArgoCD automatically syncs within 3 minutes
# Or trigger manually:
argocd app sync my-app-production
```

### Sync Waves for Ordered Deployment

```yaml
# Database migration job - runs first
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migration
  annotations:
    argocd.argoproj.io/sync-wave: "1"
spec:
  template:
    spec:
      containers:
        - name: migration
          image: my-app:v2.0.0
          command: ["npm", "run", "migrate"]
      restartPolicy: Never

---
# Application deployment - runs second
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  annotations:
    argocd.argoproj.io/sync-wave: "2"
spec:
  # ... deployment spec
```

---

