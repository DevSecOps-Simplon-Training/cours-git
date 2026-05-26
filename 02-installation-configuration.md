# 02 - Installation et Configuration

[← 01 - Introduction](01-introduction-concepts.md) | [🏠 Accueil](README.md) | [03 - Premiers pas →](03-premiers-pas.md)

---

## 1. Installation de Git

### 🍎 macOS
```bash
# Avec Homebrew
brew install git
# Ou via Xcode
xcode-select --install
```

### 🐧 Linux (Debian/Ubuntu)
```bash
sudo apt-get update
sudo apt-get install git
```

### 🪟 Windows
Téléchargez Git depuis [git-scm.com](https://git-scm.com/download/win) ou installez [Git for Windows](https://gitforwindows.org/).

---

## 2. Configuration initiale (OBLIGATOIRE)

Avant votre premier commit, vous devez configurer votre identité :

```bash
# Identité (apparaîtra dans l'historique)
git config --global user.name "Votre Prénom Nom"
git config --global user.email "votre.email@example.com"

# Éditeur par défaut (ex: VS Code)
git config --global core.editor "code --wait"

# Branche par défaut
git config --global init.defaultBranch main
```

### Voir la configuration
```bash
git config --list
```

---

## 3. Les Alias : Gagner du temps
Créez des raccourcis pour les commandes fréquentes :

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.lg 'log --oneline --graph --all --decorate'
```

Usage : `git st` au lieu de `git status`.

---

## 4. Authentification SSH (Recommandé)

L'authentification par **clé SSH** est plus sécurisée et plus pratique que le mot de passe : vous n'avez plus à vous authentifier à chaque `push` ou `pull`.

### Étape 1 : Générer une paire de clés SSH

```bash
# Générer une clé ED25519 (algorithme moderne et recommandé)
ssh-keygen -t ed25519 -C "votre.email@example.com"

# Laisser le chemin par défaut (~/.ssh/id_ed25519)
# Définir une passphrase (optionnel mais conseillé)
```

### Étape 2 : Démarrer l'agent SSH et ajouter la clé

```bash
# Démarrer l'agent SSH en arrière-plan
eval "$(ssh-agent -s)"

# Ajouter votre clé privée à l'agent
ssh-add ~/.ssh/id_ed25519
```

### Étape 3 : Copier la clé publique

```bash
# Afficher la clé publique (à copier)
cat ~/.ssh/id_ed25519.pub
```

### Étape 4 : Ajouter la clé sur GitHub / GitLab

**GitHub** : `Settings` → `SSH and GPG keys` → `New SSH key` → Coller la clé publique.

**GitLab** : `Preferences` → `SSH Keys` → Coller la clé publique.

### Étape 5 : Tester la connexion

```bash
# GitHub
ssh -T git@github.com
# Réponse attendue : "Hi username! You've successfully authenticated..."

# GitLab
ssh -T git@gitlab.com
```

### Utiliser SSH au lieu de HTTPS

```bash
# Cloner avec SSH (au lieu de HTTPS)
git clone git@github.com:username/mon-projet.git

# Changer un remote existant de HTTPS vers SSH
git remote set-url origin git@github.com:username/mon-projet.git
```

> 💡 **Astuce** : Sur macOS, ajoutez ces lignes dans `~/.ssh/config` pour que la passphrase soit mémorisée dans le trousseau :
> ```
> Host github.com
>   AddKeysToAgent yes
>   UseKeychain yes
>   IdentityFile ~/.ssh/id_ed25519
> ```

---

[← 01 - Introduction](01-introduction-concepts.md) | [🏠 Accueil](README.md) | [03 - Premiers pas →](03-premiers-pas.md)
