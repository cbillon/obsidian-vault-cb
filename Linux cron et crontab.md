---
tags:
  - linux
  - cron
---
Cron est l'outil d'automatisation de tâches le plus utilisé sous Linux. Ce guide complet explique comment planifier des tâches automatiques avec cron et crontab, de la syntaxe basique aux techniques avancées.

#### Qu'est-ce que Cron ?

##### Définition Simple

**Cron** est un démon (service en arrière-plan) Linux qui exécute des commandes à des moments programmés. C'est comme un "réveil" pour votre serveur qui déclenche des actions automatiquement.

**Exemples d'utilisation courante :**

- Sauvegardes automatiques tous les jours à 2h du matin
- Nettoyage des logs tous les dimanches
- Vérification de mises à jour toutes les heures
- Envoi de rapports par email chaque vendredi
- Redémarrage d'un service toutes les 6 heures

##### Composants du Système Cron

```
┌─────────────────────────────────────────┐
│          SYSTÈME CRON                   │
├─────────────────────────────────────────┤
│                                         │
│  ┌──────────────────────────────────┐  │
│  │  crond (daemon)                  │  │
│  │  Service qui tourne en permanence│  │
│  │  Vérifie chaque minute           │  │
│  └──────────────────────────────────┘  │
│              ↓ lit                     │
│  ┌──────────────────────────────────┐  │
│  │  Fichiers crontab                │  │
│  │  - /etc/crontab (système)        │  │
│  │  - /var/spool/cron/user (users)  │  │
│  │  - /etc/cron.d/ (apps)           │  │
│  └──────────────────────────────────┘  │
│              ↓ exécute                 │
│  ┌──────────────────────────────────┐  │
│  │  Commandes planifiées            │  │
│  │  Scripts, programmes             │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

##### Vérifier que Cron Fonctionne

```bash
# Vérifier statut service cron
sudo systemctl status cron      # Debian/Ubuntu
sudo systemctl status crond     # RedHat/CentOS

# Démarrer si arrêté
sudo systemctl start cron
sudo systemctl enable cron      # Démarrage auto au boot

# Voir les logs cron
sudo tail -f /var/log/syslog | grep CRON    # Debian/Ubuntu
sudo tail -f /var/log/cron                  # RedHat/CentOS
```

Pour une gestion complète de vos logs, consultez notre [guide journalctl et gestion des logs](https://www.shpv.fr/blog/logs-linux-2026/). Pour automatiser la rotation des logs créés par cron, découvrez [logrotate et compression](https://www.shpv.fr/blog/logrotate-compression/).

---

#### Syntaxe Crontab : Comprendre les 5 Étoiles

Pour une alternative moderne à cron, découvrez les [systemd timers avancés](https://www.shpv.fr/blog/systemd-avance/) qui offrent plus de flexibilité et une meilleure intégration système.

##### Format de Base

Chaque ligne crontab suit ce format :

```
┌───────────── minute (0-59)
│ ┌─────────── heure (0-23)
│ │ ┌───────── jour du mois (1-31)
│ │ │ ┌─────── mois (1-12)
│ │ │ │ ┌───── jour de la semaine (0-7, 0 et 7 = dimanche)
│ │ │ │ │
* * * * * commande à exécuter
```

##### Exemples Basiques

```bash
# Tous les jours à 2h30 du matin
30 2 * * * /home/user/backup.sh

# Toutes les heures, à la minute 0
0 * * * * /usr/local/bin/check-updates.sh

# Tous les lundis à 9h
0 9 * * 1 /home/user/rapport-hebdo.sh

# Le 1er de chaque mois à minuit
0 0 1 * * /home/user/facturation.sh

# Tous les jours à minuit
0 0 * * * /home/user/cleanup.sh
```

##### Syntaxe Avancée : Les Opérateurs

**Astérisque (*) : "chaque"**

```bash
# Toutes les minutes
* * * * * echo "Chaque minute"

