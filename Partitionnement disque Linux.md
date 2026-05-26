---
tags:
  - linux
  - partition
---
Le partitionnement disque est une opération fondamentale dans l'administration Linux. Ce guide explique comment créer, gérer et optimiser les partitions avec fdisk, parted et comprendre GPT vs MBR.

#### Comprendre le Partitionnement

##### Qu'est-ce qu'une Partition ?

Une **partition** est une section logique d'un disque physique. C'est comme diviser un grand terrain en plusieurs parcelles distinctes.

**Analogies :**

- Disque = Immeuble
- Partitions = Appartements
- Chaque appartement (partition) peut avoir un locataire différent (système de fichiers)

##### Pourquoi Partitionner ?

**Avantages :**

- Séparer système et données
- Multi-boot (plusieurs OS)
- Sécurité (isoler /home)
- Performance (optimiser I/O)
- Sauvegardes ciblées
- Limiter impact corruption

**Exemple partitionnement serveur :**

```
/dev/sda1 - 512 MB  - /boot    (noyau Linux)
/dev/sda2 - 50 GB   - /        (système)
/dev/sda3 - 100 GB  - /var     (logs, bases de données)
/dev/sda4 - 200 GB  - /home    (utilisateurs)
```

---

#### MBR vs GPT : Comprendre la Différence

##### MBR (Master Boot Record)

**Créé :** 1983 **Limite :** 2 TB par disque, 4 partitions primaires

```
Structure MBR :
┌────────────────────────────────────┐
│  Boot Code (446 bytes)             │
├────────────────────────────────────┤
│  Partition Table (64 bytes)        │
│  - 4 entrées × 16 bytes            │
│  - Partition 1                     │
│  - Partition 2                     │
│  - Partition 3                     │
│  - Partition 4 (peut être étendue) │
├────────────────────────────────────┤
│  Boot Signature (2 bytes: 0x55AA)  │
└────────────────────────────────────┘
```

**Limitations MBR :**

- Maximum 2 TB par partition
- 4 partitions primaires seulement
- Ou 3 primaires + 1 étendue contenant logiques
- Pas de redondance (corruption = perdu)

##### GPT (GUID Partition Table)

**Créé :** ~2000 (avec UEFI) **Limite :** 9.4 ZB par disque, 128 partitions

```
Structure GPT :
┌────────────────────────────────────┐
│  Protective MBR (rétrocompat)      │
├────────────────────────────────────┤
│  Primary GPT Header               │
│  - Signature, version             │
│  - Partition table location       │
│  - CRC32 checksums                │
├────────────────────────────────────┤
│  Partition Entries (128 max)      │
│  - Chaque entrée : 128 bytes      │
│  - Type GUID                      │
│  - Unique GUID                    │
│  - Nom partition (36 chars)       │
├────────────────────────────────────┤
│  ... Données partitions ...       │
├────────────────────────────────────┤
│  Backup Partition Entries         │
├────────────────────────────────────┤
│  Secondary GPT Header (backup)    │
└────────────────────────────────────┘
```

**Avantages GPT :**

- Jusqu'à 128 partitions (standard)
- Partitions jusqu'à 9.4 ZB
- Redondance (header backup)
- CRC32 checksums (détection corruption)
- Noms partitions lisibles
- Requis pour UEFI boot

##### MBR vs GPT : Tableau Comparatif

|Feature|MBR|GPT|
|---|---|---|
|Taille max disque|2 TB|9.4 ZB|
|Partitions max|4 primaires|128 standard|
|Firmware|BIOS|UEFI (+ BIOS compat)|
|Redondance|Non|Oui (backup header)|
|Checksums|Non|Oui (CRC32)|
|Noms partitions|Non|Oui|
|Année création|1983|~2000|

**Recommandation 2026 :** Utiliser GPT sauf compatibilité ancienne nécessaire.

---

#### Lister et Identifier les Disques

##### Commandes de Découverte

