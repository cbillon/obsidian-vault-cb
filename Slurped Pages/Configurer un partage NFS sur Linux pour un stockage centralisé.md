---
link: https://www.shpv.fr/blog/nfs-share/
byline: SHPV
site: SHPV
date: 2025-07-12T12:00
excerpt: Guide pas à pas pour installer et configurer un serveur et un client NFS sous Linux, permettant de partager des répertoires en réseau de manière sécurisée et performante.
twitter: https://twitter.com/@SHPVFR
tags:
  - linux
  - nfs
slurped: 2025-10-12T18:48
title: Configurer un partage NFS sur Linux pour un stockage centralisé
---

Publié le 12 juillet 2025

Stockage

Réseau

Administration

Le Network File System (NFS) permet de partager des répertoires entre plusieurs serveurs ou postes de travail sous Linux. Ce guide détaille l’installation du serveur NFS et la configuration des clients pour un **stockage centralisé**.

## Prérequis

- Un serveur Linux (Debian, Ubuntu, CentOS, RHEL)
- Accès root ou sudo sur le serveur et les clients
- Réseau fiable (LAN)

## 1. Installation du serveur NFS

### Debian/Ubuntu

```
sudo apt update
sudo apt install nfs-kernel-server -y
```

### RHEL/CentOS

```
sudo dnf install nfs-utils -y
sudo systemctl enable --now nfs-server
```

## 2. Configuration des exports

Éditez `/etc/exports` et ajoutez :

```
/srv/nfs/share 192.168.1.0/24(rw,sync,no_subtree_check)
```

- `rw` : lecture/écriture
- `sync` : écrit les données immédiatement
- `no_subtree_check` : évite certains problèmes de permission

Appliquez la configuration :

```
sudo exportfs -ra
```

## 3. Ouverture dans le firewall

```
sudo ufw allow from 192.168.1.0/24 to any port nfs
```

OU pour firewalld :

```
sudo firewall-cmd --add-service=nfs --permanent
sudo firewall-cmd --reload
```

## 4. Montage sur le client

### Créer le point de montage

```
sudo mkdir -p /mnt/nfs/share
```

### Monter manuellement

```
sudo mount 192.168.1.10:/srv/nfs/share /mnt/nfs/share
```

### Montage automatique

Ajoutez dans `/etc/fstab` :

```
192.168.1.10:/srv/nfs/share /mnt/nfs/share nfs defaults,_netdev 0 0
```

## 5. Tests et vérification

```
df -h | grep nfs
touch /mnt/nfs/share/testfile
ls /mnt/nfs/share
```

## Conclusion

NFS est une solution simple et efficace pour partager des répertoires sur un réseau Linux. Vous avez maintenant un partage NFS sécurisé et prêt à l’emploi.
