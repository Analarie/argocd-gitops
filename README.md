# 🚀 ArgoCD GitOps - Portal Mamaloo

Repositório de configuração GitOps para o Portal Mamaloo usando ArgoCD.

## 📋 Estrutura

```
.
├── argocd/
│   └── apps/
│       ├── dev/
│       │   └── portal-mamaloo.yaml        # Application dev
│       └── prod/
│           └── portal-mamaloo.yaml        # Application prod
├── bootstrap/
│   └── app-of-apps.yaml                   # Root application
├── infrastructure/
│   └── database-operator/
│       └── cloudnative-pg.yaml            # Database operator
└── kustomize/                              # Kustomization files (opcional)
```

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

### 4. Sincronizar Manualmente (se necessário)

```bash
argocd app sync portal-mamaloo-dev
```

## 🔐 Secrets

Os secrets do banco são gerenciados pelo Helm Chart em `Portal-Mamaloo-desenvolvimento/helm/mamaloo-app/templates/database-secret.yaml`.

No ArgoCD, os valores são passados via `values-dev.yaml` e `values-prod.yaml`.

⚠️ **Importante**: Nunca commitar credenciais em plain text. Use:
- GitHub Secrets para CI/CD
- Sealed Secrets / External Secrets Operator para Kubernetes

## 📊 CI/CD Integration

Quando há push em `Portal-Mamaloo-desenvolvimento`:

1. GitHub Actions builda imagens Docker
2. Push das imagens para GHCR
3. Update automático de `values-dev.yaml` ou `values-prod.yaml`
4. ArgoCD detecta mudanças (auto-sync)
5. Deploy automático via sync waves

## 🛠️ Troubleshooting

### Application stuck em "Syncing"

```bash
argocd app sync portal-mamaloo-dev --retry-limit 0
```

### Ver logs detalhados

```bash
argocd app logs portal-mamaloo-dev --follow
```

### Health check falhou

```bash
kubectl get all -n mamaloo-dev
kubectl describe pod -l app=portal-mamaloo -n mamaloo-dev
```

### Database operator não started

```bash
kubectl get pods -n cnpg-system
kubectl logs -n cnpg-system -l app.kubernetes.io/name=cloudnative-pg
```

## 📝 Modificar Configurações

Para alterar recursos, memory, replicas, etc:

1. Editar `argocd/apps/dev/portal-mamaloo.yaml` ou `prod`
2. Modificar a referência para `values-dev.yaml` ou `values-prod.yaml`
3. Fazer push
4. ArgoCD sincroniza automaticamente

Exemplo:
```yaml
helm:
  valueFiles:
    - values.yaml
    - values-dev.yaml
    - values-custom.yaml  # Arquivo custom adicional
```

## 🔄 Branches

- `main`: Código de produção (prod deployment)
- `develop`: Código de desenvolvimento (dev deployment)
- Este repo: Não tem deploy code, apenas GitOps config

## 📞 Links

- [Portal Mamaloo Repository](https://github.com/Analarie/Portal-Mamaloo-desenvolvimento)
- [ArgoCD Docs](https://argo-cd.readthedocs.io/)
- [CloudNative PG Docs](https://cloudnative-pg.io/)
- [Helm Chart](../Portal-Mamaloo-desenvolvimento/helm/mamaloo-app/)

---

**Última atualização**: 16 de novembro de 2025  
**Status**: ✅ Production Ready
