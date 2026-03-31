---
link: https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/
byline: Florian BURNEL
site: IT-Connect
date: 2025-11-20T18:00
excerpt: "Tutoriel complet pour installer et configurer Traefik : un reverse proxy moderne pour exposer vos services Web exécutés sur des conteneurs Docker."
twitter: https://twitter.com/@itconnect_fr
tags:
  - traefik
slurped: 2026-03-31T17:43
title: "Guide Traefik : exposer des apps Web avec un reverse proxy"
---

**Traefik est un reverse proxy (et un load balancer) open source conçu pour les applications web (HTTP/HTTPS) et la gestion des connexions TCP. Créé par le Français Émile Vauge, Traefik s'est imposé comme une solution incontournable pour publier des applications exécutées au sein de conteneurs Docker, bien que ce ne soit pas le seul cas d'utilisation. Comment fonctionne-t-il ? Comment installer Traefik ? Réponse dans ce guide complet.**

Dans un environnement basé sur **Docker**, il est fréquent d’héberger plusieurs applications web sur un même serveur. Chaque conteneur peut exposer un port différent, mais il devient vite **difficile d’organiser les accès**, de gérer **les certificats TLS**, et d’ajouter ou retirer des services sans casser les autres. La solution naturelle pour publier ces applications et mieux s'y retrouver, c’est d’utiliser un [reverse proxy](https://www.it-connect.fr/les-serveurs-proxy-et-reverse-proxy-pour-les-debutants/ "Les serveurs proxy et reverse proxy pour les débutants"). Et parmi les **reverse proxy capables de tirer parti de Docker** de manière dynamique, Traefik s’est imposé comme une référence en tant que **reverse proxy pour microservices**.

La force de Traefik, c'est notamment sa capacité à détecter automatiquement vos conteneurs et à simplifier l'intégration d'applications avec de simples labels. À cela s'ajoute la possibilité d'obtenir automatiquement des certificats TLS via Let’s Encrypt (ACME).

Dans ce tutoriel, nous allons **installer Traefik sur un serveur Linux (Debian 13) avec Docker Compose**, puis déployer une première application pour valider que tout fonctionne correctement. Cette application sera publiée avec **Traefik en HTTP puis en HTTPS** (avec l'obtention automatique d'un certificat TLS). L’objectif est de comprendre les briques fondamentales de Traefik.

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Traefik-reverse-proxy-HTTPS-ovhcloud.png.webp)

Sommaire

- [Les composants de Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Les_composants_de_Traefik)
- [Prérequis pour Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Prerequis_pour_Traefik)
- [Prise en main de Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Prise_en_main_de_Traefik)
    - [Créer la racine du projet](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Creer_la_racine_du_projet)
    - [Créer un réseau Docker pour Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Creer_un_reseau_Docker_pour_Traefik)
    - [Le fichier traefik.yml](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Le_fichier_traefikyml)
    - [Intégrer Docker à Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Integrer_Docker_a_Traefik)
    - [Accéder au tableau de bord Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Acceder_au_tableau_de_bord_Traefik)
    - [Associer un service HTTP à Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Associer_un_service_HTTP_a_Traefik)
