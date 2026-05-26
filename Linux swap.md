---
tags:
  - linux
  - swap
---
Le swap est un espace disque utilisé comme extension de la RAM. Ce guide complet explique comment configurer, optimiser et gérer le swap sous Linux pour maximiser les performances système.

#### Qu'est-ce que le Swap ?

##### Définition Simple

Le **swap** (espace d'échange) est une partie du disque dur utilisée comme mémoire virtuelle quand la RAM physique est pleine.

**Analogie :** La RAM est votre bureau de travail, le swap est une étagère à côté. Quand votre bureau est plein, vous mettez temporairement des documents sur l'étagère (plus lent d'accès, mais permet de continuer à travailler).

##### Comment Fonctionne le Swap

```
┌─────────────────────────────────────────┐
│         RAM (8 GB)                      │
│  ┌──────────────────────────────────┐  │
│  │ Processus actifs                 │  │
│  │ - Firefox : 2 GB                 │  │
│  │ - Chrome : 1.5 GB                │  │
│  │ - IDE : 1 GB                     │  │
│  │ - Système : 3 GB                 │  │
│  │ Total utilisé : 7.5 GB           │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
              ↓ RAM pleine !
┌─────────────────────────────────────────┐
│         SWAP (4 GB sur disque)          │
│  ┌──────────────────────────────────┐  │
│  │ Pages mémoire inactives          │  │
│  │ - Onglets Firefox pas utilisés   │  │
│  │ - Processus en veille            │  │
│  │ Utilisé : 1.2 GB                 │  │
│  └──────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

##### Quand Utiliser le Swap ?

**Scénarios où le swap est utile :**

- RAM limitée (moins de 8 GB)
- Workstation avec beaucoup d'applications
- Serveur avec pics de charge occasionnels
- Hibernation (suspend-to-disk)
- Sécurité : éviter OOM killer

**Scénarios où le swap est moins utile :**

- Serveur avec RAM abondante (32+ GB)
- Applications critiques temps réel
- SSD de faible endurance

---

#### Vérifier Configuration Swap Actuelle

##### Commandes de Base

```bash
# Voir swap actif
swapon --show
# NAME      TYPE      SIZE USED PRIO
# /swapfile file      2G   0B   -2

# Alternative
free -h
#               total   used   free   shared  buff/cache  available
# Mem:          7.7Gi   2.1Gi  3.2Gi  156Mi   2.4Gi       5.2Gi
# Swap:         2.0Gi   0B     2.0Gi

# Détails swap
cat /proc/swaps
# Filename     Type       Size      Used  Priority
# /swapfile    file       2097148   0     -2

# Utilisation mémoire complète
vmstat 1
# Colonne 'si' = swap in (KB/s)
# Colonne 'so' = swap out (KB/s)
```

##### Comprendre les Valeurs

```bash
# Si 'so' (swap out) > 0 de façon continue
# → Système manque de RAM, swap activement

# Si 'si' (swap in) élevé
# → Processus réactivent données swappées (lent)

