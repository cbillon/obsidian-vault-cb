---
tags:
  - linux
---
Le montage de systèmes de fichiers est une opération fondamentale sous Linux. Ce guide explique comment monter partitions, NFS, CIFS, ISO et gérer fstab pour des montages automatiques.

#### Comprendre le Montage

##### Qu'est-ce que Monter ?

**Monter** = Attacher un système de fichiers à l'arborescence Linux.

**Analogie :** Monter = brancher une clé USB et la rendre accessible dans un dossier.

##### Arborescence Unifiée

```
/                       (root filesystem)
├── /boot              (partition boot)
├── /home              (partition ou autre disque)
├── /mnt/usb           (clé USB montée)
├── /mnt/nfs           (partage NFS)
└── /media/cdrom       (CD-ROM)
```

**Différence Windows vs Linux :**

- Windows : C:, D:, E:\ (lettres disques)
- Linux : Tout sous / (montages dans arborescence)

---

#### Commande mount

##### Syntaxe de Base

```bash
# Lister tous montages
mount

# Monter partition
sudo mount /dev/sdb1 /mnt/data

# Avec type filesystem
sudo mount -t ext4 /dev/sdb1 /mnt/data

# Avec options
sudo mount -o ro,noexec /dev/sdb1 /mnt/data
```

##### Lister Montages

```bash
# Tous montages
mount

# Format lisible
mount | column -t

# Seulement type spécifique
mount -t ext4

# findmnt (meilleur que mount)
findmnt

# Format arbre
findmnt --tree

# Montages réels (pas tmpfs, etc)
findmnt --real
```

##### Créer Point de Montage

```bash
# Créer dossier montage
sudo mkdir -p /mnt/data

# Permissions appropriées
sudo chown user:user /mnt/data
```

---

#### Options de Montage

##### Options Courantes

```bash
# Lecture seule
mount -o ro /dev/sdb1 /mnt/data

# Lecture/écriture (défaut)
mount -o rw /dev/sdb1 /mnt/data

# Pas d'exécution binaires
mount -o noexec /dev/sdb1 /mnt/data

# Pas de suid
mount -o nosuid /dev/sdb1 /mnt/data

# Pas de devices
mount -o nodev /dev/sdb1 /mnt/data

# Utilisateur peut monter
mount -o user /dev/sdb1 /mnt/data

# Asynchrone (performances)
mount -o async /dev/sdb1 /mnt/data

# Synchrone (sécurité)
mount -o sync /dev/sdb1 /mnt/data

# Combiner options
mount -o ro,noexec,nosuid /dev/sdb1 /mnt/data
```

##### Options par Type

**ext4 :**

```bash
# Journal data
mount -o data=journal /dev/sdb1 /mnt/data

# Pas de access time update (performance)
mount -o noatime /dev/sdb1 /mnt/data

# Barrières I/O
mount -o barrier=1 /dev/sdb1 /mnt/data
```

**NTFS (Windows) :**

```bash
# Installer ntfs-3g
sudo apt install ntfs-3g

# Monter NTFS
sudo mount -t ntfs-3g /dev/sdb1 /mnt/windows

# Permissions Linux
sudo mount -t ntfs-3g -o uid=1000,gid=1000 /dev/sdb1 /mnt/windows
```

**FAT32 (USB) :**

```bash
# Monter FAT32
sudo mount -t vfat /dev/sdb1 /mnt/usb

# Avec permissions
sudo mount -t vfat -o uid=1000,gid=1000,umask=022 /dev/sdb1 /mnt/usb
```

---

#### Démonter : umount

##### Commandes umount

```bash
# Démonter
sudo umount /mnt/data

# Par device
sudo umount /dev/sdb1

# Force (attention, perte données possible)
sudo umount -f /mnt/data

# Lazy unmount (attend fin utilisation)
sudo umount -l /mnt/data

# Tous montages d'un type
sudo umount -a -t nfs
```

##### Device Busy

```bash
# Erreur : device is busy
# Trouver qui utilise
sudo lsof /mnt/data
sudo fuser -m /mnt/data

# Tuer processus
sudo fuser -km /mnt/data

# Ou identifier et tuer manuellement
ps aux | grep /mnt/data
sudo kill <PID>

# Puis démonter
sudo umount /mnt/data
```

---

#### fstab : Montages Automatiques

##### Format /etc/fstab

```bash
# /etc/fstab
# <device> <mount_point> <type> <options> <dump> <pass>

/dev/sda1  /              ext4    defaults        0  1
/dev/sda2  /home          ext4    defaults        0  2
/dev/sdb1  /mnt/data      ext4    defaults        0  0
/dev/sdc1  /mnt/backup    ext4    noauto          0  0
```

