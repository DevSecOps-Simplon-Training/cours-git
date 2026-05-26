# 09 - Exercices : Pratique de Git en DevOps Azure

[← 08 - Automation](08-workflows-automation.md) | [🏠 Accueil](README.md) | [10 - Projet Fil Rouge →](10-projet-fil-rouge.md)

---

## 📝 Niveau 1 : Bases

1. Configurez votre identité Git (nom, email) et votre éditeur par défaut.
2. Initialisez un dépôt nommé `git-devops-practice`.
3. Créez un fichier `versions.tf` contenant la contrainte de version Terraform suivante et faites votre premier commit :
   ```hcl
   terraform {
     required_version = ">= 1.5.0"
     required_providers {
       azurerm = {
         source  = "hashicorp/azurerm"
         version = "~> 3.85"
       }
     }
   }
   ```
4. Créez un alias `git lg` pour `log --oneline --graph --all --decorate`.
5. Configurez la branche par défaut sur `main`.

---

## 🌿 Niveau 2 : Branches

1. Créez une branche `feature/add-resource-group`.
2. Ajoutez un fichier `main.tf` avec une ressource Azure Resource Group vide et committez avec un message conventionnel (`feat:`).
3. Retournez sur `main` et fusionnez la branche.
4. Supprimez la branche feature.
5. **Bonus** : Reproduisez l'exercice en utilisant `rebase` au lieu de `merge`, puis comparez l'historique avec `git log --graph`.

---

## 🤝 Niveau 3 : Collaboration

1. Créez un dépôt sur GitHub nommé `git-devops-practice`.
2. Liez votre dépôt local au remote via SSH.
3. Poussez votre branche `main`.
4. Créez une branche `feature/add-vnet`, ajoutez un fichier `network.tf` avec un commentaire, et poussez-la.
5. Ouvrez une Pull Request sur GitHub depuis `feature/add-vnet` vers `main`, ajoutez une description expliquant le changement, puis fusionnez-la.
6. Récupérez le résultat en local (`git pull`) et supprimez la branche locale.

---

## 🔒 Niveau 4 : Sécurité et bonnes pratiques

1. Créez un fichier `.gitignore` adapté à un projet Terraform + Azure (ignorez les fichiers `.tfstate`, `.terraform/`, `.env`).
2. Créez volontairement un fichier `.env` contenant `AZURE_CLIENT_SECRET=mon_secret`, tentez de le commiter, et vérifiez que `.gitignore` l'empêche.
3. Installez `pre-commit` et configurez un hook `detect-private-key`. Testez qu'il bloque un commit contenant une fausse clé privée.
4. Créez un tag `v0.1.0` annoté sur votre dernier commit et poussez-le vers GitHub.

[📖 Voir les solutions](11-solutions.md)

---

[← 08 - Automation](08-workflows-automation.md) | [🏠 Accueil](README.md) | [10 - Projet Fil Rouge →](10-projet-fil-rouge.md)
