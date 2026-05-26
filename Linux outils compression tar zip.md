La compression est essentielle pour économiser espace disque et bande passante. Ce guide explique comment utiliser tar, gzip, bzip2, xz, zip et 7z pour créer et gérer archives sous Linux.

#### Comprendre la Compression

##### Pourquoi Compresser ?

**Avantages :**

- Économiser espace disque (50-90 % selon format)
- Transferts réseau plus rapides
- Backups plus légers
- Archivage long terme

**Use cases :**

- Backups serveurs
- Transfert fichiers
- Logs rotation
- Distribution code source

##### Compression vs Archivage

**Archivage** (tar) : Regrouper fichiers en un seul **Compression** (gzip, bzip2) : Réduire taille

**Combinaison courante :** `tar + gzip = .tar.gz`

---

#### tar : Archivage Fichiers

##### Syntaxe de Base

```bash
# Créer archive (sans compression)
tar -cf archive.tar fichiers/

# Options principales :
# c - create (créer)
# x - extract (extraire)
# t - list (lister contenu)
# f - file (fichier archive)
# v - verbose (afficher détails)
# z - gzip compression
# j - bzip2 compression
# J - xz compression
```

##### Créer Archives

```bash
# Archive simple (pas de compression)
tar -cf backup.tar /var/www

# Avec verbose (voir fichiers ajoutés)
tar -cvf backup.tar /var/www

# Archive + gzip (.tar.gz ou .tgz)
tar -czf backup.tar.gz /var/www

# Archive + bzip2 (.tar.bz2)
tar -cjf backup.tar.bz2 /var/www

# Archive + xz (.tar.xz)
tar -cJf backup.tar.xz /var/www

# Multiple dossiers/fichiers
tar -czf archive.tar.gz /var/www /etc/nginx /home/user
```

##### Extraire Archives

```bash
# Extraire dans dossier actuel
tar -xf archive.tar

# Extraire .tar.gz
tar -xzf archive.tar.gz

# Extraire .tar.bz2
tar -xjf archive.tar.bz2

# Extraire .tar.xz
tar -xJf archive.tar.xz

# Extraire dans dossier spécifique
tar -xzf backup.tar.gz -C /restore/

# Extraire fichier spécifique
tar -xzf backup.tar.gz var/www/index.html

# Avec verbose
tar -xzvf backup.tar.gz
```

##### Lister Contenu

```bash
# Voir contenu sans extraire
tar -tf archive.tar

# Avec détails
tar -tvf archive.tar

# Filtrer avec grep
tar -tzf backup.tar.gz | grep ".conf"
```

##### Options Avancées

```bash
# Exclure fichiers
tar -czf backup.tar.gz /var/www --exclude='*.log' --exclude='cache/*'

# Exclure pattern depuis fichier
echo "*.log" > exclude.txt
echo "node_modules" >> exclude.txt
tar -czf backup.tar.gz /project -X exclude.txt

# Préserver permissions
tar -czpf backup.tar.gz /var/www
# p = preserve permissions

# Update archive (ajouter si plus récent) - UNIQUEMENT sur archives non compressées
tar -uf backup.tar newfile.txt

# Append fichiers à archive existante - UNIQUEMENT sur archives non compressées
tar -rf backup.tar additional.txt

# ATTENTION: -u et -r ne fonctionnent PAS avec .tar.gz (.tar.bz2, .tar.xz, etc.)
# Pour modifier une archive compressée, vous devez:
# 1. Décompresser: tar -xzf backup.tar.gz
# 2. Modifier: tar -uf backup.tar newfile.txt
# 3. Recompresser: tar -czf backup.tar.gz backup.tar

# Différence entre archive et filesystem
tar -df backup.tar /var/www

# Archive avec date dans nom
tar -czf backup-$(date +%Y%m%d).tar.gz /var/www
```

---

#### gzip : Compression Rapide

##### Utilisation gzip

```bash
# Compresser fichier (remplace original par .gz)
gzip file.txt
# Crée : file.txt.gz

# Garder original
gzip -k file.txt

# Décompresser
gunzip file.txt.gz
# Ou
gzip -d file.txt.gz

# Niveau compression (1-9)
gzip -1 file.txt  # Rapide, moins compressé
gzip -9 file.txt  # Lent, plus compressé
gzip -6 file.txt  # Défaut

# Compresser récursivement
gzip -r dossier/

# Voir infos sans décompresser
gzip -l file.txt.gz
```

##### gzip avec Pipes