# Toutes les heures de chaque jour
0 * * * * echo "Chaque heure"
```

**Virgule (,) : "liste de valeurs"**

```bash
# À 8h, 12h et 18h
0 8,12,18 * * * /home/user/rappel.sh

# Lundi, mercredi et vendredi
0 9 * * 1,3,5 /home/user/reunion.sh
```

**Tiret (-) : "plage de valeurs"**

```bash
# De 9h à 17h (heures de bureau)
0 9-17 * * * /home/user/check-prod.sh

# Du lundi au vendredi
0 8 * * 1-5 /home/user/workday.sh
```

**Slash (/) : "intervalle"**

```bash
# Toutes les 5 minutes
*/5 * * * * /home/user/monitor.sh

# Toutes les 2 heures
0 */2 * * * /home/user/sync.sh

# Tous les 3 jours
0 0 */3 * * /home/user/maintenance.sh
```

**Combinaisons**

```bash
# Toutes les 10 minutes entre 9h et 18h, lun-ven
*/10 9-18 * * 1-5 /home/user/business-check.sh

# À 8h30 et 20h30, tous les jours impairs du mois
30 8,20 1-31/2 * * /home/user/bi-daily.sh
```

---

#### Utiliser Crontab en Pratique

##### Commandes Crontab Essentielles

```bash
# Éditer crontab de l'utilisateur actuel
crontab -e

# Lister crontab actuel
crontab -l

# Supprimer crontab
crontab -r

# Éditer crontab d'un autre utilisateur (root)
sudo crontab -u username -e

# Lister crontab d'un utilisateur
sudo crontab -u username -l
```

##### Premier Crontab : Exemple Pas à Pas

**Objectif :** Créer une sauvegarde automatique tous les jours à 3h du matin.

**Étape 1 : Créer le script de backup**

```bash
# Créer le script
nano /home/user/backup-daily.sh
```

```bash
#!/bin/bash
# Script de backup quotidien

# Variables
BACKUP_DIR="/home/user/backups"
SOURCE_DIR="/home/user/data"
DATE=$(date +%Y%m%d)
BACKUP_FILE="backup-$DATE.tar.gz"

# Créer répertoire backup si n'existe pas
mkdir -p "$BACKUP_DIR"

# Créer archive
tar -czf "$BACKUP_DIR/$BACKUP_FILE" "$SOURCE_DIR"

# Supprimer backups de plus de 7 jours
find "$BACKUP_DIR" -name "backup-*.tar.gz" -mtime +7 -delete

# Log
echo "$(date): Backup completed - $BACKUP_FILE" >> /home/user/backup.log
```

**Étape 2 : Rendre le script exécutable**

```bash
chmod +x /home/user/backup-daily.sh
```

**Étape 3 : Tester le script manuellement**

```bash
/home/user/backup-daily.sh

# Vérifier que ça fonctionne
ls -lh /home/user/backups/
cat /home/user/backup.log
```

**Étape 4 : Ajouter au crontab**

```bash
crontab -e
```

Ajouter cette ligne :

```cron
# Backup quotidien à 3h du matin
0 3 * * * /home/user/backup-daily.sh
```

**Étape 5 : Vérifier l'ajout**

```bash
crontab -l
```

---

#### Exemples Crontab Courants

##### Maintenance et Nettoyage

```bash
# Nettoyer /tmp tous les jours à 4h
0 4 * * * find /tmp -type f -mtime +7 -delete

# Vider corbeille tous les lundis
0 9 * * 1 rm -rf ~/.local/share/Trash/*

# Nettoyer logs anciens chaque dimanche
0 2 * * 0 find /var/log -name "*.log" -mtime +30 -delete

# Rotation logs application
0 0 * * * /usr/sbin/logrotate /etc/logrotate.conf
```

##### Sauvegardes

```bash
# Backup base de données tous les jours à 2h
0 2 * * * mysqldump -u root -pPASSWORD database > /backups/db-$(date +\%Y\%m\%d).sql

# Backup incrémental toutes les 6 heures
0 */6 * * * rsync -avz /data/ /backup/incremental/