```bash
# Lister tous disques et partitions
lsblk

# Output exemple :
# NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda      8:0    0  500G  0 disk
# ├─sda1   8:1    0  512M  0 part /boot
# ├─sda2   8:2    0   50G  0 part /
# └─sda3   8:3    0  100G  0 part /home
# sdb      8:16   0    1T  0 disk
# └─sdb1   8:17   0    1T  0 part /data

# Avec système de fichiers
lsblk -f

# Détails complets
sudo fdisk -l

# Seulement /dev/sda
sudo fdisk -l /dev/sda

# Partitions avec UUID
blkid

# Informations matériel disque
sudo hdparm -I /dev/sda | grep -i model

# Smart status
sudo smartctl -a /dev/sda
```

##### Nomenclature Disques

```bash
# Disques SATA/SCSI
/dev/sda  # Premier disque
/dev/sdb  # Deuxième disque
/dev/sdc  # Troisième disque

# Partitions
/dev/sda1 # Première partition de sda
/dev/sda2 # Deuxième partition de sda

# Disques NVMe
/dev/nvme0n1    # Premier disque NVMe
/dev/nvme0n1p1  # Première partition
/dev/nvme0n1p2  # Deuxième partition

# Disques virtuels (VirtIO)
/dev/vda
/dev/vda1
```

---

#### fdisk : Outil de Partitionnement MBR

##### fdisk - Usage Basique

```bash
# Lancer fdisk sur disque
sudo fdisk /dev/sdb

# Commandes fdisk (mode interactif) :
# m - Aide
# p - Afficher table partitions
# n - Nouvelle partition
# d - Supprimer partition
# t - Changer type partition
# w - Écrire changements et quitter
# q - Quitter sans sauvegarder
```

##### Créer Partition avec fdisk

```bash
# Exemple complet : créer 2 partitions sur /dev/sdb

sudo fdisk /dev/sdb

# 1. Afficher table actuelle
Command: p

# 2. Créer première partition (50 GB)
Command: n
Partition type: p (primary)
Partition number: 1
First sector: (Enter pour défaut)
Last sector: +50G

# 3. Créer deuxième partition (reste du disque)
Command: n
Partition type: p
Partition number: 2
First sector: (Enter)
Last sector: (Enter pour tout le reste)

# 4. Vérifier
Command: p

# 5. Écrire changements
Command: w

# Informer kernel des changements
sudo partprobe /dev/sdb
```

##### Changer Type Partition

```bash
sudo fdisk /dev/sdb

# Lister types disponibles
Command: l

# Changer type partition 1
Command: t
Partition number: 1
Hex code: 83  # Linux filesystem
# Ou
Hex code: 82  # Linux swap
# Ou
Hex code: 8e  # Linux LVM

# Sauvegarder
Command: w
```

---

#### parted : Outil Moderne et GPT

##### parted - Avantages

