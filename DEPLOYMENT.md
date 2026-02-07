# 🚀 Guide de Déploiement Complet

Ce guide vous accompagne pas à pas pour déployer votre jeu multijoueur de dés en production.

## 📋 Table des matières

1. [Prérequis](#prérequis)
2. [Configuration Supabase](#configuration-supabase)
3. [Configuration GitHub](#configuration-github)
4. [Déploiement sur Vercel](#déploiement-sur-vercel)
5. [Déploiement sur Netlify](#déploiement-sur-netlify)
6. [Tests Post-Déploiement](#tests-post-déploiement)
7. [Partage avec vos amis](#partage-avec-vos-amis)
8. [Maintenance](#maintenance)

---

## 🎯 Prérequis

- [ ] Compte GitHub (gratuit)
- [ ] Compte Supabase (gratuit)
- [ ] Compte Vercel ou Netlify (gratuit)
- [ ] Node.js 18+ installé localement (pour les tests)

---

## 🗄️ Configuration Supabase

### Étape 1 : Créer un projet Supabase

1. Allez sur [supabase.com](https://supabase.com)
2. Cliquez sur "Start your project"
3. Connectez-vous ou créez un compte
4. Cliquez sur "New Project"
5. Remplissez :
   - **Name** : `dice-game` (ou le nom de votre choix)
   - **Database Password** : Générez un mot de passe fort (notez-le !)
   - **Region** : Choisissez la région la plus proche de vos joueurs
   - **Pricing Plan** : Free tier (suffisant pour commencer)
6. Cliquez sur "Create new project"
7. ⏳ Attendez ~2 minutes que le projet soit prêt

### Étape 2 : Récupérer les clés API

1. Dans votre projet Supabase, allez dans **Settings** (icône engrenage)
2. Cliquez sur **API** dans le menu de gauche
3. Notez ces deux valeurs (vous en aurez besoin) :
   - **Project URL** : `https://xxxxxxxx.supabase.co`
   - **anon public** (API Key) : `eyJhbG...` (longue chaîne)

### Étape 3 : Créer les tables

1. Dans Supabase, allez dans **SQL Editor** (icône SQL)
2. Cliquez sur "+ New query"
3. Copiez-collez tout le contenu du fichier `supabase-setup.sql`
4. Cliquez sur "Run" (ou Ctrl/Cmd + Enter)
5. ✅ Vérifiez qu'il n'y a pas d'erreurs

### Étape 4 : Activer Realtime

1. Allez dans **Database** > **Replication**
2. Dans la section **Tables**, activez Realtime pour :
   - ✅ `games`
   - ✅ `teams`
   - ✅ `players`
   - ✅ `game_events`
3. Cliquez sur "Save" pour chaque table

### Étape 5 : Vérifier la configuration

Dans le SQL Editor, exécutez :

```sql
SELECT table_name 
FROM information_schema.tables 
WHERE table_schema = 'public';
```

Vous devriez voir : `games`, `teams`, `players`, `game_events`

---

## 📦 Configuration GitHub

### Étape 1 : Initialiser Git (si pas déjà fait)

```bash
cd dice-game
git init
git add .
git commit -m "Initial commit - Dice Game MVP"
```

### Étape 2 : Créer un repository sur GitHub

1. Allez sur [github.com](https://github.com)
2. Cliquez sur le "+" en haut à droite > "New repository"
3. Remplissez :
   - **Repository name** : `dice-game-multiplayer`
   - **Description** : "Jeu de dés multijoueur en temps réel"
   - **Public** ou **Private** (votre choix)
   - ❌ Ne cochez **PAS** "Add a README" (on en a déjà un)
4. Cliquez sur "Create repository"

### Étape 3 : Pusher votre code

```bash
git remote add origin https://github.com/VOTRE_USERNAME/dice-game-multiplayer.git
git branch -M main
git push -u origin main
```

---

## 🌐 Déploiement sur Vercel (Recommandé)

### Pourquoi Vercel ?
- ✅ Détection automatique de Vite
- ✅ Déploiements ultra-rapides
- ✅ SSL gratuit
- ✅ Domaine personnalisé gratuit (.vercel.app)
- ✅ Preview deployments pour les pull requests

### Étapes de déploiement

1. **Créer un compte**
   - Allez sur [vercel.com](https://vercel.com)
   - Cliquez sur "Sign Up"
   - Choisissez "Continue with GitHub"

2. **Importer le projet**
   - Sur le dashboard Vercel, cliquez sur "Add New..." > "Project"
   - Sélectionnez votre repository `dice-game-multiplayer`
   - Cliquez sur "Import"

3. **Configurer le projet**
   - **Framework Preset** : Vite (détecté automatiquement)
   - **Root Directory** : `./` (par défaut)
   - **Build Command** : `npm run build` (par défaut)
   - **Output Directory** : `dist` (par défaut)

4. **Ajouter les variables d'environnement**
   
   Cliquez sur "Environment Variables" et ajoutez :
   
   | Name | Value |
   |------|-------|
   | `VITE_SUPABASE_URL` | `https://xxxxxxxx.supabase.co` |
   | `VITE_SUPABASE_ANON_KEY` | `eyJhbG...` |

   ⚠️ Utilisez les valeurs de l'étape Supabase !

5. **Déployer**
   - Cliquez sur "Deploy"
   - ⏳ Attendez ~2 minutes
   - 🎉 Votre jeu est en ligne !

6. **Récupérer l'URL**
   - Une fois déployé, vous verrez l'URL : `https://votre-projet.vercel.app`
   - Cliquez dessus pour tester !

### Déploiements futurs

Chaque fois que vous pusherez sur GitHub :
```bash
git add .
git commit -m "Votre message"
git push
```

Vercel redéploiera automatiquement ! 🚀

---

## 🌐 Déploiement sur Netlify (Alternative)

### Pourquoi Netlify ?
- ✅ Interface très intuitive
- ✅ Excellent pour les débutants
- ✅ Fonctionnalités de formulaires intégrées
- ✅ SSL gratuit

### Étapes de déploiement

1. **Créer un compte**
   - Allez sur [netlify.com](https://netlify.com)
   - Cliquez sur "Sign up"
   - Choisissez "GitHub"

2. **Importer le projet**
   - Cliquez sur "Add new site" > "Import an existing project"
   - Sélectionnez "Deploy with GitHub"
   - Choisissez votre repository `dice-game-multiplayer`

3. **Configurer le build**
   - **Branch to deploy** : `main`
   - **Build command** : `npm run build`
   - **Publish directory** : `dist`

4. **Ajouter les variables d'environnement**
   
   Avant de déployer, cliquez sur "Show advanced" puis "New variable" :
   
   | Key | Value |
   |-----|-------|
   | `VITE_SUPABASE_URL` | `https://xxxxxxxx.supabase.co` |
   | `VITE_SUPABASE_ANON_KEY` | `eyJhbG...` |

5. **Déployer**
   - Cliquez sur "Deploy site"
   - ⏳ Attendez ~3 minutes
   - 🎉 Votre jeu est en ligne !

6. **Personnaliser l'URL (optionnel)**
   - Allez dans "Site settings" > "Change site name"
   - Choisissez : `dice-game-multiplayer.netlify.app`

---

## ✅ Tests Post-Déploiement

### Test 1 : Accès au site
1. Ouvrez l'URL de votre site
2. ✅ La page d'accueil s'affiche correctement

### Test 2 : Créer une partie
1. Entrez un pseudo
2. Cliquez sur "Créer une partie"
3. ✅ Vous arrivez sur le lobby

### Test 3 : Multi-joueurs
1. Ouvrez l'URL dans un **autre navigateur** ou en **navigation privée**
2. Entrez un autre pseudo
3. ⚠️ **PROBLÈME** : Chaque joueur crée sa propre partie

**Solution** : Vous devez partager le même lien de partie. Voir section suivante.

### Test 4 : Temps réel
1. Dans le lobby, créez une équipe
2. Dans l'autre navigateur, vous devriez voir l'équipe apparaître en temps réel
3. ✅ Le temps réel fonctionne !

---

## 🔗 Partage avec vos amis

### Option 1 : Système actuel (une partie = une URL)

**Problème** : Actuellement, chaque visiteur crée une nouvelle partie.

**Solution temporaire** :
1. Partagez votre écran via Discord/Zoom
2. Tous les joueurs utilisent le même navigateur

### Option 2 : Amélioration rapide - Salons par URL

Modifiez `src/App.jsx` pour utiliser un système de salons :

```javascript
// Ajoutez ceci au début du composant App
const urlParams = new URLSearchParams(window.location.search)
const roomId = urlParams.get('room')

// Dans handleJoinGame, si roomId existe, cherchez la partie au lieu d'en créer une
```

Puis partagez : `https://votre-site.vercel.app?room=party123`

### Option 3 : Système de codes de partie (recommandé pour V2)

Voir le fichier `README.md` section "Améliorations futures"

---

## 🔧 Maintenance

### Voir les logs

**Vercel** :
- Dashboard > Votre projet > "Deployments"
- Cliquez sur un déploiement > "Functions" > Logs

**Netlify** :
- Dashboard > Votre site > "Deploys"
- Cliquez sur un déploiement > "Deploy log"

### Nettoyer les anciennes parties

Connectez-vous à Supabase SQL Editor et exécutez :

```sql
-- Supprimer les parties de plus de 7 jours
DELETE FROM games 
WHERE created_at < NOW() - INTERVAL '7 days';
```

### Surveiller l'usage

**Supabase** :
- Dashboard > Settings > Usage
- Vérifiez : Database size, Bandwidth, Realtime connections

**Vercel/Netlify** :
- Vérifiez les limites du plan gratuit
- Upgrade si nécessaire

### Mettre à jour le code

```bash
# Faire vos modifications
git add .
git commit -m "Ajout de nouvelles fonctionnalités"
git push

# Le déploiement se fait automatiquement !
```

---

## 🎯 Checklist finale

- [ ] Supabase configuré avec toutes les tables
- [ ] Realtime activé sur toutes les tables
- [ ] Code pushé sur GitHub
- [ ] Site déployé sur Vercel ou Netlify
- [ ] Variables d'environnement configurées
- [ ] Tests effectués avec plusieurs navigateurs
- [ ] URL du jeu partagée avec vos amis

---

## 🆘 Problèmes courants

### "Cannot read properties of undefined"
➡️ Vérifiez vos variables d'environnement dans Vercel/Netlify

### Les données ne se synchronisent pas
➡️ Vérifiez que Realtime est activé dans Supabase

### "This site can't be reached"
➡️ Attendez 2-3 minutes après le déploiement

### Les animations sont saccadées
➡️ Normal sur connexions lentes. Optimisable en V2.

---

## 🎉 Félicitations !

Votre jeu multijoueur est maintenant en ligne et accessible à tous !

Prochaines étapes recommandées :
1. Testez avec vos amis
2. Collectez leurs retours
3. Implémentez les améliorations (voir README.md)
4. Partagez votre expérience !

---

**Besoin d'aide ?** Ouvrez une issue sur GitHub ou consultez :
- [Documentation Supabase](https://supabase.com/docs)
- [Documentation Vercel](https://vercel.com/docs)
- [Documentation Netlify](https://docs.netlify.com)
