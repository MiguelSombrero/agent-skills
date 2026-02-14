---
name: devops-engineer
description: Designs and implements CI/CD pipelines, GitOps workflows, and deployment automation with expertise in GitHub Actions, Kubernetes, Helm, ArgoCD, and containerization. Use when automating builds, releases, deployments, or setting up GitOps workflows. Focuses on pipeline optimization, container orchestration, and declarative infrastructure.
---

# DevOps Engineer

Automates software delivery through robust CI/CD pipelines, GitOps practices, and container orchestration. Combines development and operations expertise to enable fast, reliable, and repeatable deployments.

## When to Use This Skill

Use this skill when the user:

- Wants to **set up CI/CD pipelines** or **automate deployments**
- Needs help with **GitHub Actions** workflows or optimization
- Asks about **Kubernetes** deployments, manifests, or configurations
- Wants to create or modify **Helm charts**
- Needs guidance on **GitOps** with **ArgoCD**
- Wants to **containerize applications** or optimize Docker builds
- Asks about **deployment strategies** (rolling, blue-green, canary)
- Needs help with **secrets management** in pipelines or Kubernetes
- Wants to set up **local Kubernetes** development with k3d
- Asks about **container registries**, image tagging, or versioning
- Needs to **automate testing** in CI/CD pipelines
- Wants to implement **release workflows** or versioning strategies
- Asks "how do I deploy..." or "how do I automate..."

## Scope

**In Scope** (DevOps/CI/CD):

- CI/CD pipeline design and implementation
- GitHub Actions workflows
- Container building and optimization
- Kubernetes deployments and configurations
- Helm charts and templating
- ArgoCD and GitOps workflows
- Deployment strategies and rollbacks
- Secrets management (GitHub Secrets, Kubernetes Secrets, Sealed Secrets)
- Container registry management
- Build optimization and caching
- Pipeline security and best practices

**Out of Scope** (Cloud Platform Engineering):

- Cloud provider infrastructure (AWS, GCP, Azure)
- Service mesh (Istio, Linkerd)
- Operators and CRDs
- SLA/SLO monitoring and reliability engineering
- Multi-cluster management
- Advanced observability and tracing
- Network policies and CNI
- Storage classes and PV management

**Note**: This skill focuses on application delivery automation. For infrastructure and platform concerns, use cloud platform engineering skills.

---

## Core DevOps Principles

### The Three Ways

#### 1. Flow (Left to Right)

**Optimize the flow from development to production.**

```yaml
# Fast, automated pipeline
Development → Build → Test → Stage → Production
↓          ↓       ↓       ↓         ↓
< 5min    < 10min  < 15min < 5min   < 10min
```

#### 2. Feedback (Right to Left)

**Fast feedback from production to development.**

```yaml
# Monitoring and alerts flow back
Production → Monitoring → Alerts → Development
↓           ↓          ↓           ↓
Metrics    Dashboards  Incidents   Fixes
```

#### 3. Continuous Learning

**Culture of experimentation and learning.**

### GitOps Principles

#### 1. Declarative Configuration

**Entire system described declaratively.**

```yaml
# Git repository contains desired state
app-repo/
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── helm/
├── Chart.yaml
├── values.yaml
└── templates/
```

#### 2. Git as Single Source of Truth

**Git is the single source of truth for system state.**

```bash
# Any change requires Git commit
git commit -m "Update app to v2.0.0"
git push origin main

# ArgoCD automatically syncs
```

#### 3. Automated Deployment

**Changes are automatically applied to system.**

```yaml
# ArgoCD Application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  syncPolicy:
    automated:
      prune: true # Remove resources not in Git
      selfHeal: true # Sync if manual changes detected
```

#### 4. Continuous Reconciliation

**Agents continuously reconcile desired and actual state.**

---

## GitHub Actions

### Workflow Basics

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: "20"
  REGISTRY: ghcr.io

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

### Multi-Job Workflow with Dependencies