# Backup complet hebdomadaire (dimanche)
0 1 * * 0 tar -czf /backup/weekly-$(date +\%Y\%m\%d).tar.gz /home

# Sync vers serveur distant tous les jours
30 3 * * * rsync -avz -e ssh /backup/ user@remote:/remote-backup/
```

##### Monitoring et Alertes

```bash
# Vérifier espace disque toutes les heures
0 * * * * df -h | grep -E '9[0-9]%|100%' && mail -s "Disk Alert" admin@example.com

# Vérifier services toutes les 5 minutes
*/5 * * * * systemctl is-active nginx || systemctl restart nginx

# Ping serveur distant toutes les 10 minutes
*/10 * * * * ping -c 1 remote-server > /dev/null || echo "Server down!" | mail -s "Alert" admin@example.com

# Surveiller charge système
*/15 * * * * uptime | awk '{print $10}' | cut -d, -f1 | awk '{if($1 > 5) print "High load: "$1}' | mail -s "Load Alert" admin@example.com
```

##### Mise à Jour et Maintenance Système

```bash
# Vérifier mises à jour tous les jours
0 8 * * * apt update && apt list --upgradable > /tmp/updates.txt

# Reboot hebdomadaire (dimanche 4h)
0 4 * * 0 /sbin/reboot

# Nettoyer packages obsolètes mensuellement
0 3 1 * * apt autoremove -y

# Mise à jour certificats Let's Encrypt
0 3 * * * certbot renew --quiet
```

##### Web et Applications

```bash
# Générer sitemap toutes les nuits
0 2 * * * /var/www/scripts/generate-sitemap.sh

# Clear cache application tous les matins
0 6 * * * rm -rf /var/www/app/cache/*

# Redémarrer serveur node.js toutes les 6h (leak mémoire)
0 */6 * * * systemctl restart nodejs-app

# Crawler site pour warmer cache
0 5 * * * wget --spider --force-html -r -l 1 http://mysite.com > /dev/null 2>&1
```

---

#### Variables d'Environnement dans Crontab

##### Le Problème du PATH

Les tâches cron s'exécutent dans un environnement minimal. Le `PATH` est souvent limité à `/usr/bin:/bin`.

**Problème courant :**

```bash
# Cette ligne peut échouer si 'node' est dans /usr/local/bin
0 * * * * node /home/user/app.js
# Erreur: "node: command not found"
```

**Solution 1 : Utiliser chemins absolus**

```bash
# Trouver le chemin complet
which node
# /usr/local/bin/node

# Utiliser dans crontab
0 * * * * /usr/local/bin/node /home/user/app.js
```

**Solution 2 : Définir PATH dans crontab**

```bash
crontab -e
```

```cron
# Définir PATH en début de crontab
PATH=/usr/local/bin:/usr/bin:/bin

# Maintenant node est trouvé
0 * * * * node /home/user/app.js
```

##### Autres Variables Utiles

```cron
# Définir variables en début de crontab

# PATH pour trouver commandes
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin

# SHELL pour interpréter commandes
SHELL=/bin/bash

# MAILTO pour recevoir emails (vide = pas d'email)
MAILTO=admin@example.com
# ou
MAILTO=""    # Désactiver emails

# HOME pour ~
HOME=/home/user

# Exemple complet
PATH=/usr/local/bin:/usr/bin:/bin
SHELL=/bin/bash
MAILTO=admin@example.com

# Tâches
0 2 * * * /home/user/backup.sh
0 8 * * * cd /home/user/app && node server.js
```

---

#### Crontab Système vs Utilisateur

##### Crontab Utilisateur

```bash
# Chaque utilisateur a son propre crontab
crontab -e

