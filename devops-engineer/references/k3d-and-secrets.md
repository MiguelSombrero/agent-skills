Reference for devops-engineer skill. See [SKILL.md](../SKILL.md) for core instructions.
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

