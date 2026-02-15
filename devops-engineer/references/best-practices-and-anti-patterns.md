Reference for devops-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.
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