```bash
# Compresser output commande
mysqldump database | gzip > backup.sql.gz

# Décompresser et lire
zcat file.txt.gz
# Ou
gunzip -c file.txt.gz

# Grep dans fichier compressé
zgrep "error" logfile.gz

# Less sur fichier compressé
zless file.txt.gz

# Diff fichiers compressés
zdiff file1.txt.gz file2.txt.gz
```

---

#### bzip2 : Meilleure Compression

##### Utilisation bzip2

```bash
# Compresser (plus lent que gzip, meilleur ratio)
bzip2 file.txt
# Crée : file.txt.bz2

# Garder original
bzip2 -k file.txt

# Décompresser
bunzip2 file.txt.bz2
# Ou
bzip2 -d file.txt.bz2

# Niveau compression (1-9)
bzip2 -9 file.txt

# Compresser avec verbose
bzip2 -v file.txt

# Test intégrité
bzip2 -t file.txt.bz2
```

##### bzip2 Outils

```bash
# Lire sans décompresser
bzcat file.txt.bz2

# Grep
bzgrep "pattern" file.txt.bz2

# Less
bzless file.txt.bz2

# Diff
bzdiff file1.txt.bz2 file2.txt.bz2
```

---

#### xz : Compression Maximale

##### Utilisation xz

```bash
# Compresser (meilleur ratio, très lent)
xz file.txt
# Crée : file.txt.xz

# Garder original
xz -k file.txt

# Décompresser
unxz file.txt.xz
# Ou
xz -d file.txt.xz

# Niveaux compression (0-9)
xz -0 file.txt  # Rapide
xz -9 file.txt  # Maximum (défaut : -6)

# Extreme mode (encore meilleur)
xz -9e file.txt

# Multi-thread
xz -T 4 file.txt  # 4 threads
xz -T 0 file.txt  # Tous cores disponibles

# Test intégrité
xz -t file.txt.xz
```

##### xz Outils

```bash
# Lire
xzcat file.txt.xz

# Grep
xzgrep "pattern" file.txt.xz

# Less
xzless file.txt.xz
```

---

#### zip : Compatibilité Multi-OS

##### Créer Archives ZIP

```bash
# Archive ZIP simple
zip archive.zip file1.txt file2.txt

# Récursif (dossiers)
zip -r archive.zip dossier/

# Avec compression max
zip -9 -r archive.zip dossier/

# Sans compression (store)
zip -0 -r archive.zip dossier/

# Exclure fichiers
zip -r archive.zip dossier/ -x "*.log" "cache/*"

# Update archive
zip -u archive.zip newfile.txt

# Ajouter avec chemins relatifs
cd /var/www
zip -r /backup/web.zip .

# Protéger par mot de passe
zip -e -r secure.zip dossier/
# Demande mot de passe

# Splitter archive (100MB chunks)
zip -s 100m -r archive.zip bigsoftware/
```

##### Extraire ZIP

```bash
# Extraire tout
unzip archive.zip

# Dans dossier spécifique
unzip archive.zip -d /restore/

# Lister contenu
unzip -l archive.zip

# Test intégrité
unzip -t archive.zip

# Extraire fichier spécifique
unzip archive.zip file.txt

# Extraire avec pattern
unzip archive.zip "*.conf"

# Quiet mode
unzip -q archive.zip

# Overwrite sans demander
unzip -o archive.zip
```

##### zipinfo

```bash
# Infos détaillées archive
zipinfo archive.zip

# Court format
zipinfo -1 archive.zip

# Avec dates
zipinfo -T archive.zip
```

---

#### 7z : Compression Ultime

##### Installer 7zip

```bash
# Debian/Ubuntu
sudo apt install p7zip-full

# RedHat/CentOS/Rocky/AlmaLinux
sudo dnf install epel-release -y
sudo dnf install p7zip p7zip-plugins -y
```

##### Créer Archives 7z

```bash
# Archive 7z
7z a archive.7z fichiers/

# Compression maximale
7z a -mx=9 archive.7z fichiers/

# Multi-thread
7z a -mmt=4 archive.7z fichiers/

# Mot de passe + chiffrement noms
7z a -p -mhe=on secure.7z fichiers/

# Splitter (100MB)
7z a -v100m archive.7z bigfiles/

# Format ZIP avec 7z
7z a -tzip archive.zip fichiers/
```

##### Extraire 7z

```bash
# Extraire
7z x archive.7z

# Dans dossier
7z x archive.7z -o/restore/

# Extract sans structure dossiers
7z e archive.7z

# Lister contenu
7z l archive.7z

# Test intégrité
7z t archive.7z
```

---

#### zstd : Moderne et Rapide

##### Utilisation zstd

