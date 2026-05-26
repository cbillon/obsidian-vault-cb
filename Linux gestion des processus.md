---
tags:
  - linux
---
La gestion des processus est une compétence essentielle pour tout administrateur Linux. Ce guide complet explique comment visualiser, gérer et optimiser les processus système.

#### Comprendre les Processus Linux

##### Qu'est-ce qu'un Processus ?

Un **processus** est un programme en cours d'exécution. Chaque processus a :

- Un **PID** (Process ID) unique
- Un **PPID** (Parent Process ID)
- Un **utilisateur** propriétaire
- Une **priorité** d'exécution
- Des **ressources** (CPU, RAM, fichiers)

##### Hiérarchie des Processus

```
systemd (PID 1)
├── sshd (PID 850)
│   ├── sshd (PID 1234) - session user1
│   └── sshd (PID 1456) - session user2
├── nginx (PID 920)
│   ├── nginx worker (PID 921)
│   ├── nginx worker (PID 922)
│   └── nginx worker (PID 923)
└── mysql (PID 1050)
    ├── mysql thread (PID 1051)
    └── mysql thread (PID 1052)
```

##### États des Processus

|État|Code|Description|
|---|---|---|
|Running|R|En cours d'exécution|
|Sleeping|S|En attente (interruptible)|
|Uninterruptible|D|En attente (non-interruptible)|
|Stopped|T|Arrêté (Ctrl+Z)|
|Zombie|Z|Terminé, attend parent|

---

#### Commande ps : Lister les Processus

##### ps - Syntaxe de Base

```bash
# Processus de l'utilisateur actuel
ps

# Tous les processus (format BSD)
ps aux

# Tous les processus (format System V)
ps -ef

# Processus avec arborescence
ps auxf
ps -ejH
```

##### ps aux : Colonnes Expliquées

```bash
ps aux
# USER   PID %CPU %MEM    VSZ   RSS TTY  STAT START   TIME COMMAND

# Exemple ligne
# root   1234  2.5  1.2  123456 98765 ?   Ss   10:30   0:05 nginx
```

**Signification colonnes :**

- **USER** : Propriétaire du processus
- **PID** : Process ID
- **%CPU** : Pourcentage CPU utilisé
- **%MEM** : Pourcentage RAM utilisée
- **VSZ** : Virtual memory Size (KB)
- **RSS** : Resident Set Size - RAM réelle (KB)
- **TTY** : Terminal (? = pas de terminal)
- **STAT** : État (R/S/D/Z/T)
- **START** : Heure démarrage
- **TIME** : Temps CPU cumulé
- **COMMAND** : Commande exécutée

##### ps - Exemples Pratiques

```bash
# Processus d'un utilisateur spécifique
ps -u username

# Processus par nom
ps aux | grep nginx
# Meilleur : pgrep
pgrep nginx

# Processus utilisant le plus de CPU
ps aux --sort=-%cpu | head -10

# Processus utilisant le plus de RAM
ps aux --sort=-%mem | head -10

# Processus par PID
ps -p 1234

# Arborescence complète
ps auxf
# ou
pstree

# Format personnalisé
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head

# Threads d'un processus
ps -T -p 1234
```

##### pgrep et pkill

```bash
# Trouver PID par nom
pgrep nginx
# 920 921 922 923

# Avec détails
pgrep -a nginx
# 920 nginx: master process
# 921 nginx: worker process

# Par utilisateur
pgrep -u www-data

# Tuer par nom (attention !)
pkill nginx

# Tuer avec signal spécifique
pkill -9 nginx  # SIGKILL (force)
pkill -15 nginx # SIGTERM (graceful)
```

---

#### Commande top : Monitoring Temps Réel

##### top - Interface de Base

```bash
top

# Affichage :
# Ligne 1 : uptime, users, load average
# Ligne 2 : Tasks (total, running, sleeping, stopped, zombie)
# Ligne 3 : %CPU (us=user, sy=system, id=idle, wa=iowait)
# Ligne 4 : Memory
# Ligne 5 : Swap
# Puis : Liste processus
```

