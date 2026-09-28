---
link: https://slash-root.fr/docker-configuration-et-gestion-du-stockage/
site: slash-root.fr
date: 2025-06-15T16:56
excerpt: Comment configurer finement le stockage de docker engine.
slurped: 2026-09-23T06:53
title: "Docker : Configuration et gestion du stockage"
---

[![Featured image of post Docker : Configuration et gestion du stockage](app://obsidian.md/docker-configuration-et-gestion-du-stockage/cover.png)](app://obsidian.md/docker-configuration-et-gestion-du-stockage/)

[Docker](app://obsidian.md/categories/docker/)

### Comment configurer finement le stockage de docker engine.

Salut les admins sys et autres dompteurs de conteneurs ! Aujourd'hui, plongeons dans les profondeurs du stockage Docker. Comme on dit dans notre jargon : "Un disque mal configuré est à un admin système ce que Kryptonite est à Superman - une faiblesse qui peut tout paralyser."

## [](#comprendre-le-stockage-dans-docker)Comprendre le stockage dans Docker

Docker utilise un système de stockage en couches qui peut être à la fois génial et déroutant. Chaque instruction dans un Dockerfile crée une nouvelle couche en lecture seule, et seul le conteneur en fonctionnement dispose d'une couche supérieure en lecture-écriture.

Par défaut, Docker stocke toutes ses données (images, conteneurs, volumes) dans `/var/lib/docker`. Sur un serveur de production, cette partition peut rapidement devenir le goulot d'étranglement de votre système.

### [](#configuration-du-pilote-de-stockage)Configuration du pilote de stockage

Docker propose plusieurs pilotes de stockage, chacun avec ses avantages et inconvénients. Vous pouvez configurer le pilote à utiliser dans le fichier `/etc/docker/daemon.json` :

```
{
  "storage-driver": "overlay2"
}
```

> **Note :** `overlay2` est généralement le pilote recommandé pour les distributions Linux modernes, offrant le meilleur équilibre entre performance et fonctionnalités.

Après cette modification, un redémarrage du service Docker est nécessaire :

```
sudo systemctl restart docker
```

## [](#les-diff%c3%a9rents-pilotes-de-stockage)Les différents pilotes de stockage

1. **overlay2** : Le pilote recommandé pour la plupart des cas d'usage
    
    - Avantages : Bonnes performances, compatibilité
    - Prérequis : Noyau Linux 4.0+ et systèmes de fichiers modernes
    
    Le pilote overlay2 repose sur la fonctionnalité OverlayFS du noyau Linux, qui permet de superposer plusieurs systèmes de fichiers. Il fonctionne avec la plupart des systèmes de fichiers courants (ext4, xfs) et offre de bien meilleures performances que l'ancien pilote overlay.
    
2. **devicemapper** : Utilisé principalement sur les anciennes versions de CentOS/RHEL
    
    - Pour la production, configurez-le en mode `direct-lvm` plutôt qu'en mode loop-lvm par défaut
    - Permet d'utiliser un périphérique de bloc dédié pour les données Docker, offrant de meilleures performances

```
{
  "storage-driver": "devicemapper",
  "storage-opts": [
    "dm.directlvm_device=/dev/xvdf"
  ]
}
```

3. **Systèmes de fichiers avancés et distribués** : Pour aller plus loin
    
    - Les systèmes comme **btrfs** et **zfs** offrent des fonctionnalités avancées comme les snapshots
    - Les systèmes de fichiers distribués comme **Ceph** ou **GlusterFS** permettent de créer une infrastructure de stockage Docker hautement disponible et évolutive
    
    > **Astuce** : Pour tirer pleinement profit de ces systèmes distribués, utilisez la technique du "volume bind déguisé" qui permet de créer un volume Docker qui pointe vers un point de montage réseau
    

## [](#d%c3%a9placer-le-r%c3%a9pertoire-de-donn%c3%a9es-docker)Déplacer le répertoire de données Docker

Si votre partition `/var` est trop petite, vous pouvez déplacer le répertoire de données Docker vers un autre emplacement :

```
{
  "data-root": "/mnt/docker-data"
}
```

> **Attention :** Ne modifiez jamais ce paramètre sur une installation Docker existante sans migrer correctement les données !

Pour migrer les données d'une installation existante :

```
# Arrêter Docker
sudo systemctl stop docker

# Créer le nouveau répertoire
sudo mkdir -p /mnt/docker-data

# Copier les données existantes
sudo rsync -aP /var/lib/docker/ /mnt/docker-data/

# Configurer le nouveau chemin dans daemon.json
# Puis redémarrer Docker
sudo systemctl start docker
```

## [](#volumes-bind-d%c3%a9guis%c3%a9s--la-technique-du-ninja-du-stockage)Volumes bind déguisés : la technique du ninja du stockage

Une des techniques les plus astucieuses dans l'arsenal de l'admin Docker est celle du "volume bind déguisé" - ou comment faire passer un bind mount pour un volume Docker respectable, tout en gardant le contrôle sur l'emplacement physique des données.

Imaginez pouvoir dire à vos conteneurs : "Voici un volume Docker standard" alors qu'en coulisses, vous stockez les données sur votre cluster Ceph soigneusement configuré. C'est comme servir un plat végétarien à vos invités sans qu'ils ne remarquent l'absence de viande - l'élégance à l'état pur.

Cette technique est particulièrement précieuse quand vous travaillez avec :

- Des systèmes de stockage distribués (Ceph, GlusterFS)
- Des montages NFS pour un stockage partagé
- Des volumes qui doivent survivre à la migration de vos conteneurs

> **Pour les détails d'implémentation**, consultez mon article dédié : [Docker : Volume bind déguisé](app://obsidian.md/docker-volume-bind-deguise/) qui vous explique pas à pas comment mettre en place cette technique.

## [](#nettoyage-de-lespace-disque)Nettoyage de l’espace disque

Docker a tendance à accumuler des données inutilisées. Voici quelques commandes pour faire le ménage :

```
# Supprimer les conteneurs arrêtés, réseaux non utilisés, images dangling et cache de build
docker system prune

# Pour être encore plus agressif et supprimer aussi les images non utilisées
docker system prune -a

# Pour récupérer de l'espace sur les volumes non utilisés
docker volume prune
```

## [](#bonnes-pratiques-pour-la-gestion-du-stockage)Bonnes pratiques pour la gestion du stockage

1. **Utilisez des images légères** : Préférez les images basées sur Alpine quand c'est possible
    
2. **Optimisez vos Dockerfiles** :
    
    - Combinez les commandes RUN pour réduire le nombre de couches
    - Nettoyez les caches de paquets dans la même instruction qui les installe

```
RUN apt update && apt install -y \
    package1 \
    package2 \
 && apt clean \
 && rm -rf /var/lib/apt/lists/*
```

3. **Utilisez .dockerignore** pour exclure les fichiers inutiles du contexte de build
    
4. **Surveillez régulièrement** l'espace disque utilisé par Docker avec `docker system df`
    
5. **Configurez un cron job** pour nettoyer périodiquement les ressources inutilisées
    

## [](#conclusion)Conclusion

La gestion du stockage Docker ressemble parfois à un jeu de Tetris où les blocs seraient des gigaoctets - il faut savoir optimiser l'espace pour éviter le game over.

En configurant correctement votre pilote de stockage et en adoptant de bonnes pratiques de nettoyage, vous pourrez éviter ce moment gênant où vous devez expliquer pourquoi le serveur de production s'est arrêté juste parce qu'il y avait trop d'images "hello-world" qui traînaient.

Et souvenez-vous : dans un monde de conteneurs éphémères, les données persistantes méritent toujours une réflexion approfondie (et un bon volume Docker).