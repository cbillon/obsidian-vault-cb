---
link: https://www.it-connect.fr/traefik-log-dashboard-monitorer-et-analyser-les-logs-traefik-avec-une-interface-dediee/
byline: Florian BURNEL
site: IT-Connect
date: 2026-01-13T17:00
excerpt: La solution open source Traefik Log Dashboard permet de visualiser les
  journaux d'accès du reverse proxy Traefik directement au travers d'une
  interface web.
twitter: https://twitter.com/@itconnect_fr
tags:
  - slurp/reverse-proxy
  - slurp/serveur-web
slurped: 2026-04-08T17:36
title: "Traefik Log Dashboard : un œil sur les logs de Traefik"
---

**Traefik est un reverse proxy populaire particulièrement apprécié pour les environnements conteneurisés, mais l'analyse brute de ses journaux (logs) reste une tâche ardue sans un outillage adapté. Le projet open source Traefik Log Dashboard permet d'avoir une meilleure visibilité.**

Le tableau de bord de Traefik offre un bon aperçu sur la configuration actuelle du reverse proxy, en particulier sur les routeurs, services et middlewares. Néanmoins, elle ne permet pas de visualiser les journaux d'accès de Traefik, c'est-à-dire les tentatives de connexion des utilisateurs sur les applications publiées.

Pourtant, les journaux de Traefik contiennent des informations utiles et permettent d'en savoir plus sur l'activité des applications web : codes HTTP, erreurs rencontrées, trafic suspect, pays d'origine des clients qui se connectent, etc.

