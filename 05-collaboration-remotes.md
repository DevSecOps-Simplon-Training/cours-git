# 05 - Collaboration avec des dépôts distants

[← 04 - Branches](04-maitrise-branches.md) | [🏠 Accueil](README.md) | [06 - Commandes Avancées →](06-commandes-avancees-debogage.md)

---

## 1. Qu'est-ce qu'un remote ?

C'est la version de votre dépôt hébergée sur un serveur. En DevOps Azure, vous utiliserez principalement **GitHub** (avec GitHub Actions) ou **Azure DevOps Repos** (avec Azure Pipelines).

### Gérer les remotes
```bash
git remote add origin git@github.com:mon-org/infra-azure.git
git remote -v          # Voir la liste des remotes
git remote remove origin   # Supprimer un remote
```

---

## 2. Synchroniser le code

### Envoyer (Push)
```bash
git push origin main
git push -u origin feature/aks-cluster   # -u lie la branche locale à la distante
```

### Récupérer (Fetch / Pull)

- **Fetch** : Télécharge les commits distants sans les fusionner — utile pour inspecter avant d'intégrer.
- **Pull** : Télécharge ET fusionne (Fetch + Merge).

```bash
git fetch origin         # Inspecter sans risque
git pull origin main     # Récupérer et fusionner
```

---

## 3. Workflow collaboratif DevOps Azure

En équipe, chaque modification d'infrastructure passe par une **Pull Request (PR)** — aussi appelée **Merge Request** sur GitLab/Azure DevOps. Cela garantit qu'une revue humaine a lieu avant tout déploiement.

```
1. Pull les dernières modifs de main
2. Créer une branche feature/fix
3. Modifier les fichiers IaC (Terraform, Bicep, YAML)
4. Commit et Push la branche
5. Ouvrir une Pull Request
6. Revue du code + terraform plan automatique (CI)
7. Approbation et Merge dans main
8. Déploiement automatique déclenché (CD)
```

### Bonnes pratiques pour une PR d'infrastructure

- **Titre clair** : `feat: add AKS cluster with autoscaling` plutôt que `update infra`
- **Description** : expliquer *pourquoi* le changement, pas juste *quoi*
- **Lier un ticket** : `Closes #42` pour tracer la décision
- **Inclure le plan Terraform** : joindre la sortie de `terraform plan` en commentaire pour faciliter la review

---

## 4. Protection de la branche `main`

En entreprise, la branche `main` est **protégée** : personne ne peut pusher directement dessus. Tout passe par une PR approuvée. Configurez cela dans :

**GitHub** : `Settings → Branches → Add branch protection rule`
- ✅ Require pull request reviews before merging
- ✅ Require status checks to pass (votre CI/CD)
- ✅ Restrict who can push to matching branches

---

[← 04 - Branches](04-maitrise-branches.md) | [🏠 Accueil](README.md) | [06 - Commandes Avancées →](06-commandes-avancees-debogage.md)
