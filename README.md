# Fake Shop

## Arquitetura & Deploy
Este projeto demonstra uma pipeline GitOps completa:

![Arquitetura GitOps](docs/arquitetura-fakeshop.png)
*Fluxo multi-repo: GitHub (público) → Azure DevOps (privado) → ArgoCD → K3s*

### Stack
- **App:** Python Flask + PostgreSQL
- **CI:** GitHub Actions (build, test, push image)
- **CD:** ArgoCD sincronizando automaticamente
- **Infra:** K3s local com Kustomize overlays

### Fluxo de Deploy

Promoção de código entre branches dispara deploy automático:

| Branch | Ambiente | Namespace K8s |
|--------|----------|---------------|
| `dev`  | Desenvolvimento | `dev` |
| `hml`  | Homologação | `hml` |
| `prd`  | Produção | `prd` |

**Processo:**
1. Feature branch → PR para `dev` → merge dispara pipeline → deploy `dev`
2. PR `dev` → `hml` → merge dispara pipeline → deploy `hml`
3. PR `hml` → `prd` → merge dispara pipeline → deploy `prd`

Cada merge atualiza o overlay correspondente no `devops-config`, e o ArgoCD sincroniza o ambiente específico.

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

> **Nota:** Este repo contém apenas a aplicação. O deploy completo requer o [projetos-devops](https://github.com/adrianomatildes/projetos-devops) para setup da infraestrutura.

Pré-requisitos: K3s, ArgoCD, kubectl configurado

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