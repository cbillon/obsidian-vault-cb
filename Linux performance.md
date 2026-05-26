---
tags:
  - linux
  - performance
---
Les performances Linux se jouent sur les détails : I/O scheduler adapté, gestion mémoire optimisée, huge pages pour bases de données. Ces réglages font la différence entre un serveur qui rame et un serveur réactif.

#### Plan

- I/O schedulers : choisir le bon selon le stockage
- Swappiness et gestion mémoire
- Transparent Huge Pages (THP)
- Profils tuned pour optimisations prédéfinies
- Sysctl : tuning kernel avancé
- Conclusion

---

#### I/O schedulers

##### Lister les schedulers disponibles

```bash
cat /sys/block/sda/queue/scheduler
# [mq-deadline] kyber bfq none
```

##### Schedulers principaux

**mq-deadline** (défaut) :

- Bon compromis général
- Fusion des requêtes I/O
- Convient HDD et SSD

**kyber** :

- Optimisé pour SSD/NVMe
- Faible latence
- Bon pour workloads aléatoires

**bfq** (Budget Fair Queueing) :

- Équité entre processus
- Bon pour desktops
- Moins performant en serveur

**none** (noop) :

- Aucun ordonnancement
- Pour NVMe ultra-rapides
- Laisse le SSD gérer

##### Changer le scheduler à chaud

```bash
# Temporaire
echo kyber > /sys/block/nvme0n1/queue/scheduler

# Permanent (udev)
# /etc/udev/rules.d/60-scheduler.rules
ACTION=="add|change", KERNEL=="nvme[0-9]n[0-9]", ATTR{queue/scheduler}="kyber"
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/scheduler}="mq-deadline"
```

##### Recommandations

- **HDD** : mq-deadline
- **SSD SATA** : mq-deadline ou kyber
- **NVMe** : kyber ou none

---

#### Swappiness

##### Principe

`vm.swappiness` (0-100) contrôle l'agressivité du swap :

- **0** : swap uniquement si mémoire pleine
- **60** (défaut) : swap modéré
- **100** : swap agressif

##### Valeur actuelle

```bash
cat /proc/sys/vm/swappiness
# 60
```

##### Modifier temporairement

```bash
sysctl -w vm.swappiness=10
```

##### Modifier définitivement

```bash
# /etc/sysctl.d/99-swappiness.conf
vm.swappiness = 10
```

Recharger :

```bash
sysctl -p /etc/sysctl.d/99-swappiness.conf
```

##### Recommandations

- **Serveur avec beaucoup de RAM** : 10
- **Base de données** : 1-10
- **Desktop** : 60 (défaut)
- **Serveur léger** : 30-40

---

#### Transparent Huge Pages (THP)

##### Principe

Les huge pages (2 Mo vs 4 Ko) réduisent la pression sur le TLB (Translation Lookaside Buffer).

**THP** : allocation automatique de huge pages par le kernel.

##### Vérifier l'état

```bash
cat /sys/kernel/mm/transparent_hugepage/enabled
# [always] madvise never
```

- **always** : THP actif partout (défaut)
- **madvise** : uniquement si demandé par l'app
- **never** : désactivé

##### Désactiver THP (recommandé pour bases de données)

```bash
# Temporaire
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

# Permanent (systemd)
# /etc/systemd/system/disable-thp.service
[Unit]
Description=Disable Transparent Huge Pages

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/enabled'
ExecStart=/bin/sh -c 'echo never > /sys/kernel/mm/transparent_hugepage/defrag'

[Install]
WantedBy=multi-user.target
```

```bash
systemctl enable --now disable-thp.service
```

##### Huge Pages statiques (pour DB)

```bash
# Calculer le nombre de huge pages nécessaires
# Ex: PostgreSQL avec 8 Go de shared_buffers
# 8192 Mo / 2 Mo = 4096 huge pages

# /etc/sysctl.d/99-hugepages.conf
vm.nr_hugepages = 4096
```

---

#### Profils tuned

##### Installation

```bash
dnf install tuned  # RHEL/Fedora
apt install tuned  # Debian/Ubuntu
```

##### Profils disponibles

```bash
tuned-adm list

# Profils courants :
# - throughput-performance : débit maximum I/O
# - latency-performance : faible latence
# - network-latency : latence réseau optimisée
# - virtual-guest : VM invitée
# - virtual-host : hyperviseur
```

##### Activer un profil

```bash
tuned-adm profile throughput-performance
tuned-adm active
```

##### Créer un profil custom

```bash
# /etc/tuned/postgresql/tuned.conf
[main]
include=throughput-performance

[sysctl]
vm.swappiness=1
vm.overcommit_memory=2
vm.nr_hugepages=4096

[disk]
devices=!*
elevator=mq-deadline
```

Activer :

```bash
tuned-adm profile postgresql
```

---

#### Sysctl : tuning kernel

##### Réseau

```bash
# /etc/sysctl.d/99-network.conf
# Buffer TCP
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216

# Backlog
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_max_syn_backlog = 8192
```

##### Mémoire

```bash
# /etc/sysctl.d/99-memory.conf
vm.swappiness = 10
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5
vm.vfs_cache_pressure = 50
```

##### Filesystem

```bash
fs.file-max = 2097152
fs.inotify.max_user_watches = 524288
```

---

#### Conclusion

L'optimisation Linux passe par des réglages ciblés : scheduler adapté au stockage, swappiness réduit pour éviter le swap inutile, THP désactivé pour bases de données. Les profils tuned simplifient l'application de configurations prédéfinies.