- [Traefik HTTPS : comment ça marche ?](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Traefik_HTTPS_comment_ca_marche)
    - [ACME avec Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#ACME_avec_Traefik)
    - [Traefik : exposer le port 443](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Traefik_exposer_le_port_443)
    - [Créer une clé d'API OVHCloud pour Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Creer_une_cle_dAPI_OVHCloud_pour_Traefik)
    - [Utiliser OVHCloud avec Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Utiliser_OVHCloud_avec_Traefik)
    - [Ajouter un résolveur ACME](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Ajouter_un_resolveur_ACME)
    - [Ajouter un routeur HTTPS à Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Ajouter_un_routeur_HTTPS_a_Traefik)
    - [Traefik : redirection HTTP vers HTTPS](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Traefik_redirection_HTTP_vers_HTTPS)
- [Les journaux de debug de Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Les_journaux_de_debug_de_Traefik)
- [Conclusion](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#Conclusion)
- [FAQ](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/#FAQ)

## Les composants de Traefik

Avant même d’écrire un fichier Docker Compose et de foncer dans le **déploiement du reverse proxy Traefik**, il est nécessaire de comprendre comment il est structuré. Le schéma ci-dessous met en évidence le positionnement central de Traefik dans l'accès aux applications web et aux API. Du côté backend, même si ce tutoriel est axé sur l'intégration avec Docker, il est envisageable de router le trafic vers d'autres destinations (Kubernetes, machine virtuelle, etc.).

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Fonctionnement-de-Traefik-reverse-proxy-800x450.jpg.webp)

Source : Traefik

Traefik a quatre briques fondamentales :

|Élément|Rôle|
|---|---|
|Entrypoint|Représente un point d'entrée, c'est-à-dire un port sur lequel Traefik écoute (par exemple, du classique : HTTP sur 80, HTTPS sur 443)|
|Router|Décide vers quel service envoyer une requête, en fonction de règles. Par exemple, une correspondante avec un nom d'hôte : app1.it-connect.fr.|
|Service|Représente réellement l’application cible, le conteneur Docker qui doit recevoir les requêtes|
|Middleware|Permet d’appliquer une transformation sur une requête (par exemple : ajout d’authentification HTTP, redirection, headers, etc.)|

Bien qu'il y ait une partie de la configuration de Traefik qui soit statique, il y a aussi une partie de la configuration qui est vivante, voire même dynamique. En pratique, comme nous le verrons par la suite, pour **exposer une app via Traefik**, nous devons simplement ajouter les labels qu’il faut sur le conteneur, et Traefik récupère la configuration via Docker.

## Prérequis pour Traefik

Pour la suite, il faut un serveur (Linux) sur lequel Docker et Docker Compose sont installés. Le tutoriel s’applique aussi bien sur un VPS cloud que sur un serveur physique local. Vous devez aussi disposer d'un nom de domaine : **`it-connectlab.fr`** sera utilisé pour cette démonstration. Un enregistrement DNS a été fait pour une application dont l'adresse IP correspond au serveur Traefik, ce dernier étant en charge de router le trafic (principe du reverse proxy).

Pour installer Docker sur Linux, vous pouvez suivre ce tutoriel :

- [Installation de Docker sur Linux](https://www.it-connect.fr/installation-pas-a-pas-de-docker-sur-debian-11/ "Installation pas à pas de Docker sur Debian 13")

## Prise en main de Traefik

### Créer la racine du projet

Nous allons commencer par créer un dossier pour stocker les données de l'application Traefik.

```
mkdir /opt/docker-compose/traefik
cd /opt/docker-compose/traefik
```

À la racine de ce dossier, nous allons créer le fichier `**docker-compose.yml**`.

En guise de point de départ, nous allons utiliser le **fichier Docker Compose** disponible dans la [documentation de Traefik](https://doc.traefik.io/traefik/getting-started/docker/). La **version 3.6 de Traefik** sera utilisée, il s'agit de la dernière version stable actuelle. **Deux ports sont exposés** au niveau de ce conteneur : **`80`**, pour bénéficier d'un entryPoint en HTTP (pour publier une application en HTTP) et **`8080`** pour l'accès au tableau de bord de Traefik.

```
# docker-compose.yml
services:
  traefik:
    image: traefik:v3.6
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
```

Trois commandes sont aussi associées à l'exécution de ce conteneur en tant qu'arguments :

|Argument|Signification|
|---|---|
|`--api.insecure=true`|Active le dashboard web de Traefik, sans authentification (sur le port 8080 par défaut).|
|`--providers.docker=true`|Active le Provider Docker de Traefik pour découvrir automatiquement les services / labels.|
|`--entrypoints.web.address=:80`|Déclare un entryPoint nommé `**web**`, qui écoute sur port 80 dans le conteneur. Idéal pour les accès HTTP.|

Enfin, une dernière ligne importante est intégrée à ce fichier : **`/var/run/docker.sock:/var/run/docker.sock`**. Elle permet à Traefik d’accéder au socket Docker pour "voir" en temps réel les autres conteneurs. C'est important pour l'intégration de Traefik avec Docker, notamment pour lire les **`labels`**.

### Créer un réseau Docker pour Traefik

Vous remarquerez que ce Docker Compose ne contient pas de directives pour la connexion au réseau du conteneur. C'est la première évolution que nous allons lui apporter. Nous allons créer un réseau Docker nommé **`frontend`**, ce qui me semble être un nom adapté dans le contexte d'un reverse proxy.

```
docker network create frontend
```

Une fois le réseau créé, vérifiez qu'il est bien présent :

```
docker network ls
```

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Creer-un-reseau-Docker-pour-Traefik-800x152.jpg.webp)

Dans le fichier Docker Compose, nous allons associer ce réseau au conteneur Traefik. Il sera déclaré comme un réseau externe (**`external: true`**) pour ne pas que Docker cherche à le créer.

```
# docker-compose.yml
services:
  traefik:
    image: traefik:v3.6
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - frontend
networks:
  frontend:
    external: true
```

> **Note** : le port HTTPS, à savoir 443, n'est pas exposé pour le moment. Si vous envisagez dès maintenant de publier une application HTTPS, vous pouvez ajouter le port dès maintenant via l'ajout d'une ligne supplémentaire.

### Le fichier traefik.yml

La configuration globale et statique de Traefik se déclare par l'intermédiaire du fichier **`traefik.yml`**, au format YAML.

Nous allons ajouter une ligne sous **`volumes`** pour déclarer le fichier de configuration de Traefik, afin qu'il soit persistant et modifiable, tout en étant lu par le conteneur. Par la suite, nous manipulerons à plusieurs reprises le fichier **`traefik.yml`** pour ajouter différentes directives.

```
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
```

Le fichier doit donc être créé à cet emplacement : **`/opt/docker-compose/traefik/traefik.yml`**.

Éditez le fichier et ajoutez le code précisé ci-dessous pour définir les bases de **Traefik en mode proxy inverse pour HTTP et HTTPS**.

Nous désactivons l’envoi de statistiques anonymisées (**`sendAnonymousUsage`**) et la vérification automatique de nouvelles versions (**`checkNewVersion`**). Nous activons le tableau de bord interne (port 8080) en mode non sécurisé (pas adapté à la production !) afin de pouvoir y accéder dans un premier temps pour visualiser les routes de Traefik. Enfin, on déclare deux entryPoints : un pour le trafic HTTP classique sur le port 80 (nommé **`web`**), et un second pour le trafic HTTPS sécurisé sur le port 443 (nommé **`websecure`**). Cela prépare Traefik à écouter sur ces deux ports et à accepter des requêtes entrantes que l’on pourra ensuite router vers nos conteneurs.

```
# Configuration statique de Traefik
global:
  checkNewVersion: false
  sendAnonymousUsage: false

api:
  dashboard: true # Dashboard Traefik : 8080
  insecure: true

entryPoints:
  web:
    address: ":80" # EntryPoint pour HTTP
  websecure:
    address: ":443" # EntryPoint pour HTTPS
```

Enregistrez et fermez le fichier.

**Pour éviter les doublons entre la configuration statique et les arguments en ligne de commande**, je vous invite à supprimer la ligne command et les trois lignes associées dans le fichier Docker Compose, à savoir :

```
    command:
      - "--api.insecure=true"
      - "--providers.docker=true"
      - "--entrypoints.web.address=:80"
```

### Intégrer Docker à Traefik

Dans le fichier Docker Compose, le nécessaire a été fait pour que le conteneur Traefik dispose d'un accès au socket de Docker (**`/var/run/docker.sock`**). Mais ce lien n'est pas exploité en l'état actuel de la configuration. Dans le fichier de configuration⁣ **`traefik.yml`**, il est nécessaire d'ajouter un nouveau fournisseur (Provider) correspondant à Docker.

Éditez de nouveau le fichier **`traefik.yml`** et ajoutez ce bloc à la fin du fichier :

```
providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
```

La directive **`exposedByDefault`** joue un rôle important ! Si elle est sur **`false`** comme ici, cela signifie que **Traefik va détecter les conteneurs Docker sans les exposer automatiquement** : seuls les conteneurs qui auront **des labels Traefik** seront pris en compte et routés, ce qui évite d’exposer par erreur un service qui ne devrait pas être accessible depuis l’extérieur.

Désormais, enregistrez la configuration et lancez la construction du **conteneur Traefik** (depuis le répertoire **`/opt/docker-compose/traefik`**).

```
docker compose up -d
```

### Accéder au tableau de bord Traefik

Ouvrez un navigateur web et **accédez au tableau de bord de Traefik** via l'adresse IP de l'hôte Docker sur le port **8080**. Par exemple : **`http://192.168.10.200:8080`**. Ce tableau de bord ne permet pas d'effectuer la configuration en ligne, mais il présente l'avantage d'afficher l'état de la configuration actuelle. Vous devriez notamment **visualiser les 3 entryPoints de notre configuration Traefik** (celui du dashboard est automatique quand ce dernier est activé).

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Tableau-de-bord-de-Traefik-800x404.png.webp)

### Associer un service HTTP à Traefik

Traefik tourne sur notre serveur, c'est une bonne chose. Mais, pour le moment, il ne sert pas à grand-chose car aucun service n'est exposé par son intermédiaire. Pour commencer, nous allons **exposer une application** facile à héberger : [IT-Tools](https://www.it-connect.fr/it-tools-boite-a-outils-web-indispensable-pour-les-pros-de-informatique/ "IT-Tools : une boîte à outils web indispensable pour les pros de l’informatique"), une boite à outils en ligne. Dans un premier temps, l'accès sera effectué en HTTP, puisque la gestion du certificat rajoute une couche de complexité supplémentaire.

Créez un dossier pour ce projet :

```
mkdir /opt/docker-compose/it-tools
```

Dans ce répertoire, créez un fichier **`docker-compose.yml`** et insérez le code ci-dessous. Ce service lance l’application **IT-Tools** à partir de l’image officielle `**ghcr.io/corentinth/it-tools:latest**` et nomme le conteneur `**it-tools**`. Il est important de noter aussi que ce conteneur est **rattaché au réseau Docker externe `frontend`**, le même réseau que Traefik : c’est ce qui permet à Traefik de le découvrir et de router le trafic vers lui (grâce au montage du socket Docker côté Traefik et au provider Docker activé). Il n'y a donc pas de port exposé sur l'hôte pour ce conteneur.

```
services:
  it-tools:
      image: ghcr.io/corentinth/it-tools:latest
      container_name: it-tools
      restart: unless-stopped
      networks:
        - frontend
      labels:
        - traefik.enable=true
        - traefik.http.routers.it-tools.rule=Host(`it-tools.it-connectlab.fr`)
        - traefik.http.routers.it-tools.entrypoints=web

networks:
  frontend:
    external: true
```

Il y a aussi une section **`labels`** essentielle pour permettre la découverte par Traefik.

**Les Labels Traefik : ce qu’ils font**

- **`traefik.enable=true`** : nécessaire car, dans notre config Traefik, il y a l'instruction `**exposedByDefault: false**`. Ce label **autorise** explicitement Traefik à gérer ce service.
- **``traefik.http.routers.it-tools.rule=Host(`\it-tools.it-connectlab.fr`)``**: crée un **routeur HTTP** nommé **`it-tools`** qui ne s’active que lorsque la requête du client Web est à destination de l'hôte **`it-tools.it-connectlab.fr`** (soit un enregistrement DNS à associer).
- `**traefik.http.routers.it-tools.entrypoints=web**` : indique que ce routeur écoute sur l’**entrypPoint** intitulé `**web**` donc celui qui écoute sur le port 80.

**Petite parenthèse : il est à noter que Traefik va automatiquement associer un service à ce routeur.** Chaque service est associé à un module **load balancer Traefik** (équilibrage de charge), car c'est une fonction indispensable pour chaque reverse proxy pour distribuer la charge en plusieurs serveurs de backends. Même s'il n'y a qu'un seul serveur de backend, comme c'est le cas ici avec un seul conteneur, il y a le load balancer. Dans le cas présent, afin de compléter la configuration, nous pourrions ajouter ce label supplémentaire (c'est important sur les conteneurs où plusieurs ports sont en écoute) :

```
        - traefik.http.services.it-tools.loadbalancer.server.port=80
```

Enregistrez le fichier et lancez le déploiement de l'application :

```
docker compose up -d
```

À partir d'un navigateur Web, vous devriez pouvoir accéder à l'application :

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Publier-une-application-HTTP-avec-Traefik-800x409.png.webp)

L'application a été correctement publiée par l'intermédiaire de Traefik !

### ACME avec Traefik

Un accès en HTTP en 2025, ce n'est pas l'idéal tant le HTTPS s'est démocratisé depuis plusieurs années. Surtout, il n'est pas nécessaire de payer pour obtenir un certificat TLS valide : merci Let's Encrypt. La bonne nouvelle, c'est que **Traefik supporte deux Certificate Resolvers** : **ACME** (donc Let's Encrypt) et **[Tailscale](https://www.it-connect.fr/tuto-tailscale-reseau-vpn-mesh-base-sur-wireguard/ "Découvrez Tailscale : votre futur réseau VPN Mesh basé sur WireGuard")** pour les services internes et accessibles via cette solution.

Dans le contexte de la méthode **ACME**, Traefik prend en charge **plusieurs types de challenges** pour demander automatiquement le certificat. Il y a le **challenge HTTP** (**`httpChallenge`**) qui est surement le plus classique, mais qui implique que l'hôte soit accessible en HTTP sur le port 80. Surtout, il y a le **challenge DNS** (**`dnsChallenge`**) grâce à l'intégration de [Lego](https://go-acme.github.io/lego/), ce qui permet de jouer sur les enregistrements DNS de façon automatique en ciblant les API des gestionnaires de zones DNS.

**Par exemple** : vous disposez d'un domaine enregistré chez OVHCloud, vous pouvez effectuer des demandes de certificats Let's Encrypt via le composant ACME intégré à Traefik, le tout via l'API de OVHCloud. Autrement dit, c'est un processus pleinement automatisé. Cela est aussi vrai pour **des dizaines d'autres providers DNS compatibles Traefik**, comme Cloudflare, Azure DNS, Gandi, Hostinger ou encore Ionos (voir [cette page](https://go-acme.github.io/lego/dns/)).

Pour la suite de cet article, l'objectif sera le suivant : obtenir un certificat TLS pour le domaine **`it-tools.it-connectlab.fr`** afin de publier l'application en HTTPS. Ce domaine étant enregistré chez OVHCloud, nous devrons utiliser leur API.

### Traefik : exposer le port 443

Vous devez commencer par exposer le **`443`** au niveau du conteneur Traefik en éditant le fichier **`docker-compose.yml`** de ce projet.

```
ports:
      - "80:80"
      - "8080:8080"
      - "443:443"
```

### Créer une clé d'API OVHCloud pour Traefik

Vous devez commencer par créer une nouvelle API d'application à partir du portail OVHCloud. Pour l'Europe, il faut accéder à la page [eu.api.ovh.com/createApp/](https://eu.api.ovh.com/createApp/), tandis que pour les États-Unis et le Canada, c'est sur la page [ca.api.ovh.com/createApp/](https://ca.api.ovh.com/createApp/).

Donnez un nom à l'application et précisez une description. Par exemple :

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Traefik-API-OVHCloud-pour-ACME-1.png.webp)

En retour, vous obtenez une clé d'application et un secret associé. Vous devez garder précieusement ces deux informations.

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Traefik-API-OVHCloud-pour-ACME-2.png.webp)

Ensuite, à l'aide d'une console PowerShell (passez par PowerShell ISE ou VSCode, ce sera plus pratique), vous devez envoyer une requête sur l'API pour donner des droits à cette application. Utilisez le code ci-dessous, en adaptant le nom DNS pour cibler le bon domaine, ainsi que la valeur à la suite de **`X-Ovh-Application`** en précisant la clé d'application.

```
curl -XPOST -H "X-Ovh-Application: 3f6ed0ayzxyz" -H "Content-type: application/json" https://eu.api.ovh.com/1.0/auth/credential -d '{
  "accessRules":[
    {
      "method":"POST",
      "path":"/domain/zone/it-connectlab.fr/record"
    },
    {
      "method":"POST",
      "path":"/domain/zone/it-connectlab.fr/refresh"
    },
    {
      "method":"DELETE",
      "path":"/domain/zone/it-connectlab.fr/record/*"
    }
  ],
  "redirection":"https://www.it-connectlab.fr"
}'
```

L'exécution de cette commande retourne deux informations importantes :

- **`consumerKey`** : vous devez conserver la valeur associée à ce champ, car elle sera nécessaire pour la suite de la configuration.
- **`validationUrl`** : vous devez copier cette URL et y accéder avec votre navigateur préféré pour valider l'autorisation.

```
{"consumerKey":"cf5b9e91fffff21233464135","validationUrl":"https://www.ovh.com/auth/sso/api?credentialToken=04fa6c66011dc8498ab80814b140d6b1eea133e520612228fc3620747de0cfdc","state":"pendingValidation"}
```

L'URL à laquelle vous avez accédé précédemment affiche une demande de confirmation comme celle ci-dessous. Pensez à ajuster la durée de validité de cet accès et cliquez sur le bouton "**Authorize**".

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Traefik-API-OVHCloud-pour-ACME-3.png)

Vous serez redirigé vers la page indiquée lors de l'exécution de la commande curl. Même si le domaine ne renvoie vers rien, ce n'est pas gênant.

**Voilà, l'accès est prêt au niveau de l'API OVHCloud et de votre compte.**

### Utiliser OVHCloud avec Traefik

Puisque Traefik intègre Lego, vous devez consulter la documentation du fournisseur DNS correspondant à vos besoins pour savoir comment le configurer. Vous pouvez consulter, par exemple, [la page](https://go-acme.github.io/lego/dns/ovh/) concernant **`ovh`**. Ici, nous allons ajouter 4 variables d'environnement au fichier docker-compose.yml de Traefik :

- **`OVH_ENDPOINT`** : définit le point d’accès de l’API OVH (par exemple : **`ovh-eu`**), afin que Traefik sache vers quelle région OVH communiquer.
- **`OVH_APPLICATION_KEY`** : identifie l’application (ici Traefik) auprès de l’API OVH.
- **`OVH_APPLICATION_SECRET`** : sert à authentifier de façon sécurisée l’application auprès de l’API OVH.
- **`OVH_CONSUMER_KEY`** : autorise l’application à agir au nom de ton compte OVH, selon les droits accordés lors de la création de la clé.

Dans le fichier Docker Compose, ces variables sont introduites sans valeur directe, car nous allons utiliser un fichier externe. En même temps, dans la section **`volumes`**, rajoutez le montage du dossier **`./certs`** (soit **`/opt/docker-compose/traefik/certs/`** que vous devrez créer).

```
# docker-compose.yml
services:
  traefik:
    image: traefik:v3.6
    restart: unless-stopped
    environment:
      - OVH_ENDPOINT=${OVH_ENDPOINT}
      - OVH_APPLICATION_KEY=${OVH_APPLICATION_KEY}
      - OVH_APPLICATION_SECRET=${OVH_APPLICATION_SECRET}
      - OVH_CONSUMER_KEY=${OVH_CONSUMER_KEY}
    ports:
      - "80:80"
      - "8080:8080"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
      - ./certs:/var/traefik/certs:rw
    networks:
      - frontend
networks:
  frontend:
    external: true
```

Enregistrez et fermez ce fichier. Aux côtés du fichier Docker Compose, créez le fichier **`.env`** et associez les bonnes valeurs aux variables. Mise à part pour la première variable, pour les autres, ce sont des informations confidentielles spécifiques à votre environnement.

```
OVH_ENDPOINT=ovh-eu
OVH_APPLICATION_KEY=3f6ed77777777
OVH_APPLICATION_SECRET=f6f8d623bf73abcdefghxyzzzz
OVH_CONSUMER_KEY=cf5b9e913c5776itconnect2025
```

### Ajouter un résolveur ACME

Pour autant, la configuration ne s'arrête pas là. Ensuite, vous allez devoir ouvrir le fichier de configuration **`traefik.yml`**. Vérifiez la présence d'un entryPoint HTTPS, comme **`websecure`** dans l'exemple ci-dessous.

Dans cette configuration, nous devons définir un nouveau **résolveur de certificats ACME** pour OVHCloud utilisé par Traefik pour obtenir automatiquement des certificats SSL/TLS via **Let's Encrypt**.

Dans cet exemple, le résolveur sera simplement nommé **`ovhcloud`**, une adresse e-mail (**`[[email protected]](https://www.it-connect.fr/cdn-cgi/l/email-protection)`**) est associée au compte ACME, tandis que le fichier **`/var/traefik/certs/ovh-acme.json`** stocke les certificats et clés générés. Le paramètre **`caServer`** indique l’URL de l’API Let’s Encrypt, ici en mode production (avec une alternative en mode staging, moins limitée). Vous pouvez tout à fait utiliser le mode staging pour tester la configuration, puis basculer en mode production ensuite.

Enfin, la section **`dnsChallenge`** précise que la validation du domaine s’effectue via OVHCloud, en créant automatiquement un enregistrement DNS (demandé par Let's Encrypt au cours du processus). Le **`delayBeforeCheck`** de 10 secondes associé laisse le temps à la propagation DNS avant la vérification du certificat. Si vous indiquez **`0`**, cela améliore la réactivité mais c'est aussi une source d'erreur, ce qui est contreproductif.

```
# Configuration statique de Traefik
global:
  checkNewVersion: false
  sendAnonymousUsage: false

api:
  dashboard: true # Dashboard Traefik : 8080
  insecure: true

entryPoints:
  web:
    address: ":80" # EntryPoint pour HTTP
  websecure:
    address: ":443" # EntryPoint pour HTTPS

certificatesResolvers:
  ovhcloud:
    acme:
      email: "[email protected]"
      storage: "/var/traefik/certs/ovh-acme.json"
      caServer: "https://acme-v02.api.letsencrypt.org/directory" # Production
      #caServer: "https://acme-staging-v02.api.letsencrypt.org/directory"
      dnsChallenge:
        provider: ovh
        delayBeforeCheck: 10

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
```

Enregistrez et fermez le fichier. La configuration de Traefik est terminée ! Pensez à faire un Down/Up sur le conteneur pour recharger la configuration.

### Ajouter un routeur HTTPS à Traefik

Pour que notre application IT-Tools soit accessible en HTTPS, sa configuration doit être adaptée. Je vous rappelle que ses **`labels`** actuels s'appuient sur l'entryPoint en HTTP. Ouvrez le fichier Docker Compose de l'application, puis ajoutez quatre lignes supplémentaires dans les **`labels`** regroupées avec le nom de routeur **`it-tools-https`** :

```
services:
  it-tools:
      image: ghcr.io/corentinth/it-tools:latest
      container_name: it-tools
      restart: unless-stopped
      networks:
        - frontend
      labels:
        - traefik.enable=true
        - traefik.http.routers.it-tools.rule=Host(`it-tools.it-connectlab.fr`)
        - traefik.http.routers.it-tools.entrypoints=web
        - traefik.http.routers.it-tools-https.rule=Host(`it-tools.it-connectlab.fr`)
        - traefik.http.routers.it-tools-https.entrypoints=websecure
        - traefik.http.routers.it-tools-https.tls=true
        - traefik.http.routers.it-tools-https.tls.certresolver=ovhcloud
networks:
  frontend:
    external: true
```

Ces directives configurent le **routeur HTTPS** de Traefik pour l’application `**it-tools**`. L’entrée **`entrypoints=websecure`** indique que ce routeur utilise le point d’entrée HTTPS. Le paramètre **`tls=true`** active le chiffrement TLS pour sécuriser les connexions. Enfin, **`tls.certresolver=ovhcloud`** précise que les certificats TLS/SSL doivent être obtenus automatiquement via le résolveur ACME nommé **`ovhcloud`** (nom précisé dans **`traefik.yml`**).

Arrêtez ce projet et relancez-le (Down/Up).

Il ne reste plus qu'à tester l'accès en HTTPS à l'application : **`https://it-tools.it-connectlab.fr`**. Si vous êtes en mode STAGING sur Let's Encrypt, il y aura bien un certificat mais il n'est pas valide (ce qui est normal). Cela permet de valider le bon fonctionnement de la configuration.

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Traefik-HTTPS-Lets-Encrypt-Staging-800x452.png.webp)

Suite au passage en mode production, un certificat TLS valide 90 jours est obtenu via Let's Encrypt. La connexion à l'application s'effectue en **HTTPS avec le reverse proxy Traefik**.

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Traefik-HTTPS-Lets-Encrypt-OK-800x386.png.webp)

### Traefik : redirection HTTP vers HTTPS

Pour finaliser la configuration, il me semble judicieux de configurer la redirection HTTP vers HTTPS. De ce fait, les applications publiées seront accessibles uniquement au travers d'une connexion sécurisée. Les requêtes HTTP seront redirigées, plutôt que de finir dans le vide...

À ce sujet, il y a [une technique évoquée dans la documentation](https://doc.traefik.io/traefik-hub/api-gateway/expose/middleware/redirect) de Traefik et que nous pouvons appliquer. Au sein du fichier traefik.yml, nous allons éviter l'entryPoint **`web`** pour ajouter une redirection en HTTPS vers l'entryPoint **`websecure`**.

```
entryPoints:
  web:
    address: ":80" # EntryPoint pour HTTP
    http:
      redirections:
        entryPoint:
          to: websecure
          scheme: https    
  websecure:
    address: ":443" # EntryPoint pour HTTPS
```

Ce qui correspond à l'ajout de ces lignes :

![](https://www.it-connect.fr/wp-content-itc/uploads/2025/11/Traefik-Redirection-HTTP-vers-HTTPS.png)

Vous n'avez plus qu'à relancer le conteneur Traefik et à tester !

## Les journaux de debug de Traefik

Dans le cas où la configuration ne fonctionne pas, vous devez aller faire un tour dans **les journaux de Traefik**. Par défaut, seules les erreurs sont retournées, ce qui peut être trop limite pour du **troubleshooting Traefik**.

Ouvrez le fichier traefik.yml et ajoutez ces lignes pour ajuster le niveau de logs. Ici, nous configurons le mode **`DEBUG`** qui est le plus verbeux.

```
log:
  level: DEBUG # Niveau de log
```

Ensuite, arrêtez et relancez le conteneur. Attention, les journaux ne sont pas persistants, donc le redémarrage implique un effacement des journaux.

Consultez les journaux du conteneur Traefik, vous devriez voir de nombreuses informations passer dans la console. Si tout se passe bien, il y aura quand même des informations dans les logs, c'est aussi tout l'intérêt du mode debug.

```
/opt/docker-compose/traefik$ docker compose logs -f --tail 100
```

Je vous encourage aussi à vérifier :

- La syntaxe de vos fichiers de configuration : attention au copier-coller, au formatage et aux erreurs de frappe.
- La configuration des entryPoints et des ports exposés au niveau du conteneur Traefik.
- Les permissions sur les répertoires associés aux données persistantes de Traefik.

## Conclusion

Grâce à ce **guide d'introduction à Traefik**, vous devriez être capable de mettre en place un reverse proxy dans des environnements basés sur Docker. Vous disposez des bases nécessaires pour bien comprendre le fonctionnement de Traefik, afin de l'adapter à vos besoins et votre contexte.

Pour aller plus loin, consultez la [documentation officielle de Traefik](https://doc.traefik.io/traefik/) et notamment [cette page](https://doc.traefik.io/traefik/reference/routing-configuration/other-providers/docker/) qui regroupe tous les labels Docker. Aimeriez-vous d'autres tutoriels sur Traefik ? N'hésitez pas à partager vos suggestions en commentaire.

## FAQ

#### Qu’est-ce que Traefik et à quoi sert-il ?

Traefik est un reverse proxy et un load balancer qui simplifie la gestion du trafic réseau vers des applications, conteneurs ou services, tout en automatisant la configuration grâce à l’intégration avec Docker, Kubernetes, ou d’autres orchestrateurs. Il a été créé par Émile Vauge, un Français.

#### Comment fonctionne l'auto-discovery des services dans Traefik ?

Traefik se connecte à un "provider" (tel que Docker ou Kubernetes) et détecte les conteneurs ou services exposés. Il crée automatiquement les routes et règles associées selon les labels configurés. La découverte automatique peut être désactivée pour éviter que des services inappropriés soient exposés. Le provider agit comme une source de configuration dynamique.

#### Comment Traefik gère-t-il les certificats SSL/TLS ?

Traefik prend en charge **Let’s Encrypt** et plus particulièrement l'ACME. Ainsi, il peut demander, renouveler et gérer automatiquement les certificats SSL/TLS pour les domaines associés à vos applications, garantissant un accès HTTPS avec un certificat valide et sans configuration complexe.

#### Traefik peut-il gérer plusieurs domaines ou sous-domaines ?

Oui. Chaque domaine ou sous-domaine peut être routé vers un service différent grâce aux règles (comme `**Host()**`) dans la configuration ou via des labels Docker.

#### À quoi sert le tableau de bord de Traefik ?

Le tableau de bord (Dashboard) de Traefik permet de visualiser en temps réel la configuration active : routes, middlewares, services, certificats et métriques. Il aide au diagnostic et à la supervision du trafic. Mais, il ne permet pas de faire la configuration en ligne : c'est uniquement de la lecture seule.

#### Comment fonctionnent les middlewares dans Traefik ?

Les middlewares servent à transformer les requêtes : ajout d’en-têtes, authentification avant de pouvoir accéder à une application, filtrage applicatif (WAF), etc. On peut dire qu'ils ajoutent des capacités supplémentaires à Traefik.

#### Quels sont les journaux (logs) générés par Traefik ?

Traefik produit deux types de logs : les **access logs** (requêtes HTTP) et les **logs applicatifs** (erreurs, warnings, événements internes). Comme l'explique cet article, il y a notamment un paramètre permettant d'ajuster le niveau de log.

#### Comment monitorer Traefik ?

Traefik expose des métriques au format OpenTelemetry et dans des formats adaptés pour des solutions comme Prometheus et InfluxDB2. Il y a également un tableau de bord prêt à l'emploi pour Grafana. Un bloc de configuration `metrics:` permet d'ajuster la configuration des métriques exposées et leur accès.

![author avatar](https://www.it-connect.fr/wp-content-itc/uploads/2025/07/Avatar-FB.jpg.webp)

Ingénieur système et réseau, cofondateur d'IT-Connect et Microsoft MVP "Cloud and Datacenter Management". Je souhaite partager mon expérience et mes découvertes au travers de mes articles. Généraliste avec une attirance particulière pour les solutions Microsoft et le scripting. Bonne lecture.