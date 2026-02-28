---
link: https://slash-root.fr/restic-ou-limportance-des-sauvegardes-de-donnees/
byline: John Johnk
site: slash-root.fr
date: 2024-11-11T13:55
excerpt: Backup is important ! Discover restic to do that efficient
slurped: 2026-02-24T10:27
title: Restic ou l’Importance des Sauvegardes de Données - slash-root.fr
tags:
  - backup
---

Dans un monde où les pertes de données peuvent avoir des conséquences catastrophiques, que ce soit pour des entreprises, des gouvernements, ou des individus, il devient impératif de prendre des mesures pour sécuriser ses informations. Une perte de données peut entraîner des interruptions de services, voire des pertes financières importantes.

Face à cette réalité, la mise en place d’une stratégie de sauvegarde robuste devient essentielle. Dans cet article, nous allons explorer une solution pratique et accessible à tous : Restic. Cet outil, à la fois rapide, fiable et facile à configurer, permet de créer des sauvegardes cryptées et décentralisées, tout en offrant la possibilité de gérer facilement les anciennes versions de fichiers et d'automatiser le processus.

Voyons maintenant comment mettre en place un script de sauvegarde avec Restic, en tenant compte de quelques bonnes pratiques essentielles pour une gestion sécurisée et automatisée des données.

Je commence par définir les variables essentielles de son script, qui permettront de personnaliser la sauvegarde en fonction de mes besoins spécifiques.

```
#!/bin/bash
export RESTIC_PASSWORD='ZZZZZZZZZZZZ'
export RESTIC_REPO='/mnt/restic'
export BACKUP_TARGET='.'
export BACKUP_SOURCE='/mnt/shared/nas'
NUM_KEEP_SNAPSHOTS=15
```

**RESTIC_PASSWORD** : Cette variable contient le mot de passe nécessaire à l'accès au dépôt Restic. Ce mot de passe est utilisé pour chiffrer et déchiffrer les données sauvegardées.

**RESTIC_REPO** : Le répertoire où les sauvegardes seront stockées. Cela peut être un stockage local ou distant (par exemple, un serveur S3 ou un serveur FTP).

**BACKUP_TARGET** : La destination de la sauvegarde.

**BACKUP_SOURCE** : La source des fichiers à sauvegarder, ici /mnt/shared/nas, qui est un répertoire contenant des fichiers critiques.

**NUM_KEEP_SNAPSHOTS** : Le nombre de sauvegardes à conserver. Ce paramètre est essentiel pour éviter l'accumulation excessive de données, ce qui pourrait entraîner un encombrement du système.

L'une des premières étapes de mon script consiste à sauvegarder les bases de données. Étant donné que les bases de données peuvent contenir des informations sensibles et cruciales, il est primordial de m'assurer qu'elles sont bien incluses dans mes sauvegardes.

Cependant, je sais que mes bases de données ne sont enrichies que les jours de semaine, c'est-à-dire du lundi au vendredi. Il n'a donc pas de sens de créer un dump de la base de données pendant le week-end, lorsque les données ne sont pas mises à jour. C'est pourquoi j'ai décidé de ne créer un dump de la base de données que durant les jours ouvrés, afin d'éviter une sauvegarde inutile et de préserver des ressources.

```
DB_HOST="localhost"
DB_PORT="5432"
DB_USER="estx50"
DB_DUMP_PATH="/mnt/shared/nas/Trading/all_databases_backup.sql"

export PGPASSWORD='XXXXXXXXXXXXXX'

DAY_OF_WEEK=$(date +%u)
if [[ "$DAY_OF_WEEK" -ge 1 && "$DAY_OF_WEEK" -le 5 ]]; then
    echo "Jour de semaine - Création du dump de la base de données."
    pg_dumpall -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" > "$DB_DUMP_PATH" || exit 1
else
    echo "Aujourd'hui est le week-end, aucun dump de la base de données ne sera créé."
fi

sleep 10
```

