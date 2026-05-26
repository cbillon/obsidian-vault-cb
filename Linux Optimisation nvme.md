---
tags:
  - linux
  - performance
---
Les disques NVMe atteignent 7+ GB/s et 1M+ IOPS, mais une mauvaise configuration en réduit le débit utile. Guide complet : IO schedulers, TRIM, alignement, monitoring et benchmarks.

#### Plan

- Architecture NVMe vs SATA
- IO schedulers pour NVMe
- TRIM et discard automatique
- Alignement partitions et filesystem
- Monitoring wear leveling et santé
- Benchmarks et tuning avancé
- Optimisations kernel et applications
- Conclusion

---

#### Architecture NVMe vs SATA

##### Pourquoi NVMe est plus rapide

**SATA/AHCI :**

- Interface : SATA 6 Gbps (750 MB/s max)
- Queue depth : 32 commandes max
- Latence : ~100-500 µs
- Protocol overhead : AHCI legacy

**NVMe (PCIe) :**

- Interface : PCIe 4.0 x4 (8 GB/s), PCIe 5.0 x4 (16 GB/s)
- Queue depth : 64K commandes, 65K queues
- Latence : ~10-50 µs
- Protocol : optimisé pour flash NAND

**Comparaison typique :**

|Métrique|SATA SSD|NVMe PCIe 3.0|NVMe PCIe 4.0|NVMe PCIe 5.0|
|---|---|---|---|---|
|Lecture seq|550 MB/s|3500 MB/s|7000 MB/s|12000 MB/s|
|Écriture seq|520 MB/s|3000 MB/s|6000 MB/s|10000 MB/s|
|IOPS 4K read|100k|500k|1000k|1500k|
|IOPS 4K write|90k|400k|800k|1200k|
|Latence|100 µs|25 µs|15 µs|10 µs|

##### Identifier vos disques NVMe

```bash
# Lister disques NVMe
lsblk | grep nvme
nvme0n1     259:0    0   1.8T  0 disk

# Détails NVMe
nvme list
# Node             SN                   Model                      Version  Namespace Usage
# /dev/nvme0n1     S5XYNS0T123456       Samsung SSD 990 PRO 2TB    1.0      1         1.82  TB / 2.00 TB

# Info PCIe
lspci | grep -i nvme
# 01:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller

# Version PCIe et lanes
lspci -vv -s 01:00.0 | grep -E "LnkCap|LnkSta"
# LnkCap: Port #0, Speed 16GT/s, Width x4
# LnkSta: Speed 16GT/s (ok), Width x4 (ok)
```

---

#### IO schedulers pour NVMe

##### Types de schedulers disponibles

**none (noop) :**

- Pas de scheduling, FIFO simple
- **Recommandé pour NVMe**
- Latence minimale
- Queue depth élevé géré par device

**mq-deadline :**

- Deadline-based scheduling
- Bon pour mix read/write
- Évite write starvation

**kyber :**

- Token-based, limite latence
- Bon pour latence sensible
- Adaptatif selon charge

**bfq :**

- Budget Fair Queueing
- Bon pour desktop/interactif
- Overhead élevé, pas pour serveur

##### Configuration scheduler

```bash
# Voir scheduler actuel
cat /sys/block/nvme0n1/queue/scheduler
# [none] mq-deadline kyber

# Changer temporairement
echo none > /sys/block/nvme0n1/queue/scheduler

# Permanent (udev rule)
cat > /etc/udev/rules.d/60-nvme-scheduler.rules << 'EOF'
# Set none scheduler for all NVMe devices
ACTION=="add|change", KERNEL=="nvme[0-9]n[0-9]", ATTR{queue/scheduler}="none"
EOF

# Recharger udev
udevadm control --reload-rules
udevadm trigger --subsystem-match=block
```

##### Benchmarks schedulers

