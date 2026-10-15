---
link: https://tech-insider.org/fr/podman-remplacer-docker-2026/
byline: Nadia Dubois
site: Tech Insider
date: 2026-08-07T09:55
excerpt: "Migrez de Docker vers Podman en 13 étapes : rootless, podman-compose,
  Kubernetes et systemd. Guide complet avec projet et dépannage."
slurped: 2026-10-15T10:13
title: "Podman : Remplacer Docker en 13 Étapes, 75 Min [2026]"
---

7 August 2026

26 min de lecture

Docker Desktop facture désormais chaque poste professionnel au-delà de l’usage personnel, et de plus en plus d’équipes françaises et européennes cherchent une porte de sortie crédible. Podman s’est imposé comme le remplaçant le plus mûr : gratuit, open source, piloté par Red Hat, et compatible avec la quasi-totalité des commandes Docker que vos développeurs connaissent déjà. Ce tutoriel vous accompagne pas à pas pour migrer une installation Docker existante vers Podman, avec 13 étapes concrètes, un projet complet fonctionnel, une checklist de déploiement en entreprise et une liste de pièges à éviter.

1. [Pourquoi migrer de Docker vers Podman en 2026](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-0)
2. [Qu’est-ce que Podman ? Architecture daemonless et rootless](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-1)

1. [Docker vs Podman : la différence fondamentale](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-2)
2. [Les pods, une unité d’organisation native](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-3)

4. [Prérequis avant de commencer la migration](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-4)
5. [Tableau comparatif : commandes Docker et équivalents Podman](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-5)
6. [Guide étape par étape : migrer de Docker vers Podman en 13 étapes](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-6)

1. [Étape 1 : Faire l’inventaire de votre installation Docker actuelle](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-7)
2. [Étape 2 : Désinstaller ou désactiver Docker Desktop proprement](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-8)
3. [Étape 3 : Installer Podman sur Linux](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-9)
4. [Étape 4 : Installer Podman Desktop sur macOS et Windows](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-10)
5. [Étape 5 : Initialiser et démarrer podman machine](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-11)
6. [Étape 6 : Créer l’alias docker=podman et activer le socket API](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-12)
7. [Étape 7 : Retélécharger ou reconstruire vos images](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-13)
8. [Étape 8 : Installer et configurer podman-compose](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-14)
9. [Étape 9 : Convertir et lancer votre docker-compose.yml existant](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-15)
10. [Étape 10 : Migrer les volumes et les données persistantes](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-16)
11. [Étape 11 : Recréer votre configuration réseau](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-17)
12. [Étape 12 : Générer un manifeste Kubernetes avec podman kube generate](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-18)
13. [Étape 13 : Automatiser le démarrage avec systemd et Quadlet](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-19)

