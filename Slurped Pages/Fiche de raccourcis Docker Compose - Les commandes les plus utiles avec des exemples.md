---
link: https://www.glukhov.org/fr/developer-tools/containers/docker-compose-cheatsheet/
byline: À propos de Rost Glukhov
site: Rost Glukhov | Site personnel et blog technique
excerpt: "La liste des commandes, structures et exemples de Docker Compose les
  plus utiles, avec des descriptions : Docker Compose Cheatsheet"
slurped: 2026-03-22T10:54
title: Fiche de raccourcis Docker Compose - Les commandes les plus utiles avec
  des exemples
---

Voici un **[fichier d’astuces Docker Compose](https://www.glukhov.org/fr/developer-tools/containers/docker-compose-cheatsheet/ "fichier d’astuces Docker Compose")** avec des exemples annotés pour vous aider à maîtriser rapidement les fichiers et les commandes Compose.
## Référence du fichier Compose : `docker-compose.yml`

**Structure de base** :

```
version: '3'       # Version du format du fichier Compose

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"   # Port hôte 8080 : Port conteneur 80
  db:
    image: postgres
    environment:            # Variables d’environnement
      POSTGRES_PASSWORD: example
    volumes:
      - db_data:/var/lib/postgresql/data

networks:          # Réseau personnalisé
  appnet:
    driver: bridge

volumes:           # Volume nommé
  db_data:
```

- **services** : Chaque conteneur dans votre application à conteneurs multiples. Dans l’exemple ci-dessus, nous avons deux services : `web` et `db`.
- **réseaux & volumes** : Définir des réseaux isolés et un stockage persistant – nous avons ici le réseau `appnet` et le volume `db_data`.

### Un seul service avec un mappage de port

```
services:
  app:
    build: .
    ports:
      - "8000:80"   # Port hôte 8000 : Port conteneur 80
```

_Expose l’application sur le port 8000 de l’hôte et construit à partir du Dockerfile du répertoire courant._

### Plusieurs services avec un volume partagé et un réseau personnalisé

```
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - type: bind
        source: ./app
        target: /app
    networks:
      - mynet
  db:
    image: postgres
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - mynet

networks:
  mynet:

volumes:
  db_data:
```

_Web et DB sur le même réseau ; DB utilise un volume nommé persistant – `db_data`._

### Utilisation du contexte de build et du chemin du Dockerfile

Vous pouvez [construire une image Docker](https://www.glukhov.org/fr/developer-tools/containers/docker-cheatsheet/ "liste et description des commandes Docker les plus fréquentes et utiles - fiche d’astuces Docker") à la demande à partir du Dockerfile spécifié dans le fichier docker-compose.yml.

```
services:
  app:
    build:
      context: .
      dockerfile: docker/MyDockerfile
```

### Partage de données entre services

```
services:
  web:
    image: nginx
    volumes:
      - shared_data:/usr/share/nginx/html
  worker:
    image: myworker
    volumes:
      - shared_data:/usr/src/app/data

volumes:
  shared_data:
```

_Les deux services accèdent au même volume (pour les fichiers statiques ou les échanges de données) – `shared_data`._

## Options avancées des fichiers Compose

- **environment** : Définir des variables d’environnement pour les conteneurs.
- **depends_on** : Contrôler l’ordre de démarrage des services.
- **deploy.replicas** : Échelonner le service en mode Swarm.

Exemple :

```
services:
  web:
    image: nginx
    deploy:
      replicas: 3
    depends_on:
      - db
```

_Démarrer 3 instances web ; contrôle uniquement l’ordre de démarrage (pas la disponibilité)._

## Commandes essentielles de Docker Compose

|Commande|Description|Utilisation exemple|
|---|---|---|
|`docker-compose up`|Créer et démarrer les conteneurs|`docker-compose up`|
|`docker-compose up -d`|Exécuter en arrière-plan|`docker-compose up -d`|
|`docker-compose exec`|Exécuter une commande dans un conteneur en cours d’exécution|`docker-compose exec web bash`|
|`docker-compose build`|Construire/reconstruire les images|`docker-compose build`|
|`docker-compose down`|Arrêter et supprimer les conteneurs, réseaux, volumes et images|`docker-compose down`|
|`docker-compose logs -f`|Afficher et suivre les journaux|`docker-compose logs -f`|
|`docker-compose ps`|Liste des conteneurs en cours d’exécution|`docker-compose ps`|
|`docker-compose run`|Exécuter des commandes ponctuelles (contournant la commande du fichier Compose)|`docker-compose run web python manage.py migrate`|
|`docker-compose stop`|Arrêter les conteneurs en cours d’exécution (peut redémarrer avec `start`)|`docker-compose stop`|
|`docker-compose restart`|Redémarrer les services|`docker-compose restart web`|
|`docker-compose pull`|Tirer les images des services|`docker-compose pull`|
|`docker-compose rm`|Supprimer les conteneurs arrêtés des services|`docker-compose rm web`|
|`docker-compose config`|Valider et afficher le fichier Compose|`docker-compose config`|
|`docker-compose up --scale web=3`|Démarrer plusieurs instances d’un service|`docker-compose up --scale web=3`|

## Schémas courants de Compose

- **Bases de données avec données persistantes**
    
    ```
    services:
      mysql:
        image: mysql
        environment:
          MYSQL_ROOT_PASSWORD: password
        volumes:
          - mysql_data:/var/lib/mysql
    
    volumes:
      mysql_data:
    ```
    
    _Les données de la base de données persistent dans le volume `mysql_data` lors des redémarrages des conteneurs._
    
- **Montage de code pour le développement**
    
    ```
    services:
      app:
        build: .
        volumes:
          - .:/app
    ```
    
    _Éditer en direct le code sur l’hôte, reflété automatiquement dans le conteneur._
    

## Drapeaux utiles

- `-d`: Mode détaché (exécuter en arrière-plan).
- `--build`: Forcer la reconstruction des images avant le démarrage.
- `--force-recreate`: Recréer les conteneurs même s’ils n’ont pas changé.
- `--remove-orphans`: Supprimer les conteneurs non définis dans le fichier Compose.

## Définir et personnaliser les services

Vous pouvez **définir et personnaliser les services, réseaux et volumes** dans Docker Compose en utilisant le fichier `docker-compose.yml`, qui centralise toutes vos configurations et besoins d’orchestration d’applications.

- **Les services** sont définis sous la clé `services`.
- Chaque service représente une configuration de conteneur, où vous pouvez définir :
    - **Image** : Sélectionner une image depuis Docker Hub ou un autre registre.
    - **Ports** : Mapper les ports du conteneur aux ports de l’hôte.
    - **Variables d’environnement** : Passer des valeurs de configuration.
    - **Volumes** : Persister des données ou partager des fichiers/dossiers avec l’hôte ou d’autres services.
    - **Réseaux** : Contrôler lesquels réseaux le service peut accéder.

**Exemple :**

```
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"   # Port hôte 8080 : Port conteneur 80
    environment:
      - NGINX_HOST=localhost
    volumes:
      - web_data:/usr/share/nginx/html
    networks:
      - frontend

  db:
    image: postgres:13
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend
```

- Ici, le service `web` utilise l’image nginx, définit une variable d’environnement, attache un volume, ouvre le port 80 comme 8080 sur l’hôte, et se connecte au réseau `frontend`. Le service `db` fait de même pour PostgreSQL.

## Personnaliser les réseaux

- **Réseaux** contrôlent lesquels services peuvent communiquer. Compose crée un réseau par défaut, mais vous pouvez définir davantage, spécifier des pilotes personnalisés, définir des options et déterminer lesquels services rejoignent lesquels réseaux pour une isolation fine.
- Définir les réseaux au niveau supérieur sous `networks`, et lister lesquels réseaux un service doit se connecter avec la clé `networks` au niveau du service.

**Exemple :**

```
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.host_binding_ipv4: "127.0.0.1"
```

- Connecter les réseaux aux services :

```
services:
  app:
    networks:
      - frontend
      - backend
  db:
    networks:
      - backend
```

- Cette configuration permet au service `app` d’accéder aux utilisateurs des réseaux `frontend` et `backend`, tandis que `db` n’est accessible qu’à l’intérieur du `backend`.

## Personnaliser les volumes

- **Volumes** sont définis sous la clé `volumes` au niveau supérieur. Les monter dans les conteneurs en utilisant la clé `volumes` sous un service.
- Les volumes peuvent être nommés, utiliser des pilotes personnalisés et être partagés entre plusieurs services pour la persistance des données et le partage.

**Exemple :**

```
volumes:
  web_data:                # Volume nommé pour le contenu web
  db_data:                 # Volume nommé pour la base de données

services:
  web:
    volumes:
      - web_data:/usr/share/nginx/html

  db:
    volumes:
      - db_data:/var/lib/postgresql/data
```

- Dans cet exemple, `web_data` est persistant et disponible à tout conteneur qui le monte. `db_data` garantit que les données de la base de données ne sont jamais perdues lors de la recréation du conteneur.
- Vous pouvez définir des montages de type bind avec des options de pilote personnalisées pour les cas avancés :

```
volumes:
  db_data:
    driver: local
    driver_opts:
      type: none
      device: /data/db_data
      o: bind
```

- Cette configuration configure un montage de type bind depuis le chemin de l’hôte `/data/db_data` vers le conteneur.

**Résumé des bonnes pratiques :**

- Utilisez le nom du service comme hôte DNS pour la communication inter-service.
- Attachez les services à plusieurs réseaux selon besoin pour contrôler l’accès.
- Utilisez des volumes nommés pour le stockage persistant et le partage de données.
- Définissez tout en YAML, permettant le contrôle de version et les scripts de déploiement faciles.

## Plusieurs fichiers compose

Pour **organiser des configurations complexes à plusieurs services** dans Docker Compose, vous pouvez utiliser **plusieurs fichiers compose et des fichiers de surcharge**, permettant des configurations modulaires, spécifiques à l’environnement et échelonnables. Voici comment cela fonctionne :

1. **Structure de fichiers de base et de surcharge**

- **Créer un fichier de base** (`compose.yaml` ou `docker-compose.yml`) contenant toutes les définitions de services communes et par défaut.
- **Ajouter des fichiers de surcharge spécifiques à l’environnement** (par exemple, `docker-compose.override.yml`, `docker-compose.dev.yml`, `docker-compose.prod.yml`).

**Structure d’exemple de fichiers :**

```
/projet
|-- docker-compose.yml           # Configuration de base
|-- docker-compose.override.yml  # Surcharge locale/dev (appliquée automatiquement)
|-- docker-compose.prod.yml      # Surcharge de production
|-- docker-compose.test.yml      # Surcharge de test (si nécessaire)
```

_La configuration de base définit les services principaux, tandis que chaque surcharge personnalise les paramètres pour un environnement ou un cas spécifique._

2. **Fonctionnement des surcharges de fichiers**

- **Fusion** : Lorsque vous exécutez `docker compose up`, Docker Compose fusionne le fichier de base avec tout fichier de surcharge dans l’ordre ; les fichiers suivants remplacent, étendent ou ajoutent des paramètres définis dans les fichiers précédents.
- **Surcharges de champs** : Si un service ou un champ est défini dans plusieurs fichiers, la valeur du dernier fichier spécifié est utilisée. Les nouveaux champs sont ajoutés.

**Exemple de fusion :**

- `docker-compose.yml` :
    
    ```
    services:
      web:
        image: myapp
        ports:
          - "8000:80"
    ```
    
- `docker-compose.override.yml` :
    
    ```
    services:
      web:
        environment:
          - DEBUG=true
    ```
    
- **Résultat** : Le service `web` utilise à la fois l’image et le port de base ainsi que la variable d’environnement surchargée `DEBUG`.

3. **Utilisation de commandes pour plusieurs fichiers**

- **Comportement par défaut** : Si présent, Docker Compose charge automatiquement `docker-compose.override.yml` avec `docker-compose.yml` lors de l’exécution de toute commande.
- **Spécifier manuellement les fichiers** : Utilisez les drapeaux `-f` pour contrôler lesquels fichiers sont fusionnés et dans quel ordre :
    
    ```
    docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
    ```
    
    - Cela ignore la surcharge par défaut et utilise les paramètres spécifiques à la production.

4. **Stratégies d’organisation pratiques**

- **Séparation par environnement** : Utilisez une surcharge par environnement : dev, test, prod, etc.
- **Microservices et équipes** : Divisez la configuration en fichiers séparés pour différents services ou équipes, et combinez-les selon besoin.
- **Commutateurs de fonctionnalité** : Des fichiers supplémentaires peuvent introduire ou supprimer des services ou des configurations pour des besoins temporaires (par exemple, un `compose.debug.yml` pour un journalisation supplémentaire).

5. **Avantages**

- **Clarté** : Garde les fichiers individuels petits et axés.
- **Échelonnabilité** : Ajoutez facilement de nouveaux services, environnements ou paramètres.
- **Maintenabilité** : Modifiez ou relisez uniquement les sections pertinentes pour un déploiement donné.

6. **Exemple : Changer d’environnement**

**Développement :**

```
docker compose -f docker-compose.yml -f docker-compose.dev.yml up
```

**Production :**

```
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

_Chaque environnement obtient uniquement la configuration nécessaire, avec tout le contenu partagé dans le fichier de base._

Organiser des configurations Compose complexes avec plusieurs fichiers – et utiliser le système de surcharge/fusion – garantit la modularité, la personnalisation spécifique à l’environnement et l’échelonnabilité pour de grandes applications Docker à plusieurs services.

## Liens utiles

- [https://docs.docker.com/compose/](https://docs.docker.com/compose/)
- [Fiche d’astuces Docker](https://www.glukhov.org/fr/developer-tools/containers/docker-cheatsheet/ "liste et description des commandes Docker les plus fréquentes et utiles - fiche d’astuces Docker")
- [Fiche d’astuces Kubernetes](https://www.glukhov.org/fr/post/2024/10/kubernetes-cheatsheet/ "liste et description des commandes Kubernetes les plus fréquentes et utiles - fiche d’astuces Kubernetes")
- [Dockeriser une application Flutter Web avec une build Dockerisée et Nginx](https://www.glukhov.org/fr/post/2025/06/dockerising-flutter-web-app/ "Dockeriser une application Flutter Web avec une build Dockerisée et Nginx")
- [Installer Portainer sur Linux](https://www.glukhov.org/fr/developer-tools/containers/install-portainer-on-linux/ "Comment installer, se connecter, utiliser et supprimer Portainer sur Linux")