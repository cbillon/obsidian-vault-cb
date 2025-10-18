---
link: https://www.shpv.fr/blog/borgbackup-postgresql/
byline: SHPV
site: SHPV
date: 2025-07-13T09:00
excerpt: Déployez une stratégie fiable pour la sauvegarde et le monitoring automatisés de vos bases PostgreSQL grâce à BorgBackup, avec alertes et vérification d'intégrité.
twitter: https://twitter.com/@SHPVFR
tags:
  - postgresql
  - backup
slurped: 2025-10-12T18:45
title: Automatiser les sauvegardes PostgreSQL avec BorgBackup et monitoring
---

Publié le 13 juillet 2025

Sauvegarde

Base de données

Automatisation

Sauvegarder ses bases PostgreSQL est un enjeu critique pour toute organisation. Coupler **BorgBackup** à un script d’automatisation et une supervision active vous garantit des restaurations fiables et un contrôle en temps réel de la chaîne de backup.

## Prérequis

- PostgreSQL (>=12)
- BorgBackup installé (sur source et cible)
- Serveur Linux (Debian, Ubuntu, Rocky…)
- Droits sudo/root
- Système de supervision (Prometheus Node Exporter, Zabbix agent, etc.)

## Installation de BorgBackup

### 1. Installer BorgBackup

```
sudo apt update && sudo apt install borgbackup -y
```

### 2. Initialiser le dépôt distant

```
export BORG_REPO=ssh://backup@backup-server:/data/borg/postgresql
borg init --encryption=repokey $BORG_REPO
```

## Script de sauvegarde PostgreSQL avec Borg

Sauvegarde automatisée et rotation :

```
#!/bin/bash
set -e

DATE=$(date +'%Y-%m-%d')
export BORG_REPO=ssh://backup@backup-server:/data/borg/postgresql

# Dump PostgreSQL (remplacez les variables selon votre config)
PGUSER="backup"
PGDATABASE="ma_base"
pg_dump -U $PGUSER $PGDATABASE | gzip > /tmp/db-$DATE.sql.gz

# Sauvegarde avec Borg
borg create --stats $BORG_REPO::db-$DATE /tmp/db-$DATE.sql.gz

# Rotation (7 quotidiennes, 4 hebdo, 6 mensuelles)
borg prune -v --keep-daily=7 --keep-weekly=4 --keep-monthly=6 $BORG_REPO

rm /tmp/db-$DATE.sql.gz
```

Planifiez ce script via `cron` :

```
0 2 * * * /root/backup_postgresql_borg.sh >> /var/log/borg_pg.log 2>&1
```

## Monitoring et alertes

### 1. Vérification automatique de l’intégrité

Ajoutez une tâche :

```
borg check $BORG_REPO
```

### 2. Exporter les stats de sauvegarde

- Utilisez un script pour publier les résultats (taille, statut, durée) dans un fichier ou une base Prometheus.
- Créez une alerte en cas d’échec ou d’absence de sauvegarde récente.

### 3. Exemple de métrique Prometheus

```
backup_success{job="postgresql-borg"} 1
backup_last_duration_seconds{job="postgresql-borg"} 871
backup_last_size_bytes{job="postgresql-borg"} 14222345
```

## Bonnes pratiques

- **Chiffrer toutes les sauvegardes**
- **Isoler le dépôt backup du serveur de prod**
- **Tester la restauration régulièrement**
- **Superviser chaque étape (dump, upload, rotation, check)**
- **Documenter la procédure de disaster recovery**

## Avantages

- Compression et déduplication efficaces
- Monitoring avancé et alertes en cas d’échec
- Automatisation totale (cron + scripts + monitoring)
- Restauration granulaire (fichiers individuels ou dump complet)

## Limites

- Dépendance au script pour le dump PostgreSQL (pas de snapshot natif)
- Besoin de capacité réseau/disque sur la cible backup
- Complexité croissante avec la volumétrie ou le multibase

## Conclusion

En couplant BorgBackup à des scripts intelligents et une supervision active, vous sécurisez la sauvegarde PostgreSQL, détectez proactivement les anomalies, et vous vous donnez les moyens d’une restauration efficace, même dans les situations les plus critiques.