**[Traefik Log Dashboard](https://github.com/hhftechnology/traefik-log-dashboard)** développé par hhftechnology répond à ce besoin précis : offrir une visualisation graphique et globale sur les logs d'accès, sans avoir à déployer une stack plus lourde. Grâce à un système d'agents, cet outil open source peut présenter les journaux d'une instance Traefik unique ou de plusieurs instances Traefik.

**Ce tutoriel explique comment installer Traefik Log Dashboard avec Docker. Ce tableau de bord regroupe différentes métriques :**

- Une vue d'ensemble avec des statistiques clés (nombre total de requêtes, temps de réponse, taux de succès, nombre de services actifs),
- L'état du trafic selon les routes et les services définis dans Traefik,
- Les adresses IP et les user-agent utilisés par les clients qui se connectent à vos applications,
- Les hôtes les plus sollicitées (répartition par nom de domaine),
- Une vue d'ensemble sur l'emplacement géographique des clients,
- L'état global du système,
- Les journaux bruts de Traefik en temps réel, mis en forme sur une page Web.

Ci-dessous, quelques copies d'écran du tableau de bord de Traefik Log Dashboard.

![](https://www.it-connect.fr/wp-content-itc/uploads/2026/01/Traefik-Log-Dashboard-1.jpg.webp)

![](https://www.it-connect.fr/wp-content-itc/uploads/2026/01/Traefik-Log-Dashboard-3.jpg.webp)

![](https://www.it-connect.fr/wp-content-itc/uploads/2026/01/Traefik-Log-Dashboard-2.jpg.webp)

D'autres fonctionnalités sont disponibles :

- Gestion des agents, avec possibilité de basculer d'un agent à l'autre depuis le tableau de bord
- Gestion en ligne de commande avec une CLI,
- Création de filtres pour épurer les logs avec les informations inutiles (certaines adresses IP, par exemple), avec la possibilité d'avoir plusieurs conditions,
- Création d'alertes en déclenchant un appel sur un Webhook externe (Discord, Telegram, etc.),
- Configuration de l'historique des données et sélection d'une période plus ou moins importante pour afficher les données.

Sommaire

- [Prérequis](https://www.it-connect.fr/traefik-log-dashboard-monitorer-et-analyser-les-logs-traefik-avec-une-interface-dediee/#Prerequis)
- [Configuration des logs d'accès dans Traefik](https://www.it-connect.fr/traefik-log-dashboard-monitorer-et-analyser-les-logs-traefik-avec-une-interface-dediee/#Configuration_des_logs_dacces_dans_Traefik)
- [Déploiement de Traefik Log Dashboard](https://www.it-connect.fr/traefik-log-dashboard-monitorer-et-analyser-les-logs-traefik-avec-une-interface-dediee/#Deploiement_de_Traefik_Log_Dashboard)
- [Première connexion à Traefik Log Dashboard](https://www.it-connect.fr/traefik-log-dashboard-monitorer-et-analyser-les-logs-traefik-avec-une-interface-dediee/#Premiere_connexion_a_Traefik_Log_Dashboard)
- [Conclusion](https://www.it-connect.fr/traefik-log-dashboard-monitorer-et-analyser-les-logs-traefik-avec-une-interface-dediee/#Conclusion)

## Prérequis

Avant de nous lancer dans la configuration, il est important de comprendre comment les composants vont interagir. **Traefik Log Dashboard** ne s'intercale pas dans le trafic : il fonctionne en "lecture seule" sur le fichier de log généré par Traefik.

Pour suivre ce tutoriel, vous devez disposer de :

- Un serveur avec **Docker** et **Docker Compose** installés : [comment installer Docker sur Debian](https://www.it-connect.fr/installation-pas-a-pas-de-docker-sur-debian-11/)
- Une instance de **Traefik** déjà fonctionnelle : [débuter avec Traefik](https://www.it-connect.fr/tuto-traefik-reverse-proxy-avec-docker/ "Débuter avec Traefik : un reverse proxy moderne pour vos services Web Docker")

L'architecture est simple : Traefik écrit ses logs d'accès dans un fichier `**.log**` sur l'hôte Docker (via un volume pour assurer la persistance des données). Le conteneur "agent" de Traefik Log Dashboard monte ce même fichier en lecture pour l'analyser en temps réel et présente les métriques via une interface Web.

## Configuration des logs d'accès dans Traefik

Par défaut, Traefik n'écrit pas toujours les logs d'accès, ou il le fait dans un format moins adapté au parsing. L'idéal, comme je l'avais expliqué dans le [tutoriel Traefik x CrowdSec](https://www.it-connect.fr/reverse-proxy-traefik-integration-de-crowdsec-pour-bloquer-les-attaques/ "Reverse Proxy Traefik : intégration de CrowdSec pour bloquer les attaques"), c'est de configurer Traefik pour qu'il exporte les logs au format **JSON**. Ce qui suit est facultatif si vous avez déjà configuré les logs sur votre reverse proxy Traefik.

Ainsi, le fichier de configuration `**traefik.yml**` doit être édité pour configurer la section **`accessLog`**. La documentation de Traefik Log Dashboard suggère la configuration suivante :

```
# traefik.yml
accessLog:
  filePath: "/logs/access.log"
  format: json
  bufferingSize: 100

  fields:
    defaultMode: keep
    names:
      ClientUsername: drop
    headers:
      defaultMode: keep
      names:
        Authorization: drop
        Cookie: drop
```

En réalité, vous pouvez adopter la configuration qui vous semble la plus adaptée, du moment que le format JSON est utilisé et que vous conservez un minimum d'informations pour permettre l'analyse des journaux.

Dans le cadre de l'intégration de CrowdSec avec Traefik, j'utilise cette configuration (car CrowdSec doit aussi lire les logs de Traefik pour détecter les comportements suspects).

```
accessLog:
  filePath: "/logs/traefik.log"
  format: json
  filters:
    statusCodes:
      - "200-299"
      - "400-599"
  fields:
    headers:
      defaultMode: drop # drop all headers per default
      names:
        User-Agent: keep # log user agent strings
        X-Real-Ip: keep
        X-Forwarded-For: keep
        X-Forwarded-Proto: keep
```

Il est impératif que ce fichier soit persistant et accessible depuis l'hôte, ce qui va le rendre accessible par les deux conteneurs. Vous devez donc déclarer un volume dans la définition de votre service Traefik. Le fichier Docker Compose de Traefik contient ceci :

```
    volumes:
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
      - ./config.yml:/etc/traefik/config.yml:ro
      - ./certs:/var/traefik/certs:rw
      - ./logs:/logs:rw
```

Une fois la modification effectuée, redémarrez Traefik pour prendre en compte les modifications.

## Déploiement de Traefik Log Dashboard

Nous allons maintenant déployer l'outil avec Docker Compose. Commençons par créer le dossier `/opt/docker-compose/traefik-dashboard` qui sera la racine de notre projet, puis à l'intérieur le dossier `data` avec trois sous-dossiers (`dashboard`, `logs`, `positions`).

Désormais, nous allons créer un fichier `**docker-compose.yml**` pour préparer le déploiement de Traefik Log Dashboard. Il y aura deux conteneurs :

- **`traefik-log-dashboard-agent`** : l'agent Traefik Log Dashboard à déployer sur chaque serveur Traefik (ici un seul, donc en local)
- **`traefik-log-dashboard`** : le serveur principal, sur lequel il conviendra de se connecter en web.

Avant d'ouvrir le fichier de configuration, lancez la commande ci-dessous pour générer un jeton. La valeur retournée servira à l'authentification entre le serveur et l'agent, soit entre les deux conteneurs.

```
openssl rand -hex 32

# Exemple de résultat : 
c3c576623a85d80c61d9848046d8821399d7204843d06cbeeafae77feebcc1d7
```

Ouvrez le fichier Docker Compose et collez le contenu précisé ci-dessous. Spécifiez le jeton au niveau des directives **`TRAEFIK_LOG_DASHBOARD_AUTH_TOKEN`** et **`AGENT_API_TOKEN`**. Pour le mode multi-agents, vous devez utiliser un jeton unique par agent (voir [cette documentation](https://traefik-log-dashboard.hhf.technology/docs/quickstart#multi-agent-setup)). Un agent configuré de façon statique dans le fichier Docker Compose ne pourra pas être supprimé.

```
services:
  traefik-agent:
    image: hhftechnology/traefik-log-dashboard-agent:latest
    container_name: traefik-log-dashboard-agent
    restart: unless-stopped
    volumes:
      - /opt/docker-compose/traefik/logs:/logs:ro
      - ./data/positions:/data
    environment:
      - TRAEFIK_LOG_DASHBOARD_ACCESS_PATH=/logs/traefik.log
      - TRAEFIK_LOG_DASHBOARD_ERROR_PATH=/logs/traefik.log
      - TRAEFIK_LOG_DASHBOARD_AUTH_TOKEN=c3c576623a85d80c61d9848046d8821399d7204843d06cbeeafae77feebcc1d7
      - TRAEFIK_LOG_DASHBOARD_SYSTEM_MONITORING=true
      - TRAEFIK_LOG_DASHBOARD_LOG_FORMAT=json
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:5000/api/logs/status"]
      interval: 2m
      timeout: 10s
      retries: 3
      start_period: 30s
    networks:
      - frontend

  traefik-dashboard:
    image: hhftechnology/traefik-log-dashboard:latest
    container_name: traefik-log-dashboard
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - ./data/dashboard:/app/data
      - ./data/positions:/data
    environment:
      # Agent Configuration - REPLACE WITH YOUR TOKEN
      - AGENT_API_URL=http://traefik-agent:5000
      - AGENT_API_TOKEN=c3c576623a85d80c61d9848046d8821399d7204843d06cbeeafae77feebcc1d7
      - AGENT_NAME=Default Agent
      
      # Node Environment
      - NODE_ENV=production
      - PORT=3001
      
      # Display Configuration
      - NEXT_PUBLIC_SHOW_DEMO_PAGE=false
      - NEXT_PUBLIC_MAX_LOGS_DISPLAY=500

      # Security (CORS)
      - ALLOWED_ORIGINS=https://web-logs.it-connectlab.fr

    depends_on:
      traefik-agent:
        condition: service_healthy
    networks:
      - frontend

networks:
  frontend:
    external: true
```

Quelques explications pour vous aider à interpréter et adapter cette configuration :

- **`/opt/docker-compose/traefik/logs:/logs:ro`** : cette ligne dans le conteneur de l'agent sert à donner accès en lecture seule aux journaux de Traefik. La lecture des logs Traefik s'effectue par chaque agent déployé.
- **`TRAEFIK_LOG_DASHBOARD_ACCESS_PATH=/logs/traefik.log`** : cette ligne indique le nom et le chemin du fichier de log Traefik. Cela implique qu'il y ait un fichier nommé **`traefik.log`** dans **`/opt/docker-compose/traefik/logs`**.
- **`TRAEFIK_LOG_DASHBOARD_ERROR_PATH=/logs/traefik.log`** : la même chose pour les logs d'erreur.
- **`ALLOWED_ORIGINS`** : correspond à CORS, vous devez mettre l'adresse "publique" d'accès à votre tableau de bord (facultatif).
- **`ports: - "3001:3001"`** : le tableau de bord sera accessible sur le port 3001. C'est facultatif si vous publiez l'application via Traefik. Dans la documentation, le port par défaut est 3 000, mais je l'ai changé car il est déjà utilisé par mon hôte.
- Les conteneurs seront connectés au réseau **`frontend`**, sur lequel se situe déjà mon conteneur Traefik.
- **`user: "1000:1000"`** : à ajouter si vous rencontrez des problèmes de permissions

Vous pouvez publier le tableau de bord avec les journaux, mais attention, il est accessible sans authentification. Dans ce cas, protégez l'interface par une page de login, avec Authentik, [Tinyauth](https://www.it-connect.fr/tinyauth-traefik-ajoutez-un-portail-authentification-a-vos-applications-web/ "Tinyauth + Traefik : ajoutez un portail d’authentification à vos applications web") ou une autre solution (le Basic Auth est aussi possible). Voici à titre d'information la configuration des **`labels`** que j'ai mis en place dans le Docker Compose de Traefik Log Dashboard.

```
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=frontend"
      - "traefik.http.routers.web-logs-https.rule=Host(`web-logs.it-connectlab.fr`)"
      - "traefik.http.routers.web-logs-https.entrypoints=websecure"
      - "traefik.http.routers.web-logs-https.tls=true"
      - "traefik.http.routers.web-logs-https.tls.certresolver=ovhcloud"
      - "traefik.http.services.web-logs-https.loadbalancer.server.port=3001"    
      - "traefik.http.routers.web-logs-https.middlewares=crowdsec@file,tinyauth"
      - "tinyauth.apps.web-logs.config.domain=web-logs.it-connectlab.fr"
```

Enregistrez et fermez le fichier.

Démarrez le service :

```
docker compose up -d
```

## Première connexion à Traefik Log Dashboard

Une fois le conteneur lancé, accédez à votre instance par son URL, soit l'adresse IP ou le nom de domaine selon la configuration. J'ai testé l'accès via **`http://<IP hôte Docker>:3001`** et publié sur **`https://web-logs.it-connectlab.fr`**, les deux sont opérationnels.

Pour accéder au tableau de bord, cliquez sur le bouton "**Launch Dashboard**" en haut à droite.

![](https://www.it-connect.fr/wp-content-itc/uploads/2026/01/Connexion-a-Traefik-Log-Dashboard-800x475.jpg.webp)

Vous arrivez ensuite sur l'interface de la solution Traefilk Log Dashboard. L'utilisation est relativement simple puisque vous n'avez qu'à naviguer dans les différentes sections. La distribution géographique du trafic est visible en cliquant sur l'onglet "**Geography**".

Ce qui est intéressant, ce sont les temps de réponse moyens par poucentiles (Response Time Percentiles) pour avoir des statistiques globales sur la latence des services. On peut y observer un **temps de réponse moyen** (Average) et les indicateurs **P95** et **P99**.

![](https://www.it-connect.fr/wp-content-itc/uploads/2026/01/Traefik-Log-Dashboard-Geo-IP.jpg.webp)

La section "Logs" met en évidence les dernières requêtes effectuées par les clients. Vous avez l'adresse IP du client, la date et l'heure, la méthode HTTP, le chemin ciblé, et le code HTTP obtenu en retour. Les erreurs sont mises en évidence directement dans l'en-tête, ce qui permet de la visualiser directement.

![](https://www.it-connect.fr/wp-content-itc/uploads/2026/01/Traefik-Log-Dashboard-Les-erreurs-dacces-800x461.jpg.webp)

> **Note** : ce projet semble assez léger, car il consomme uniquement 100 Mo de RAM sur mon serveur Docker.

## Conclusion

**Traefik Log Dashboard est une solution pertinente pour avoir une meilleure visibilité sur les logs et l'activité du reverse proxy Traefik.** Cet outil comble un vide auquel ne répond pas le tableau de bord natif de Traefik et évite d'avoir une solution externe. Il peut aussi être utilisé de façon ponctuelle en tant qu'outil de diagnostic pour tout administrateur utilisant Traefik.

Pour aller plus loin, vous pourriez coupler Traefik avec **Prometheus** et **Grafana** pour aller plus loin dans le monitoring de votre reverse proxy. Enfin, voici le lien vers la documentation de Traefik Log Dashboard :

- [Documentation - Traefik Log Dashboard](https://traefik-log-dashboard.hhf.technology/docs)