##### top - Raccourcis Clavier

```bash
# Pendant top est actif :

# Tri
M - Trier par mémoire
P - Trier par CPU
T - Trier par temps CPU
N - Trier par PID

# Filtres
u - Filtrer par utilisateur
k - Tuer processus (demande PID)
r - Changer priorité (renice)

# Affichage
c - Toggle path complet commande
1 - Afficher tous CPU
f - Choisir colonnes
W - Sauvegarder configuration

# Contrôle
d - Changer délai refresh
q - Quitter
h - Aide
```

##### top - Exemples Pratiques

```bash
# Démarrer avec délai 2 secondes
top -d 2

# Afficher seulement utilisateur
top -u nginx

# Mode batch (pour scripts)
top -b -n 1 > top-output.txt

# Top 10 processus CPU
top -b -n 1 | head -n 17

# Couleurs
top -d 1
# Puis taper 'z' pour couleurs
```

##### Comprendre Load Average

```bash
top
# load average: 1.5, 2.0, 1.8
#               1min 5min 15min

# Interprétation (système 4 cores) :
# < 4.0  : Normal
# 4.0-8.0 : Chargé
# > 8.0  : Surchargé
```

---

#### htop : Alternative Améliorée

##### Installation et Lancement

```bash
# Installer
sudo apt install htop    # Debian/Ubuntu
sudo yum install htop    # RedHat/CentOS

# Lancer
htop
```

##### htop - Avantages sur top

- Interface couleur
- Souris fonctionnelle
- Arborescence processus (F5)
- Recherche interactive (F3)
- Filtres faciles (F4)
- Tri facile (F6)
- Kill multiple processus
- Barres graphiques CPU/RAM

##### htop - Raccourcis Essentiels

```bash
# Navigation
F1  - Aide
F2  - Setup (configuration)
F3  - Search (chercher processus)
F4  - Filter (filtrer affichage)
F5  - Tree (vue arbre)
F6  - Sort (trier)
F9  - Kill (tuer processus)
F10 - Quit

# Autres
Space - Marquer processus
U - Filtrer utilisateur
t - Arbre
H - Cacher threads
K - Cacher kernel threads
```

##### htop - Configuration

```bash
# F2 pour setup, puis :

# Colonnes à afficher
- Ajouter VIRT, RES, SHR, CPU%, MEM%
- Enlever colonnes inutiles

# Couleurs
- Scheme de couleurs

# Options
- Update process names
- Hide userland threads
- Shadow other users' processes
```

---

#### Tuer des Processus : kill et killall

##### Comprendre les Signals

```bash
# Lister tous signals
kill -l

# Signals principaux :
# SIGTERM (15) - Arrêt gracieux (défaut)
# SIGKILL (9)  - Arrêt forcé immédiat
# SIGHUP (1)   - Reload config
# SIGSTOP (19) - Pause
# SIGCONT (18) - Reprendre
```

##### Commande kill

```bash
# Tuer avec SIGTERM (graceful)
kill 1234

# Tuer avec SIGKILL (force)
kill -9 1234
kill -SIGKILL 1234

# Recharger config (HUP)
kill -1 1234
kill -HUP 1234

# Tuer multiple PIDs
kill 1234 1235 1236

# Vérifier si processus existe
kill -0 1234
echo $?  # 0 = existe, 1 = n'existe pas
```

##### Commande killall

```bash
# Tuer tous processus par nom
killall nginx

# Force
killall -9 nginx

# Par utilisateur
killall -u username

# Interactive (demande confirmation)
killall -i firefox

# Verbose
killall -v nginx
```

##### Exemples Pratiques

