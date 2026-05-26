---
tags:
  - tmux
  - linux
---
Tmux est un multiplexeur terminal qui permet de gérer plusieurs sessions, windows et panes dans un seul terminal. Ce guide complet explique comment maîtriser Tmux pour maximiser votre productivité.

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