---
link: https://www.shpv.fr/blog/sauvegarde-restic-minio-rclone/
byline: SHPV
site: SHPV
date: 2025-05-28T12:00
excerpt: Déployez une infrastructure de sauvegarde chiffrée, dédupliquée et répliquée entre plusieurs datacenters avec des outils open source performants.
twitter: https://twitter.com/@SHPVFR
tags:
  - restic
  - minio
  - rclone
  - backup
slurped: 2025-10-12T19:14
title: Construire une plateforme de sauvegarde multi-sites avec Restic, MinIO et Rclone
---

Dans un contexte de cybermenaces croissantes, de conformité réglementaire et de nécessité de résilience, une **plateforme de sauvegarde distribuée, chiffrée et dédupliquée** est indispensable. Ce guide vous montre comment construire une infrastructure de sauvegarde multi-sites à l’aide de trois outils open source puissants : **Restic**, **MinIO** et **Rclone**.

## Prérequis

Avant de commencer, assurez-vous d’avoir :

- Deux sites distants (ou au moins deux serveurs)
- Un accès root ou sudo
- Docker et Docker Compose installés
- Des volumes de stockage suffisants
- Un nom de domaine (optionnel, pour sécuriser avec HTTPS)

## Technologies utilisées

- **Restic** : Outil de sauvegarde moderne, chiffré, dédupliqué
- **MinIO** : Stockage objet compatible S3 auto-hébergé
- **Rclone** : Outil de synchronisation entre systèmes compatibles S3
- **Docker Compose** : Déploiement simplifié
- **Systemd ou cron** : Automatisation des sauvegardes
- **Prometheus + Restic Exporter** (optionnel) : Supervision

## Architecture cible

```
SITE A                             SITE B
+-------------------+             +-------------------+
| Restic + Docker   |             | Restic + Docker   |
| MinIO (local S3)  | <=====>     | MinIO (réplica)   |
| Rclone Sync       |             | Rclone Sync       |
+-------------------+             +-------------------+
```

## Déploiement de MinIO avec Docker

```
mkdir -p ~/minio/data
cd ~/minio

nano docker-compose.yml
```

```
version: '3.7'
services:
  minio:
    image: quay.io/minio/minio
    container_name: minio
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - ./data:/data
    environment:
      MINIO_ROOT_USER: admin
      MINIO_ROOT_PASSWORD: strongpassword
    command: server /data --console-address ":9001"
    restart: always
```

```
docker compose up -d
```

Accédez à l’interface via `http://<ip_serveur>:9001`, connectez-vous, puis créez un bucket `restic-backups`.

## Configuration de Restic

### Installation

```
sudo apt install restic
```

### Initialisation

```
export AWS_ACCESS_KEY_ID=admin
export AWS_SECRET_ACCESS_KEY=strongpassword
export RESTIC_REPOSITORY=s3:http://localhost:9000/restic-backups

restic init
```

### Sauvegarde

```
restic backup /etc /var/www /home
```

### Listing & restauration

```
restic snapshots
restic restore <snapshot_id> --target /tmp/restore
```

## Réplication entre sites avec Rclone

### Installation

```
curl https://rclone.org/install.sh | sudo bash
```

### Configuration des remotes

```
rclone config
```

Créez deux remotes : `site-a` et `site-b` en utilisant le type **S3 compatible**. Utilisez les credentials MinIO de chaque site.

### Synchronisation manuelle

```
rclone sync site-a:restic-backups site-b:restic-backups
```

Cette opération peut être cronée ou exécutée via `systemd`.

## Automatisation avec systemd

Créez un service :

```
# /etc/systemd/system/restic-backup.service
[Unit]
Description=Sauvegarde Restic quotidienne

[Service]
Environment="AWS_ACCESS_KEY_ID=admin"
Environment="AWS_SECRET_ACCESS_KEY=strongpassword"
ExecStart=/usr/bin/restic backup /etc /var/www /home
```

Créez un timer :

```
# /etc/systemd/system/restic-backup.timer
[Unit]
Description=Timer de sauvegarde quotidienne

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```

```
systemctl daemon-reexec
systemctl enable --now restic-backup.timer
```

## Supervision avec Prometheus (optionnel)

Utilisez `restic-exporter` :

```
docker run -d -p 8000:8000   -e RESTIC_REPOSITORY=s3:http://minio:9000/restic-backups   -e AWS_ACCESS_KEY_ID=admin   -e AWS_SECRET_ACCESS_KEY=strongpassword   ghcr.io/creativeprojects/restic-exporter
```

Scrapez l’exporter avec Prometheus et visualisez l’état des snapshots avec Grafana.

## Maintenance

### Nettoyage automatisé

```
restic forget --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --prune
```

### Mise à jour

```
docker compose pull && docker compose up -d
sudo apt update && sudo apt install --only-upgrade restic
```

## Avantages de cette solution

|Fonctionnalité|Implémentation|
|---|---|
|Chiffrement|✅ AES-256 intégré (Restic)|
|Déduplication|✅ Bloc par bloc|
|Compatibilité S3|✅ MinIO|
|Réplication inter-site|✅ Rclone|
|Monitoring|✅ Prometheus exporter disponible|
|Auto-hébergement|✅ Sans dépendance cloud|
|Résilience|✅ Données répliquées|
|Coût|⭐⭐⭐⭐⭐ (open source)|
|Complexité|⭐⭐⭐|

## Cas d’usage

- Sauvegarde d’infrastructures critiques
- Réplication entre datacenters
- Remplacement de solutions propriétaires (Veeam, Acronis)
- Conformité (chiffrement, rétention)

## Conclusion

En combinant **Restic**, **MinIO** et **Rclone**, vous obtenez une plateforme de sauvegarde robuste, performante et distribuée. C’est une excellente alternative souveraine et open source aux solutions cloud. Avec un minimum de ressources, vous pouvez mettre en œuvre une stratégie 3-2-1 (3 copies, 2 supports, 1 site distant) fiable et scalable.