# 01 - Introduction à Git et au versionnage

[🏠 Accueil](README.md) | [02 - Installation et Configuration →](02-installation-configuration.md)

---

## 1. Qu'est-ce que Git ?

**Git** est un système de contrôle de version distribué créé par Linus Torvalds en 2005.
C'est l'outil le plus utilisé au monde pour gérer les versions de code source et suivre l'historique des modifications d'un projet.

![Git Logo](https://git-scm.com/images/logos/downloads/Git-Logo-2Color.png)

### Pourquoi Git est essentiel en DevOps Azure ?

En DevOps Azure, tout est code : les infrastructures (Terraform, Bicep), les pipelines CI/CD, les configurations Kubernetes, les scripts Azure CLI. Git est le fil conducteur qui relie chaque modification à un contexte et une personne.

- **🏗️ Versionnage de l'Infrastructure as Code** : Suivez chaque modification de vos fichiers Terraform, Bicep ou ARM templates.
- **🤝 Collaboration d'équipe** : Plusieurs ingénieurs travaillent simultanément sur la même infrastructure sans écraser leurs modifications.
- **🔄 Reproductibilité** : Retournez à l'état exact de votre infrastructure à n'importe quel point dans le temps.
- **🚀 Intégration CI/CD** : Déclenchez automatiquement un `terraform apply` ou un déploiement AKS à chaque merge sur `main`.
- **🔍 Audit et conformité** : Identifiez qui a modifié une règle de sécurité, un RBAC ou un pare-feu Azure, quand et pourquoi.

---

## 2. Les concepts fondamentaux

| Concept | Description | Analogie DevOps |
|---|---|---|
| **Repository (Dépôt)** | Conteneur qui stocke tout l'historique du projet | Un dépôt Terraform avec tout son historique |
| **Commit** | Snapshot de vos fichiers à un instant T | Un état figé de votre infrastructure Azure |
| **Branch (Branche)** | Ligne de développement indépendante | Un environnement de staging isolé |
| **Merge (Fusion)** | Combinaison de deux branches | Promotion d'une feature en production |
| **Remote** | Version du dépôt hébergée sur un serveur | Votre repo GitHub ou Azure DevOps Repos |

---

## 3. Architecture Git : Distribué vs Centralisé

Contrairement aux systèmes centralisés (SVN, CVS), Git est **distribué**. Chaque ingénieur possède une copie complète de l'historique, permettant de travailler hors ligne et de créer des branches sans toucher au serveur central.

### Les trois états de Git

1. **Working Directory** : Vos fichiers actuels (`main.tf`, `deployment.yaml`, etc.).
2. **Staging Area (Index)** : Zone de préparation avant le commit.
3. **Repository (.git)** : Base de données contenant tout l'historique.

```text
Working Directory  ─────►  Staging Area  ─────►  Repository
   (main.tf modifié)         (Préparé)            (Commité)
```

---

## 4. Git dans l'écosystème DevOps Azure

Git est la pierre angulaire de toute la chaîne DevOps :

```
Code / IaC        CI/CD              Cloud Azure
──────────        ──────             ───────────
git push    →   GitHub Actions   →   az deployment
            →   terraform plan   →   Azure Resources
            →   docker build     →   ACR / AKS
```

Les plateformes principales utilisées en entreprise pour héberger les dépôts Git sont **GitHub** (avec GitHub Actions pour le CI/CD) et **Azure DevOps Repos** (avec Azure Pipelines).

---

[🏠 Accueil](README.md) | [02 - Installation et Configuration →](02-installation-configuration.md)