**Colonnes :**

1. **Device** : /dev/sdX, UUID, LABEL
2. **Mount point** : Où monter
3. **Type** : ext4, ntfs, nfs, etc.
4. **Options** : defaults, ro, noexec, etc.
5. **Dump** : Backup (0=non, 1=oui)
6. **Pass** : fsck check (0=non, 1=root, 2=autres)

##### UUID vs Device

```bash
# Device peut changer (/dev/sdb devient /dev/sdc)
# UUID est stable

# Trouver UUID
sudo blkid
# /dev/sdb1: UUID="abc-123-def" TYPE="ext4"

# Dans fstab (recommandé)
UUID=abc-123-def  /mnt/data  ext4  defaults  0  2

# Par LABEL
LABEL=MyData  /mnt/data  ext4  defaults  0  2
```

##### Options fstab Courantes

```bash
# defaults = rw,suid,dev,exec,auto,nouser,async

# Pas monter au boot
UUID=xxx  /mnt/backup  ext4  noauto  0  0

# Lecture seule
UUID=xxx  /mnt/ro  ext4  ro  0  0

# Utilisateur peut monter
UUID=xxx  /mnt/usb  vfat  user,noauto  0  0

# Performance (pas atime)
UUID=xxx  /mnt/data  ext4  defaults,noatime  0  2

# Sécurité
UUID=xxx  /mnt/shared  ext4  defaults,noexec,nosuid  0  2
```

##### Tester fstab

```bash
# Éditer fstab
sudo nano /etc/fstab

# Tester SANS reboot
sudo mount -a

# Vérifier montages
mount | grep /mnt/data

# Si erreur dans fstab, système peut ne pas booter !
# Toujours tester avec mount -a
```

---

#### NFS : Network File System

##### Serveur NFS

```bash
# Installer serveur NFS
sudo apt install nfs-kernel-server

# Configurer exports
sudo nano /etc/exports

# Exemple exports
/srv/nfs        192.168.1.0/24(rw,sync,no_subtree_check)
/home/shared    *(ro,sync)
/data           client1.example.com(rw,async,no_root_squash)

# Options :
# rw              - lecture/écriture
# ro              - lecture seule
# sync            - synchrone (sûr)
# async           - asynchrone (rapide)
# no_subtree_check - pas vérif sous-répertoires (performance)
# root_squash     - root client = nobody serveur (sécurité)
# no_root_squash  - root client = root serveur (danger)

# Appliquer
sudo exportfs -a

# Redémarrer NFS
sudo systemctl restart nfs-kernel-server

# Voir exports
showmount -e localhost
```

##### Client NFS

```bash
# Installer client
sudo apt install nfs-common

# Voir exports serveur
showmount -e nfs-server.example.com

# Monter temporairement
sudo mount -t nfs nfs-server:/srv/nfs /mnt/nfs

# Dans fstab (permanent)
nfs-server:/srv/nfs  /mnt/nfs  nfs  defaults  0  0

# Options NFS utiles
nfs-server:/data  /mnt/data  nfs  rw,soft,intr,timeo=30  0  0

# soft    - timeout si serveur down (vs hard)
# intr    - interruptible
# timeo   - timeout (dixièmes seconde)
# vers=4  - forcer NFSv4
```

##### NFSv4

```bash
# Serveur NFSv4
sudo nano /etc/exports
/srv/nfs  *(rw,sync,fsid=0,crossmnt,no_subtree_check)

# Client NFSv4
sudo mount -t nfs4 nfs-server:/ /mnt/nfs

# fstab
nfs-server:/  /mnt/nfs  nfs4  defaults  0  0
```

---

#### CIFS/SMB : Windows Shares

##### Monter Partage Windows

```bash
# Installer client CIFS
sudo apt install cifs-utils

# Monter temporairement
sudo mount -t cifs //server/share /mnt/windows \
    -o username=user,password=pass

# Sans mot de passe dans commande
sudo mount -t cifs //server/share /mnt/windows \
    -o credentials=/root/.smbcredentials

# Fichier credentials
# /root/.smbcredentials
username=user
password=password
domain=WORKGROUP

# Permissions
sudo chmod 600 /root/.smbcredentials
```

##### fstab CIFS

```bash
# /etc/fstab
//server/share  /mnt/windows  cifs  credentials=/root/.smbcredentials,uid=1000,gid=1000  0  0

# Options utiles
//server/share  /mnt/windows  cifs  \
    credentials=/root/.smbcredentials,\
    uid=1000,gid=1000,\
    file_mode=0644,dir_mode=0755,\
    vers=3.0  0  0

# vers=3.0  - SMB version
# noperm    - pas check permissions client-side
# noserverino - générer inodes côté client
```