```bash
# Installer fio
apt install fio

# Test avec none
echo none > /sys/block/nvme0n1/queue/scheduler
fio --name=rand-read-none --rw=randread --bs=4k --size=1G \
    --numjobs=4 --filename=/mnt/nvme/test --direct=1 --group_reporting

# Test avec mq-deadline
echo mq-deadline > /sys/block/nvme0n1/queue/scheduler
fio --name=rand-read-deadline --rw=randread --bs=4k --size=1G \
    --numjobs=4 --filename=/mnt/nvme/test --direct=1 --group_reporting

# Test avec kyber
echo kyber > /sys/block/nvme0n1/queue/scheduler
fio --name=rand-read-kyber --rw=randread --bs=4k --size=1G \
    --numjobs=4 --filename=/mnt/nvme/test --direct=1 --group_reporting
```

**Résultats typiques (Samsung 990 PRO) :**

|Scheduler|IOPS 4K read|Latence p99|CPU usage|
|---|---|---|---|
|**none**|**1.2M**|**45 µs**|8 %|
|mq-deadline|950k|120 µs|12 %|
|kyber|1.1M|80 µs|10 %|

**Recommandation : `none` pour NVMe haute performance**

---

#### TRIM et discard automatique

##### Pourquoi TRIM est essentiel

**Sans TRIM :**

- Cellules "used" jamais effacées
- Performance écriture dégradée au fil du temps
- Wear leveling moins efficace
- Durée de vie réduite

**Avec TRIM :**

- OS notifie SSD des blocs libres
- SSD peut effacer en background
- Performance maintenue
- Wear leveling optimisé

##### Vérifier support TRIM

```bash
# Vérifier support discard
lsblk --discard
# NAME    DISC-GRAN DISC-MAX DISC-ZERO
# nvme0n1      512B       2G         0

# Si DISC-GRAN et DISC-MAX != 0 : TRIM supporté

# Vérifier avec hdparm (SATA)
hdparm -I /dev/sda | grep TRIM
# Data Set Management TRIM supported
```

##### TRIM automatique (recommandé)

```bash
# Activer discard au montage (fstab)
UUID=xxx /mnt/nvme ext4 defaults,noatime,discard 0 2

# Ou pour XFS
UUID=xxx /mnt/nvme xfs defaults,noatime,discard 0 2

# Remonter
mount -o remount /mnt/nvme

# Vérifier
mount | grep nvme
# /dev/nvme0n1p1 on /mnt/nvme type ext4 (rw,noatime,discard)
```

##### TRIM périodique (alternative)

```bash
# Service fstrim (inclus dans systemd)
systemctl status fstrim.timer
# Active, déclenche fstrim.service hebdomadaire

# Activer si pas déjà
systemctl enable --now fstrim.timer

# Lancer manuellement
fstrim -v /mnt/nvme
# /mnt/nvme: 450.2 GiB (483183820800 bytes) trimmed

# Logs
journalctl -u fstrim.service
```

**Discard continu vs périodique :**

|Mode|Avantages|Inconvénients|Usage|
|---|---|---|---|
|**discard (continu)**|TRIM immédiat, perf constante|Petit overhead écriture|Workload avec delete fréquent|
|**fstrim (hebdo)**|Pas d'overhead runtime|TRIM différé|Workload stable|

**Recommandation : `discard` pour NVMe (overhead négligeable)**

---

#### Alignement partitions et filesystem

##### Importance alignement

**Mal aligné :**

- 1 opération = 2 blocks physiques
- Performance -30-50 %
- Wear prématuré

**Bien aligné :**

- 1 opération = 1 block physique
- Latence minimale

##### Vérifier alignement

```bash
# Offset secteur de début partition
fdisk -l /dev/nvme0n1
# Device         Start        End    Sectors  Size Type
# /dev/nvme0n1p1  2048 3907029134 3907027087  1.8T Linux filesystem

# 2048 sectors * 512 bytes = 1 MiB : ALIGNÉ ✓

# Vérifier avec parted
parted /dev/nvme0n1 align-check optimal 1
# 1 aligned

# Vérifier filesystem alignment (ext4)
tune2fs -l /dev/nvme0n1p1 | grep -i "block size"
# Block size: 4096
```

##### Créer partitions alignées

