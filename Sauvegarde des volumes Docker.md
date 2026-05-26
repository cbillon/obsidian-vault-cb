---
tags:
  - docker
  - volumes
  - backup
---

Pour comprendre ce phénomène, il faut remonter à une vérité inconfortable : Docker a été conçu pour l'éphémère. Les conteneurs naissent, vivent, meurent. Mais les données qu'ils manipulent, elles, doivent survivre. Et c'est précisément là que le bât blesse.

En 2026, Docker propulse une part considérable des applications en production. Pourtant, quand on interroge les équipes sur leur stratégie de sauvegarde des volumes, la réponse la plus fréquente reste un silence gêné, parfois accompagné d'un vague « on fait des snapshots du serveur ». Les données racontent une autre histoire : les incidents de perte de données sur des environnements conteneurisés restent fréquents, et dans la majorité des cas, l'absence de stratégie de backup des volumes en est la cause directe.

#### Le paradoxe de la persistance dans Docker

Docker distingue trois mécanismes de persistance, et les confondre est la première source d'erreurs.

##### Les bind mounts

Un bind mount projette un répertoire de l'hôte directement dans le conteneur. C'est simple, transparent, et l'hôte garde un accès direct aux fichiers.

```yaml
services:
  nginx:
    image: nginx:alpine
    volumes:
      - /srv/nginx/conf:/etc/nginx/conf.d:ro
      - /srv/nginx/html:/usr/share/nginx/html
```

**Avantages** : visibilité immédiate depuis l'hôte, facilité de sauvegarde avec des outils classiques (rsync, restic), pas de couche d'abstraction.

**Limites** : couplage fort avec le filesystem de l'hôte, portabilité réduite, risque de conflits de permissions entre l'hôte et le conteneur.

##### Les named volumes

Les named volumes sont gérés par Docker lui-même. Les données vivent dans `/var/lib/docker/volumes/` et Docker en contrôle le cycle de vie.

```yaml
services:
  postgres:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

**Avantages** : portabilité, gestion du cycle de vie par Docker, compatibilité avec les drivers de stockage, isolation propre.

**Limites** : les fichiers sont « cachés » dans l'arborescence Docker, ce qui complique la sauvegarde directe. Accéder à `/var/lib/docker/volumes/` depuis l'hôte fonctionne, mais c'est fragile et non recommandé par Docker.

##### Les volumes anonymes

Créés implicitement quand un Dockerfile déclare un `VOLUME` sans que le `docker-compose.yml` ne le monte explicitement. Ce sont les pires : ils portent des noms aléatoires (des hash), sont faciles à oublier lors d'un `docker compose down -v`, et quasi impossibles à identifier après coup.

Nuançons toutefois : les volumes anonymes ont leur utilité pour des données véritablement temporaires (caches, fichiers de session). Mais pour des données de production, c'est un anti-pattern.

#### La méthode naive : `docker run` avec tar

La documentation officielle de Docker propose une approche simple pour sauvegarder un volume :

```bash
docker run --rm \
  -v mon_volume:/data \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/mon_volume_backup.tar.gz -C /data .
```

Cette commande crée un conteneur temporaire, monte le volume cible et un répertoire de backup, puis archive le tout. Pour la restauration :

```bash
docker run --rm \
  -v mon_volume:/data \
  -v $(pwd):/backup \
  ubuntu bash -c "cd /data && tar xzf /backup/mon_volume_backup.tar.gz"
```

Nuançons toutefois : cette approche fonctionne pour des volumes simples et des sauvegardes ponctuelles. Mais elle présente des failles sérieuses en production :

- **Pas de cohérence garantie** : si une base de données écrit pendant l'archivage, le backup peut être corrompu
- **Pas d'incrémental** : chaque sauvegarde est complète, ce qui consomme espace et temps
- **Pas de chiffrement** : les archives transitent et sont stockées en clair
- **Pas de rétention** : la gestion des anciennes sauvegardes est manuelle

Pour comprendre pourquoi ces limitations sont critiques, il faut regarder ce que font les outils spécialisés.

#### Les outils qui changent la donne

##### Restic : le choix pragmatique

Restic est devenu un standard de fait pour la sauvegarde de serveurs Linux, et il s'adapte remarquablement bien aux volumes Docker. Son architecture repose sur la déduplication par blocs et le chiffrement AES-256-CTR avec authentification Poly1305-AES de bout en bout.

```bash
# Initialiser un dépôt de sauvegarde
restic -r sftp:backup-server:/backups/docker init

