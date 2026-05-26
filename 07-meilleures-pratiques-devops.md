# 07 - Meilleures pratiques pour le DevOps Azure

[← 06 - Avancé](06-commandes-avancees-debogage.md) | [🏠 Accueil](README.md) | [08 - Workflows et Automation →](08-workflows-automation.md)

---

## 1. Le .gitignore : Critique pour l'IaC Azure

En Infrastructure as Code, certains fichiers ne doivent **jamais** être versionnés : états Terraform, credentials Azure, fichiers générés. Un secret commité est un secret compromis.

```gitignore
# ============================================
# TERRAFORM
# ============================================
# État Terraform (contient des données sensibles en clair)
*.tfstate
*.tfstate.*
.terraform/
.terraform.lock.hcl

# Fichiers de plan (peuvent contenir des secrets)
*.tfplan

# Variables locales (surcharges personnelles)
*.auto.tfvars
!example.tfvars          # Garder les fichiers d'exemple

# ============================================
# AZURE CREDENTIALS & SECRETS
# ============================================
.env
.env.*
!.env.example            # Garder le template
*.pem
*.key
service-principal.json
azure-credentials.json

# ============================================
# HELM / KUBERNETES
# ============================================
charts/*.tgz
values-prod.yaml         # Valeurs de production avec secrets

# ============================================
# OUTILS & IDE
# ============================================
.DS_Store
.idea/
.vscode/settings.json    # Garder .vscode/extensions.json
__pycache__/
*.pyc
```

### Utiliser les secrets Azure Key Vault

Ne stockez jamais de secrets dans Git. Utilisez **Azure Key Vault** :

```bash
# Récupérer un secret depuis Key Vault dans un pipeline
az keyvault secret show \
  --vault-name mon-keyvault \
  --name "db-password" \
  --query value -o tsv
```

Dans GitHub Actions, utilisez les **GitHub Secrets** pour stocker les credentials du Service Principal Azure.

---

## 2. Conventional Commits

Les messages de commit structurés permettent de générer des changelogs automatiques et de déclencher des actions CI/CD conditionnelles.

| Préfixe | Usage DevOps Azure | Exemple |
|---|---|---|
| `feat:` | Nouvelle ressource Azure | `feat: add Azure Container Registry` |
| `fix:` | Correction de configuration | `fix: correct NSG inbound rule for port 443` |
| `docs:` | Documentation | `docs: add runbook for AKS scaling` |
| `chore:` | Maintenance, mise à jour | `chore: bump azurerm provider to 3.85` |
| `perf:` | Optimisation | `perf: enable AKS node autoprovisioning` |
| `ci:` | Pipeline CI/CD | `ci: add terraform plan step on PR` |
| `refactor:` | Restructuration IaC | `refactor: split network module from main.tf` |

### Format complet

```
<type>(<scope>): <description courte>

[Corps optionnel : expliquer le POURQUOI, pas le QUOI]

[Footer : liens, breaking changes]
```

**Exemple réel :**

```
feat(aks): add autoscaling to node pool

Enable cluster autoscaler with min 2 / max 10 nodes to handle
peak loads without manual intervention.

Closes #87
```

---

## 3. Structurer son dépôt IaC

Un dépôt bien structuré facilite la collaboration et la navigation.

### Structure recommandée (mono-repo)

```
infra-azure/
├── modules/               # Modules Terraform réutilisables
│   ├── aks/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── outputs.tf
│   ├── network/
│   └── keyvault/
├── environments/          # Configuration par environnement
│   ├── dev/
│   │   ├── main.tf        # Appelle les modules
│   │   └── terraform.tfvars
│   ├── staging/
│   └── prod/
├── kubernetes/            # Manifests K8s par application
│   └── app-backend/
├── .github/workflows/     # Pipelines CI/CD
├── .gitignore
└── README.md
```

---

## 4. Git LFS pour les artefacts binaires

Si votre dépôt contient de gros fichiers binaires (certificats, archives Helm, images de base), utilisez **Git LFS** pour ne pas alourdir le dépôt.

```bash
# Installer Git LFS
git lfs install

# Tracker les fichiers .tgz (archives Helm)
git lfs track "*.tgz"
git add .gitattributes
git commit -m "chore: configure Git LFS for Helm archives"
```

---

[← 06 - Avancé](06-commandes-avancees-debogage.md) | [🏠 Accueil](README.md) | [08 - Workflows et Automation →](08-workflows-automation.md)
