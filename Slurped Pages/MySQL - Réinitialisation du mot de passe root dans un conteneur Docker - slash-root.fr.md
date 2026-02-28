---
link: https://slash-root.fr/mysql-reinitialisation-du-mot-de-passe-root-dans-un-conteneur-docker/
byline: Julien LOUIS
site: slash-root.fr
date: 2024-06-17T15:57
excerpt: Réinitialisation du mot de passe root dans mysql en environnement docker.
slurped: 2026-02-24T10:01
title: "MySQL : Réinitialisation du mot de passe root dans un conteneur Docker - slash-root.fr"
tags:
  - mysql
  - password
---

## Introduction

Dans certains cas, vous pouvez perdre ou oublier le mot de passe root de votre base de données MySQL, rendant ainsi difficile l'exécution de tâches administratives essentielles. Cela peut se produire si le mot de passe a été généré automatiquement lors de l'initialisation du conteneur et non enregistré. Ce guide explique comment réinitialiser le mot de passe root de MySQL dans un environnement Docker.

## Préparation

### 1. Créer le fichier de réinitialisation

Commencez par créer un fichier `mysql-init.sql` dans le même répertoire que votre fichier `docker-compose.yml`. Ce fichier contiendra une requête SQL pour réinitialiser le mot de passe root.

```
USE mysql;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'NEWPASSWORD';
FLUSH PRIVILEGES;
```

Remplacez `NEWPASSWORD` par le nouveau mot de passe souhaité.

### 2. Modifier `docker-compose.yml`

Pour utiliser le fichier de réinitialisation avec Docker, vous devez monter ce fichier dans le conteneur MySQL.

Ajoutez ou modifiez la section `volumes` pour inclure le montage du fichier `mysql-init.sql` :

```
services:
  mysql:
    # Autres configurations de service
    volumes:
      # Autres volumes
      - ./mysql-init.sql:/tmp/mysql-init.sql
```

## Processus de Réinitialisation

### 1. Arrêter le conteneur MySQL

Avant de réinitialiser le mot de passe, assurez-vous d'arrêter le conteneur MySQL :

```
docker compose stop mysql
```

### 2. Accéder au conteneur

Lancez le conteneur MySQL sans démarrer le service pour obtenir un shell bash :

```
docker compose run mysql bash
```

### 3. Exécuter le script de réinitialisation

Dans le conteneur, lancez MySQL avec le fichier de réinitialisation :

```
mysqld --init-file=/tmp/mysql-init.sql &
```

Attendez quelques instants que le processus se termine. Vous pouvez vérifier si le mot de passe a été réinitialisé en tentant une connexion à MySQL.

### 4. Nettoyer et relancer le service

Quittez le conteneur et supprimez le montage du fichier SQL dans `docker-compose.yml` :

- Supprimez ou commentez la ligne `- ./mysql-init.sql:/tmp/mysql-init.sql`.

Ensuite, arrêtez complètement la stack Docker :

```
docker compose down
```

Relancez la stack :

```
docker compose up -d
```

## Vérification

Pour vérifier que le nouveau mot de passe fonctionne, essayez de vous connecter à MySQL en utilisant le nouveau mot de passe :

```
docker exec -it <CONTENEUR_MYSQL> mysql -u root -p
```

Remplacez `<CONTENEUR_MYSQL>` par l'ID ou le nom du conteneur MySQL.