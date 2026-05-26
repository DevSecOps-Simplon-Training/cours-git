# 11 - Solutions des exercices

[← 10 - Projet Fil Rouge](10-projet-fil-rouge.md) | [🏠 Accueil](README.md)

---

## 📝 Solution Niveau 1 : Bases

```bash
# Identité
git config --global user.name "Nom Prénom"
git config --global user.email "email@cloudops.com"

# Éditeur par défaut
git config --global core.editor "code --wait"

# Branche par défaut
git config --global init.defaultBranch main

# Créer le dépôt
mkdir git-devops-practice && cd git-devops-practice
git init

# Créer le fichier versions.tf et committer
cat > versions.tf << 'EOF'
terraform {
  required_version = ">= 1.5.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.85"
    }
  }
}
EOF

git add versions.tf
git commit -m "chore: add Terraform version constraints"

# Alias utiles
git config --global alias.lg 'log --oneline --graph --all --decorate'
git config --global alias.st status
```

---

## 🌿 Solution Niveau 2 : Branches

```bash
# Créer et basculer sur la branche feature
git switch -c feature/add-resource-group

# Créer main.tf avec une ressource Azure Resource Group
cat > main.tf << 'EOF'
resource "azurerm_resource_group" "main" {
  name     = var.resource_group_name
  location = var.location
}
EOF

git add main.tf
git commit -m "feat: add Azure Resource Group resource"

# Retourner sur main et fusionner
git switch main
git merge feature/add-resource-group

# Supprimer la branche feature
git branch -d feature/add-resource-group

# --- BONUS : Même exercice avec rebase ---
git switch -c feature/add-rg-v2
echo '# network' > network.tf
git add network.tf
git commit -m "feat: add network placeholder"

git switch main
git switch -c feature/add-rg-v2
# Simuler une avancée de main
echo "# updated" >> versions.tf
git add versions.tf
git commit -m "chore: update comment"

git switch feature/add-rg-v2
git rebase main   # Historique linéaire, sans commit de merge

git switch main
git merge feature/add-rg-v2   # Fast-forward propre
git branch -d feature/add-rg-v2

# Comparer avec un merge classique :
git log --oneline --graph
```

---

## 🤝 Solution Niveau 3 : Collaboration

```bash
# 1. Lier le dépôt local à GitHub via SSH
git remote add origin git@github.com:votre-user/git-devops-practice.git

# 2. Pousser la branche main
git push -u origin main

# 3. Créer et pousser une branche feature
git switch -c feature/add-vnet

cat > network.tf << 'EOF'
resource "azurerm_virtual_network" "main" {
  name                = "vnet-cloudops-prod"
  address_space       = ["10.0.0.0/16"]
  location            = var.location
  resource_group_name = var.resource_group_name
}
EOF

git add network.tf
git commit -m "feat(network): add Azure Virtual Network"
git push -u origin feature/add-vnet

# 4. Ouvrir une Pull Request depuis GitHub :
#    GitHub → votre repo → "Compare & pull request"
#    Titre : "feat(network): add Azure Virtual Network"
#    Description : expliquer l'adresse CIDR choisie et pourquoi
#    Cliquer "Create pull request" puis "Merge pull request"

# 5. Récupérer le merge en local et nettoyer
git switch main
git pull origin main
git branch -d feature/add-vnet
```

---

## 🔒 Solution Niveau 4 : Sécurité et bonnes pratiques

```bash
# 1. Créer le .gitignore Terraform/Azure
cat > .gitignore << 'EOF'
# Terraform
*.tfstate
*.tfstate.*
.terraform/
.terraform.lock.hcl
*.tfplan
*.auto.tfvars
!example.tfvars

# Azure secrets
.env
.env.*
!.env.example
*.pem
*.key
service-principal.json
EOF

git add .gitignore
git commit -m "chore: add Terraform and Azure .gitignore"

# 2. Tester que .gitignore bloque .env
echo "AZURE_CLIENT_SECRET=mon_super_secret" > .env
git status   # .env ne doit PAS apparaître
git add .env 2>&1 || echo ".env correctement ignoré"

# 3. Installer pre-commit et configurer detect-private-key
pip install pre-commit

cat > .pre-commit-config.yaml << 'EOF'
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: detect-private-key
      - id: check-yaml
      - id: trailing-whitespace
EOF

pre-commit install

# Test : tenter de committer une fausse clé privée
echo "-----BEGIN RSA PRIVATE KEY-----" > fake-key.pem
git add fake-key.pem
git commit -m "test: should be blocked"
# → Le hook detect-private-key doit bloquer le commit

# Nettoyage
git restore --staged fake-key.pem
rm fake-key.pem
git add .pre-commit-config.yaml
git commit -m "ci: add pre-commit hooks for security"

# 4. Créer et pousser un tag
git tag -a v0.1.0 -m "Initial IaC structure with security hooks"
git push origin v0.1.0
```

---

## 🚀 Solution Projet Fil Rouge

### Phase 1 : Initialisation

```bash
mkdir infra-aks-cloudops && cd infra-aks-cloudops
git init

# Structure du projet
mkdir -p modules/aks modules/network environments/prod kubernetes .github/workflows

touch modules/aks/main.tf modules/aks/variables.tf modules/aks/outputs.tf
touch modules/network/main.tf modules/network/variables.tf
touch environments/prod/main.tf environments/prod/terraform.tfvars.example
touch kubernetes/.gitkeep
touch .github/workflows/terraform.yml
touch README.md

cat > .gitignore << 'EOF'
*.tfstate
*.tfstate.*
.terraform/
.terraform.lock.hcl
*.tfplan
*.auto.tfvars
!*.tfvars.example
.env
*.pem
*.key
EOF

git add .
git commit -m "chore: initial IaC project structure"
git switch -c develop
```