# Voir processus qui swappent
for file in /proc/*/status; do
    awk '/VmSwap|Name/{printf $2 " " $3}END{print ""}' $file
done | grep -v "0 kB" | sort -k2 -n -r | head -10
```

---

#### Créer du Swap

##### Méthode 1 : Partition Swap

**Avantages :** Performance légèrement meilleure, dédié

**Inconvénients :** Nécessite partition libre, taille fixe

```bash
# Lister partitions
sudo fdisk -l

# Créer partition avec fdisk
sudo fdisk /dev/sda

# Dans fdisk :
# n (nouvelle partition)
# p (primaire)
# Taille : +4G (exemple)
# t (changer type)
# 82 (Linux swap)
# w (écrire et quitter)

# Créer système de fichiers swap
sudo mkswap /dev/sda3

# Activer swap
sudo swapon /dev/sda3

# Vérifier
swapon --show

# Rendre permanent dans /etc/fstab
echo '/dev/sda3 none swap sw 0 0' | sudo tee -a /etc/fstab
```

##### Méthode 2 : Fichier Swap (Recommandé)

**Avantages :** Flexible, redimensionnable, pas besoin partition

**Inconvénients :** Légère surcharge performance (négligeable)

```bash
# Créer fichier 4 GB
sudo fallocate -l 4G /swapfile
# Alternative si fallocate pas disponible
# sudo dd if=/dev/zero of=/swapfile bs=1M count=4096

# Permissions strictes (sécurité)
sudo chmod 600 /swapfile

# Vérifier
ls -lh /swapfile
# -rw------- 1 root root 4.0G

# Créer système swap
sudo mkswap /swapfile

# Activer
sudo swapon /swapfile

# Vérifier activation
swapon --show
free -h

# Rendre permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

##### Redimensionner Fichier Swap

```bash
# Désactiver swap actuel
sudo swapoff /swapfile

# Supprimer ancien
sudo rm /swapfile

# Créer nouveau (8 GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Vérifier
free -h
```

---

#### Configuration Swappiness

Consultez notre [guide sur le tuning des paramètres kernel avec sysctl](https://www.shpv.fr/blog/kernel-linux-2026/) pour une optimisation complète.

##### Qu'est-ce que Swappiness ?

**swappiness** contrôle l'agressivité du kernel pour swapper des pages mémoire.

**Valeurs :**

- **0** : Swap seulement en dernier recours
- **60** : Défaut (équilibre)
- **100** : Swap agressivement

##### Voir Swappiness Actuel

```bash
cat /proc/sys/vm/swappiness
# 60 (défaut)
```

##### Modifier Swappiness

**Temporaire (test) :**

```bash
# Swappiness conservateur (serveur)
sudo sysctl vm.swappiness=10

# Swappiness agressif (desktop)
sudo sysctl vm.swappiness=60
```

**Permanent :**

```bash
# Éditer sysctl.conf
sudo nano /etc/sysctl.conf

# Ajouter
vm.swappiness=10

# Appliquer
sudo sysctl -p
```

##### Recommandations Swappiness

|Type Système|Swappiness|Raison|
|---|---|---|
|Serveur web/DB|1-10|Minimiser latence|
|Serveur général|10-20|Équilibre|
|Desktop/Laptop|60|Défaut OK|
|Workstation RAM supérieure à 16 GB|10|Peu de swap nécessaire|

**Exemple serveur web :**

```bash
# Configuration optimale serveur web
vm.swappiness=10
vm.vfs_cache_pressure=50
```

---

#### Zswap : Swap Compressé en RAM

##### Qu'est-ce que Zswap ?

**zswap** compresse les pages avant de les swapper sur disque. Gain : 2-3x de capacité effective.

```
Sans zswap :
RAM pleine → Écriture directe sur disque (lent)

Avec zswap :
RAM pleine → Compression en RAM (rapide)
           → Si toujours plein → Disque
```

##### Activer Zswap

```bash
# Vérifier si déjà actif
cat /sys/module/zswap/parameters/enabled
# Y = activé, N = désactivé

# Activer temporairement
echo 1 | sudo tee /sys/module/zswap/parameters/enabled

# Activer au boot
# Éditer GRUB
sudo nano /etc/default/grub

# Ajouter à GRUB_CMDLINE_LINUX_DEFAULT
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash zswap.enabled=1"

# Mettre à jour GRUB
sudo update-grub  # Debian/Ubuntu
sudo grub2-mkconfig -o /boot/grub2/grub.cfg  # RedHat/CentOS

# Reboot
sudo reboot
```

##### Configuration Zswap Avancée

```bash
# Voir configuration
grep -r . /sys/module/zswap/parameters/

# Algorithme compression (défaut: lzo)
echo lz4 | sudo tee /sys/module/zswap/parameters/compressor
# lz4 = plus rapide, lzo = bon compromis, zstd = meilleure compression

# Pourcentage RAM pour zswap (défaut: 20%)
echo 25 | sudo tee /sys/module/zswap/parameters/max_pool_percent

# Activer/désactiver
echo 1 | sudo tee /sys/module/zswap/parameters/enabled
```

**Configuration optimale (GRUB) :**

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet zswap.enabled=1 zswap.compressor=lz4 zswap.max_pool_percent=25"
```

---

#### Zram : Swap Compressé Pure RAM

##### Zram vs Zswap

|Feature|Zram|Zswap|
|---|---|---|
|Backend|RAM uniquement|RAM + disque|
|Vitesse|Très rapide|Rapide|
|Use case|RAM limitée, SSD protégé|Serveur général|

##### Installer Zram

```bash
# Installer
sudo apt install zram-config  # Ubuntu
sudo yum install zram          # CentOS

# Ou script manuel
sudo modprobe zram

# Créer device zram
sudo zramctl --find --size 2G --algorithm lz4

# Créer swap dessus
sudo mkswap /dev/zram0
sudo swapon /dev/zram0 -p 100  # Priorité haute

# Vérifier
swapon --show
# NAME       TYPE SIZE USED PRIO
# /dev/zram0 partition 2G   0B  100
```

##### Configuration Zram Automatique

```bash
# /etc/systemd/system/zram.service
[Unit]
Description=Swap with zram
After=multi-user.target

[Service]
Type=oneshot
RemainAfterExit=true
ExecStart=/usr/bin/sh -c 'modprobe zram && echo lz4 > /sys/block/zram0/comp_algorithm && echo 2G > /sys/block/zram0/disksize && mkswap /dev/zram0 && swapon /dev/zram0 -p 100'
ExecStop=/usr/bin/sh -c 'swapoff /dev/zram0'

[Install]
WantedBy=multi-user.target
```

```bash
# Activer
sudo systemctl enable zram.service
sudo systemctl start zram.service
```

---

#### Priorité Swap

##### Comprendre Priorités

Le système utilise swap avec priorité la plus haute en premier.

```bash
# Voir priorités
swapon --show
# NAME       TYPE  SIZE USED PRIO
# /swapfile  file  2G   0B   -2
# /dev/sda3  partition 4G 0B -1

# Priorité : -2 à 32767
# Plus haut = utilisé en premier
```

##### Configurer Priorités

```bash
# Activer avec priorité spécifique
sudo swapon /swapfile -p 10
sudo swapon /dev/sda3 -p 5

# Dans /etc/fstab
/swapfile none swap sw,pri=10 0 0
/dev/sda3 none swap sw,pri=5  0 0

# Maintenant swapfile utilisé avant partition
```

**Use case priorités :**

- Zram (priorité 100) → Très rapide
- SSD swap (priorité 10) → Rapide
- HDD swap (priorité 1) → Lent, dernier recours

---

#### Hibernation (Suspend to Disk)

##### Configurer Hibernation

Hibernation nécessite swap ≥ RAM.

```bash
# Taille swap pour hibernation
# Si RAM = 8 GB → swap minimum 8 GB

# Trouver UUID swap
sudo blkid | grep swap
# /dev/sda3: UUID="abc123..." TYPE="swap"

# Éditer GRUB
sudo nano /etc/default/grub

# Ajouter resume
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=UUID=abc123..."

# Update GRUB
sudo update-grub

# Installer pm-utils
sudo apt install pm-utils

# Tester hibernation
sudo systemctl hibernate
```

##### Fichier Swap et Hibernation

Plus complexe car nécessite offset.

```bash
# Trouver offset fichier swap
sudo filefrag -v /swapfile | head -n 4
# Physical offset : 123456

# GRUB
GRUB_CMDLINE_LINUX_DEFAULT="resume=/dev/sda1 resume_offset=123456"
```

---

#### Monitoring et Diagnostics

Pour analyser les logs du système, découvrez notre [guide sur l'analyse des logs Linux avec journalctl](https://www.shpv.fr/blog/logs-linux-2026/).

##### Voir Utilisation Swap

```bash
# Vue globale
free -h

# Par processus
for file in /proc/*/status; do
    awk '/VmSwap|Name/{printf $2 " " $3}END{print ""}' $file
