---
tags:
  - backup
  - rsync
---
[](https://www.shpv.fr/blog/backup-rsync-321/)

Vous avez des données importantes sur un serveur Linux. Vous pensez être couvert parce que vous faites un `cp` de temps en temps, ou un `tar` une fois par semaine. Et puis un jour, le disque lâche. Ou pire, un ransomware chiffre tout, backup inclus, parce que votre backup était sur le même volume. La stratégie 3-2-1, c'est la réponse à ça. Trois copies, deux supports différents, une copie hors site. Rsync est votre outil. Ce guide vous montre comment la déployer, du plus simple au plus éprouvé.

#### Prérequis

- Linux : Debian, Ubuntu, RHEL, Rocky
- `rsync` installé (`apt install rsync` ou `yum install rsync`)
- Accès SSH vers un serveur distant (pour la copie offsite)
- Un support externe ou un second disque (pour la copie locale)
- Optionnel : un compte cloud compatible (Backblaze B2, Wasabi) pour l'offsite

#### La stratégie 3-2-1 : pourquoi ça marche

La règle est simple. Trois copies de vos données. Sur deux types de supports différents. Et une copie hors du même lieu physique.

Par exemple : une copie sur votre disque principal, une sur un disque externe branché au même serveur, une sur un serveur distant via SSH. Si le datacenter brûle, vous avez encore la copie distante. Si le ransomware frappe, il ne peut pas atteindre le serveur SSH distant. C'est la couverture minimale qu'on doit avoir en production.

#### Rsync : les options qui comptent

Avant d'écrire un script, il faut maîtriser les flags. Pas tous, juste celles qui changent la donne.

```bash
rsync -avzP --delete source/ destination/
```

- `-a` : mode archive, préserve les permissions, timestamps, liens symboliques, propriétaires
- `-v` : verbose, vous voyez ce qui se passe
- `-z` : compression en transit, essentiel pour les sauvegardes distantes
- `-P` : reprend les transferts interrompus + affiche la progression
- `--delete` : supprime dans la destination ce qui n'existe plus dans la source, votre destination reste un miroir exact

```bash
# Exclure des fichiers ou répertoires
rsync -avzP --delete --exclude='*.tmp' --exclude='/var/log/*' source/ dest/

# Limiter la bande passante (utile pour l'offsite)
rsync -avzP --delete --bwlimit=500K source/ dest/

# Dry run — voir ce qui va être fait sans toucher à quoi que ce soit
rsync -avzP --delete -n source/ dest/
```

**Attention** : le slash entre `source/` et `source` fait une énorme différence. `source/` copie le contenu du répertoire. `source` copie le répertoire lui-même.

#### Copie 1 : backup local sur un second disque

C'est votre première ligne de défense. Rapide, pas besoin de réseau.

```bash
#!/usr/bin/env bash
set -euo pipefail

# ─── Configuration ───────────────────────────────────────────
readonly SOURCE="/var/www"
readonly DEST="/mnt/backup-local/www"
readonly LOG="/var/log/backup-local.log"
readonly TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

log()   { echo "[$TIMESTAMP] [INFO]  $*" | tee -a "$LOG"; }
error() { echo "[$TIMESTAMP] [ERROR] $*" | tee -a "$LOG" >&2; }

# ─── Vérifications ──────────────────────────────────────────
main() {
    # Le disque backup monté ?
    if ! mountpoint -q /mnt/backup-local; then
        error "Le disque backup n'est pas monté sur /mnt/backup-local"
        exit 1
    fi

    mkdir -p "$DEST"

    log "Backup local démarré : $SOURCE → $DEST"
    if rsync -avzP --delete \
        --exclude='*.tmp' \
        --exclude='*.log' \
        --log-file="$LOG" \
        "$SOURCE/" "$DEST/"; then
        log "Backup local OK"
    else
        error "Backup local échoué"
        exit 1
    fi
}

main "$@"
```

#### Copie 2 : backup vers un serveur distant via SSH

C'est votre copie offsite. Elle sort physiquement de votre datacenter.

```bash
#!/usr/bin/env bash
set -euo pipefail

# ─── Configuration ───────────────────────────────────────────
readonly SOURCE="/var/www"
readonly REMOTE_USER="backup"
readonly REMOTE_HOST="backup.exemple.com"
readonly REMOTE_DEST="/var/backups/www"
readonly LOG="/var/log/backup-distant.log"
readonly TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
readonly BWLIMIT="2M"  # 2 Mo/s max

log()   { echo "[$TIMESTAMP] [INFO]  $*" | tee -a "$LOG"; }
error() { echo "[$TIMESTAMP] [ERROR] $*" | tee -a "$LOG" >&2; }

main() {
    log "Backup distant démarré : $SOURCE → ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DEST}"

    if rsync -avzP --delete \
        --bwlimit="$BWLIMIT" \
        --exclude='*.tmp' \
        --exclude='*.log' \
        -e "ssh -i /root/.ssh/id_ed25519_backup -o StrictHostKeyChecking=yes" \
        --log-file="$LOG" \
        "$SOURCE/" "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DEST}/"; then
        log "Backup distant OK"
    else
        error "Backup distant échoué"
        exit 1
    fi
}

main "$@"
```

##### Configurer l'utilisateur backup côté serveur distant

L'utilisateur `backup` ne doit avoir qu'un accès minimal, pas de shell, juste rsync.

```bash
# Sur le serveur distant
sudo useradd -r -s /usr/sbin/nologin backup
sudo mkdir -p /var/backups/www
sudo chown backup:backup /var/backups/www

# Copier la clé SSH publique
sudo mkdir -p /home/backup/.ssh
sudo cp /chemin/vers/id_ed25519_backup.pub /home/backup/.ssh/authorized_keys
sudo chown -R backup:backup /home/backup/.ssh
sudo chmod 700 /home/backup/.ssh
sudo chmod 600 /home/backup/.ssh/authorized_keys
```

#### Copie 3 : backup vers le cloud (offsite ultime)

Si vous voulez une troisième couche vraiment indépendante, Backblaze B2 est le meilleur rapport prix/fiabilité. Rclone fait le job.

```bash
# Installation de rclone
curl https://rclone.org/install.sh | sudo bash

# Configuration pour Backblaze B2
rclone config
# → Suivez l'assistant : choisissez "b2", entrez votre Application Key ID + Key

# Sync vers B2
rclone sync /var/backups/local b2:votre-bucket/backups --log-level INFO
```

```bash
# Script d'automatisation
#!/usr/bin/env bash
set -euo pipefail

readonly SOURCE="/var/backups/local"
readonly DEST="b2:votre-bucket/backups"
readonly LOG="/var/log/backup-cloud.log"

log()   { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO]  $*" | tee -a "$LOG"; }
error() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [ERROR] $*" | tee -a "$LOG" >&2; }

main() {
    log "Sync cloud démarré"
    if rclone sync "$SOURCE" "$DEST" --log-level INFO --log-file="$LOG"; then
        log "Sync cloud OK"
    else
        error "Sync cloud échoué"
        exit 1
    fi
}

main "$@"
```

#### Backup d'une base de données : dump avant rsync

Rsync copie des fichiers. Une base de données en cours d'écriture, vous ne la copiez pas comme ça, vous la dump d'abord.

```bash
#!/usr/bin/env bash
set -euo pipefail

readonly DB_NAME="myapp"
readonly DB_USER="backup_user"
readonly DUMP_DIR="/var/backups/db"
readonly RETENTION_DAYS=7
readonly TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
readonly LOG="/var/log/backup-db.log"

log()   { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO]  $*" | tee -a "$LOG"; }
error() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [ERROR] $*" | tee -a "$LOG" >&2; }

main() {
    mkdir -p "$DUMP_DIR"
    local dump_file="${DUMP_DIR}/${DB_NAME}_${TIMESTAMP}.sql.gz"

    log "Dump PostgreSQL : $DB_NAME"
    if pg_dump -U "$DB_USER" -d "$DB_NAME" -Fc -Z 6 -f "$dump_file"; then
        log "Dump OK : $dump_file ($(du -h "$dump_file" | cut -f1))"
    else
        error "Dump échoué pour $DB_NAME"
        exit 1
    fi

    # Vérification intégrité du dump
    if ! pg_restore --list "$dump_file" >/dev/null 2>&1; then
        error "Dump corrompu : $dump_file"
        rm -f "$dump_file"
        exit 1
    fi
    log "Intégrité vérifié OK"

    # Nettoyage ancien dumps
    find "$DUMP_DIR" -name "*.sql.gz" -mtime +"$RETENTION_DAYS" -delete
    log "Nettoyage terminé"

    # Le dump est maintenant dans /var/backups/db — rsync le copiera automatiquement
}

main "$@"
```

#### Automatisation avec cron

```bash
# /etc/cron.d/backup-321

# Backup local — toutes les 6 heures
0 */6 * * * root /opt/scripts/backup-local.sh

# Dump DB — quotidien à 1h
0 1 * * * root /opt/scripts/backup-db.sh

# Backup distant — quotidien à 2h (après le dump)
0 2 * * * root /opt/scripts/backup-distant.sh

# Sync cloud — quotidien à 3h
0 3 * * * root /opt/scripts/backup-cloud.sh
```

#### Restauration : tester avant que ça arrive

Un backup qui n'a jamais été testé n'existe pas vraiment. Testez régulièrement.

```bash
# Restaurer depuis le backup local vers un répertoire temporaire
rsync -avP /mnt/backup-local/www/ /tmp/restore-test/

# Vérifier que les fichiers sont intègres
diff -rq /var/www /tmp/restore-test/

# Restaurer un dump PostgreSQL
pg_restore -U postgres -d myapp_test --clean /var/backups/db/myapp_20260203_020000.sql.gz

# Nettoyer
rm -rf /tmp/restore-test
```

#### Tableau récapitulatif, votre 3-2-1

|Copie|Destination|Fréquence|Outil|Rétention|
|---|---|---|---|---|
|1|Disque local secondaire|Toutes les 6h|rsync|Miroir live|
|2|Serveur SSH distant|Quotidien (2h)|rsync over SSH|30 jours|
|3|Cloud (B2/Wasabi)|Quotidien (3h)|rclone|90 jours|

#### Pièges classiques

Le piège numéro un, c'est de faire tous ses backups sur le même disque que la source. Un crash, et tout part ensemble. Le deuxième, c'est d'oublier `--delete` - votre destination gonfle indéfiniment avec des fichiers qui n'existent plus. Et le troisième, le plus dangereux : ne jamais tester la restauration. Le jour où vous en avez besoin, c'est déjà trop tard pour découvrir que le dump était corrompu depuis des semaines.

#### Aller plus loin

Comprenez la théorie derrière la stratégie 3-2-1 avec notre article sur [comprendre la stratégie 3-2-1](https://www.shpv.fr/blog/strategie-sauvegarde-321/). Pour une alternative moderne, découvrez [Restic comme alternative à rsync](https://www.shpv.fr/blog/restic-sauvegarde/). Pour chiffrer vos backups, consultez [LUKS](https://www.shpv.fr/blog/luks-chiffrement/).

#### Conclusion

La stratégie 3-2-1 n'est pas un luxe, c'est un minimum. Rsync vous donne toute la puissance nécessaire pour la déployer en moins d'une heure. Automatisez, loggez, testez régulièrement. La prochaine étape logique après ça ? Chiffrer vos backups avec LUKS avant de les envoyer hors site. Mais ça fait un bon article à part.