Une fois la base de données sauvegardée, je me tourne vers la sauvegarde de mes fichiers. Restic permet de sauvegarder facilement des répertoires et d'exclure certains fichiers ou dossiers.

J'ai décidé d'exclure certains répertoires spécifiques, comme récupération_Oscar et Trading/data, car ils contiennent des données que je ne souhaite pas inclure dans la sauvegarde, parce qu'elles ne sont pas pertinentes pour mes besoins de sauvegarde.

```
cd "$BACKUP_SOURCE" && restic -r "$RESTIC_REPO" backup --exclude "/mnt/shared/nas/récupération_Oscar" --exclude "/mnt/shared/nas/Trading/data" "$BACKUP_TARGET" --skip-if-unchanged || exit 1
```

> Remarque importante  
> **Vous noterez que les exclude requièrent le chemin absolu.**

`skip-if-unchanged`:

Cette option permet à Restic de vérifier si les fichiers ont changé avant de créer un nouveau snapshot. Si un fichier n'a pas été modifié depuis la dernière sauvegarde, Restic évite de le sauvegarder à nouveau. Cela permet de réduire le volume des données sauvegardées et d'économiser de l'espace disque.

Une fois les sauvegardes effectuées, je dois gérer les anciens snapshots pour éviter que mon dépôt de sauvegarde ne devienne trop encombré. Pour cela, j'utilise la commande forget, qui permet de supprimer les snapshots anciens et inutiles. J'ai choisi de conserver un nombre limité de snapshots récents — ici, 15 — afin de m'assurer que je dispose toujours d'un historique suffisant, sans accumuler trop de données obsolètes.

```
restic -r "$RESTIC_REPO" forget --keep-last "$NUM_KEEP_SNAPSHOTS" --keep-tag preserve --prune || exit 1
```

`--keep-last $NUM_KEEP_SNAPSHOTS` : Cette option permet de définir le nombre de snapshots récents à conserver. Par exemple, si vous gardez 15 snapshots, seuls les 15 derniers seront conservés.

J'utilise également l'option `--prune` pour nettoyer les points de sauvegarde qui ne sont plus nécessaires, ce qui permet de récupérer de l'espace disque et de garder le dépôt de sauvegarde bien organisé.

Enfin dernier point à noter : j'ai utilisé l'option `--keep-tag preserve` dans ma commande de nettoyage. En effet, il y a un point de sauvegarde que je ne veux en aucun cas voir supprimé lors des nettoyages. Pour cela, j'ai préalablement tagué ce snapshot avec la commande `restic tag`, ce qui me permet de l'identifier spécifiquement.

```
restic -r [RESTIC_REPO] tag [ID_SNAPSHOT] preserve
```

La commande restic tag permet d'ajouter un tag personnalisé à un snapshot (dans mon cas je l'ai apppelé preserve), ce qui peut être très utile pour marquer des sauvegardes importantes ou spécifiques. Ainsi, le snapshot marqué avec le tag `preserve` restera dans le dépôt, grâce à `--keep-tag preserve` même lorsque je lance le nettoyage avec l'option `--prune`, qui supprimera tout autre snapshot plus ancien.

**Conclusion** : Un Outil de Sauvegarde Puissant

En appliquant ces pratiques, j'ai réussi à mettre en place une solution de sauvegarde fiable, efficace et parfaitement adaptée à mes besoins. L'automatisation de la sauvegarde de mes fichiers et bases de données, combinée à une gestion rigoureuse des anciens snapshots, me permet de garantir une protection optimale de mes données.

Dans le prochain épisode, je me pencherai sur l'utilisation de Restic pour stocker mes sauvegardes sur un espace S3 Amazon. Ce choix m'offrira encore plus de sécurité et de flexibilité pour mes sauvegardes, tout en ajoutant une couche de fiabilité supplémentaire.