```bash
# Avec parted (recommandé)
parted /dev/nvme0n1 mklabel gpt
parted /dev/nvme0n1 mkpart primary 0% 100%

# Vérifier
parted /dev/nvme0n1 align-check optimal 1
# 1 aligned

# Ou avec fdisk (défaut aligné depuis 2010+)
fdisk /dev/nvme0n1
# n (new), p (primary), 1, <enter>, <enter>
# w (write)
```

##### Options mkfs optimales

```bash
# ext4 (recommandé pour NVMe)
mkfs.ext4 -L data \
  -O ^has_journal \
  -E stride=128,stripe-width=128 \
  /dev/nvme0n1p1

# Options :
# -O ^has_journal : désactiver journal (NVMe = rapide + batterie backup)
# -E stride : taille chunk RAID (128 = 512KB pour NVMe)

# XFS (bon pour gros fichiers)
mkfs.xfs -L data \
  -f \
  -d agcount=8 \
  /dev/nvme0n1p1

# -d agcount : allocation groups (parallélisme)

# F2FS (optimisé flash, expérimental)
mkfs.f2fs -l data /dev/nvme0n1p1
```

##### Options montage optimales

```bash
# /etc/fstab

# ext4
UUID=xxx /mnt/nvme ext4 defaults,noatime,discard,commit=60 0 2

# XFS
UUID=xxx /mnt/nvme xfs defaults,noatime,discard,logbsize=256k 0 2

# Options importantes :
# noatime : pas d'update access time (perf +5-10%)
# discard : TRIM automatique
# commit=60 : flush metadata toutes les 60s (vs 5s défaut)
```

---

#### Monitoring wear leveling et santé

##### nvme-cli : outil essentiel

```bash
# Installer
apt install nvme-cli

# SMART log complet
nvme smart-log /dev/nvme0n1

# Sortie clés :
# critical_warning         : 0x00
# temperature              : 36 C
# available_spare          : 100%
# available_spare_threshold: 10%
# percentage_used          : 1%
# data_units_read          : 12,345,678
# data_units_written       : 23,456,789
# host_read_commands       : 234,567,890
# host_write_commands      : 345,678,901
# power_cycles             : 123
# power_on_hours           : 8,760
# unsafe_shutdowns         : 2
# media_errors             : 0
# num_err_log_entries      : 0
```

##### Métriques critiques

**percentage_used :**

- % wear du SSD (0-100 %)
- 80 % : préparer remplacement
    
- 95 % : remplacer d'urgence
    

**available_spare :**

- Blocks de réserve restants
- Inférieur à 50 % : fin de vie proche

**media_errors :**

- Erreurs matériel
- 0 : remplacer immédiatement
    

**temperature :**

- Température actuelle
- 70°C : throttling possible
    
- 85°C : critique
    

##### Script monitoring NVMe

```bash
#!/bin/bash
# /usr/local/bin/check-nvme.sh

EMAIL="admin@example.com"

for nvme in /dev/nvme[0-9]n[0-9]; do
    # Extraire métriques
    TEMP=$(nvme smart-log $nvme | grep "temperature" | awk '{print $3}')
    WEAR=$(nvme smart-log $nvme | grep "percentage_used" | awk '{print $3}' | tr -d '%')
    SPARE=$(nvme smart-log $nvme | grep "available_spare" | awk '{print $3}' | tr -d '%')
    ERRORS=$(nvme smart-log $nvme | grep "media_errors" | awk '{print $3}')

    # Alerte température
    if [ "$TEMP" -gt 70 ]; then
        mail -s "⚠️ HIGH NVMe Temperature: $nvme ${TEMP}°C" $EMAIL << EOF
WARNING: $nvme temperature is ${TEMP}°C (threshold: 70°C)

$(nvme smart-log $nvme)

Check cooling and workload.
EOF
    fi

    # Alerte wear
    if [ "$WEAR" -gt 80 ]; then
        mail -s "⚠️ NVMe High Wear: $nvme ${WEAR}%" $EMAIL << EOF
WARNING: $nvme wear level is ${WEAR}% (threshold: 80%)

Plan replacement soon.

$(nvme smart-log $nvme)
EOF
    fi

    # Alerte spare
    if [ "$SPARE" -lt 50 ]; then
        mail -s "⚠️ NVMe Low Spare: $nvme ${SPARE}%" $EMAIL << EOF
WARNING: $nvme spare blocks at ${SPARE}% (threshold: 50%)

End of life approaching.

$(nvme smart-log $nvme)
EOF
    fi

    # Alerte erreurs
    if [ "$ERRORS" -gt 0 ]; then
        mail -s "🚨 CRITICAL: NVMe Media Errors: $nvme" $EMAIL << EOF
CRITICAL: $nvme has $ERRORS media errors

IMMEDIATE ACTION REQUIRED: Backup data and replace drive.

$(nvme smart-log $nvme)
EOF
    fi
done
```

