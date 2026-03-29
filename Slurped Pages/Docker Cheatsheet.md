---
link: https://www.glukhov.org/fr/developer-tools/containers/docker-cheatsheet/
byline: À propos de Rost Glukhov
site: Rost Glukhov | Site personnel et blog technique
excerpt: Fiche de raccourcis Docker - liste et description des commandes Docker
  les plus fréquentes et utiles
slurped: 2026-03-22T10:57
title: Docker Cheatsheet
---

Voici une [fiche de référence Docker](https://www.glukhov.org/fr/developer-tools/containers/docker-cheatsheet/ "fiche de référence Docker")  
couvrant [les commandes les plus importantes](https://www.glukhov.org/fr/ "blog technique avec") et les concepts allant de l’installation à l’exécution de conteneurs et au nettoyage :
## Installation

### Sur Ubuntu

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
sudo apt install docker-ce
sudo systemctl start docker
```

## Commandes générales Docker

### Version et informations système

```
docker version          # Affiche les versions du CLI Docker et du démon
docker system info      # Liste les données concernant votre environnement Docker
docker help             # Affiche l'index d'aide
docker <commande> --help # Affiche les informations d'aide pour une commande spécifique
```

### Connexion et déconnexion

```
docker login            # Se connecter à Docker Hub
docker logout           # Se déconnecter de Docker Hub
```

## Gestion des images

### Lister les images

```
docker images           # Liste toutes les images
docker images -a        # Liste toutes les images, y compris les images intermédiaires
```

### Tirer des images

```
docker pull <nom-de-l-image:version> # Télécharger une image depuis Docker Hub
```

### Construire des images

```
docker build -t <nom-de-l-image> . # Construire une image à partir du Dockerfile du répertoire courant
docker build -f <chemin-du-Dockerfile> -t <nom-de-l-image> . # Construire une image à partir d'un Dockerfile spécifique
docker build --build-arg foo=bar . # Construire une image avec des arguments de construction
docker build --pull . # Tirer les versions mises à jour des images mentionnées dans les instructions FROM
docker build --quiet . # Construire une image sans afficher de sortie
```

### Étiqueter et pousser des images

```
docker tag <nom-de-l-image-locale> <nom-d-utilisateur>/<nom-de-préférence-de-l-image>
docker push <nom-d-utilisateur>/<nom-de-préférence-de-l-image>
```

### Supprimer des images

```
docker rmi <nom-de-l-image>        # Supprimer une image spécifique
docker image prune             # Supprimer les images inutilisées
docker image prune -a          # Supprimer toutes les images inutilisées
```

### Supprimer les images orphelines

```
docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
```

### Détacher une image

```
docker rmi repository/image-name:tag
```

## Gestion des conteneurs

### Exécuter des conteneurs

```
docker run -itd --name <nom-du-conteneur> <nom-de-l-image> # Exécuter un conteneur en mode détaché
docker run -it -p <port-hôte>:<port-docker> <nom-de-l-image> # Exécuter un conteneur avec un mappage de port
docker run -it --name <nom-du-conteneur> <nom-de-l-image> # Exécuter un conteneur de manière interactive
```

### Lister les conteneurs

```
docker ps                  # Liste les conteneurs en cours d'exécution
docker ps -a               # Liste tous les conteneurs
docker ps -s               # Liste les conteneurs en cours d'exécution avec l'utilisation du CPU et de la mémoire
```

### Démarrer, arrêter et redémarrer des conteneurs

```
docker start <nom-du-conteneur>   # Démarrer un conteneur arrêté
docker stop <nom-du-conteneur>    # Arrêter un conteneur en cours d'exécution
docker restart <nom-du-conteneur> # Redémarrer un conteneur
```

### Supprimer des conteneurs

```
docker rm <nom-du-conteneur>      # Supprimer un conteneur arrêté
docker rm -f <nom-du-conteneur>   # Supprimer fortement un conteneur
docker stop $(docker ps -a -q) && docker rm $(docker ps -a -q) # Supprimer tous les conteneurs
```

### supprimer tous les conteneurs arrêtés

### Se connecter aux conteneurs

```
docker attach <nom-du-conteneur>  # Se connecter à un conteneur en cours d'exécution
docker exec -it <nom-du-conteneur> bash # Exécuter une commande dans un conteneur en cours d'exécution de manière interactive
```

### Inspecter les conteneurs

```
docker inspect <nom-du-conteneur> # Inspecter un conteneur
docker logs <nom-du-conteneur>    # Afficher les journaux d'un conteneur
docker port <nom-du-conteneur>    # Afficher les mappages de port d'un conteneur
```

## Gestion des réseaux

```
docker network ls            # Liste tous les réseaux
docker network create <nom-du-réseau> # Créer un nouveau réseau
docker network rm <nom-du-réseau>    # Supprimer un réseau
```

## Gestion des volumes

```
docker volume ls             # Liste tous les volumes
docker volume create <nom-du-volume> # Créer un nouveau volume
docker volume rm <nom-du-volume>    # Supprimer un volume
docker run -v <chemin-hôte>:<chemin-conteneur> <nom-de-l-image> # Monter un volume
```

## Docker Compose

### Commandes de base

```
docker compose up           # Démarrer les services définis dans docker-compose.yml
docker compose up -d        # Démarrer les services en mode détaché
docker compose down         # Arrêter et supprimer les services
docker compose ps           # Liste les services en cours d'exécution
docker compose logs         # Afficher les journaux des services
docker compose start        # Démarrer les services
docker compose stop         # Arrêter les services
docker compose pause        # Mettre en pause les services
docker compose unpause      # Reprendre les services
```

## Commandes Dockerfile

### Construire une image à partir d’un Dockerfile

```
docker build -t <nom-de-l-image> <chemin-du-Dockerfile> # Construire une image à partir d'un Dockerfile
```

### Exemple de Dockerfile

```
FROM <image-de-base>
RUN <commande>
COPY <source> <destination>
EXPOSE <port>
CMD ["<commande>", "<argument>"]
```

## Sécurité et secrets

### Secrets Docker

```
docker secret create <nom-du-secret> <fichier> # Créer un secret
docker secret ls                         # Liste tous les secrets
docker secret rm <nom-du-secret>          # Supprimer un secret
```

### Sécurité Docker

- Utilisez les secrets Docker pour gérer centralisément les données sensibles et les transmettre de manière sécurisée aux conteneurs.
- Les secrets sont chiffrés en transit et au repos dans un swarm Docker.

## Nettoyage

### Supprimer les ressources inutilisées

```
docker system prune          # Supprimer les données inutilisées (images, conteneurs, réseaux, volumes)
docker image prune           # Supprimer les images inutilisées
docker container prune       # Supprimer les conteneurs inutilisés
docker network prune         # Supprimer les réseaux inutilisés
docker volume prune          # Supprimer les volumes inutilisés
```

### Supprimer les images non étiquetées

Parfois, après une grande compilation réussie, il peut y avoir une image comme celle-ci provenant de la commande `docker images` :

![beaucoup d’images non étiquetées](https://www.glukhov.org/developer-tools/containers/docker-cheatsheet/image.png "voir beaucoup d'images Docker non étiquetées")

pour supprimer ces images avec des étiquettes :

```
docker rmi $(docker images --filter “dangling=true” -q --no-trunc)
```

celui-ci est ancien et incompatible. Le paramètre dangling=true est obsolète. Utilisez

il demandera :

```
AVERTISSEMENT ! Cela supprimera :
	- tous les conteneurs arrêtés
	- tous les volumes non utilisés par au moins un conteneur
	- tous les réseaux non utilisés par au moins un conteneur
	- toutes les images orphelines
```

vous pourriez décider de répondre Oui à cet avertissement…

Cette fiche de référence couvre les commandes et les concepts essentiels nécessaires pour gérer les conteneurs, les images, les réseaux, les volumes et bien plus avec Docker. Pour une plongée plus approfondie, faites référence aux guides et tutoriels détaillés disponibles en ligne.

## Liens utiles

- [Fiche de référence Python](https://www.glukhov.org/fr/post/2024/08/python-cheat-sheet/ "Exemples de code utiles en Python")
- [Fiche de référence Conda](https://www.glukhov.org/fr/post/2024/10/conda-cheatsheet/ "Fiche de référence Conda")
- [Fiche de référence Bash](https://www.glukhov.org/fr/developer-tools/terminals-shell/bash-cheat-sheet/)
- [Fiche de référence Ollama](https://www.glukhov.org/fr/llm-hosting/ollama/ollama-cheatsheet/ "Fiche de référence Ollama")
- [Fiche de référence Kubernetes](https://www.glukhov.org/fr/post/2024/10/kubernetes-cheatsheet/ "Liste et description des commandes Kubernetes les plus fréquentes et utiles - fiche de référence Kubernetes")
- [Encodage/décodage Base64 sur Windows, Linux et Mac](https://www.glukhov.org/fr/developer-tools/cheatsheets/base64/ "Fiche de référence d'encodage/décodage Base64 sur Linux, Windows et Mac")
- [Installer Portainer sur Linux](https://www.glukhov.org/fr/developer-tools/containers/install-portainer-on-linux/ "Comment installer, se connecter et supprimer Portainer sur Linux")
- [Réinstaller Linux](https://www.glukhov.org/fr/developer-tools/terminals-shell/reinstall-linux/ "Réinstaller Linux")
- [Comment démarrer des fenêtres de terminal en tuiles sur Linux Mint et Ubuntu](https://www.glukhov.org/fr/developer-tools/terminals-shell/howto-start-terminal-windows-tiled-linux-mint-ubuntu/ "Comment démarrer des fenêtres de terminal en tuiles sur Linux Mint et Ubuntu")
- [MinIO en alternative à AWS S3. Aperçu et installation de MinIO](https://www.glukhov.org/fr/data-infrastructure/object-storage/minio-vs-aws-s3/ "MinIO en alternative à AWS S3. Aperçu et installation de MinIO")
- [Fiche de référence des paramètres de ligne de commande MinIO](https://www.glukhov.org/fr/data-infrastructure/object-storage/minio-cheatsheet/ "Fiche de référence des paramètres de ligne de commande MinIO")