```bash
# Installer
sudo apt install zstd

# Compresser
zstd file.txt
# Crée : file.txt.zst

# Garder original
zstd -k file.txt

# Décompresser
unzstd file.txt.zst
# Ou
zstd -d file.txt.zst

# Niveaux (1-19, défaut 3)
zstd -1 file.txt  # Rapide
zstd -19 file.txt # Max

# Ultra mode (20-22)
zstd --ultra -22 file.txt

# Multi-thread
zstd -T0 file.txt

# Tar + zstd
tar --zstd -cf archive.tar.zst dossier/
tar --zstd -xf archive.tar.zst
```

---

#### Comparaison Formats

##### Benchmark Compression

**Test sur 1GB fichiers logs :**

|Format|Taille|Temps|Ratio|Décompression|
|---|---|---|---|---|
|gzip -6|250 MB|15s|75 %|5s|
|bzip2 -9|200 MB|45s|80 %|20s|
|xz -6|180 MB|90s|82 %|10s|
|xz -9|170 MB|180s|83 %|12s|
|7z -mx=9|165 MB|120s|83.5 %|15s|
|zstd -3|260 MB|8s|74 %|3s|
|zstd -19|190 MB|60s|81 %|4s|

##### Choisir Format

**gzip** : Standard, rapide, compatible partout

- Logs quotidiens
- Backups fréquents
- Bon compromis vitesse/ratio

**bzip2** : Meilleur ratio que gzip

- Archives moyen terme
- Moins utilisé maintenant

**xz** : Meilleure compression

- Distribution software (Linux kernels)
- Archives long terme
- Bande passante limitée

**zip** : Compatibilité Windows/Mac

- Partage multi-OS
- Moins bon que tar.gz

**7z** : Maximum compression

- Archives très volumineuses
- Storage long terme

**zstd** : Moderne, très rapide

- Backups temps réel
- Streaming data
- Meilleur équilibre vitesse/ratio

---

#### Scripts Automatisation

##### Backup Automatique

```bash
#!/bin/bash
# /usr/local/bin/backup.sh

DATE=$(date +%Y%m%d)
BACKUP_DIR="/backup"
SOURCE="/var/www"

# Créer archive
tar -czf "$BACKUP_DIR/www-$DATE.tar.gz" "$SOURCE"

# Garder seulement 7 derniers jours
find "$BACKUP_DIR" -name "www-*.tar.gz" -mtime +7 -delete

# Vérifier taille
SIZE=$(du -h "$BACKUP_DIR/www-$DATE.tar.gz" | cut -f1)
echo "Backup created: www-$DATE.tar.gz ($SIZE)"
```

##### Backup Incrémental

```bash
#!/bin/bash
# Backup incrémental avec tar

FULL_BACKUP="/backup/full-$(date +%Y%m%d).tar.gz"
SNAPSHOT="/backup/snapshot.snar"

# Full backup si snapshot n'existe pas
if [ ! -f "$SNAPSHOT" ]; then
    tar -czf "$FULL_BACKUP" -g "$SNAPSHOT" /var/www
else
    # Incrémental
    INCR_BACKUP="/backup/incr-$(date +%Y%m%d-%H%M).tar.gz"
    tar -czf "$INCR_BACKUP" -g "$SNAPSHOT" /var/www
fi
```

##### Compression Logs Anciens

```bash
#!/bin/bash
# Compresser logs > 1 jour

find /var/log -name "*.log" -mtime +1 -exec gzip {} \;

# Supprimer logs compressés > 30 jours
find /var/log -name "*.log.gz" -mtime +30 -delete
```

##### Backup Base Données

```bash
#!/bin/bash
# Backup MySQL compressé

DATE=$(date +%Y%m%d-%H%M)
DB="mydb"

mysqldump "$DB" | gzip > "/backup/db-$DB-$DATE.sql.gz"

# Rotation 14 jours
find /backup -name "db-$DB-*.sql.gz" -mtime +14 -delete
```

---

#### Extraction Intelligente

##### Script Extract Universel

```bash
#!/bin/bash
# extract.sh - Extraire n'importe quel format

extract() {
    if [ -f "$1" ]; then
        case "$1" in
            *.tar.gz|*.tgz)   tar -xzf "$1"     ;;
            *.tar.bz2|*.tbz2) tar -xjf "$1"     ;;
            *.tar.xz|*.txz)   tar -xJf "$1"     ;;
            *.tar.zst)        tar --zstd -xf "$1" ;;
            *.tar)            tar -xf "$1"      ;;
            *.gz)             gunzip "$1"       ;;
            *.bz2)            bunzip2 "$1"      ;;
            *.xz)             unxz "$1"         ;;
            *.zip)            unzip "$1"        ;;
            *.7z)             7z x "$1"         ;;
            *.rar)            unrar x "$1"      ;;
            *.zst)            unzstd "$1"       ;;
            *)                echo "Format non supporté: $1" ;;
        esac
    else
        echo "Fichier introuvable: $1"
    fi
}

extract "$1"
```