```yaml
# .github/workflows/build-deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  # Job 1: Build and test
  build:
    runs-on: ubuntu-latest

    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}

    steps:
      - uses: actions/checkout@v4

      - name: Docker meta
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ github.repository }}
          tags: |
            type=sha,prefix={{branch}}-
            type=ref,event=branch
            type=semver,pattern={{version}}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # Job 2: Update manifests (depends on build)
  update-manifests:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          repository: org/gitops-repo
          token: ${{ secrets.GITOPS_PAT }}

      - name: Update image tag
        run: |
          sed -i "s|image: .*|image: ${{ needs.build.outputs.image-tag }}|" \
            k8s/deployment.yaml

      - name: Commit changes
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add k8s/deployment.yaml
          git commit -m "Update image to ${{ needs.build.outputs.image-tag }}"
          git push

  # Job 3: Deploy to staging (depends on build)
  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging

    steps:
      - name: Deploy to staging
        run: |
          kubectl set image deployment/my-app \
            app=${{ needs.build.outputs.image-tag }} \
            --namespace=staging
```

### Reusable Workflows

```yaml
# .github/workflows/reusable-build.yml
name: Reusable Build

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      environment:
        required: true
        type: string
    outputs:
      image-tag:
        description: "Built Docker image tag"
        value: ${{ jobs.build.outputs.image-tag }}
    secrets:
      registry-token:
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}

      # ... build steps

# .github/workflows/main.yml
name: Main Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
    with:
      node-version: '20'
      environment: production
    secrets:
      registry-token: ${{ secrets.REGISTRY_TOKEN }}

  deploy:
    needs: build
    # ... deploy job
```

### Matrix Builds

```yaml
# Test multiple versions
jobs:
  test:
    runs-on: ${{ matrix.os }}

    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
        exclude:
          - os: windows-latest
            node-version: 18

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - run: npm ci
      - run: npm test
```

### Conditional Execution

```yaml
jobs:
  deploy:
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest

    steps:
      - name: Deploy to production
        if: startsWith(github.ref, 'refs/tags/')
        run: ./deploy-prod.sh

      - name: Deploy to staging
        if: github.ref == 'refs/heads/develop'
        run: ./deploy-staging.sh
```

### Caching for Speed

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      # Cache npm dependencies
      - name: Cache node modules
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - run: npm ci

      # Cache Docker layers
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### Security Scanning in Pipeline

```yaml
jobs:
  security:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      # Dependency scanning
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

      # Container scanning
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ github.repository }}:latest
          format: "sarif"
          output: "trivy-results.sarif"

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: "trivy-results.sarif"

      # SAST scanning
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: javascript, typescript

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
```

### Environments and Approvals

```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com

    steps:
      - name: Deploy
        run: ./deploy.sh
        env:
          API_KEY: ${{ secrets.PROD_API_KEY }}
          DATABASE_URL: ${{ secrets.PROD_DATABASE_URL }}

# Configure environment protection rules in GitHub:
# Settings → Environments → production
# - Required reviewers
# - Wait timer
# - Deployment branches
```

---

## Docker & Containerization

### Multi-Stage Builds

```dockerfile
# Dockerfile - Optimized multi-stage build

# Stage 1: Dependencies
FROM node:20-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 3: Production
FROM node:20-alpine AS production
WORKDIR /app

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy only necessary files
COPY --from=dependencies /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./

# Security: Run as non-root
USER nodejs

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
  CMD node healthcheck.js

CMD ["node", "dist/index.js"]
```

### Build Optimization

```dockerfile
# .dockerignore - Reduce build context
node_modules
npm-debug.log
dist
.git
.github
.env
.env.local
*.md
coverage
.vscode
.idea
```

```dockerfile
# Use specific versions, not latest
FROM node:20.11.0-alpine3.19

# Combine RUN commands to reduce layers
RUN apk add --no-cache \
    python3 \
    make \
    g++ \
 && rm -rf /var/cache/apk/*

# Leverage layer caching - copy package files first
COPY package*.json ./
RUN npm ci

# Copy source code last (changes most frequently)
COPY . .
```

### Docker Compose for Local Development

```yaml
# docker-compose.yml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: development
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres-data:
```

---

## Kubernetes

### Deployment Manifest

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
  labels:
    app: my-app
    version: v1.0.0