##### Troubleshooting CIFS

```bash
# Tester connexion
smbclient -L //server -U user

# Lister shares
smbclient -L //server -N

# Se connecter interactivement
smbclient //server/share -U user

# Debug montage
sudo mount -t cifs //server/share /mnt/windows \
    -o username=user,password=pass,vers=3.0 -vvv
```

---

#### Monter Images ISO

##### Mount ISO

```bash
# Monter ISO
sudo mount -o loop image.iso /mnt/iso

# Lecture seule automatique
ls /mnt/iso

# Démonter
sudo umount /mnt/iso

# Dans fstab
/path/to/image.iso  /mnt/iso  iso9660  loop,ro,noauto  0  0

# Monter
sudo mount /mnt/iso
```

##### Créer ISO

```bash
# À partir de CD/DVD
dd if=/dev/cdrom of=backup.iso bs=4M

# À partir de dossier
genisoimage -o backup.iso -R -J /path/to/folder

# Ou avec mkisofs
mkisofs -o backup.iso -R -J /path/to/folder
```

---

#### Systèmes Fichiers Spéciaux

##### tmpfs (RAM)

```bash
# Monter tmpfs
sudo mount -t tmpfs -o size=1G tmpfs /mnt/ramdisk

# fstab
tmpfs  /mnt/ramdisk  tmpfs  size=1G,mode=1777  0  0

# Défaut /tmp est tmpfs sur beaucoup systèmes
df -h /tmp
```

##### /proc et /sys

```bash
# Déjà montés au boot
mount | grep proc
# proc on /proc type proc

mount | grep sys
# sysfs on /sys type sysfs

# Ne jamais démonter !
```

##### bind Mount

```bash
# Monter dossier ailleurs
sudo mount --bind /var/www /mnt/www-mirror

# fstab
/var/www  /mnt/www-mirror  none  bind  0  0

# Use case : chroot, containers
```

---

#### autofs : Montage Automatique

##### Installer autofs

```bash
sudo apt install autofs
```

##### Configuration autofs

```bash
# Master map
sudo nano /etc/auto.master

# Ajouter
/mnt/auto  /etc/auto.nfs  --timeout=60

# Map NFS
sudo nano /etc/auto.nfs

# Format : mount_point  options  location
data  -rw,soft,intr  nfs-server:/srv/data
backup  -ro  nfs-server:/srv/backup

# Redémarrer
sudo systemctl restart autofs

# Accès automatique
cd /mnt/auto/data  # Monte automatiquement
# Après 60s inactivité : démonte
```

##### autofs pour USB

```bash
# /etc/auto.master
/media/usb  /etc/auto.usb  --timeout=10

# /etc/auto.usb
*  -fstype=auto,nodev,nosuid,user  :/dev/&

# USB /dev/sdb1 → accessible à /media/usb/sdb1
```

---

#### Swap : Mémoire Virtuelle

##### Monter Swap

```bash
# Activer partition swap
sudo swapon /dev/sda2

# Voir swap actif
swapon --show

# Dans fstab
/dev/sda2  none  swap  sw  0  0
# Ou par UUID
UUID=xxx  none  swap  sw  0  0

# Désactiver
sudo swapoff /dev/sda2
```

##### Fichier Swap

```bash
# Créer fichier swap
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# fstab
/swapfile  none  swap  sw  0  0

# Vérifier
free -h
```

---

#### Monitoring Montages

##### Voir Utilisation Disque

```bash
# Espace disque
df -h

# Inodes
df -i

# Type filesystem
df -T

# Seulement filesystems réels
df -h -x tmpfs -x devtmpfs

# Utilisation dossier
du -sh /var/log
du -h --max-depth=1 /var | sort -rh | head
```

##### Surveiller I/O

```bash
# iostat
sudo apt install sysstat
iostat -x 2

# iotop (par processus)
sudo apt install iotop
sudo iotop

# dstat (tout)
sudo apt install dstat
dstat -cdngy
```

---

#### Troubleshooting

##### Montage Échoue

```bash
# Vérifier device existe
lsblk
ls -l /dev/sdb1

# Vérifier filesystem
sudo blkid /dev/sdb1

# Fsck si erreurs
sudo umount /dev/sdb1
sudo fsck -y /dev/sdb1

# Vérifier permissions point montage
ls -ld /mnt/data

# Logs
sudo journalctl -xe
dmesg | tail
```

##### fstab Cassé (Boot Échoue)

```bash
# Booter en rescue mode
# Ou appuyer 'e' au GRUB → ajouter 'single' → boot

# En rescue :
mount -o remount,rw /
nano /etc/fstab
# Commenter ligne problématique
reboot
```