- Support GPT et MBR
- Mode interactif ET ligne de commande
- Redimensionnement partitions
- Alignement automatique
- Plus sûr (vérifie avant d'écrire)

##### parted - Commandes de Base

```bash
# Mode interactif
sudo parted /dev/sdb

# Commandes parted :
# print        - Afficher partitions
# mklabel      - Créer table partition
# mkpart       - Créer partition
# rm           - Supprimer partition
# resizepart   - Redimensionner partition
# quit         - Quitter

# Mode ligne de commande (scripts)
sudo parted /dev/sdb print
```

##### Créer Table Partition GPT

```bash
# Attention : EFFACE toutes données !
sudo parted /dev/sdb mklabel gpt

# Vérifier
sudo parted /dev/sdb print

# Output :
# Model: ATA Samsung SSD (scsi)
# Disk /dev/sdb: 1000GB
# Sector size: 512B/512B
# Partition Table: gpt
```

##### Créer Partitions avec parted

```bash
# Créer partition 100 GB au début
sudo parted /dev/sdb mkpart primary ext4 0% 100GB

# Créer partition swap 8 GB
sudo parted /dev/sdb mkpart primary linux-swap 100GB 108GB

# Créer partition avec le reste
sudo parted /dev/sdb mkpart primary ext4 108GB 100%

# Vérifier alignement
sudo parted /dev/sdb align-check optimal 1
# 1 aligned
```

##### Redimensionner Partition

```bash
# IMPORTANT : Démonter partition d'abord !
sudo umount /dev/sdb1

# Vérifier système fichiers
sudo e2fsck -f /dev/sdb1

# Redimensionner partition dans parted
sudo parted /dev/sdb
(parted) resizepart 1 200GB

# Redimensionner système de fichiers
sudo resize2fs /dev/sdb1

# Remonter
sudo mount /dev/sdb1 /mnt
```

---

#### gdisk : fdisk pour GPT

##### gdisk - Utilisation

```bash
# Lancer gdisk
sudo gdisk /dev/sdb

# Commandes similaires fdisk :
# p - Print partition table
# n - New partition
# d - Delete partition
# t - Change partition type
# w - Write and quit
# q - Quit without saving

# Commandes GPT spécifiques :
# i - Show detailed information
# x - Extra functionality (expert mode)
# r - Recovery options
```

##### Créer Partitions GPT avec gdisk

```bash
sudo gdisk /dev/sdb

# Nouvelle partition
Command: n
Partition number: 1
First sector: (Enter)
Last sector: +100G
Hex code or GUID: 8300  # Linux filesystem

# Autre partition
Command: n
Partition number: 2
First sector: (Enter)
Last sector: (Enter pour reste)
Hex code: 8300

# Sauvegarder
Command: w
```

##### Types Partition GPT Courants

```bash
# Dans gdisk, commande 't' puis 'L' liste types

# Types importants :
8200 - Linux swap
8300 - Linux filesystem
8301 - Linux reserved
8e00 - Linux LVM
ef00 - EFI System Partition
fd00 - Linux RAID
```

---

#### Formater Partitions

##### Systèmes de Fichiers Courants

|FS|Usage|Performance|Fonctionnalités|
|---|---|---|---|
|ext4|Défaut Linux|Bonne|Journal, stable|
|xfs|Serveur, gros fichiers|Excellente|Scalable, performant|
|btrfs|Moderne|Bonne|Snapshots, compression|
|f2fs|SSD/Flash|Très bonne|Optimisé flash|

##### Créer Systèmes de Fichiers

```bash
# ext4 (défaut recommandé)
sudo mkfs.ext4 /dev/sdb1

# Avec label
sudo mkfs.ext4 -L "Data" /dev/sdb1

# xfs
sudo mkfs.xfs /dev/sdb1

# btrfs
sudo mkfs.btrfs /dev/sdb1

# FAT32 (compatibilité Windows)
sudo mkfs.vfat -F 32 /dev/sdb1

# NTFS (Windows)
sudo mkfs.ntfs /dev/sdb1

# Swap
sudo mkswap /dev/sdb2
```

##### Options Formatage Avancées

```bash
# ext4 avec options
sudo mkfs.ext4 \
  -L "MyData" \         # Label
  -m 1 \                # Reserved blocks 1% (au lieu de 5%)
  -O ^has_journal \     # Sans journal (SSD)
  /dev/sdb1

# xfs avec options
sudo mkfs.xfs \
  -f \                  # Force
  -L "Storage" \        # Label
  -d agcount=8 \        # Allocation groups
  /dev/sdb1

# btrfs avec compression
sudo mkfs.btrfs \
  -L "Backup" \
  -f \
  /dev/sdb1
```

---

#### Monter Partitions

##### Montage Manuel

```bash
# Créer point de montage
sudo mkdir -p /mnt/data

# Monter
sudo mount /dev/sdb1 /mnt/data

# Vérifier
df -h | grep /mnt/data
mount | grep /mnt/data

# Démonter
sudo umount /mnt/data
# Ou par point montage
sudo umount /dev/sdb1
```

##### Montage Automatique (/etc/fstab)

```bash
# Éditer fstab
sudo nano /etc/fstab

# Format :
# <device> <mount point> <type> <options> <dump> <pass>

# Par device
/dev/sdb1 /mnt/data ext4 defaults 0 2

# Par UUID (recommandé - stable)
UUID=abc-123-def /mnt/data ext4 defaults 0 2

# Par LABEL
LABEL=Data /mnt/data ext4 defaults 0 2

# Trouver UUID
sudo blkid /dev/sdb1
# /dev/sdb1: UUID="abc-123-def" TYPE="ext4"
```

##### Options Montage Courantes

```bash
# Options fstab importantes :

# defaults
# = rw,suid,dev,exec,auto,nouser,async

# Lecture seule
/dev/sdb1 /mnt/backup ext4 ro 0 2

# Pas d'exécution binaires
/dev/sdb1 /mnt/data ext4 defaults,noexec 0 2

# Pas de suid
/dev/sdb1 /mnt/shared ext4 defaults,nosuid 0 2

# Montage utilisateur
/dev/sdb1 /mnt/usb ext4 defaults,user 0 0

# NFS
server:/export /mnt/nfs nfs defaults 0 0

# Tester fstab sans reboot
sudo mount -a
```

---

#### Schémas Partitionnement Recommandés

##### Desktop / Laptop

```bash
# Disque 500 GB SSD

/dev/sda1  512 MB   EFI System Partition (fat32)
/dev/sda2   50 GB   / (root, ext4)
/dev/sda3    8 GB   swap
/dev/sda4  441 GB   /home (ext4)

# Avantages :
# - /home séparé (réinstall OS sans perdre données)
# - Swap pour hibernation
# - EFI pour UEFI boot
```

##### Serveur Web

```bash
# Disque 1 TB

/dev/sda1  512 MB   /boot (ext4)
/dev/sda2   30 GB   / (root, ext4)
/dev/sda3  200 GB   /var (xfs - logs, cache)
/dev/sda4  100 GB   /var/lib/mysql (xfs - database)
/dev/sda5  reste    /home (ext4)

# Swap : 4-8 GB fichier dans /

# Avantages :
# - /var séparé (logs ne remplissent pas /)
# - DB séparée (optimisation I/O)
# - /boot petit (noyaux multiples OK)
```

##### Serveur Base de Données

```bash
# Disque 2 TB SSD

/dev/sda1  512 MB   /boot (ext4)
/dev/sda2   50 GB   / (ext4)
/dev/sda3  reste    LVM physical volume

# Dans LVM :
lv_data   1.5 TB    /var/lib/postgresql (xfs)
lv_backup 400 GB    /backup (xfs)

# Avantages :
# - LVM flexible (redimensionner facilement)
# - Snapshots LVM pour backup cohérent
# - xfs optimisé gros fichiers DB
```

---

#### LVM : Logical Volume Manager

##### Pourquoi LVM ?

**Avantages :**

- Redimensionner partitions à chaud
- Snapshots pour backups
- Pools de stockage
- Migration données entre disques

##### Concepts LVM

```
Physical Volumes (PV)
    ↓
Volume Groups (VG)
    ↓
Logical Volumes (LV)
    ↓
Filesystems

Exemple :
/dev/sda3 (PV) ─┐
/dev/sdb1 (PV) ─┤→ vg_data (VG) ─┬→ lv_mysql (LV) → /var/lib/mysql
/dev/sdc1 (PV) ─┘                 └→ lv_www (LV)   → /var/www
```

##### Créer LVM

```bash
# 1. Créer Physical Volumes
sudo pvcreate /dev/sdb1 /dev/sdc1

# Vérifier
sudo pvdisplay

# 2. Créer Volume Group
sudo vgcreate vg_data /dev/sdb1 /dev/sdc1

# Vérifier
sudo vgdisplay

# 3. Créer Logical Volumes
sudo lvcreate -L 100G -n lv_mysql vg_data
sudo lvcreate -L 200G -n lv_www vg_data

# Vérifier
sudo lvdisplay

# 4. Formater LV
sudo mkfs.xfs /dev/vg_data/lv_mysql
sudo mkfs.ext4 /dev/vg_data/lv_www

# 5. Monter
sudo mount /dev/vg_data/lv_mysql /var/lib/mysql
```

##### Redimensionner LVM

```bash
# Agrandir LV
sudo lvextend -L +50G /dev/vg_data/lv_mysql
sudo xfs_growfs /var/lib/mysql  # xfs
# Ou
sudo resize2fs /dev/vg_data/lv_mysql  # ext4

# Réduire LV (ext4 seulement, démonter avant)
sudo umount /dev/vg_data/lv_www
sudo e2fsck -f /dev/vg_data/lv_www
sudo resize2fs /dev/vg_data/lv_www 150G
sudo lvreduce -L 150G /dev/vg_data/lv_www
sudo mount /dev/vg_data/lv_www /var/www
```

---

#### Sauvegardes et Clonage

##### dd : Cloner Partition

```bash
# Cloner partition complète
sudo dd if=/dev/sda1 of=/dev/sdb1 bs=4M status=progress

# Créer image disque
sudo dd if=/dev/sda of=/backup/sda.img bs=4M status=progress

# Restaurer image
sudo dd if=/backup/sda.img of=/dev/sda bs=4M status=progress

# Compresser image à la volée
sudo dd if=/dev/sda bs=4M | gzip > /backup/sda.img.gz

# Restaurer compressée
gunzip -c /backup/sda.img.gz | sudo dd of=/dev/sda bs=4M
```

##### Backup Table Partitions

```bash
# Sauvegarder MBR
sudo dd if=/dev/sda of=/backup/mbr.bin bs=512 count=1

# Sauvegarder GPT
sudo sgdisk --backup=/backup/gpt-sda.bin /dev/sda

# Restaurer GPT
sudo sgdisk --load-backup=/backup/gpt-sda.bin /dev/sda
```

---

#### Troubleshooting

##### Partition Introuvable

```bash
# Forcer relecture table partitions
sudo partprobe /dev/sda

# Ou
sudo blockdev --rereadpt /dev/sda

# Ou reboot (si ci-dessus échoue)
sudo reboot
```

##### Erreur "Device is busy"

```bash
# Trouver ce qui utilise partition
sudo lsof | grep /dev/sdb1
sudo fuser -m /dev/sdb1

# Tuer processus
sudo fuser -km /dev/sdb1

# Puis démonter
sudo umount /dev/sdb1
```

##### Réparer Table Partitions

```bash
# MBR corrompu
sudo testdisk /dev/sda

# GPT corrompu (restaurer backup)
sudo gdisk /dev/sda
Command: r (recovery)
Command: c (use backup header)
Command: w (write)

# Vérifier cohérence GPT
sudo gdisk -l /dev/sda
```

---

#### Bonnes Pratiques

##### Sécurité

✓ **Toujours backup avant partitionnement** ✓ **Vérifier device (/dev/sda vs /dev/sdb)** ✓ **Tester commandes en read-only d'abord** ✓ **Utiliser UUID dans fstab (pas /dev/sdX)**

##### Performance

✓ **Aligner partitions (4096 bytes boundary)** ✓ **Utiliser GPT sur disques > 2TB** ✓ **Séparer OS et données** ✓ **xfs pour gros fichiers/DB**

##### Maintenance

✓ **Documenter schéma partitionnement** ✓ **Monitorer espace disque** ✓ **Planifier croissance future** ✓ **Sauvegarder table partitions**

---

#### Étapes suivantes

Après le partitionnement, consultez [montage des systèmes de fichiers](https://www.shpv.fr/blog/montage-filesystems-linux-2026/). Pour utiliser LVM, voyez [LVM en détail](https://www.shpv.fr/blog/lvm-logical-volume-management/). Pour le chiffrement, consultez [chiffrement LUKS](https://www.shpv.fr/blog/luks-chiffrement/). Pour RAID, découvrez [RAID avec mdadm](https://www.shpv.fr/blog/raid1-mdadm-linux/).

#### Conclusion

Le partitionnement disque Linux nécessite compréhension des outils et formats :

**Choix format :**

- GPT : Nouveau système, disque > 2TB, UEFI
- MBR : Compatibilité ancienne seulement

**Outils :**

```bash
fdisk    # MBR, simple, interactif
parted   # GPT/MBR, moderne, scriptable
gdisk    # GPT, comme fdisk
```

**Workflow typique :**

```bash
# 1. Identifier disque
lsblk

# 2. Créer table partitions
sudo parted /dev/sdb mklabel gpt

# 3. Créer partitions
sudo parted /dev/sdb mkpart primary ext4 0% 50%

# 4. Formater
sudo mkfs.ext4 /dev/sdb1

# 5. Monter
sudo mount /dev/sdb1 /mnt/data

# 6. Permanent (fstab)
UUID=xxx /mnt/data ext4 defaults 0 2
```

Avec ce guide, vous maîtrisez le partitionnement disque sous Linux de A à Z !