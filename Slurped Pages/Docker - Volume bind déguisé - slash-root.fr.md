---
link: https://slash-root.fr/docker-volume-bind-deguise/
byline: Julien LOUIS
site: slash-root.fr
date: 2023-10-11T09:25
excerpt: Créer un volume docker, mais en faites ça sera un volume bind.
slurped: 2025-01-17T14:54
title: "Docker : Volume bind déguisé - slash-root.fr"
tags:
  - docker
  - volumes
---

Avec Docker, il y a principalement 2 types de volumes qui sont utilisés :

- **Les volumes "volume"** : Permet de stocker un répertoire du conteneur dans un volume docker stocké sur l'host (/var/lib/docker/volumes).
- **Les volumes "bind"** : Permet de monter un répertoire présent sur l'host dans un conteneur.

Personnellement, j'utilise la plupart du temps des volumes de type "bind" afin d'avoir de la persistence pour certaines données d'un conteneur. Par exemple, une base de donnée.

Le problème que je rencontre parfois est la gestion des utilisateurs/groupes propriétaires et des permissions sur les données stockées dans ce type de volume "bind".

Avec un volume "volume", l'avantage est que tout est géré par docker. Par contre, je ne souhaite pas forcément stocker ce volume docker dans /var/lib/docker/volumes de l'host.

Alors, nous allons voir comment utilisé un volume docker mais qui en faites sera un volume de type bind. Ce que je nomme un volume bind déguisé en volume docker 🙂

## Création du volume docker

Tout d'abord, créer un répertoire local sur l'host qui stockera les données :

```
sudo mkdir -m 775 -p /srv/docker/volumes/data
```

Puis, changer l'utilisateur et groupe propriétaire :

```
sudo chown -R $USER:docker /srv/docker/volumes
```

Enfin, créer le volume docker :

```
docker volume create --driver local \
    --opt type=none \
    --opt device=/srv/docker/volumes/data \
    --opt o=bind vol_data
```

- **--driver local** : Définit le driver docker à utiliser à local (par défaut).
- **--opt type=none** : Il est important de bien définir le type à none.
- **--opt device=** : Où seront stocké les données du volume sur l'host.
- **--opt o=bind** : C'est avec cette option que le volume docker sera en faites un volume bind.

Vérifier la création du volume :

```
$ docker volume list
DRIVER  VOLUME NAME
local   vol_data
```

Vérifier les propriétés du volume :

```
$ docker inspect vol_data
[
    {
        "CreatedAt": "2023-10-11T11:15:38+02:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/vol_data/_data",
        "Name": "vol_data",
        "Options": {
            "device": "/srv/docker/volumes/data",
            "o": "bind",
            "type": "none"
        },
        "Scope": "local"
    }
]
```

Un volume nommé `vol_data` est à présent disponible pour un conteneur.

### Utilisation du volume docker

#### Avec docker run

Pour utiliser ce volume docker avec `docker run` , comme avec un volume docker "classique", utiliser l'option `--volume vol_data:/data`. Cela montera le volume docker "vol_data" précédement créé dans le point de montage "/data" du conteneur.

#### Avec docker compose

Pour utiliser ce volume docker dans un `docker-compose.yml`, c'est comme https://slash-root.fr/docker-volume-bind-deguise/un volume classique :

```
version: '3'
services
  app:
    #...
    volumes:
      - vol_data:/data
    #...
volumes:
  vol_data:
```

#### Portainer

Si portainer est utilisé pour gérer l'hôte docker, le volume apparaitra dans la section "Volumes" de l'interface web. Si vous créez vos conteneurs, services, stacks swarm sur Portainer, vous pourrez l'utiliser sans problème.

Toutefois, il est impossible de créer ce genre de volume via l'interface web de portainer.

> Attention cependant au nom du volume que vous lui donner lors de sa création.

Par exemple, je souhaite utiliser un volume docker (qui finalement sera de type bind) dans une stack swarm qui s'appelle "webapp". Lors de la création du volume, je le nommerai "webapp_vol_data" et dans mon fichier yaml de la stack, j'appelerai ce volume uniquement "vol_data".

Si le volume utilisé dans un docker-compose n'est pas bien définit au niveau du nommage, un volume docker "standard" sera alors automatiquement créer à la place et celui-ci sera dans /var/lib/docker/volumes.

Donc bien penser à vérifier que les données soient au bon endroit sur l'host. Surtout si les données doivent être stockées sur un volume réseau !