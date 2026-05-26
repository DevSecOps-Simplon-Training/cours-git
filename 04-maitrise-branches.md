# 04 - Maîtriser les branches

[← 03 - Premiers pas](03-premiers-pas.md) | [🏠 Accueil](README.md) | [05 - Collaboration et Remotes →](05-collaboration-remotes.md)

---

## 1. Pourquoi utiliser des branches ?
Les branches permettent d'isoler le développement. En DevOps Azure, chaque modification d'infrastructure (nouveau module Terraform, mise à jour d'un manifest Kubernetes, ajout d'un pipeline) passe par une branche dédiée avant d'être fusionnée dans `main` — qui représente l'état **déployé en production**.

---

## 2. Gérer les branches

### Créer et naviguer
```bash
git branch feature/aks-cluster       # Créer
git switch feature/aks-cluster       # Basculer
# OU (syntaxe moderne pour créer et changer)
git switch -c feature/aks-cluster
```

### Nommage recommandé en DevOps Azure

| Préfixe | Usage | Exemple |
|---|---|---|
| `feature/` | Nouvelle ressource ou fonctionnalité | `feature/keyvault-integration` |
| `fix/` | Correction de configuration | `fix/nsg-rule-typo` |
| `hotfix/` | Correctif urgent en production | `hotfix/aks-node-pool-crash` |
| `release/` | Préparation d'une version | `release/v2.1.0` |
| `chore/` | Maintenance (docs, refacto) | `chore/update-providers` |

### Fusionner (Merge)
```bash
git switch main
git merge feature/aks-cluster
```

---

## 3. Stratégies courantes

**GitHub Flow** : Une branche `main` stable, chaque feature part de `main` et y retourne après review. Adapté aux déploiements continus (CD) où chaque merge déclenche un `terraform apply`.

**Git Flow** : Plus structuré, avec branches `develop`, `release` et `hotfix`. Recommandé quand vous gérez plusieurs environnements Azure (dev, staging, prod) avec des cycles de release planifiés.

---

## 4. Résoudre les conflits

Un conflit survient quand deux branches modifient la même ligne — par exemple, deux ingénieurs qui ont chacun modifié la taille d'un node pool AKS dans `main.tf`.

1. Tentez le merge : `git merge feature/X`.
2. Si conflit, ouvrez le fichier et cherchez les marqueurs `<<<<<<<`, `=======`, `>>>>>>>`.
3. Éditez pour garder le bon code (concertez-vous avec le collègue !).
4. Marquez comme résolu : `git add <fichier>`.
5. Finalisez : `git commit`.

---

## 5. Rebase : Réécrire l'historique proprement

Le `rebase` est une alternative au `merge`. Au lieu de créer un commit de fusion, il **réapplique** vos commits sur la pointe d'une autre branche, produisant un historique linéaire et lisible.

### Rebase simple

```bash
# Situation : vous êtes sur feature/aks-cluster
# et main a avancé depuis que vous avez branché

git switch feature/aks-cluster
git rebase main
# Git réapplique vos commits un par un sur la pointe de main
```

**Avant rebase :**
```
main:    A──B──C
feature:    \──D──E
```

**Après rebase :**
```
main:    A──B──C
feature:          D'──E'
```

### Rebase interactif : `git rebase -i`

Le rebase interactif permet de **nettoyer vos commits avant de les partager** : fusionner des micro-commits, reformuler des messages, supprimer des commits inutiles.

```bash
# Réécrire les 3 derniers commits
git rebase -i HEAD~3
```

Un éditeur s'ouvre avec la liste de vos commits :

```
pick a1b2c3 feat: add AKS node pool configuration
pick d4e5f6 fix typo in variables.tf
pick g7h8i9 wip: debug output

# Commandes disponibles :
# pick   = conserver le commit tel quel
# reword = conserver mais modifier le message
# squash = fusionner avec le commit précédent
# drop   = supprimer le commit
```

**Exemple : fusionner les 3 commits en un seul**

```
pick a1b2c3 feat: add AKS node pool configuration
squash d4e5f6 fix typo in variables.tf
squash g7h8i9 wip: debug output
```

Git fusionnera les trois en un seul commit propre et vous demandera de rédiger le message final.

### Merge vs Rebase : quand choisir ?

| Situation | Recommandation |
|---|---|
| Branche partagée avec l'équipe | `merge` (ne pas réécrire l'historique public) |
| Branche locale avant une PR | `rebase` (historique propre et linéaire) |
| Synchroniser sa feature avec main | `rebase main` |

> ⚠️ **Règle d'or** : Ne jamais faire de `rebase` sur une branche déjà poussée et partagée avec d'autres développeurs. Cela réécrit l'historique et crée des conflits pour tout le monde.

---

[← 03 - Premiers pas](03-premiers-pas.md) | [🏠 Accueil](README.md) | [05 - Collaboration et Remotes →](05-collaboration-remotes.md)