# Fichiers stockés dans
/var/spool/cron/crontabs/username    # Debian/Ubuntu
/var/spool/cron/username              # RedHat/CentOS
```

**Format : 5 champs + commande**

```cron
# minute heure jour mois jour_semaine commande
0 2 * * * /home/user/backup.sh
```

##### Crontab Système (/etc/crontab)

```bash
# Fichier édité directement
sudo nano /etc/crontab
```

**Format : 5 champs + utilisateur + commande**

```cron
# minute heure jour mois jour_semaine utilisateur commande
0 2 * * * root /usr/local/bin/system-backup.sh
30 3 * * * www-data /var/www/scripts/cleanup.sh
```

##### Répertoires /etc/cron.*

Linux fournit des raccourcis pour les tâches courantes :

```bash
/etc/cron.hourly/    # Scripts exécutés toutes les heures
/etc/cron.daily/     # Scripts exécutés tous les jours
/etc/cron.weekly/    # Scripts exécutés toutes les semaines
/etc/cron.monthly/   # Scripts exécutés tous les mois
```

**Utilisation :**

```bash
# Créer script dans /etc/cron.daily/
sudo nano /etc/cron.daily/cleanup

#!/bin/bash
find /tmp -type f -mtime +7 -delete
echo "Cleanup done: $(date)" >> /var/log/cleanup.log

# Rendre exécutable
sudo chmod +x /etc/cron.daily/cleanup

# Sera exécuté automatiquement chaque jour
# (timing géré par /etc/crontab ou anacron)
```

##### /etc/cron.d/ pour Applications

Les applications peuvent installer leurs propres crontabs :

```bash
# Fichier /etc/cron.d/myapp
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin

# minute heure jour mois jour_semaine utilisateur commande
*/5 * * * * myapp /usr/local/bin/myapp-check.sh
0 3 * * * myapp /usr/local/bin/myapp-backup.sh
```

---

#### Debugging : Pourquoi Mon Cron Ne Fonctionne Pas ?

##### Checklist de Debugging

**1. Le service cron tourne-t-il ?**

```bash
sudo systemctl status cron

# Si arrêté
sudo systemctl start cron
```

**2. La syntaxe est-elle correcte ?**

```bash
# Vérifier syntaxe en ligne
# https://crontab.guru/

# Ou utiliser crontab-validator
pip install crontab-validator
crontab-validator "*/5 * * * *"
```

**3. Le script est-il exécutable ?**

```bash
ls -l /home/user/script.sh
# Doit afficher -rwxr-xr-x

# Rendre exécutable si besoin
chmod +x /home/user/script.sh
```

**4. Le script fonctionne-t-il manuellement ?**

```bash
# Tester en tant qu'utilisateur concerné
sudo -u username /home/user/script.sh

# Regarder les erreurs
bash -x /home/user/script.sh
```

**5. Le PATH est-il correct ?**

```bash
# Ajouter en début de crontab
PATH=/usr/local/bin:/usr/bin:/bin

# Ou utiliser chemins absolus dans script
#!/bin/bash
/usr/bin/node /home/user/app.js
```

**6. Rediriger la sortie pour debug**

```bash
# Capturer stdout et stderr
* * * * * /home/user/script.sh >> /home/user/cron.log 2>&1

# Maintenant regarder le log
tail -f /home/user/cron.log
```

##### Voir les Logs Cron

```bash
# Debian/Ubuntu
sudo tail -f /var/log/syslog | grep CRON

# RedHat/CentOS
sudo tail -f /var/log/cron

# Voir toutes exécutions cron aujourd'hui
sudo grep CRON /var/log/syslog | grep "$(date +%b\ %d)"

