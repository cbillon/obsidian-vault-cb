---
tags:
  - docker
  - lamp
  - php
---

## Ma stack lamp 
[site](https://blog.kulakowski.fr/post/docker-pour-ma-stack-lamp)


Pour illustrer le tout rien ne vaut un schéma (on commence par la version simplifiée) :
{{ :docker:architecture_docker_light.webp |}}

Pour ce qui est de docker, je suis parti de la version officiellement disponible dans CentOS 7 avec comme système de stockage aufs.


===== 2 services (HTTPd & PHP-FPM) donc 2 containers =====

Pour faire un serveur HTTP/PHP, certains font un gros container contenant les 2 services. Personnellement, je préfère respecter la bonne pratique qui consiste à avoir 1 container pour 1 service. Comme d’habitude, j’ai fait le choix de ne pas mettre WordPress dans le container PHP + le container HTTPd. C’est une solution de facilité mais qui s’explique :

Pas besoin de mettre à jour 2 containers pour mettre à jour WordPress.
Possibilité de mettre à jour WordPress depuis Deployer.
Plus simple.
Du coup j’ai un volume partagé entre les 2 containers et que je peux trifouiller depuis mon serveur.

===== Serveur HTTP =====

Pour le serveur HTTP, je reste fidèle à Apache. 

En ce qui concerne le container Docker, je dérive de la dernière version officielle disponible sur le store, mais en version Alpine, c’est plus léger et plus joueur que la version Debian.

À ceci j’embarque quelques optimisations telles que :

  * [[https://github.com/llaumgui/docker-images-httpd/tree/main/2.4/conf.d/deflate.conf|Configuration du module deflate]].
  * [[https://github.com/llaumgui/docker-images-httpd/tree/main/2.4/conf.d/etags.conf|Paramétrage des ETags]].
  * [[https://github.com/llaumgui/docker-images-httpd/tree/main/2.4/conf.d/expires.conf|Mise en place des entêtes expires]].
  * [[https://github.com/llaumgui/docker-images-httpd/tree/main/2.4/conf.d/security.conf|Quelques entêtes et configurations minimales pour la sécurité]].
  *[[ https://github.com/llaumgui/docker-images-httpd/tree/main/2.4/conf.d/vhost.conf|Un répertoire pour les virtual host]].
  *[[ https://github.com/llaumgui/docker-images-httpd/tree/main/2.4/conf.d/ssl-intermediate.conf|Et le support du SSL avec une configuration basée sur l’excellent outil de Mozilla]].

[[https://github.com/llaumgui/docker-images-httpd/tree/main/2.4|Sources de mon images Apache HTTP disponibles sur GitHub]].

## Serveur PHP-FPM

à encore, je pars de la version officielle du store mais sous Alpine. Mon image a de particulier que lorsqu’on la build via docker-compose, on peut lui passer des paramètres. Ça me permet d’avoir la même image en dev et en prod mais avec XDebug seulement sur la prod.

  php:
    container_name: php
    image: llaumgui/php:7.2-fpm
    build:
      context: build/php-fpm/7.2/
      args:
        DOCKER_PHP_ENABLE_APCU: 'on'
        DOCKER_PHP_ENABLE_COMPOSER: 'on'
        DOCKER_PHP_ENABLE_LDAP: 'off'
        DOCKER_PHP_ENABLE_MEMCACHED: 'off'
        DOCKER_PHP_ENABLE_MONGODB: 'off'
        DOCKER_PHP_ENABLE_MYSQL: 'on'
        DOCKER_PHP_ENABLE_POSTGRESQL: 'off'
        DOCKER_PHP_ENABLE_REDIS: 'on'
        DOCKER_PHP_ENABLE_SYMFONY: 'off'
        DOCKER_PHP_ENABLE_XDEBUG: 'off'
	DOCKER_USER_UID: 1000
	DOCKER_USER_GID:1000
  [..]
  
À noter que lorsque je déploie mon application, j’ai besoin de lancer des tâches PHP. Or, je n’ai pas PHP sur mon serveur mais dans mon container. Pour ceci j’utilise un simple script shell qui va vérifier que je suis bien dans le bon répertoire et lancer php-cli via le container. Pour éviter des problèmes de droits, le php-cli s’exécute sous un ID identique à mon utilisateur de déploiement (passé en paramètre dans mon docker-compose).

  #!/bin/bash
  PHP_PATH=/var/www
  [[ ! "$PWD" =~ ${PHP_PATH} ]] && echo "Les commandes php ne sont utilisables que sous ${PHP_PATH}" && exit 1 
  CONTAINER_NAME="php"
  COMMAND="php $@"
  PWD=$(pwd)
  docker exec -u user -i ${CONTAINER_NAME} /bin/sh -c "cd ${PWD} && ${COMMAND}"