done | sort -k2 -n -r | head

# Graphique (si sysstat installé)
sar -S 1 10
# Affiche swap usage toutes les secondes

# htop avec colonne swap
htop
# F2 → Columns → SWAP → F10
```

##### Diagnostiquer Problèmes Swap

```bash
# Swap utilisé alors que RAM libre ?
# Mauvais signe, vérifier swappiness

free -h
#               total   used   free
# Mem:          8Gi     3Gi    4Gi  ← RAM libre
# Swap:         2Gi     500Mi  1.5Gi ← Swap utilisé quand même

# Solution
sudo sysctl vm.swappiness=10

# Forcer libération swap (si RAM dispo)
sudo swapoff -a && sudo swapon -a
```

##### Alertes Swap

```bash
# Script monitoring swap
#!/bin/bash
SWAP_USED=$(free | grep Swap | awk '{print $3/$2 * 100}')
THRESHOLD=80

if (( $(echo "$SWAP_USED > $THRESHOLD" | bc -l) )); then
    echo "WARNING: Swap usage ${SWAP_USED}% > ${THRESHOLD}%"
    # Envoyer alerte
    mail -s "Swap Alert" admin@example.com <<< "Swap at ${SWAP_USED}%"
fi
```

---

#### Optimisations Avancées

##### Cache Pressure

Contrôle agressivité récupération cache.

```bash
# Voir valeur actuelle
cat /proc/sys/vm/vfs_cache_pressure
# 100 (défaut)