# Voir uniquement les erreurs
sudo grep CRON /var/log/syslog | grep -i error
```

Pour une analyse plus complète, consultez le [guide complet sur la gestion des logs Linux](https://www.shpv.fr/blog/logs-linux-2026/).

##### Tester Cron avec Commande Simple

```bash
# Ajouter test qui écrit dans fichier toutes les minutes
crontab -e

* * * * * echo "Test $(date)" >> /tmp/crontest.txt

# Attendre 2 minutes puis vérifier
cat /tmp/crontest.txt

# Si ça fonctionne, le problème vient de votre script
# Si ça ne fonctionne pas, problème avec cron lui-même
```

---

#### Alternatives Modernes à Cron

##### Systemd Timers

Les distributions Linux modernes utilisent systemd, qui offre une alternative à cron. Pour découvrir le [guide complet sur les services systemd et timers](https://www.shpv.fr/blog/systemd-services-2026/), consultez notre article dédié.

**Avantages systemd timers :**

- Logs intégrés dans journald
- Dépendances entre services
- Exécution au boot si tâche manquée
- Syntaxe plus lisible

**Exemple : Backup quotidien avec systemd**

**1. Créer le service (/etc/systemd/system/backup.service)**

```ini
[Unit]
Description=Daily Backup
Wants=backup.timer

[Service]
Type=oneshot
ExecStart=/home/user/backup.sh
User=user

[Install]
WantedBy=multi-user.target
```

**2. Créer le timer (/etc/systemd/system/backup.timer)**

```ini
[Unit]
Description=Run backup daily at 3am
Requires=backup.service

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

**3. Activer et démarrer**

```bash
sudo systemctl daemon-reload
sudo systemctl enable backup.timer
sudo systemctl start backup.timer

# Vérifier status
sudo systemctl status backup.timer

# Voir prochaine exécution
sudo systemctl list-timers backup.timer
```

**Comparaison Cron vs Systemd Timer**

|Feature|Cron|Systemd Timer|
|---|---|---|
|Syntaxe|Compacte mais obscure|Verbose mais claire|
|Logs|syslog|journalctl intégré|
|Execution manquée|Ignorée|Peut rattraper|
|Dépendances|Non|Oui|
|Précision|Minute|Seconde|
|Learning curve|Faible|Moyenne|

##### Anacron

Pour machines qui ne tournent pas 24/7 (laptops).

```bash
# Installer anacron
sudo apt install anacron

# Fichier /etc/anacrontab
# period delay job-id command
1    5    cron.daily    run-parts /etc/cron.daily
7    10   cron.weekly   run-parts /etc/cron.weekly
30   15   cron.monthly  run-parts /etc/cron.monthly
```

Anacron garantit l'exécution même si machine éteinte au moment prévu.

---

#### Bonnes Pratiques Production

##### 1. Toujours Logger

```bash
# Mauvais : pas de trace
0 2 * * * /home/user/backup.sh

# Bon : logs complets
0 2 * * * /home/user/backup.sh >> /home/user/backup.log 2>&1
```

##### 2. Utiliser Flock pour Éviter Overlaps

Si un script prend du temps et peut overlap :

```bash
# Sans protection, si backup prend 2h, 2 instances tournent
0 */1 * * * /home/user/long-backup.sh

# Avec flock, n'exécute que si précédent terminé
0 */1 * * * flock -n /tmp/backup.lock /home/user/long-backup.sh
```

##### 3. Gestion des Erreurs

```bash
#!/bin/bash
set -e  # Arrêter si erreur
set -u  # Erreur si variable non définie
set -o pipefail  # Propager erreurs dans pipes

# Fonction de logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" >> /var/log/myscript.log
}

# Trap pour cleanup
cleanup() {
    log "Script interrupted"
    # Cleanup actions
    exit 1
}
trap cleanup INT TERM

log "Starting backup..."

# Ton code ici

log "Backup completed successfully"
```

##### 4. Notifications

