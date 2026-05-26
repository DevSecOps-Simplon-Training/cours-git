# 10 - Projet Fil Rouge : Déploiement d'une infrastructure AKS sur Azure

[← 09 - Exercices](09-exercices.md) | [🏠 Accueil](README.md) | [11 - Solutions →](11-solutions.md)

---

## 🎯 Objectif

Versionner et déployer une infrastructure Azure complète en utilisant Git selon le **Git Flow** : features, releases et hotfix. Ce projet simule les conditions réelles d'un Ingénieur DevOps Azure en entreprise.

## 📋 Scénario

Vous êtes Ingénieur DevOps chez **CloudOps Corp.** Votre mission : versionner le déploiement d'un cluster AKS avec son infrastructure réseau et son pipeline CI/CD, de la conception jusqu'au correctif en production.

## 🌿 Schéma Git Flow du projet

```
main         ●─────────────────────────────●──────────────────●
              \                           / \                  /
develop        ●──────────●─────────────●   \                /
                \         /                  \              /
feature/...      ●───────●    release/v1.0.0  ●────────────●
                                              hotfix/v1.0.1 ●──●
```

---

## 🛠️ Phases du projet

### Phase 1 : Initialisation

**Contexte :** Le projet démarre. Vous posez les fondations du dépôt IaC.

1. Initialisez le dépôt Git local nommé `infra-aks-cloudops`.
2. Créez la structure de dossiers suivante :
   ```
   infra-aks-cloudops/
   ├── modules/
   │   ├── aks/
   │   │   ├── main.tf
   │   │   ├── variables.tf
   │   │   └── outputs.tf
   │   └── network/
   │       ├── main.tf
   │       └── variables.tf
   ├── environments/
   │   └── prod/
   │       ├── main.tf
   │       └── terraform.tfvars.example
   ├── kubernetes/
   │   └── .gitkeep
   ├── .github/
   │   └── workflows/
   │       └── terraform.yml
   ├── .gitignore
   └── README.md
   ```
3. Configurez un `.gitignore` robuste (état Terraform, secrets Azure, fichiers générés).
4. Faites un premier commit sur `main` : `"chore: initial IaC project structure"`.
5. Créez la branche `develop` à partir de `main`.

---

### Phase 2 : Développement Feature

**Contexte :** Vous développez le module Terraform pour créer le cluster AKS.

1. Depuis `develop`, créez la branche `feature/aks-cluster-module`.
2. Dans `modules/aks/main.tf`, écrivez la ressource Terraform `azurerm_kubernetes_cluster` avec un node pool par défaut.
3. Dans `modules/aks/variables.tf`, déclarez les variables `cluster_name`, `resource_group_name`, `location`, `node_count`.
4. Dans `modules/aks/outputs.tf`, exportez `kube_config` et `cluster_id`.
5. Committez : `"feat(aks): add AKS cluster Terraform module"`.
6. Dans `environments/prod/main.tf`, appelez le module AKS avec les valeurs de production.
7. Committez : `"feat(prod): wire AKS module in production environment"`.
8. Fusionnez `feature/aks-cluster-module` dans `develop`.
9. Supprimez la branche feature.

---

### Phase 3 : Release

**Contexte :** Le module AKS est validé, vous préparez la mise en production.

1. Depuis `develop`, créez la branche `release/v1.0.0`.
2. Mettez à jour le `README.md` avec les prérequis (Azure CLI, Terraform, permissions RBAC) et les instructions de déploiement.
3. Committez : `"docs: add deployment instructions for v1.0.0"`.
4. Complétez le workflow GitHub Actions `terraform.yml` avec les jobs `validate`, `plan` (sur PR) et `apply` (sur merge dans `main`).
5. Committez : `"ci: add Terraform CI/CD pipeline for Azure"`.
6. Fusionnez `release/v1.0.0` dans `main`.
7. Taguez la version : `git tag -a v1.0.0 -m "Production release: AKS cluster v1.0.0"`.
8. Fusionnez également `release/v1.0.0` dans `develop` (synchronisation).
9. Supprimez la branche release.

---

### Phase 4 : Hotfix

**Contexte :** 🚨 En production, une alerte de sécurité est levée ! Le cluster AKS a été créé avec l'API Server accessible publiquement (`api_server_authorized_ip_ranges` vide). Il faut restreindre l'accès en urgence directement depuis `main`.

1. Depuis `main` (le code en production), créez la branche `hotfix/v1.0.1`.
2. Dans `modules/aks/main.tf`, ajoutez la variable `api_server_authorized_ip_ranges` et appliquez-la sur la ressource `azurerm_kubernetes_cluster`.
3. Dans `modules/aks/variables.tf`, déclarez la nouvelle variable avec une liste de CIDRs autorisés.
4. Committez : `"fix(aks): restrict API server access to authorized IP ranges"`.
5. Fusionnez `hotfix/v1.0.1` dans `main`.
6. Taguez la correction : `git tag -a v1.0.1 -m "Security hotfix: restrict AKS API server access"`.
7. Fusionnez `hotfix/v1.0.1` dans `develop`.
8. Supprimez la branche hotfix.

> ⚠️ **Règle d'or Git Flow** : Un hotfix part toujours de `main` (l'état de production) et doit être fusionné dans **les deux branches** : `main` ET `develop`.

---

## 📦 Livrable attendu

À la fin du projet, vérifiez votre historique avec `git log --oneline --graph --all` :

```
* (tag: v1.0.1) Merge hotfix/v1.0.1 into main
* fix(aks): restrict API server access to authorized IP ranges
* (tag: v1.0.0) Merge release/v1.0.0 into main
* ci: add Terraform CI/CD pipeline for Azure
* docs: add deployment instructions for v1.0.0
* feat(prod): wire AKS module in production environment
* feat(aks): add AKS cluster Terraform module
* chore: initial IaC project structure
```

[📖 Voir les solutions](11-solutions.md)

---

[← 09 - Exercices](09-exercices.md) | [🏠 Accueil](README.md) | [11 - Solutions →](11-solutions.md)
