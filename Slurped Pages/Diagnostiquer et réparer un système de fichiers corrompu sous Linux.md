---
link: https://www.shpv.fr/blog/reparer-fs-linux/
byline: SHPV
site: SHPV
date: 2025-06-23T12:00
excerpt: Un guide complet pour identifier, diagnostiquer et réparer les systèmes de fichiers Linux corrompus, avec les outils fsck, e2fsck, xfs_repair et btrfs check.
twitter: https://twitter.com/@SHPVFR
tags:
  - linux
  - repair
slurped: 2025-10-12T19:06
title: Diagnostiquer et réparer un système de fichiers corrompu sous Linux
---

Les systèmes de fichiers Linux peuvent être corrompus suite à des coupures électriques, des erreurs matérielles ou des bugs logiciels. Ce guide vous explique comment **diagnostiquer** et **réparer** ces problèmes pour restaurer la santé de votre système.

## Symptômes d’un système de fichiers corrompu

- Erreurs au démarrage, blocages ou plantages
- Partition qui ne se monte plus
- Messages d’erreur comme `input/output error` ou `EXT4-fs error`
- Fichiers corrompus ou disparus
- Serveur en mode rescue ou emergency

---

## Étape 1 : Démarrer en mode rescue ou live USB

Pour éviter d’aggraver la situation, démarrez sur une clé USB live Linux ou un mode rescue.

Ouvrez un terminal et tapez la commande `sudo su -` pour passer en root.

---

## Étape 2 : Identifier les partitions concernées

Listez vos disques et partitions avec :

```
lsblk
blkid
```

Notez les partitions à vérifier, par exemple `/dev/sda1`, `/dev/sdb2`.

---

## Étape 3 : Vérifier le système de fichiers avec fsck

### Ext4 / Ext3 / Ext2

```
fsck -f /dev/sda1
```

Ou plus précisément :

```
e2fsck -f /dev/sda1
```

Répondez `y` pour réparer les erreurs détectées.

### XFS

XFS nécessite l’outil `xfs_repair` :

```
xfs_repair /dev/sdb1
```

> Attention : la partition ne doit pas être montée.

### Btrfs

Pour Btrfs, utilisez :

```
btrfs check --repair /dev/sdc1
```

> Sauvegardez vos données avant cette opération, car `--repair` peut être destructif.

---

## Étape 4 : Forcer une vérification au prochain démarrage

Pour forcer une vérification au boot sur `/dev/sda1` :

```
tune2fs -c 1 /dev/sda1
```

---

## Étape 5 : Que faire si fsck échoue ?

- Tentez une récupération de données avec `photorec` ou `testdisk`
- Remplacez le disque si matériellement défectueux
- Envisagez une restauration à partir de sauvegardes

---

## Cas pratique

Un serveur Debian ne démarre plus et affiche une erreur EXT4 :

1. Boot sur live USB
2. Identifier la partition root `/dev/sda2`
3. `e2fsck -f /dev/sda2`
4. Correction des erreurs
5. Redémarrage et vérification

---

## Conclusion

La maintenance régulière et la surveillance des systèmes de fichiers est cruciale en environnement de production. `fsck` et ses outils associés sont vos alliés pour garantir la disponibilité et la fiabilité de vos données.

---

## Ressources

- [man fsck](https://linux.die.net/man/8/fsck)
- [e2fsck documentation](https://e2fsprogs.sourceforge.io/ext2.html)
- [xfs_repair guide](https://xfs.org/index.php/XFS_FAQ#How_do_I_repair_a_broken_XFS_file_system.3F)
- [Btrfs wiki](https://btrfs.wiki.kernel.org/index.php/Main_Page)

###### Besoin d'aide sur ce sujet ?

Notre équipe d'experts est là pour vous accompagner dans vos projets.

[Contactez-nous](https://www.shpv.fr/demander-un-devis)

---

##### Articles similaires qui pourraient vous intéresser