```bash
# Envoyer email si échec
0 2 * * * /home/user/backup.sh || echo "Backup failed!" | mail -s "ALERT" admin@example.com

# Utiliser service de monitoring (healthchecks.io, etc.)
0 2 * * * /home/user/backup.sh && curl -fsS https://hc-ping.com/your-uuid > /dev/null
```

##### 5. Variables Sensibles

```bash
# Ne JAMAIS mettre passwords dans crontab
# Mauvais
0 2 * * * mysqldump -u root -pMYPASSWORD db > backup.sql

# Bon : utiliser .my.cnf
# /home/user/.my.cnf
[client]
user=root
password=MYPASSWORD

# Crontab
0 2 * * * mysqldump db > backup.sql
```

---

#### Exemples Complets Scripts Production

Pour des scripts plus avancés, consultez nos [bonnes pratiques du bash scripting sysadmin](https://www.shpv.fr/blog/bash-scripting-sysadmin/).

##### Script Backup PostgreSQL

```bash
#!/bin/bash
# /home/user/scripts/postgres-backup.sh

set -euo pipefail

# Configuration
BACKUP_DIR="/backups/postgres"
RETENTION_DAYS=7
DB_NAME="myapp"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/${DB_NAME}_${TIMESTAMP}.sql.gz"
LOG_FILE="/var/log/postgres-backup.log"

# Fonction logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Créer répertoire si n'existe pas
mkdir -p "$BACKUP_DIR"

# Backup
log "Starting backup of $DB_NAME"
if pg_dump "$DB_NAME" | gzip > "$BACKUP_FILE"; then
    log "Backup successful: $BACKUP_FILE"
    SIZE=$(du -h "$BACKUP_FILE" | cut -f1)
    log "Size: $SIZE"
else
    log "ERROR: Backup failed!"
    exit 1
fi

# Nettoyer anciens backups
log "Cleaning old backups (>$RETENTION_DAYS days)"
DELETED=$(find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete -print | wc -l)
log "Deleted $DELETED old backups"

# Vérifier espace disque
DISK_USAGE=$(df -h "$BACKUP_DIR" | tail -1 | awk '{print $5}' | sed 's/%//')
if [ "$DISK_USAGE" -gt 80 ]; then
    log "WARNING: Disk usage high: ${DISK_USAGE}%"
fi

log "Backup completed successfully"
```

**Crontab :**

```cron
# Backup PostgreSQL tous les jours à 2h30
30 2 * * * /home/user/scripts/postgres-backup.sh
```

---

#### Ressources et Outils

##### Outils Utiles

**crontab.guru** - Tester syntaxe cron

```
https://crontab.guru/
```

**cronitor** - Monitoring de cron jobs

```bash
curl https://cronitor.link/XXXX/run
```

**healthchecks.io** - Ping service pour monitoring

```bash
0 3 * * * /home/user/backup.sh && curl https://hc-ping.com/UUID
```

##### Documentation

```bash
# Man pages
man 5 crontab    # Format crontab
man 1 crontab    # Commande crontab
man 8 cron       # Daemon cron

# Exemples système
cat /etc/crontab
ls /etc/cron.d/
```

---

#### Conclusion

Cron reste l'outil de planification de tâches le plus simple sous Linux. Points clés à retenir :

**Syntaxe de base :**

```
minute heure jour mois jour_semaine commande
```

**Commandes essentielles :**

```bash
crontab -e    # Éditer
crontab -l    # Lister
crontab -r    # Supprimer
```

**Bonnes pratiques :**

- ✓ Toujours logger (avec timestamps)
- ✓ Utiliser chemins absolus
- ✓ Tester scripts manuellement avant
- ✓ Gérer les erreurs et notifications
- ✓ Utiliser flock pour éviter overlaps

**Alternatives modernes :**

- systemd timers (plus complet, logs intégrés)
- anacron (machines pas 24/7)

Avec ce guide, vous pouvez maintenant automatiser n'importe quelle tâche récurrente sur vos serveurs Linux !