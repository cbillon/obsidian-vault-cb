---
link: https://wiki.bodaine.fr/logiciels/moodle-custom-docker/
byline: Nicolas BODAINE
site: Wiki Tech — NicoBOD
excerpt: Wiki technique personnel — systèmes, réseaux, cloud, cybersécurité, IA,
  automatisation
slurped: 2026-10-14T08:02
title: Migration et sécurisation de Moodle sous Docker — Wiki Tech — NicoBOD
---

[Coolify](https://wiki.bodaine.fr/tags/#tag:coolify) [Docker](https://wiki.bodaine.fr/tags/#tag:docker) [Moodle](https://wiki.bodaine.fr/tags/#tag:moodle) [PHP](https://wiki.bodaine.fr/tags/#tag:php) [Sécurité](https://wiki.bodaine.fr/tags/#tag:s%C3%A9curit%C3%A9)

[](https://github.com/NicoBOD/wiki-tech/edit/main/docs/logiciels/moodle-custom-docker.md "Editer cette page")[](https://github.com/NicoBOD/wiki-tech/raw/main/docs/logiciels/moodle-custom-docker.md "Consulter la source de cette page")

Résumé

Ce guide documente la migration d'une instance Moodle en production (historiquement sous image Bitnami obsolète) vers une image Docker personnalisée, sécurisée et sans état (_stateless_) basée sur l'image Apache/PHP officielle. Il détaille également l'intégration de cette pile autonome dans l'interface de Coolify, la mise à niveau de la base de données vers MariaDB 12, l'installation sécurisée de plugins personnalisés via le processus de build, ainsi que les erreurs courantes à éviter.

|Propriété|Valeur|
|---|---|
|Difficulté|Intermédiaire|
|OS / Environnement|Debian 13 (Trixie) / Coolify v4|
|Dernière mise à jour|2026-06-24|

---

## Contexte[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#contexte "Permanent link")

Pendant de nombreuses années, Bitnami a fourni des images Docker prêtes à l'emploi pour Moodle. Cependant, Bitnami a cessé de publier et de maintenir ces images (figées à la version 5.0.2). Pour des raisons évidentes de **sécurité** et de **pérennité applicative**, il était impératif de migrer vers une solution alternative stable et maintenable.

La solution mise en place repose sur : 1. Une **image Docker sur-mesure** basée sur l'image officielle `php:8.2-apache`. 2. Une **sécurisation stricte** : le code PHP de l'application est figé en lecture seule dans l'image. 3. Un **orchestrateur de production** : la pile est déclarée sous forme de service Docker Compose autonome et gérée depuis l'interface web de **Coolify** pour simplifier la supervision, la sauvegarde et les redémarrages.

---

## Architecture de la Solution[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#architecture-de-la-solution "Permanent link")

[](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/
```
┌────────────────────────────────────────────────────────┐
│                   Reverse Proxy (Traefik)              │
└───────────────────────────┬────────────────────────────┘
                            │ (Port 80, Réseau: traefik-moodle)
                            ▼
┌────────────────────────────────────────────────────────┐
│             Conteneur moodle-custom-app                │
│                                                        │
│  - Code PHP en Lecture Seule (/var/www/html) -> root   │
│  - Config (/var/www/html/config.php) -> www-data       │
│  - Démon Cron en tâche de fond (toutes les minutes)    │
└───────────────────────────┬────────────────────────────┘
                            │ (Port 3306, Réseau: db)
                            ▼
┌────────────────────────────────────────────────────────┐
│              Conteneur moodle-custom-db                │
│                                                        │
│  - MariaDB 12 (Bitnami)                                │
│  - Persistance Physique (hk1pkqqots65e6occy3wfmhn_db)  │
└────────────────────────────────────────────────────────┘
```


---

## Prérequis[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#prerequis "Permanent link")

- Docker et Docker Compose installés sur le serveur hôte.
- Volumes Docker existants de l'ancien déploiement Moodle Bitnami :
- `hk1pkqqots65e6occy3wfmhn_db` (données MariaDB)
- `hk1pkqqots65e6occy3wfmhn_data` (données utilisateurs de Moodle / `moodledata`)
- Un réseau Docker externe nommé `traefik-moodle` pour le routage HTTPS.

---

## Procédure de déploiement[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#procedure-de-deploiement "Permanent link")

### Étape 1 : Construction de l'image personnalisée[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#etape-1-construction-de-limage-personnalisee "Permanent link")

Les fichiers nécessaires à la construction de l'image sont regroupés dans `/home/nicolas-bodaine/moodle-custom-build/`.

#### Le Dockerfile[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#le-dockerfile "Permanent link")

Le `Dockerfile` installe les dépendances système requises pour Moodle, configure les extensions PHP recommandées (OPcache, GD, etc.), clone le code source officiel de Moodle et fige les permissions.

```
FROM php:8.2-apache

SHELL ["/bin/bash", "-c"]
ENV DEBIAN_FRONTEND=noninteractive

# Dépendances système requises par Moodle
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpng-dev libjpeg-dev libfreetype6-dev libzip-dev \
    libicu-dev libxml2-dev libxslt1-dev libldap2-dev \
    libgmp-dev libpq-dev libtidy-dev libbz2-dev \
    libsodium-dev unzip git cron ghostscript \
    && rm -rf /var/lib/apt/lists/*

# Compilation et activation des extensions PHP
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
    && docker-php-ext-install -j$(nproc) \
        bcmath bz2 calendar exif gd gettext gmp intl ldap \
        mysqli opcache pdo_mysql pdo_pgsql soap sockets sodium tidy xsl zip

# Configuration OPcache recommandée
RUN { \
    echo 'opcache.enable=1'; \
    echo 'opcache.memory_consumption=512'; \
    echo 'opcache.max_accelerated_files=10000'; \
    echo 'opcache.revalidate_freq=60'; \
    echo 'opcache.use_cwd=1'; \
    echo 'opcache.validate_timestamps=1'; \
    echo 'opcache.save_comments=1'; \
    echo 'opcache.enable_file_override=0'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Limites de ressources PHP recommandées
RUN { \
    echo 'max_execution_time=600'; \
    echo 'memory_limit=512M'; \
    echo 'post_max_size=100M'; \
    echo 'upload_max_filesize=100M'; \
    echo 'max_input_vars=5000'; \
    } > /usr/local/etc/php/conf.d/moodle-recommended.ini

RUN a2enmod rewrite

# Clonage du code source stable de Moodle
ARG MOODLE_BRANCH=MOODLE_500_STABLE
RUN rm -rf /var/www/html/* \
    && git clone --depth 1 -b ${MOODLE_BRANCH} https://github.com/moodle/moodle.git /var/www/html

# Copie du fichier de configuration Moodle
COPY config.php /var/www/html/config.php

# Copie des plugins et thèmes personnalisés (le cas échéant)
COPY tutotech /var/www/html/local/tutotech
COPY theme/moove /var/www/html/theme/moove
COPY blocks/xp /var/www/html/blocks/xp

# Figer les permissions système
RUN mkdir -p /var/moodledata \
    && chown -R www-data:www-data /var/moodledata \
    && chmod 770 /var/moodledata \
    && chown -R root:root /var/www/html \
    && find /var/www/html -type d -exec chmod 755 {} + \
    && find /var/www/html -type f -exec chmod 644 {} + \
    && chown www-data:www-data /var/www/html/config.php \
    && chmod 440 /var/www/html/config.php

# Planification du Cron Moodle
RUN echo "* * * * * www-data /usr/local/bin/php /var/www/html/admin/cli/cron.php >/dev/null 2>&1" > /etc/cron.d/moodle-cron \
    && chmod 0644 /etc/cron.d/moodle-cron

COPY entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/entrypoint.sh

ENTRYPOINT ["entrypoint.sh"]
CMD ["apache2-foreground"]
```

#### Le Script d'Entrée (entrypoint.sh)[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#le-script-dentree-entrypointsh "Permanent link")

Ce script démarre le service `cron` système en arrière-plan avant de lancer le serveur Apache au premier plan.

```
#!/bin/bash
set -e

# Démarrer le démon cron
cron

# Exécuter la commande par défaut du conteneur (apache2-foreground)
exec "$@"

```

#### Fichier de build (build.sh)[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#fichier-de-build-buildsh "Permanent link")

Pour simplifier la mise à jour, un script `build.sh` permet de relancer la compilation de l'image locale :

```
#!/bin/bash
set -e
cd "$(dirname "$0")"
docker build --build-arg MOODLE_BRANCH=MOODLE_500_STABLE -t moodle-custom:latest .

```

---

### Étape 2 : Configuration de la Persistance et du Docker Compose[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#etape-2-configuration-de-la-persistance-et-du-docker-compose "Permanent link")

Les fichiers de configuration opérationnelle sont regroupés sur l'hôte dans `/opt/docker/moodle-custom/` : - `docker-compose.yml` : Fichier de description des conteneurs. - `.env` : Fichier de variables d'environnement. - `secrets/` : Mots de passe de production isolés.

#### Fichier docker-compose.yml[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#fichier-docker-composeyml "Permanent link")

```services:
  mariadb:
    image: docker.io/bitnami/mariadb:latest
    container_name: moodle-custom-db
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - MARIADB_ROOT_PASSWORD_FILE=/run/secrets/mariadb_root_password
      - MARIADB_PASSWORD_FILE=/run/secrets/mariadb_password
      - MARIADB_USER=tutotech_moodle_db_user
      - MARIADB_DATABASE=tutotech_moodle_db
      - MARIADB_CHARACTER_SET=utf8mb4
      - MARIADB_COLLATE=utf8mb4_unicode_ci
      - ALLOW_EMPTY_PASSWORD=no
    volumes:
      - db:/bitnami/mariadb
    secrets:
      - mariadb_root_password
      - mariadb_password
    networks:
      - db

  moodle:
    image: moodle-custom:latest
    container_name: moodle-custom-app
    restart: unless-stopped
    env_file:
      - .env
    environment:
      - MOODLE_DATABASE_HOST=mariadb
      - MOODLE_DATABASE_PORT_NUMBER=3306
      - MOODLE_DATABASE_USER=tutotech_moodle_db_user
      - MOODLE_DATABASE_NAME=tutotech_moodle_db
      - MOODLE_DATABASE_PASSWORD_FILE=/run/secrets/mariadb_password
      - MOODLE_WWWROOT=https://learn.tutotech.org
      - MOODLE_SSL_PROXY=true
    volumes:
      - data:/var/moodledata
    secrets:
      - mariadb_password
    depends_on:
      - mariadb
    networks:
      - db
      - app
      - traefik-moodle
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=traefik-moodle"
      - "traefik.http.routers.moodle-custom.rule=Host(`learn.tutotech.org`)"
      - "traefik.http.routers.moodle-custom.entrypoints=https"
      - "traefik.http.routers.moodle-custom.tls=true"
      - "traefik.http.routers.moodle-custom.tls.certresolver=letsencrypt"
      - "traefik.http.services.moodle-custom.loadbalancer.server.port=80"

volumes:
  db:
    external: true
    name: hk1pkqqots65e6occy3wfmhn_db
  data:
    external: true
    name: hk1pkqqots65e6occy3wfmhn_data

networks:
  db:
  app:
  traefik-moodle:
    name: traefik-moodle
    external: true

secrets:
  mariadb_root_password:
    file: /opt/docker/moodle-custom/secrets/mariadb_root_password
  mariadb_password:
    file: /opt/docker/moodle-custom/secrets/mariadb_password

```
---

### Étape 3 : Migration de la Base de Données et Droits d'Accès[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#etape-3-migration-de-la-base-de-donnees-et-droits-dacces "Permanent link")

#### Le Piège des Droits d'Accès[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#le-piege-des-droits-dacces "Permanent link")

L'ancienne image Bitnami exécutait le serveur web en tant qu'utilisateur non-root `1001` (user `daemon`). L'image officielle `php:8.2-apache` exécute Apache sous l'utilisateur `www-data` (UID `33`).

Si vous démarrez directement le nouveau conteneur sans modifier la propriété physique du dossier de données sur l'hôte, le conteneur Moodle renverra des **erreurs HTTP 500** car il ne pourra pas écrire dans son dossier de données `/var/moodledata` (sessions, cache, fichiers).

Avant de lancer le nouveau déploiement, il faut réassigner la propriété du volume de données :
````
docker run --rm -v hk1pkqqots65e6occy3wfmhn_data:/var/moodledata alpine chown -R 33:33 /var/moodledata
```


#### Mise à niveau MariaDB[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#mise-a-niveau-mariadb "Permanent link")

L'ancienne base Bitnami reposait sur MariaDB 10.6. En pointant le nouveau Compose vers la dernière image `bitnami/mariadb:latest` (MariaDB 12), le conteneur MariaDB se chargera automatiquement de mettre à jour la base physique persistée dans le volume `hk1pkqqots65e6occy3wfmhn_db` au premier démarrage.

---

### Étape 4 : Intégration dans l'interface de Coolify[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#etape-4-integration-dans-linterface-de-coolify "Permanent link")

Afin de pouvoir contrôler Moodle directement via le tableau de bord Coolify (graphes, logs, gestion de statut) sans modifier l'emplacement des fichiers sur l'hôte :

1. Ouvrez l'interface Coolify, allez dans votre Projet -> service **moodle**.
2. Modifiez la configuration brute Docker Compose du service avec le contenu ci-dessus.
3. Assurez-vous d'avoir adapté le chemin des secrets et de l'environnement (`/opt/docker/moodle-custom/` au lieu de `/opt/docker/moodle/`).
4. Stoppez la pile de conteneurs démarrés manuellement sur l'hôte pour éviter les conflits :
    
```
   cd /opt/docker/moodle-custom && docker compose down
```
    
5. Déployez le service depuis Coolify. Les conteneurs créés s'appelleront `moodle-hk1pkqqots65e6occy3wfmhn` et `mariadb-hk1pkqqots65e6occy3wfmhn`.
6. Exécutez l'upgrade CLI de Moodle à l'intérieur du conteneur pour mettre à niveau les schémas de base de données :
    
```docker exec moodle-hk1pkqqots65e6occy3wfmhn php /var/www/html/admin/cli/upgrade.php --non-interactive
    
```
    

---

## Gestion des Plugins en Environnement Sans État (Stateless)[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#gestion-des-plugins-en-environnement-sans-etat-stateless "Permanent link")

### Pourquoi ne PAS installer de plugin via l'interface web ?[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#pourquoi-ne-pas-installer-de-plugin-via-linterface-web "Permanent link")

Dans un conteneur standard, le dossier `/var/www/html` n'est pas persistant (il n'est pas mappé à un volume pour des raisons de sécurité). Si vous autorisez l'interface web à installer un plugin, les fichiers PHP seront écrits de manière temporaire. Au prochain déploiement ou redémarrage de la pile, **le plugin disparaîtra**, provoquant un crash ou des dysfonctionnements majeurs puisque la base de données Moodle s'attend à ce que le plugin soit toujours présent.

### Procédure générale de déploiement d'un plugin[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#procedure-generale-de-deploiement-dun-plugin "Permanent link")

1. **Extraction** : Téléchargez et extrayez le plugin ou thème sur l'hôte dans `/home/nicolas-bodaine/moodle-custom-build/`.
2. **Nommage et structure du dossier** : Respectez strictement la structure attendue par Moodle.
3. Par exemple, pour un plugin local nommé `local_tutotech`, le dossier de build doit être `tutotech/` et copié vers `/var/www/html/local/tutotech`.
4. Pour un thème comme `theme_moove`, le dossier de build doit être `theme/moove/` et copié vers `/var/www/html/theme/moove`.
5. Pour un bloc comme `block_xp`, le dossier de build doit être `blocks/xp/` et copié vers `/var/www/html/blocks/xp`.
6. **Déclaration de copie** : Ajoutez les instructions de copie dans le `Dockerfile` (voir Étape 1) :
    
```
COPY tutotech /var/www/html/local/tutotech
COPY theme/moove /var/www/html/theme/moove
COPY blocks/xp /var/www/html/blocks/xp
   
```
    
7. **Build** : Compilez la nouvelle image locale en exécutant le script `./build.sh` dans `/home/nicolas-bodaine/moodle-custom-build/`.
8. **Déploiement** : Lancez une redéploiement forcé du service dans Coolify pour instancier les nouveaux conteneurs avec l'image fraîchement construite.
9. **Mise à niveau de la base de données** : Indiquez à Moodle d'appliquer les schémas et configurations en base :
   ``` 
    docker exec moodle-hk1pkqqots65e6occy3wfmhn php /var/www/html/admin/cli/upgrade.php --non-interactive`
    ```
    
10. **Purge du Cache** :
    ```
   docker exec moodle-hk1pkqqots65e6occy3wfmhn php /var/www/html/admin/cli/purge_caches.php`
    ```

---

### Cas Particulier : Le Thème Moove (`theme_moove`) et la modification du bandeau publicitaire[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#cas-particulier-le-theme-moove-theme_moove-et-la-modification-du-bandeau-publicitaire "Permanent link")

Le thème **Moove** (`theme_moove`) a été personnalisé pour retirer un bandeau publicitaire indésirable de "Conecti.me Partners" faisant la promotion de services tiers (ReadSpeaker, etc.) sur la page d'administration.

**Détails de la modification réalisée sur le thème :** 1. **Fichier modifié** : 
```
```

Ce thème épuré est directement intégré à l'image via l'étape de copie dans le `Dockerfile`.

---

### Cas Particulier : Module Level Up XP (`block_xp`)[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#cas-particulier-module-level-up-xp-block_xp "Permanent link")

Le module de gamification **Level Up XP** (`block_xp`) est un plugin tiers installé pour attribuer des points d'expérience aux apprenants. Il est stocké dans `/home/nicolas-bodaine/moodle-custom-build/blocks/xp` sur l'hôte et automatiquement injecté au même titre que les autres extensions.

---

## Points de Vigilance et Erreurs à Éviter[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#points-de-vigilance-et-erreurs-a-eviter "Permanent link")

Reversion de Permissions

Si vous devez interagir manuellement avec le volume de données utilisateurs via d'autres scripts ou conteneurs, veillez à ne jamais repasser la propriété à `1001:1001` (Bitnami) sous peine de casser l'accès en écriture d'Apache (`www-data:www-data`).

Dossier applicatif non sécurisé

N'utilisez jamais la commande `chown -R www-data:www-data /var/www/html` dans le conteneur de production. Bien que cela facilite l'installation de plugins via le web, cela détruit la barrière de sécurité principale en lecture seule et expose l'instance aux injections de code PHP malveillant.

---

## Vérification[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#verification "Permanent link")

Pour valider le fonctionnement et l'intégration :
```
# Vérifier la réponse HTTP [](https://wiki.bodaine.fr/logiciels/moodle-custom-docker
curl -kI https://learn.tutotech.org`
```
Résultat attendu

La commande renvoie un code de statut `HTTP/2 200` ou un redirect `302` avec l'en-tête `set-cookie: MoodleSession=...`.

---

## Ressources[¶](https://wiki.bodaine.fr/logiciels/moodle-custom-docker/#ressources "Permanent link")

- [Dépôt Officiel de Moodle sur GitHub](https://github.com/moodle/moodle)
- [Image Officielle PHP sur Docker Hub](https://hub.docker.com/_/php)
- [Documentation Docker Compose](https://docs.docker.com/compose/)