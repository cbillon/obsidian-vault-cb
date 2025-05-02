---
link: https://bearstech.com/societe/blog/securiser-et-optimiser-le-build-des-images-docker-pour-vos-applications/
byline: Emmanuel Mazurier
site: Bearstech | Coopérative d'ingénieurs expert en hébergement et infogérance
excerpt: Explorez des méthodes éprouvées pour sécuriser et optimiser le build de
  vos images Docker. Maîtrisez les bonnes pratiques pour builder des
  applications Docker sûres.
slurped: 2024-05-13T13:52:46.182Z
title: Sécuriser et optimiser le build des images Docker pour vos applications.
  | blog Bearstech
---

Explorez des méthodes éprouvées pour sécuriser et optimiser le build de vos images Docker. Maîtrisez les bonnes pratiques pour builder des applications Docker sûres.

Docker se distingue par sa flexibilité et ses possibilités de personnalisation. Cette liberté, si elle n'est pas gérée avec prudence, peut mener à des pratiques suboptimales, compromettant la performance et la sécurité de vos systèmes. Dans cet article, nous vous présentons notre méthodologie, rigoureuse, pour minimiser ces risques, assurer la maintenabilité et optimiser vos applications sur nos infrastructures Docker.

Dans un article précédent, on vous avait parlé des bonnes pratiques de Bearstech pour [la gestion et la mise à jour sécurisée des images Docker](https://bearstech.com/societe/blog/mettre-a-jour-les-images-docker-la-methode-de-bearstech/).

Les images Docker qui sont maintenues par Bearstech bénéficient de la même rigueur que nous apportons au reste de notre infrastructure, en particulier du point de vue de la sécurité, le tout en proposant à nos clients un moyen de s'assurer que les images qu'ils utilisent sont bien à jour.

Ces images dérivent toutes de notre image de base [bearstech/debian](https://hub.docker.com/r/bearstech/debian) et grâce au système de "layers" de Docker, l'image de base Debian est partagée par toutes les images de notre arborescence : au final le coût de stockage des images sur les serveurs reste très modeste et n'est PAS proportionnel au nombre d'images.

A partir de [nos images de base](https://hub.docker.com/u/bearstech), vous pouvez builder facilement votre image pour vos applications : Nodejs, Php, Python, Ruby ...

## Principe d'unicité de Docker

Afin de respecter le principe d'unicité de Docker : 1 conteneur = 1 service (voir [la documentation de Docker](https://docs.docker.com/config/containers/multi-service_container/) ou [cette conversation](https://stackoverflow.com/a/48242863)), nos images sont généralistes, elles ne contiennent que les librairies nécessaires à l'exécution des tâches dédiées à l'instance applicative et à son lien avec les autres services dont elle aura besoin pour fonctionner (serveur web, bdd, cache ...).

### Exemple de build d'une application utilisant l'image PHP de Bearstech

Par exemple, dans le cas de l'image Docker PHP maintenue par Bearstech, le build de l'image d'une application PHP va hériter de 3 images:

- Debian
    - php-cli : intègre les éléments de base de php (php-cli, mysql, gd ...)
        - php : intègre les éléments pour un runtime web (fpm, cache, redis, xdebug ...)

Comme Bearstech assure le suivi en permanence des mises à jour de sécurité Debian sur toutes les images que nous maintenons, ceci inclut PHP. Les versions de PHP maintenues dépendent donc de la version de Debian afin que les images contiennent bien tous les patch de sécurité à jour.

A l'heure actuelle, les versions suivantes sont maintenues sous Docker:

- Debian 10 (buster) -> php-cli:7.3 -> php:7.3
- Debian 11 (bullseye) -> php-cli:7.4 -> php:7.4
- Debian 12 (bookworm) -> php-cli:8.2 -> php:8.2

Nous maintenons également des versions intermédiaires de PHP selon le principe du "Rolling Release": la versions déployée suit les mises à jour mineures PHP au fil de l'eau, mais quand le projet PHP les abandonne, elles ne reçoivent plus de patches de sécurité :

- Debian 11 (bullseye) -> php-cli:8.1-rr -> php:8.1-rr

Pour les applications qui ont besoin d'ajouter des librairies ou un framework (Symfony, Laravel ...) via [Composer](https://getcomposer.org/), vous avez la possibilité d'utiliser notre image php-composer qui est maintenue selon la même logique :

- Debian 10 (buster) -> php-cli:7.3 -> php-composer:7.3
- Debian 11 (bullseye) -> php-cli:7.4 -> php-composer:7.4
- Debian 11 (bullseye) -> php-cli:8.1-rr -> php-composer:8.1-rr
- Debian 12 (bookworm) -> php-cli:8.2 -> php-composer:8.2

## Optimiser le build sur une CI avec Gitlab

Comme chacune des images importe les couches des images précédentes, la mise en cache au niveau du serveur d'intégration continue (CI) est efficiente, et tout ce qu'aura à faire le build de votre image sera d'ajouter l'opération qui consiste à copier le code de votre application.

Un simple Dockerfile comme suit devrait suffire:

FROM bearstech/php:8.2

ARG UID=1001

RUN useradd app --uid $UID --shell /bin/bash
COPY . /var/www/app

USER app

CMD ["/usr/sbin/php-fpm"]

De cette manière, depuis l'image de base Debian à celle de votre application, en passant par l'image de l'environnement d'application, chaque image bénéficie des mises à jour de sécurité héritées de la version parente.

A noter également, pour les applications qui ont besoin d'installer des librairies via Composer, l'utilisation de l'image [bearstech/php-composer](https://hub.docker.com/r/bearstech/php-composer) permet de traiter cette étape en amont du build de votre application dans la CI Gitlab, via une commande du type:

docker run --rm -u `id -u` -v `pwd`:/var/www/app -w /var/www/app bearstech/php-composer:8.2 bash -c "composer install"

### Layer caching

Vous pouvez aussi optimiser le temps de build de l'image de votre application en suivant les principes du ["Layer Caching"](https://docs.gitlab.com/ee/ci/docker/docker_layer_caching.html)

Dans la CI, vous allez récupérer la dernière version de votre image via un `docker pull` qui sera utilisée comme cache pour builder la nouvelle version de votre image grâce à l'argument `--cache-from` donné à la commande `docker build`:

```
docker build --cache-from $CI_REGISTRY_IMAGE:latest --tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA --tag $CI_REGISTRY_IMAGE:latest .
```

Il est donc important de labelliser correctement vos images afin de savoir quelle version a été versionnée en dernier ("latest") et de l'inclure dans un flux d'imbrication des couches ("layer"), au fur et à mesure des montées en version de votre application.

Toutes ces étapes sont reproductibles et automatisables grâce à notre [workflow devops](https://workflow.bearstech.com/). Celui-ci permet d'organiser de manière efficace le travail des développeurs en plaçant en continu notre travail d'infogérance (OPS) dans les procédés d'intégration et de déploiement de vos applications. Vous pouvez lire aussi à ce sujet nos articles qui expliquent [en quoi notre workflow vous permet d'améliorer la fiabilité de vos applications](https://bearstech.com/societe/blog/pourquoi-mettre-en-oeuvre-un-workflow-devops/), et vous assure des déploiements cohérents et reproductibles.