# Optimisation serveur
# Conserver cache plus longtemps
sudo sysctl vm.vfs_cache_pressure=50

# Permanent
echo "vm.vfs_cache_pressure=50" | sudo tee -a /etc/sysctl.conf
```

##### Dirty Ratio

Contrôle quand flush cache disque.

```bash
# Paramètres dirty
sysctl -a | grep dirty

# Optimisation SSD
sudo sysctl vm.dirty_ratio=10
sudo sysctl vm.dirty_background_ratio=5

# Permanent
cat <<EOF | sudo tee -a /etc/sysctl.conf
vm.dirty_ratio=10
vm.dirty_background_ratio=5
vm.dirty_expire_centisecs=3000
vm.dirty_writeback_centisecs=500
EOF
```

##### Transparent Huge Pages (THP)

Pour des optimisations complètes des paramètres kernel, consultez notre [guide sur le tuning sysctl](https://www.shpv.fr/blog/sysctl-optimisation/).

```bash
# Voir statut
cat /sys/kernel/mm/transparent_hugepage/enabled
# [always] madvise never

# Désactiver (recommandé DB)
echo never | sudo tee /sys/kernel/mm/transparent_hugepage/enabled

# Permanent (GRUB)
GRUB_CMDLINE_LINUX_DEFAULT="quiet transparent_hugepage=never"
```

---

#### Configuration par Type de Serveur

##### Serveur Web (Nginx/Apache)

```bash
# /etc/sysctl.conf
vm.swappiness=10
vm.vfs_cache_pressure=50
vm.dirty_ratio=10
vm.dirty_background_ratio=5

# Swap : 2-4 GB suffisant
# Priorité performance réseau sur swap
```

##### Serveur Base de Données

```bash
# Configuration DB (MySQL/PostgreSQL)
vm.swappiness=1   # Minimum swap
vm.vfs_cache_pressure=50
vm.overcommit_memory=2  # Pas d'overcommit
vm.overcommit_ratio=80

# Désactiver THP
transparent_hugepage=never

# Swap : Égal RAM ou 0 si RAM > 32GB
```

##### Desktop/Workstation

```bash
# Configuration desktop
vm.swappiness=60  # Défaut OK
vm.vfs_cache_pressure=100

