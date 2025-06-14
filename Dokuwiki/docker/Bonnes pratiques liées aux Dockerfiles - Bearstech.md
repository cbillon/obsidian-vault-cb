---
link: https://bearstech.com/societe/blog/securiser-et-optimiser-notre-liste-des-bonnes-pratiques-liees-aux-dockerfiles/
byline: |-
  Joseph Priou
                    - github
site: Bearstech | Coopérative d'ingénieurs expert en hébergement et infogérance
excerpt: Cette semaine, nous renforçons votre démarche devops avec cet article présentant les bonnes pratiques d'utilisation de Docker, outils devops par excellence.
slurped: 2024-05-13T13:47:14.107Z
title: "Sécuriser et optimiser : notre liste de bonnes pratiques liées aux Dockerfiles | blog Bearstech"
tags:
  - docker
  - dockerfile
---

Cette semaine, nous renforçons votre démarche devops avec cet article présentant les bonnes pratiques d'utilisation de Docker, outils devops par excellence.

Docker participe très largement à la démocratisation de la conteneurisation, se faisant, il s'est placé comme moteur du cycle de vie des services déployés sur le web : de la conteneurisation, en passant par le développement et le déploiement. Il s'est affirmé comme l'outil indispensable pour tout DevOps, sans compter tous les outils qui gravitent autour : docker-compose, docker swarm, kubernetes, etc.

Cet article s'intéresse aux Dockerfiles qui sont au cœur de l'utilisation de Docker, il s'agit d'une recette de construction, une recette (de cuisine) qu'il faut respecter à la lettre, pour aboutir à une image Docker.

Néanmoins, il existe toute une série de comportements anormaux, de problèmes qu'il faut connaître, d'astuces que l'on peut appliquer lors de la construction d'une image docker.

## Outils existants

Avant d'entrer dans le cœur du sujet, nous vous proposons une liste des meilleurs projets relatifs à l'écosystème Docker, tous open source et accesibles via Github.

Il existe de nombreux projets ayant pour objectif de faciliter et optimiser la construction d'image Docker :