8. [Projet complet : conteneuriser une stack Node.js, PostgreSQL et Nginx](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-20)
9. [Sécurité : ce que change réellement le mode rootless](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-21)
10. [5 erreurs courantes à éviter pendant la migration](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-22)
11. [Dépannage : 9 problèmes fréquents et leurs solutions](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-23)
12. [Astuces avancées pour aller plus loin avec Podman](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-24)
13. [Quand privilégier Podman plutôt que Docker](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-25)
14. [Planifier un déploiement en entreprise](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-26)
15. [Podman face aux autres alternatives à Docker](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-27)
16. [Podman en France et en Europe : l'angle souveraineté](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-28)
17. [Questions fréquentes](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-29)

1. [Podman est-il totalement compatible avec Docker ?](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-30)
2. [Peut-on utiliser un fichier docker-compose.yml directement avec Podman ?](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-31)
3. [Podman fonctionne-t-il sous Windows et macOS ?](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-32)
4. [Le mode rootless a-t-il un impact sur les performances ?](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-33)
5. [Faut-il migrer toute l'infrastructure d'un coup ?](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-34)
6. [Podman est-il gratuit pour un usage professionnel ?](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-35)
7. [Comment revenir à Docker si la migration pose problème ?](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-36)
8. [Podman est-il adapté à un usage Kubernetes ?](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-37)
9. [Contenus associés sur Tech Insider](https://tech-insider.org/fr/podman-remplacer-docker-2026/#toc-38)

Comptez environ 75 minutes pour dérouler l’ensemble du guide sur un projet de taille moyenne. Vous repartirez avec un environnement Podman fonctionnel, une stack applicative migrée de bout en bout, et une méthode pour étendre la migration au reste de votre organisation sans tout casser en cours de route.


## Pourquoi migrer de Docker vers Podman en 2026

La grille tarifaire officielle de Docker, publiée sur [docker.com/pricing](https://www.docker.com/pricing/), montre l’ampleur du problème pour les équipes qui grandissent. L’offre Personal reste gratuite, mais l’offre Pro grimpe à 9 $ par mois en engagement annuel (11 $ en paiement mensuel), l’offre Team atteint 15 $ par utilisateur et par mois (16 $ en mensuel, présentée comme l’offre la plus populaire par Docker lui-même), et l’offre Business plafonne à 24 $ par utilisateur et par mois, sans limite de sièges. Pour une équipe de 20 développeurs sur l’offre Business, la facture annuelle dépasse 5 700 $ rien que pour l’outil de conteneurisation local, sans compter les minutes de build cloud supplémentaires ou les runtimes Testcontainers Cloud facturés à part.

Cette pression tarifaire touche particulièrement les structures qui dépassent le seuil de l’usage personnel gratuit : startups en forte croissance, ESN qui équipent des dizaines de consultants, ou grands comptes qui standardisent leurs postes de développement. Beaucoup découvrent la facture au moment du renouvellement annuel, quand le nombre de comptes actifs a doublé sans que personne n’ait revu le contrat.

Le calcul du retour sur investissement reste simple à poser. Le temps de migration décrit dans ce tutoriel, environ 75 minutes par projet, se rembourse en moins d’un mois pour une équipe de taille moyenne sur l’offre Team ou Business. Pour une DSI qui gère plusieurs centaines de postes, l’économie annuelle se chiffre en dizaines de milliers d’euros, un budget qui se redéploie facilement vers d’autres priorités techniques comme l’observabilité ou la sécurité applicative.

Podman propose une alternative sans coût de licence, développée dans l’écosystème Containers de Red Hat aux côtés de Buildah et Skopeo. Le dépôt officiel sur GitHub affiche 32 469 étoiles et 3 278 forks, et le rythme de publication ne faiblit pas : après une version 6.0.2 signée en juillet 2026 par podman-container-tools/podman, la 6.1.0 est sortie dès août 2026, quelques mois seulement après que la 5.8.2 a corrigé la faille CVE-2026-33414 en avril 2026, selon les correctifs recensés par containers/podman. L’outil graphique associé, Podman Desktop, en est à la version 1.29.1 depuis août 2026, et couvre Linux, macOS et Windows avec une interface proche de celle de Docker Desktop.

Le mot-clé « podman » enregistre à lui seul 3 600 recherches mensuelles en France selon les données DataForSEO, avec une concurrence classée faible. Les recherches associées comme « podman desktop » (590 recherches par mois), « podman compose » (590) et « podman vs docker » (480) confirment un intérêt réel et croissant chez les développeurs francophones, pas seulement chez les early adopters anglophones. Ce n’est plus un outil de niche réservé aux administrateurs système Red Hat.

## Qu’est-ce que Podman ? Architecture daemonless et rootless

Podman signifie littéralement « Pod Manager ». Contrairement à Docker, qui repose sur un daemon (dockerd) tournant en permanence avec des privilèges root, Podman exécute chaque conteneur comme un processus enfant direct, sans intermédiaire privilégié. Cette différence d’architecture n’est pas un détail technique : elle change fondamentalement le modèle de sécurité de vos environnements de développement et de production.

Avec un daemon classique, tous les conteneurs d’une machine dépendent d’un seul processus central. Si ce processus plante, l’intégralité des conteneurs tombe avec lui, même ceux qui n’ont rien à voir avec l’incident d’origine. Podman évite ce point de défaillance unique : chaque conteneur vit indépendamment des autres, rattaché directement à l’utilisateur qui l’a démarré.

### Docker vs Podman : la différence fondamentale

Selon l’explication officielle de [Red Hat sur l’architecture de Podman](https://www.redhat.com/en/topics/containers/what-is-podman), l’outil « ne repose pas sur un processus disposant de privilèges root pour exécuter des conteneurs ». Docker a ajouté un mode rootless plus tard dans son histoire, en option, alors que Podman fonctionne en rootless nativement depuis ses premières versions. Red Hat résume la distinction ainsi : Docker se présente comme un outil tout-en-un, tandis que Podman s’appuie sur des composants modulaires spécialisés, Buildah pour la construction d’images et Skopeo pour leur gestion et leur transfert entre registres.

Sur le plan pratique, la compatibilité des commandes reste très élevée. La plupart des sous-commandes docker fonctionnent à l’identique avec podman, ce qui permet à beaucoup d’équipes de basculer simplement en créant un alias, sans réécrire leurs scripts d’intégration continue du jour au lendemain. Un développeur qui tape docker ps par réflexe obtient le même résultat, formaté de la même façon, sans rien changer à ses habitudes.

### Les pods, une unité d’organisation native

Podman introduit la notion de pod comme brique de base, un concept emprunté directement à Kubernetes. Un pod regroupe un conteneur infra, chargé de maintenir les espaces de noms réseau et utilisateur, et un nombre variable de conteneurs applicatifs qui partagent cet espace réseau. Cette organisation facilite grandement le passage vers Kubernetes plus tard, puisque la commande `podman kube generate` transforme directement un pod local en manifeste Kubernetes exploitable.

Concrètement, plusieurs conteneurs d’un même pod se voient comme s’ils tournaient sur localhost entre eux, exactement comme dans un pod Kubernetes réel. Un développeur qui teste en local une architecture microservices peut ainsi reproduire fidèlement le comportement réseau qu’il retrouvera plus tard sur un cluster de production, sans surprise au moment du déploiement.

## Prérequis avant de commencer la migration

Avant de désinstaller quoi que ce soit, vérifiez que votre environnement répond aux versions minimales suivantes. Ces chiffres correspondent aux dernières versions stables publiées au moment de la rédaction, vérifiées directement sur les dépôts officiels.

|Composant|Version recommandée|Plateforme|Remarque|
|---|---|---|---|
|Podman|6.1.0 (août 2026)|Linux, macOS, Windows|Dernière version stable sur GitHub|
|Podman Desktop|1.29.1 (31 juillet 2026)|macOS, Windows, Linux|Interface graphique optionnelle|
|podman-compose|1.6.0 (3 juin 2026)|Toutes plateformes (Python)|Nécessite Python 3.9 ou supérieur|
|WSL2|Dernière version|Windows 10/11|Requis pour podman machine sous Windows|
|RAM allouée|8 Go minimum|macOS et Windows|Pour la VM podman machine|
|Espace disque|60 Go recommandés|Toutes plateformes|Images et volumes cumulés|

Vous aurez aussi besoin d’un accès administrateur temporaire pour l’installation initiale, même si les conteneurs eux-mêmes tourneront ensuite sans droits root. Sur Linux, un noyau récent avec support des espaces de noms utilisateur activé par défaut suffit, ce qui est le cas sur toutes les distributions maintenues depuis plusieurs années. Vérifiez également que votre compte dispose d’une plage d’UID et de GID subordonnés suffisante, généralement configurée automatiquement à la création du compte via /etc/subuid et /etc/subgid.

Avant de basculer, mesurez aussi l’espace occupé par votre installation Docker actuelle. Une commande docker system df vous donne en quelques secondes le détail par catégorie, images, conteneurs et volumes, ce qui vous permet d’estimer combien d’espace libérer avant de démarrer Podman en parallèle. Sur un poste de développement actif depuis plus d’un an, ce chiffre dépasse fréquemment les 40 Go rien qu’en images inutilisées accumulées au fil des projets.

## Tableau comparatif : commandes Docker et équivalents Podman

La compatibilité de syntaxe entre Docker et Podman constitue l’un des arguments majeurs en faveur de la migration. Le tableau ci-dessous couvre les commandes que vos équipes utilisent quotidiennement, et signale les rares cas où le comportement diffère réellement en arrière-plan.

|Commande Docker|Équivalent Podman|Différence notable|
|---|---|---|
|docker run|podman run|Identique, exécution rootless par défaut|
|docker build|podman build|Identique, s’appuie sur Buildah en interne|
|docker ps|podman ps|Identique|
|docker-compose up|podman-compose up|Nécessite l’installation séparée de podman-compose|
|docker network create|podman network create|Backend netavark au lieu de libnetwork|
|docker volume create|podman volume create|Identique|
|N/A|podman pod create|Concept sans équivalent direct chez Docker|
|N/A|podman kube generate|Génère un manifeste Kubernetes depuis un pod local|
|docker system prune|podman system prune|Identique|
|N/A|podman machine|Gère la VM Linux sous macOS et Windows|

Notez que certaines commandes n’ont pas d’équivalent direct dans un sens comme dans l’autre. Docker Swarm, par exemple, ne trouve aucun correspondant côté Podman : l’orchestration multi-nœuds passe entièrement par Kubernetes une fois que vous dépassez le cadre d’une seule machine.

## Guide étape par étape : migrer de Docker vers Podman en 13 étapes

Voici le processus complet, testé sur Linux, macOS et Windows. Suivez les étapes dans l’ordre pour éviter de perdre des données ou des configurations réseau en cours de route. Chaque étape indique la commande exacte à exécuter, ainsi que le raisonnement derrière elle.

### Étape 1 : Faire l’inventaire de votre installation Docker actuelle

Avant de toucher à quoi que ce soit, listez ce que vous devrez migrer. Exportez la liste de vos images, conteneurs et volumes dans des fichiers texte que vous pourrez consulter pendant toute la migration. Cette étape prend cinq minutes et vous évite de redécouvrir en urgence, trois semaines plus tard, qu’un volume oublié contenait des données importantes.

```
docker images --format "{{.Repository}}:{{.Tag}}" > mes-images.txt
docker ps -a --format "{{.Names}}" > mes-conteneurs.txt
docker volume ls --format "{{.Name}}" > mes-volumes.txt
```

### Étape 2 : Désinstaller ou désactiver Docker Desktop proprement

Sur macOS et Windows, quittez Docker Desktop puis désinstallez-le via le panneau de configuration ou le Finder. Sur Linux, vous pouvez conserver Docker Engine en parallèle le temps de valider la migration : les deux runtimes coexistent sans conflit tant que vous ne mélangez pas leurs sockets réseau. Beaucoup d’équipes gardent volontairement les deux installés pendant deux à trois semaines, le temps de vérifier qu’aucun cas limite n’a été oublié.

### Étape 3 : Installer Podman sur Linux

Sur Debian et Ubuntu, Podman est disponible directement dans les dépôts officiels depuis plusieurs versions. Sur Fedora et RHEL, l’outil est même préinstallé sur certaines images, une conséquence directe de son développement par Red Hat, et l’intégration continue de se resserrer : la version 6.0 a été proposée pour Fedora 44 dès juillet 2026, d’après le Fedora Project Wiki, ce qui confirme que Podman n’est plus un simple paquet additionnel mais un composant de base des distributions portées par Red Hat.

```
# Debian / Ubuntu
sudo apt update
sudo apt install -y podman

# Fedora / RHEL
sudo dnf install -y podman podman-compose

podman --version
```

### Étape 4 : Installer Podman Desktop sur macOS et Windows

Sur macOS, Homebrew reste la méthode la plus simple. Sur Windows, winget installe le paquet officiel signé par Red Hat. Dans les deux cas, l’installation ajoute à la fois le binaire en ligne de commande et l’interface graphique Podman Desktop, dont la cadence de sorties s’est nettement accélérée depuis fin 2025 : la version 1.23 embarquait encore Podman 5.7.0 en novembre 2025, la 1.24 a clôturé l’année en décembre 2025 comme dernière publication du millésime, et l’outil affiche désormais la 1.29.1 depuis août 2026, signe d’un développement soutenu par l’équipe Podman Desktop.

```
# macOS
brew install podman

# Windows (PowerShell)
winget install -e --id RedHat.Podman
```

### Étape 5 : Initialiser et démarrer podman machine

macOS et Windows ne disposent pas d’un noyau Linux natif. Podman crée donc une petite machine virtuelle Linux qui héberge le runtime des conteneurs, en s’appuyant sur WSL2 sous Windows et sur un hyperviseur léger sous macOS. Allouez suffisamment de ressources dès l’initialisation pour éviter de devoir la recréer plus tard, ce qui vous ferait perdre les images déjà téléchargées à l’intérieur.

```
podman machine init --cpus 4 --memory 8192 --disk-size 60
podman machine start
podman machine list
```

### Étape 6 : Créer l’alias docker=podman et activer le socket API

Cette étape est celle qui rend la transition transparente pour vos développeurs. L’alias redirige toutes les commandes docker vers podman, et le socket API rend Podman compatible avec les outils tiers qui parlent au démon Docker par défaut, comme certains plugins d’IDE ou certains outils de CI qui vérifient la présence de /var/run/docker.sock avant de démarrer.

```
echo "alias docker=podman" >> ~/.bashrc
source ~/.bashrc

systemctl --user enable --now podman.socket
export DOCKER_HOST="unix://$XDG_RUNTIME_DIR/podman/podman.sock"
```

### Étape 7 : Retélécharger ou reconstruire vos images

Podman ne partage pas le stockage local de Docker, même sur la même machine. Reprenez la liste établie à l’étape 1 et retéléchargez chaque image depuis son registre d’origine, ou reconstruisez-la depuis votre Dockerfile existant, sans modification, puisque la syntaxe reste compatible à plus de 95 % dans la pratique.

```
while read -r image; do podman pull "$image"; done < mes-images.txt
podman build -t mon-app:latest .
```

### Étape 8 : Installer et configurer podman-compose

Contrairement à Docker, qui intègre Compose nativement depuis 2021, Podman garde cette fonctionnalité dans un paquet séparé, podman-compose, distribué en Python et maintenu par la communauté du projet Containers.

```
pip3 install --user podman-compose
podman-compose --version
```

Le dépôt officiel [containers/podman-compose sur GitHub](https://github.com/containers/podman-compose) documente les rares directives non supportées, principalement celles liées à Docker Swarm, absent de l'écosystème Podman. Pour la grande majorité des fichiers de développement classiques, aucune incompatibilité ne remonte.

### Étape 9 : Convertir et lancer votre docker-compose.yml existant

Dans la majorité des cas, aucune conversion n'est nécessaire. Copiez votre fichier tel quel et lancez-le avec podman-compose plutôt que docker-compose. Si vous référencez des images sans préciser de registre, ajoutez le préfixe complet, par exemple docker.io/library/postgres, pour éviter toute ambiguïté de résolution, un point qui piège souvent les équipes lors de leur premier essai.

```
cp docker-compose.yml podman-compose.yml
podman-compose -f podman-compose.yml up -d
podman-compose ps
```

### Étape 10 : Migrer les volumes et les données persistantes

C'est l'étape la plus délicate de toute la migration. En mode rootless, Podman applique une correspondance d'UID et de GID différente de celle de Docker à cause des espaces de noms utilisateur (user namespaces). Copiez vos données via un montage temporaire, puis corrigez les permissions avec podman unshare, un outil qui exécute une commande dans le même espace de noms que vos conteneurs rootless, sans quoi vos conteneurs verront des fichiers appartenant à un utilisateur qui n'existe pas de leur point de vue.

```
mkdir -p ./data-migres
docker run --rm -v ancien_volume:/source -v ./data-migres:/dest alpine cp -a /source/. /dest/
podman unshare chown -R 999:999 ./data-migres
```

### Étape 11 : Recréer votre configuration réseau

Podman utilise netavark comme backend réseau par défaut, à la place de libnetwork chez Docker. La syntaxe des commandes reste identique, mais recréez vos réseaux personnalisés manuellement : ils ne sont pas importés automatiquement depuis votre ancienne configuration Docker.

```
podman network create mon-reseau-app
podman network inspect mon-reseau-app
```

### Étape 12 : Générer un manifeste Kubernetes avec podman kube generate

Si votre feuille de route inclut un passage vers Kubernetes, cette étape est celle qui justifie le plus la migration. La commande `podman kube generate`, documentée dans la [référence officielle Podman](https://docs.podman.io/en/latest/markdown/podman-kube-generate.1.html), transforme un pod local en fichier YAML prêt à être appliqué sur un cluster, sans que vous ayez besoin d'écrire ce manifeste à la main.

```
podman kube generate mon-pod -f mon-pod-k8s.yaml
podman kube play mon-pod-k8s.yaml
```

### Étape 13 : Automatiser le démarrage avec systemd et Quadlet

Quadlet remplace avantageusement les scripts de démarrage manuels et les entrées crontab bricolées. Il s'agit de fichiers d'unité systemd au format .container, déposés dans ~/.config/containers/systemd/ pour un usage rootless, que systemd traduit automatiquement en services gérés, avec redémarrage automatique en cas de plantage.

```
systemctl --user daemon-reload
systemctl --user enable --now appdb.service
systemctl --user status appdb.service
```

## Projet complet : conteneuriser une stack Node.js, PostgreSQL et Nginx

Pour valider l'ensemble du processus, voici une stack à trois services entièrement migrée vers Podman : une application Node.js, une base PostgreSQL et un reverse proxy Nginx. Ce fichier fonctionne tel quel avec podman-compose, sans modification par rapport à un fichier Docker Compose classique en dehors des préfixes de registre.

```
version: "3.9"
services:
  web:
    image: docker.io/library/node:22-alpine
    working_dir: /app
    volumes:
      - ./app:/app:Z
    command: node server.js
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://appuser:apppass@db:5432/appdb
    depends_on:
      - db
  db:
    image: docker.io/library/postgres:16-alpine
    volumes:
      - db_data:/var/lib/postgresql/data:Z
    environment:
      - POSTGRES_USER=appuser
      - POSTGRES_PASSWORD=apppass
      - POSTGRES_DB=appdb
  proxy:
    image: docker.io/library/nginx:1.27-alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro,Z
    depends_on:
      - web
volumes:
  db_data:
```

Remarquez le suffixe `:Z` sur les volumes montés. Il indique à SELinux d'appliquer un label privé à ce contenu, une précaution absente des configurations Docker classiques mais recommandée par la documentation Podman dès qu'un système avec SELinux actif est utilisé, ce qui est le cas par défaut sur Fedora et RHEL. Sans ce label, le conteneur PostgreSQL refusera parfois d'écrire dans son propre répertoire de données, avec une erreur de permission qui n'a rien d'évident à première vue.

Le reverse proxy Nginx a besoin d'un fichier de configuration minimal pour rediriger le trafic entrant vers le service Node.js. Créez-le à la racine du projet, au même niveau que votre fichier compose.

```
events {}
http {
  server {
    listen 80;
    location / {
      proxy_pass http://web:3000;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
    }
  }
}
```

Lancez la stack, vérifiez l'état des trois services, puis consultez les journaux en temps réel pour confirmer que l'application répond correctement derrière le proxy.

```
podman-compose up -d
podman ps
podman-compose logs -f web
curl http://localhost:8080
```

Une fois la stack validée en local, convertissez-la en fichiers Quadlet pour un déploiement permanent sur un serveur. Générez d'abord un pod à partir des conteneurs en cours d'exécution, puis exportez-le au format Kubernetes avant de l'adapter en unités systemd individuelles pour chaque service. Cette dernière étape reste manuelle aujourd'hui, mais elle prend rarement plus d'une vingtaine de minutes pour une stack de cette taille.

Pour aller plus loin, consultez notre [tutoriel Docker Compose pour une stack de production](https://tech-insider.org/fr/docker-compose-tutoriel-stack-production-13-etapes-2026/), dont la structure générale reste transposable à podman-compose presque sans modification, ainsi que notre [tutoriel Kubernetes pour déployer une application complète](https://tech-insider.org/fr/tutoriel-kubernetes-deployer-application-2026/) une fois votre manifeste généré.

## Sécurité : ce que change réellement le mode rootless

L'argument sécurité revient dans presque toutes les décisions de migration vers Podman, et il mérite d'être détaillé plutôt que répété comme un slogan marketing. Avec Docker, le daemon dockerd tourne en permanence avec des privilèges root. Une faille dans ce processus, ou dans un conteneur mal isolé, peut potentiellement compromettre l'hôte tout entier. Le projet [Rootless Containers](https://rootlesscontaine.rs/), qui documente les techniques d'isolation sans privilèges pour l'ensemble de l'écosystème des conteneurs Linux, explique pourquoi cette classe de vulnérabilités disparaît presque totalement quand aucun processus privilégié ne supervise l'exécution.

Avec Podman, chaque conteneur rootless s'exécute avec les droits de l'utilisateur qui l'a lancé, à l'intérieur d'un espace de noms utilisateur isolé. Même si un attaquant parvient à s'évader d'un conteneur, il se retrouve avec les privilèges limités de ce compte utilisateur, pas avec un accès root à la machine hôte. Pour les équipes soumises à des audits de sécurité réguliers, cette propriété simplifie considérablement la documentation des contrôles et réduit le périmètre à justifier auprès d'un auditeur externe.

SELinux ajoute une couche supplémentaire sur les distributions qui l'activent par défaut. Chaque conteneur reçoit un contexte de sécurité distinct, et les volumes montés doivent porter l'étiquette appropriée, comme illustré plus haut avec le suffixe `:Z`. Sur Ubuntu et Debian, où AppArmor remplace SELinux, l'isolation rootless reste active mais sans cette étape de labellisation spécifique.

Cela dit, rootless ne signifie pas invulnérable. Un conteneur mal configuré, avec l'option --privileged activée sans raison ou un montage direct du socket de l'hôte, retrouve une bonne partie des risques que le mode rootless cherche justement à éliminer. La migration vers Podman ne dispense donc pas d'une revue régulière des options de lancement de vos conteneurs les plus sensibles.

Pour les équipes qui documentent leur conformité auprès d'un RSSI ou d'un auditeur externe, l'argument rootless se traduit concrètement dans les grilles d'évaluation les plus courantes. Un contrôle qui demande à démontrer l'absence de processus root superflu sur les postes de développement se répond en une phrase avec Podman, alors qu'il exige souvent une justification plus longue avec un daemon Docker traditionnel, même bien configuré. Ce gain de temps documentaire pèse autant que le gain de sécurité réel dans la décision de migrer pour certaines directions techniques.

## 5 erreurs courantes à éviter pendant la migration

Ces pièges reviennent le plus souvent dans les retours d'expérience d'équipes qui ont migré leur infrastructure. Les anticiper vous fera gagner plusieurs heures de débogage frustrant, souvent concentrées sur les deux premiers jours suivant le basculement.

- **Confondre le socket Docker et le socket Podman.** Les outils tiers qui cherchent /var/run/docker.sock par défaut ne trouveront rien tant que vous n'avez pas activé et exposé le socket API de Podman à l'étape 6. Certains plugins d'IDE affichent alors une erreur de connexion trompeuse qui laisse croire à un problème d'installation.
- **Vouloir lier un port privilégié en rootless sans ajustement système.** Les ports inférieurs à 1024, comme 80 ou 443, restent bloqués pour les processus non root sur la plupart des distributions Linux tant que net.ipv4.ip_unprivileged_port_start n'est pas modifié.
- **Oublier le comportement des espaces de noms utilisateur sur les volumes.** Un fichier créé par un conteneur rootless n'appartient pas au même UID que celui vu depuis l'hôte, ce qui provoque des erreurs de permission déroutantes si vous ne passez pas par podman unshare.
- **Copier des directives Docker Swarm dans un fichier podman-compose.** Podman ne gère pas l'orchestration multi-nœuds de Swarm. Pour ce besoin, la conversion vers un manifeste Kubernetes via podman kube generate est la voie recommandée.
- **Négliger la persistance de session sur un serveur distant.** Sans activation du linger systemd, les conteneurs rootless d'un utilisateur s'arrêtent automatiquement à la fin de sa session SSH, ce qui surprend beaucoup d'équipes la première fois qu'elles déploient en production.

## Dépannage : 9 problèmes fréquents et leurs solutions

Voici les incidents les plus signalés lors d'une migration, avec la cause probable et la commande de résolution associée. Gardez ce tableau à portée de main pendant vos premières semaines sur Podman.

|Symptôme|Cause probable|Solution|
|---|---|---|
|« permission denied » sur un volume monté|Décalage UID/GID en rootless|podman unshare chown -R UID:GID ./dossier|
|« Cannot connect to the Docker daemon »|Socket API non démarré|systemctl --user enable --now podman.socket|
|Le conteneur refuse le port 80 ou 443|Ports privilégiés bloqués en rootless|sudo sysctl -w net.ipv4.ip_unprivileged_port_start=80|
|Les conteneurs s'arrêtent à la déconnexion SSH|Linger systemd non activé|loginctl enable-linger $USER|
|podman-compose échoue sur une directive Swarm|Fonctionnalité Swarm non supportée|Adapter le fichier ou migrer vers un manifeste Kubernetes|
|podman machine refuse de démarrer sous macOS|Ressources insuffisantes allouées à la VM|podman machine rm puis podman machine init avec plus de RAM|
|Image introuvable après migration|Registre non précisé dans le nom d'image|Configurer unqualified-search-registries dans registries.conf|
|Erreur SELinux « avc denied » sur un volume|Label SELinux manquant|Ajouter le suffixe :Z au montage du volume|
|Lenteur perçue sous Windows avec WSL2|Fichiers montés hors du système de fichiers WSL2|Déplacer le projet dans le filesystem natif WSL2 (\\wsl$)|

La lenteur sous WSL2 mérite une attention particulière parce qu'elle passe souvent inaperçue au départ. Un projet monté depuis /mnt/c/ traverse une couche de traduction entre le système de fichiers Windows et le système de fichiers Linux, ce qui ralentit sensiblement les opérations d'écriture intensives, comme l'installation de dépendances Node.js. Déplacer simplement le dossier de travail vers le filesystem natif WSL2 règle le problème dans la quasi-totalité des cas signalés.

Si aucune des solutions ci-dessus ne résout votre cas précis, la commande podman info fournit un diagnostic complet de votre installation, souvent plus utile qu'un message d'erreur isolé pour comprendre ce qui cloche. Le forum communautaire du projet et les issues GitHub du dépôt officiel restent également très actifs, avec des temps de réponse rapides sur les problèmes déjà rencontrés par d'autres équipes en migration.

## Astuces avancées pour aller plus loin avec Podman

Une fois la migration de base terminée, plusieurs fonctionnalités méritent d'être explorées pour tirer pleinement parti de l'architecture de Podman, au-delà d'un simple remplacement en façade de Docker.

Les healthchecks natifs de Podman s'intègrent directement à systemd, ce qui permet de redémarrer automatiquement un conteneur défaillant sans passer par un orchestrateur externe. Ajoutez simplement une directive HealthCmd dans votre fichier Quadlet pour bénéficier d'une supervision proche de celle d'un cluster Kubernetes, mais sur une seule machine, sans installer d'agent supplémentaire.

Pour vos pipelines d'intégration continue, remplacez progressivement docker build par Buildah lorsque vous avez besoin d'un contrôle plus fin sur la construction d'image, par exemple pour assembler une image depuis plusieurs sources sans Dockerfile unique. Skopeo, de son côté, simplifie la copie d'images entre registres sans jamais les télécharger localement, ce qui accélère nettement les promotions d'image entre environnements de recette et de production, un gain particulièrement net sur les images volumineuses avec beaucoup de couches.

Pensez aussi à activer le nettoyage automatique des images inutilisées via un minuteur systemd plutôt qu'un cron manuel. Une unité .timer associée à podman system prune, planifiée une fois par semaine, évite l'accumulation silencieuse de couches d'images orphelines qui finissent par saturer l'espace disque d'un serveur de développement partagé.

Pour les équipes qui publient des images destinées à tourner sur plusieurs architectures processeur, x86_64 et ARM64 notamment, podman build accepte l'option --platform pour produire des images multi-architecture en une seule commande, sans dépendre d'un service de build cloud tiers. C'est un avantage concret pour les équipes qui développent sur des postes Apple Silicon tout en déployant sur des serveurs x86_64 classiques, un scénario de plus en plus fréquent depuis la généralisation des puces ARM chez les fournisseurs cloud.

Enfin, pensez à scanner vos images pour les vulnérabilités connues avant chaque déploiement. Notre [tutoriel Trivy](https://tech-insider.org/fr/trivy-scanner-vulnerabilites-2026/) détaille comment intégrer ce scan directement dans un pipeline Podman, avec les mêmes commandes que pour une image Docker classique, puisque Trivy s'appuie sur le format d'image OCI standard plutôt que sur une API propriétaire.

## Quand privilégier Podman plutôt que Docker

Podman ne s'impose pas avec la même force dans tous les contextes. Voici les scénarios où le basculement apporte le plus de valeur, et ceux où il vaut mieux temporiser.

Les environnements de développement local individuels constituent le cas d'usage le plus évident. Un développeur qui travaille seul sur son poste n'a besoin d'aucune fonctionnalité propre à Docker Swarm ou aux offres payantes de Docker, et bénéficie immédiatement de l'absence de daemon en arrière-plan qui consomme des ressources même sans conteneur actif. Les environnements d'intégration continue suivent la même logique : chaque job de build tourne dans un contexte isolé et éphémère, où l'absence de daemon partagé simplifie l'isolation entre jobs concurrents.

Les équipes qui visent Kubernetes à moyen terme tirent également un bénéfice direct de la migration, grâce à la compatibilité native des pods et à la génération de manifestes. À l'inverse, une équipe fortement investie dans Docker Swarm pour son orchestration de production a intérêt à traiter cette migration séparément, en substituant Swarm par une solution Kubernetes avant, ou en parallèle, du passage à Podman côté développement.

Les environnements Windows très dépendants de fonctionnalités spécifiques à Docker Desktop, comme certaines intégrations avec des IDE propriétaires peu maintenues, méritent un test préalable approfondi avant un déploiement à grande échelle. La compatibilité API rapprochée par le socket Docker de Podman couvre la majorité des cas, mais pas la totalité des extensions tierces les plus exotiques.

## Planifier un déploiement en entreprise

Migrer un poste de développeur isolé est une chose. Faire basculer cinquante ou deux cents développeurs sans interrompre la production en est une autre. Voici une trame de déploiement progressif, éprouvée par des équipes qui ont mené cette transition sans incident majeur.

Durant les deux premières semaines, limitez la migration à une équipe pilote volontaire, idéalement une équipe qui maîtrise déjà bien ses propres outils de conteneurisation. Documentez chaque friction rencontrée dans un espace partagé, même les plus mineures, elles serviront de base à votre guide interne de migration. Profitez de cette phase pour identifier les scripts de build ou les extensions d'IDE qui dépendent implicitement de comportements spécifiques à Docker.

Entre la troisième et la sixième semaine, étendez la migration aux environnements d'intégration continue, en commençant par les projets les moins critiques. C'est le moment de remplacer progressivement les images de build basées sur docker:dind par des runners compatibles Podman, et de vérifier que vos outils de scan de sécurité et de qualité de code fonctionnent sans adaptation.

À partir du deuxième mois, généralisez la migration au reste des équipes de développement, avec un accompagnement dédié pendant la première semaine de chaque vague. Gardez Docker Desktop désinstallable en un clic pour les cas de blocage réel, plutôt que d'imposer un aller simple qui découragerait l'adoption. La production, elle, ne devrait basculer qu'après plusieurs mois de stabilité côté développement et intégration continue, jamais en même temps que les postes des développeurs.

## Podman face aux autres alternatives à Docker

Podman n'est pas la seule option sur le marché, et il ne convient pas à tous les usages. Le tableau suivant situe les principales alternatives selon leur cas d'usage privilégié, pour vous aider à choisir en connaissance de cause si votre contexte diffère de celui décrit dans ce tutoriel.

|Outil|Cas d'usage privilégié|Modèle de daemon|
|---|---|---|
|Podman|Développement local, migration directe depuis Docker|Daemonless, rootless natif|
|containerd|Production orientée Kubernetes|Daemon léger, sans interface CLI riche|
|Rancher Desktop|Équipes qui veulent choisir entre backend containerd et dockerd|Configurable selon le backend choisi|
|LXC/LXD|Conteneurs système complets, proches de VM légères|Daemon LXD, isolation plus large qu'un conteneur applicatif|

containerd s'impose surtout comme composant interne d'un cluster Kubernetes plutôt que comme outil de développement quotidien, la plupart des équipes ne l'utilisent jamais directement en ligne de commande. Rancher Desktop séduit les équipes qui veulent garder la possibilité de basculer entre plusieurs backends sans changer d'interface, un compromis pratique pour les organisations en transition qui n'ont pas encore tranché définitivement. LXC et LXD, enfin, ciblent des besoins différents : des conteneurs système complets qui se comportent presque comme des machines virtuelles légères, plutôt que des conteneurs applicatifs isolés au sens de Docker ou Podman.

## Podman en France et en Europe : l'angle souveraineté

Pour les équipes françaises et européennes, la migration vers Podman s'inscrit souvent dans une réflexion plus large sur la souveraineté numérique et la maîtrise des coûts en devises étrangères. Les tarifs Docker Business, exprimés en dollars, subissent directement les variations de change, un facteur que beaucoup de directions financières préfèrent éliminer purement et simplement en repassant sur un outil sans facturation récurrente.

Cette dynamique s'accompagne d'un mouvement plus large vers des registres et des plateformes d'hébergement européens. Associer Podman à un registre de conteneurs hébergé en France ou dans l'Union européenne, plutôt qu'à Docker Hub par défaut, permet de garder l'ensemble de la chaîne de conteneurisation sous juridiction européenne, un argument qui pèse de plus en plus dans les appels d'offres du secteur public et des industries régulées comme la santé ou la finance.

Sur le plan communautaire, la documentation francophone autour de Podman s'est nettement étoffée depuis deux ans, avec des guides détaillés publiés par des experts techniques indépendants et des cabinets de conseil européens, signe que l'outil dépasse désormais le cercle des premiers adoptants pour toucher les équipes de production classiques. Plusieurs écoles d'ingénieurs françaises ont également commencé à intégrer Podman dans leurs modules de conteneurisation, aux côtés de Docker plutôt qu'à sa place, ce qui devrait accélérer encore l'adoption dans les prochaines promotions de développeurs.

Les administrations publiques françaises, soumises à des recommandations de plus en plus strictes sur l'usage de logiciels open source et sur l'hébergement souverain, figurent parmi les organisations les plus actives sur ce terrain. Plusieurs collectivités et opérateurs publics ont documenté publiquement leur passage à des piles techniques reposant sur des outils libres comme Podman, associés à des registres hébergés localement, une tendance qui devrait se renforcer à mesure que les clauses de souveraineté numérique se généralisent dans les marchés publics informatiques.

## Questions fréquentes

### Podman est-il totalement compatible avec Docker ?

La compatibilité couvre la grande majorité des commandes et des fichiers Dockerfile sans modification. Les principales exceptions concernent Docker Swarm, absent de Podman, et certaines options avancées du daemon Docker qui n'ont pas d'équivalent dans une architecture daemonless. Pour un usage de développement classique, la quasi-totalité des équipes ne rencontre aucune incompatibilité bloquante.

### Peut-on utiliser un fichier docker-compose.yml directement avec Podman ?

Oui, à condition d'installer podman-compose séparément, comme détaillé à l'étape 8 de ce guide. La syntaxe du fichier reste identique dans la quasi-totalité des cas, seules les directives spécifiques à Swarm posent parfois problème et nécessitent une adaptation.

### Podman fonctionne-t-il sous Windows et macOS ?

Oui, via Podman Desktop et la commande podman machine, qui déploie une machine virtuelle Linux légère pour héberger le runtime. Sous Windows, cette VM s'appuie sur WSL2, déjà présent sur la majorité des postes de développement récents.

### Le mode rootless a-t-il un impact sur les performances ?

L'écart reste marginal pour la majorité des charges applicatives classiques. Les opérations réseau intensives peuvent montrer une légère différence à cause de la traduction supplémentaire des espaces de noms, mais elle passe généralement inaperçue en dehors de benchmarks très ciblés sur des charges à très haut débit.

### Faut-il migrer toute l'infrastructure d'un coup ?

Non, et ce n'est pas recommandé. Commencez par les environnements de développement local avec une équipe pilote, validez la stabilité pendant plusieurs semaines, puis étendez progressivement vers l'intégration continue et enfin la production, en suivant la trame décrite plus haut dans ce guide.

### Podman est-il gratuit pour un usage professionnel ?

Oui, sans restriction liée au nombre de postes ou à la taille de l'entreprise. Le projet reste open source sous licence Apache 2.0, sans offre commerciale à sièges limités comme celle de Docker, et sans version bridée réservée aux particuliers. Les organisations qui ont besoin d'un support contractuel peuvent toutefois se tourner vers le Red Hat build of Podman Desktop, passé en disponibilité générale dès janvier 2026 selon l'annonce officielle de Red Hat, une option supportée qui vient s'ajouter au projet communautaire gratuit sans le remplacer.

### Comment revenir à Docker si la migration pose problème ?

Conservez vos fichiers Dockerfile et docker-compose.yml d'origine sans les modifier directement, travaillez sur des copies pendant toute la migration. Réinstaller Docker Desktop et reprendre les fichiers originaux suffit à revenir en arrière sans perte de configuration.

### Podman est-il adapté à un usage Kubernetes ?

Particulièrement bien adapté, grâce à son concept natif de pod et à la commande podman kube generate qui produit directement des manifestes compatibles, comme détaillé à l'étape 12. C'est même l'un des principaux arguments avancés par les équipes qui migrent en prévision d'un déploiement Kubernetes futur.

### Contenus associés sur Tech Insider

- [Podman vs Docker 2026 : le comparatif détaillé des moteurs de conteneurs](https://tech-insider.org/fr/podman-vs-docker-2026/)
- [Installer Docker sur Ubuntu 24.04 en 13 étapes](https://tech-insider.org/fr/installer-docker-ubuntu-24-04-tutoriel-13-etapes-2026/)
- [Tutoriel Docker : conteneuriser une application en 13 étapes](https://tech-insider.org/fr/tutoriel-docker-conteneuriser-app-13-etapes-2026/)
- [Tutoriel Docker Compose : stack de production en 13 étapes](https://tech-insider.org/fr/docker-compose-tutoriel-stack-production-13-etapes-2026/)
- [Tutoriel Docker Swarm : cluster de production en 13 étapes](https://tech-insider.org/fr/docker-swarm-tutoriel-cluster-production-13-etapes-2026/)
- [Trivy : scanner de vulnérabilités pour vos images de conteneurs](https://tech-insider.org/fr/trivy-scanner-vulnerabilites-2026/)
- [Tutoriel Kubernetes : déployer une application complète](https://tech-insider.org/fr/tutoriel-kubernetes-deployer-application-2026/)

![Nadia Dubois](https://tech-insider.org/wp-content/uploads/2026/03/ti-author-nadia.jpg)

#### [Nadia Dubois](https://tech-insider.org/fr/author/nadia-dubois/)

AI & Innovation Editor

Nadia Dubois est rédactrice IA & Innovation chez Tech Insider, où elle suit l'évolution rapide de l'intelligence artificielle. Elle a précédemment couvert l'IA et les startups pour La Tribune et contribué à la couverture européenne du MIT Technology Review. Nadia est spécialisée dans l'IA générative, la régulation de l'IA et l'intersection entre technologie et politique industrielle européenne. Elle est diplômée de Sciences Po Paris en linguistique computationnelle et journalisme.

[Voir tous les articles](https://tech-insider.org/fr/author/nadia-dubois/)