# Zswap activé
zswap.enabled=1
zswap.compressor=lz4

# Swap : 4-8 GB
```

---

#### Désactiver Swap

##### Temporairement

```bash
# Désactiver tout swap
sudo swapoff -a

# Réactiver
sudo swapon -a

# Désactiver swap spécifique
sudo swapoff /swapfile
```

##### Définitivement

```bash
# Désactiver
sudo swapoff -a

# Supprimer de /etc/fstab
sudo sed -i '/swap/d' /etc/fstab

# Supprimer fichier
sudo rm /swapfile

# Ou partition
sudo fdisk /dev/sda
# Supprimer partition swap
```

---

#### Benchmarking Swap

##### Tester Performance

```bash
# Test écriture swap
time dd if=/dev/zero of=/swapfile bs=1M count=1024

# Test avec stress
sudo apt install stress-ng

# Remplir RAM + swap
stress-ng --vm 4 --vm-bytes 8G --timeout 60s

# Observer
watch -n 1 'free -h && swapon --show'
```

##### Comparer Configurations

```bash
# Scénario 1 : Sans swap
swapoff -a
# Lancer workload
# Mesurer performance

# Scénario 2 : Swap swappiness=60
swapon -a
sysctl vm.swappiness=60
# Lancer workload

# Scénario 3 : Swap swappiness=10
sysctl vm.swappiness=10
# Lancer workload

# Scénario 4 : Zswap
# Activer zswap
# Lancer workload
```

---

#### Troubleshooting

##### Swap Non Actif au Boot

```bash
# Vérifier /etc/fstab
cat /etc/fstab | grep swap

# Vérifier syntaxe
# Correct : /swapfile none swap sw 0 0
# Erreur commune : mauvais path

# Tester activation manuelle
sudo swapon -a
# Si erreur, regarder dmesg
dmesg | grep -i swap
```

##### OOM Killer Actif

Pour une gestion complète des processus, consultez notre guide sur la [gestion des processus et debugging](https://www.shpv.fr/blog/gestion-processus-linux-2026/).

```bash
# Voir logs OOM
dmesg | grep -i "out of memory"
journalctl -k | grep -i "killed process"

# Processus tués
grep -i 'killed process' /var/log/syslog

# Solution : Augmenter swap ou RAM
```

##### Swap sur SSD : Usure

```bash
# Vérifier écriture SSD
sudo smartctl -a /dev/sda | grep Written

# Réduire usure
vm.swappiness=1        # Utiliser swap minimum
zswap.enabled=1        # Compression en RAM d'abord

# Ou désactiver swap totalement si RAM suffisante
```

---

#### Bonnes Pratiques

##### Taille Swap Recommandée

|RAM|Swap sans hibernation|Swap avec hibernation|
|---|---|---|
|Moins de 2 GB|2x RAM|3x RAM|
|2-8 GB|= RAM|2x RAM|
|8-32 GB|0.5x RAM|1.5x RAM|
|Plus de 32 GB|2 à 4 GB ou 0|= RAM|

##### Checklist Configuration

```bash
# Configuration optimale serveur production
□ Swap fichier 2 à 4 GB créé
□ swappiness=10
□ vfs_cache_pressure=50
□ zswap activé (si SSD)
□ Monitoring swap actif
□ Alertes configurées
□ THP désactivé (si DB)
```

---

#### Conclusion

Le swap est un composant essentiel de la gestion mémoire Linux :

**Points clés :**

- Créer swap avec fichier (flexible) ou partition (performance)
- Configurer swappiness selon usage (10 serveur, 60 desktop)
- Activer zswap pour compression (gain 2-3x)
- Monitorer utilisation régulièrement
- Adapter taille selon RAM disponible

**Commandes essentielles :**

```bash
# Créer swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Configurer swappiness
sudo sysctl vm.swappiness=10
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf

# Vérifier
free -h
swapon --show
```

Avec cette configuration, votre système Linux exploite la mémoire disponible sans saturer le swap.