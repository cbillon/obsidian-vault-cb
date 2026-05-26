---
tags:
  - bash
---
#### Prérequis

- Linux (Debian, Ubuntu, RHEL, Rocky, peu importe)
- Bash 4.x minimum (`bash --version`)
- Accès root ou sudo sur au moins un serveur
- Connaissance de base des commandes : `ls`, `grep`, `awk`, `cp`, `ssh`

#### Les fondamentaux que tout le monde oublie

Avant d'écrire un script qui va tourner en production, il y a des habitudes qui séparent les scripts qui survivent de ceux qui cassent silencieusement.

##### Shebang et mode strict

```bash
#!/usr/bin/env bash
set -euo pipefail
```

- `set -e` : le script s'arrête à la première erreur
- `set -u` : erreur si une variable est utilisée sans être définie
- `set -o pipefail` : un pipeline échoue si n'importe quelle commande dans le pipe échoue

Sans ces trois lignes, vos scripts vont échouer silencieusement en production. C'est la cause numéro un des incidents.

##### Logging structuré

```bash
# Fonctions de logging réutilisables
readonly SCRIPT_NAME=$(basename "$0")
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"

log()   { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO]  $*" | tee -a "$LOG_FILE"; }
warn()  { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [WARN]  $*" | tee -a "$LOG_FILE" >&2; }
error() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [ERROR] $*" | tee -a "$LOG_FILE" >&2; }

# Usage
log "Script démarré"
warn "Espace disque sous 20%"
error "Impossible de se connecter à la base de données"
```