- [hadolint](https://github.com/hadolint/hadolint) est un analyseur statique de Dockerfiles, qui (selon moi) est le plus utile parmi tous les analyseurs statiques qui existent.
- [shellcheck](https://github.com/koalaman/shellcheck) est un analyseur statique de scripts shell. Hadolint utilise shellcheck, mais il est bon de savoir qu'il y a bien 2 outils.
- [container-diff](https://github.com/GoogleContainerTools/container-diff) est un outil d'analyse de contenu une fois l'image construite. Sa fonction principale est de lister les différences de paquets (apt, pip, npm...), de fichiers. Il peut aussi tout simplement les lister. Outil maintenu par Google.
- [clair](https://github.com/coreos/clair) est un outil de recherche de failles de sécurité, developpé par RedHat.
- [distroless](https://github.com/GoogleContainerTools/distroless) représente un groupe d'images Docker, qui possède le strict minimum pour des services compilés statiquement, maintenu par Google.
- [Awesome Docker list](https://github.com/veggiemonk/awesome-docker) est une liste très complète et bien maintenue de l'écosystème Docker.

## Bonnes pratiques

Il existe déjà de nombreux articles relatifs aux bonnes pratiques liées à Docker.

Je vous propose de revenir sur une liste de bonnes pratiques qu'il est bon d'expliquer parce que leurs effets ne sont pas transparents ou parce qu'ils semblent complexes. Par ailleurs, nous vous proposons deux comportements qu'il me semble préférable d'adopter et qui, selon moi, devraient devenir de bonnes pratiques généralisées.

### Minimiser la taille de l'image

Une bonne pratique connue est de minimiser la taille de l'image. La taille finale, mais également le nombre de couches que l'on ajoute à chaque directive Docker.

Il s'agit ici de renforcer autant que faire se peut la sécurité de vos services : en réduisant les couches de Dockerfiles, on diminue alors la surface d'attaque d'une éventuelle intrusion. On optimise également les temps de chargement sur le réseau, on améliore alors la vitesse de vos services.

La première étape est d'utiliser une image de base plus petite, comme une image alpine ou encore une image sans distribution ([distroless](https://github.com/GoogleContainerTools/distroless)), dans la mesure du possible.

Il est également pertinent de supprimer le cache après avoir installé différents paquets.

Par exemple :

# Sur debian
RUN apt-get update \
    apt-get install -y --no-install-recommends \
          [QUELQUES PAQUETS] \
    apt-get clean \
    rm -rf /var/lib/apt/lists/*

Une solution complémentaire consiste à utiliser différentes étapes ou images intermédiaires (multi-stages en anglais). [Multi-stages](https://docs.docker.com/develop/develop-images/multistage-build/). Ces images sont nécessaires à la construction de l'image, mais ne sont pas nécessaires à l'utilisation.

Cette pratique permet par exemple de télécharger des binaires, sans que l'image finale n'ait besoin d'installer les paquets de téléchargement sécurisés comme ca-certificates ou tout simplement curl. C'est également pratique lors de la reconstruction d'une image, on utilise alors le cache fourni par Docker pour ces étapes.

### Utilisateurs

Il est essentiel de définir un utilisateur par défaut qui n'aura pas les droits root, cela permet de restreindre les comportements indéterminées.

Voici un exemple illustrant cette idée :

RUN useradd -d /home/myappuser -m -s /bin/bash myappuser \
    &&  chown -R myappuser:myappuser /var/www

USER myappuser

### Un Dockerfile explicite

Il y a certaines directives Docker qui peuvent paraître inutiles lors de l'écriture d'un Dockerfile, comme EXPOSE, ou VOLUMES.

Toutefois, ces directives aident à la bonne compréhension lors de la lecture d'un Dockerfile. Comme souvent `explicite > implicite`.

### 'Les petits plus de Bearstech' par Joseph Priou

Je n'ai pas trouvé de ressources en ligne allant dans le sens de ces recommandations, néanmoins, il me semble qu'il s'agit là de pratiques qu'il convient de mettre en place et de généraliser lors de la conception de Dockerfiles.

#### Utiliser bash en tant que SHELL

La directive SHELL des Dockerfiles désigne la commande par défaut que Docker va utiliser sur chacune des directives RUN.

Le SHELL par défaut de Docker est ["/bin/sh", "-c"]

Néanmoins, il existe de nombreuses failles liés à ce shell. Notamment :

FROM debian

RUN curl bearstech.com.com | tee home.html

Ici, il y a 2 erreurs, l'addresse est mauvaise, et curl n'est pas installé par défaut sur l'image de base debian. Néanmoins ce Dockerfile est valide. la commande `docker build .` est valide, son code de retour est 0. Ce qui est embêtant, notamment dans un process d'intégration continue où l'erreur n'est pas visible. Cette commande est correcte car il s'agit d'une suite de commande dans un pipe. Seule la dernière commande compte.

Prenons un exemple plus concret:

FROM bearstech/debian-dev:stretch

ENV GOLANG_VERSION=1.10.3
ENV PATH=/opt/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV GOPATH=/go
ENV GOROOT=/opt/go

WORKDIR /opt
RUN curl -qL https://storage.googleapis.com/golang/go${GOLANG_VERSION}.linux-amd64.tar.gz | tar -xz \
    &&  useradd --home-dir /go --create-home --shell /bin/bash go \
    &&  chmod 755 /go

USER go
WORKDIR /go

Imaginons que pour une raison quelconque, le curl dans la directive RUN échoue, le build complet n'échouera pas, parce qu'il est dans un pipe, et que la commande complète est valide. Ceci n'est pas une erreur. Tous les shells sont construits là dessus. Néanmoins, on voudrait que dans un Dockerfile, si n'importe quelle commande échoue, la construction échoue.

On définit alors un SHELL un peu plus évolué.

SHELL ["/bin/bash", "-eux", "-o", "pipefail", "-c"]

L'argument pipefail donné à bash empêche une commande dans un pipe d'échouer. Les arguments '-eux' sont souvent utilisés dans les scripts bash pour que le script s'arrête à la première instruction erronée (-e), à la première variable non définie (-u), ainsi qu'afficher de manière explicite la commande actuellement exécutée (-x).

Dans un Dockerfile, les arguments '-ex' ne sont pas nécessaires. Il s'agit ici de respecter les bonnes pratiques d'écriture d'un script bash, dans les Dockerfiles. Ils n'ont aucune incidence sur l'image finale.

En reprenant notre exemple:

FROM bearstech/debian-dev:stretch

# On définit un nouveau SHELL avec les différents arguments.
SHELL ["/bin/bash", "-eux", "-o", "pipefail", "-c"]

ENV GOLANG_VERSION=1.10.3
ENV PATH=/opt/go/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV GOPATH=/go
ENV GOROOT=/opt/go

WORKDIR /opt
RUN curl -qL https://storage.googleapis.com/golang/go${GOLANG_VERSION}.linux-amd64.tar.gz | tar -xz \
    &&  useradd --home-dir /go --create-home --shell /bin/bash go \
    &&  chmod 755 /go

USER go
WORKDIR /go

# On remet en place le SHELL par défaut
SHELL ["/bin/sh", "-c"]

#### Notation ENTRYPOINT et CMD

Quelle est la différence, selon vous entre ces deux directives ?

CMD /usr/sbin/php-fpm

CMD ["/usr/sbin/php-fpm"]

La première commande est exécutée directement (il s'agit de l'équivalent d'un exec en shell), a contrario, la seconde commande est appelé via un shell. Il en est de même pour l'entrypoint, il faut privilégier la notation Json, avec les crochets.

Il arrive de nombreuses fois que l'entrypoint soit en réalité un script de mise en place pour préparer l'environnement avant d'exécuter la vraie commande spécifiée dans CMD. Privilégiez /bin/sh -c plutôt que exec.

#!/bin/bash
# Script lancé à l'entrypoint.

# On prépare l'environnment
# ...

# exec "$@"
/bin/bash -c "$@"

Il se trouve qu'utiliser exec ne permet pas d'avoir les logs, in fine, avec docker logs, car stdin et stdout sont fermées par défaut lors d'un exec au sein d'un script.

## Pro tips

### Hadolint

Hadolint est une référence en tant qu'analyseur statique sur les Dockerfiles. La manière la plus simple de l'utiliser est via une image Docker.

docker run --rm -i hadolint/hadolint < Dockerfile

Néanmoins, certains règles par défaut de Hadolint ne sont pas viables pour une image qui doit s'auto-maintenir.

Par exemple, on veut pouvoir faire confiance aux dépôts Debian, pour que la dernière version d'une librairie soit installée.

docker run --rm -i hadolint/hadolint hadolint \
          --ignore DL3008 \
          --ignore DL3013 \
          --ignore DL3016 \
          - < Dockerfile

Ces trois règles empêchent de fixer la version d'un paquet avec apt ([DL3008](https://github.com/hadolint/hadolint/wiki/DL3008)), pip ([DL3013](https://github.com/hadolint/hadolint/wiki/DL3013)), et npm ([DL3016](https://github.com/hadolint/hadolint/wiki/DL3016)).

### VOLUMES

La directive VOLUME reste utile, pour un cas particulier : après la directive VOLUME, les dossiers des différents chemins sont créés.

FROM alpine

RUN ls /opt

VOLUME ["/opt/web"]

RUN ls /opt

Ainsi, les 2 directives RUN n'affichent pas la même chose.