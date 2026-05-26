# 08 - Workflows et Automation

[← 07 - Pratiques DevOps](07-meilleures-pratiques-devops.md) | [🏠 Accueil](README.md) | [09 - Exercices →](09-exercices.md)

---

## 1. Pre-commit Hooks pour l'IaC Azure

Les hooks Git sont des scripts exécutés automatiquement avant chaque commit. En DevOps Azure, ils permettent de valider la qualité de votre code Terraform, YAML et Bicep **avant** qu'il n'atteigne le dépôt.

### Installation

```bash
pip install pre-commit
```

### Configuration : `.pre-commit-config.yaml`

```yaml
# .pre-commit-config.yaml
repos:
  # Vérifications générales
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace       # Espaces en fin de ligne
      - id: end-of-file-fixer         # Saut de ligne final
      - id: check-yaml                # Syntaxe YAML (K8s, pipelines)
      - id: check-added-large-files   # Bloque fichiers > 500 KB
        args: ['--maxkb=500']
      - id: detect-private-key        # Détecte les clés privées accidentelles

  # Formatage et validation Terraform
  - repo: https://github.com/antonbabenko/pre-commit-terraform
    rev: v1.88.0
    hooks:
      - id: terraform_fmt             # Formate automatiquement les .tf
      - id: terraform_validate        # Valide la syntaxe Terraform
      - id: terraform_tflint          # Linting Terraform (bonnes pratiques)
      - id: terraform_docs            # Génère la doc des modules

  # Linting YAML (manifests Kubernetes, pipelines)
  - repo: https://github.com/adrienverge/yamllint
    rev: v1.35.1
    hooks:
      - id: yamllint
        args: ['--strict']

  # Détection de secrets dans le code (Azure credentials, tokens)
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
```

### Activation

```bash
# Installer les hooks (une seule fois par développeur)
pre-commit install

# Tester sur tous les fichiers existants
pre-commit run --all-files

# Mettre à jour les versions des hooks
pre-commit autoupdate
```

---

## 2. CI/CD GitHub Actions pour Azure

GitHub Actions permet de déclencher automatiquement des validations et déploiements Azure à chaque modification du dépôt.

### Workflow complet : Terraform sur Azure

```yaml
# .github/workflows/terraform.yml
name: Terraform Azure CI/CD

on:
  pull_request:
    branches: [ main ]
  push:
    branches: [ main ]

env:
  ARM_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
  ARM_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
  ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  ARM_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}

jobs:
  terraform-validate:
    name: Validate
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: '1.7.0'

      - name: Terraform Init
        run: terraform init
        working-directory: ./environments/prod

      - name: Terraform Format Check
        run: terraform fmt -check -recursive

      - name: Terraform Validate
        run: terraform validate
        working-directory: ./environments/prod

  terraform-plan:
    name: Plan
    needs: terraform-validate
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Terraform Init
        run: terraform init
        working-directory: ./environments/prod

      - name: Terraform Plan
        id: plan
        run: terraform plan -no-color -out=tfplan
        working-directory: ./environments/prod

      # Publier le plan en commentaire de la PR
      - name: Comment PR with Plan
        uses: actions/github-script@v7
        with:
          script: |
            const output = `#### Terraform Plan 📋
            \`\`\`
            ${{ steps.plan.outputs.stdout }}
            \`\`\``;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            })

  terraform-apply:
    name: Apply
    needs: terraform-validate
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Terraform Init
        run: terraform init
        working-directory: ./environments/prod

      - name: Terraform Apply
        run: terraform apply -auto-approve
        working-directory: ./environments/prod
```

### Déclencheurs courants (`on:`)

| Déclencheur | Usage DevOps Azure |
|---|---|
| `pull_request` | Lancer `terraform plan` et afficher le diff en PR |
| `push: branches: [main]` | Lancer `terraform apply` après merge |
| `push: tags: ['v*']` | Déployer une release versionnée |
| `workflow_dispatch` | Déploiement manuel depuis l'interface GitHub |
| `schedule` | Audit de conformité quotidien |

---

## 3. Git + Docker + Azure Container Registry (ACR)

En DevOps Azure, les images Docker sont poussées vers **Azure Container Registry (ACR)** et déployées sur **AKS**. Le hash du commit Git garantit la traçabilité.

### Tagger une image avec le hash Git

```bash
COMMIT_HASH=$(git rev-parse --short HEAD)
ACR_NAME="monregistry.azurecr.io"
IMAGE="mon-app"

# Construire
docker build -t ${ACR_NAME}/${IMAGE}:${COMMIT_HASH} .

# S'authentifier sur ACR
az acr login --name monregistry

# Pousser
docker push ${ACR_NAME}/${IMAGE}:${COMMIT_HASH}
```

### Workflow GitHub Actions : Build → ACR → AKS

```yaml
name: Build and Deploy to AKS

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Login Azure
        uses: azure/login@v2
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: Build and push to ACR
        run: |
          COMMIT_HASH=$(git rev-parse --short HEAD)
          az acr build \
            --registry monregistry \
            --image mon-app:${COMMIT_HASH} \
            --image mon-app:latest .

      - name: Deploy to AKS
        uses: azure/k8s-deploy@v4
        with:
          manifests: kubernetes/deployment.yaml
          images: monregistry.azurecr.io/mon-app:${{ github.sha }}
```

> 🔍 **Traçabilité** : L'image `mon-app:a3f5c12` en production correspond exactement au commit `a3f5c12` dans Git. En cas d'incident, `git show a3f5c12` révèle le changement exact qui a causé le problème.

---

[← 07 - Pratiques DevOps](07-meilleures-pratiques-devops.md) | [🏠 Accueil](README.md) | [09 - Exercices →](09-exercices.md)
