# AI Task Platform — Infrastructure

Kubernetes manifests and ArgoCD configuration for the AI Task Platform.

## Repository Structure

```
k8s/
├── namespace.yaml        # Project namespace
├── configmap.yaml        # Non-sensitive config (Redis host, port)
├── secret.yaml           # Placeholders only — create secrets manually
├── ingress.yaml          # Routes / to frontend, /api to backend
├── frontend/             # Deployment + Service
├── backend/              # Deployment + Service
└── worker/               # Deployment (2 replicas, no service needed)
argocd/
└── application.yaml      # ArgoCD Application — auto-sync enabled
```

## Note
MongoDB and Redis are cloud-hosted (MongoDB Atlas + Upstash Redis).
No in-cluster database deployments needed.

## Setup

### Create secrets manually
```bash
kubectl create secret generic app-secrets \
  --from-literal=MONGO_URI="your_mongo_uri" \
  --from-literal=JWT_SECRET="your_jwt_secret" \
  --from-literal=REDIS_PASSWORD="your_redis_password" \
  -n ai-task-platform
```

### Apply manifests
```bash
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/frontend/
kubectl apply -f k8s/backend/
kubectl apply -f k8s/worker/
kubectl apply -f k8s/ingress.yaml
```

### Install ArgoCD
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl apply -f argocd/application.yaml
```

### Access ArgoCD Dashboard
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open `https://localhost:8080` — username: `admin`

## ArgoCD Auto-sync
ArgoCD watches this repo and automatically deploys changes to the Kubernetes cluster.