# Sauvegarder un volume via un conteneur temporaire
docker run --rm \
  -v mon_volume:/data:ro \
  -v /root/.config/restic:/root/.config/restic:ro \
  restic/restic \
  -r sftp:backup-server:/backups/docker \
  backup /data --tag mon_volume

# Politique de rétention
restic -r sftp:backup-server:/backups/docker forget \
  --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
```

Pour un guide complet sur restic, notre article [Restic : guide de la sauvegarde moderne](https://www.shpv.fr/blog/restic-sauvegarde/) couvre l'installation et la configuration en détail. Et pour combiner restic avec un stockage distant S3, consultez notre guide [Restic avec rclone et S3](https://www.shpv.fr/blog/restic-rclone-s3/).

##### Duplicity : le vétéran

Duplicity existe depuis plus de 20 ans et reste utile. Son approche par sauvegardes incrémentales basées sur librsync le rend efficace sur les connexions à faible bande passante.

```bash
# Sauvegarde incrémentale chiffrée vers S3
duplicity \
  --encrypt-key VOTRE_CLE_GPG \
  /var/lib/docker/volumes/mon_volume/_data \
  s3://s3.amazonaws.com/mon-bucket/docker-backups/mon_volume
```

Les données racontent une autre histoire que celle du « vieux logiciel dépassé » : duplicity gère nativement le chiffrement GPG, supporte une vingtaine de backends de stockage, et sa maturité signifie moins de bugs en production.

##### Volumerize et docker-volume-backup : l'approche conteneurisée

Pour ceux qui préfèrent rester dans l'écosystème Docker, des projets comme `docker-volume-backup` automatisent le processus :

```yaml
services:
  backup:
    image: offen/docker-volume-backup:v2
    environment:
      BACKUP_CRON_EXPRESSION: '0 2 * * *'
      BACKUP_FILENAME: 'backup-%Y-%m-%dT%H-%M-%S.tar.gz'
      AWS_S3_BUCKET_NAME: mon-bucket-backup
      AWS_ACCESS_KEY_ID: ${AWS_KEY}
      AWS_SECRET_ACCESS_KEY: ${AWS_SECRET}
    volumes:
      - pgdata:/backup/pgdata:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
```

L'avantage : tout reste déclaratif dans votre `docker-compose.yml`. L'inconvénient : vous ajoutez un conteneur qui a accès au socket Docker, ce qui élargit la surface d'attaque. Notre article sur le [hardening Docker](https://www.shpv.fr/blog/hardening-docker/) explique pourquoi ce point mérite attention.

#### Le cas des bases de données : ne sauvegardez pas le volume

C'est le piège classique. Pour PostgreSQL, MySQL, MongoDB ou Redis, sauvegarder le volume brut sans arrêter le conteneur, c'est jouer à la roulette russe avec la cohérence des données.

##### La bonne approche : dump applicatif puis sauvegarde

```bash
# PostgreSQL : dump cohérent
docker exec postgres pg_dumpall -U postgres > /backup/pg_dump_$(date +%F).sql

# MySQL/MariaDB : dump cohérent
docker exec mysql mysqldump -u root -p"$MYSQL_ROOT_PASSWORD" --all-databases \
  --single-transaction > /backup/mysql_dump_$(date +%F).sql

