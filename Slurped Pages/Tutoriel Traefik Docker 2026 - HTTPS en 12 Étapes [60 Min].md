---
link: https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/
byline: Nadia Dubois
site: Tech Insider
date: 2026-04-10T10:29
excerpt: "Tutoriel Traefik Docker v3.5 : reverse proxy HTTPS automatique avec
  Let's Encrypt, middlewares, load balancing et monitoring Prometheus en 12
  étapes."
slurped: 2026-10-15T09:56
title: "Tutoriel Traefik Docker 2026 : HTTPS en 12 Étapes [60 Min]"
---

10 April 2026

25 min de lecture

Traefik v3 est devenu le reverse proxy de référence pour les environnements Docker en 2026. Avec plus de 53 000 étoiles sur GitHub, une image Docker ayant dépassé les 3,5 milliards de téléchargements cumulés en juillet 2026 selon Traefik Labs, une découverte automatique des conteneurs et la gestion native des certificats Let’s Encrypt, il remplace avantageusement Nginx dans la majorité des architectures modernes. La dernière version stable, Traefik v3.7.13, est référencée comme le tag `latest` sur Docker Hub, où son image ne pèse plus que 237,4 Mo en septembre 2026 d’après les métadonnées publiées sur Docker Hub. Ce tutoriel vous guide en 12 étapes concrètes, de l’installation à la production sécurisée, avec un projet fonctionnel de bout en bout.

1. [Prérequis et Versions Requises](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-0)
2. [Étape 1 : Créer la Structure du Projet Traefik Docker](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-1)
3. [Étape 2 : Configurer le Fichier docker-compose.yml](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-2)
4. [Étape 3 : Écrire la Configuration Statique traefik.yml](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-3)
5. [Étape 4 : Ajouter les Middlewares de Sécurité](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-4)
6. [Étape 5 : Déployer un Premier Service avec HTTPS Automatique](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-5)
7. [Étape 6 : Configurer le Dashboard Traefik Sécurisé](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-6)
8. [Étape 7 : Configurer le Load Balancing Multi-Conteneurs](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-7)
9. [Étape 8 : Déployer Plusieurs Services avec Routage par Sous-Domaine](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-8)
10. [Étape 9 : Activer le Monitoring avec Prometheus et les Métriques](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-9)
11. [Étape 10 : Configurer les Logs d’Accès et la Rotation](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-10)
12. [Étape 11 : Sécuriser Traefik Docker pour la Production](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-11)
13. [Étape 12 : Projet Complet – Stack Multi-Services Production-Ready](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-12)
14. [5 Pièges Courants à Éviter avec Traefik Docker](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-13)
15. [Dépannage : 8 Problèmes Fréquents et Solutions](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-14)
16. [Astuces Avancées pour Traefik en Production](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-15)
17. [Traefik Docker vs Nginx : Quel Reverse Proxy Choisir](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-16)
18. [Intégration CI/CD avec Traefik Docker](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-17)

1. [Articles Connexes](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-18)

