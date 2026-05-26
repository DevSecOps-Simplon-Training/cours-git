# 03 - Premiers pas : Créer et gérer un dépôt

[← 02 - Configuration](02-installation-configuration.md) | [🏠 Accueil](README.md) | [04 - Maîtrise des branches →](04-maitrise-branches.md)

---

## 1. Initialiser ou Cloner un dépôt

### Nouveau projet local
```bash
mkdir infra-azure
cd infra-azure
git init
```

### Récupérer un projet existant
```bash
# Via HTTPS
git clone https://github.com/username/infra-azure.git

# Via SSH (recommandé, voir module 02)
git clone git@github.com:username/infra-azure.git
```

---

## 2. Le cycle de vie d'un fichier

En DevOps Azure, vos fichiers sont typiquement des configurations IaC (`.tf`, `.bicep`), des manifests Kubernetes (`.yaml`) ou des scripts Azure CLI (`.sh`).

### Vérifier l'état
```bash
git status
```

### Ajouter des modifications (Staging)
```bash
git add main.tf              # Fichier Terraform spécifique
git add kubernetes/           # Tout un dossier de manifests
git add .                    # Tout le dossier actuel
```

### Créer un commit (Snapshot)
```bash
git commit -m "feat: add AKS cluster Terraform module"
```

---

## 3. Consulter l'historique

### Liste des commits
```bash
git log                # Complet
git log --oneline      # Résumé (idéal pour voir les changements d'infra)
git log --graph        # Vue graphique des branches
```

### Voir les modifications d'un commit
```bash
git show <commit_hash>
# Affiche le diff exact : quelles lignes de main.tf ont changé
```

### Voir les différences non committées
```bash
git diff               # Working directory vs Staging
git diff --staged      # Staging vs dernier commit
```

---

## 4. Structure typique d'un dépôt IaC Azure

```
infra-azure/
├── terraform/
│   ├── main.tf          # Ressources principales
│   ├── variables.tf     # Déclaration des variables
│   ├── outputs.tf       # Valeurs exportées
│   └── versions.tf      # Contraintes de versions
├── kubernetes/
│   ├── deployment.yaml
│   └── service.yaml
├── scripts/
│   └── bootstrap.sh     # Script Azure CLI d'initialisation
├── .github/
│   └── workflows/
│       └── deploy.yml   # Pipeline CI/CD GitHub Actions
├── .gitignore
└── README.md
```

---

## 💡 Conseil : Commits atomiques en IaC

Un commit IaC doit représenter **une seule ressource ou une seule décision d'architecture**. Ne mélangez pas la création d'un AKS et la configuration d'un Key Vault dans le même commit. Cela facilite les revues et les retours en arrière en cas d'incident.

```bash
# ✅ Bon : un commit = une ressource
git commit -m "feat: add Azure Key Vault for secrets management"

# ❌ Mauvais : tout dans un commit
git commit -m "update infra"
```

---

[← 02 - Configuration](02-installation-configuration.md) | [🏠 Accueil](README.md) | [04 - Maîtrise des branches →](04-maitrise-branches.md)
