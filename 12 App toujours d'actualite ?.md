---
link:
---
C’est en s’inspirant, entre autres, des écrits de Martin Fowler, qu’en 2012, Heroku éditeur d’outils permettant le déploiement d’applicatifs web en mode PaaS, présente un ensemble de guides de bonnes pratiques : the 12-Factor App. L’objectif est d’aider les développeurs à concevoir des applications sous forme de microservices ayant un couplage minimal vis-à-vis des plateformes cloud ou « on premises ». Au travers de cette méthodologie sont abordés des thématiques opérationnelles, d’architecture et de déploiement.

Le but de 12-Factor App est d’écrire du code applicatif, et seulement du code applicatif, indépendamment de l’infrastructure sur laquelle l’application va être exécutée. Pour cela, la gestion des dépendances est un point clé afin d’obtenir un couplage le plus faible possible entre l’environnement d’exécution et le code et, ainsi, faciliter la migration d’application d’un service à l’autre ou d’un fournisseur à l’autre.

![12 factor app pattern](https://www.softfluent.fr/wp-content/uploads/2021/10/pattern.png)

###### source bytise.org

Aujourd’hui, 10 ans après la sortie du manifeste, nous vous proposons un état des lieux.

## Etat des lieux de 12-Factor app

### 1. 12-Factor app : Base de code (Codebase)

Une “application” = une seule base de code

Cette base de code peut être déployée à plusieurs endroits, qu’on appelle “environnements” de tests, de préproduction, de production, avec un système de contrôle de versions tel que Git et possibilité d’effectuer un rollback du code vers une version précédente à tout moment. Ce référentiel permet d’effectuer aisément [l’intégration continu et le déploiement continu](https://www.softfluent.fr/blog/devops-implementation-ci-cd/)

![Base de code](https://www.softfluent.fr/wp-content/uploads/2021/10/Base-de-code-e1633540174166.png)

### **2. 12-Factor App : Dépendances (Dependencies)**

Les dépendances (librairies, outils, …) doivent être décrites explicitement, de manière exhaustive, et doivent être installées lors de l’étape de “build” de l’application. Les [microservices](https://www.softfluent.fr/blog/microservices/) peuvent être mis à jour, étendus et déployés plus rapidement, indépendamment les uns des autres et en limitant le risque d’une mise en péril de l’ensemble de l’applicatif.

![Dépendances](https://www.softfluent.fr/wp-content/uploads/2021/10/Dependances.png)

### 3. 12-Factor App : Configuration (Config)

La configuration ne doit pas faire partie du code mais doit être injectée. En pratique, le plus simple est d’utiliser des variables qui seront définies par environnement. Par ce moyen, l’ensemble des acteurs et outils au sein de la chaine de production applicative **peut** cibler une configuration spécifique à un environnement en fonction de ses besoins.

![Configuration](https://www.softfluent.fr/wp-content/uploads/2021/10/Configuration-1-e1633611267689.png)

> A lire sur le même sujet [Infrastructure as code : appliquer le DevOps à l’infrastructur](https://www.softfluent.fr/blog/infrastructure-as-code-appliquer-le-devops-a-linfrastructure/)

### 3. 12-Factor App : Services de stockage externes (Backing Services)

Une application web utilise très souvent des services externes : base de données, stockage de fichier, envoi d’emails/sms, …Ces services doivent être traités comme externes, découplés de l’application, et utilisés indépendamment de leur méthode de déploiement (service managé par un fournisseur cloud, hébergé par un tiers, installé en local sur la machine). Les ressources auxiliaires (magasins de données, caches, courtiers de messages) doivent être exposées via une URL adressable. La ressource étant ainsi découplée de l’application, elle est interchangeable. On peut ainsi mettre à jour ou modifier ces services à tout moment, sans perturber le fonctionnement de l’application grâce à un détachement/attachement dynamique.

![Stockage](https://www.softfluent.fr/wp-content/uploads/2021/10/Stockage-1.png)

### 5. 12-Factor app : Build, release, run (CI/CD)

Il est recommandé de séparer le déploiement d’une application sur un nouvel environnement en trois phases :

- la phase de ‘build’ installe les dépendances
- la phase de ‘release’ envoie le résultat de cette build sur l’environnement cible
- la phase de ‘run’ lance le ou les processus de l’application

Ce découpage en 3 phases réduit le temps d’indisponibilité de l’applicatif car la phase de build, est souvent une longue opération et peut se faire en parallèle de l’exécution de l’ancienne application sur l’environnement cible. Chaque version doit appliquer une séparation stricte entre les étapes de génération, de mise en œuvre et d’exécution et doit être marquée d’un ID unique avec la possibilité d’effectuer une restauration à tout moment.

![Build, release, run](https://www.softfluent.fr/wp-content/uploads/2021/10/Build-release-run-2.png)

La chaine CI/CD du DevOps respecte complètement ce principe.

### 6. 12-Factor app : Processus (Stateless)

Le principe est d’exécuter pour une application (serveurs web, workers, …) ses processus dans un mode à la fois sans état et sans partage interne :

- En particulier, les processus n’ont pas accès à un système de fichier partagé : les systèmes de fichiers doivent être considérés comme des ressources attachées.
- Chaque microservice doit s’exécuter dans son propre processus, isolé des autres services en cours d’exécution. Cela permet d’externaliser l’État requis sur un service de sauvegarde, tel qu’un cache distribué ou un magasin de données
- Les services sont mis à l’échelle sur un grand nombre de petits processus identiques au lieu d’une mise à l’échelle une seule machine

![Stateless](https://www.softfluent.fr/wp-content/uploads/2021/10/Stateless-1-e1633611447334.png)

### 7. 12-Factor App : Liaison et association de port (Port Binding)

Un processus de type “web” (qui gère des requêtes client en tant que serveur) ne doit pas s’attendre à être contacté par un serveur web externe (Apache, Nginx, …). Il doit porter son propre serveur (par exemple en l’installant comme une dépendance), et exposer un port sur lequel il écoute les requêtes entrantes. Ce port interne sera ensuite associé à un port externe par une association de ports. Chaque microservice doit être autonome avec ses interfaces et fonctionnalités exposées sur son propre port. Cela permet de l’isoler des autres microservices. En fait les choix qui étaient fait à l’échelle d’une application monolithique, doivent être fait de façon identique pour chaque microservice en gardant à l’esprit le besoin d’interopérabilité des différents services, voire l’intégration d’un service bus.

![Port Binding](https://www.softfluent.fr/wp-content/uploads/2021/10/Port-Binding-1.png)

### 8. 12-Factor app : Mise à l’échelle (scalability)

Nous avons vu que les processus ne partagent rien entre eux et sont indépendants des services qu’ils utilisent. Cela permet de lancer chaque processus indépendamment, et de le faire évoluer selon le besoin (“scaler”) et cela de manière manuelle ou dynamique : horizontalement (en augmentant ou diminuant le nombre de processus identiques) ou verticalement (en relançant un processus avec plus ou moins de ressources). On peut s’appuyer sur le pattern CQRS (Command Query Responsibility Segregation) pour séparer les logiques d’écriture et de lecture et ainsi optimiser les performances, l’évolutivité et la sécurité de l’application.

![scalability](https://www.softfluent.fr/wp-content/uploads/2021/10/Scalability-e1633539071318.png)

### 9. 12-Factor App : Jetable (Disposability)

Les processus sont considérés comme jetables. Ils doivent pouvoir être démarrés, redémarrés rapidement, stoppés progressivement pour augmenter les possibilités d’évolutivité, sans affecter le fonctionnement de l’application. La mise en œuvre de conteneurs est un bon exemple de ce principe.

![Jetable](https://www.softfluent.fr/wp-content/uploads/2021/10/Jetable.png)

### 10. 12-Factor App : Parité Dev/Prod (fast releases)

Les développeurs chercheront à respecter les règles des 12-factor app tant sur leur machine qu’en environnement de production, c’est notamment le cas avec le DevOps et le déploiement continu. Cela permet de passer dynamiquement d’un service OnPremise à un service hébergé. Les environnements sont conservés tout au long du cycle de vie des applications ; là encore, l’adoption de conteneurs peut y contribuer en promouvant le même environnement d’exécution

![Parité Dev/Prod](https://www.softfluent.fr/wp-content/uploads/2021/10/Dev-prod.png)

### 11. 12-Factor App : Logs (Logging)

L’objet de ce facteur est d’indiquer que la meilleure pratique pour traiter les journaux générés par les microservices est de les gérer en tant que flux d’événements puis de les agréger afin d’obtenir une supervision unifiée. L’utilisation d’outils comme Splunk ou Azure application Insight permettent de gérer ce point afin d’analyser, de traiter et de transformer en informations exploitables, les données collectées.

![Logs](https://www.softfluent.fr/wp-content/uploads/2021/10/Logs-e1633539601627.png)

> A lire sur le même sujet : [l’aspect supervision du DevOps](https://www.softfluent.fr/blog/laspect-supervision-du-devops/)

### 12. 12-Factor app : Processus d’administration (Admin Process)

On appelle processus d’administration des processus lancés manuellement par un administrateur, afin d’effectuer une tâche manuelle à usage unique (migration de données, exécution d’un script, …). Ces processus d’administration doivent s’exécuter dans un environnement semblable aux processus standards, avoir accès aux mêmes services et aux mêmes variables d’environnement.

Comme nous avons pu le constater au travers de la description des 12 facteurs, cette méthodologie reste pertinente aujourd’hui. On constate toutefois que depuis 10 ans de nouvelles bonnes pratiques additionnelles ont émergées.  Ainsi on peut ajouter trois nouveaux facteurs :

### 13. Priorité aux API

Une API, pour Application programming interface, est un programme permettant à deux applications distinctes de communiquer entre elles et d’échanger des données. Cela évite notamment de recréer et redévelopper entièrement une application pour y ajouter ses informations. Par exemple, elle est là pour faire le lien entre des données déjà existantes et un programme indépendant.

> A lire sur le même sujet : [le rôle d’une passerelle d’API](https://www.softfluent.fr/blog/microservices-api-gateway/)

### 14. Télémétrie

En comparaison à un déploiement local, par nature, l’exécution d’un programme dans le cloud ne permet pas d’avoir une visibilité exhaustive sur l’ensemble du comportement applicatif et de son environnement. Il faut donc s’assurer lors de la conception que les besoins de collecte de données de surveillance, spécifiques à un domaine et concernant l’intégrité du système, soient bien pris en compte.

### 15. Authentification/autorisation

Parmi les bonnes pratiques à retenir, il est également intéressant de mettre en place une procédure d’attribution des droits et un contrôle d’accès en fonction des rôles et ainsi sécuriser l’accès à l’environnement cloud avec des briques d’authentification.

> A lire sur le même sujet : [cloud native, répondre aux enjeux de sécurité](https://www.softfluent.fr/blog/cloud-native-repondre-aux-enjeux-de-securite/)

Plus globalement, il s’agit de définir comment vos services identifient les personnes qui y accèdent et les autorisations dont ils disposent ?

> A lire sur le même sujet : [devSecOps, pourquoi est-ce important ?](https://www.softfluent.fr/blog/devsecops-pourquoi-est-ce-important/)

Outre les conseils fournis dans le cadre de la méthodologie de douze facteurs, il se peut que vous ayez à prendre plusieurs décisions de conception critiques lors de la construction de systèmes distribués.

**Au niveau communication :**

Comment les services centraux principaux communiquent-ils entre eux ? Autorisez-vous les appels HTTP directs ? Pouvez-vous faire abstraction des services principaux avec une passerelle qui assure la flexibilité, le contrôle et la sécurité ? Ou peut-être envisagez-vous une messagerie découplée avec des technologies de file d’attente et de rubrique ?

On parle de requête, commande, événement …

> A lire sur le sujet : [Tout savoir sur message queuing,](https://www.softfluent.fr/blog/tout-savoir-sur-message-queuing/) et aussi [Azure functions : avantages et bonnes pratiques](https://www.softfluent.fr/blog/azure-functions-avantages-et-bonnes-pratiques/)

**En termes de résilience :**

Une architecture de microservices déplace votre système du processus in-process vers la communication réseau hors processus. Dans une architecture distribuée, que se passe-t-il quand le service B ne répond pas à un appel réseau du service A ? Ou bien, que se passe-t-il lorsque le service C devient temporairement indisponible et que les autres services qui l’appellent sont bloqués ?

La résilience permet à votre application de réagir à une défaillance tout en restant fonctionnelle que ce soit une défaillance matérielle, une défaillance régionale, un pic de charge, une suppression ou une altération des données.

**Les données distribuées :**

Par défaut, chaque microservice encapsule ses propres données, exposant les opérations via son interface publique. Si c’est le cas, comment interroger les données ou implémenter une transaction sur plusieurs services ?