##### Alias Bash

```bash
# Ajouter dans ~/.bashrc

# Extraction automatique
extract() {
    if [ -f $1 ]; then
        case $1 in
            *.tar.*)  tar -xf $1      ;;
            *.gz)     gunzip $1       ;;
            *.zip)    unzip $1        ;;
            *.7z)     7z x $1         ;;
        esac
    fi
}

# Backup rapide
backup() {
    tar -czf "$1-$(date +%Y%m%d).tar.gz" "$1"
}
```

---

#### Performance et Optimisation

##### Multi-Threading

```bash
# pigz (parallel gzip)
sudo apt install pigz

# Utilise tous cores
pigz file.txt
tar -I pigz -cf archive.tar.gz dossier/

# pbzip2 (parallel bzip2)
sudo apt install pbzip2
pbzip2 file.txt
tar -I pbzip2 -cf archive.tar.bz2 dossier/

# xz multi-thread
xz -T 0 file.txt  # Auto threads
```

##### Compression par Blocs

```bash
# zstd avec dictionnaire (meilleur ratio)
zstd --train files/*.txt -o dict
zstd -D dict file.txt
```

##### Benchmark Custom

```bash
#!/bin/bash
# Comparer formats sur fichier

FILE="testfile.bin"
echo "Testing compression on: $FILE"

# gzip
time gzip -k -6 $FILE
ls -lh $FILE.gz

# bzip2
time bzip2 -k -9 $FILE
ls -lh $FILE.bz2

# xz
time xz -k -6 $FILE
ls -lh $FILE.xz

# zstd
time zstd -k $FILE
ls -lh $FILE.zst

# Cleanup
rm $FILE.{gz,bz2,xz,zst}
```

---

#### Troubleshooting

##### Archive Corrompue

```bash
# Tester intégrité
gzip -t file.gz
bzip2 -t file.bz2
xz -t file.xz
unzip -t file.zip
7z t file.7z

# Réparer si possible (zip)
zip -F broken.zip --out fixed.zip
```

##### Espace Disque Insuffisant

```bash
# Compresser sur place (remplace original)
gzip largefile.log

# Stream vers autre disque
tar -czf - /var/www | ssh remote "cat > backup.tar.gz"

# Split archive
split -b 1G largefile.tar.gz part-
# Reconstituer
cat part-* > largefile.tar.gz
```

##### Extraction Lente

```bash
# Extraire seulement fichiers nécessaires
tar -xzf backup.tar.gz specific/file.txt

# Paralléliser extraction
unpigz file.gz  # Plus rapide que gunzip
```

---

#### Bonnes Pratiques

##### Nommage Archives

```bash
# Format recommandé :
project-YYYYMMDD.tar.gz
backup-full-20260125.tar.gz
db-incremental-20260125-1430.tar.gz

# Éviter espaces et caractères spéciaux
# Bon : web-backup-20260125.tar.gz
# Mauvais : Web Backup (Jan 25).tar.gz
```

##### Checksums

```bash
# Créer checksum avec archive
tar -czf backup.tar.gz /data
sha256sum backup.tar.gz > backup.tar.gz.sha256

# Vérifier avant extraction
sha256sum -c backup.tar.gz.sha256
```

##### Checklist Backup

```bash
□ Format adapté (gzip quotidien, xz long terme)
□ Rotation automatique (7j, 30j, 1an)
□ Test extraction régulier
□ Checksums générés
□ Stockage redondant (local + remote)
□ Monitoring espace disque
□ Documentation procédure restore
```

---

#### Lectures avancées

