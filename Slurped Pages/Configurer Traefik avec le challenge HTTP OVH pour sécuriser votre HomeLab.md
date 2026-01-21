---
link: https://networkpulse.fr/configurer-traefik-avec-le-challenge-http-ovh-pour-securiser-votre-homelab/
byline: Guillaume REYNAUD
site: Network Pulse
date: 2025-12-07T20:09
excerpt: "J’ai décidé de me lancer dans un petit projet sympa : monter une machine dédiée à mon Homelab avec Traefik pour exposer mes applications sur le Web."
slurped: 2026-01-21T16:56
title: Configurer Traefik avec le challenge HTTP OVH pour sécuriser votre HomeLab
tags:
  - traefik
  - homelab
---

## **Traefik V3 — Concepts clés**

Traefik est un reverse proxy et un load balancer , utilisé pour gérer dynamiquement la manière dont les requêtes entrantes sont dirigées vers tes applications/services.

Dans Traefik v3, comme dans les versions précédentes, quatre concepts fondamentaux organisent ce fonctionnement : _Routers, Services, Middlewares, Entrypoint, Providers_.

### **Routers**

Le router (ou "routeur") est responsable de faire correspondre une requête entrante à une règle, et décider quoi faire avec cette requête.

**Que fait un router ?**

- Il écoute sur des entrées réseau (par exemple HTTP ou HTTPS)
- Il applique une règle de matching (Host(), Path(), Headers(), etc.)
- Il décide quel service appeler après

Exemple simple de router :

![Le router (ou "routeur") est responsable de faire correspondre une requête entrante à une règle, et décider quoi faire avec cette requête.](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-1a.webp)

Explication rapide :

- Si quelqu'un accède à http://example.com, ce router prend la main
- Ensuite, il délègue la requête au service my-service

### **Services**

Le service représente ton application ou backend réel, c’est là que Traefik va finalement envoyer la requête après qu'elle a été routée.

**Que fait un service ?**

- Il décrit où Traefik doit envoyer le trafic
- Il peut être un seul serveur, ou un ensemble de serveurs pour faire du load balancing

Exemple de service simple :

