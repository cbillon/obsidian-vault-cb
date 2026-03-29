---
link: https://www.metal3d.org/blog/2026/git-worktree-comme-un-chef/
site: Metal3d
date: 2026-03-19T07:50
excerpt: |-
  S'il y a bien un outil Git que peu de gens connaissent, c'est « worktree ». Une fois que vous l'aurez maîtrisé, 
  vous aurez un mal fou à vous en passer. Mais encore faut-il l'utiliser de la bonne manière. Voici la méthode que 
  j'utilise depuis très longtemps.
tags:
  - git
slurped: 2026-03-28T10:04
title: Git Worktree Comme Un Chef
---

S'il y a bien un outil Git que peu de gens connaissent, c'est « worktree ». Une fois que vous l'aurez maîtrisé, vous aurez un mal fou à vous en passer. Mais encore faut-il l'utiliser de la bonne manière. Voici la méthode que j'utilise depuis très longtemps.

Les “worktrees”, ou arbres de travail, vont vous permettre de travailler sur différentes branches sans passer par des checkout en pagaille. Vous n’aurez plus à vous demander pourquoi ce fichier n’est pas ajouté et depuis quelle branche vous l’avez engendré, et vous allez enfin pouvoir copier des fichiers d’une branche à l’autre sans vous énerver avec les commandes à rallonge. Bref, ça va vous plaire.

## TL;DR (Pour les pressés)

Dans un dossier vide :

```
# 1. Cloner le repo dans un dossier caché .bare
git clone --bare git@github.com:user/repo.git .bare

# 2. Indiquer au dossier racine où se cache l'historique Git
echo "gitdir: ./.bare" > .git

# 3. Corriger la config de fetch pour voir toutes les branches distantes
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"

# 4. Créer votre premier worktree (la branche main)
git worktree add main
```

_Chaque dossier correspond à une branche, tout est centralisé. Lisez la suite pour comprendre pourquoi c’est révolutionnaire._

## Avant tout, c’est quoi un « worktree » ?

Dans le monde du développement, le changement de contexte (_context switching_) est une réalité quotidienne. Vous connaissez la situation : vous êtes en plein milieu d’une branche complexe quand un bug critique en production exige votre attention immédiate.

Ou parfois, vous devez gérer deux branches en même temps pour une grosse migration ou un refactoring complexe. Vous avez donc besoin de plusieurs “arbres de travail” directs.

Traditionnellement, on utilise `git stash` pour nettoyer l’espace de travail, ou on clone carrément le dépôt dans un autre dossier. Mais ces méthodes sont soit lourdes, soit sources d’erreurs, soit gourmandes en espace disque.

C’est là qu’intervient **Git Worktree**. Introduit avec la version 2.5, cet outil permet d’avoir plusieurs répertoires de travail attachés à un seul dépôt. Au lieu d’avoir une seule branche extraite à la fois, vous pouvez en avoir plusieurs simultanément dans des dossiers différents. Tous ces dossiers partagent le même historique `.git`.

## La “mauvaise” façon de faire

On voit souvent des articles expliquant qu’on peut faire un worktree avec `../nom-du-worktree`. Ça fonctionne, mais ce n’est pas ce que je conseille. C’est souvent une solution “rapide” quand on a déjà un clone classique, mais on se retrouve vite avec des dossiers éparpillés.

## Faire mieux : la méthode “Bare”

Ma méthode consiste à utiliser un clone **“bare”** (nu). Ce type de clone ne contient pas de fichiers de travail, uniquement l’historique.

```
mkdir -p ~/Projects/MonProjet
cd ~/Projects/MonProjet
git clone --bare git@github.com:user/repo.git .bare
```

Dans le dossier `.bare`, vous verrez la structure interne de Git (config, objets, refs…). Pour que le dossier racine se comporte comme un dépôt Git sans être encombré, voici le “magic trick” :

```
echo "gitdir: ./.bare" > .git
```

Par défaut, si vous êtes à la racine, mais hors d’un worktree, Git ne sait pas où est l’historique. Ce fichier `.git` d’une ligne dit à votre terminal : « Considère tout ce répertoire comme faisant partie du projet Git situé dans `./.bare` ».

## Et maintenant, la magie

```
# J'explique cette ligne juste après
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"

# Commençons à travailler sur différentes branches
git worktree add main
git worktree add feature/ma-super-feature
```

Cela génère la structure suivante :

```
.
├── feature/
│   └── ma-super-feature/
└── main/
```

Chaque dossier contient le contenu de la branche souhaitée. Vous pouvez travailler dans les deux, faire des `push`, des `pull`, etc.

## Le piège du “Clone Aveugle”

Si vous avez suivi les étapes, vous pourriez être surpris : en essayant d’ajouter un worktree pour une branche distante existante, Git crée parfois une nouvelle branche vide à partir de `main`.

Si `git fetch` ne vous montre que :