spec:
  replicas: 3

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1 # Max pods above desired count
      maxUnavailable: 0 # Always maintain availability

  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app
        version: v1.0.0

    spec:
      # Security context
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001

      # Init containers
      initContainers:
        - name: migration
          image: my-app:v1.0.0
          command: ["npm", "run", "migrate"]
          envFrom:
            - secretRef:
                name: db-credentials

      containers:
        - name: app
          image: my-app:v1.0.0
          imagePullPolicy: IfNotPresent

          ports:
            - name: http
              containerPort: 3000
              protocol: TCP

          # Environment variables
          env:
            - name: NODE_ENV
              value: production
            - name: PORT
              value: "3000"

          # From ConfigMap
          envFrom:
            - configMapRef:
                name: app-config
            - secretRef:
                name: app-secrets

          # Resource limits
          resources:
            requests:
              memory: "256Mi"
              cpu: "100m"
            limits:
              memory: "512Mi"
              cpu: "500m"

          # Probes
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3

          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 10
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3

          # Security
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

          # Volume mounts
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: cache
              mountPath: /app/.cache

      volumes:
        - name: tmp
          emptyDir: {}
        - name: cache
          emptyDir: {}

      # Node selection
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: my-app
                topologyKey: kubernetes.io/hostname
```

### Service & Ingress

```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
  namespace: production
  labels:
    app: my-app
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
  sessionAffinity: ClientIP # Sticky sessions

---
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - app.example.com
      secretName: app-tls
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app
                port:
                  name: http
```

### ConfigMap & Secrets

```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  LOG_LEVEL: info
  CACHE_TTL: "3600"
  API_TIMEOUT: "30000"

---
# k8s/secret.yaml (encrypted with Sealed Secrets)
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: production
type: Opaque
stringData:
  DATABASE_URL: postgresql://user:password@db:5432/myapp
  API_KEY: secret-key-here
  JWT_SECRET: jwt-secret-here
```

### HorizontalPodAutoscaler

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app

  minReplicas: 3
  maxReplicas: 10

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70

    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80

  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300 # Wait 5 min before scaling down
      policies:
        - type: Percent
          value: 50
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0 # Scale up immediately
      policies:
        - type: Percent
          value: 100
          periodSeconds: 30
```

### Namespace with ResourceQuota

```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    env: production

---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    persistentvolumeclaims: "10"
    pods: "50"
```

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

## k3d (Local Kubernetes)

### Create Local Cluster

```bash
# Install k3d
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

# Create cluster with registry
k3d cluster create dev \
  --agents 2 \
  --registry-create registry.localhost:5000 \
  --port 8080:80@loadbalancer \
  --port 8443:443@loadbalancer \
  --api-port 6443 \
  --volume $(pwd)/k3d-volumes:/volumes

# List clusters
k3d cluster list

# Stop cluster
k3d cluster stop dev

# Start cluster
k3d cluster start dev

# Delete cluster
k3d cluster delete dev
```

### Configuration File

```yaml
# k3d-config.yaml
apiVersion: k3d.io/v1alpha5
kind: Simple
metadata:
  name: dev
servers: 1
agents: 2

image: rancher/k3s:v1.28.5-k3s1

ports:
  - port: 8080:80
    nodeFilters:
      - loadbalancer
  - port: 8443:443
    nodeFilters:
      - loadbalancer

registries:
  create:
    name: registry.localhost
    host: "0.0.0.0"
    hostPort: "5000"

options:
  k3d:
    wait: true
    timeout: "60s"
  k3s:
    extraArgs:
      - arg: --disable=traefik
        nodeFilters:
          - server:*
  kubeconfig:
    updateDefaultKubeconfig: true
    switchCurrentContext: true

# Create cluster from config
k3d cluster create --config k3d-config.yaml
```

### Local Development Workflow

```bash
# Build and push to local registry
docker build -t registry.localhost:5000/my-app:dev .
docker push registry.localhost:5000/my-app:dev

# Deploy to k3d
kubectl apply -f k8s/

# Or with Helm
helm upgrade --install my-app ./helm/my-app \
  --set image.repository=registry.localhost:5000/my-app \
  --set image.tag=dev

# Port forward for local access
kubectl port-forward svc/my-app 3000:80

# View logs
kubectl logs -f deployment/my-app

# Shell into pod
kubectl exec -it deployment/my-app -- sh
```

---

## Secrets Management

### GitHub Secrets

```yaml
# Access secrets in workflow
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          KUBE_CONFIG: ${{ secrets.KUBE_CONFIG }}
          REGISTRY_TOKEN: ${{ secrets.REGISTRY_TOKEN }}
        run: |
          echo "$KUBE_CONFIG" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
          kubectl apply -f k8s/
```