Pour un deep dive sur zstd, consultez [compression Zstandard en détail](https://www.shpv.fr/blog/zstd-compression/). Pour les sauvegardes, découvrez [backups avec tar](https://www.shpv.fr/blog/backup-system/) et [backups avec rsync](https://www.shpv.fr/blog/backup-rsync-321/).

#### Conclusion

La compression Linux offre de nombreux outils :

**Usage quotidien :**

```bash
tar -czf backup.tar.gz /data      # Archive + gzip
tar -xzf backup.tar.gz            # Extraire
```

**Formats par use case :**

- **gzip** : Standard, rapide, quotidien
- **xz** : Distribution, long terme
- **zstd** : Moderne, meilleur équilibre
- **zip** : Compatibilité Windows
- **7z** : Maximum compression

**Automatisation :**

```bash
# Backup quotidien avec rotation
tar -czf backup-$(date +%Y%m%d).tar.gz /data
find /backup -mtime +7 -delete
```

**Performance :**

```bash
pigz -k file.txt          # Parallel gzip
xz -T 0 file.txt          # Multi-thread xz
zstd -19 file.txt         # Meilleur ratio rapide
```

Avec ces outils, vous gérez efficacement compression et archivage sous Linux !Tmux est un multiplexeur terminal qui permet de gérer plusieurs sessions, windows et panes dans un seul terminal. Ce guide complet explique comment maîtriser Tmux pour maximiser votre productivité.

#### Comprendre Tmux

##### Qu'est-ce que Tmux ?

**Tmux** (Terminal Multiplexer) permet de :

- Diviser terminal en plusieurs panes
- Créer windows (onglets)
- Sessions persistantes (survivent à la déconnexion SSH)
- Partager sessions (pair programming)

**Analogie :** Tmux = gestionnaire fenêtres pour terminal

##### Pourquoi Tmux ?

**Avantages :**

- Sessions survivent à la déconnexion SSH
- Workflow multi-tâches efficace
- Pas besoin de GUI (serveurs)
- Scripts d'automatisation
- Collaboration en temps réel

**Cas usage :**

- Administration serveurs SSH
- Développement remote
- Monitoring multiple services
- Pair programming

Pour [personnaliser votre environnement Bash](https://www.shpv.fr/blog/bash-configuration/), combiné avec Tmux donne un shell vraiment exploitable au quotidien. Découvrez aussi [bash scripting avancé](https://www.shpv.fr/blog/bash-scripting-sysadmin/) pour créer des scripts complexes à lancer via Tmux.

---

#### Installation

##### Installer Tmux

```bash
# Debian/Ubuntu
sudo apt install tmux

# RedHat/CentOS
sudo yum install tmux

# macOS
brew install tmux

# Vérifier version
tmux -V
# tmux 3.3a
```

---

#### Concepts Fondamentaux

##### Hiérarchie Tmux

```
Session (projet)
  └── Window (terminal/onglet)
        └── Pane (split terminal)
```

**Exemple :**

```
Session "webapp"
  ├── Window 0: "editor"
  │     ├── Pane: vim
  │     └── Pane: terminal
  ├── Window 1: "servers"
  │     ├── Pane: npm run dev
  │     └── Pane: logs
  └── Window 2: "db"
        └── Pane: mysql
```

##### Prefix Key

**Toutes commandes Tmux** commencent par **prefix** (défaut : `Ctrl+b`)

**Syntaxe :** `Prefix` puis `commande`

Exemple : `Ctrl+b` puis `c` = créer window

---

#### Commandes de Base

##### Sessions

```bash
# Créer session
tmux new -s myproject

# Créer session nommée + commande
tmux new -s monitoring -d 'htop'

# Lister sessions
tmux ls

# Attacher session
tmux attach -t myproject
# Raccourci
tmux a -t myproject

# Détacher session (depuis tmux)
Prefix + d

# Tuer session
tmux kill-session -t myproject

# Tuer toutes sessions
tmux kill-server
```

##### Windows (onglets)

```bash
# Dans tmux :

# Créer window
Prefix + c

# Renommer window
Prefix + ,

# Naviguer windows
Prefix + n    # Next
Prefix + p    # Previous
Prefix + 0-9  # Window numéro
Prefix + l    # Last window

# Lister windows
Prefix + w

# Fermer window
Prefix + &    # Avec confirmation
# Ou
exit
```

##### Panes (splits)

```bash
# Split horizontal (haut/bas)
Prefix + "

# Split vertical (gauche/droite)
Prefix + %

# Naviguer panes
Prefix + ↑↓←→  # Flèches
Prefix + o     # Cycle panes
Prefix + q     # Numéros panes (puis taper numéro)

# Zoom pane (plein écran)
Prefix + z

# Fermer pane
Prefix + x    # Avec confirmation
# Ou
exit

# Redimensionner pane
Prefix puis maintenir Ctrl + ↑↓←→

# Convertir pane en window
Prefix + !

# Déplacer pane vers window
Prefix + :
move-pane -t target_window
```

---

#### Configuration .tmux.conf

##### Fichier Configuration

```bash
# Créer config
nano ~/.tmux.conf
```

##### Configuration Recommandée

```bash
# ~/.tmux.conf

# ====================
# Prefix
# ====================
# Changer prefix (Ctrl+a plus ergonomique)
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# ====================
# Général
# ====================
# Numérotation à partir de 1 (plus logique)
set -g base-index 1
setw -g pane-base-index 1

# Renommer automatiquement windows
setw -g automatic-rename on
set -g renumber-windows on

# Historique
set -g history-limit 10000

# Mode mouse
set -g mouse on

# Pas de délai ESC (Vim)
set -sg escape-time 0

# Notifications
setw -g monitor-activity on
set -g visual-activity off

# ====================
# Splits
# ====================
# Splits intuitifs (même répertoire)
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
unbind '"'
unbind %

# Nouvelle window même répertoire
bind c new-window -c "#{pane_current_path}"

# ====================
# Navigation Panes (style Vim)
# ====================
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Redimensionner panes
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# ====================
# Copy Mode (Vim style)
# ====================
setw -g mode-keys vi
bind [ copy-mode
bind -T copy-mode-vi v send -X begin-selection
bind -T copy-mode-vi y send -X copy-selection-and-cancel
bind -T copy-mode-vi C-v send -X rectangle-toggle
bind ] paste-buffer

# ====================
# Apparence
# ====================
# 256 colors
set -g default-terminal "screen-256color"
set -ga terminal-overrides ",xterm-256color:Tc"

# Status bar
set -g status-position bottom
set -g status-justify left
set -g status-style 'bg=colour234 fg=colour137'
set -g status-left ''
set -g status-right '#[fg=colour233,bg=colour241] %d/%m #[fg=colour233,bg=colour245] %H:%M:%S '
set -g status-right-length 50
set -g status-left-length 20

# Window status
setw -g window-status-current-style 'fg=colour81 bg=colour238 bold'
setw -g window-status-current-format ' #I#[fg=colour250]:#[fg=colour255]#W#[fg=colour50]#F '
setw -g window-status-style 'fg=colour138 bg=colour235'
setw -g window-status-format ' #I#[fg=colour237]:#[fg=colour250]#W#[fg=colour244]#F '

# Pane borders
set -g pane-border-style 'fg=colour238'
set -g pane-active-border-style 'fg=colour51'

# Messages
set -g message-style 'fg=colour232 bg=colour166 bold'

# ====================
# Reload config
# ====================
bind r source-file ~/.tmux.conf \; display "Config reloaded!"
```

##### Appliquer Configuration

```bash
# Depuis tmux
Prefix + :
source-file ~/.tmux.conf

# Ou raccourci (si configuré)
Prefix + r
```

---

#### Workflows Productifs

##### Workflow Développement Web

```bash
# Session projet
tmux new -s webapp

# Window 1: Editor (splits)
Prefix + | # Split vertical
# Gauche: vim
# Droite: terminal

# Window 2: Servers
Prefix + c
Prefix + - # Split horizontal
# Haut: npm run dev
# Bas: npm run build:watch

# Window 3: Logs
Prefix + c
Prefix + | # Split vertical
# Gauche: tail -f app.log
# Droite: tail -f error.log

# Window 4: Database
Prefix + c
# mysql client

# Naviguer
Prefix + 0-3  # Windows
Prefix + h/j/k/l  # Panes
```

##### Workflow Administration Serveur

```bash
# Session monitoring
tmux new -s monitor

# Window 1: System
Prefix + " # Split 4 panes
# Pane 1: htop
# Pane 2: iostat -x 2
# Pane 3: free -h
# Pane 4: df -h

# Window 2: Network
Prefix + c
Prefix + "
# Pane 1: iftop
# Pane 2: netstat -tulpn

# Window 3: Logs
Prefix + c
tail -f /var/log/syslog

# Détacher et laisser tourner
Prefix + d
```

---

#### Copy Mode

##### Mode Copie

```bash
# Entrer copy mode
Prefix + [

# Navigation (Vim keys si configuré)
h/j/k/l    # Déplacer curseur
Ctrl+u     # Page up
Ctrl+d     # Page down
g          # Début buffer
G          # Fin buffer

# Sélectionner texte
Space      # Début sélection (ou v si Vim mode)
Enter      # Copier sélection

# Rechercher
/          # Rechercher forward
?          # Rechercher backward
n          # Next occurrence
N          # Previous

# Quitter
q ou Esc

# Coller
Prefix + ]
```

##### Copier dans Clipboard Système

```bash
# Installer xclip (Linux)
sudo apt install xclip

# Config tmux
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel 'xclip -in -selection clipboard'

# macOS (pbcopy)
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel 'pbcopy'
```

---

#### Sessions Persistantes

##### Workflow SSH

```bash
# Se connecter serveur
ssh user@server

# Créer session tmux
tmux new -s work

# Travailler...

# Déconnexion SSH (volontaire ou pas)
Prefix + d
# Ou connexion coupée

# Re-connexion SSH
ssh user@server

# Retrouver session
tmux attach -t work

# Tout est là où vous l'avez laissé !
```

##### Multiple Sessions

```bash
# Créer plusieurs sessions
tmux new -s project1
tmux new -s project2
tmux new -s monitoring

# Lister
tmux ls
# project1: 3 windows
# project2: 2 windows
# monitoring: 1 windows

# Switcher entre sessions (depuis tmux)
Prefix + s    # Menu sessions
# Choisir avec ↑↓, Enter

# Ou depuis shell
tmux switch -t project2
```

---

#### Scripting Tmux

Pour [automatiser avec des scripts bash](https://www.shpv.fr/blog/bash-scripting-sysadmin/), consultez notre guide complet.

##### Script Startup Projet

```bash
#!/bin/bash
# ~/scripts/start-webapp.sh

SESSION="webapp"

# Tuer si existe
tmux has-session -t $SESSION 2>/dev/null
if [ $? == 0 ]; then
    tmux kill-session -t $SESSION
fi

# Créer session
tmux new-session -d -s $SESSION -n editor

# Window 1: Editor
tmux send-keys -t $SESSION:editor "cd ~/projects/webapp" C-m
tmux send-keys -t $SESSION:editor "vim" C-m
tmux split-window -h -t $SESSION:editor
tmux send-keys -t $SESSION:editor "cd ~/projects/webapp" C-m

# Window 2: Servers
tmux new-window -t $SESSION -n servers
tmux send-keys -t $SESSION:servers "cd ~/projects/webapp" C-m
tmux send-keys -t $SESSION:servers "npm run dev" C-m
tmux split-window -h -t $SESSION:servers
tmux send-keys -t $SESSION:servers "cd ~/projects/webapp" C-m
tmux send-keys -t $SESSION:servers "npm run logs" C-m

# Window 3: Git
tmux new-window -t $SESSION -n git
tmux send-keys -t $SESSION:git "cd ~/projects/webapp" C-m
tmux send-keys -t $SESSION:git "git status" C-m

# Retour window 1
tmux select-window -t $SESSION:editor

# Attacher
tmux attach -t $SESSION
```

```bash
# Utiliser
chmod +x ~/scripts/start-webapp.sh
~/scripts/start-webapp.sh
```

##### Tmuxinator (Tool de Config)

```bash
# Installer
gem install tmuxinator

# Ou via package manager
sudo apt install tmuxinator

# Créer config projet
tmuxinator new webapp

# Éditer ~/.config/tmuxinator/webapp.yml
name: webapp
root: ~/projects/webapp

windows:
  - editor:
      layout: main-vertical
      panes:
        - vim
        -
  - servers:
      layout: even-horizontal
      panes:
        - npm run dev
        - npm run logs
  - git:
      panes:
        - git status

# Démarrer
tmuxinator start webapp
# Raccourci
mux webapp
```

---

#### Pair Programming

##### Partager Session

```bash
# User 1 (host) : Créer session
tmux new -s pair

# User 2 : SSH même serveur
ssh user@server

# Attacher même session
tmux attach -t pair

# Les deux voient/contrôlent même terminal !
```

##### Session Read-Only

```bash
# User 2 : Attacher en lecture seule
tmux attach -t pair -r

# Peut voir, pas modifier
```

##### Tmate (Partage Internet)

```bash
# Installer tmate
sudo apt install tmate

# Démarrer session
tmate

# Obtenir lien partage
# Affiche automatiquement :
# ssh session: ssh ABC123@ny2.tmate.io
# web session: https://tmate.io/t/ABC123

# Partager lien à collaborateur
# Il peut se connecter sans compte serveur !
```

---

#### Plugins Tmux

##### TPM (Tmux Plugin Manager)

```bash
# Installer TPM
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# Ajouter à ~/.tmux.conf
# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'

# Initialize TPM (en fin fichier)
run '~/.tmux/plugins/tpm/tpm'

# Reload config
Prefix + r

# Installer plugins
Prefix + I
```

##### Plugins Utiles

```bash
# ~/.tmux.conf

# Sensible defaults
set -g @plugin 'tmux-plugins/tmux-sensible'

# Copier dans clipboard
set -g @plugin 'tmux-plugins/tmux-yank'

# Sauvegarder/restaurer sessions
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @continuum-restore 'on'

# Navigation panes + Vim
set -g @plugin 'christoomey/vim-tmux-navigator'

# Thème
set -g @plugin 'dracula/tmux'
```

---

#### Tmux + Vim Integration

##### Navigation Seamless

```bash
# Plugin : vim-tmux-navigator

# Installer dans Vim (.vimrc)
Plug 'christoomey/vim-tmux-navigator'

# Installer dans Tmux (.tmux.conf)
set -g @plugin 'christoomey/vim-tmux-navigator'

# Navigation avec Ctrl+h/j/k/l
# Fonctionne entre panes Tmux ET splits Vim !
```

##### Vim dans Tmux

```bash
# Fix couleurs Vim
# ~/.tmux.conf
set -g default-terminal "screen-256color"
set -ga terminal-overrides ",xterm-256color:Tc"

# ~/.vimrc
set termguicolors
```

---

#### Troubleshooting

##### Colors Incorrects

```bash
# Test colors
tmux info | grep Tc

# Fix ~/.tmux.conf
set -g default-terminal "screen-256color"
set -ga terminal-overrides ",xterm-256color:Tc"

# Test
echo $TERM
# Devrait être screen-256color
```

##### Mouse Ne Fonctionne Pas

```bash
# Activer mouse
# ~/.tmux.conf
set -g mouse on

# Reload
Prefix + :
source-file ~/.tmux.conf
```

##### Sessions Perdues Après Reboot

```bash
# Sauvegarder sessions automatiquement
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @continuum-restore 'on'

# Sauvegarder manuellement
Prefix + Ctrl+s

# Restaurer
Prefix + Ctrl+r
```

---

#### Astuces Avancées

##### Status Bar Personnalisée

```bash
# ~/.tmux.conf
set -g status-right '#[fg=yellow]#(uptime | cut -d "," -f 3-) #[default] #[fg=cyan]%H:%M#[default]'

# Variables utiles :
# #H - hostname
# #S - session name
# #I - window index
# #W - window name
# #T - pane title
```

##### Synchroniser Panes

```bash
# Envoyer commandes à tous panes simultanément
Prefix + :
setw synchronize-panes on

# Taper commande → exécutée partout

# Désactiver
setw synchronize-panes off

# Raccourci
bind e setw synchronize-panes
```

##### Layouts Prédéfinis

```bash
# Cycle layouts
Prefix + Space

# Layouts disponibles :
even-horizontal    # Panes côte à côte égaux
even-vertical      # Panes empilés égaux
main-horizontal    # 1 grand + petits en bas
main-vertical      # 1 grand + petits à droite
tiled             # Grid égale
```

---

#### Cheat Sheet

##### Commandes Essentielles

```bash
# Sessions
tmux new -s name        # Créer
tmux ls                 # Lister
tmux a -t name          # Attacher
Prefix + d              # Détacher
tmux kill-session -t    # Tuer

# Windows
Prefix + c              # Créer
Prefix + ,              # Renommer
Prefix + n/p            # Next/Previous
Prefix + 0-9            # Numéro
Prefix + &              # Fermer

# Panes
Prefix + %              # Split vertical
Prefix + "              # Split horizontal
Prefix + o              # Cycle
Prefix + ↑↓←→           # Naviguer
Prefix + z              # Zoom
Prefix + x              # Fermer

# Autres
Prefix + [              # Copy mode
Prefix + ]              # Paste
Prefix + ?              # Liste shortcuts
Prefix + t              # Horloge
```

---

#### Bonnes Pratiques

##### Organisation Sessions

```bash
# Session par projet/contexte
tmux new -s work
tmux new -s personal
tmux new -s learning

# Windows par tâche
0: editor
1: servers
2: tests
3: git
```

##### Naming Convention

```bash
# Nommer explicitement
tmux new -s webapp-dev
tmux new -s monitoring-prod
tmux new -s db-maintenance

# Renommer windows
Prefix + , → taper nom
```

##### Checklist Tmux

```bash
□ Config .tmux.conf personnalisée
□ Prefix ergonomique (Ctrl+a)
□ Mouse enabled
□ Vim keybindings si Vim user
□ Auto-save sessions (resurrect)
□ Scripts startup projets fréquents
□ Status bar utile
```

---

#### Conclusion

Tmux booste productivité terminal :

**Démarrer :**

```bash
tmux new -s myproject
Prefix + |    # Split
Prefix + c    # New window
Prefix + d    # Detach
```

**Configuration :**

```bash
~/.tmux.conf
# Prefix, colors, shortcuts
Prefix + r    # Reload
```

**Workflows :**

```bash
# Dev: editor + servers + logs
# Admin: monitoring multi-panes
# Pair: session partagée
```

**Persistance :**

```bash
# Sessions survivent SSH disconnect
tmux attach -t myproject
```

**Scripting :**

```bash
# Automatiser startup
# Tmuxinator pour projets
```

Avec Tmux, terminal devient IDE puissant et flexible ! 🚀