---

### Phase 2 : Développement Feature

```bash
git switch -c feature/aks-cluster-module

# Module AKS - main.tf
cat > modules/aks/main.tf << 'EOF'
resource "azurerm_kubernetes_cluster" "main" {
  name                = var.cluster_name
  location            = var.location
  resource_group_name = var.resource_group_name
  dns_prefix          = var.cluster_name

  default_node_pool {
    name       = "default"
    node_count = var.node_count
    vm_size    = "Standard_D2_v2"
  }

  identity {
    type = "SystemAssigned"
  }
}
EOF

# Module AKS - variables.tf
cat > modules/aks/variables.tf << 'EOF'
variable "cluster_name" {
  type        = string
  description = "Nom du cluster AKS"
}

variable "resource_group_name" {
  type        = string
  description = "Nom du Resource Group Azure"
}

variable "location" {
  type        = string
  description = "Région Azure"
  default     = "West Europe"
}

variable "node_count" {
  type        = number
  description = "Nombre de nœuds dans le node pool"
  default     = 2
}
EOF

# Module AKS - outputs.tf
cat > modules/aks/outputs.tf << 'EOF'
output "kube_config" {
  value     = azurerm_kubernetes_cluster.main.kube_config_raw
  sensitive = true
}

output "cluster_id" {
  value = azurerm_kubernetes_cluster.main.id
}
EOF

git add modules/aks/
git commit -m "feat(aks): add AKS cluster Terraform module"

# Environnement de production
cat > environments/prod/main.tf << 'EOF'
module "aks" {
  source              = "../../modules/aks"
  cluster_name        = "aks-cloudops-prod"
  resource_group_name = "rg-cloudops-prod"
  location            = "West Europe"
  node_count          = 3
}
EOF

git add environments/prod/main.tf
git commit -m "feat(prod): wire AKS module in production environment"

git switch develop
git merge feature/aks-cluster-module
git branch -d feature/aks-cluster-module
```

---

### Phase 3 : Release

```bash
git switch -c release/v1.0.0

cat > README.md << 'EOF'
# Infrastructure AKS - CloudOps Corp

## Prérequis
- Azure CLI >= 2.50
- Terraform >= 1.5.0
- Permissions : Contributor sur la subscription Azure

## Déploiement
az login
cd environments/prod
terraform init
terraform plan
terraform apply
EOF

git add README.md
git commit -m "docs: add deployment instructions for v1.0.0"

cat > .github/workflows/terraform.yml << 'EOF'
name: Terraform Azure CI/CD
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
env:
  ARM_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
  ARM_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
  ARM_SUBSCRIPTION_ID: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  ARM_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
      - run: terraform init
        working-directory: ./environments/prod
      - run: terraform validate
        working-directory: ./environments/prod
  apply:
    needs: validate
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    steps:
      - uses: actions/checkout@v4
      - uses: hashicorp/setup-terraform@v3
      - run: terraform init && terraform apply -auto-approve
        working-directory: ./environments/prod
EOF

git add .github/workflows/terraform.yml
git commit -m "ci: add Terraform CI/CD pipeline for Azure"

git switch main
git merge release/v1.0.0
git tag -a v1.0.0 -m "Production release: AKS cluster v1.0.0"

git switch develop
git merge release/v1.0.0
git branch -d release/v1.0.0
```

---

### Phase 4 : Hotfix

```bash
git switch main
git switch -c hotfix/v1.0.1

# Corriger la faille de sécurité : restreindre l'accès à l'API Server
cat > modules/aks/variables.tf << 'EOF'
variable "cluster_name" {
  type        = string
  description = "Nom du cluster AKS"
}

variable "resource_group_name" {
  type        = string
  description = "Nom du Resource Group Azure"
}

variable "location" {
  type        = string
  description = "Région Azure"
  default     = "West Europe"
}

variable "node_count" {
  type        = number
  description = "Nombre de nœuds dans le node pool"
  default     = 2
}

variable "api_server_authorized_ip_ranges" {
  type        = list(string)
  description = "CIDRs autorisés à accéder à l'API Server AKS"
}
EOF

# Appliquer la restriction dans la ressource
cat > modules/aks/main.tf << 'EOF'
resource "azurerm_kubernetes_cluster" "main" {
  name                = var.cluster_name
  location            = var.location
  resource_group_name = var.resource_group_name
  dns_prefix          = var.cluster_name

  default_node_pool {
    name       = "default"
    node_count = var.node_count
    vm_size    = "Standard_D2_v2"
  }

  api_server_access_profile {
    authorized_ip_ranges = var.api_server_authorized_ip_ranges
  }

  identity {
    type = "SystemAssigned"
  }
}
EOF

git add modules/aks/
git commit -m "fix(aks): restrict API server access to authorized IP ranges"

git switch main
git merge hotfix/v1.0.1
git tag -a v1.0.1 -m "Security hotfix: restrict AKS API server access"

git switch develop
git merge hotfix/v1.0.1
git branch -d hotfix/v1.0.1
```

> ✅ **Vérification finale** : `git log --oneline --graph --all` pour visualiser l'historique complet avec branches et tags.

---

[← 10 - Projet Fil Rouge](10-projet-fil-rouge.md) | [🏠 Accueil](README.md)
