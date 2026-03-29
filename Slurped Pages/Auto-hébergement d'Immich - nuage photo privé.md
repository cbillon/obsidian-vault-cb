---
link: https://www.glukhov.org/fr/post/2025/11/selfhosting-immich/
byline: À propos de Rost Glukhov
site: Rost Glukhov | Site personnel et blog technique
excerpt: Guide complet pour déployer Immich - la solution open source et
  auto-hébergée de gestion de photos et de vidéos avec des fonctionnalités
  alimentées par l'IA, la reconnaissance faciale et le backup mobile
  automatique. Prenez le contrôle de vos souvenirs avec une pleine
  confidentialité.
slurped: 2026-03-22T10:51
title: "Auto-hébergement d'Immich : nuage photo privé"
---

[Immich](https://www.glukhov.org/fr/post/2025/11/selfhosting-immich/ "Hébergement personnel d'Immich") est une solution révolutionnaire, open source et auto-hébergée pour la gestion de vos photos et vidéos, vous donnant un contrôle complet sur vos souvenirs. Avec des fonctionnalités rivales de celles de Google Photos, notamment la reconnaissance faciale alimentée par l’intelligence artificielle, la recherche intelligente et le sauvegarde automatique depuis les appareils mobiles, tout en maintenant vos données privées et sécurisées sur votre propre serveur.

De la même manière que [la gestion personnelle des connaissances](https://www.glukhov.org/fr/post/2025/07/personal-knowledge-management/ "Gestion personnelle des connaissances - Définition, objectifs, méthodes et outils à utiliser en 2025") vous aide à organiser et préserver vos pensées et informations, Immich vous aide à organiser et préserver vos souvenirs visuels.
## Qu’est-ce qu’Immich ?

Immich est une alternative open source et auto-hébergée aux services cloud propriétaires comme Google Photos et iCloud. Construit avec des technologies modernes, notamment TypeScript, PostgreSQL et l’apprentissage automatique, Immich propose une plateforme riche en fonctionnalités pour sauvegarder, organiser et parcourir votre collection de photos et de vidéos.

### Fonctionnalités clés

**Conception axée sur la confidentialité** : Toutes vos photos restent sur votre infrastructure. Aucun serveur tiers, aucune extraction de données, aucun problème de confidentialité. Vos souvenirs appartiennent à vous.

**Sauvegarde automatique depuis les appareils mobiles** : Des applications natives iOS et Android permettent une sauvegarde en arrière-plan automatique, similaire à Google Photos. Vos photos sont sécurisées quelques instants après les avoir prises.

**Recherche alimentée par l’intelligence artificielle** : En exploitant des modèles d’apprentissage automatique tels que CLIP et la reconnaissance faciale, Immich permet une recherche sémantique. Recherchez “coucher de soleil sur la plage”, “chien qui joue” ou des personnes spécifiques sans étiquetage manuel.

**Reconnaissance faciale** : Détection et regroupement automatique des visages dans vos photos. Immich identifie les personnes à travers toute votre bibliothèque, facilitant ainsi la recherche de photos de personnes spécifiques.

**Interface web moderne** : Une belle interface web réactive pour parcourir, organiser et partager vos photos depuis n’importe quel appareil doté d’un navigateur.

**Prise en charge multi-utilisateurs** : Créez des comptes pour les membres de votre famille, chacun disposant de sa propre bibliothèque privée. Partagez des albums sélectionnés tout en maintenant la confidentialité des photos personnelles.

**Prise en charge des photos Live** : Une prise en charge complète des photos Live iOS et des photos en mouvement Android, préservant à la fois l’image statique et la composante vidéo.

**Préservation des métadonnées** : Les données EXIF, y compris la localisation, les paramètres de l’appareil photo et les horodatages, sont préservées. Visualisez vos photos sur une carte basée sur les coordonnées GPS.

**Prise en charge des bibliothèques externes** : Importez des bibliothèques de photos existantes depuis un stockage externe sans copier les fichiers, économisant ainsi de l’espace disque. De la même manière que les outils comme [Obsidian](https://www.glukhov.org/fr/post/2025/07/obsidian-for-personal-knowledge-management/ "Utilisation d'Obsidian pour la gestion personnelle des connaissances (PKI)") aident à gérer et organiser les connaissances basées sur le texte, Immich propose une organisation puissante pour les médias visuels.

## Pourquoi auto-héberger vos photos ?

### Contrôle complet de la confidentialité

Lorsque vous utilisez des services cloud commerciaux, vos photos sont stockées sur des serveurs que vous ne contrôlez pas. Elles peuvent être analysées pour la publicité, incluses dans les données d’entraînement de l’apprentissage automatique ou consultées par des tiers. Avec Immich, vos photos ne quittent jamais votre serveur sauf si vous les partagez explicitement.

### Aucune limite de stockage

Les services cloud facturent en fonction des niveaux de stockage. Avec l’auto-hébergement, votre seule limite est la capacité matérielle. Un disque dur de 10 To coûte moins de deux ans de stockage cloud premium.

### Efficacité économique

Après l’investissement initial dans le matériel, l’auto-hébergement a des coûts minimaux. Aucun abonnement mensuel, aucune surprise lorsqu’on dépasse les limites de stockage.

### Durabilité des données

Les services cloud peuvent changer leurs conditions, augmenter les prix ou fermer complètement. Votre solution auto-hébergée reste sous votre contrôle indéfiniment.

### Opportunité d’apprentissage

L’auto-hébergement d’Immich offre une expérience pratique avec Docker, les proxys inversés, les certificats SSL, la gestion de base de données et l’administration de serveur - des compétences précieuses pour tout développeur ou professionnel DevOps.

## Aperçu de l’architecture

Immich suit une architecture en microservices avec plusieurs composants :

**Serveur Immich** : Serveur API principal gérant l’authentification, les uploads de photos et les opérations de base de données. Construit avec Node.js et TypeScript.

**Immich Machine Learning** : Service Python séparé exécutant des modèles TensorFlow pour la reconnaissance faciale, la détection d’objets et la recherche sémantique basée sur CLIP.

**Immich Web** : Interface web basée sur React fournissant l’application utilisateur.

**PostgreSQL** : Base de données relationnelle stockant les métadonnées, les informations utilisateur et les relations entre les photos, les personnes et les albums.

**Redis** : Cache en mémoire pour la gestion des sessions et la coordination de la file d’attente des tâches.

**TypeSense** (optionnel) : Moteur de recherche pour des capacités de recherche améliorées et des performances accrues.

Tous les composants s’exécutent en tant que conteneurs Docker, orchestrés avec Docker Compose pour un déploiement et une gestion simplifiés.

## Spécifications matérielles

### Spécifications minimales

- **CPU** : 2 cœurs (x86_64 ou ARM64)
- **RAM** : 4 Go (8 Go recommandés)
- **Stockage** : 10 Go pour l’application + la taille de la bibliothèque de photos
- **Réseau** : 100 Mbps pour l’accès local

### Spécifications recommandées

- **CPU** : 4+ cœurs avec de bonnes performances mono-thread
- **RAM** : 8-16 Go (plus pour de grandes bibliothèques)
- **Stockage** : SSD pour la base de données et l’application, HDD pour le stockage des photos
- **GPU** : Optionnel mais accélère significativement les tâches d’apprentissage automatique (NVIDIA avec support CUDA)

### Considérations sur le stockage

Prévoyez environ 1,15 fois la taille actuelle de votre bibliothèque de photos pour tenir compte des miniatures et des résolutions multiples. Utilisez un stockage SSD pour la base de données PostgreSQL pour une meilleure performance.

Pour de grandes bibliothèques (100 000+ photos), envisagez :

- NAS avec RAID pour la redondance des données
- SSD séparé pour la base de données
- Couche de cache NVMe pour les photos fréquemment accessibles

## Guide d’installation

### Prérequis

Avant d’installer Immich, assurez-vous d’avoir :

1. **Serveur Linux** : Ubuntu 22.04 LTS ou Debian 12 recommandés (consultez notre guide complet sur [Comment installer Ubuntu 24.04 et les outils utiles](https://www.glukhov.org/fr/developer-tools/local-dev-platforms/install-linux-ubuntu-24-04/ "Étapes et outils utiles pour l'installation d'Ubuntu 24.04") pour des instructions détaillées)
2. **Docker** : Version 20.10 ou plus récente
3. **Docker Compose** : Version 2.0 ou plus récente
4. **Nom de domaine** : Optionnel mais recommandé pour l’accès externe
5. **Proxy inverse** : Nginx ou Caddy pour la terminaison SSL

### Étapes d’installation

**1. Installer Docker et Docker Compose**

```
# Mettre à jour les paquets du système
sudo apt update && sudo apt upgrade -y

# Installer Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Ajouter l'utilisateur au groupe docker
sudo usermod -aG docker $USER

# Installer Docker Compose
sudo apt install docker-compose-plugin
```

Pour plus de détails sur les commandes curl et les options, consultez notre [Feuille de triche curl](https://www.glukhov.org/fr/developer-tools/api-tooling/curl-cheatsheet/ "Feuille de triche curl - liste et description des commandes curl utiles"). Si vous êtes nouveau avec Docker, notre [Feuille de triche Docker](https://www.glukhov.org/fr/developer-tools/containers/docker-cheatsheet/ "Liste et description des commandes Docker les plus fréquentes et utiles - feuille de triche Docker") fournit des commandes et concepts essentiels.

**2. Créer la structure de répertoires Immich**

```
# Créer le répertoire de l'application
mkdir -p ~/immich/{library,database,machine-learning}
cd ~/immich

# Télécharger docker-compose.yml
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml

# Télécharger le modèle d'environnement
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

Si vous êtes nouveau avec le scripting bash et les opérations en ligne de commande, notre [Feuille de triche bash](https://www.glukhov.org/fr/developer-tools/terminals-shell/bash-cheat-sheet/ "Feuille de triche bash") fournit une référence utile pour les commandes courantes et les opérations de répertoire.

**3. Configurer les variables d’environnement**

Éditez le fichier `.env` avec vos paramètres :

```
# Configuration de la base de données
DB_PASSWORD=your_secure_password_here
DB_DATABASE_NAME=immich
DB_USERNAME=postgres

# Emplacement de chargement
UPLOAD_LOCATION=./library

# Apprentissage automatique
MACHINE_LEARNING_ENABLED=true

# Fuseau horaire
TZ=America/New_York

# URL publique (pour l'accès externe)
IMMICH_SERVER_URL=https://photos.yourdomain.com
```

**Important** : Générez un mot de passe aléatoire fort pour `DB_PASSWORD` :

**4. Démarrer Immich**

```
# Démarrer tous les services
docker compose up -d

# Vérifier l'état des services
docker compose ps

# Afficher les journaux
docker compose logs -f
```

Ce sont juste les bases - pour une référence complète des commandes Docker Compose et des opérations, consultez notre [Feuille de triche Docker Compose](https://www.glukhov.org/fr/developer-tools/containers/docker-compose-cheatsheet/ "La liste des commandes Docker Compose les plus utiles, structures et exemples avec des descriptions : feuille de triche").

**5. Accéder à l’interface web**

Accédez à `http://your-server-ip:2283` et créez votre compte administrateur. Ce premier compte devient l’administrateur du système.

### Configuration de l’accès externe

Pour un accès externe sécurisé, configurez un proxy inverse :

**Exemple de configuration Nginx**

```
server {
    listen 443 ssl http2;
    server_name photos.yourdomain.com;

    ssl_certificate /path/to/fullchain.pem;
    ssl_certificate_key /path/to/privkey.pem;

    client_max_body_size 50000M;

    location / {
        proxy_pass http://localhost:2283;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        
        proxy_read_timeout 600s;
        proxy_send_timeout 600s;
    }
}
```

**Exemple de configuration Caddy** (plus simple avec HTTPS automatique)

```
photos.yourdomain.com {
    reverse_proxy localhost:2283
    
    @uploads {
        path /api/upload/*
    }
    request_body @uploads {
        max_size 50GB
    }
}
```

## Configuration de l’application mobile

### iOS

1. Téléchargez **Immich** depuis l’App Store
2. Entrez votre URL du serveur ([https://photos.yourdomain.com](https://photos.yourdomain.com/))
3. Connectez-vous avec vos identifiants
4. Activez la sauvegarde automatique dans les Paramètres
5. Choisissez les albums à sauvegarder (ou toutes les photos)
6. Configurez les paramètres de sauvegarde (uniquement sur Wi-Fi, uniquement lors de la recharge, etc.)

### Android

1. Téléchargez **Immich** depuis le Google Play Store ou F-Droid
2. Entrez l’URL du serveur et vos identifiants
3. Accordez les autorisations d’accès aux photos/vidéos
4. Configurez les paramètres de sauvegarde automatique
5. Activez le service en arrière-plan pour une sauvegarde fiable

### Conseils de configuration de la sauvegarde

- **Uniquement sur Wi-Fi** : Activez pour éviter les frais de données mobiles
- **Uniquement lors de la recharge** : Évitez la décharge de la batterie lors de grands transferts
- **Inclure les vidéos** : Les vidéos consomment beaucoup d’espace de stockage et de bande passante
- **Mise à jour de l’application en arrière-plan** : Activez sur iOS pour une synchronisation fiable
- **Service en arrière-plan** : Activez sur Android pour une sauvegarde constante

## Fonctionnalités d’apprentissage automatique

### Reconnaissance faciale

La reconnaissance faciale d’Immich détecte et regroupe automatiquement les visages dans votre bibliothèque :

1. **Traitement initial** : Après le téléchargement, le service ML analyse chaque photo pour les visages
2. **Regroupement des visages** : Les visages similaires sont regroupés
3. **Affectation manuelle** : Vérifiez les groupes et attribuez des noms aux personnes
4. **Apprentissage continu** : Lorsque vous étiquetez davantage de photos, la précision s’améliore

**Configuration** :

```
# Dans docker-compose.yml, environnement du service ML
MACHINE_LEARNING_MODEL_CACHE=/cache
MACHINE_LEARNING_WORKERS=1  # Augmentez avec plus de cœurs CPU
```

### Détection d’objets et recherche CLIP

Immich utilise CLIP (Contrastive Language-Image Pre-training) pour la recherche sémantique :

- Recherchez des concepts sans étiquettes : “paysage de montagne”, “gâteau d’anniversaire”, “voiture rouge”
- Les requêtes en langage naturel comprennent le contexte et les relations
- Fonctionne en plusieurs langues (bien que l’anglais donne généralement les meilleurs résultats)

### Accélération GPU

Pour une accélération significative du traitement ML, activez le support GPU :

**GPU NVIDIA avec CUDA**

```
# Dans docker-compose.yml, service ML
services:
  immich-machine-learning:
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

Assurez-vous d’avoir le NVIDIA Container Toolkit installé :

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
    sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo systemctl restart docker
```

## Stratégies de sauvegarde

Bien que Immich sauvegarde vos photos depuis les appareils mobiles, vous devez également sauvegarder Immich lui-même :

### Ce que sauvegarder

1. **Bibliothèque de photos** : Le répertoire `UPLOAD_LOCATION` contenant les photos originales
2. **Base de données** : Base de données PostgreSQL avec les métadonnées et les relations
3. **Configuration** : Fichier `.env` et `docker-compose.yml`

### Sauvegarde de la base de données

**Script de sauvegarde quotidienne automatisée**

```
#!/bin/bash
# enregistrer sous ~/immich/backup.sh

BACKUP_DIR=~/immich-backups
DATE=$(date +%Y%m%d_%H%M%S)

# Créer le répertoire de sauvegarde
mkdir -p $BACKUP_DIR

# Sauvegarder la base de données PostgreSQL
docker exec -t immich-postgres pg_dumpall -c -U postgres | \
    gzip > $BACKUP_DIR/immich_db_$DATE.sql.gz

# Conserver uniquement les 30 derniers jours de sauvegardes
find $BACKUP_DIR -name "immich_db_*.sql.gz" -mtime +30 -delete

echo "Sauvegarde terminée : $BACKUP_DIR/immich_db_$DATE.sql.gz"
```

Rendre exécutable et planifier avec cron :

```
chmod +x ~/immich/backup.sh
crontab -e
# Ajouter la ligne : 0 2 * * * ~/immich/backup.sh
```

Pour plus d’informations sur le scripting bash, l’automatisation et les tâches cron, consultez notre feuille de triche complète [Bash](https://www.glukhov.org/fr/developer-tools/terminals-shell/bash-cheat-sheet/ "Feuille de triche bash").

### Sauvegarde de la bibliothèque de photos

Votre bibliothèque de photos doit être sauvegardée séparément vers un autre emplacement :

**Option 1 : Rsync vers un NAS**

```
rsync -avz --delete ~/immich/library/ nas:/backups/immich-photos/
```

**Option 2 : Sauvegarde cloud (chiffrée)**

```
# En utilisant rclone avec le chiffrement
rclone sync ~/immich/library/ remote:immich-backup-encrypted/ --encrypt
```

**Option 3 : Disque externe local**

```
rsync -avz --delete ~/immich/library/ /mnt/backup-drive/immich/
```

## Maintenance et mises à jour

### Tâches de maintenance régulières

**Surveiller l’espace disque**

```
# Vérifier l'utilisation du disque
df -h ~/immich/library
df -h ~/immich/database

# Vérifier les volumes Docker
docker system df
```

**Surveiller les performances**

```
# Afficher l'utilisation des ressources
docker stats

# Vérifier les journaux de service spécifique
docker compose logs immich-server --tail=100
docker compose logs immich-machine-learning --tail=100
```

### Mise à jour d’Immich

Immich est régulièrement mis à jour avec de nouvelles fonctionnalités et des correctifs de bugs. Mettez-le à jour régulièrement :

```
cd ~/immich

# Sauvegarder la base de données avant la mise à jour
docker exec -t immich-postgres pg_dumpall -c -U postgres > backup_pre_update.sql

# Tirer les images les plus récentes
docker compose pull

# Arrêter et supprimer les anciens conteneurs
docker compose down

# Démarrer avec les nouvelles images
docker compose up -d

# Vérifier les journaux pour tout problème
docker compose logs -f
```

### Maintenance de la base de données

La maintenance périodique de la base de données assure une performance optimale :

```
# Nettoyage et analyse de la base de données
docker exec -it immich-postgres psql -U postgres -d immich -c "VACUUM ANALYZE;"

# Vérifier la taille de la base de données
docker exec -it immich-postgres psql -U postgres -d immich -c \
    "SELECT pg_size_pretty(pg_database_size('immich'));"
```

## Optimisation des performances

### Optimisation du stockage

**Utiliser un SSD pour la base de données** : PostgreSQL bénéficie considérablement du stockage SSD. Considérez :

```
volumes:
  pgdata:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /mnt/ssd/immich-db
```

**Stockage séparé des photos** : Gardez les grandes bibliothèques de photos sur HDD tout en maintenant la base de données sur SSD :

```
UPLOAD_LOCATION=/mnt/hdd/immich-photos
```

### Optimisation des performances de la base de données

Pour les bibliothèques avec plus de 50 000 photos, optimisez PostgreSQL :

```
# Dans docker-compose.yml, environnement du service postgres
POSTGRES_SHARED_BUFFERS=256MB
POSTGRES_EFFECTIVE_CACHE_SIZE=1GB
POSTGRES_MAINTENANCE_WORK_MEM=64MB
POSTGRES_CHECKPOINT_COMPLETION_TARGET=0.9
POSTGRES_WAL_BUFFERS=16MB
POSTGRES_DEFAULT_STATISTICS_TARGET=100
```

### Performance d’apprentissage automatique

**Traitement par lots** : Traitez simultanément plusieurs photos :

```
MACHINE_LEARNING_WORKERS=4  # Correspondre au nombre de cœurs CPU
```

**Accélération GPU** : Comme mentionné précédemment, l’accélération GPU fournit une accélération de 5 à 10 fois pour les tâches d’apprentissage automatique.

## Bonnes pratiques de sécurité

### Authentification et contrôle d’accès

1. **Mots de passe forts** : Utilisez un gestionnaire de mots de passe pour générer et stocker des mots de passe complexes
2. **Authentification à deux facteurs** : Activez la 2FA pour le compte administrateur (si pris en charge dans votre version)
3. **Révision régulière des accès** : Révisez périodiquement les comptes utilisateurs et supprimez les comptes inutilisés

### Sécurité réseau

**Proxy inverse avec SSL** : Ne laissez jamais Immich exposé directement à Internet sans HTTPS :

```
# Utilisez Let's Encrypt pour des certificats SSL gratuits
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d photos.yourdomain.com
```

**Configuration du pare-feu** :

```
# Autoriser uniquement les ports nécessaires
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP (rediriger vers HTTPS)
sudo ufw allow 443/tcp   # HTTPS
sudo ufw enable
```

**VPN ou Tailscale** : Pour une sécurité maximale, accédez à Immich via un VPN :

- Installez Tailscale sur votre serveur et vos appareils
- Accédez via l’IP Tailscale (100.x.x.x)
- Aucun port n’est exposé à Internet

### Sécurité des conteneurs

**Mises à jour régulières** : Maintenez les images Docker à jour pour corriger les vulnérabilités de sécurité

**Exécution sans root** : Configurez le mode rootless de Docker pour une isolation supplémentaire

**Limites de ressources** : Empêchez les attaques DDoS en limitant les ressources des conteneurs :

```
services:
  immich-server:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
```

## Dépannage des problèmes courants

### Échecs de téléchargement

**Symptôme** : Les photos ne se téléchargent pas depuis l’application mobile

**Solutions** :

- Vérifiez l’espace disque du serveur : `df -h`
- Vérifiez le paramètre `client_max_body_size` du proxy inverse
- Vérifiez les journaux du serveur : `docker compose logs immich-server`
- Assurez-vous que l’application mobile est à la dernière version

### L’apprentissage automatique ne traite pas

**Symptôme** : La reconnaissance faciale ou la recherche ne fonctionne pas

**Solutions** :

- Vérifiez l’état du conteneur ML : `docker compose ps immich-machine-learning`
- Affichez les journaux ML : `docker compose logs immich-machine-learning`
- Redémarrez le service ML : `docker compose restart immich-machine-learning`
- Vérifiez que les fichiers de modèle ont été téléchargés : `ls ~/immich/machine-learning/cache/`

### Erreurs de connexion à la base de données

**Symptôme** : L’interface web affiche des erreurs de connexion à la base de données

**Solutions** :

- Vérifiez que le conteneur PostgreSQL est en cours d’exécution : `docker compose ps immich-postgres`
- Vérifiez les journaux de la base de données : `docker compose logs immich-postgres`
- Vérifiez le mot de passe correct dans le fichier `.env`
- Testez la connexion : `docker exec -it immich-postgres psql -U postgres`

### Performance lente

**Symptôme** : L’interface web ou les recherches sont lentes

**Solutions** :

- Vérifiez les ressources système : CPU, RAM, I/O disque
- Exécutez le nettoyage de la base de données : `VACUUM ANALYZE;`
- Redémarrez les services : `docker compose restart`
- Révisez les paramètres de performance de PostgreSQL
- Considérez une mise à niveau matérielle (SSD, plus de RAM)

### L’application mobile ne synchronise pas

**Symptôme** : Les photos ne sont pas sauvegardées depuis l’appareil mobile

**Solutions** :

- Vérifiez que la mise à jour en arrière-plan est activée (iOS)
- Activez le service en arrière-plan (Android)
- Vérifiez les paramètres Wi-Fi uniquement si vous êtes sur la 4G
- Vérifiez que l’URL du serveur est accessible depuis le réseau mobile
- Révisez les autorisations d’accès aux photos
- Effacez le cache de l’application et reconnectez-vous

## Migration depuis d’autres services

### Depuis Google Photos

**Exporter vos données** :

1. Allez sur Google Takeout (takeout.google.com)
2. Sélectionnez Google Photos
3. Choisissez le format d’exportation et la taille
4. Téléchargez les archives

**Importer vers Immich** :

1. Décompressez les archives téléchargées
2. Utilisez l’outil CLI d’Immich ou le téléchargement web
3. Les métadonnées sont préservées à partir des fichiers JSON sidecar

```
# En utilisant l'outil CLI d'Immich (téléchargez depuis les releases)
immich upload --recursive /path/to/google-photos-export/
```

### Depuis iCloud Photos

**Exporter** :

1. Visitez iCloud.com
2. Sélectionnez les photos à télécharger
3. Ou utilisez iCloud pour Windows/Photos app export

**Importer** : Similaire à Google Photos, téléchargez via l’interface web ou l’outil CLI

### Depuis le stockage local

**Fonctionnalité de bibliothèque externe** : Au lieu de télécharger, pointez Immich vers un répertoire de photos existant :

1. Accédez à l’Administration > Bibliothèques externes
2. Ajoutez le chemin de la bibliothèque (doit être accessible depuis le conteneur Docker)
3. Immich scanne et indexe sans copier les fichiers
4. Économise un espace disque et du temps

```
# Dans docker-compose.yml, ajoutez le montage de volume
volumes:
  - /mnt/existing-photos:/mnt/external-library:ro
```

## Communauté et support

### Ressources officielles

- **GitHub** : [https://github.com/immich-app/immich](https://github.com/immich-app/immich)
- **Documentation** : [https://immich.app/docs](https://immich.app/docs)
- **Discord** : Communauté active pour l’aide en temps réel
- **Reddit** : r/immich pour les discussions et les conseils

### Contribution

Immich est open source et accueille les contributions :

- **Signalement de bugs** : Soumettez des rapports détaillés sur GitHub
- **Demandes de fonctionnalités** : Discutez sur Discord ou GitHub Discussions
- **Contributions de code** : Soumettez des demandes de tirage suivant les lignes directrices de contribution
- **Documentation** : Améliorez la documentation pour une meilleure expérience utilisateur
- **Traductions** : Aidez à localiser Immich dans votre langue

### Outils alternatifs

Si Immich ne répond pas à vos besoins, envisagez :

- **PhotoPrism** : Plus mature, riche en fonctionnalités, architecture différente
- **Piwigo** : Interface de galerie traditionnelle
- **Nextcloud Photos** : Partie de l’écosystème Nextcloud
- **Photoview** : Alternative plus simple et légère
- **LibrePhotos** : Autre alternative à Google Photos avec des fonctionnalités d’IA

## Conclusion

L’auto-hébergement d’Immich offre une alternative puissante et privée aux services cloud commerciaux de gestion de photos. Bien qu’il exige un setup technique initial et une maintenance continue, les avantages du contrôle complet de la confidentialité, l’absence de limites de stockage et la liberté d’éviter les frais d’abonnement en font une option valable pour de nombreux utilisateurs.

La combinaison de la sauvegarde automatique depuis les appareils mobiles, de la recherche alimentée par l’intelligence artificielle, de la reconnaissance faciale et de l’interface moderne crée une expérience comparable à celle des offres commerciales tout en maintenant vos données sous votre contrôle. Que vous soyez un passionné de confidentialité, un utilisateur économique avec de grandes bibliothèques de photos ou un professionnel technique souhaitant avoir une expérience pratique d’infrastructure, Immich propose une solution convaincante.

Commencez petit avec un serveur domestique ou un VPS, testez avec un sous-ensemble de votre bibliothèque de photos, et migrez progressivement à mesure que vous devenez familier avec le système. La communauté active et les mises à jour fréquentes assurent que Immich continuera à s’améliorer et à ajouter des fonctionnalités.

Vos souvenirs sont précieux - prenez le contrôle d’eux avec Immich.

## Liens utiles

- **Site officiel** : [https://immich.app](https://immich.app/)
- **Dépôt GitHub** : [https://github.com/immich-app/immich](https://github.com/immich-app/immich)
- **Documentation** : [https://immich.app/docs/overview/introduction](https://immich.app/docs/overview/introduction)
- **Guide d’installation** : [https://immich.app/docs/install/docker-compose](https://immich.app/docs/install/docker-compose)
- **Communauté Discord** : [https://discord.gg/immich](https://discord.gg/immich)
- **Applications mobiles** :
    - iOS : [https://apps.apple.com/app/immich/id1613945652](https://apps.apple.com/app/immich/id1613945652)
    - Android : [https://play.google.com/store/apps/details?id=app.alextran.immich](https://play.google.com/store/apps/details?id=app.alextran.immich)
- **Docker Hub** : [https://hub.docker.com/r/ghcr.io/immich-app](https://hub.docker.com/r/ghcr.io/immich-app)
- **Notes de version** : [https://github.com/immich-app/immich/releases](https://github.com/immich-app/immich/releases)
- **Awesome Immich** : [https://github.com/varun-raj/awesome-immich](https://github.com/varun-raj/awesome-immich) (Ressources de la communauté)
- **r/immich** : [https://reddit.com/r/immich](https://reddit.com/r/immich)