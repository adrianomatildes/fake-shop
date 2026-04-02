# Fake Shop

## Arquitetura & Deploy

Este projeto demonstra uma pipeline GitOps completa:

### Stack
- **App:** Python Flask + PostgreSQL
- **CI:** GitHub Actions (build, test, push image)
- **CD:** ArgoCD sincronizando automaticamente
- **Infra:** K3s local com Kustomize overlays

### Fluxo de Deploy
1. Push na branch `main` dispara GitHub Actions
2. Pipeline builda imagem Docker e atualiza tag em `devops-config`
3. ArgoCD detecta mudança e synca automaticamente
4. Aplicação deployada no namespace `dev`

### Status do Ambiente

![ArgoCD Healthy](docs/argocd-healthy-synced.png)
*ArgoCD: Healthy & Synced - 02/04/2026*

![Pods Running](docs/pods-running.png)
*Pod fake-shop rodando no namespace dev*

### Estrutura
├── src/                    # Código Python
├── k8s/                    # Manifests base
├── .github/workflows/      # CI Pipeline
└── docs/                   # Evidências

### Como Executar Local
Pré-requisitos: K3s, ArgoCD, kubectl

```bash
# Aplique o ApplicationSet
kubectl apply -f https://dev.azure.com/.../applicationset-fakeshop.yaml

# Acesse
kubectl port-forward svc/fake-shop -n dev 8080:80
curl localhost:8080
```

## Variável de Ambiente
| Variável                   | Descrição                             |
| -------------------------- | ------------------------------------- |
| `DB_HOST`                  | Host do PostgreSQL                    |
| `DB_USER`                  | Usuário do banco                      |
| `DB_PASSWORD`              | Senha do usuário                      |
| `DB_NAME`                  | Nome do database                      |
| `DB_PORT`                  | Porta (default 5432)                  |
| `FLASK_APP`                | Arquivo de inicialização (`index.py`) |
| `PROMETHEUS_MULTIPROC_DIR` | Diretório métricas (`/tmp/metrics`)   |