```bash
# Cron quotidien
echo "0 2 * * * /usr/local/bin/check-nvme.sh" | crontab -
```

##### Monitoring Prometheus

```bash
# nvme_exporter (Go)
wget https://github.com/prometheus-community/nvme_exporter/releases/download/v1.0.0/nvme_exporter-linux-amd64
chmod +x nvme_exporter-linux-amd64
./nvme_exporter-linux-amd64 &

# Métriques
curl localhost:9998/metrics | grep nvme
# nvme_temperature_celsius{device="nvme0n1"} 36
# nvme_percentage_used{device="nvme0n1"} 1
# nvme_available_spare_percent{device="nvme0n1"} 100
```

---

#### Benchmarks et tuning avancé

##### Benchmark complet avec fio

```bash
# 1. Lecture séquentielle
fio --name=seq-read --rw=read --bs=1M --size=10G \
    --numjobs=1 --filename=/mnt/nvme/test --direct=1

# 2. Écriture séquentielle
fio --name=seq-write --rw=write --bs=1M --size=10G \
    --numjobs=1 --filename=/mnt/nvme/test --direct=1

# 3. Lecture aléatoire 4K (IOPS)
fio --name=rand-read-4k --rw=randread --bs=4k --size=1G \
    --numjobs=4 --filename=/mnt/nvme/test --direct=1 \
    --iodepth=32 --group_reporting

# 4. Écriture aléatoire 4K
fio --name=rand-write-4k --rw=randwrite --bs=4k --size=1G \
    --numjobs=4 --filename=/mnt/nvme/test --direct=1 \
    --iodepth=32 --group_reporting

# 5. Mix 70/30 read/write (réaliste)
fio --name=mixed-rw --rw=randrw --rwmixread=70 --bs=4k \
    --size=1G --numjobs=4 --filename=/mnt/nvme/test --direct=1 \
    --iodepth=32 --group_reporting

# 6. Latence (QD=1)
fio --name=latency --rw=randread --bs=4k --size=500M \
    --numjobs=1 --filename=/mnt/nvme/test --direct=1 \
    --iodepth=1
```

##### Résultats attendus (Samsung 990 PRO 2TB)

```
=== Sequential ===
Read:  7000 MB/s
Write: 6000 MB/s

=== Random 4K (QD=32, 4 threads) ===
Read IOPS:  1.2M
Write IOPS: 800k

=== Mixed 70/30 ===
Read IOPS:  850k
Write IOPS: 350k

=== Latency (QD=1) ===
Read:  18 µs (p50), 45 µs (p99)
```

##### Tuning queue depth

```bash
# Voir queue depth actuel
cat /sys/block/nvme0n1/queue/nr_requests
# 1023

# Augmenter (plus de parallélisme)
echo 4096 > /sys/block/nvme0n1/queue/nr_requests

# Benchmark avant/après
fio --name=test --rw=randread --bs=4k --size=1G \
    --numjobs=8 --filename=/mnt/nvme/test --direct=1 \
    --iodepth=64
```

##### Tuning read-ahead

```bash
# Voir valeur actuelle (en sectors 512B)
blockdev --getra /dev/nvme0n1
# 256 (= 128 KB)

# Augmenter pour workload séquentiel
blockdev --setra 8192 /dev/nvme0n1
# 8192 = 4 MB

# Benchmark
fio --name=seq-read --rw=read --bs=1M --size=10G \
    --filename=/mnt/nvme/test --direct=0
```

