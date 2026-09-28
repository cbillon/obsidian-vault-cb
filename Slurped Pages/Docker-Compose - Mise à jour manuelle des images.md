---
link: https://slash-root.fr/docker-compose-mise-a-jour-manuelle-des-images/
site: slash-root.fr
date: 2024-07-01T11:15
excerpt: Aide-mémoire pour mettre à jour les images des conteneurs d’un fichier
  docker-compose.yml
slurped: 2026-09-23T06:56
title: "Docker-Compose : Mise à jour manuelle des images"
---

[![Featured image of post Docker-Compose : Mise à jour manuelle des images](app://obsidian.md/docker-compose-mise-a-jour-manuelle-des-images/cover.png)](app://obsidian.md/docker-compose-mise-a-jour-manuelle-des-images/)

[Docker](app://obsidian.md/categories/docker/)

### Aide-mémoire pour mettre à jour les images des conteneurs d’un fichier docker-compose.yml

## [](#pr%c3%a9-requis)Pré-requis

Assurez-vous de vous placer dans le répertoire contenant le fichier `docker-compose.yml` avant de suivre les étapes ci-dessous.

## [](#%c3%a9tapes-pour-la-mise-%c3%a0-jour)Étapes pour la mise à jour

### [](#t%c3%a9l%c3%a9charger-les-images-mises-%c3%a0-jour)Télécharger les images mises à jour

Utilisez la commande suivante pour télécharger les dernières versions des images spécifiées dans le fichier `docker-compose.yml` :

```
docker compose pull
```

### [](#red%c3%a9marrer-les-conteneurs-avec-les-nouvelles-images)Redémarrer les conteneurs avec les nouvelles images

Redémarrez les conteneurs pour appliquer les mises à jour. Utilisez l'option `-d` pour détacher le processus et `--remove-orphans` pour supprimer les conteneurs qui ne sont plus définis dans le fichier `docker-compose.yml` :

```
docker compose up -d --remove-orphans
```

### [](#nettoyer-les-images-obsol%c3%a8tes)Nettoyer les images obsolètes

Pour libérer de l'espace disque, supprimez les images qui ne sont plus utilisées par aucun conteneur actif :

```
docker image prune
```