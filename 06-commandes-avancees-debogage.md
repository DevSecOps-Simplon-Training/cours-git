# 06 - Commandes avancées et débogage

[← 05 - Collaboration](05-collaboration-remotes.md) | [🏠 Accueil](README.md) | [07 - Meilleures Pratiques DevOps →](07-meilleures-pratiques-devops.md)

---

## 1. Annuler des modifications

### Restaurer un fichier (Undo local)
```bash
# Annuler les modifications locales sur un fichier IaC
git restore main.tf
git restore kubernetes/deployment.yaml
```

### Unstage un fichier
```bash
# Retirer du staging sans perdre les modifications
git restore --staged main.tf
```

### Annuler des commits (Reset)

- `--soft` : Annule le commit, garde les modifs stagées — utile pour reformuler un message.
- `--mixed` : Annule le commit et le staging, garde les fichiers modifiés.
- `--hard` : Annule tout (⚠️ DANGER : perte de données).

```bash
# Annuler le dernier commit mais garder les modifications
git reset --soft HEAD~1

# Annuler complètement (uniquement si jamais pushé)
git reset --hard HEAD~1
```

> ⚠️ **Règle d'or** : N'utilisez jamais `reset --hard` sur des commits déjà poussés vers le remote. Utilisez `git revert` à la place (crée un commit d'annulation sans réécrire l'historique).

```bash
# Annuler un commit déjà pushé, en toute sécurité
git revert <commit_hash>
```

---

## 2. Stash : Mettre de côté

Utile quand vous devez changer de branche en urgence (ex: hotfix de prod) sans committer un travail inachevé sur votre module Terraform.

```bash
git stash                                    # Sauvegarder le travail en cours
git stash push -m "wip: AKS node pool config"  # Avec un message descriptif
git stash list                               # Voir la liste des stashs
git stash pop                                # Récupérer et supprimer le dernier
git stash apply stash@{1}                    # Appliquer sans supprimer
```

---

## 3. Débogage et Recherche

### Blame : qui a modifié quoi ?

Indispensable en audit : identifier qui a modifié une règle de sécurité ou une configuration réseau.

```bash
git blame main.tf
git blame kubernetes/deployment.yaml
```

### Bisect : trouver le commit fautif

`git bisect` effectue une recherche binaire dans l'historique pour identifier quel commit a introduit un problème (ex: une règle NSG qui bloque du trafic).

```bash
git bisect start
git bisect bad                  # Le commit actuel est cassé
git bisect good v1.2.0          # Cette version fonctionnait

# Git vous positionne sur un commit intermédiaire
# Testez, puis indiquez :
git bisect good   # ou
git bisect bad

# Quand Git a trouvé le commit fautif :
git bisect reset  # Revenir à l'état normal
```

### Reflog : votre filet de sécurité ultime

Le reflog enregistre **toutes** les actions Git, même les reset. Si vous avez supprimé des commits par erreur, le reflog vous sauve.

```bash
git reflog                        # Voir tout l'historique des actions
git checkout HEAD@{3}             # Revenir à l'état d'il y a 3 actions
```

---

## 4. Tags : versionner vos releases Azure

Les tags marquent les versions déployées en production. En DevOps Azure, un tag déclenche souvent un pipeline de déploiement.

```bash
# Créer un tag annoté
git tag -a v1.2.0 -m "Release: add AKS autoscaling and Key Vault integration"

# Pousser le tag vers le remote (déclencheur CI/CD possible)
git push origin v1.2.0

# Pousser tous les tags
git push origin --tags

# Lister les tags
git tag -l
```

### Utilisation dans GitHub Actions

```yaml
# Déclencher un déploiement Azure uniquement sur un tag
on:
  push:
    tags:
      - 'v*'
```

---

[← 05 - Collaboration](05-collaboration-remotes.md) | [🏠 Accueil](README.md) | [07 - Meilleures Pratiques DevOps →](07-meilleures-pratiques-devops.md)
