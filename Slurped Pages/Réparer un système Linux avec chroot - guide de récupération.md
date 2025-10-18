---
link: https://www.shpv.fr/blog/reparer-linux-avec-chroot/
byline: SHPV
site: SHPV
date: 2025-06-21T12:00
excerpt: Un guide simple et efficace pour accéder et réparer un système Linux endommagé à l'aide de chroot depuis un environnement live.
twitter: https://twitter.com/@SHPVFR
tags:
  - linux
  - repair
  - chroot
slurped: 2025-10-12T19:09
title: "Réparer un système Linux avec chroot : guide de récupération"
---

Quand un système Linux refuse de démarrer — suite à une mauvaise mise à jour, une erreur de configuration du bootloader ou une corruption système — il est possible de le réparer **depuis un environnement live** en utilisant la commande `chroot`. Ce guide vous montre pas à pas comment effectuer cette récupération.

## Qu’est-ce que `chroot` ?

`chroot` permet de changer la racine apparente (`/`) d’un processus vers un autre répertoire. Cela permet d’**émuler un environnement complet**, comme si vous étiez connecté directement au système installé sur disque, même depuis un live CD.

---

## Prérequis

- Une clé USB bootable avec une distribution Linux live (ex: Ubuntu, Debian)
- Un accès root dans l’environnement live
- Le système de fichiers du disque dur encore accessible

---

## Étapes de récupération avec chroot

### 1. Démarrer sur un live CD

Démarrez votre machine sur une clé USB live.

Ouvrez un terminal avec privilèges root :

```
sudo su -
```

### 2. Identifier les partitions système

Utilisez `lsblk`, `fdisk -l` ou `blkid` pour repérer les partitions (/, /boot, /boot/efi, /home...)

```
lsblk
```

Exemple :

```
sda
├─sda1 /boot/efi
├─sda2 /
├─sda3 /home
```

### 3. Monter la partition root

```
mount /dev/sda2 /mnt
```

Montez ensuite les partitions additionnelles si nécessaire :

```
mount /dev/sda1 /mnt/boot/efi
mount /dev/sda3 /mnt/home
```

### 4. Monter les pseudo-systèmes nécessaires

```
mount --bind /dev /mnt/dev
mount --bind /proc /mnt/proc
mount --bind /sys /mnt/sys
mount --bind /run /mnt/run
```

### 5. Chrooter dans le système installé

```
chroot /mnt
```

Vous êtes maintenant "dans" le système installé, comme si vous aviez démarré dessus.

---

## Ce que vous pouvez faire depuis le chroot

- Réinstaller ou mettre à jour GRUB :

```
grub-install /dev/sda
update-grub
```

- Réinitialiser le mot de passe root :

```
passwd
```

- Réparer un fichier de configuration (fstab, sources.list, network...)
    
- Mettre à jour les paquets :
    

```
apt update && apt upgrade
```

---

## Sortir du chroot et démonter proprement

```
exit
umount -l /mnt/dev /mnt/proc /mnt/sys /mnt/run
umount -l /mnt/boot/efi
umount -l /mnt/home
umount -l /mnt
```

Puis redémarrez :

```
reboot
```

---

## Astuces & conseils

- Si `chroot` ne fonctionne pas correctement (ex : commandes absentes), vérifiez que `/mnt` contient `/bin`, `/lib` etc.
- Vous pouvez utiliser `arch-chroot` (Arch Linux) ou `debootstrap` si les binaires système sont absents
- Pensez à activer l’accès au réseau dans le chroot si besoin :

```
cp /etc/resolv.conf /mnt/etc/resolv.conf
```

---

## Conclusion

La commande `chroot` est un outil puissant pour dépanner un système Linux sans avoir à tout réinstaller. Elle permet d’accéder à un système cassé, de réparer le bootloader, de corriger des erreurs de configuration ou même de réinitialiser un mot de passe.

---

## Ressources complémentaires

- [Manpage de chroot](https://man7.org/linux/man-pages/man2/chroot.2.html)
- [GRUB rescue tutorial](https://wiki.ubuntu.com/Grub2/Troubleshooting)
- [Debian chroot recovery](https://wiki.debian.org/RescueChroot)