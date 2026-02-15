Reference for devops-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.
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