### Sealed Secrets

```bash
# Install sealed-secrets controller
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets sealed-secrets/sealed-secrets \
  --namespace kube-system

# Install kubeseal CLI
KUBESEAL_VERSION=0.24.0
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v${KUBESEAL_VERSION}/kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz
tar xfz kubeseal-${KUBESEAL_VERSION}-linux-amd64.tar.gz
sudo install -m 755 kubeseal /usr/local/bin/kubeseal

# Create secret
kubectl create secret generic app-secrets \
  --from-literal=DATABASE_URL=postgresql://... \
  --dry-run=client -o yaml > secret.yaml

# Seal the secret
kubeseal --format yaml < secret.yaml > sealed-secret.yaml

# Commit sealed secret to Git
git add sealed-secret.yaml
git commit -m "Add sealed secrets"
git push

# Controller automatically unseals in cluster
kubectl apply -f sealed-secret.yaml
```

### External Secrets Operator

```yaml
# Install External Secrets Operator
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets \
  external-secrets/external-secrets \
  --namespace external-secrets-system \
  --create-namespace

# SecretStore - connects to secret provider
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault
  namespace: production
spec:
  provider:
    vault:
      server: "https://vault.example.com"
      path: "secret"
      version: "v2"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "my-app"

---
# ExternalSecret - fetches and syncs secrets
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: app-secrets
  namespace: production
spec:
  refreshInterval: 1h

  secretStoreRef:
    name: vault
    kind: SecretStore

  target:
    name: app-secrets
    creationPolicy: Owner

  data:
    - secretKey: DATABASE_URL
      remoteRef:
        key: my-app/database
        property: url

    - secretKey: API_KEY
      remoteRef:
        key: my-app/api
        property: key
```

---

## Deployment Strategies

### Rolling Update (Default)

```yaml
# Zero-downtime rolling update
apiVersion: apps/v1
kind: Deployment
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1 # Can have 5 pods during update
      maxUnavailable: 0 # Always keep 4 running
```

### Blue-Green Deployment

```bash
# Blue environment (current)
kubectl apply -f deployment-blue.yaml
kubectl apply -f service.yaml  # Points to blue

# Deploy green environment (new version)
kubectl apply -f deployment-green.yaml

# Test green environment
kubectl port-forward deployment/my-app-green 8080:3000

# Switch traffic to green
kubectl patch service my-app -p '{"spec":{"selector":{"version":"green"}}}'

# If issues, rollback to blue
kubectl patch service my-app -p '{"spec":{"selector":{"version":"blue"}}}'

# Clean up blue when confident
kubectl delete deployment my-app-blue
```

### Canary Deployment

```yaml
# Stable deployment - 90% of traffic
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-stable
spec:
  replicas: 9
  template:
    metadata:
      labels:
        app: my-app
        version: stable

---
# Canary deployment - 10% of traffic
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-canary
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: my-app
        version: canary
        track: canary

---
# Service sends traffic to both
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app # Matches both stable and canary
```

### Progressive Delivery with Argo Rollouts

```yaml
# Install Argo Rollouts
kubectl create namespace argo-rollouts
kubectl apply -n argo-rollouts -f https://github.com/argoproj/argo-rollouts/releases/latest/download/install.yaml

# Rollout resource
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 5

  strategy:
    canary:
      steps:
        - setWeight: 20
        - pause: {duration: 5m}
        - setWeight: 40
        - pause: {duration: 5m}
        - setWeight: 60
        - pause: {duration: 5m}
        - setWeight: 80
        - pause: {duration: 5m}

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
          image: my-app:v2.0.0

# Promote rollout
kubectl argo rollouts promote my-app

# Abort rollout
kubectl argo rollouts abort my-app
```

---

## Monitoring CI/CD

### Pipeline Metrics

```yaml
# GitHub Actions - Track build times
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        id: build
        run: |
          start=$(date +%s)
          npm run build
          end=$(date +%s)
          echo "duration=$((end-start))" >> $GITHUB_OUTPUT

      - name: Report metrics
        run: |
          curl -X POST https://metrics.example.com/api/metrics \
            -H "Content-Type: application/json" \
            -d '{
              "metric": "build_duration",
              "value": ${{ steps.build.outputs.duration }},
              "pipeline": "${{ github.workflow }}",
              "branch": "${{ github.ref_name }}"
            }'
```

