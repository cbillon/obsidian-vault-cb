---
tags:
  - docker
  - moodle
  - Domincius
  - moc
---

##Version d Alexander Domincius
La version minimal a éte reprisepour docker_moodle_cb sur github
La version full integre xdebug et traefik
Le depot moodle scripts contient script php pour lister les versions des plugins disponibles
[moodle scripts](https://github.com/Dmfama20/moodle_scripts/tree/main)

[version full](https://github.com/Dmfama20/docker_moodle_full.git)

[version minimal](https://github.com/Dmfama20/docker_moodle_minimal)

## Moodle HQ

[Docker Moodle]( https://github.com/wyveo/nginx-php-fpm.git)

## Alexandre Essaer Paris 42ae

[Docker Moodle](https://github.com/42ae/moodle-docker.git)

Blog sur la securite les malwares
[Blog](https://arsen.co/en)

## Guillaume Kolakowski

[Blog]([https://blog.kulakowski.fr](https://blog.kulakowski.fr/))

[Php-fpm Docker Images](https://github.com/llaumgui/docker-images-php-fpm.git)

## Mark Hilton
Out of the box, multi-version, fully loaded PHP-FPM docker images, that can support all my PHP projects. I work with WordPress & Laravel. The images are no light weight. The aim is to support maximum number of features out of the box, that could be easily turn ON/OFF with environment settings.
[Docker Php fpm Images](https://github.com/markhilton/docker-php-fpm.git)

## Howard Miller
The main configuration is setup in the file docker-compose.yml. Each service is a container and the compose file gives the various configuration details for that service. The volumes directives map paths inside the containers to local paths. Note that local paths are relative to the directory with the compose file. There are no absolute paths.

PHP is slightly more complicated. The default PHP image doesn't have all the extensions we need. We therefore have a PHP.Dockerfile referenced by the compose file. This tells docker to build a new image using these instructions. As PHP sits on a very limited Debian Linux instance most of this such be fairly obvious. Note that the confiuration files (e.g php.ini) are in the local folder and copied there on the build.

The Moodle program and data files are mapped to local directories under this folder so you can access them as normal without worrying about the containers.

Network host names are the same as the service names (e.g. just 'redis')

== To set up ==

Install Docker daemon and get running
Stop any local instances of web server and mysql
Make sure you have the docker-compose command installed
Clone this repo somewhere suitable (everything else is relative to this folder)
Creat subdirectories app/moodledata app/public.
Clone/copy Moodle into app/public (not as a subdir, public itself)
Copy config.php from here to that directory - modify as required
app/moodledata should be chmod 0777
docker-compose up --build -d
If I haven't missed anything, you should be able to access/install Moodle at localhost
MySQL should be accessible by your favourite client also at localhost

[docker Moodle](https://github.com/thepurpleblob/DockerMoodle)

## Wyveo

[Nginx Php-fpm]( https://github.com/wyveo/nginx-php-fpm.git)