```bash
# Redémarrer nginx proprement
sudo killall -HUP nginx

# Tuer tous chrome qui freeze
killall -9 chrome

# Arrêter service qui ne répond pas
sudo killall -9 mysql
sudo systemctl start mysql

# Tuer processus zombies (tuer parent)
ps aux | grep defunct
kill -9 PPID_du_zombie
```

---

#### Priorités : nice et renice

##### Comprendre les Priorités

**Nice values** : -20 (haute priorité) à +19 (basse priorité)

- Défaut : 0
- Root peut mettre valeurs négatives
- User normal : 0 à 19 seulement

```bash
# Voir priorités
ps -eo pid,ni,cmd
# PID  NI  COMMAND
# 1234  0  nginx
# 1235 10  backup.sh
# 1236 -5  important-app
```

##### Commande nice

```bash
# Lancer avec priorité basse
nice -n 10 ./backup.sh

# Priorité très basse (pour tâches fond)
nice -n 19 find / -name "*.log"

# Priorité haute (root seulement)
sudo nice -n -10 ./critical-app

# Sans spécifier (défaut +10)
nice ./script.sh
```

##### Commande renice

```bash
# Changer priorité processus existant
renice -n 10 -p 1234

# Multiple PIDs
renice -n 15 -p 1234 1235 1236

# Tous processus d'un user
renice -n 10 -u username

# Tous processus d'un groupe
renice -n 5 -g developers

# Augmenter priorité (root seulement)
sudo renice -n -5 -p 1234
```

##### Exemples Use Cases

```bash
# Backup nocturne (basse priorité)
nice -n 19 tar -czf /backup/data.tar.gz /data

# Compilation (priorité moyenne-basse)
nice -n 10 make -j4

# Service critique (haute priorité)
sudo nice -n -10 ./trading-app

# Réduire priorité processus CPU-intensif
renice -n 19 -p $(pgrep ffmpeg)
```

---

#### Gestion Services : systemctl

##### systemctl - Commandes de Base

```bash
# Voir tous services
systemctl list-units --type=service

# Status service
systemctl status nginx

# Démarrer service
sudo systemctl start nginx

# Arrêter
sudo systemctl stop nginx

# Redémarrer
sudo systemctl restart nginx

# Reload config (sans interruption)
sudo systemctl reload nginx

# Activer au boot
sudo systemctl enable nginx

# Désactiver au boot
sudo systemctl disable nginx

# Vérifier si activé
systemctl is-enabled nginx

# Vérifier si actif
systemctl is-active nginx
```

##### systemctl - Monitoring

