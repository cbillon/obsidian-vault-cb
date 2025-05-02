---
link: https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/
byline: Matthieu Anceret
site: Matthieu Anceret
date: 2022-05-02T00:00
excerpt: |-
  J’aime l’informatique !
  De fait, je commence à accumuler quelques appareils chez moi : un NAS Synology, un Intel NUC (qui fait office de serveur), parfois quelques Raspberry Pi… En plus de ça, j’adore bidouiller et tester des services que je peux self-hosted (via Docker le plus souvent).
  Jusqu’à maintenant, ces derniers étaient cantonnés à mon réseau local. Mais j’ai souvent rencontré le besoin d’y avoir accès depuis l’extérieur de mon domicile.
slurped: 2024-04-29T16:52:31.748Z
title: Traefik - A modern reverse proxy
---

J’aime l’informatique !

De fait, je commence à accumuler quelques appareils chez moi : un NAS Synology, un Intel NUC (qui fait office de serveur), parfois quelques Raspberry Pi… En plus de ça, j’adore _bidouiller_ et tester des services que je peux _self-hosted_ (via Docker le plus souvent).

Jusqu’à maintenant, ces derniers étaient cantonnés à mon réseau local. Mais j’ai souvent rencontré le besoin d’y avoir accès depuis l’extérieur de mon domicile. Pour cela, je souhaitais une solution souple (en particulier lors de l’ajout/suppression de service), sécurisée et suffisamment _simple_ pour éviter d’y passer trop de temps :D

C’est là que j’ai découvert **Traefik** !

## Table des matières

