---
link: https://slash-root.fr/docker-compose-mise-a-jour-manuelle-des-images/
byline: Julien LOUIS
site: slash-root.fr
date: 2024-07-01T11:15
excerpt: Aide-mémoire pour mettre à jour les images des conteneurs d'un fichier docker-compose.yml
slurped: 2026-02-24T10:20
title: "Docker-Compose : Mise à jour manuelle des images - slash-root.fr"
tags:
  - docker
  - images
---

## Pré-requis

Assurez-vous de vous placer dans le répertoire contenant le fichier `docker-compose.yml` avant de suivre les étapes ci-dessous.

## Étapes pour la mise à jour

### Télécharger les images mises à jour

Utilisez la commande suivante pour télécharger les dernières versions des images spécifiées dans le fichier `docker-compose.yml` :

```
docker compose pull
```

### Redémarrer les conteneurs avec les nouvelles images

Redémarrez les conteneurs pour appliquer les mises à jour. Utilisez l'option `-d` pour détacher le processus et `--remove-orphans` pour supprimer les conteneurs qui ne sont plus définis dans le fichier `docker-compose.yml` :

```
docker compose up -d --remove-orphans
```

### Nettoyer les images obsolètes

Pour libérer de l'espace disque, supprimez les images qui ne sont plus utilisées par aucun conteneur actif :

```
docker image prune
```

## Navigation de l’article

Les commentaires sont fermés.