##### NFS Mount Hang

```bash
# Monter avec soft option
mount -t nfs -o soft,timeo=30,intr server:/share /mnt/nfs

# Vérifier serveur accessible
ping nfs-server
showmount -e nfs-server

# Firewall ?
sudo ufw allow from 192.168.1.0/24 to any port nfs
```

---

#### Sécurité Montages

##### Options Sécurité

```bash
# Partitions données utilisateurs
/dev/sdb1  /home  ext4  defaults,nodev,nosuid  0  2

# Partitions temporaires
/dev/sdc1  /tmp  ext4  defaults,nodev,nosuid,noexec  0  2

# Partages réseau
nfs-server:/data  /mnt/nfs  nfs  ro,noexec,nosuid  0  0
```

##### Chiffrement

```bash
# LUKS (partition chiffrée)
sudo cryptsetup luksFormat /dev/sdb1
sudo cryptsetup luksOpen /dev/sdb1 encrypted_data

# Créer filesystem
sudo mkfs.ext4 /dev/mapper/encrypted_data

# Monter
sudo mount /dev/mapper/encrypted_data /mnt/secure

# fstab (avec keyfile)
/dev/mapper/encrypted_data  /mnt/secure  ext4  defaults  0  2

# /etc/crypttab
encrypted_data  /dev/sdb1  /root/keyfile  luks
```

---

#### Scripts Utiles

##### Vérifier Montages

```bash
#!/bin/bash
# check-mounts.sh

MOUNTS="/mnt/nfs /mnt/backup /mnt/data"

for mount in $MOUNTS; do
    if ! mountpoint -q "$mount"; then
        echo "WARNING: $mount not mounted"
        # Tenter remontage
        mount "$mount" 2>/dev/null || echo "Failed to mount $mount"
    fi
done
```

##### Backup Automatique

```bash
#!/bin/bash
# backup-to-nfs.sh

NFS_MOUNT="/mnt/backup"
SOURCE="/var/www"
DATE=$(date +%Y%m%d)

# Vérifier montage
if ! mountpoint -q "$NFS_MOUNT"; then
    mount "$NFS_MOUNT" || exit 1
fi

# Backup
tar -czf "$NFS_MOUNT/backup-$DATE.tar.gz" "$SOURCE"

# Rotation (30 jours)
find "$NFS_MOUNT" -name "backup-*.tar.gz" -mtime +30 -delete
```

---

#### Bonnes Pratiques

##### Checklist Montage

```bash
□ Utiliser UUID dans fstab (pas /dev/sdX)
□ Tester fstab avec 'mount -a' avant reboot
□ Options sécurité (nodev, nosuid, noexec)
□ Backup /etc/fstab avant modifications
□ Documenter montages critiques
□ Monitoring espace disque
□ Alertes si montage échoue
□ Permissions appropriées points montage
```

##### Performance

```bash
# noatime = pas update access time (performance)
UUID=xxx  /mnt/data  ext4  defaults,noatime  0  2

# async pour NFS (performance vs sécurité)
nfs:/data  /mnt/nfs  nfs  async,soft  0  0

# Augmenter buffer NFS
nfs:/data  /mnt/nfs  nfs  rsize=32768,wsize=32768  0  0
```

---

#### Conclusion

Les montages Linux permettent d'intégrer différents systèmes de fichiers :

**Commandes essentielles :**

```bash
mount /dev/sdb1 /mnt/data       # Monter
umount /mnt/data                # Démonter
findmnt                         # Lister montages
blkid                           # UUID devices
```

**fstab permanent :**

```bash
UUID=xxx  /mnt/data  ext4  defaults  0  2
```

**Types montages :**

- **Locaux** : ext4, ntfs, vfat
- **Réseau** : NFS, CIFS/SMB
- **Spéciaux** : tmpfs, bind, loop (ISO)

**Troubleshooting :**

```bash
lsof /mnt/data           # Qui utilise ?
fuser -km /mnt/data      # Tuer processus
fsck /dev/sdb1           # Vérifier filesystem
mount -a                 # Tester fstab
```

**Sécurité :**

```bash
mount -o ro,noexec,nosuid /dev/sdb1 /mnt/data
```

Avec ce guide, vous maîtrisez montage, fstab, NFS, CIFS et troubleshooting filesystems Linux !

Pour aller plus loin : consultez [partitionnement disque Linux](https://www.shpv.fr/blog/partitionnement-disque-linux-2026/) pour la préparation, [NFS détaillé](https://www.shpv.fr/blog/nfs-share/) pour le partage réseau, et [chiffrement LUKS](https://www.shpv.fr/blog/luks-chiffrement/) pour la sécurité.