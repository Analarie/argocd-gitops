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

## Monitoramento

```bash
# Ver status das applications
kubectl get applications -n argocd

# Ver pods do ambiente dev
kubectl get pods -n mamaloo-dev

# Ver pods do ambiente prod
kubectl get pods -n mamaloo-prod

# Logs do backend
kubectl logs -n mamaloo-dev -l app.kubernetes.io/component=backend --tail=100

# Logs do frontend
kubectl logs -n mamaloo-dev -l app.kubernetes.io/component=frontend --tail=100
```

## CI/CD

O pipeline no GitHub Actions:
1. Builda imagens Docker (backend/frontend)
2. Faz push para GHCR com tag `<env>-<sha>`
3. Atualiza este repositório com novas tags
4. ArgoCD detecta mudanças e sincroniza automaticamente

## Troubleshooting

```bash
# Ver eventos do namespace
kubectl get events -n mamaloo-dev --sort-by='.lastTimestamp'

# Descrever pod com problema
kubectl describe pod <pod-name> -n mamaloo-dev

# Forçar sync no ArgoCD
argocd app sync mamaloo-dev
argocd app sync mamaloo-prod
```
