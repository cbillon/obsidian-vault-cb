---
link: https://www.shpv.fr/blog/restic-sauvegarde/
byline: SHPV
site: SHPV
date: 2025-06-27T12:00
excerpt: Tutoriel détaillé pour mettre en place des sauvegardes automatisées, chiffrées et incrémentales sur un serveur Linux en utilisant Restic et un backend SFTP.
twitter: https://twitter.com/@SHPVFR
tags:
  - sauvegarde
  - linux
slurped: 2025-10-12T19:03
title: Automatiser les sauvegardes sur un serveur Linux avec Restic et un stockage distant SFTP
---

La sauvegarde régulière de vos serveurs est cruciale. Restic est un outil moderne, rapide et sécurisé permettant de réaliser des sauvegardes chiffrées et incrémentales vers divers backends dont SFTP.

## Prérequis

- Serveur Linux (Debian, Ubuntu, CentOS, Rocky…)
- Accès SSH à un serveur de stockage distant
- Restic installé localement

## Installation de Restic

### Debian/Ubuntu

```
sudo apt update
sudo apt install restic -y
```

### RHEL/AlmaLinux/Rocky

```
sudo dnf install epel-release -y
sudo dnf install restic -y
```

## Initialiser le dépôt Restic

```
export RESTIC_REPOSITORY="sftp:user@backupserver:/data/restic"
export RESTIC_PASSWORD="MotDePasseDeSauvegarde"

restic init
```

## Lancer une première sauvegarde

```
restic backup /etc /var/www /home
```

## Vérifier les snapshots

```
restic snapshots
```

## Programmer la sauvegarde avec cron

Éditer la crontab :

```
crontab -e
```

Exemple de job quotidien à 3h00 :

```
0 3 * * * RESTIC_REPOSITORY="sftp:user@backupserver:/data/restic" RESTIC_PASSWORD="VotreMotDePasse" /usr/bin/restic backup /etc /var/www /home >> /var/log/restic-backup.log 2>&1
```

## Nettoyage automatique

Pour limiter la taille du dépôt :

```
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
```

## Vérification d’intégrité

```
restic check
```

## Conclusion

Avec Restic et un backend SFTP, vous disposez d’une solution de sauvegarde sécurisée, fiable et économique adaptée à tout type de serveur Linux.

###### Besoin d'aide sur ce sujet ?

Notre équipe d'experts est là pour vous accompagner dans vos projets.

[Contactez-nous](https://www.shpv.fr/demander-un-devis)

---

##### Articles similaires qui pourraient vous intéresser