- [Traefik, qu’est-ce que c’est ? à quoi ça sert ?](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#traefik-quest-ce-que-cest--%C3%A0-quoi-%C3%A7a-sert-)
- [Architecture de Traefik](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#architecture-de-traefik)
    - [Les providers](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#les-providers)
    - [Les entrypoints](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#les-entrypoints)
    - [Les routers](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#les-routers)
    - [Les middlewares](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#les-middlewares)
    - [Les services](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#les-services)
- [Step by step, comment on le met en place ?](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#step-by-step-comment-on-le-met-en-place-)
    - [Pré-requis](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#pr%C3%A9-requis)
    - [Création du fichier docker-compose](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#cr%C3%A9ation-du-fichier-docker-compose)
    - [Configuration](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#configuration)
        - [Configuration statique](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#configuration-statique)
        - [Configuration dynamique (la magie !)](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#configuration-dynamique-la-magie-)
        - [Focus sur le provider _Docker_](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#focus-sur-le-provider-docker)
    - [Le dashboard](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#le-dashboard)
    - [Certificats SSL](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#certificats-ssl)
- [Plugins](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#plugins)
- [Pour aller plus loin…](https://anceret-matthieu.fr/2022/05/traefik-a-modern-reverse-proxy/#pour-aller-plus-loin)

## Traefik, qu’est-ce que c’est ? à quoi ça sert ?

[Traefik](https://docs.traefik.io/) est un reverse proxy écrit en Go. Autrement dit, c’est un outil permettant d’accéder à des services d’un réseau privé à partir d’un réseau public, et ce, sans exposer directement ces services.

Ses principaux concurrents sont Nginx, HAproxy ou encore Apache.

Il s’agit d’une solution open source française, créée par Emile Vauge en 2015. A la base, il s’agissait d’un side-project répondant à un besoin rencontré dans un projet. Après une publication sur GitHub, une communauté s’est rapidement créée et le succès est arrivé. Traefik est désormais une société, _Traefik Labs_ (anciennement _Containous_), et propose d’autres produits comme Mesh et Pilot. Comme quoi, il est toujours intéressant de partager ses projets personnels, ils peuvent parfois révéler de bonnes surprises et susciter un intérêt ;)

Petite anecdote rigolote, chaque release de Traefik porte le nom d’un fromage :D

- v1.1 : Camembert
- v2 : Montdor
- v2.3 : Picodon
- v2.5 : Brie

Traefik est particulièrement adapté dans le cas où l’on travaille avec des conteneurs (et donc une infrastructure potentiellement _mouvante_). Il est en effet nativement capable de _dialoguer_ avec des orchestrateurs ou des services registry pour adapter sa configuration dynamiquement. Dans le langage Traefik, on appelle ça des _providers_, et on y trouve Docker, Consul, Kubernetes, Rancher… C’est donc extrêmement pratique pour réagir lorsqu’un changement a lieu dans l’infrastructure (déploiement d’une nouvelle application, ajout d’un nouveau serveur, démarrage d’un conteneur…).

Il dispose de bien d’autres avantages intéressants comme l’intégration de Let’s Encrypt, une UI Web pour visualiser la configuration, des connecteurs pour les métriques et les logs (Prometheus, Datadog, InfluxDB), mais je vais détailler cela dans la suite de l’article.

La période d’apprentissage n’est pas très longue et la documentation est vraiment très bien faite. Ensuite c’est un réel plaisir (et un énorme gain de productivité) de l’utiliser ! Par contre, il **FAUT** pratiquer ;)

## Architecture de Traefik

![https://doc.traefik.io/traefik/routing/](https://res.cloudinary.com/anceret-matthieu/image/upload/v1649341191/posts/traefik/architecture-overview.png) https://doc.traefik.io/traefik/routing/

Pour bien utiliser Traefik, il est important de comprendre les différentes briques et leur rôle dans le traitement d’une requête entrante. Je vais les présenter une par une, et je vous invite à vous référer au schéma ci-dessus pour visualiser leur ordonnancement.

Pour chaque brique, j’ai ajouté un schéma ainsi que le lien vers la documentation associée. N’hésitez pas à la consulter pour obtenir des détails complémentaires ;)

## Les providers

C’est la brique principale, car sans elle, il n’y aurait rien à exposer ;) Les providers se chargent de la découverte des services qui s’exécutent sur votre infrastructure. Traefik va les interroger (via leurs APIs) pour obtenir les informations nécessaires.

Il existe 4 catégories de providers :

- **Label-based :** chaque conteneur possède des labels qui lui sont attachés
- **Key-Value-based :** chaque conteneur met à jour un magasin de type _key-value_
- **Annotation-based :** un objet distinct contient l’ensemble des caractéristiques du conteneur
- **File-based :** utilisation d’un fichier pour définir la configuration

Chaque provider possède son propre espace de nom (namespace) contenant les middlewares et les services.

Je présente un peu plus loin dans l’article le provider que j’utilise le plus : Docker (qui est du type _label-based_).

## Les entrypoints

![https://doc.traefik.io/traefik/routing/entrypoints/](https://res.cloudinary.com/anceret-matthieu/image/upload/v1651150596/posts/traefik/entrypoints.png) https://doc.traefik.io/traefik/routing/entrypoints/

Il s’agit des points d’entrée sur lesquels Traefik écoute pour prendre en compte les requêtes entrantes. Pour chacun, on va définir au minimum un port et un protocole (TCP ou UDP).

Les entrypoints font partie de la configuration statique (rassurez-vous, j’explique un petit peu plus loin de quoi il s’agit).

Dans mon cas, j’ai choisi de définir 2 points d’entrées : le premier, _web_, chargé d’écouter sur le port 80 et le second, _websecure_, sur le port 443. J’en ai profité pour ajouter quelques directives me permettant de rediriger automatiquement les flux du port 80 vers le port 443 (autrement dit, HTTP vers HTTPs).

Voici la configuration nécessaire pour réaliser cela :

```
# traefik/conf/traefik.yml

entryPoints:
  web:
    address: ":80"
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https
          permanent: true

  websecure:
    address: ":443"
```

## Les routers

![https://doc.traefik.io/traefik/routing/routers/](https://res.cloudinary.com/anceret-matthieu/image/upload/v1651151089/posts/traefik/routers.png) https://doc.traefik.io/traefik/routing/routers/

Ils ont pour rôle d’analyser les requêtes (hôte, headers…) et de les connecter vers les services appropriés.

Pour cela, on va définir des règles pour permettre à Traefik de savoir si la requête match. Le plus souvent, on va se baser sur le nom de domaine (`rule = "Host('my-host-name.com')"`), mais on peut aussi utiliser les en-têtes, le path ou même l’adresse IP du client. On peut évidemment combiner plusieurs critères (avec les opérateurs _&&_ et _||_) ainsi qu’utiliser des regex. Avec tout ça, on obtient un système très puissant permettant de configurer _aux petits oignons_ notre mécanisme de routage !

Il est possible de définir plusieurs routeurs pour un même service (par exemple, dans le cas où un conteneur expose plusieurs ports).

Pour illustrer un peu ce que l’on a vu jusqu’à maintenant, voici un docker-compose d’un service que j’expose sur mon serveur. Attention, celui-ci n’est pas complet, j’ai retiré les éléments que l’on a pas encore vu.

```
# whoami/docker-compose.yml

version: '3.7'

services:
    whoami:
        image: traefik/whoami:v1.6.0
        container_name: "whoami"
        labels:
            # On indique que Traefik doit prendre en compte ce conteneur
            - "traefik.enable=true"
            # On déclare le routeur 'whoami' pour notre application avec une règle qui match sur le hostname
            - "traefik.http.routers.whoami.rule=Host(`whoami.my-domain.fr`)"
            # On précise le entrypoint sur lequel ce routeur va être effectif
            - "traefik.http.routers.whoami.entrypoints=websecure"
```

J’aurais aussi pu exposer mon service via l’url suivante : `my-domain.fr/whoami`. Dans ce cas, j’aurais utilisé la combinaison de règles suivante : `Host('my-domain.fr') && Path('/whoami')`.

Si jamais vous avez spécifié dans la configuration du provider Docker `exposedByDefault=true`, la ligne `traefik.enable=true` n’est pas nécessaire.

## Les middlewares

![https://doc.traefik.io/traefik/middlewares/overview/](https://res.cloudinary.com/anceret-matthieu/image/upload/v1651152927/posts/traefik/middlewares.png) https://doc.traefik.io/traefik/middlewares/overview/

Les middlewares sont attachés à un routeur. Ils permettent de modifier la requête avant qu’elle soit envoyée au service.

Traefik embarque quelques middlewares _prêts à l’utilisation_ :

- **BasicAuth :** permet l’ajout d’une authentification basique. Il est très utile pour _protéger_ un service n’ayant pas de système d’authentification (même si nous aborderons une solution plus fiable à la fin de l’article)
- **Headers :** permet d’ajouter ou de modifier les headers d’une requête
- **RateLimit :** permet d’appliquer une limitation sur le nombre d’appels
- **CircuitBreaker :** en se basant sur la santé du service sous-jacent, ce middleware permet d’éviter d’appeler un service qui est down
- **Retry :** permet de réessayer une requête plusieurs fois en l’absence de réponse

Une liste exhaustive des middlewares HTTP est [disponible ici](https://doc.traefik.io/traefik/middlewares/http/overview/).

## Les services

![https://doc.traefik.io/traefik/routing/services/](https://res.cloudinary.com/anceret-matthieu/image/upload/v1651153573/posts/traefik/services.png) https://doc.traefik.io/traefik/routing/services/

Il s’agit du dernier maillon de la chaine. Les services transmettent la requête au système concerné. C’est ici que l’on va définir comment on atteint le système : adresse IP (ou les adresses IP si l’on souhaite mettre en place du load-balancing), schéma, port…

C’est essentiellement utilisé avec le **File Provider**. Par exemple, je l’utilise pour donner l’accès à mon NAS Synology (qui est une autre machine de mon réseau) :

```
# traefik/dyn_conf/synology.yml

http:
  services:
    service-dsm:
      loadBalancer:
        passHostHeader: true
        servers:
          - url: "https://192.168.1.1:1234/"
  routers:
    dsm:
      entryPoints:
        - websecure
      rule: "Host(`synology.my-domain.fr`)"
      service: service-dsm
```

J’ai laissé la partie `routers` dans laquelle je définis le point d’entrée et la règle sur le hostname. Comme ça, vous avez tout ;)

## Pré-requis

- Un serveur (une machine) exposé sur internet (= accessible) à travers les ports 80 et 443 (pensez à faire les ouvertures et les redirections au niveau de votre box internet)
- Un nom de domaine pointant sur ce serveur
- [Docker](https://docs.docker.com/get-docker/) et [docker-compose](https://docs.docker.com/compose/install/)

## Création du fichier docker-compose

Traefik est disponible sous la forme d’une image Docker. L’installation est donc simple si vous avez l’habitude de cette stack ;)

J’ai pour habitude de séparer mes services dans leur propre fichier Docker Compose (et donc dans leur propre dossier). Cela me permet de gagner en lisibilité et de pouvoir intervenir sur un service en particulier sans impacter les autres. Evidemment, je garde les services fortement liés dans un même fichier. Par exemple, Umami, mon outil de statistiques et sa base PostgreSQL sont dans un même fichier Compose (pour le moment, je n’ai pas de base de données mutualisée car j’ai peu de service qui en ont besoin).

Attention à la gestion des réseaux dans Docker. Les réseaux définissent les règles de communication entre les conteneurs et entre les conteneurs et le système hôte. Ils garantissent l’isolation et le cloisonnement tout en permettant aux applications de fonctionner ensemble. Par défaut, Compose configure un unique réseau pour chaque conteneur. Chaque conteneur rejoint automatiquement ce réseau par défaut.

J’ai créé un réseau _web_, en dehors du fichier Compose (d’où le fait qu’il soit taggé _external_), via la commande `docker network create web`. Ce réseau sera commun entre Traefik et l’ensemble des conteneurs que je souhaite exposer.

```
version: '3'

services:
  traefik:
    image: traefik:latest
    container_name: "traefik"
    restart: always
    ports:
      - "80:80"
      - "443:443"
    networks:
      - web
      - default
    env_file:
      - './conf/.ovh-api.env'
    volumes:
      # Mapping sur le socket interne de Docker
      - '/var/run/docker.sock:/var/run/docker.sock'
      # Mapping du fichier de configuration statique
      - './conf/traefik.yml:/traefik.yml'
      # Mapping du dossier contenant la configuration dynamique
      - './conf/dyn_traefik/:/dyn_traefik/'
      # Mapping du fichier de stockage des certificats
      - './conf/acme.json:/acme.json'
    labels:
      - "traefik.enable=true"

networks:
  default:
  web:
    external: true
```

Attention, dans le cas où le conteneur est attaché à plusieurs réseaux (le réseau `web` car il est exposé par Traefik et un réseau `backend` pour discuter avec une base de données ou des composants internes), il est nécessaire d’ajouter un label indiquant à Traefik quel réseau utiliser : `traefik.docker.network=web`. Sur ce point, je vous invite à faire un tour sur [cet article](https://lefthandbrain.com/isolating-docker-containers/) qui explique parfaitement le fonctionnement et les différents cas de figure.

Par la suite, la commande `docker-compose logs -f` pourra vous être utile en cas de comportement _bizarre_ ;) Si vous utilisez Portainer, vous pouvez aussi passer par sa WebUI.

Attention, ne démarrez pas encore la stack Docker, il reste à faire la configuration !

## Configuration

La configuration de Traefik est un peu particulière car elle est composée de 2 _modes_ : la configuration **statique** et la configuration **dynamique**.

Concernant le format, Traefik supporte le TOML et le YAML. Personnellement, je privilégie le YAML, que je trouve plus lisible et avec lequel j’ai plus l’habitude de travailler (sur les pipelines Azure DevOps par exemple !).

### Configuration statique

La configuration statique, aussi appelée _startup configuration_, est chargée uniquement à chaque démarrage de Traefik. En cas de modification, elle nécessite donc un redémarrage pour être prise en compte ;)

Elle concerne principalement le paramétrage des providers ainsi que la définition des entrypoints. Logiquement, ce sont des éléments qui ne sont pas sensés changer souvent, donc pas d’inquiétude !

Traefik propose 3 façons d’écrire cette configuration. Elles sont exclusives, il faut donc en choisir une et une seule et s’y tenir.

- via un fichier de configuration
- via les arguments de ligne de commande
- via les variables d’environnement

J’ai choisi de passer par un fichier de configuration car cela me permet de séparer la configuration pure de Traefik de la partie concernant la construction de la stack Docker Compose (et aussi car je trouve la syntaxe YAML plus claire).

Une fois Traefik correctement configuré, il n’y aura normalement plus besoin de le relancer. C’est désormais la configuration dynamique qui va prendre le relais !

### Configuration dynamique (la magie !)

C’est cette configuration dynamique qui apporte le côté _magique_ à Traefik !

Elle provient de l’exploitation des providers définis dans la configuration statique. Honnêtement, il n’y a pas grand chose à dire sur cette configuration (hors spécificités liées à chaque provider). Je vais donc détailler les cas du provider Docker, qui est celui que j’utilise le plus sur mon serveur.

### Focus sur le provider _Docker_

Comme son nom l’indique, ce provider permet d’exposer des conteneurs Docker ;)

La 1ère chose à faire est donc de configurer le provider dans la configuration statique. Celui-ci a besoin de se connecter au socket Docker. Dans mon cas, j’ai désactivé l’exposition par défaut des conteneurs car je souhaite l’activer manuellement sur les conteneurs concernés.

```
# traefik/conf/traefik.yml

docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
```

Ensuite, la configuration passe uniquement par les labels, que l’on vient positionner sur nos conteneurs. Sur le même principe que le conteneur `whoami` vu plus haut, j’ai un conteneur `smokeping` qui est un service qui me permet de mesurer et d’historiser la latence de mon réseau. On note la configuration du routeur avec une règle sur le hostname, un point d’entrée et l’activation du TLS avec Let’s Encrypt (pour la génération du certificat).

```
# smokeping/docker-compose.yml

version: '3'

services:
  smokeping:
    image: linuxserver/smokeping
    container_name: smokeping
    restart: always
    networks:
      - web
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.smokeping.rule=Host(`smokeping.my-domain.fr`)"
      - "traefik.http.routers.smokeping.entrypoints=websecure"
      - "traefik.http.routers.smokeping.tls=true"
      - "traefik.http.routers.smokeping.tls.certResolver=letsencrypt"

networks:
  web:
    external: true
```

Il ne reste qu’à répéter l’opération sur l’ensemble des conteneurs que vous souhaitez exposer. Je vous invite à consulter [la documentation](https://doc.traefik.io/traefik/providers/docker/) pour une vue exhaustive de l’ensemble des paramètres.

## Le dashboard

![Traefik dashboard](https://res.cloudinary.com/anceret-matthieu/image/upload/v1651176042/posts/traefik/traefik-dashboard.png) Traefik dashboard

Le dashboard est une interface web permettant de visualiser l’état de Traefik en temps réel :

- les routes
- les services
- les providers

C’est très pratique pour consulter de manière visuelle et synthétique l’ensemble de la configuration de Traefik. On peut consulter le détail d’un routeur, ce qui peut parfois permettre de comprendre un dysfonctionnement (que l’on n’aurait pas réussi à identifier via les fichiers de configuration).

![Détail du routeur whoami](https://res.cloudinary.com/anceret-matthieu/image/upload/v1651176218/posts/traefik/traefik-dashboard-2.png) Détail du routeur whoami

Il existe aussi une API qui permet d’accéder aux mêmes informations. L’ensemble des endpoints disponibles est détaillé [ici](https://doc.traefik.io/traefik/operations/api/).

## Certificats SSL

En 2022, on ne peut plus se permettre de passer à côté du HTTPS, et ce, même pour un usage personnel. Par chance, Traefik propose l’utilisation du provider ACME avec Let’s Encrypt pour la génération automatique des certificats !

> Le protocole ACME (de l’anglais _Automatic Certificate Management Environment_ littéralement « environnement de gestion automatique de certificat ») est un protocole de communication pour l’automatisation des échanges entre les autorités de certification et les propriétaires de serveur web. Il a été conçu par l’Internet Security Research Group pour leur service Let’s Encrypt.
> 
> Wikipédia : [https://fr.wikipedia.org/wiki/ACME_(protocole)](https://fr.wikipedia.org/wiki/ACME_(protocole))

Les certificats sont stockés dans un fichier spécifique qu’il est nécessaire de créer **AVANT** le démarrage de Traefik. Dans le cas contraire, celui-ci va créer un dossier et ça ne fonctionnera pas…

```
touch acme.json
chmod 600 acme.json
```

Il faut ensuite configurer un _résolveur de certificat_ au niveau de la configuration statique :

```
certificatesResolvers:
  letsencrypt:
    acme:
      caServer: "https://acme-v02.api.letsencrypt.org/directory"
      #caServer: https://acme-staging-v02.api.letsencrypt.org/directory
      # Attention à bien saisir une adresse mail valide !
      email: "[email protected]"
      storage: "/acme.json"
      dnsChallenge:
        provider: ovh
        delayBeforeCheck: 10
        resolvers:
          - "1.1.1.1:53"
          - "8.8.8.8:53"
```

Attention, lors de vos tests, pensez à utiliser le serveur `staging` de Let’s Encrypt. En effet, la version de production possède [des limitations](https://letsencrypt.org/docs/rate-limits) au niveau du nombre d’appels. Ne faites la bascule que lorsque vous êtes totalement prêt ;)

La partie `challenge` (aka. défi) permet de définir la façon dont Let’s Encrypt valide que l’on contrôle bien le nom de domaine du certificat. Il existe 3 types de challenge :

- **HTTP challenge (HTTP-01) :** c’est le type le plus courant. Il consiste à déposer un fichier sur son serveur web (fichier fourni par Let’s Encrypt et contenant un jeton et une empreinte). Une fois ceci fait, Let’s Encrypt va essayer d’accéder à ce fichier pour le valider et autoriser l’émission de certificat. Ce défi ne peut-être effectué que sur le port 80.
- **TLS challenge (TLS-ALPN-01) :** assez compliqué à mettre en place car peu pris en charge.
- **DNS challenge (DNS-01) :** consiste à prouver que l’on contrôle le DNS de notre nom de domaine en plaçant une valeur spécifique dans un enregistrement TXT. Ce défi permet d’émettre des certificats génériques. Pour utiliser ce défi, il faut que le fournisseur DNS dispose d’une API permettant de réaliser ce type d’opération ([la liste est assez longue](https://community.letsencrypt.org/t/dns-providers-who-easily-integrate-with-lets-encrypt-dns-validation/86438)).

Dans mon cas, j’ai choisi d’utiliser le challenge DNS car cela me permet de générer des certificats SSL pour des services internes (non exposé sur internet) ainsi que d’obtenir des certificats _wildcard_. Si comme moi, votre fournisseur DNS est OVH, je vous invite à suivre [cet excellent article](https://www.grottedubarbu.fr/traefik-dns-challenge-ovh/) pour connaitre la procédure détaillée.

## Plugins

Traefik propose des plugins permettant d’étendre ses fonctionnalités. La contrepartie est de connecter son instance Traefik à Traefik Pilot, qui est une solution SaaS offrant quelques fonctionnalités supplémentaires, comme des métriques sur l’activité réseau et des alertes sur la santé de Traefik.

J’ai identifié 2 plugins intéressants :

- **[Containers on demand](https://pilot.traefik.io/plugins/605afbdba5f67ab9a1b0e53a/containers-on-demand) :** permet de démarrer un conteneur uniquement lors de la 1ère requête vers ce service (et d’afficher une page d’attente propre). Très pratique pour éviter de laisser tourner des services qui sont peu utilisés.
- **[Fail2Ban](https://pilot.traefik.io/plugins/61d71dc2d80115c9a5b32e68/fail2-ban) :** implémentation simplifiée de fail2ban permettant de faire du filtrage IP (_whitelist_ et _blacklist_) via un middleware.

## Pour aller plus loin…

Nous avons vu les bases pour déployer et configurer Traefik avec un backend composé d’une multitude de conteneurs Docker. Cependant, il reste encore de nombreux points à aborder :

- Mettre en place des middlewares pour gérer de manière plus _propre_ les erreurs : les 404 pour les services inexistants et les 5xx pour les indisponibilités ([https://doc.traefik.io/traefik/middlewares/http/errorpages/](https://doc.traefik.io/traefik/middlewares/http/errorpages/))
- Externaliser les métriques dans Prometheus ([https://doc.traefik.io/traefik/observability/metrics/overview/](https://doc.traefik.io/traefik/observability/metrics/overview/)) et les traces dans Jaeger ou Zipkin ([https://doc.traefik.io/traefik/observability/tracing/overview/](https://doc.traefik.io/traefik/observability/tracing/overview/))
- Mettre en place une brique d’authentification centralisée via la solution open-source [Authelia](https://github.com/authelia/authelia)

J’espère pouvoir aborder ces points dans les semaines à venir, et je ne manquerais pas d’écrire à ce sujet.

Pour conclure, Traefik est un outil formidable qui offre beaucoup de flexibilité. Il est particulièrement utile dans le cas d’un environnement _hybride_ (une partie sous Docker et une partie en physique par exemple).

Je vous laisse avec un peu de lecture :

- Un interview d’Emile Vauge au début de l’aventure Traefik : [https://blog.zenika.com/2016/02/18/du-nouveau-dans-les-projets-zstartup-de-zenika-presentation-du-projet-traefik-io-cree-par-emile-vauge/](https://blog.zenika.com/2016/02/18/du-nouveau-dans-les-projets-zstartup-de-zenika-presentation-du-projet-traefik-io-cree-par-emile-vauge/)
- Un benchmark sur les performances de plusieurs reverse proxy : [https://github.com/NickMRamirez/Proxy-Benchmarks](https://github.com/NickMRamirez/Proxy-Benchmarks)
- La documentation sur [le provider HTTP](https://doc.traefik.io/traefik/providers/http/) qui permet de fournir la configuration via un endpoint HTTP (et donc potentiellement générée via une API !)