Chaque script de production doit logger. Pas dans stdout uniquement, dans un fichier, avec un timestamp, avec un niveau. Pour aller plus loin, consultez le [guide complet sur la gestion des logs Linux](https://www.shpv.fr/blog/logs-linux-2026/).

##### Gestion des variables et des arguments

```bash
# Valeurs par défaut
RETENTION_DAYS="${RETENTION_DAYS:-7}"
BACKUP_DIR="${BACKUP_DIR:-/var/backups}"

# Validation
if [[ ! -d "$BACKUP_DIR" ]]; then
    error "Le répertoire de backup n'existe pas : $BACKUP_DIR"
    exit 1
fi

# Arguments positionnels avec aide
usage() {
    echo "Usage: $0 [-d JOURS] [-p CHEMIN]"
    echo "  -d  Nombre de jours de rétention (defaut: 7)"
    echo "  -p  Chemin du répertoire backup (defaut: /var/backups)"
    exit 0
}

while getopts "d:p:h" opt; do
    case $opt in
        d) RETENTION_DAYS="$OPTARG" ;;
        p) BACKUP_DIR="$OPTARG"    ;;
        h) usage                    ;;
        *) usage                    ;;
    esac
done
```

#### Script 1 : Monitoring système en temps réel

Un script de monitoring qui vous envoie une alerte quand quelque chose ne va pas. Pas Prometheus, pas Grafana, juste un script bash qui fait le travail, déployable en 30 secondes. Pour un monitoring plus avancé, découvrez [comment surveiller vos serveurs avec Netdata](https://www.shpv.fr/blog/supervision-serveurs-netdata/).

```bash
#!/usr/bin/env bash
set -euo pipefail

# ─── Configuration ───────────────────────────────────────────
readonly CPU_THRESHOLD=85
readonly MEM_THRESHOLD=80
readonly DISK_THRESHOLD=75
readonly SLACK_WEBHOOK="${SLACK_WEBHOOK:-}"
readonly HOSTNAME=$(hostname -f)

# ─── Logging ─────────────────────────────────────────────────
log()   { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO]  $*"; }
warn()  { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [WARN]  $*" >&2; }
error() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [ERROR] $*" >&2; }

# ─── Fonctions de check ──────────────────────────────────────
check_cpu() {
    local cpu_usage
    cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print int($2)}')
    if (( cpu_usage > CPU_THRESHOLD )); then
        warn "CPU critique : ${cpu_usage}% (seuil: ${CPU_THRESHOLD}%)"
        return 1
    fi
    log "CPU OK : ${cpu_usage}%"
    return 0
}

check_memory() {
    local mem_usage
    mem_usage=$(free | awk '/Mem:/ {print int($3/$2 * 100)}')
    if (( mem_usage > MEM_THRESHOLD )); then
        warn "Mémoire critique : ${mem_usage}% (seuil: ${MEM_THRESHOLD}%)"
        return 1
    fi
    log "Mémoire OK : ${mem_usage}%"
    return 0
}

check_disk() {
    local alerts=0
    while IFS= read -r line; do
        local usage mount
        usage=$(echo "$line" | awk '{print int($5)}')
        mount=$(echo "$line" | awk '{print $6}')

        # Ignorer les partitions temporaires
        [[ "$mount" =~ ^/run|^/dev|^/sys|^/tmp ]] && continue

        if (( usage > DISK_THRESHOLD )); then
            warn "Disque critique : $mount à ${usage}% (seuil: ${DISK_THRESHOLD}%)"
            (( alerts++ ))
        fi
    done < <(df -h | tail -n +2)

    if (( alerts > 0 )); then
        return 1
    fi
    log "Disques OK"
    return 0
}

check_services() {
    local services=("nginx" "postgresql" "sshd" "docker")
    local alerts=0

    for service in "${services[@]}"; do
        if ! systemctl is-active --quiet "$service" 2>/dev/null; then
            warn "Service DOWN : $service"
            (( alerts++ ))
        fi
    done

    if (( alerts > 0 )); then
        return 1
    fi
    log "Services OK"
    return 0
}

# ─── Alerte Slack ────────────────────────────────────────────
send_alert() {
    local message="$1"
    if [[ -z "$SLACK_WEBHOOK" ]]; then
        warn "Pas de SLACK_WEBHOOK configuré — alerte non envoyée"
        return 0
    fi
    curl -s -X POST -H 'Content-type: application/json' \
        --data "{\"text\": \"🚨 *[$HOSTNAME]* $message\"}" \
        "$SLACK_WEBHOOK"
}

# ─── Main ────────────────────────────────────────────────────
main() {
    log "=== Début du monitoring ==="
    local alerts=0

    check_cpu    || { send_alert "CPU > ${CPU_THRESHOLD}%";              (( alerts++ )); }
    check_memory || { send_alert "Mémoire > ${MEM_THRESHOLD}%";         (( alerts++ )); }
    check_disk   || { send_alert "Disque > ${DISK_THRESHOLD}%";         (( alerts++ )); }
    check_services || { send_alert "Service(s) en arrêt";               (( alerts++ )); }

    if (( alerts == 0 )); then
        log "=== Tout OK ==="
    else
        log "=== $alerts alerte(s) émise(s) ==="
    fi
}

main "$@"
```

**Déploiement avec cron :**

```bash
# Toutes les 5 minutes
echo "*/5 * * * * root SLACK_WEBHOOK='https://hooks.slack.com/...' /opt/scripts/monitor.sh >> /var/log/monitor.log 2>&1" | sudo tee /etc/cron.d/system-monitor
```

Pour en savoir plus sur la planification de tâches, consultez le [guide complet sur cron et crontab](https://www.shpv.fr/blog/cron-crontab-2026/).

#### Script 2 : Backup automatisé avec rotation

Le script de backup classique, mais fait correctement. Rotation de fichiers, compression, vérification d'intégrité, alertes si échec.

```bash
#!/usr/bin/env bash
set -euo pipefail

# ─── Configuration ───────────────────────────────────────────
readonly BACKUP_DIR="${BACKUP_DIR:-/var/backups}"
readonly RETENTION_DAYS="${RETENTION_DAYS:-7}"
readonly TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
readonly LOG_FILE="/var/log/backup.log"

# Les sources à sauvegarder
declare -a BACKUP_SOURCES=(
    "/etc"
    "/var/www"
    "/home"
)

# ─── Logging ─────────────────────────────────────────────────
log()   { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO]  $*" | tee -a "$LOG_FILE"; }
error() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [ERROR] $*" | tee -a "$LOG_FILE" >&2; }

# ─── Fonctions ───────────────────────────────────────────────
create_backup() {
    local source="$1"
    local name
    name=$(basename "$source")
    local archive="${BACKUP_DIR}/${name}_${TIMESTAMP}.tar.gz"

    log "Backup en cours : $source"

    if ! tar -czf "$archive" -C "$(dirname "$source")" "$name" 2>>"$LOG_FILE"; then
        error "Échec du backup : $source"
        return 1
    fi

    # Vérification intégrité
    if ! tar -tzf "$archive" >/dev/null 2>&1; then
        error "Archive corrompue : $archive"
        rm -f "$archive"
        return 1
    fi

    log "Backup OK : $archive ($(du -h "$archive" | cut -f1))"
    return 0
}

cleanup_old_backups() {
    log "Nettoyage des backups > ${RETENTION_DAYS} jours"
    local count
    count=$(find "$BACKUP_DIR" -name "*.tar.gz" -mtime +"$RETENTION_DAYS" -print | wc -l)

    if (( count > 0 )); then
        find "$BACKUP_DIR" -name "*.tar.gz" -mtime +"$RETENTION_DAYS" -delete
        log "$count fichier(s) supprimé(s)"
    else
        log "Rien à nettoyer"
    fi
}

# ─── Main ────────────────────────────────────────────────────
main() {
    log "=== Début du backup ==="

    # Créer le répertoire si inexistant
    mkdir -p "$BACKUP_DIR"

    local failed=0
    for source in "${BACKUP_SOURCES[@]}"; do
        if [[ -d "$source" ]] || [[ -f "$source" ]]; then
            create_backup "$source" || (( failed++ ))
        else
            error "Source introuvable : $source"
            (( failed++ ))
        fi
    done

    cleanup_old_backups

    if (( failed > 0 )); then
        error "=== Backup terminé avec $failed erreur(s) ==="
        exit 1
    fi
    log "=== Backup terminé avec succès ==="
}

main "$@"
```

**Déploiement :**

```bash
# Backup quotidien à 2h du matin
echo "0 2 * * * root BACKUP_DIR='/var/backups' RETENTION_DAYS=7 /opt/scripts/backup.sh" | sudo tee /etc/cron.d/daily-backup
```

#### Script 3 : Maintenance préventive

Un script qui fait le nettoyage régulier que personne ne pense à faire : logs anciens, paquets obsolètes, fichiers temporaires, certificats qui approchent de leur expiration.

```bash
#!/usr/bin/env bash
set -euo pipefail

# ─── Configuration ───────────────────────────────────────────
readonly LOG_FILE="/var/log/maintenance.log"
readonly CERT_WARN_DAYS=30

# ─── Logging ─────────────────────────────────────────────────
log()   { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [INFO]  $*" | tee -a "$LOG_FILE"; }
warn()  { echo "[$(date '+%Y-%m-%d %H:%M:%S')] [WARN]  $*" | tee -a "$LOG_FILE" >&2; }

# ─── Nettoyage des logs ─────────────────────────────────────
cleanup_logs() {
    log "Nettoyage des logs anciens (> 30 jours)"
    find /var/log -name "*.log" -mtime +30 -exec gzip {} \;
    find /var/log -name "*.log.gz" -mtime +90 -delete
    log "Logs nettoyés"
}

# ─── Nettoyage des paquets ───────────────────────────────────
cleanup_packages() {
    log "Nettoyage des paquets"
    if command -v apt &>/dev/null; then
        apt-get clean
        apt-get autoremove -y
    elif command -v yum &>/dev/null; then
        yum clean all
    fi
    log "Paquets nettoyés"
}

# ─── Fichiers temporaires ────────────────────────────────────
cleanup_tmp() {
    log "Nettoyage des fichiers temporaires (> 7 jours)"
    local count
    count=$(find /tmp -type f -mtime +7 -print | wc -l)
    find /tmp -type f -mtime +7 -delete
    log "$count fichier(s) temporaire(s) supprimé(s)"
}

# ─── Vérification des certificats SSL ────────────────────────
check_certificates() {
    log "Vérification des certificats SSL"

    for cert in /etc/letsencrypt/live/*/cert.pem; do
        [[ -f "$cert" ]] || continue

        local domain expiry days_left
        domain=$(openssl x509 -in "$cert" -noout -subject 2>/dev/null | sed 's/.*CN\s*=\s*//')
        expiry=$(openssl x509 -in "$cert" -noout -enddate 2>/dev/null | cut -d= -f2)
        days_left=$(( ($(date -d "$expiry" +%s) - $(date +%s)) / 86400 ))

        if (( days_left < CERT_WARN_DAYS )); then
            warn "Certificat '$domain' expire dans $days_left jour(s) ($expiry)"
        else
            log "Certificat '$domain' OK : expire dans $days_left jour(s)"
        fi
    done
}

# ─── Main ────────────────────────────────────────────────────
main() {
    log "=== Début de la maintenance ==="

    cleanup_logs
    cleanup_packages
    cleanup_tmp
    check_certificates

    log "=== Maintenance terminée ==="
}

main "$@"
```

**Déploiement :**

```bash
# Chaque dimanche à 3h du matin
echo "0 3 * * 0 root /opt/scripts/maintenance.sh" | sudo tee /etc/cron.d/weekly-maintenance
```

#### Tableau récapitulatif

|Script|Fréquence recommandée|Logs|Alertes|
|---|---|---|---|
|monitor.sh|Toutes les 5 min|`/var/log/monitor.log`|Slack webhook|
|backup.sh|Quotidien (2h)|`/var/log/backup.log`|Exit code non-zero|
|maintenance.sh|Hebdomadaire (dim. 3h)|`/var/log/maintenance.log`|Warn dans les logs|

#### Pièges classiques à éviter

Il y a quelques erreurs que font régulièrement les sysadmins quand ils écrivent des scripts bash en production. Ne pas utiliser `set -euo pipefail` est le numéro un, un script qui continue après une erreur silencieuse peut faire des dégâts considérables. Oublier de vérifier l'intégrité après une compression est l'autre classique : un backup corrompu qu'on découvre seulement au moment de la restauration, c'est catastrophique. Enfin, hardcoder les chemins et les valeurs au lieu d'utiliser des variables d'environnement, c'est garantir que le script ne fonctionnera que sur votre machine et nulle part ailleurs.

#### Conclusion

Ces trois scripts couvrent l'essentiel de ce que tout sysadmin fait régulièrement, à la main. Monitoring, backup, maintenance, automatisé, loggé, alerté. L'étape suivante ? Versionnez ces scripts dans un dépôt Git, ajoutez-leur des tests basiques avec `bats`, et déployez-les via Ansible. Mais ça, c'est un autre article.

Pour intégrer ces scripts dans votre cron, consultez le [guide complet du cron et crontab](https://www.shpv.fr/blog/cron-crontab-2026/).