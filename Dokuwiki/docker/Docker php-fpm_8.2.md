---
tags:
  - docker
  - php-fpm
---

## Docker Php-fpm 8.2


[site](https://blog.kulakowski.fr/post/mise-a-disposition-de-mon-image-docker-pour-php-8-2)


Parmi les nouveautés :

Support de PHP 8.2.
Toujours du 100% PHP-FPM.
Toujours de l’Alpine Linux.
Comme d’habitude, un max d’extension et des paramètres pour refaire une construction avec des options en plus.
Inclue à présent l’extension sysvsem nécessaire à Nextcloud 26.

Pour l’utiliser :

  docker pull [docker pull](ghcr.io/llaumgui/php-fpm:latest)
  
Ou :

  docker pull llaumgui/php
  
[La documentation](https://github.com/llaumgui/docker-images-php-fpm)