# Redis : déclencher un snapshot
docker exec redis redis-cli BGSAVE
# Puis sauvegarder le fichier dump.rdb du volume
```

La séquence en production devrait être :

1. **Dump applicatif** (pg_dump, mysqldump, mongodump)
2. **Sauvegarde du dump** avec restic ou duplicity
3. **Sauvegarde du volume** en complément (pour la configuration, les logs, etc.)
4. **Vérification de restauration** (le backup que vous ne testez pas n'existe pas)

Pour approfondir les sauvegardes PostgreSQL spécifiquement, notre article [BorgBackup et PostgreSQL](https://www.shpv.fr/blog/borgbackup-postgresql/) détaille les stratégies WAL et les restaurations point-in-time.

#### La règle 3-2-1 appliquée aux conteneurs

La règle 3-2-1 (3 copies, 2 supports différents, 1 copie hors site) est un classique de la sauvegarde. Mais comment l'appliquer concrètement à un environnement Docker ?

##### Copie 1 : le volume lui-même

C'est votre donnée de production. Elle existe déjà, mais ne constitue pas une sauvegarde.

##### Copie 2 : sauvegarde locale sur un stockage différent

Un dépôt restic sur un disque ou une partition séparé du stockage Docker. Si votre volume est sur un SSD NVMe, mettez vos backups sur un HDD local ou un NAS.

```bash
# Sauvegarde locale sur un point de montage différent
restic -r /mnt/backup-disk/restic-repo backup /var/lib/docker/volumes/
```

##### Copie 3 : réplication hors site

C'est la copie qui vous sauve quand le serveur brûle, quand le ransomware chiffre tout, ou quand un `docker compose down -v` un peu trop enthousiaste efface vos données.

```bash
# Synchronisation vers un stockage S3 distant
restic -r s3:s3.amazonaws.com/mon-bucket-backup backup /var/lib/docker/volumes/
```

Pour une vision complète de la stratégie 3-2-1, notre article [La stratégie de sauvegarde 3-2-1](https://www.shpv.fr/blog/strategie-sauvegarde-321/) couvre les principes fondamentaux, et [Backup avec rsync et la règle 3-2-1](https://www.shpv.fr/blog/backup-rsync-321/) propose une implémentation avec rsync.

#### Automatisation : le script qui tourne à 3h du matin

En production, la sauvegarde manuelle n'existe pas. Voici un squelette de script qui combine dump applicatif et sauvegarde restic :

```bash
#!/bin/bash
set -euo pipefail

RESTIC_REPOSITORY="sftp:backup-srv:/backups/docker-prod"
RESTIC_PASSWORD_FILE="/root/.restic-password"
BACKUP_DIR="/tmp/docker-backups"
TIMESTAMP=$(date +%F_%H-%M)

export RESTIC_REPOSITORY RESTIC_PASSWORD_FILE

mkdir -p "$BACKUP_DIR"

# 1. Dumps applicatifs
docker exec postgres pg_dumpall -U postgres \
  > "$BACKUP_DIR/postgres_${TIMESTAMP}.sql"

docker exec redis redis-cli BGSAVE
sleep 5
docker cp redis:/data/dump.rdb "$BACKUP_DIR/redis_${TIMESTAMP}.rdb"

# 2. Sauvegarde des volumes et des dumps
restic backup \
  /var/lib/docker/volumes/app_pgdata \
  /var/lib/docker/volumes/app_redis \
  "$BACKUP_DIR" \
  --tag docker-prod --tag "$TIMESTAMP"

# 3. Rétention
restic forget \
  --keep-daily 7 \
  --keep-weekly 4 \
  --keep-monthly 12 \
  --prune