```
* branch HEAD -> FETCH_HEAD` 
```

> Vous êtes officiellement “aveugle”.

Quand on utilise `git clone --bare`, Git suppose que vous voulez un backup ou un miroir serveur, pas un espace de travail actif. Il ne configure donc pas le “Remote Tracking”. Pour corriger ça et voir tout ce qui se passe sur le serveur :

```
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
```

Refaites un `git fetch --all` et toutes les branches de vos collègues apparaîtront.

## Quelques conseils de survie (Le mode “Boss”)

Travailler avec des worktrees change radicalement la donne. Voici quelques règles d’or :

### 1. Ne faites PLUS JAMAIS de `git checkout`

C’est le piège numéro un. Dans un setup worktree, **un dossier = une branche**. Si vous faites un `git checkout une-autre-branche` à l’intérieur d’un répertoire existant, vous cassez cette logique et vous allez vous mélanger les pinceaux.

- **Pourquoi ?** Parce que l’intérêt est de garder vos environnements (dépendances, build, cache) isolés. Si vous changez de branche dans le même dossier, vous perdez cet avantage et Git vous bloquera si la branche est déjà ouverte ailleurs.

Comme tous vos dossiers partagent la même base de données `.git` (le dossier `.bare`), vous n’avez pas besoin de faire un `git pull` dans chaque dossier.

- Faites simplement un `git fetch --all` depuis n’importe quel dossier.
- Toutes vos branches locales et distantes seront mises à jour dans la base commune. Il ne vous restera qu’à faire un `git merge` ou `git rebase` rapide dans le dossier concerné pour avancer votre pointeur local.

### 3. Tester un merge sans polluer ses branches

Vous voulez tester le merge de `feature-A` dans `develop` sans risquer de salir votre dossier `develop` de travail ? Utilisez l’option `--detach` :

```
# On extrait l'état de 'develop' dans un dossier temporaire sans créer de branche
git worktree add --detach test-merge develop
cd test-merge
git merge feature-A
# Lancez vos tests...
```

Une fois fini, supprimez le dossier et lancez `git worktree prune`. Comme vous étiez en mode “détaché”, aucune branche locale inutile n’aura été créée.

> **Note sur le nettoyage :** Le `git worktree prune` nettoie les liens internes de Git, mais il ne supprime jamais vos branches locales. Si vous avez créé une branche dédiée pour un worktree, vous devrez toujours la supprimer manuellement avec `git branch -d` si elle ne vous sert plus. Le mode “detach” vous permet de vous abstraire de cela, aucune branche n’est créée.

## Pourquoi ne pas simplement faire deux clones ?

Comparer les Worktrees à plusieurs clones, c’est comme comparer un système audio multi-room synchronisé à deux chaînes hifi séparées qu’on essaierait de lancer en même temps.

1. **Le poids sur le disque** : Si votre repo fait 500 Mo, deux clones occupent 1 Go. Avec les Worktrees, seul le dossier `.bare` pèse lourd. Les dossiers de travail ne contiennent que vos fichiers sources.
2. **La source de vérité unique** : C’est le vrai “game-changer”. Si vous faites un `git fetch` dans le dossier A, le dossier B est instantanément au courant. Une branche locale créée dans l’un est visible dans l’autre. Un seul cerveau, plusieurs corps.
3. **Hooks et Configs** : Vos alias, vos hooks Git et vos configurations locales sont universels. Pas besoin de les redéfinir pour chaque clone.
4. **La sécurité** : Git vous empêche d’extraire la même branche dans deux dossiers différents. Cela évite d’écraser votre propre travail par mégarde.

## Quelques astuces pour la route

### Supprimer un worktree

Ne vous contentez pas d’un `rm -rf`. Git gardera une trace du worktree en interne. Si vous essayez de le recréer, il dira qu’il existe déjà. Pour nettoyer proprement :

```
git worktree prune
```

### Le script magique

Pour automatiser tout ça, j’utilise un petit script nommé `wtree` dans mon dossier `~/.local/bin` :

```
#!/bin/bash
# Usage: wtree <git url> (dans un dossier vide)
REPO_URL=$1
SCRIPT_NAME=$(basename "$0")

if [ -z "$REPO_URL" ]; then
  echo "❌ Erreur : URL du dépôt manquante."
  echo "Usage: $0 <repo-url>"
  exit 1
fi

# 1. Vérifier si le dossier courant est vide
# On autorise la présence du script lui-même
FILES_IN_DIR=$(ls -A | grep -v "$SCRIPT_NAME")
if [ ! -z "$FILES_IN_DIR" ]; then
  echo "❌ Erreur : Le dossier courant n'est pas vide !"
  echo "Pour garder votre configuration worktree propre, merci de lancer ceci dans un dossier frais."
  exit 1
fi

echo "🚀 Initialisation du setup Pro Git Worktree..."

# 2. Cloner en bare dans un dossier caché
git clone --bare "$REPO_URL" .bare

# 3. Lier la racine au dépôt bare
echo "gitdir: ./.bare" > .git

# 4. Activer le remote tracking (Le fix "Remote Awareness")
# Cela assure que 'git fetch' voit toutes les branches du serveur
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"

# 5. Tout fetcher
git fetch --all

echo "✅ Setup terminé !"
echo "Prochaine étape : git worktree add main"


```

## Conclusion : Essayez pendant une semaine

Si vous avez l’habitude du `git stash` ou des multiples clones, ce workflow peut sembler un peu “over-engineered”. Mais je vous lance un défi : essayez-le sur votre prochain gros projet durant une semaine.

Le moment où vous ferez tourner une suite de tests lourds dans un dossier tout en corrigeant un bug urgent dans un autre — sans jamais perdre votre “flow” — vous aurez le déclic. On ne revient jamais en arrière.

Cet article existe en anglais ici: [Git workflow like a boss sur Dev.to](https://dev.to/metal3d/git-worktree-like-a-boss-2j1b)