### Deployment Notifications

```yaml
# Slack notification on deployment
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        id: deploy
        run: ./deploy.sh

      - name: Notify Slack
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Deployment ${{ steps.deploy.outcome }}",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Deployment to Production*\nStatus: ${{ steps.deploy.outcome }}\nCommit: ${{ github.sha }}\nAuthor: ${{ github.actor }}"
                  }
                }
              ]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Best Practices

### Pipeline Design

```yaml
# ✅ Good - Fast feedback, fail fast
jobs:
  # Fast checks first
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm run lint  # ~30 seconds

  # Parallel test matrix
  test:
    needs: lint
    strategy:
      matrix:
        node: [18, 20, 22]
    runs-on: ubuntu-latest
    steps:
      - run: npm test  # ~2 minutes

  # Expensive builds only after tests pass
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - run: docker build .  # ~5 minutes

# ❌ Bad - Slow feedback
jobs:
  everything:
    steps:
      - run: docker build .  # 5 min, fails if lint fails later
      - run: npm run lint    # Should fail fast
      - run: npm test
```

### Container Security

```dockerfile
# ✅ Good practices
# 1. Use specific versions
FROM node:20.11.0-alpine3.19

# 2. Run as non-root
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
USER nodejs

# 3. Read-only root filesystem
# Combined with emptyDir volumes for writable paths

# 4. Drop capabilities
securityContext:
  capabilities:
    drop:
      - ALL

# 5. Scan images
- name: Scan image
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: my-app:latest
    severity: 'CRITICAL,HIGH'
```

### Resource Management

```yaml
# ✅ Always set resource requests and limits
resources:
  requests:
    memory: "256Mi" # Minimum guaranteed
    cpu: "100m" # 0.1 CPU
  limits:
    memory: "512Mi" # Maximum allowed
    cpu: "500m" # 0.5 CPU


# Set QoS class
# Guaranteed: requests == limits
# Burstable: requests < limits
# BestEffort: no requests/limits (avoid)
```

### Health Checks

```yaml
# ✅ Proper health checks
livenessProbe:
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 30 # Wait for app to start
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3 # Restart after 3 failures

readinessProbe:
  httpGet:
    path: /ready
    port: http
  initialDelaySeconds: 10
  periodSeconds: 5
  successThreshold: 1
  failureThreshold: 3 # Remove from load balancer after 3 failures
```

---

## Anti-Patterns

### Hard-coded Values

```yaml
# ❌ Bad - Hard-coded values
image: my-app:v1.0.0
replicas: 3

# ✅ Good - Parameterized
image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
replicas: {{ .Values.replicaCount }}
```

### Running as Root

```dockerfile
# ❌ Bad - Runs as root
FROM node:20
COPY . .
CMD ["node", "server.js"]

# ✅ Good - Non-root user
FROM node:20-alpine
RUN addgroup -g 1001 nodejs && adduser -S nodejs -u 1001
USER nodejs
COPY --chown=nodejs:nodejs . .
CMD ["node", "server.js"]
```

### Missing Resource Limits

```yaml
# ❌ Bad - No limits (can consume entire node)
containers:
  - name: app
    image: my-app

# ✅ Good - Set limits
containers:
  - name: app
    image: my-app
    resources:
      requests:
        memory: 256Mi
        cpu: 100m
      limits:
        memory: 512Mi
        cpu: 500m
```

### Secrets in Git

```yaml
# ❌ Bad - Plain secrets in Git
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
stringData:
  DATABASE_URL: postgresql://user:password@host/db

# ✅ Good - Use Sealed Secrets or External Secrets
# Encrypted SealedSecret in Git
# Or reference external secret manager
```

### No Rollback Strategy

```yaml
# ❌ Bad - No way to rollback
kubectl apply -f deployment.yaml

# ✅ Good - Use Helm or track versions
helm upgrade my-app ./chart --version 2.0.0

# Easy rollback
helm rollback my-app 1
```

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

## Summary

**Key Principles:**

- Automate everything
- Git as single source of truth
- Fail fast with quick feedback
- Security throughout pipeline
- Infrastructure as code
- Declarative over imperative
- Monitor and measure

**Remember**: Good DevOps enables developers to ship code confidently and quickly while maintaining stability and security.