# 4. Nettoyage
rm -f "$BACKUP_DIR"/*

# 5. Vérification (une fois par semaine)
if [ "$(date +%u)" = "1" ]; then
  restic check
fi
```

Planifiez ce script via cron ou un timer systemd. Et surtout, **monitorez les échecs** : un backup en erreur silencieuse, c'est pire que pas de backup du tout (vous pensez être protégé alors que vous ne l'êtes pas).

#### Les erreurs que tout le monde fait

Pour comprendre ce phénomène, il faut admettre que la sauvegarde Docker est un domaine où les mauvaises pratiques se transmettent d'article en article :

1. **Sauvegarder `/var/lib/docker/` en entier** : c'est tentant, mais vous sauvegardez aussi les layers d'images, les réseaux, les conteneurs arrêtés. C'est volumineux et inutile. Sauvegardez uniquement `/var/lib/docker/volumes/`.
    
2. **Oublier `docker compose down -v`** : le flag `-v` supprime les volumes. Un classique du "je voulais juste relancer mes conteneurs". La solution : ne jamais utiliser `-v` en production, ou protéger vos volumes avec des labels et des scripts de vérification.
    
3. **Ne pas tester la restauration** : 47 % des sauvegardes ne se restaurent pas correctement selon les études du secteur. Planifiez des tests de restauration mensuels.
    
4. **Ignorer les volumes anonymes** : un `docker volume ls` révèle souvent des volumes orphelins contenant des données critiques que personne ne sauvegarde.
    
5. **Sauvegarder une base de données "à chaud" sans dump** : le fichier de données d'une base active n'est pas cohérent. Sans dump applicatif préalable, votre backup est potentiellement inutilisable.
    

#### Quelle stratégie pour quel contexte ?

| Contexte                      | Stratégie recommandée                                     |
| ----------------------------- | --------------------------------------------------------- |
| Dev/staging                   | Bind mounts + rsync quotidien                             |
| Production simple (1 serveur) | Named volumes + restic + S3                               |
| Production multi-serveurs     | Dumps applicatifs + restic + réplication cross-site       |
| Bases de données critiques    | WAL archiving (PG) / binlog (MySQL) + dumps + restic      |
| Conformité réglementaire      | Chiffrement bout en bout + rétention longue + audit trail |

Chez SHPV, nos offres d'[infogérance](https://www.shpv.fr/services/infogerance/) incluent la mise en place de ces stratégies de sauvegarde adaptées au contexte de chaque client. Pour les environnements Kubernetes, notre article sur [Velero pour le backup Kubernetes](https://www.shpv.fr/blog/velero-k8s-backup/) couvre les spécificités de l'orchestrateur.

#### Monitoring et alerting : savoir quand ça casse

Une stratégie de sauvegarde sans monitoring, c'est comme une assurance dont vous ne vérifiez jamais les conditions. En production, chaque exécution de backup doit être tracée :

- **Code de retour** : un `restic backup` qui échoue silencieusement est pire qu'un backup absent
- **Durée d'exécution** : une sauvegarde qui prend 4 heures au lieu de 20 minutes signale un problème (volume qui grossit, réseau saturé, disque lent)
- **Taille du snapshot** : une variation brutale (trop petit ou trop gros) mérite investigation
- **Fraîcheur** : alertez si le dernier backup réussi date de plus de 24 heures

Un simple webhook vers votre outil de monitoring (Prometheus, Grafana, ou même un canal Slack) suffit pour commencer. L'important est de ne jamais découvrir un échec de backup le jour où vous en avez besoin.

#### Vers une sauvegarde mature

Nuançons toutefois : il n'existe pas de solution universelle. Le bon choix dépend de votre contexte, de votre budget, de vos contraintes réglementaires et de votre tolérance à la perte de données (RPO) et au temps d'indisponibilité (RTO).

Ce qui est universel, en revanche, c'est la nécessité d'avoir une stratégie. Un volume Docker sans backup, c'est une donnée en sursis. Et les données racontent toujours la même histoire : ce n'est pas une question de "si" vous perdrez des données, mais de "quand".

Commencez simple : un script restic, un stockage distant, un test de restauration mensuel. Puis itérez. La perfection n'est pas l'objectif ; la résilience, si.

#### Sources

- [Volumes, documentation officielle Docker](https://docs.docker.com/engine/storage/volumes/)
- [Backup Docker volumes done right, Augmented Mind](https://www.augmentedmind.de/2023/08/20/backup-docker-volumes/)
- [Duplicacy vs Restic vs Borg: Which Backup Tool is Right in 2025, MangoHost](https://mangohost.net/blog/duplicacy-vs-restic-vs-borg-which-backup-tool-is-right-in-2025/)
- 