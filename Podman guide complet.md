---
tags:
  - podman
---
[](https://www.shpv.fr/blog/podman-2026/)

Podman s'est imposé comme l'alternative à Docker dans les environnements d'entreprise exigeants en matière de sécurité. Développé par Red Hat et adopté massivement depuis 2024, Podman a une approche différente de la conteneurisation : pas de daemon central, exécution rootless par défaut, et support natif des pods Kubernetes. Ce guide couvre Podman pour une gestion de containers sécurisée et conforme aux standards modernes.

Si vous utilisez Kubernetes, consultez notre [guide d'introduction aux concepts fondamentaux de Kubernetes](https://www.shpv.fr/blog/kubernetes-introduction/). Pour approfondir les aspects de sécurité, explorez [le hardening Docker](https://www.shpv.fr/blog/hardening-docker/), [les profils AppArmor et seccomp](https://www.shpv.fr/blog/docker-security-profile/), et [le scanning avec Trivy](https://www.shpv.fr/blog/trivy-container-scanning/).

#### Comprendre Podman et son écosystème

##### L'évolution du paysage des containers

Le monde de la conteneurisation a considérablement évolué depuis les débuts de Docker en 2013. Si Docker a démocratisé les containers et reste largement utilisé, son architecture avec daemon central pose des problèmes de sécurité et de complexité opérationnelle qui deviennent critiques en production. C'est dans ce contexte que Podman (Pod Manager) a émergé comme une alternative architecturale fondamentalement différente.

**Podman** n'est pas un simple clone de Docker. C'est une refonte complète de l'approche des containers, construite autour de trois principes fondamentaux : pas de daemon central (daemonless), exécution rootless native, et support des pods comme citoyens de première classe. Cette philosophie répond aux exigences de sécurité accrues des entreprises tout en conservant une compatibilité CLI avec Docker qui facilite grandement la migration.

##### Architecture : daemon vs daemonless

La différence fondamentale entre Docker et Podman réside dans leur architecture. **Docker** repose sur un daemon central (dockerd) qui tourne en tant que root et gère tous les containers. Ce daemon est un point unique de défaillance : s'il plante, tous vos containers deviennent inaccessibles. Plus problématique encore, il représente une surface d'attaque massive puisqu'il s'exécute avec les privilèges root.

**Podman**, à l'inverse, adopte une architecture daemonless. Chaque container est simplement un processus enfant lancé directement par la commande podman. Pas de daemon intermédiaire, pas de socket privilégié à protéger. Quand vous exécutez `podman run`, le processus container est directement fils de votre shell. Cette approche présente des avantages majeurs : pas de single point of failure, isolation complète entre containers, et sécurité renforcée par l'absence de daemon root.

##### Rootless : sécurité par défaut

Le mode rootless est probablement l'innovation la plus importante de Podman. Contrairement à Docker où le rootless est une fonctionnalité optionnelle et complexe à configurer, **Podman exécute les containers rootless par défaut**. Concrètement, cela signifie que même si un attaquant compromet votre container, il ne dispose que des privilèges de votre utilisateur système, pas de root.

Cette sécurité repose sur les user namespaces Linux. Podman mappe votre UID utilisateur (disons 1000) vers le root (UID 0) à l'intérieur du container. Le processus pense être root dans son namespace isolé, mais pour le système hôte, c'est juste votre utilisateur 1000. Ainsi, même une vulnérabilité permettant une évasion du container ne donne à l'attaquant que vos privilèges utilisateur limités.

##### Pods : l'approche Kubernetes native

Podman introduit le concept de **pods**, directement inspiré de Kubernetes. Un pod est un groupe de containers qui partagent le même namespace réseau et les mêmes volumes. C'est l'équivalent d'un pod Kubernetes mais géré localement. Cette approche résout élégamment les problèmes de communication inter-containers : au lieu de créer des réseaux Docker complexes, vous mettez simplement vos containers dans le même pod où ils peuvent communiquer via localhost.

Cette compatibilité conceptuelle avec Kubernetes n'est pas anodine. Podman peut générer des manifestes Kubernetes à partir de vos pods locaux, et inversement exécuter des pods définis en YAML Kubernetes. Cette bidirectionnalité facilite considérablement le développement local d'applications destinées à tourner sur Kubernetes en production.

---

#### Installation et configuration initiale

##### Prérequis système

Avant d'installer Podman, comprenez les prérequis système pour le mode rootless. Podman nécessite des **user namespaces** configurés correctement, ce qui implique des mappages subuid/subgid. Ces mappages définissent quelle plage d'UIDs votre utilisateur peut utiliser à l'intérieur des containers.

La plupart des distributions modernes (Ubuntu 22.04+, RHEL 8+, Fedora 35+) ont ces configurations par défaut, mais il est toujours bon de vérifier. Vous devez également disposer d'un kernel Linux 4.18+ avec le support des user namespaces activé. Sur certains systèmes plus anciens ou durcis, ce support peut être désactivé par sécurité, ce qui bloquera le mode rootless.

##### Installation sur distributions majeures

**Sur Ubuntu/Debian**, Podman est disponible dans les dépôts officiels depuis Ubuntu 20.10, mais les versions récentes (22.04+) offrent une expérience nettement meilleure avec Podman 4.x :

```bash
# Ubuntu 22.04+ / Debian 12+
sudo apt update
sudo apt install -y podman

# Vérifier installation
podman --version
# podman version 4.9.4

# Installer outils complémentaires
sudo apt install -y podman-compose buildah skopeo

# Vérifier support rootless
podman info | grep -i rootless
# rootless: true
```

**Sur RHEL, CentOS Stream et Fedora**, Podman est non seulement préinstallé mais également supporté officiellement par Red Hat. C'est le runtime de containers par défaut sur RHEL 8+ :

```bash
# RHEL 8+ / CentOS Stream 8+
sudo dnf install -y podman podman-compose

# Fedora (généralement préinstallé)
podman --version

# Installer plugins additionnels
sudo dnf install -y podman-plugins podman-docker
```

Le paquet `podman-docker` est particulièrement intéressant : il fournit un alias `docker` qui pointe vers `podman`, permettant d'utiliser vos scripts et workflows Docker existants sans modification.

##### Configuration rootless détaillée

La configuration rootless nécessite que votre utilisateur dispose de mappages subuid/subgid. Ces fichiers `/etc/subuid` et `/etc/subgid` définissent les plages d'UIDs que vous pouvez utiliser :

```bash
# Vérifier mappages existants
cat /etc/subuid
# username:100000:65536

cat /etc/subgid
# username:100000:65536
```

Ce mapping signifie que votre utilisateur peut mapper 65536 UIDs à partir de 100000. À l'intérieur du container, l'UID 0 (root) sera mappé vers votre UID utilisateur, l'UID 1 vers 100000, l'UID 2 vers 100001, etc. Ce mécanisme assure l'isolation tout en permettant aux applications containers de croire qu'elles tournent en root.

Si ces mappages sont absents, ajoutez-les manuellement :

```bash
# Ajouter mappages pour utilisateur courant
sudo usermod --add-subuids 100000-165535 --add-subgids 100000-165535 $USER

# Reboot nécessaire pour appliquer
sudo reboot

# Vérifier après reboot
podman system migrate
podman info | grep -A 10 IDMappings
```

La commande `podman system migrate` réinitialise l'environnement rootless avec les nouveaux mappages. C'est important de l'exécuter après tout changement de configuration subuid/subgid.

##### Configuration registries

Podman utilise `/etc/containers/registries.conf` pour configurer les registries d'images. Par défaut, il cherche d'abord dans plusieurs registries (docker.io, quay.io, etc.), ce qui peut créer des ambiguïtés :

```bash
# Voir configuration actuelle
cat /etc/containers/registries.conf

# Créer configuration utilisateur
mkdir -p ~/.config/containers
nano ~/.config/containers/registries.conf
```

Configuration recommandée pour prioriser Docker Hub :

```toml
# ~/.config/containers/registries.conf

[registries.search]
registries = ['docker.io', 'quay.io']

[registries.insecure]
registries = []

[registries.block]
registries = []
```

Cette configuration privilégie docker.io (Docker Hub) comme registry principal, évitant les ambiguïtés lors des pulls d'images.

---

#### Commandes de base et migration depuis Docker

##### Compatibilité CLI : de docker à podman

L'une des forces majeures de Podman est sa **compatibilité CLI avec Docker**. Presque toutes les commandes docker fonctionnent en remplaçant simplement `docker` par `podman`. Cette compatibilité n'est pas superficielle : elle inclut les options, flags et comportements.

```bash
# Docker
docker run -d --name web nginx
docker ps
docker logs web
docker exec -it web bash
docker stop web
docker rm web

# Podman (syntaxe identique)
podman run -d --name web nginx
podman ps
podman logs web
podman exec -it web bash
podman stop web
podman rm web
```

Cette équivalence fonctionne pour l'écrasante majorité des commandes. Quelques différences subsistent cependant, notamment autour des fonctionnalités spécifiques à Docker daemon (swarm, plugins daemon) qui n'ont pas d'équivalent Podman puisqu'il n'y a pas de daemon.

##### Alias docker pour transition transparente

Pour faciliter la migration, vous pouvez créer un alias système qui redirige `docker` vers `podman`. Sur RHEL/Fedora, le paquet `podman-docker` fait cela automatiquement. Sur d'autres distributions :

```bash
# Alias temporaire (session actuelle)
alias docker=podman

# Alias permanent
echo "alias docker='podman'" >> ~/.bashrc
source ~/.bashrc

# Vérifier
docker --version
# podman version 4.9.4
```

Avec cet alias, vos scripts shell utilisant `docker` fonctionneront avec Podman sans modification. C'est particulièrement utile pour les outils CI/CD ou les makefiles qui invoquent docker.

##### Gestion images : pull, build, tag

La gestion des images avec Podman est identique à Docker. Podman utilise le même format OCI (Open Container Initiative) et peut tirer des images depuis n'importe quel registry compatible :

```bash
# Pull image depuis Docker Hub
podman pull nginx:alpine

# Pull depuis registry spécifique
podman pull quay.io/podman/hello

# Pull depuis registry privé
podman pull registry.example.com/myapp:latest \
    --creds username:password

# Lister images locales
podman images

# Inspecter image
podman inspect nginx:alpine

# Tag image
podman tag nginx:alpine myregistry.com/nginx:v1

# Push vers registry
podman push myregistry.com/nginx:v1 \
    --creds username:password
```

Les layers d'images sont stockés dans `~/.local/share/containers/storage` pour le mode rootless. Cette séparation entre utilisateurs signifie que chaque utilisateur a son propre cache d'images, renforçant l'isolation mais augmentant l'utilisation disque si plusieurs utilisateurs utilisent Podman.

##### Build d'images : Containerfile et Dockerfile

Podman peut builder des images à partir de Dockerfiles (ou Containerfiles, nom OCI standard) de manière totalement compatible avec Docker :

```bash
# Build simple
podman build -t myapp:v1 .

# Build avec Dockerfile spécifique
podman build -f Dockerfile.prod -t myapp:prod .

# Build multi-stage
podman build --target production -t myapp:prod .

# Build avec arguments
podman build --build-arg VERSION=1.2.3 -t myapp:1.2.3 .

# Build sans cache
podman build --no-cache -t myapp:latest .
```

Podman utilise **Buildah** en backend pour les builds. Buildah est un outil spécialisé dans la construction d'images OCI qui offre plus de flexibilité que `docker build`. Vous pouvez d'ailleurs utiliser Buildah directement pour des builds complexes :

```bash
# Build avec Buildah (plus de contrôle)
buildah from alpine
buildah run alpine-working-container apk add nginx
buildah commit alpine-working-container mynginx
```

Cette séparation entre runtime (Podman) et build tool (Buildah) suit la philosophie Unix de spécialisation des outils.

---

#### Containers rootless : sécurité avancée

##### User namespaces en pratique

Le mode rootless de Podman repose sur les **user namespaces**, une fonctionnalité kernel Linux qui permet de remap les UIDs. Comprendre ce mécanisme est central pour diagnostiquer les problèmes de permissions.

Quand vous lancez un container rootless, Podman crée un user namespace où votre UID utilisateur (ex: 1000) devient l'UID 0 (root) à l'intérieur du container. Les autres UIDs dans le container sont mappés vers votre plage subuid :

```bash
# Lancer container rootless
podman run -it --rm alpine sh

# À l'intérieur du container
id
# uid=0(root) gid=0(root)

# Depuis l'hôte (autre terminal)
ps aux | grep alpine
# username  12345  ... /usr/bin/conmon
```

Le processus container apparaît sous votre username sur l'hôte, pas comme root. Cette isolation protège l'hôte : même si l'attaquant obtient root dans le container, il n'a que vos privilèges utilisateur sur l'hôte.

##### Limitations du mode rootless

Le mode rootless présente cependant quelques limitations inhérentes à l'absence de privilèges root :

**Ports privilégiés (&lt ;1024)** : Par défaut, un container rootless ne peut pas binder les ports &lt ;1024 car cela nécessite CAP_NET_BIND_SERVICE. La solution est d'utiliser des ports &gt ;1024 ou de configurer un port forwarding :

```bash
# Erreur : port 80 refusé
podman run -p 80:80 nginx
# Error: rootlessport cannot expose privileged port 80

# Solution 1 : utiliser port &gt;1024
podman run -p 8080:80 nginx

# Solution 2 : activer ports privilégiés (nécessite sysctl)
sudo sysctl net.ipv4.ip_unprivileged_port_start=80
podman run -p 80:80 nginx
```

**Performance réseau** : Le réseau rootless utilise **slirp4netns** par défaut, qui fournit une isolation réseau mais avec une surcharge performance de 10-20 % comparé au réseau root. Pour les workloads intensifs réseau, cette surcharge peut être notable.

**Volumes et permissions** : Les volumes montés doivent être accessibles par votre UID utilisateur. Monter `/etc` ou d'autres répertoires système protégés échouera. Les fichiers créés dans les volumes depuis le container apparaissent avec votre UID sur l'hôte, pas root, ce qui peut surprendre initialement mais est en fait un comportement plus sécurisé.

##### Mode root quand nécessaire

Bien que le rootless soit recommandé, certains cas nécessitent le mode root : accès raw aux périphériques, ports privilégiés, intégration avec systemd socket activation. Podman supporte également le mode root :

```bash
# Lancer container en mode root
sudo podman run -d -p 80:80 nginx

# Lister containers root
sudo podman ps

# Attention : containers root et rootless sont isolés
# Les images et containers ne sont PAS partagés entre les deux modes
```

Les containers root de Podman bénéficient toujours de l'architecture daemonless, donc l'isolation inter-containers est meilleure que Docker même en mode root.

---

#### Pods : groupes de containers

##### Concept de pods Kubernetes-like

Les **pods** sont une fonctionnalité unique à Podman qui n'existe pas nativement dans Docker. Un pod est un groupe de containers qui partagent :

- Le même namespace réseau (adresse IP commune)
- Le même namespace IPC
- Optionnellement, le même namespace PID
- Les mêmes volumes

Cette approche reflète exactement le modèle des pods Kubernetes. En pratique, cela signifie que les containers d'un pod communiquent via localhost sans configuration réseau complexe :

```bash
# Créer un pod
podman pod create --name webapp -p 8080:80

# Lister pods
podman pod list
# POD ID     NAME      STATUS   CREATED   INFRA ID

# Ajouter containers au pod
podman run -d --pod webapp --name nginx nginx:alpine
podman run -d --pod webapp --name app node:alpine node app.js

# Les containers nginx et app partagent le réseau
# nginx peut accéder à app via localhost:3000
# Seul le port 8080 du pod est exposé à l'hôte
```

##### Architecture interne d'un pod

Quand vous créez un pod, Podman lance automatiquement un **container infra** (basé sur l'image pause). Ce container infra sert d'ancre pour les namespaces partagés. Les autres containers rejoignent ensuite ces namespaces. Cette architecture garantit que les namespaces persistent même si vous arrêtez temporairement les containers applicatifs.

```bash
# Voir détails pod incluant container infra
podman pod inspect webapp

# Le container infra apparaît dans ps
podman ps -a --pod
# Vous verrez le container infra + vos containers
```

Le container infra consomme très peu de ressources (quelques MB mémoire) car il ne fait qu'holder les namespaces sans exécuter de processus applicatif.

##### Use cases pods

Les pods sont particulièrement adaptés pour :

**1. Architecture sidecar** : Un container principal + un container auxiliaire (logs, metrics, proxy)

```bash
podman pod create --name app-with-sidecar -p 8080:80

# Container principal
podman run -d --pod app-with-sidecar --name app myapp:latest

# Sidecar : log forwarder
podman run -d --pod app-with-sidecar --name logger \
    -v /var/log/app:/logs fluent-bit:latest
```

**2. Services multi-containers** : App + database localement

```bash
podman pod create --name stack

podman run -d --pod stack --name db postgres:15
podman run -d --pod stack --name app \
    -e DATABASE_URL=postgresql://localhost/db \
    myapp:latest

# App accède DB via localhost sans network bridge
```

**3. Développement local avec services** : Reproduire architecture Kubernetes localement

```bash
# Créer pod équivalent à pod Kubernetes
podman pod create --name k8s-local -p 8080:80

podman run -d --pod k8s-local --name frontend nginx
podman run -d --pod k8s-local --name backend node:alpine
podman run -d --pod k8s-local --name cache redis:alpine
```

##### Génération manifestes Kubernetes

Podman peut exporter vos pods en YAML Kubernetes, facilitant le passage du développement local vers Kubernetes production :

```bash
# Générer YAML Kubernetes depuis pod Podman
podman generate kube webapp > webapp-pod.yaml

# Ce YAML peut être appliqué directement sur Kubernetes
kubectl apply -f webapp-pod.yaml

# Ou rejoué avec Podman
podman play kube webapp-pod.yaml
```

Cette bidirectionnalité est unique et précieuse pour les workflows cloud-native.

---

#### Systemd integration

##### Génération units systemd

L'intégration systemd est un avantage majeur de Podman sur Docker. Podman peut générer automatiquement des unit files systemd pour vos containers, permettant leur gestion comme n'importe quel service système :

```bash
# Créer container
podman run -d --name web \
    -p 8080:80 \
    --restart=always \
    nginx:alpine

# Générer unit systemd utilisateur
podman generate systemd --new --name web \
    > ~/.config/systemd/user/container-web.service

# Recharger systemd
systemctl --user daemon-reload

# Gérer avec systemctl
systemctl --user enable container-web
systemctl --user start container-web
systemctl --user status container-web

# Le container démarre automatiquement au login utilisateur
```

Le flag `--new` génère un unit file qui recrée le container à chaque démarrage, garantissant un état propre. L'alternative est de générer un unit qui démarre le container existant.

##### Containers en tant que services système

Pour les services nécessitant un démarrage au boot système (avant login utilisateur), utilisez systemd system units avec Podman root :

```bash
# Container root pour service système
sudo podman run -d --name traefik \
    -p 80:80 -p 443:443 \
    -v /etc/traefik:/etc/traefik:ro \
    traefik:latest

# Générer unit système
sudo podman generate systemd --new --name traefik \
    > /etc/systemd/system/container-traefik.service

sudo systemctl daemon-reload
sudo systemctl enable container-traefik
sudo systemctl start container-traefik
```

Cette approche transforme vos containers en citoyens de première classe du système, gérés par systemd avec toutes les fonctionnalités : dependencies, restart policies, logging, etc.

##### Quadlet : configuration declarative

Podman 4.4+ introduit **Quadlet**, un système de configuration déclarative pour containers systemd. Au lieu de générer des unit files, vous écrivez des fichiers `.container` simples :

```bash
# ~/.config/containers/systemd/web.container
[Unit]
Description=Web Server
After=network.target

[Container]
Image=nginx:alpine
PublishPort=8080:80
Volume=/data/www:/usr/share/nginx/html:ro
Environment=TZ=Europe/Paris

[Service]
Restart=always

[Install]
WantedBy=default.target
```

Quadlet convertit automatiquement ces fichiers en units systemd au démarrage :

```bash
# Recharger pour découvrir nouveaux .container
systemctl --user daemon-reload

# Le service web est créé automatiquement
systemctl --user start web
systemctl --user enable web
```

Quadlet simplifie grandement la gestion de containers avec systemd en évitant la génération manuelle d'unit files.

---

#### Podman Compose : orchestration locale

##### Docker Compose vs Podman Compose

Podman supporte Docker Compose via deux approches :

**1. podman-compose** : Implémentation Python compatible docker-compose **2. podman compose** : Support natif (Podman 4.1+) via libpod

La syntaxe `docker-compose.yml` standard est supportée, permettant une migration transparente :

```bash
# Installer podman-compose
pip3 install podman-compose

# Ou utiliser version native (recommandé)
# Disponible dans podman 4.1+
```

##### Exemple stack multi-containers

Créons une stack applicative complète avec Podman Compose :

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - db-data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD', 'pg_isready', '-U', 'postgres']
      interval: 10s

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data

  app:
    image: myapp:latest
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - '8080:8080'
    environment:
      DATABASE_URL: postgresql://postgres:secret@db/myapp
      REDIS_URL: redis://redis:6379
    depends_on:
      - db
      - redis
    restart: unless-stopped

volumes:
  db-data:
  redis-data:
```

Déploiement avec Podman Compose :

```bash
# Démarrer stack (version native)
podman compose up -d

# Ou avec podman-compose
podman-compose up -d

# Voir services
podman compose ps

# Logs
podman compose logs -f app

# Arrêter
podman compose down

# Arrêter + supprimer volumes
podman compose down -v
```

##### Différences comportementales

Quelques différences subtiles existent entre Docker Compose et Podman Compose :

**Réseau** : Podman Compose crée automatiquement un pod si tous les services n'ont pas de network_mode spécifié, alors que Docker Compose crée un bridge network.

**Build** : `podman compose build` utilise Buildah en backend, offrant parfois des comportements légèrement différents des builds Docker.

**DNS** : La résolution des noms de services (ex: accéder à `db` depuis `app`) fonctionne via DNS dans Docker, mais via `/etc/hosts` dans les pods Podman.

Ces différences sont le plus souvent transparentes, mais peuvent demander des ajustements mineurs sur des setups complexes.

---

#### Migration depuis Docker

##### Audit containers Docker existants

Avant de migrer, auditez vos containers Docker pour identifier les incompatibilités potentielles :

```bash
# Lister containers Docker
docker ps -a --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"

# Identifier containers avec volumes système
docker inspect CONTAINER | jq '.[].Mounts[] | select(.Type=="bind")'

# Vérifier réseaux personnalisés
docker network ls

# Identifier containers avec ports &lt;1024
docker ps --format "{{.Names}}: {{.Ports}}" | grep ":80\|:443\|:22"
```

Les containers utilisant des ports privilégiés ou montant des répertoires système nécessiteront des ajustements pour tourner rootless.

##### Stratégie migration progressive

Plutôt qu'une big-bang migration, adoptez une approche progressive :

**Phase 1 : Coexistence**

```bash
# Installer Podman sans désinstaller Docker
sudo apt install podman

# Tester containers Podman en parallèle
podman run -d --name test-nginx -p 8081:80 nginx
```

**Phase 2 : Migration par container**

```bash
# Exporter config Docker
docker inspect myapp > myapp-docker.json

# Recréer avec Podman (ajuster ports si nécessaire)
podman run -d --name myapp \
    -p 8080:8080 \
    -v /data:/data:Z \
    myapp:latest

# Tester longuement
# Une fois validé, arrêter version Docker
docker stop myapp
docker rm myapp
```

**Phase 3 : Transition systemd**

```bash
# Générer units systemd pour containers Podman
podman generate systemd --new --name myapp \
    > ~/.config/systemd/user/container-myapp.service

systemctl --user enable container-myapp
systemctl --user start container-myapp
```

**Phase 4 : Désinstaller Docker**

```bash
# Une fois tous containers migrés et validés
sudo apt remove docker-ce docker-ce-cli
sudo rm -rf /var/lib/docker
```

##### Checklist migration

```
□ Identifier containers avec ports &lt;1024 → Ajuster ou activer ports privilégiés
□ Vérifier volumes montés → Assurer permissions utilisateur
□ Tester performance réseau → Évaluer impact slirp4netns
□ Valider build images → Tester compatibilité Buildah
□ Adapter docker-compose.yml si nécessaire
□ Documenter différences comportementales
□ Former équipes aux spécificités Podman
□ Mettre à jour CI/CD pipelines
```

---

#### Sécurité et bonnes pratiques

##### Scan vulnérabilités images

Podman s'intègre avec des scanners de vulnérabilités pour auditer vos images :

```bash
# Installer Trivy
sudo apt install trivy

# Scanner image
trivy image nginx:alpine

# Scanner avec sévérité minimum
trivy image --severity HIGH,CRITICAL myapp:latest

# Format JSON pour automatisation
trivy image -f json -o report.json myapp:latest
```

Intégrez ces scans dans vos pipelines CI/CD pour bloquer les images vulnérables en production.

##### Labels SELinux

Sur systèmes avec SELinux (RHEL, Fedora), utilisez les options de labeling pour renforcer l'isolation :

```bash
# Volumes avec label privé (recommandé)
podman run -v /data:/data:Z myapp

# Z = label privé, seul ce container y accède
# z = label partagé, tous containers peuvent accéder

# Vérifier labels
ls -lZ /data
```

Le `:Z` est central pour éviter les AVC denials SELinux avec volumes rootless.

##### Capabilities et sécurité

Limitez les capabilities des containers pour réduire la surface d'attaque :

```bash
# Lister capabilities par défaut
podman run --rm alpine cat /proc/1/status | grep Cap

# Dropper toutes capabilities
podman run --cap-drop=ALL nginx

# Ajouter seulement capabilities nécessaires
podman run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx
```

En rootless, la plupart des capabilities dangereuses sont déjà inaccessibles, mais cette pratique reste valable.

---

#### Performance et monitoring

##### Optimisations réseau rootless

Le réseau rootless par défaut (slirp4netns) peut être un bottleneck. Pour de meilleures performances, utilisez **pasta** :

```bash
# Pasta offre 2-3x meilleures perfs que slirp4netns
podman run --network=pasta -p 8080:80 nginx
```

Pasta est disponible Podman 4.4+ et devient le défaut sur Fedora 38+.

##### Monitoring containers

Utilisez `podman stats` pour surveiller la consommation ressources :

```bash
# Stats temps réel
podman stats

# Stats container spécifique
podman stats web

# Format personnalisé
podman stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

Pour un monitoring plus avancé, Podman expose des métriques Prometheus :

```bash
# Activer serveur métriques
podman --log-level=info events --stream=false --since=1s

# Métriques accessibles via API REST
curl http://localhost:8080/libpod/stats
```

##### Logs et troubleshooting

Podman intègre journald par défaut, offrant une gestion logs centralisée :

```bash
# Logs via journald
podman logs web
journalctl -u container-web

# Logs avec timestamps
podman logs --timestamps web

# Follow logs
podman logs -f web

# Dernières 100 lignes
podman logs --tail 100 web
```

Pour débugger les problèmes réseau rootless :

```bash
# Tester connectivité depuis container
podman run --rm nicolaka/netshoot ping google.com

# Inspecter réseau container
podman inspect web | jq '.[].NetworkSettings'

# Vérifier slirp4netns
ps aux | grep slirp4netns
```

---

#### Conclusion

Podman représente une évolution majeure dans l'écosystème des containers, apportant sécurité et simplicité opérationnelle. Son architecture daemonless, le mode rootless par défaut et le support natif des pods en font un choix de plus en plus évident pour les environnements d'entreprise.

**Points clés :**

- **Architecture daemonless** : Pas de single point of failure, meilleure isolation
- **Rootless natif** : Sécurité renforcée sans compromis usabilité
- **Compatibilité Docker** : Migration transparente, courbe d'apprentissage minimale
- **Pods Kubernetes-like** : Orchestration locale simplifiée, compatibilité cloud-native
- **Intégration systemd** : Containers comme services système de première classe

**Commandes essentielles :**

```bash
# Démarrage rapide
podman run -d --name web -p 8080:80 nginx

# Gestion pods
podman pod create --name app -p 8080:80
podman run -d --pod app nginx

# Systemd integration
podman generate systemd --new --name web > web.service

# Compose
podman compose up -d
```

Avec Podman 4.x et les évolutions à venir, l'écart avec Docker ne cesse de se creuser en faveur de Podman pour les usages professionnels. Son adoption croissante dans les entreprises (Red Hat, IBM, gouvernements) confirme qu'il est bien plus qu'une alternative : c'est l'avenir des containers sur Linux.