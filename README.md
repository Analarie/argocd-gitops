# ArgoCD GitOps - Portal Mamaloo

Repositório GitOps para gerenciar deployments da aplicação Portal Mamaloo usando ArgoCD.

## Estrutura

```
argocd-gitops/
├── apps/
│   ├── mamaloo-dev.yaml          # Application ArgoCD para dev
│   └── mamaloo-prod.yaml         # Application ArgoCD para prod
├── infrastructure/
│   └── cloudnative-pg/
│       ├── operator.yaml         # CloudNative PG Operator
│       ├── cluster-dev.yaml      # PostgreSQL Cluster dev
│       └── cluster-prod.yaml     # PostgreSQL Cluster prod
└── README.md
```

## Pré-requisitos

- Kubernetes cluster (local ou cloud)
- ArgoCD instalado no cluster
- Helm 3.x

## Instalação do ArgoCD

```bash
# Criar namespace
kubectl create namespace argocd

# Instalar ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Obter senha inicial
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port-forward para acessar UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

## Deploy das Applications

```bash
# Aplicar infraestrutura (operador PostgreSQL)
kubectl apply -f infrastructure/cloudnative-pg/operator.yaml

# Aplicar Applications ArgoCD
kubectl apply -f apps/mamaloo-dev.yaml
kubectl apply -f apps/mamaloo-prod.yaml
```

## Ambientes

### Dev
- **Namespace**: `mamaloo-dev`
- **URL**: Configurado via Ingress
- **Database**: PostgreSQL gerenciado pelo CloudNative PG

### Prod
- **Namespace**: `mamaloo-prod`
- **URL**: Configurado via Ingress
- **Database**: PostgreSQL com HA e backup



## 🎯 Environments

### Dev (desenvolvimento)
- Branch: `desenvolvimento`
- Namespace: `mamaloo-dev`
- Sync automático: **SIM** (prune + selfHeal)
- Replica padrão: 1

### Prod (produção)
- Branch: `main`
- Namespace: `mamaloo-prod`
- Sync automático: SIM (apenas selfHeal, prune desativado)
- Replicas padrão: 3
- Manual approval: Desativado (automático)

## 🔄 Sync Waves (Ordem de Deploy)

1. **Wave -5**: CloudNative PG Operator (infra)
2. **Wave 5**: Database migrations job (pré-deploy)
3. **Wave 10**: Backend + Frontend (aplicações)

## 📦 O Que é Deployado

### Backend
- FastAPI application
- 1 replica (dev) / 3 replicas (prod)
- Health checks automáticas
- CPU: 100m-500m, Memory: 128Mi-512Mi

### Frontend
- Vue.js + Vite
- 1 replica (dev) / 3 replicas (prod)
- VITE_API_URL configurado automaticamente

### Database
- PostgreSQL via CloudNative PG Operator
- StatefulSet com persistent storage
- 2Gi (dev) / 20Gi (prod)
- Backups automáticos (via operator)

## 🚀 Como Usar

### 1. Instalar ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Bootstrap com App-of-Apps

```bash
kubectl apply -f bootstrap/app-of-apps.yaml
```

Isso vai criar:
- CloudNative PG Operator (wave -5)
- Portal Mamaloo Dev (wave 10)
- Portal Mamaloo Prod (wave 10)

### 3. Verificar Status

```bash
# Listar applications
argocd app list

# Ver status de uma application
argocd app get portal-mamaloo-dev
argocd app get portal-mamaloo-prod

# Watch sync
argocd app wait portal-mamaloo-dev --sync
```

