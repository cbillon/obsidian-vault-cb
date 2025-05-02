---
"tags:": disk, disque, partition
type: tips
---
## Identification du disque

Les commandes :
. lsblk   visualise les partions extistantes  -> /dev/sda1
. blkid /dev/sda1  affiche **uuid** du disque

## Créer le point de montage

```
sudo mkdir /data
```

## Montage temporaire

```
mount -t ext4 /dev/sda1  /data

```
le montage disparaît après le reboot du système

## Montage permanent

Ajouter la ligne dans /etc/fstab

```
/dev/disk/by-uuid/<uuid> /data ext4 default 0 1
```