![Le service représente ton application ou backend réel, c’est là que Traefik va finalement envoyer la requête après qu'elle a été routée.](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-1b.webp)

Explication rapide :

- Traefik redirige ici vers http://localhost:8080
- Tu peux mettre plusieurs serveurs sous loadBalancer pour répartir les charges

### **Middlewares**

Les middlewares sont des outils de traitement intermédiaire, ils modifient la requête avant qu’elle n’arrive au service, ou la réponse avant qu’elle ne reparte vers le client.

**Que fait un middleware ?**

- Authentification, redirections, réécritures d'URL, ajout de headers, ratelimit, etc
- Les middlewares sont chaînables : tu peux en appliquer plusieurs sur un router

Exemple d'usage d'un middleware :

![Les middlewares sont des outils de traitement intermédiaire, ils modifient la requête avant qu’elle n’arrive au service, ou la réponse avant qu’elle ne reparte vers le client.](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-1c.webp)

Explication rapide :

- Avant de router la requête vers my-service, Traefik applique my-redirect
- Ici, my-redirect force toutes les requêtes HTTP vers HTTPS

**Résumé visuel**

![Voici un résumé visuel d'une requête pour accéder à une application via Traefik.](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-1d.webp)

- Router : À quelles requêtes je dois répondre ?
- Middlewares : Dois-je modifier quelque chose en cours de route ?
- Service : Où est-ce que j’envoie la requête ?

### **Qu’est-ce qu’un entrypoint ?**

Un entrypoint est un port ouvert (et une configuration associée) que Traefik écoute.

_En gros : Entrypoint = "Traefik écoute sur ce port, en utilisant tel protocole."_

**Pourquoi c’est utile ?**

Parce que tu peux avoir :

- Un port pour HTTP (ex : 80)
- Un autre pour HTTPS (ex : 443)
- Un pour gRPC, TCP pur, WebSocket, etc

Et tu peux configurer ces entrypoints avec :

- TLS activé ou non
- Certificats
- Timeout, ProxyProtocol, etc

Exemple d’entrypoints (dans traefik.yml) :

![Un entrypoint est un port ouvert (et une configuration associée) que Traefik écoute.](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-1e.webp)

Ici :

- web écoute sur le port 80 (HTTP)
- websecure écoute sur 443 (HTTPS avec TLS)

### **Qu’est-ce qu’un provider ?**

La notion de provider dans Traefik v3 (comme dans les versions précédentes) est centrale : c’est ce qui permet à Traefik de découvrir dynamiquement la configuration des services, middlewares, routes, etc.

Un provider dans Traefik est une source de configuration dynamique. Chaque provider "alimente" Traefik avec des infos sur :

- Les services à router (par ex. les conteneurs Docker)
- Les middlewares à appliquer
- Les routers (les règles de routage)
- Les certificats, options TLS, etc

les types de providers les plus fréquents :

- **docker:**  Découvre automatiquement les conteneurs Docker via leurs labels
- **file:** Charge une configuration dynamique depuis un fichier YAML ou TOML
- **kubernetesCRD:** Utilise des Custom Resources sur Kubernetes (CRDs) pour définir les routes
- **kubernetesIngress:** Utilise les objets Ingress standards dans Kubernetes
- **consulCatalog, etcd, marathon:** Intègrent des services depuis d’autres orchestrateurs

**Règle à retenir**

Chaque objet dynamique (router, middleware, service, TLS option…) est lié à un provider. Si tu veux utiliser un objet défini ailleurs (par ex. middleware défini dans un fichier YAML depuis un router Docker), tu dois spécifier @file ou @docker selon le cas.

## **Configuration initiale de Traefik**

J’utilise Debian 12 sur mon homelab. Généralement je mets toute la configuration des mes containers dans le répertoire _/opt/containers_.

- **Etape 1 - Création du répertoire traefik** 

```
mkdir traefik
```

![Création du répertoire traefik](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-2.webp)

- **Etape 2 - Utilisez la commande « docker network »** pour créer le réseau « web » (vous pouvez lui donner ce que vous voulez comme nom), qui sera réservé à Traefik.

```
docker network create web
```

![Utilisez la commande « docker network » pour créer le réseau « web »](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-3.webp)

En tapant _"docker network ls"_, vous devriez découvrir le réseau nouvellement créé.

![En tapant "docker network ls", vous devriez découvrir le réseau nouvellement créé.](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-4.webp)

Ce réseau sera configuré en mode _"bridge"_, c’est-à-dire qu’il utilisera la carte réseau physique de la machine virtuelle comme s’il était directement branché sur le réseau local. Concrètement, un pont réseau ("bridge") est créé entre l’interface physique de l’hôte et la carte virtuelle de la VM, ce qui permet à la machine virtuelle d’apparaître sur le réseau comme un appareil à part entière, avec sa propre adresse IP, exactement comme un PC ou un smartphone connecté à la box.

- **Etape 3 - Créer un répertoire nommé « letsencrypt »**, puis à l'intérieur, créez le fichier acme.json destiné à stocker vos certificats SSL et appliquez les permissions appropriées.

![Créer un répertoire nommé « letsencrypt »](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-5.webp)

- **Etape 4 - Créer deux fichiers de configuration** qui seront chargés par Traefik au démarrage (un fichier statique et un fichier dynamique). Ces fichiers seront au format YAML.

_Fichier Statique :_

![Créer deux fichiers de configuration qui seront chargés par Traefik au démarrage (un fichier statique et un fichier dynamique).](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-6.webp)

Voici quelques explications :

- **Ligne 1 et 2:** Activation du dashboard de Traefik
- **Ligne 4 à 6:** Indiquez le chemin des logs et le niveau de journalisation. Au moment de l’installation je vous conseille DEBUG, puis une fois en production ERROR.
- **Ligne 8 à 10:** Indiquez le chemin des logs d’accès et le nombre de lignes à afficher
- **Ligne 12 à 24 :** On indique à Traefik d'écouter les ports 80 appelé web et 443 appelé websecure. Toutes les requêtes du port 80 seront redirigées vers le port 443 pour bénéficier du HTTPS et des certificats SSL.  Sur la dernière ligne on indique que c’est Let’s Encrypt notre fournisseur de certificat.
- **Ligne 26 à 34:** Pour générer les certificats SSL, vous avez le choix entre deux serveurs,  acme-staging-v02 pour la pré-production et acme-v02 pour la production. indiquer bien le chemin pour le fichier acme.json.
- **Ligne de 36 à 37:** Cette option dans le fichier traefik.yml indique à Traefik de ne pas vérifier les certificats TLS des serveurs en backend (les services vers lesquels il redirige le trafic, comme tes conteneurs web). Lorsque Traefik agit comme reverse proxy et qu’il communique en HTTPS avec un backend (par exemple, un service web tournant sur https://monapp:443), il vérifie normalement que le certificat TLS de ce backend est valide (autorité de certification reconnue, domaine correspondant, etc.). Avec insecureSkipVerify: true, Traefik ignore ces vérifications. A utiliser en toute connaissance de cause.
- **Ligne de 39 à 44:** Nous allons configurer Traefik pour surveiller en temps réel le socket de Docker, ce qui lui permettra d'identifier toutes les modifications apportées aux conteneurs sur le réseau « traefik » (ce réseau est spécifiquement dédié aux conteneurs accessibles sur le web). Il est à noter que si l'option « exposeByDefault » est définie sur « true », vous n'avez pas nécessité d'ajouter le label « traefik.enable.true » à vos conteneurs, tous seront exposés par défaut. Toutefois, je vous recommande de ne pas modifier cette configuration et de recourir au label pour une exposition manuelle des conteneurs.

## **Déploiement du container Traefik**

Créer un fichier _compose.yml_ et copier le contenu suivant dedans :

![Fichier compose.yml pour Traefik v3](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-8.webp)

- **Ligne 1 à 3:** external: true = indique à Docker d’utiliser un réseau Docker déjà existant nommé web. Sinon, Docker Compose tenterait de créer ce réseau, ce qui provoquerait une erreur s’il existe déjà sous un autre projet ou s’il est géré manuellement.
- **Ligne 5:** services : c'est là qu'il faudra ajouter les configurations de vos conteneurs
- **Ligne 7:** on indique généralement le nom du logiciel
- **Ligne 8:** container_name : le nom du conteneur, sinon Docker en choisira un de manière aléatoire 
- **Ligne 9:** image : nom du registry suivi du nom de l'application et du tag de version
- **Ligne 10:** Pour utiliser les politiques de redémarrage, Docker propose les options suivantes : no : Les conteneurs ne redémarrent pas automatiquement, on-failure[:max-retries] : Redémarrer le conteneur s'il se termine avec un code de sortie non nul, et fournir un nombre maximum de tentatives pour que le démon Docker redémarre le conteneur, always : Redémarre toujours le conteneur s'il s'arrête, unless-stopped : Redémarre toujours le conteneur sauf s'il a été arrêté arbitrairement ou par le démon Docker.
- **Ligne 11 à 12:** network : indique uniquement le réseau "web" dédié à Traefik
- **Ligne de 13 à 15:** ports : 80 (HTTP) et 443 (HTTPS). Aucune autre application n'utilisera ces ports à part Traefik
- **Ligne de 16 à 20:** volumes : on indique le sock de Docker pour que Traefik puisse écouter en permanence (en lecteure seule), puis 3 des chemins où vous allez indiquer le chemin des logs, du fichier statique, du fichier dynamique et du fichier acme qui contiendra tout vos certificats SSL
- **Ligne 21 à 27 :** configuration lié à Traefik.

Voici quelques explications sur les principales commandes utilisées dans le fichier de configuration de Traefik :

- traefik.enable=true,  active Traefik pour pouvoir lui ajouter des services et des routeurs
- traefik.http.routers.traefik-dashboard.entrypoints=websecure, indique que le dashboard de Traefik doit passer par le point d'entrée "websecure" soit le HTTPS, obligatoire pour obtenir le précieux certificat SSL et avoir des échanges sécurisés
- traefik.http.routers.traefik-dashboard.rule=Host( traefik.mondomaine.fr ), on créer une règle (rule) en indiquant le nom de domaine lié à l'application pour qu'on puisse y accéder à travers ce dernier
- traefik.http.routers.traefik-dashboard.service=api@internal, cette ligne configure le router HTTP traefik-dashboard pour qu’il pointe vers un service interne spécial de Traefik : le dashboard. C’est un service interne fourni par Traefik lui-même, qui permet d’accéder à : l’interface dashboard de Traefik, l’API de configuration dynamique, le visualiseur de routes en direct
- traefik.http.routers.traefik-dashboard.middlewares=auth, cela signifie que tu attaches un middleware nommé auth à ton router traefik-dashboard. Ce middleware va agir avant d'autoriser la requête vers le dashboard. Par défaut le dashboard de Traefik n’est pas sécurisé, ici avec ce middleware nous allons le protéger par un login et un mot de passe
- traefik.http.middlewares.auth.basicauth.users=guillaume:$$apr1$$5u7pt4qd$$bxe0zjyQgcXH9K690PbV. Elle définit un middleware auth de type basicAuth, avec l’utilisateur guillaume et un mot de passe encodé en htpasswd

Pour générer un mot de passe encodé, vous pouvez utiliser ce site : [https://www.web2generators.com/apache-tools/htpasswd-generator](https://www.web2generators.com/apache-tools/htpasswd-generator)

![Générer un mot de passe encodé via HTPassword Generator](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-7.webp)

💡

_****Petit rappel hyper important 1****_: Tu dois toujours doubler les $ ($$) dans un fichier docker-compose.yml pour éviter que Docker Compose n'interprète $... comme une variable d’environnement. 

💡

_****Petit rappel hyper important 2****_: Tu ne vois d’appel de certificat dans le fichier docker-compose, car je l’ai rajouté directement le fichier statique de Traefik dans l’entrypoint websecure. Cela permet d’alléger la configuration du fichier docker-compose de l’application.

Pour obtenir une note A+ sur SSL Labs avec Traefik v3, il faut appliquer une configuration stricte et sécurisée concernant TLS/SSL. Cela implique notamment :

- Activer uniquement TLS 1.2 et 1.3
- Utiliser des suites de chiffrement robustes
- Activer HTTP/2
- Désactiver les redirections non sécurisées
- Forcer le HSTS

Les principaux paramètres à modifier/ajouter dans les fichiers de configuration (souvent traefik.yml, traefik.toml, ou via les labels Docker).

### **Modifier la version de TLS minimum et maximum et durcir les suites de chiffrement**

Pour réaliser cette configuration je vous conseille d’utiliser un fichier dynamique nommé _dynamic.yml_ par exemple pour mettre toute la configuration.

Pour que ce fichier dynamique soit reconnu par traefik, vous devez le configurer à plusieurs endroits.

_Référence 1 : traefik.yml_ 

Vous devez ajouter cette ligne au niveau des provider (en rouge sur la capture) _=> Lignes 45 & 46_

![Modifier la version de TLS minimum et maximum et durcir les suites de chiffrement](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-9.webp)

_Référence 2: compose.yml_

Vous devez faire référence à ce fichier au niveau de vos volumes pour le monter dans votre container. _=> Ligne 20_

![Modifier la version de TLS minimum et maximum et durcir les suites de chiffrement](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-10.webp)

Maintenant vous pouvez commencer à l’exploiter en ajoutant vos restrictions pour le TLS et les ciphers. _=> Ligne 1 à 13_

![Ajouter les restrictions sur les Ciphers et les versions de TLS](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-11.webp)

### **Forcer HSTS (HTTP Strict Transport Security)**

Toujours dans votre fichier dynamic.yml, rajouter ces informations. _=> Ligne 15 à 26_

![Forcer HSTS (HTTP Strict Transport Security)](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-12.webp)

Dernière étape mais pas des moindres, vous devez faire référence à ces paramètres dans les labels de votre fichier compose. _=> Ligne 26 & 28_

![Forcer HSTS (HTTP Strict Transport Security) dans le fichier compose.yml d'une appli](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-13.webp)

⚠️

_****Attention !!!****_ Quand vous faites référence au fichier dynamic.yml, vous devez dire explicitement à Traefik de chercher l'option TLS secure ou les secureHeader dans le bon provider, souvent file, en ajoutant @file à la fin de la référence.

Enfin, pour tester votre configuration de votre container Traefik, allez sur [https://www.ssllabs.com/](https://www.ssllabs.com/) puis vérifier votre domaine et vous devriez obtenir la fameuse note A+

![Tester votre configuration de votre container Traefik, allez sur https://www.ssllabs.com/ puis vérifier votre domaine et vous devriez obtenir la fameuse note A+](https://networkpulse.fr/content/images/2025/11/traefik-et-ovh-14.webp)

![Signature](https://networkpulse.fr/content/images/signature-guillaume-claire-1.webp)