Pour une gestion complète des services, consultez notre [guide complet des services systemd](https://www.shpv.fr/blog/systemd-services-2026/).

```bash
# Services échoués
systemctl --failed

# Services actifs
systemctl list-units --type=service --state=running

# Logs service (journalctl)
journalctl -u nginx

# Logs temps réel
journalctl -u nginx -f

# Logs depuis boot
journalctl -u nginx -b

# Logs dernière heure
journalctl -u nginx --since "1 hour ago"
```

Pour une analyse complète des logs, découvrez notre [guide complet des logs Linux](https://www.shpv.fr/blog/logs-linux-2026/).

##### systemctl - Exemples Pratiques

```bash
# Redémarrer si config changée
sudo nano /etc/nginx/nginx.conf
sudo systemctl reload nginx

# Voir dépendances service
systemctl list-dependencies nginx

# Masquer service (impossible de démarrer)
sudo systemctl mask apache2

# Démasquer
sudo systemctl unmask apache2

# Éditer override service
sudo systemctl edit nginx

# Recharger configuration systemd
sudo systemctl daemon-reload
```

---

#### Monitoring Avancé

##### vmstat : Statistiques Système

```bash
# Afficher toutes les 2 secondes
vmstat 2

# Colonnes importantes :
# r  - Processus en attente CPU
# b  - Processus bloqués I/O
# si - Swap in
# so - Swap out
# us - % CPU user
# sy - % CPU system
# id - % CPU idle
# wa - % CPU wait I/O

# Exemple
vmstat 1 10
# procs -------memory------ ---swap-- -----io---- -system-- ------cpu-----
#  r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa
#  2  0      0 123456  45678 234567    0    0    12    45  123  456 10  2 87  1
```

##### iostat : I/O Statistiques

```bash
# Installer
sudo apt install sysstat

# Statistiques CPU et disques
iostat

# Toutes les 2 secondes
iostat 2

# Détails étendus
iostat -x

# Seulement disques spécifiques
iostat -x sda sdb
```

Pour une optimisation complète du système, consultez notre [tuning des performances Linux](https://www.shpv.fr/blog/linux-performance/).

##### pidstat : Stats par Processus

```bash
# Stats tous processus
pidstat

# Toutes les 2 secondes
pidstat 2

# Avec threads
pidstat -t

# I/O par processus
pidstat -d

# Mémoire par processus
pidstat -r

# Context switches
pidstat -w
```

---

#### Debugging Processus

##### strace : Tracer Appels Système

```bash
# Tracer processus existant
sudo strace -p 1234

# Lancer avec trace
strace ls -la

# Sauvegarder output
strace -o trace.log ls -la

# Compter appels système
strace -c ls -la

# Tracer seulement open/read/write
strace -e trace=open,read,write ls -la

# Timestamp
strace -t ls -la
```

##### lsof : Fichiers Ouverts

```bash
# Fichiers ouverts par processus
lsof -p 1234

# Processus utilisant un fichier
lsof /var/log/nginx/access.log

# Processus écoutant sur port
lsof -i :80

# Tous ports réseau
lsof -i

# Par utilisateur
lsof -u www-data

# Fichiers d'un répertoire
lsof +D /var/log
```

##### Processus Zombies

```bash
# Trouver zombies
ps aux | grep Z
ps aux | awk '$8=="Z"'

# Voir parent (PPID)
ps -o ppid= -p ZOMBIE_PID

# Tuer parent pour nettoyer zombie
kill PPID

# Forcer si parent ne répond pas
kill -9 PPID
```

---

#### Limites Ressources : ulimit

##### Voir Limites Actuelles

```bash
# Toutes limites
ulimit -a

# Fichiers ouverts max
ulimit -n

# Processus max
ulimit -u

# Taille fichier max
ulimit -f

# Mémoire virtuelle max
ulimit -v
```

##### Modifier Limites

```bash
# Fichiers ouverts (soft limit)
ulimit -n 4096

# Hard limit (root)
sudo ulimit -Hn 65536

# Processus max
ulimit -u 2048

# Permanent dans /etc/security/limits.conf
sudo nano /etc/security/limits.conf

# Ajouter
username soft nofile 4096
username hard nofile 65536
username soft nproc 2048
username hard nproc 4096

# Pour tous users
* soft nofile 4096
* hard nofile 65536
```

---

#### cgroups : Contrôle Ressources

Pour une [isolation avancée des ressources avec cgroups v2 et systemd](https://www.shpv.fr/blog/cgroups-systemd-qos/), consultez notre guide dédié.

##### Limiter CPU avec cgroups v2

```bash
# Créer slice systemd pour isoler ressources
sudo mkdir -p /run/systemd/system/limited.slice.d

# Configurer limites CPU (cgroups v2)
sudo tee /run/systemd/system/limited.slice << EOF
[Slice]
CPUWeight=50
MemoryMax=1G
EOF

# Recharger configuration systemd
sudo systemctl daemon-reload

# Lancer processus avec limites
sudo systemd-run --scope --slice=limited.slice ./app

# Attacher processus existant (via cgroup direct)
echo 1234 | sudo tee /sys/fs/cgroup/limited.slice/cgroup.procs

# Vérifier limites appliquées
cat /sys/fs/cgroup/limited.slice/cpu.weight
cat /sys/fs/cgroup/limited.slice/memory.max
```

##### Limiter Mémoire avec cgroups v2

```bash
# Via systemd-run (méthode simple)
sudo systemd-run --scope --slice=limited.slice -p MemoryMax=1G ./app

# Ou configuration permanente via slice
sudo mkdir -p /etc/systemd/system/limited.slice.d
sudo tee /etc/systemd/system/limited.slice.d/override.conf << EOF
[Slice]
MemoryMax=1G
EOF

sudo systemctl daemon-reload
sudo systemd-run --scope --slice=limited.slice ./app

# Vérifier mémoire utilisée par slice
cat /sys/fs/cgroup/limited.slice/memory.current
cat /sys/fs/cgroup/limited.slice/memory.max
```

---

#### Scripts Monitoring

Pour des scripts de monitoring plus avancés, découvrez nos [scripts bash pour sysadmin](https://www.shpv.fr/blog/bash-scripting-sysadmin/).

##### Script Alerte CPU

```bash
#!/bin/bash
# alert-cpu.sh

THRESHOLD=80

while true; do
    CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)

    if (( $(echo "$CPU > $THRESHOLD" | bc -l) )); then
        echo "ALERT: CPU at ${CPU}%"

        # Top 5 processus
        ps aux --sort=-%cpu | head -6

        # Envoyer email
        echo "CPU usage: ${CPU}%" | mail -s "CPU Alert" admin@example.com
    fi

    sleep 60
done
```

##### Script Tuer Processus Zombies

```bash
#!/bin/bash
# kill-zombies.sh

# Trouver zombies
ZOMBIES=$(ps aux | awk '$8=="Z" {print $2}')

if [ -n "$ZOMBIES" ]; then
    echo "Found zombies: $ZOMBIES"

    for zombie in $ZOMBIES; do
        # Trouver parent
        PPID=$(ps -o ppid= -p $zombie)
        echo "Zombie $zombie has parent $PPID"

        # Tuer parent
        kill $PPID
        sleep 1

        # Force si toujours là
        if ps -p $zombie > /dev/null; then
            kill -9 $PPID
        fi
    done
else
    echo "No zombies found"
fi
```

---

#### Bonnes Pratiques

##### Monitoring Régulier

```bash
# Cron job monitoring quotidien
# /etc/cron.daily/system-check

#!/bin/bash
{
    echo "=== System Report $(date) ==="
    echo ""

    echo "Load Average:"
    uptime
    echo ""

    echo "Top 10 CPU:"
    ps aux --sort=-%cpu | head -11
    echo ""

    echo "Top 10 Memory:"
    ps aux --sort=-%mem | head -11
    echo ""

    echo "Zombies:"
    ps aux | grep Z
    echo ""

    echo "Failed Services:"
    systemctl --failed

} | mail -s "Daily System Report" admin@example.com
```

##### Checklist Gestion Processus

```bash
□ Monitoring actif (htop, vmstat)
□ Alertes configurées (CPU, RAM)
□ Services critiques activés au boot
□ Logs vérifiés régulièrement
□ Zombies nettoyés automatiquement
□ Limites ressources appropriées
□ Backup scripts avec nice
□ Documentation processus critiques
```

---

#### Conclusion

La gestion des processus Linux nécessite plusieurs outils complémentaires :

**Visualisation :**

```bash
ps aux           # Vue statique
top              # Monitoring temps réel
htop             # Interface améliorée
```

**Contrôle :**

```bash
kill PID         # Arrêt gracieux
kill -9 PID      # Arrêt forcé
systemctl        # Gestion services
```

**Optimisation :**

```bash
nice -n 10 cmd   # Priorité basse
renice -n 5 PID  # Changer priorité
ulimit -n 4096   # Augmenter limites
```

**Debugging :**

```bash
strace -p PID    # Tracer appels système
lsof -p PID      # Fichiers ouverts
journalctl -u    # Logs service
```

Avec ces outils, vous maîtrisez complètement la gestion des processus sous Linux !