---

#### Optimisations kernel et applications

##### Paramètres sysctl

```bash
# /etc/sysctl.d/99-nvme.conf

# Dirty pages (cache écriture)
vm.dirty_background_ratio = 5    # Défaut 10
vm.dirty_ratio = 10              # Défaut 20
vm.dirty_expire_centisecs = 3000 # 30s
vm.dirty_writeback_centisecs = 500 # 5s

# Swappiness (réduire pour garder en RAM)
vm.swappiness = 10               # Défaut 60

# VFS cache pressure
vm.vfs_cache_pressure = 50       # Défaut 100
```

```bash
# Appliquer
sysctl -p /etc/sysctl.d/99-nvme.conf
```

##### Optimisations applications

**PostgreSQL :**

```ini
# postgresql.conf

# Utiliser NVMe pour WAL
wal_sync_method = fdatasync
wal_level = replica

# Cache
shared_buffers = 16GB
effective_cache_size = 48GB

# Checkpoint
checkpoint_timeout = 15min
checkpoint_completion_target = 0.9

# Disques rapides = moins de random_page_cost
random_page_cost = 1.1   # Défaut 4.0
```

**MySQL/MariaDB :**

```ini
# my.cnf

# InnoDB sur NVMe
innodb_flush_method = O_DIRECT
innodb_io_capacity = 10000
innodb_io_capacity_max = 20000

# Log
innodb_log_file_size = 2G
innodb_flush_log_at_trx_commit = 2
```

**Redis :**

```conf
# redis.conf

# Persistence
save 900 1
save 300 10
save 60 10000

# AOF sur NVMe
appendonly yes
appendfsync everysec
no-appendfsync-on-rewrite yes
```

---

#### Checklist optimisation NVMe

✅ **Configuration :**

- [ ]  IO scheduler = none
- [ ]  TRIM/discard activé
- [ ]  Partitions alignées (2048 sectors)
- [ ]  Filesystem noatime,discard
- [ ]  Queue depth optimal

✅ **Monitoring :**

- [ ]  nvme-cli installé
- [ ]  Script monitoring wear/temp
- [ ]  Alertes si wear >80 %
- [ ]  Alertes si temp >70°C
- [ ]  Prometheus exporter

✅ **Performance :**

- [ ]  Benchmarks fio effectués
- [ ]  Read-ahead adapté
- [ ]  Sysctl optimisé
- [ ]  Applications configurées
- [ ]  PCIe lanes vérifiées (x4)

✅ **Maintenance :**

- [ ]  SMART log hebdomadaire
- [ ]  Backup régulier
- [ ]  Remplacement planifié si wear >80 %
- [ ]  Tests performance trimestriels

---

#### Conclusion

Les disques NVMe offrent des performances exceptionnelles quand configurés correctement. L'IO scheduler `none`, TRIM automatique et monitoring du wear sont essentiels pour maintenir performance et durée de vie.

**Points clés :**

- Scheduler none pour NVMe
- TRIM continu (discard) recommandé
- Monitoring wear et température
- Alignement partitions critique
- Applications à optimiser spécifiquement

**Gains typiques :**

- IOPS : 10-20x vs SATA SSD
- Latence : -80 % vs SATA
- Throughput : 7-14x vs SATA
- Durée de vie : +30 % avec bon monitoring

**Actions prioritaires :**

1. Configurer IO scheduler none
2. Activer TRIM/discard
3. Implémenter monitoring wear
4. Benchmarker performance
5. Optimiser applications critiques

**Lectures complémentaires :** Découvrez [TRIM basique pour SSD](https://www.shpv.fr/blog/ssd-trim/) et [partitionnement et alignement](https://www.shpv.fr/blog/partitionnement-disque-linux-2026/). Pour ZFS sur NVMe : [ZFS sur NVMe avec tuning avancé](https://www.shpv.fr/blog/zfs-tuning-extreme/).