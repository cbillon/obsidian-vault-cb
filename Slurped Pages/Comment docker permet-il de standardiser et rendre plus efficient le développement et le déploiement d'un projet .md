---
link: https://linuxembedded.fr/2023/11/comment-docker-permet-il-de-standardiser-et-rendre-plus-efficient-le-developpement-et-le
byline: Ben
excerpt: Dans cet article nous allons voir ce qu'est Docker et comment l'utiliser dans le contexte du développement d'applications. Qu'est-ce que Docker ?Docker est une plate-forme logicielle libre et open source qui permet de lancer des applications dans un environnement isolé, appelé conteneur.
slurped: 2024-11-22T13:25
title: Comment docker permet-il de standardiser et rendre plus efficient le développement et le déploiement d'un projet ?
tags:
  - docker
---

Dans cet article nous allons voir ce qu'est Docker et comment l'utiliser dans le contexte du développement d'applications.

## Qu'est-ce que Docker ?

Docker est une plate-forme logicielle **libre** et **open source** qui permet de lancer des applications dans un environnement isolé, appelé conteneur.

## Historique

Le développement de Docker a démarré en 2010 par le Franco-américain **Solomon Hykes** avec les contributions de plusieurs autres développeurs comme Andrea Luzzardi et Francois-Xavier Bourlet. Il s'agissait d'un projet interne de la société dotCloud pour du PaaS (Platform as a service), les sources ont été ouvertes en mars 2013 lors de la [PyCon](https://en.wikipedia.org/wiki/Python_Conference).

Il est soutenu et/ou supporté par de nombreuses entreprises comme [Red Hat](https://en.wikipedia.org/wiki/Red_Hat) depuis 2013, [AWS](https://en.wikipedia.org/wiki/Amazon_Web_Services), [IBM](https://en.wikipedia.org/wiki/IBM) et [Microsoft](https://en.wikipedia.org/wiki/Microsoft) depuis 2014, ...  

## Pourquoi utiliser Docker ?

Docker permet d'isoler une application de son système hôte, il est donc possible de lancer une application sur n'importe quelle distribution Linux, sans avoir à se soucier des dépendances, des versions des bibliothèques, ...

De plus, il permet de gérer facilement de multiple versions d'une même application via les tags sur les images, mais également de gérer les ressources allouées à un conteneur (CPU, RAM, périphériques, réseau, volumes, ...).

Docker est très populaire et a bientôt 10 ans, il est assez facile de trouver de la documentation, des images, des tutoriels, ...

Il permet la création de dépôt d'images privées, ce qui permet de garder le contrôle sur les images utilisées (bien utile en entreprise).

Docker peut être comparé à [chroot](https://en.wikipedia.org/wiki/Chroot), mais en plus puissant sur certains points, les deux permettent de lancer des applications dans un environnement isolé, mais docker peut en plus, créer des images, permet de les partager, gérer les versions, des réseaux, la gestion de ressources matérielles etc.

[Il y a peu d'impact sur les performances](https://stackoverflow.com/a/26149994) par rapport à une application lancée directement sur le système hôte Linux, pour un hôte Windows, il y a un impact sur les performances plus important, car il utilise la virtualisation via WSL pour faire tourner un noyau Linux en plus de docker.

Docker est bien plus léger qu'une machine virtuelle, il n'y a pas de système d'exploitation complet pour faire tourner l'application, il utilise le noyau du système l'hôte, mais un conteneur docker est moins "sécurisé" qu'une machine virtuelle, car l'isolation n'est que logiciel via les [cgroups](https://en.wikipedia.org/wiki/Cgroups) et [namespaces](https://en.wikipedia.org/wiki/Linux_namespaces).

![Comparaison entre une solution de virtualisation et de dockerisation](https://linuxembedded.fr/sites/default/files/inline-images/dockervsvm.png)

Comparaison entre une solution de virtualisation et de dockerisation

## Installation

Il est disponible sur la plupart des distributions Linux, ainsi que sur Windows (via [WSL](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux)) et macOS, il supporte plusieurs architectures: x86, x86-64,, ARMv7, ARMv8, Risc-V et bien d'autres.

### Linux

Pour les systèmes Linux, il peut être installé via le gestionnaire de paquets de votre distribution, après avoir ajouté le dépôt maintenu par docker.io (procédure pour [Debian](https://docs.docker.com/engine/install/debian/), [Ubuntu](https://docs.docker.com/engine/install/ubuntu/), ...).

Il est également possible de compiler Docker à partir des [sources](https://github.com/docker/docker).

### Windows

Pour Windows, il faut activer WSL (Windows Subsystem for Linux), puis installer Docker via le [site officiel](https://docs.docker.com/docker-for-windows/install/).

**Le support de Docker sous Windows ARM n'est pas encore disponible (24.0.7):** [**Support Docker Desktop (WSL2) on Windows on ARM**](https://github.com/docker/roadmap/issues/91)

## Utilisation

### Images

Les images sont référencées via une URL avec chemin finissant par un nom et un tag, par exemple **docker.io/debian:bookworm** réfère à l'image **debian** avec le tag **bookworm** dans le dépôt **docker.io**.

Le _**docker.io/**_ est optionnel et c'est la valeur par défaut de docker vers le [dépôt officiel de docker](https://hub.docker.com/), cela peut être changé pour un dépôt privé par exemple. Si il n'y a pas de tag spécifié, c'est le tag _latest_ qui est utilisé par défaut.

Le [dépôt officiel de docker](https://hub.docker.com/) est ouvert à toute personne ou entreprise possédant un compte, donc ne téléchargez que des **images** **officielles** (debian, alpine, ubuntu ect...) ou de **confiances**.

Les tags sont des versions de l'image, pour **debian,** il y a **bookworm, bookworm-slim, bullseye, bullseye-slim, buster** ect...

Les images sont composées de couches, chaque couche sauvegarde les modifications apportées à l'image, dans le même principe que des commits git, cela permet de réduire la taille des images dans le cache de docker et construire celles-ci plus rapidement lorsqu'il y a des nouvelles modifications.

### Les commandes

Nous allons voir certaines commandes de Docker, vous trouverez plus de détails dans la [documentation](https://docs.docker.com/engine/reference/commandline/docker/).

#### Pull

La commande **docker pull** **<image>** télécharge la dernière version de l'image et ses dépendances, puis la stocke dans le cache de docker, par exemple **docker pull debian:bookworm** pour télécharger l'image de **debian** avec le tag **bookworm** (https://hub.docker.com/_/debian), il est **important** de régulièrement mettre à jour ses images pour des raisons de sécurité.

Vous pouvez lister les images téléchargées ou construites dans le cache de docker avec la commande **docker images**

Sur certains dépôts ou images privées, il peut être nécessaire de se connecter via **cat <Fichier txt avec le MDP> | docker login <dépôt docker> --username <utilisateur> --password-stdin** pour avoir accès à celles-ci, il est également fortement recommandé d'activer la [double authentification](https://en.wikipedia.org/wiki/Multi-factor_authentication) si elle est disponible et de se connecter via un [access token](https://hub.docker.com/settings/security).

#### Push

La commande **docker push** **<image>** permet de pousser une image dans un dépôt, cela permet de partager une image avec d'autres utilisateurs. Si l'image n'existe pas dans le dépôt elle est créée, si elle existe elle est mise à jour. Par exemple **docker push monprojet/monimage:1.0.5** pour pousser l'image **monprojet/monimage** avec le tag **1.0.5** dans le dépôt **docker.io**. L'option **--all-tags**, vous permez de pousser tous les tags d'une image.

#### Run

La commande **docker run** permet de lancer une image dans un conteneur, par exemple **docker run debian:bookworm** pour lancer l'image de **debian** avec le tag **bookworm**.

Vous pouvez y ajouter des options, par exemple **docker run -it --rm debian:bookworm** pour lancer l'image de **debian** avec le tag **bookworm** avec accès au terminal, puis supprimer le conteneur à la fin de l'exécution.

Lorsqu'un conteneur est supprimé, **toutes les modifications sont perdues**, il faut utiliser des volumes pour sauvegarder les données entre les exécutions si besoin.

|Option|Min|Défaut|Description|
|---|---|---|---|
|--interactive|-i|Désactivée|Mode interactif|
|--tty|-t|Désactivée|Mode tty|
|--rm|-|Désactivée|Supprime le conteneur à la fin de l'exécution|
|--name|-|sha256|Donne un nom au conteneur en cours d'exécution|
|--env|-e|-|Spécifie une variable d'environnement|
|--read-only|-|Désactivée|Montre le système de fichier en lecture seule|
|--mount|-|-|Monter un répertoire local dans le conteneur|
|--volume|-v|-|Monter un volume de docker|
|--privileged|-|Désactivée|Donne tous les privilèges au conteneur (non recommandé)|
|--security-opt|-|-|Spécifie des options de sécurité (**no-new-privileges** etc...)|

#### Build

La commande **docker build** permet de créer une image à partir d'un fichier appelé **Dockerfile**, par exemple **docker build .** pour créer/construire une image à partir du fichier **Dockerfile** dans le répertoire courant.

Vous pouvez y ajouter des tags lors de la construction, par exemple **docker build -t monimage:latest -t monimage:1.0 .** pour créer une image avec le tag **latest** et **1.0** , cela ne fonctionnera pas si il n'y a pas de **Dockerfile** dans le répertoire courant, nous verrons cela dans la partie application.

|Option|Min|Défaut|Description|
|---|---|---|---|
|--file|-f|Dockerfile|Spécifie le fichier Dockerfile|
|--tag|-t|-|Spécifie un tag de l'image|
|--build-arg|-|-|Spécifie une variable pour le build|
|--platform|-|-|Spécifie l'architecture de l'image|

### Nettoyage

Si nous voulons par exemple supprimer les conteneurs et les couches qui ne sont plus utilisées et les images qui n'ont plus de tag, il faut utiliser la commande **docker system prune**.

## Dockerfile

Le fichier **Dockerfile** est un fichier texte qui contient une liste d'instructions qui permettent de créer une image et lancer une application dans un conteneur.

Il est possible de créer des images à partir d'autres images, par exemple **FROM debian:bookworm** pour créer une image à partir de l'image **debian** avec le tag **bookworm**.

### Application

Nous prendrons l'exemple d'une application C++ simple qui affiche **Hello World!**.

**main.cpp:**

```
#include <iostream>

int main() {
 std::cout << "Hello World! " << std::endl;
 return 0;
}
```

**Dockerfile**:

```
# Utilise l'image alpine avec le tag 3.18
FROM alpine:3.18

# Lance les commandes suivantes lors de la construction de l'image, ici les dependances pour compiler un programme
RUN apk add --no-cache g++ make

# Définit le répertoire courant, ici /build
WORKDIR /build

# Copie le fichier main.cpp du répertoire courant local vers le répertoire /app de l'image
COPY main.cpp .

# Creation d un programme simple et construction de celui-ci
RUN g++ main.cpp -O3 -o main

# Définit le point d'entrée, c'est à dire la commande qui sera exécutée au lancement du conteneur
# CMD est écrasée si une commande est spécifiée lors du lancement du conteneur (docker run)
# Alors que ENTRYPOINT est toujours exécuté lors du lancement du conteneur (hors options --entrypoint)
CMD ["./main"]
```

Chaque instruction (RUN, COPY etc...) va créer une nouvelle couche dans l'image, il est donc **important** de les ordonner et de les combiner pour optimiser la taille de l'image.

Il est recommandé de mettre une application par conteneur, cela permet de les isoler et de les mettre à jour plus facilement de manière indépendante.

Pour construire l'image, il faut utiliser la commande **docker build <chemin>** vu précédemment, pour créer une image à partir du fichier **Dockerfile** dans le répertoire courant:

Nous allons maintenant construire **monimage** avec le **Dockerfile** qui est dans le répertoire courant: 

```
docker build -t monimage:latest -t monimage:1.0 .
```

Maintenant que nous avons créé une image, nous pouvons démarré l'image, elle doit afficher **Hello World!**.

```
docker run --rm monimage:1.0
```

Avec la commande **docker image ls**, vous pouvez lister les images locales, l'image **monimage** doit apparaître au moins 2 fois dans la liste avec les tags **latest** et **1.0**, elles devraient faire environ 200Mo, ce qui est très lourd pour une application qui affiche seulement **Hello World!**.

```
docker image ls
```

### Multi-stage build

Nous allons voir comment réduire la taille de l'image grâce au **multi-stage build**, avec cette méthode, nous allons créer une image temporaire pour construire l'application, puis une image finale qui contiendra seulement l'application et les dépendances nécessaires pour l'exécution (sans GCC ect...).

Voici le nouveau **Dockerfile**:

```
# On utilise l'image alpine avec le tag 3.18 pour créer une image temporaire
FROM alpine:3.18 as builder

# Lance les commandes suivantes lors de la construction de l'image, ici les dépendances pour compiler un programme
RUN apk add --no-cache g++ make

WORKDIR /build

# Pour copier des fichiers locaux vers l'image, il faut utiliser COPY <source> <destination>
# Ici on copie le fichier main.cpp du répertoire courant local vers le répertoire /build de l'image
COPY main.cpp .

# Création d un programme et construction de celui-ci
RUN g++ main.cpp -O3 -o main

# Nous réutilisons la base de l image précédente pour créer une image finale, ce processus est appelé multi-stage build
# Cela permet de reduire la taille de l'image en ne gardant que l'essentiel pour lancer l'application
FROM alpine:3.18 as final

# On ajoute les dépendances nécessaires pour lancer le programme
RUN apk add --no-cache libstdc++

# Copie le contenu du répertoire /build dans l'image build vers le répertoire /app de l'image finale
COPY --from=builder /build /app

WORKDIR /app

# Execute la commande ./main lors du lancement du conteneur
CMD ["./main"]
```

Nous allons construire l'image, mais avec le tag **2.0**:

```
docker build -t monimage:latest -t monimage:2.0 .
```

Maintenant que nous avons créé une nouvelle image avec le multi-stage build, nous pouvons lancer la lancer, elle doit afficher **Hello World!**.

```
docker run --rm monimage:2.0
```

Lorsque vous listez de nouveau les images locales, vous pouvez voir que l'image **monimage** apparaît avec le nouveau tag **2.0**, elle devrait faire environ 10Mo, ce qui est bien plus raisonnable.

```
docker image ls
```

## Multi-version

Imaginons que nous voulons créer une nouvelle version de notre programme **hello world**, nous allons modifier le fichier **main.cpp** pour afficher: **Hello World! From x86_64 architecture**.

**main.cpp**:

```
#include <iostream>
#include <string>

int main() {
    std::string arch = "";
    // On définit la variable arch en fonction de l'architecture de la machine qui est la cible
    #if defined(__x86_64__) || defined(_M_X64)
        arch = "x86_64";
    #elif defined(__i386__) || defined(_M_IX86)
        arch = "x86";
    #elif defined(__aarch64__) || defined(_M_ARM64)
        arch = "aarch64";
    #else
        arch = "unknown";
    #endif

    std::cout << "Hello World! From " << arch << " architecture" << std::endl;
    return 0;
}
```

Puis construire l'image nouvelle image avec le tag **3.0**:

```
docker build -t monimage:latest -t monimage:3.0 .
```

Maintenant que nous avons créé une image avec la nouvelle version de notre programme, nous pouvons la lancer, elle doit afficher **Hello World! From x86_64 architecture** si vous êtes sur un PC x86_64 ou **Hello World! From aarch64 architecture** si vous êtes sur une machine aarch64 (Mac M1, Raspberry Pi ect...)

```
docker run --rm monimage:3.0
```

Si nous voulons ajouter un nouveau tag **3.5** à l'image **monimage:3.0**

```
docker tag monimage:3.0 monimage:3.5
```

Nous avons maintenant plusieurs versions de l'image **monimage**:

- **monimage:1.0** Affiche seulement _hello world!_
- **monimage:2.0** Une version optimisée de la 1.0
- **monimage:3.0** La nouvelle version de notre programme qui affiche _hello world! From x86_64 architecture_
- **monimage:3.5** Un nouveau tag pour la 3.0
- **monimage:latest** La dernière version de notre image soit la 3.0/3.5

### Héritage

Nous avons vue précédemment que notre image **monimage** est basée sur **alpine:3.18**, si d'autres images utilisent **alpine:3.18** comme base, il n'y aura que les changements apportées à cette base qui est sauvegardée grâce au système de couches, cela permet de réduire la place par les images dans le cache de docker.

Nous pouvons faire de même avec une nouvelle image mais basée sur **monimage:3.0** pour ajouter une nouvelle application dans notre image.

**main2.cpp**:

```
#include <iostream>

int main() {
    std::cout << "Je suis le programme N°2" << std::endl;
    return 0;
}
```

Le nouveau **DockerfileApp2**:

```
FROM alpine:3.18 as builder

RUN apk add --no-cache g++ make

WORKDIR /build

COPY main2.cpp .

RUN g++ main2.cpp -O3 -o main2

# On utilise notre image monimage:3.0 comme base pour l'image finale
# Plus besoin d installer les dépendances, elles le sont déja via monimage:3.0
FROM monimage:3.0 as final

COPY --from=builder /build /app

WORKDIR /app

CMD ["./main2"]
```

Puis construire l'image nouvelle image avec le tag **4.0** et **--file DockerfileApp2** pour un nouveau Dockerfile

```
docker build -t monimage:latest -t monimage:4.0 --file DockerfileApp2 .
```

Nous pouvons lancer la nouvelle image, le résultat devrait etre **Je suis le programme N°2**

```
docker run --rm monimage:4.0
```

## Améliorations

Il y a de nombreuses améliorations possibles à appliquer à notre **Dockerfile**, nous allons en voir quelques unes.

Pour des raisons de sécurité, il est recommandé de ne pas utiliser l'utilisateur **root** dans le conteneur, il faut donc créer un utilisateur et un groupe dans le conteneur dans l'image finale, c'est lui qui exécutera l'application.

```
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
# Changement d'utilisateur
USER appuser
```

Nous pouvons également utiliser des arguments pour construire l'image avec l'option **--build-arg**, cela permet de spécifier des variables d'environnement lors de la construction de l'image, si par exemple nous voulons construire l'image sur plusieurs versions de la distribution.

```
ARG IMG_VERSION=3.18
FROM alpine:${IMG_VERSION} as builder
```

Pour construire l'image alpine avec la version **3.17:**

```
docker build --build-arg IMG_VERSION=3.17 -t monimage:latest -t monimage:2.0 .
```

Nous pourrions également avoir besoin de spécifier des variables d'environnement lors de la construction de l'image, par exemple une branche git, un numéro de version, etc. Elles resteront dans l'image et seront disponibles lors de l'exécution du conteneur.

```
ARG GIT_BRANCH=master
ENV GIT_BRANCH=${GIT_BRANCH}
```

## Allocation des ressources et des périphériques

Avec docker, vous pouvez spécifier les ressources allouées à un conteneur, cela permet de **limiter** l'impact d'un conteneur sur le système hôte lors de **son exécution**.

### CPU

Pour définir le nombre de coeurs alloués à un conteneur vous pouvez utiliser l'option **--cpus**, par exemple **docker run --cpus 2.0 debian:bookworm** le conteneur pourra utiliser jusqu'à 2 coeurs CPU.

Vous pouvez définir certains coeurs qui seront alloués à un conteneur avec l'option **--cpuset-cpus**, par exemple **docker run --cpuset-cpus 0,1 debian:bookworm**, le conteneur pourra seulement utiliser les coeurs 0 et 1 du CPU. Cela peut être utile sur les CPU avec une architecture asymétrique (ARM [big.LITTLE/DynamIQ](https://en.wikipedia.org/wiki/ARM_big.LITTLE), par exemple).

### Mémoire

Vous pouvez limiter la mémoire allouée à un conteneur avec l'option **--memory**, par exemple **docker run --memory 1g debian:bookworm**, le conteneur pourra utiliser jusqu'à 1Go de mémoire RAM.

### Périphériques

Pour donner accès d'un périphérique à un conteneur, il faut utiliser l'option **--device**, par exemple **docker run --device /dev/ttyUSB0 debian:bookworm** pour donner accès au device **/dev/ttyUSB0** au conteneur.

Cela fonctionne également avec les GPU via l'option **--gpus**, par exemple **docker run --gpus all debian:bookworm** pour donner accès à tous les GPU au conteneur, cela permet d'utiliser des applications qui utilisent les GPU dans un conteneur (Tensorflow, FFMPEG, etc.), plus d'infos: [GPU Nvidia](https://docs.docker.com/config/containers/resource_constraints/#gpu).

### Réseaux

Docker peut créer des réseaux virtuels, cela permet d'isoler les conteneurs et de les faire communiquer entre eux.

Pour créer un réseau, il faut utiliser la commande **docker network create <nom>,** par exemple **docker network create monreseau** pour créer un réseau appelé **monreseau**.

Pour utiliser un réseau, il faut utiliser l'option **--network <réseau>**, par exemple **docker run --network monreseau debian:bookworm** pour utiliser le réseau **monreseau** dans le conteneur.

Dans le même registre, vous pouvez ouvrir un port d'un conteneur avec **-p 80:8080** pour ouvrir et bind le port **80** dans le conteneur au port **8080** en externe.

### Les volumes

Les conteneurs sont éphémères, toutes les modifications sont perdues à la fin de l'exécution, nous pourrions par exemple avoir besoin de sauvegarder les logs, ou avoir besoin de partager des données entre plusieurs conteneurs, par exemple une base de données. Les volumes permettent de résoudre ces deux problèmes, ils offrent **un stockage persistant** et peuvent être **partagés** entre les conteneurs.

Pour créer un volume, il faut utiliser la commande **docker volume create <nom>**, par exemple **docker volume create monvolume** pour créer un volume appelé **monvolume**.

Pour utiliser un volume, il faut utiliser l'option **--volume <volume>**, par exemple **docker run --volume monvolume:/data debian:bookworm** pour utiliser le volume **monvolume** dans le conteneur et le monter dans le répertoire /data.

Il est également possible de monter le volume en read-only avec l'option **:ro** par exemple **--volume monvolume:/data:ro**

Vous pouvez lister les volumes avec la commande **docker volume ls**.

### Bind mount

Une "alternative" aux volumes sont les bind mount, cela permet de monter un répertoire de l'hôte dans le conteneur, mais ils ont comme désavantage de ne pas être gérés par docker, donc il faut faire attention aux droits, au sytème de fichier,... Il est préférable d'utiliser les volumes.

Il est possible de monter un répertoire de l'hôte dans le conteneur avec l'option **--mount**, par exemple **docker run --mount type=bind,source="$(pwd)",target=/work debian:bookworm** pour monter le répertoire courant dans le répertoire **/work** du conteneur.

Dans le même registre, vous pouvez monter un répertoire en RAM via tmpfs: **--mount type=tmpfs,destination=/tmpfs,mode=1777,size=65536k** pour limiter la taille du tmpfs à 64Mo avec les droits 1777.

## Multi-architectures

Comme nous l'avons vu précédemment, Docker est disponible sur plusieurs architectures. Il est donc possible de créer des images Docker pour ces architectures, encore mieux, il est possible via [QEMU](https://www.qemu.org/) d'exécuter des images d'architectures différentes de celle de l'hôte.

Pour configurer docker pour utiliser QEMU, il faut executer les commandes suivantes:

```
# Activer le mode expérimental de docker
export DOCKER_CLI_EXPERIMENTAL=enabled
# Lancer le conteneur qemu 
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
# Créer un builder qemu pour docker du nom de qemu_builder
docker buildx create --name mybuilder --bootstrap --use
# Afficher les architectures supportées
docker buildx inspect --bootstrap
docker buildx ls
```

### Création d'images multi-architectures

Pour créer une image multi-architectures, il faut utiliser la commande **docker buildx build --platform <arch> <chemin>**, par exemple docker **buildx build --platform linux/arm64 .** pour créer une image à partir du fichier **Dockerfile** dans le répertoire courant pour l'architecture **arm64**.

```
docker buildx build --platform linux/arm64 . --tag monimage:arm64-latest --load
```

Vous devez utiliser `--load` pour charger l'image dans le cache local de docker, sinon l'image reste dans le cache de buildx.

Il est possible de spécifier plusieurs architectures, par exemple **--platform linux/arm64,linux/amd64**, il faut ajouter l'option **--push** pour pousser l'image dans un dépôt, car la version actuelle de docker (**24.0.6**) ne supporte pas les images multi-architectures **locales avec le même tag** lors de sa création: [docker roadmap issues 371](https://github.com/docker/roadmap/issues/371)

### Exécution d'images multi-architectures

Pour éxecuter une image docker d'une architecture différente de celle de l'hôte, il faut utiliser la commande **docker run --platform <arch> <image>**, par exemple **docker run --rm -it --platform linux/arm64 debian:bookworm** pour lancer l'image de **debian**:**bookworm** construite pour l'architecture **arm64**.

```
docker run --rm -it --platform linux/arm64 debian:bookworm
```

Une fois l'image lancée, il est possible de vérifier l'architecture de l'image avec la commande **uname -m**, le résultat doit être **aarch64**.

Un des cas d'utilisation est de pouvoir tester une application sur une cible différente de celle de l'hôte, par exemple tester une application sur une architecture **aarch64** sur un hôte **x86_64**.

Nous pouvons lancer notre application **Hello World!** sur une architecture **aarch64** sur un hôte **x86_64**.

```
docker run --rm -it --platform linux/arm64 monimage:arm64-latest
```

### Dockerfile multi-architectures

Vous pouvez créer des images pour d'autres architectures dans le Dockerfile, il faut utiliser l'option **--platform** dans le Dockerfile, par exemple:

```
FROM --platform=linux/arm64 alpine:3.18
```

## Docker compose

Une autre force de Docker, c'est la possibilité de gérer de multiple services ou applications facilement via [docker compose](https://docs.docker.com/compose/), il permet de créer et de gérer images, dépendances, réseaux, etc. Cela permet de simuler une stack technique/une infrastructure, par exemple un serveur web, une BDD et un proxy chacun dans un conteneur distinct.

Les commandes **docker-compose** et **docker compose** sont identiques dans les dernières versions de docker, mais la première est dépréciée.

Sur les versions plus anciennes (1.x) de docker, **docker-compose** était une application écrite en python alors que **docker compose** est un plugin de docker écrit en go.

### Les commandes

Nous allons voir les commandes de base de docker compose, vous trouverez plus d'informations sur les commandes dans la [documentation](https://docs.docker.com/compose/reference/).

#### Build

Pour construire les images des conteneurs, il faut utiliser la commande **docker compose build**

Vous pouvez spécifier plusieurs fichiers **docker-compose.yml** avec l'option **-f**, par exemple **docker compose -f docker-compose-backend.yml -f docker-compose-frontend.yml build** pour construire les images des conteneurs à partir des fichiers **docker-compose-backend.yml** et **docker-compose-frontend.yml ,** cela s'applique également aux autres commandes (**up**, **down** etc...).

|Option|Min|Défaut|Description|
|---|---|---|---|
|--file|-f|docker-compose.yml|Spécifie le fichier docker-compose.yml|
|--build-arg|-|-|Spécifie une variable pour le build|

#### Up

Pour lancer les conteneurs, il faut utiliser la commande **docker compose up**

|Option|Min|Défaut|Description|
|---|---|---|---|
|--build|-|Désactivée|Construit les images des conteneurs avant de les lancer|
|--detach|-d|Désactivée|Lance les conteneurs en arrière plan|
|--force-recreate|-|Désactivée|Recrée les conteneurs même si ils existent déjà|

#### Stop

Pour arrêter les conteneurs, il faut utiliser la commande **docker compose stop**.

#### Down

Pour arrêter et supprimer les conteneurs, il faut utiliser la commande **docker compose down**

#### Exemple

Voici un exemple de fichier **docker-compose.yml** avec une application:

```
# Définit la version de docker-compose, elle est optionnelle depuis les dernières versions de docker
version: "3.9"
services:
  # Nom du service (ou conteneur)
  app:
    # On utilise monimage:3.0 pour le service app
    image: image: monimage:3.0
    restart: on-failure
    
    # Permet de lancer le conteneur avec une commande spécifique (surcharge la commande CMD du Dockerfile)
    command: ["./main"]
```

Nous allons construire les images des conteneurs:

```
docker compose build
```

Nous lançons les conteneurs:

```
docker compose up
```

Vous pouvez arrêter et supprimer les conteneurs:

```
docker compose down
```

Nous pouvons complexifier notre fichier **docker-compose.yml** avec plusieurs applications:

```
# Définit la version de docker-compose, elle est optionnelle depuis les dernières versions de docker
version: "3.9"
# Nom du service (ou conteneur)
services:
  app1:
    image: image: monimage:3.0
    restart: on-failure
    
    # Permet de lancer le conteneur avec une commande spécifique (surcharge la commande CMD du Dockerfile)
    command: ["./main"]

    # Indique que le conteneur app1 peut utiliser jusqu'à 0.7 coeurs CPU et 128Mo de mémoire vive lors de son exécution
    deploy:
      resources:
        limits:
          cpus: "0.70"
          memory: 128M

  # Une autre application
  app2:
    image: monimage:1.0
    # Permet de dire que app2 dépend de app1, app1 sera lancé avant app2
    depends_on:
      - app1
```

On rebuild les images:

```
docker compose build
```

Puis on lance les conteneurs:

```
docker compose up
```

Les conteneurs **app1** et **app2** doivent être lancés, **app1** doit être lancé avant **app2**. **app1** doit afficher **Hello World! From x86_64 architecture** et **app2** doit afficher **Hello World!**.

Cet exemple est très simple, mais il est possible de créer des applications bien plus complexes avec plusieurs conteneurs, des réseaux, des volumes, etc.

Pour des volumes partagés entre les conteneurs, il faut utiliser l'option **volumes** dans chaque service qui utilise le volume, par exemple:

```
# Le volume app sera monté dans le répertoire /app du conteneur
volumes:
  - app:/app
```

Il faut ensuite définir en dehors des services le volume **app**, par exemple:

```
volumes:
  app:
```

Cela s'applique également aux réseaux, il faut utiliser l'option **networks** à la place de **volumes**, on peut également ajouter le port pour l'accès depuis l'extérieur à un service dans conteneur, par exemple un serveur web avec le port **80** qui est accessible depuis l'extérieur avec le port **8080** en [TCP](https://en.wikipedia.org/wiki/Transmission_Control_Protocol) et [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol): 

```
ports:
  - "80:8080"
```

Si nous voulons par exemple limiter au TCP, il faut ajouter **/tcp** après le port de sortie.

## Les bonnes pratiques avec docker

Nous allons voir cette partie quelques bonnes pratiques lorsque vous utiliserez docker, certains de ces points ont déjà été abordés auparavant dans l'article:

- Ne **jamais** mettre les MDP/Identifiants ou des tokens dans les images docker
- Créer et utiliser un utilisateur avec le **minimum de droits** (non-root) dans les images
- **Supprimer** toutes les libs/bins **non-nécessaires** (Via multi-stage build ou flatten une image, ...)
- Externaliser si possible les fichiers de configurations (via --build-arg, volumes, ...)
- Utiliser des images docker **officielles** ou de **confiances**
- Mettre à jour **régulièrement** les images docker
- Mettre en **read-only** le système de fichiers racine et les volumes si possible (hors logs, BDD, ...)
- Utiliser des volumes plutôt que bind mount
- Éviter de construire des images via docker-compose, il ne doit que les utiliser
- Docker **ne remplace pas** une machine virtuelle et l'**isolation** n'est que **logiciel**.

## Quelques exemples d'utilisations de Docker

### Devcontainer

Devcontainer est un outil qui permet de créer des conteneurs pour le développement d'applications, dans le but de standardiser l'environnement de développement et de le rendre plus portable.

L'un des plus connus est [devcontainer de VSCode](https://code.visualstudio.com/docs/remote/containers), cela permet d'avoir un environnement de développement standardisé, avec des versions, ...

### CI/CD

Docker est très utilisé dans les pipelines de CI/CD, cela permet de standardiser l'environnement de build et de tester les applications, il est utilisé par exemple par [Gitlab](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html), [Github](https://docs.github.com/en/actions/guides/building-and-testing-nodejs).

### Applications

Il y a de nombreuses applications qui ont une version docker, cela permet de standardiser l'environnement de production ou de développement, de faciliter le déploiement, etc.

[Nginx](https://hub.docker.com/_/nginx), [Wordpress](https://hub.docker.com/_/wordpress), [nodejs](https://hub.docker.com/r/paketobuildpacks/nodejs), [Apache](https://hub.docker.com/_/httpd), [mysql](https://hub.docker.com/_/mysql), [postgres](https://hub.docker.com/_/postgres), [redis](https://hub.docker.com/_/redis), [mongoDB](https://hub.docker.com/_/mongo), ... sont des exemples d'applications qui sont disponibles sous forme d'images docker dans le [Hub de Docker](https://hub.docker.com/).

### Toolchains et compilateurs

Il existe de nombreuses toolchains dockerisées, cela permet de faciliter l'utilisation de celles-ci sans dépendances ni installation, d'avoir plusieurs versions, etc.

Voici quelques exemples:

- [Dockcross](https://github.com/dockcross/dockcross) : Toolchains pour cross-compilation sur différentes architectures
- [Builroot](https://hub.docker.com/r/buildroot/base) : Générateur de systèmes Linux embarqués
- [GCC](https://hub.docker.com/_/gcc) : Compilateur C/C++
- [Rust](https://hub.docker.com/_/rust) : Compilateur Rust
- [Go](https://hub.docker.com/_/golang) : Compilateur Go
- [Python](https://hub.docker.com/_/python) : Interpréteur Python

## Conclusion

Je vous remercie d'avoir lu cet article, j'espère qu'il vous a permis d'en apprendre plus sur Docker et que cela vous permettra d'appliquer ces connaissances dans vos projets.