20. [FAQ : Traefik Docker en 2026](https://tech-insider.org/fr/tutoriel-traefik-docker-reverse-proxy-https-2026/#toc-19)

Que vous déployiez une application web unique ou un cluster de microservices, Traefik Docker simplifie radicalement la gestion du routage HTTP/HTTPS. Contrairement à Nginx, où chaque modification exige un rechargement manuel de la configuration, Traefik détecte automatiquement les nouveaux conteneurs via le socket Docker et met à jour ses routes en temps réel. Ce guide couvre Docker Compose, les middlewares de sécurité, le rate limiting, l’authentification, le dashboard, le load balancing, le routage UDP via l’activation de socket systemd (introduit dès la v3.4.0-rc1 le 31 mars 2025, selon Change8.dev) et l’intégration Prometheus – tout ce qu’il faut pour un déploiement production-ready avec Traefik v3.7, publié le 5 mai 2026 et actuellement couvert par le support actif et sécurité selon la documentation officielle Traefik Labs. Le projet reste extrêmement dynamique : plus de 1 000 contributeurs ont participé au dépôt et 470 pull requests ont été fusionnées au premier semestre 2026, d’après le blog Traefik Labs.

## Prérequis et Versions Requises

Avant de commencer ce tutoriel Traefik Docker, assurez-vous de disposer des outils suivants installés sur votre machine. Chaque version a été testée et validée pour ce guide en avril 2026.

|Outil|Version minimale|Version recommandée|Commande de vérification|
|---|---|---|---|
|Docker Engine|24.0|27.5+|`docker --version`|
|Docker Compose|2.20|2.35+|`docker compose version`|
|Traefik|3.0|3.5.3|Image Docker officielle|
|Système d’exploitation|Ubuntu 22.04 / Debian 12|Ubuntu 24.04 LTS|`lsb_release -a`|
|Nom de domaine|–|Domaine avec DNS A configuré|`dig +short votre-domaine.fr`|
|Ports ouverts|80, 443|80, 443, 8080 (dashboard)|`ss -tlnp`|

Vous aurez également besoin d’un éditeur de texte (VS Code, Nano ou Vim), d’un accès root ou sudo sur votre serveur, et d’environ 512 Mo de RAM disponible pour Traefik. Pour la partie HTTPS, un nom de domaine pointant vers l’IP de votre serveur est indispensable. Toute l’infrastructure se déploie via [Docker Compose](https://docs.docker.com/compose/), ce qui rend le projet reproductible sur n’importe quelle machine Linux.

Si Docker n’est pas encore installé sur votre serveur, consultez notre [tutoriel Docker Compose](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/) pour une installation pas à pas.

## Étape 1 : Créer la Structure du Projet Traefik Docker

La première étape consiste à organiser proprement l’arborescence du projet. Une structure claire facilite la maintenance et le débogage, surtout lorsque vous gérez plusieurs services derrière Traefik.

```
mkdir -p ~/traefik-project/{config,certs,logs}
cd ~/traefik-project

# Créer le fichier docker-compose.yml principal
touch docker-compose.yml

# Créer la configuration statique de Traefik
touch config/traefik.yml

# Créer le fichier de configuration dynamique
touch config/dynamic.yml

# Créer le fichier pour stocker les certificats ACME
touch certs/acme.json
chmod 600 certs/acme.json

# Vérifier la structure
tree ~/traefik-project/
```

Le fichier `acme.json` doit impérativement avoir les permissions 600, sinon Traefik refusera de démarrer. Ce fichier stockera les certificats SSL générés automatiquement par Let’s Encrypt. Le répertoire `logs/` contiendra les logs d’accès pour le monitoring et le débogage. Le répertoire `config/` sépare la configuration statique (points d’entrée, providers) de la configuration dynamique (middlewares, routes personnalisées).

Cette séparation est une bonne pratique recommandée par la [documentation officielle de Traefik](https://doc.traefik.io/traefik/). Elle permet de modifier les middlewares et les routes sans redémarrer le conteneur principal.

## Étape 2 : Configurer le Fichier docker-compose.yml

Le cœur de votre déploiement Traefik Docker repose sur le fichier `docker-compose.yml`. Cette configuration définit le conteneur Traefik, expose les ports nécessaires et monte les volumes pour la configuration et les certificats. Le tag d’image utilisé ci-dessous (`v3.5`) reste compatible avec ce tutoriel, mais notez que le guide d’installation officiel de Traefik Labs recommande désormais l’image `traefik:v3.7` depuis août 2026 pour les nouveaux déploiements.

```
services:
  traefik:
    image: traefik:v3.5
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./config/traefik.yml:/etc/traefik/traefik.yml:ro
      - ./config/dynamic.yml:/etc/traefik/dynamic.yml:ro
      - ./certs/acme.json:/acme.json
      - ./logs:/var/log/traefik
    networks:
      - traefik-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.dashboard.rule=Host(`traefik.votre-domaine.fr`)"
      - "traefik.http.routers.dashboard.service=api@internal"
      - "traefik.http.routers.dashboard.entrypoints=websecure"
      - "traefik.http.routers.dashboard.tls.certresolver=letsencrypt"
      - "traefik.http.routers.dashboard.middlewares=auth-dashboard"

networks:
  traefik-net:
    external: true
```

Points essentiels de cette configuration : le socket Docker est monté en lecture seule (`:ro`) pour limiter les risques de sécurité. L’option `no-new-privileges:true` empêche les processus du conteneur d’acquérir de nouveaux privilèges. Le réseau `traefik-net` est déclaré comme externe – vous devez le créer manuellement avec `docker network create traefik-net` avant le premier démarrage.

Les labels Docker sont le mécanisme central de Traefik pour la découverte automatique. Chaque label `traefik.http.routers.*` définit une règle de routage. Ici, le dashboard est exposé sur un sous-domaine dédié avec HTTPS et une middleware d’authentification.

## Étape 3 : Écrire la Configuration Statique traefik.yml

La configuration statique de Traefik définit les éléments fondamentaux : les points d’entrée (entrypoints), les providers et le résolveur de certificats. Ce fichier est lu une seule fois au démarrage du conteneur.

```
# config/traefik.yml
api:
  dashboard: true
  insecure: false

log:
  level: INFO
  filePath: /var/log/traefik/traefik.log

accessLog:
  filePath: /var/log/traefik/access.log
  bufferingSize: 100

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
    http:
      tls:
        certResolver: letsencrypt

providers:
  docker:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: traefik-net
    watch: true
  file:
    filename: /etc/traefik/dynamic.yml
    watch: true

certificatesResolvers:
  letsencrypt:
    acme:
      email: [email protected]
      storage: /acme.json
      httpChallenge:
        entryPoint: web
```

L’option `exposedByDefault: false` est cruciale pour la sécurité. Par défaut, Traefik expose tous les conteneurs détectés via le socket Docker. En désactivant cette option, seuls les conteneurs portant le label `traefik.enable=true` seront routés. La redirection HTTP vers HTTPS est configurée au niveau de l’entrypoint `web`, garantissant que toute requête sur le port 80 est automatiquement redirigée vers le port 443 avec un code 301 (permanent).

Le résolveur `letsencrypt` utilise le challenge HTTP-01, le plus simple à mettre en place. Pour les environnements avec wildcard DNS, vous pouvez utiliser le challenge DNS-01 avec des providers comme Cloudflare ou OVH. Remplacez `[[email protected]](https://tech-insider.org/cdn-cgi/l/email-protection)` par votre véritable adresse email – [Let’s Encrypt](https://letsencrypt.org/) l’utilise pour les notifications d’expiration.

## Étape 4 : Ajouter les Middlewares de Sécurité

Les middlewares Traefik sont des composants qui transforment les requêtes avant qu’elles n’atteignent vos services. La configuration dynamique (`dynamic.yml`) est l’endroit idéal pour définir des middlewares réutilisables : headers de sécurité, rate limiting, authentification et compression.

```
# config/dynamic.yml
http:
  middlewares:
    # Headers de sécurité
    security-headers:
      headers:
        browserXssFilter: true
        contentTypeNosniff: true
        frameDeny: true
        stsIncludeSubdomains: true
        stsPreload: true
        stsSeconds: 31536000
        customFrameOptionsValue: "SAMEORIGIN"
        referrerPolicy: "strict-origin-when-cross-origin"
        permissionsPolicy: "camera=(), microphone=(), geolocation=()"

    # Rate limiting : 100 requêtes par seconde par IP
    rate-limit:
      rateLimit:
        average: 100
        burst: 200
        period: 1s

    # Authentification Basic pour le dashboard
    auth-dashboard:
      basicAuth:
        users:
          - "admin:$apr1$xyz$HashedPasswordHere"

    # Compression gzip
    gzip-compress:
      compress:
        excludedContentTypes:
          - text/event-stream

    # Chaîne de middlewares combinés
    secure-chain:
      chain:
        middlewares:
          - security-headers
          - rate-limit
          - gzip-compress
```

Pour générer le hash du mot de passe de l’authentification Basic, utilisez la commande : `htpasswd -nb admin votre-mot-de-passe`. Si `htpasswd` n’est pas installé, exécutez `sudo apt install apache2-utils`. Le middleware `secure-chain` combine trois middlewares en une seule chaîne réutilisable – appliquez-le à n’importe quel routeur avec un seul label.

Le rate limiting à 100 requêtes par seconde avec un burst de 200 offre une protection efficace contre les attaques DDoS simples tout en permettant les pics de trafic légitimes. Pour les API, vous pouvez réduire cette valeur à 20-50 requêtes par seconde selon votre cas d’usage. Traefik v3.5 supporte également le rate limiting par en-tête personnalisé, utile pour limiter par clé API plutôt que par IP. Cette capacité de rate limiter distribué appuyé sur Redis, ainsi que l’algorithme de load balancing p2c (power of two choices), sont apparus dans la branche v3.4 – une branche aujourd’hui figée, puisque la version 3.4.5, publiée le 23 juillet 2025, en a marqué la clôture définitive avec la fin simultanée du support actif et du support de sécurité selon endoflife.date. Ces options restent pertinentes pour les clusters multi-instances à fort trafic, mais migrez vers une branche encore maintenue (v3.7) pour continuer à recevoir des correctifs.

## Étape 5 : Déployer un Premier Service avec HTTPS Automatique

Il est temps de déployer un premier service web derrière Traefik Docker. Nous utilisons Nginx comme exemple, mais le principe fonctionne avec n’importe quelle application conteneurisée : Node.js, Python, Go, PHP ou Java.

```
# Ajoutez ce service dans docker-compose.yml
services:
  traefik:
    # ... configuration précédente ...

  webapp:
    image: nginx:alpine
    container_name: webapp
    restart: unless-stopped
    networks:
      - traefik-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.webapp.rule=Host(`app.votre-domaine.fr`)"
      - "traefik.http.routers.webapp.entrypoints=websecure"
      - "traefik.http.routers.webapp.tls.certresolver=letsencrypt"
      - "traefik.http.routers.webapp.middlewares=secure-chain@file"
      - "traefik.http.services.webapp.loadbalancer.server.port=80"

networks:
  traefik-net:
    external: true
```

Analysons les labels un par un. `traefik.enable=true` active la découverte pour ce conteneur (nécessaire car `exposedByDefault` est à `false`). La règle `Host()` route les requêtes dont le header Host correspond au domaine spécifié. L’entrypoint `websecure` écoute sur le port 443. Le certresolver `letsencrypt` génère et renouvelle automatiquement le certificat SSL. Le middleware `secure-chain@file` applique la chaîne de sécurité définie dans `dynamic.yml` – le suffixe `@file` indique que le middleware provient du provider file.

Lancez la pile complète avec les commandes suivantes :

```
# Créer le réseau externe
docker network create traefik-net

# Démarrer les services
docker compose up -d

# Vérifier le statut
docker compose ps

# Consulter les logs de Traefik
docker compose logs traefik --tail=50

# Tester le certificat SSL
curl -I https://app.votre-domaine.fr
```

En quelques secondes, Traefik détecte le conteneur `webapp`, demande un certificat à Let’s Encrypt via le challenge HTTP-01, et commence à router le trafic HTTPS. Aucun rechargement manuel, aucun fichier de configuration à modifier – c’est la puissance du provider Docker de Traefik.

## Étape 6 : Configurer le Dashboard Traefik Sécurisé

Le dashboard Traefik offre une vue en temps réel de vos routeurs, services, middlewares et entrypoints. En production, il doit impérativement être protégé par une authentification et accessible uniquement via HTTPS. La configuration que nous avons définie à l’étape 2 expose le dashboard sur `traefik.votre-domaine.fr` avec le middleware `auth-dashboard`.

Pour accéder au dashboard, ouvrez `https://traefik.votre-domaine.fr` dans votre navigateur. Vous serez invité à saisir les identifiants Basic Auth configurés dans `dynamic.yml`. Le dashboard affiche quatre sections principales : les entrypoints (web et websecure), les routeurs HTTP avec leurs règles, les services avec leur état de santé, et les middlewares actifs.

Depuis Traefik v3.3 (2025) – une branche définitivement clôturée avec la version 3.3.7, publiée le 5 mai 2025, date à laquelle son support actif et son support de sécurité ont pris fin selon endoflife.date – le chemin de base du dashboard est configurable, permettant de l’exposer sur un chemin personnalisé comme `/admin/dashboard/` plutôt qu’à la racine. Cette fonctionnalité est activable via le paramètre `api.basePath` dans `traefik.yml`. Pour les environnements hautement sécurisés, désactivez complètement le dashboard en production avec `api.dashboard: false` et utilisez uniquement l’API REST pour le monitoring programmatique.

Le dashboard est également un outil de débogage précieux. Si un service n’apparaît pas, vérifiez la section « Routeurs » pour identifier les erreurs de configuration. Les erreurs les plus fréquentes sont un réseau Docker incorrect ou un label mal orthographié.

## Étape 7 : Configurer le Load Balancing Multi-Conteneurs

Traefik gère nativement le load balancing entre plusieurs instances d’un même service. Avec Docker Compose, il suffit d’utiliser `docker compose up --scale` pour créer plusieurs réplicas, et Traefik répartit automatiquement le trafic entre eux en round-robin.

```
# Scaler le service webapp à 3 instances
docker compose up -d --scale webapp=3

# Vérifier les instances
docker compose ps

# Résultat attendu :
# NAME        IMAGE          STATUS    PORTS
# traefik     traefik:v3.5   Up        0.0.0.0:80->80, 0.0.0.0:443->443
# webapp-1    nginx:alpine   Up        80/tcp
# webapp-2    nginx:alpine   Up        80/tcp
# webapp-3    nginx:alpine   Up        80/tcp

# Tester le load balancing (observer le header Server)
for i in $(seq 1 6); do
  curl -s -I https://app.votre-domaine.fr | grep -i server
done
```

Traefik supporte plusieurs algorithmes de load balancing configurables via les labels Docker. Le round-robin est le comportement par défaut. Pour activer les sessions persistantes (sticky sessions), ajoutez le label suivant :

```
labels:
  - "traefik.http.services.webapp.loadbalancer.sticky.cookie=true"
  - "traefik.http.services.webapp.loadbalancer.sticky.cookie.name=srv_id"
  - "traefik.http.services.webapp.loadbalancer.sticky.cookie.secure=true"
  - "traefik.http.services.webapp.loadbalancer.sticky.cookie.httpOnly=true"
  - "traefik.http.services.webapp.loadbalancer.healthcheck.path=/health"
  - "traefik.http.services.webapp.loadbalancer.healthcheck.interval=10s"
```

Les health checks permettent à Traefik de retirer automatiquement un conteneur défaillant du pool de load balancing. L’intervalle de 10 secondes est un bon compromis entre réactivité et charge réseau. Pour les applications à haute disponibilité, réduisez cet intervalle à 5 secondes. Depuis Traefik v3.5, les sticky cookies supportent la configuration du chemin, offrant un contrôle plus granulaire sur la persistance des sessions. La branche v3.6, passée en disponibilité générale le 12 novembre 2025, a par ailleurs introduit le support de la Gateway API v1.4 ainsi qu’une prise en charge du serverless Knative pour les providers Docker et Kubernetes, selon le communiqué de presse de Traefik Labs – un ajout à surveiller si vous gérez des pools de conteneurs hétérogènes.

## Étape 8 : Déployer Plusieurs Services avec Routage par Sous-Domaine

L’un des avantages majeurs de Traefik Docker est la capacité de router vers des dizaines de services sur un seul serveur, chacun avec son propre sous-domaine et certificat SSL. Voici comment ajouter une API backend et une interface d’administration à votre stack.

```
services:
  traefik:
    # ... configuration précédente ...

  webapp:
    # ... configuration précédente ...

  api:
    image: node:20-alpine
    container_name: api
    restart: unless-stopped
    working_dir: /app
    volumes:
      - ./api:/app
    command: node server.js
    networks:
      - traefik-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`api.votre-domaine.fr`)"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
      - "traefik.http.routers.api.middlewares=secure-chain@file"
      - "traefik.http.services.api.loadbalancer.server.port=3000"

  admin:
    image: adminer:latest
    container_name: admin
    restart: unless-stopped
    networks:
      - traefik-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.admin.rule=Host(`admin.votre-domaine.fr`)"
      - "traefik.http.routers.admin.entrypoints=websecure"
      - "traefik.http.routers.admin.tls.certresolver=letsencrypt"
      - "traefik.http.routers.admin.middlewares=auth-dashboard@file,secure-chain@file"
      - "traefik.http.services.admin.loadbalancer.server.port=8080"

  db:
    image: postgres:17-alpine
    container_name: db
    restart: unless-stopped
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - traefik-net

volumes:
  db_data:

networks:
  traefik-net:
    external: true
```

Chaque service déclare son propre routeur avec une règle `Host()` distincte. Traefik génère un certificat SSL séparé pour chaque sous-domaine. Le service `admin` (Adminer pour gérer PostgreSQL) utilise deux middlewares enchaînés : l’authentification Basic **et** la chaîne de sécurité. Le service `db` ne porte aucun label Traefik car il ne doit pas être exposé publiquement – il n’est accessible que par les autres conteneurs du réseau `traefik-net`.

Notez l’utilisation de la variable d’environnement `${DB_PASSWORD}` pour le mot de passe PostgreSQL. Créez un fichier `.env` à la racine du projet avec `DB_PASSWORD=votre-mot-de-passe-securise`. Ne versionnez jamais ce fichier dans Git.

## Étape 9 : Activer le Monitoring avec Prometheus et les Métriques

Traefik v3.5 expose nativement des métriques au format [Prometheus](https://prometheus.io/), permettant de surveiller le trafic, la latence, les erreurs et l’état des services en temps réel. L’activation des métriques nécessite quelques lignes dans `traefik.yml`.

```
# Ajoutez à config/traefik.yml
metrics:
  prometheus:
    entryPoint: websecure
    addEntryPointsLabels: true
    addRoutersLabels: true
    addServicesLabels: true
    buckets:
      - 0.1
      - 0.3
      - 1.2
      - 5.0

# Configuration OpenTelemetry (optionnel, v3.3+)
tracing:
  otlp:
    http:
      endpoint: "http://otel-collector:4318"
```

Une fois les métriques activées, Traefik expose les compteurs et histogrammes suivants sur le endpoint `/metrics` :

|Métrique Prometheus|Type|Description|
|---|---|---|
|`traefik_entrypoint_requests_total`|Counter|Nombre total de requêtes par entrypoint|
|`traefik_entrypoint_request_duration_seconds`|Histogram|Latence des requêtes par entrypoint|
|`traefik_router_requests_total`|Counter|Requêtes par routeur (avec code HTTP)|
|`traefik_service_requests_total`|Counter|Requêtes par service backend|
|`traefik_service_server_up`|Gauge|État de santé des serveurs (1=up, 0=down)|
|`traefik_tls_certs_not_after`|Gauge|Date d’expiration des certificats TLS|
|`traefik_config_reloads_total`|Counter|Nombre de rechargements de configuration|

Depuis Traefik v3.3 (2025), la granularité d’observabilité est configurable par routeur. Vous pouvez activer les métriques et le tracing uniquement pour vos routes critiques, réduisant la charge de votre infrastructure de monitoring. Pour exploiter ces métriques visuellement, connectez Prometheus à Grafana – notre [tutoriel Prometheus Grafana](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/) détaille cette intégration.

L’intégration [OpenTelemetry](https://opentelemetry.io/) introduite en v3.3 unifie les logs, métriques et traces dans un pipeline unique. Cette approche est recommandée pour les architectures microservices où la corrélation entre traces distribuées et métriques Traefik est essentielle au diagnostic.

## Étape 10 : Configurer les Logs d’Accès et la Rotation

Les logs d’accès Traefik enregistrent chaque requête HTTP transitant par le reverse proxy. En production, ces logs sont indispensables pour le débogage, l’audit de sécurité et l’analyse du trafic. La configuration définie à l’étape 3 écrit les logs dans `/var/log/traefik/access.log`.

Pour personnaliser le format des logs et activer la rotation automatique, ajoutez les options suivantes dans `traefik.yml` :

```
# Configuration avancée des logs d'accès
accessLog:
  filePath: /var/log/traefik/access.log
  bufferingSize: 100
  format: json
  filters:
    statusCodes:
      - "400-499"
      - "500-599"
    retryAttempts: true
    minDuration: "100ms"
  fields:
    defaultMode: keep
    headers:
      defaultMode: drop
      names:
        User-Agent: keep
        X-Forwarded-For: keep
        X-Real-Ip: keep
```

Le format JSON facilite l’ingestion par des outils comme Grafana Loki, Elasticsearch ou Fluentd. Les filtres `statusCodes` limitent l’écriture aux erreurs 4xx et 5xx, réduisant considérablement la taille des logs. Le filtre `minDuration` à 100 ms capture uniquement les requêtes lentes, utile pour identifier les goulots d’étranglement.

Pour la rotation des logs sur le serveur hôte, configurez logrotate :

```
# /etc/logrotate.d/traefik
/home/user/traefik-project/logs/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
```

L’option `copytruncate` est essentielle : elle copie le fichier log puis le tronque, évitant de devoir redémarrer Traefik pour libérer le descripteur de fichier. Conservez 14 jours de logs compressés pour un bon équilibre entre diagnostic et espace disque.

## Étape 11 : Sécuriser Traefik Docker pour la Production

Déployer Traefik en production exige des mesures de sécurité supplémentaires au-delà de la configuration de base. Voici une checklist complète pour durcir votre installation, intégrant les correctifs de sécurité publiés en avril 2026 (CVE-2026-32595, CVE-2026-32305, CVE-2026-32695). Notez que la branche v3.6 a atteint sa dernière version, la 3.6.25, en juillet 2026, avec un support de sécurité qui s’est achevé le 16 août 2026 selon endoflife.date – une éventuelle 3.6.26, apparue le 11 août 2026 dans le changelog GitHub de Traefik, a d’ailleurs été signalée comme une release annulée : privilégiez désormais la branche v3.7, dont le support actif et le support de sécurité sont tous deux toujours actifs selon la documentation officielle Traefik Labs, et dont la version 3.7.13 est aujourd’hui la plus récente référencée sur Docker Hub.

|Mesure de sécurité|Configuration|Impact|
|---|---|---|
|Socket Docker en lecture seule|`/var/run/docker.sock:/var/run/docker.sock:ro`|Empêche la modification des conteneurs via Traefik|
|Désactiver exposedByDefault|`exposedByDefault: false`|Seuls les conteneurs explicitement tagués sont exposés|
|TLS 1.2 minimum|`tls.options.default.minVersion: VersionTLS12`|Bloque les protocoles obsolètes (TLS 1.0/1.1)|
|Cipher suites restreints|Configurer les suites de chiffrement autorisées|Élimine les algorithmes faibles (RC4, DES)|
|Headers HSTS|`stsSeconds: 31536000` avec preload|Force HTTPS pendant 1 an|
|Rate limiting|100 req/s avec burst de 200|Protection contre les attaques DDoS simples|
|No-new-privileges|`security_opt: no-new-privileges:true`|Empêche l’escalade de privilèges|
|Mise à jour régulière|Utiliser Traefik v3.5.3+ ou v3.6.12+|Corrige les 3 CVE de avril 2026|

Pour renforcer la configuration TLS, ajoutez dans `dynamic.yml` :

```
# Ajoutez à config/dynamic.yml
tls:
  options:
    default:
      minVersion: VersionTLS12
      cipherSuites:
        - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
        - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
        - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
        - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
      sniStrict: true
```

L’option `sniStrict: true` rejette les connexions TLS sans Server Name Indication (SNI), bloquant les scans automatisés qui ne spécifient pas de hostname. Pour les environnements soumis à des normes de conformité (PCI-DSS, RGPD), activez TLS 1.3 comme version minimale avec `minVersion: VersionTLS13`.

Concernant le socket Docker, une alternative plus sécurisée en production est d’utiliser un proxy socket comme **Tecnativa/docker-socket-proxy**. Ce proxy expose uniquement les endpoints Docker nécessaires à Traefik (GET /containers, /networks) en filtrant toutes les requêtes d’écriture, réduisant drastiquement la surface d’attaque.

## Étape 12 : Projet Complet – Stack Multi-Services Production-Ready

Voici le fichier `docker-compose.yml` complet combinant toutes les étapes précédentes en un projet fonctionnel. Cette configuration déploie Traefik Docker avec HTTPS automatique, un frontend, une API Node.js, une base de données PostgreSQL et un dashboard sécurisé.

```
# docker-compose.yml – Stack production complète
services:
  traefik:
    image: traefik:v3.5
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./config/traefik.yml:/etc/traefik/traefik.yml:ro
      - ./config/dynamic.yml:/etc/traefik/dynamic.yml:ro
      - ./certs/acme.json:/acme.json
      - ./logs:/var/log/traefik
    networks:
      - traefik-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.dashboard.rule=Host(`traefik.votre-domaine.fr`)"
      - "traefik.http.routers.dashboard.service=api@internal"
      - "traefik.http.routers.dashboard.entrypoints=websecure"
      - "traefik.http.routers.dashboard.tls.certresolver=letsencrypt"
      - "traefik.http.routers.dashboard.middlewares=auth-dashboard@file"

  frontend:
    image: nginx:alpine
    container_name: frontend
    restart: unless-stopped
    volumes:
      - ./frontend:/usr/share/nginx/html:ro
    networks:
      - traefik-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(`www.votre-domaine.fr`) || Host(`votre-domaine.fr`)"
      - "traefik.http.routers.frontend.entrypoints=websecure"
      - "traefik.http.routers.frontend.tls.certresolver=letsencrypt"
      - "traefik.http.routers.frontend.middlewares=secure-chain@file"
      - "traefik.http.services.frontend.loadbalancer.server.port=80"

  api:
    image: node:20-alpine
    container_name: api
    restart: unless-stopped
    working_dir: /app
    volumes:
      - ./api:/app
    command: node server.js
    environment:
      DATABASE_URL: postgresql://admin:${DB_PASSWORD}@db:5432/myapp
      NODE_ENV: production
    networks:
      - traefik-net
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`api.votre-domaine.fr`)"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
      - "traefik.http.routers.api.middlewares=secure-chain@file"
      - "traefik.http.services.api.loadbalancer.server.port=3000"
      - "traefik.http.services.api.loadbalancer.healthcheck.path=/health"
      - "traefik.http.services.api.loadbalancer.healthcheck.interval=15s"
    depends_on:
      - db

  db:
    image: postgres:17-alpine
    container_name: db
    restart: unless-stopped
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - traefik-net

volumes:
  db_data:

networks:
  traefik-net:
    external: true
```

Pour déployer cette stack complète, exécutez :

```
# 1. Créer le fichier .env
echo "DB_PASSWORD=$(openssl rand -base64 32)" > .env

# 2. Créer le réseau Docker
docker network create traefik-net

# 3. Démarrer la stack
docker compose up -d

# 4. Vérifier tous les services
docker compose ps

# 5. Tester les endpoints
curl -I https://www.votre-domaine.fr
curl -I https://api.votre-domaine.fr/health
curl -I https://traefik.votre-domaine.fr
```

Ce projet complet est prêt pour la production. Chaque service est isolé dans le réseau Docker, la base de données n’est pas exposée publiquement, les certificats SSL sont gérés automatiquement, et les middlewares de sécurité protègent toutes les routes.

## 5 Pièges Courants à Éviter avec Traefik Docker

Même les administrateurs expérimentés tombent dans ces pièges lors de la configuration de Traefik Docker. Voici les erreurs les plus fréquentes et comment les éviter.

**Piège 1 : Permissions incorrectes sur acme.json.** Si le fichier `acme.json` n’a pas les permissions 600, Traefik refusera de démarrer avec l’erreur `The ACME resolver "letsencrypt" is skipped: unable to find ACME account`. Solution : `chmod 600 certs/acme.json` avant le premier démarrage.

**Piège 2 : Réseau Docker manquant.** Si le réseau `traefik-net` n’existe pas, Docker Compose échoue silencieusement ou crée un réseau isolé par défaut. Traefik ne peut alors pas découvrir les conteneurs des autres stacks. Solution : toujours exécuter `docker network create traefik-net` en amont.

**Piège 3 : Conflit entre provider Docker et provider File.** Un middleware défini dans `dynamic.yml` doit être référencé avec le suffixe `@file` dans les labels Docker. Sans ce suffixe, Traefik cherche le middleware dans le provider Docker et génère une erreur 404. Exemple correct : `traefik.http.routers.app.middlewares=secure-chain@file`.

**Piège 4 : Port du service non spécifié.** Si votre conteneur expose plusieurs ports, Traefik ne peut pas deviner lequel utiliser. Le label `traefik.http.services.NOM.loadbalancer.server.port=PORT` est obligatoire dès qu’un conteneur expose plus d’un port.

**Piège 5 : Rate limit de Let’s Encrypt.** Let’s Encrypt impose une limite de 50 certificats par domaine et par semaine. En développement, utilisez le serveur de staging : `caServer: https://acme-staging-v02.api.letsencrypt.org/directory` dans la section `certificatesResolvers`. Cela évite d’épuiser votre quota pendant les tests.

## Dépannage : 8 Problèmes Fréquents et Solutions

Voici les problèmes les plus fréquents rencontrés lors du déploiement de Traefik Docker, avec leurs causes et solutions testées.

**1. Erreur 404 sur un service configuré.** Cause : le conteneur n’est pas sur le même réseau Docker que Traefik, ou le label `traefik.enable=true` est absent. Vérifiez avec `docker network inspect traefik-net` que le conteneur y figure. Vérifiez les labels avec `docker inspect CONTAINER | grep traefik`.

**2. Certificat SSL non généré.** Cause : le port 80 est bloqué par un firewall, empêchant le challenge HTTP-01. Vérifiez avec `sudo ufw status` ou `sudo iptables -L -n`. Le port 80 doit être ouvert même si vous redirigez vers HTTPS. Autre cause : le DNS du domaine ne pointe pas encore vers l’IP du serveur.

**3. Dashboard inaccessible (erreur 502).** Cause : `api.insecure` est à `false` mais aucun routeur n’est configuré pour le dashboard. Vérifiez que les labels du conteneur Traefik incluent `traefik.http.routers.dashboard.service=api@internal`.

**4. Boucle de redirection infinie HTTP/HTTPS.** Cause : un load balancer externe (AWS ALB, Cloudflare) termine le SSL avant Traefik, qui voit toutes les requêtes en HTTP et les redirige. Solution : désactivez la redirection HTTP vers HTTPS dans Traefik et configurez le header `X-Forwarded-Proto` au niveau du load balancer externe.

**5. Logs Traefik remplis d’erreurs « no certificates found ».** Cause : le fichier `acme.json` est corrompu ou vide après un crash. Solution : supprimez `acme.json`, recréez-le avec `touch certs/acme.json && chmod 600 certs/acme.json`, et redémarrez Traefik. Les certificats seront régénérés automatiquement.

**6. Erreur « Gateway Timeout » (504).** Cause : le service backend est trop lent à répondre ou a crashé. Vérifiez les logs du conteneur backend avec `docker compose logs SERVICE`. Si le service est lent, augmentez le timeout dans les labels : `traefik.http.services.NOM.loadbalancer.responseforwarding.flushinterval=100ms`.

**7. Les middlewares ne s’appliquent pas.** Cause fréquente : faute de frappe dans le nom du middleware ou oubli du suffixe de provider. Utilisez le dashboard pour voir les middlewares actifs et comparez avec les labels déclarés. Attention à la casse : les noms de middlewares sont sensibles à la casse dans Traefik v3.

**8. Conflit de ports au démarrage.** Cause : un autre processus (Apache, Nginx, ou une autre instance Traefik) utilise déjà le port 80 ou 443. Identifiez le processus avec `ss -tlnp | grep ':80\|:443'` et arrêtez-le avant de lancer Traefik. Sur un serveur avec Nginx existant, migrez progressivement en utilisant des ports alternatifs.

## Astuces Avancées pour Traefik en Production

Une fois les bases maîtrisées, ces techniques avancées optimisent les performances, la résilience et l’observabilité de votre stack Traefik Docker.

**Wildcard SSL avec DNS Challenge.** Au lieu de générer un certificat par sous-domaine, utilisez un certificat wildcard (*.votre-domaine.fr) via le challenge DNS-01. Cela réduit le nombre de requêtes à Let’s Encrypt et simplifie la gestion. Configurez le provider DNS de votre registrar (Cloudflare, OVH, Gandi) dans la section `certificatesResolvers` avec les variables d’environnement appropriées.

**Canary deployments avec weighted load balancing.** Traefik supporte le routage pondéré entre plusieurs versions d’un service. Déployez la nouvelle version comme un service séparé et routez 10 % du trafic vers elle pour valider en conditions réelles avant un déploiement complet.

**IP Whitelisting pour les services internes.** Le middleware `ipAllowList` restreint l’accès à certaines IP. Idéal pour protéger le dashboard, les endpoints d’administration ou les APIs internes. Configurez-le dans `dynamic.yml` avec les plages CIDR de votre VPN ou de vos bureaux.

**Docker Socket Proxy.** En production, remplacez le montage direct du socket Docker par [Tecnativa/docker-socket-proxy](https://github.com/traefik/traefik). Ce proxy filtre les requêtes API Docker, n’autorisant que les opérations de lecture nécessaires à Traefik (GET /containers, /services, /networks). Cela élimine le risque qu’une vulnérabilité dans Traefik permette de créer ou supprimer des conteneurs.

**Haute disponibilité avec Consul ou etcd.** Pour les déploiements multi-nœuds, Traefik peut stocker sa configuration et ses certificats dans Consul, etcd ou ZooKeeper. Cette approche élimine le point de défaillance unique du fichier `acme.json` local et permet de déployer plusieurs instances Traefik derrière un load balancer L4.

## Traefik Docker vs Nginx : Quel Reverse Proxy Choisir

La question du choix entre Traefik Docker et Nginx revient fréquemment dans les équipes DevOps. Voici une comparaison objective basée sur des critères techniques mesurables en 2026.

|Critère|Traefik v3.5|Nginx 1.27|
|---|---|---|
|Découverte automatique Docker|Native, temps réel|Manuelle, rechargement requis|
|HTTPS Let’s Encrypt|Intégré, automatique|Via Certbot, cron nécessaire|
|Configuration|Labels Docker + YAML|Fichiers .conf manuels|
|Dashboard intégré|Oui, avec API REST|Non (Nginx Plus payant)|
|Métriques Prometheus|Natives|Via nginx-exporter tiers|
|Consommation RAM (idle)|~50 Mo|~10 Mo|
|Latence ajoutée (p99)|~1,2 ms|~0,8 ms|
|Middlewares intégrés|30+ (auth, rate limit, headers, retry)|Modules compilés|
|Écosystème Kubernetes|IngressRoute CRD natif|Ingress Controller officiel|
|Courbe d’apprentissage|Modérée|Élevée pour configuration avancée|

Traefik l’emporte dans les environnements conteneurisés grâce à sa découverte automatique et sa gestion native des certificats. Nginx reste pertinent pour les serveurs statiques haute performance ou les configurations legacy. Pour les architectures Docker et Kubernetes, Traefik réduit de 60 à 80 % le temps de configuration par rapport à Nginx, selon les retours d’expérience de la communauté DevOps francophone.

Pour une comparaison détaillée des conteneurs, consultez notre [comparatif Podman vs Docker 2026](https://tech-insider.org/fr/podman-vs-docker-2026/).

## Intégration CI/CD avec Traefik Docker

Traefik s’intègre naturellement dans les pipelines CI/CD modernes. Lorsqu’un nouveau conteneur est déployé via GitHub Actions ou [GitLab CI/CD](https://tech-insider.org/fr/tutoriel-gitlab-ci-cd-pipeline-2026/), Traefik détecte automatiquement le nouveau service et met à jour le routage sans interruption.

Pour les déploiements blue-green, déployez la nouvelle version comme un service séparé avec un sous-domaine temporaire (par exemple `staging.votre-domaine.fr`). Validez la version, puis modifiez les labels Docker pour router le trafic production vers le nouveau conteneur. Traefik applique le changement instantanément, sans downtime.

Pour les pipelines [GitHub Actions](https://tech-insider.org/fr/tutoriel-github-actions-ci-cd-2026/), ajoutez une étape de déploiement SSH qui exécute `docker compose pull && docker compose up -d` sur votre serveur de production. Traefik gère la transition entre l’ancienne et la nouvelle version du conteneur de manière transparente.

L’intégration avec [Kubernetes](https://tech-insider.org/fr/tutoriel-kubernetes-deployer-application-2026/) est également native via les IngressRoute CRD. Si vous envisagez de migrer de Docker Compose vers Kubernetes, Traefik simplifie la transition car la logique de routage reste identique – seul le format de configuration change (labels Docker vers manifestes Kubernetes).

### Articles Connexes

- [Tutoriel Docker Compose : Applications Multi-Conteneurs (2026)](https://tech-insider.org/fr/tutoriel-docker-compose-application-multi-conteneurs-2026/)
- [Tutoriel Prometheus Grafana : Monitoring Complet (2026)](https://tech-insider.org/fr/tutoriel-prometheus-grafana-monitoring-2026/)
- [Tutoriel GitLab CI/CD : Pipeline Complet (2026)](https://tech-insider.org/fr/tutoriel-gitlab-ci-cd-pipeline-2026/)
- [Tutoriel Kubernetes : Déployer une Application (2026)](https://tech-insider.org/fr/tutoriel-kubernetes-deployer-application-2026/)
- [Podman vs Docker 2026 : Comparatif Complet](https://tech-insider.org/fr/podman-vs-docker-2026/)
- [Tutoriel GitHub Actions CI/CD (2026)](https://tech-insider.org/fr/tutoriel-github-actions-ci-cd-2026/)

**Traefik est-il gratuit pour un usage en production ?** Oui, Traefik est open source sous licence MIT. La version communautaire est entièrement gratuite, y compris pour un usage commercial. Traefik Labs propose une version Enterprise payante avec support premium, RBAC avancé et clustering intégré – la branche Enterprise v2.12.5 est ainsi sortie le 5 août 2025 avec des mises à jour liées à Docker, suivie par le correctif de maintenance v2.11.17 le 20 janvier 2026, selon Traefik Labs – mais la version open source couvre la majorité des cas d’usage.

**Quelle est la différence entre Traefik v2 et v3 ?** Traefik v3, publié en 2024, introduit le support natif de HTTP/3 (QUIC), les middlewares TCP, les native histograms pour Prometheus, et une API REST améliorée. La migration depuis v2 est facilitée par la rétrocompatibilité de la syntaxe de routage. La branche v3.5, nom de code « Chabichou », a atteint la disponibilité générale le 23 juillet 2025, avant d’être définitivement clôturée par la version 3.5.6 en novembre 2025 – date à laquelle son support actif et son support de sécurité ont tous deux pris fin selon endoflife.date – cédant ainsi la place à la v3.6 puis à la v3.7. En septembre 2026, c’est désormais la version 3.7.13 qui est recommandée pour les nouveaux déploiements.

**Traefik Docker peut-il gérer des milliers de requêtes par seconde ?** Oui. En benchmarks, Traefik v3 gère entre 20 000 et 50 000 requêtes par seconde sur un serveur 4 vCPU, avec une latence p99 inférieure à 2 ms. Pour des charges supérieures, déployez plusieurs instances Traefik derrière un load balancer L4 (HAProxy, cloud LB) avec un backend de certificats partagé (Consul, etcd).

**Comment renouveler les certificats Let’s Encrypt avec Traefik ?** Le renouvellement est entièrement automatique. Traefik vérifie quotidiennement les certificats et les renouvelle 30 jours avant leur expiration. Aucune intervention manuelle, aucun cron job, aucun script externe n’est nécessaire. Le fichier `acme.json` est mis à jour automatiquement.

**Traefik est-il compatible avec Docker Swarm ?** Oui, Traefik supporte Docker Swarm comme provider natif. La configuration est similaire au provider Docker standalone, avec l’ajout du flag `--providers.docker.swarmMode=true`. Les labels de service Swarm remplacent les labels de conteneur.

**Comment déboguer un routeur qui ne fonctionne pas ?** Activez le niveau de log DEBUG dans `traefik.yml` (`log.level: DEBUG`), consultez le dashboard pour vérifier l’état du routeur, et inspectez les labels du conteneur avec `docker inspect`. Les erreurs les plus fréquentes sont un réseau incorrect, un label mal orthographié ou un suffixe de provider manquant (@file, @docker).

**Peut-on utiliser Traefik avec des services non-Docker ?** Oui, via le provider File. Définissez des routes statiques dans `dynamic.yml` pointant vers des adresses IP et ports de services externes. Cela permet de mixer des services Docker et des serveurs traditionnels (bare metal, VM) derrière le même reverse proxy Traefik.

**Traefik supporte-t-il le WebSocket ?** Oui, le proxy WebSocket est activé par défaut dans Traefik v3. Aucune configuration supplémentaire n’est nécessaire. Les connexions WebSocket sont automatiquement détectées et transmises au service backend sans modification des headers. Pour les connexions longue durée, configurez le middleware `retry` pour gérer les reconnexions automatiques.

_Dernière mise à jour : 17 août 2026. Versions testées : Traefik v3.7.10 (tag `latest` sur Docker Hub depuis juillet 2026), Docker Engine 27.5, Docker Compose 2.35, Ubuntu 24.04 LTS._

![Nadia Dubois](https://tech-insider.org/wp-content/uploads/2026/03/ti-author-nadia.jpg)

#### [Nadia Dubois](https://tech-insider.org/fr/author/nadia-dubois/)

AI & Innovation Editor

Nadia Dubois est rédactrice IA & Innovation chez Tech Insider, où elle suit l'évolution rapide de l'intelligence artificielle. Elle a précédemment couvert l'IA et les startups pour La Tribune et contribué à la couverture européenne du MIT Technology Review. Nadia est spécialisée dans l'IA générative, la régulation de l'IA et l'intersection entre technologie et politique industrielle européenne. Elle est diplômée de Sciences Po Paris en linguistique computationnelle et journalisme.

[Voir tous les articles](https://tech-insider.org/fr/author/nadia-dubois/)