---
link: https://blog.levassb.ovh/post/gitea/
excerpt: Auto-héberger une forge GIT avec Gitea
twitter: https://twitter.com/@levassb
slurped: 2025-09-01T11:48
title: Gitea
tags:
  - gitea
---

Gitea est une forge logicielle GIT écrite en Go dérivée de Gogs.

Comme souvent, au moment de choisir une solution logicielle, je regarde :

- la couverture fonctionnelle
- la vitalité du projet
- les ressources nécessaires (mon VPS est un peu limité)
- la qualité de la documentation

Gitea coche toutes ces cases avec une version Docker pour la facilité de maintenance, SQLite pour limiter les ressources (un seul conteneur) et des fonctionnalités à mi-chemin entre un Gitolite un peu trop “roots” et l’usine à gaz GitLab. Le développement communautaire semble dynamique et la documentation est plutôt bien faite.

Gitea propose un [comparatif des fonctionnalités](https://docs.gitea.io/en-us/comparison/) avec les principales alternatives.

## Installation

Une fois de plus, je vais partir de [ma plateforme](https://blog.levassb.ovh/post/traefik-new-conf/) déjà en place avec Traefik pour gérer les accés HTTPS (reverse-proxy) et SSH (routeur TCP) et Docker pour l’exécution des conteneurs. Je commence par modifier la configuration de Traefik pour ajouter un point d’entrée pour les connexions TCP vers le service SSH du conteneur Gitea:

- `--entrypoints.ssh.address=:2222` : pour créer un point d’entrée `ssh` dans Traefik
- `2222:2222/tcp` : pour ouvrir le port 2222 (le port 22 standard est déjà utilisé sur l’hôte)

```
 1version: '3'
 2services:
 3  traefik:
 4    container_name: traefik
 5    image: traefik:v2.9.6
 6    restart: unless-stopped
 7    command:
 8      - "--providers.docker=true"
 9      - "--providers.docker.exposedbydefault=false"
10      - "--providers.file.directory=/etc/traefik/dynamic-conf"
11      - "--providers.file.watch=true"
12      - "--api.dashboard=true"
13      - "--entrypoints.web.address=:80"
14      - "--entrypoints.websecure.address=:443"
15      - "--entrypoints.ssh.address=:2222"  
16      - "--entrypoints.web.http.redirections.entrypoint.to=websecure"
17      - "--entrypoints.web.http.redirections.entrypoint.scheme=https"
18      - "--certificatesResolvers.letsencrypt.acme.email=admin@domaine.tld"
19      - "--certificatesResolvers.letsencrypt.acme.storage=acme.json"
20      - "--certificatesResolvers.letsencrypt.acme.tlsChallenge=true"
21      - "--log=true"
22      - "--log.level=INFO"
23      - "--log.filepath=/var/log/traefik.log"
24      - "--accesslog.filepath=/var/log/traefix-access.log"
25    labels:
26      - "traefik.enable=true"
27      - "traefik.http.routers.dashboard.rule=Host(`traefik.domaine.tld`)"
28      - "traefik.http.routers.dashboard.service=api@internal"
29      - "traefik.http.routers.dashboard.entrypoints=websecure"
30      - "traefik.http.routers.dashboard.middlewares=auth-dashboard"
31      - "traefik.http.middlewares.auth-dashboard.basicauth.users=admin:#########################"
32      - "traefik.http.routers.dashboard.tls=true"
33      - "traefik.http.routers.dashboard.tls.certresolver=letsencrypt"
34    networks:
35      - traefik_lan
36    ports:
37      - 80:80
38      - 443:443
39      - 2222:2222/tcp
40    volumes:
41      - /var/run/docker.sock:/var/run/docker.sock:ro
42      - ./config/acme.json:/acme.json
43      - ./config:/etc/traefik:ro
44      - logs:/var/log/
45
46networks:
47  traefik_lan:
48    external: true
49
50volumes:
51  logs:
```

> Pour les autres paramètres, je vous laisse jeter un coup d’œil à mon [précédent article.](https://blog.levassb.ovh/post/traefik-new-conf/).

Je crée un compte local `git` qui sera utilisé pour l’exécution des processus du conteneur Gitea :

```
1$ sudo adduser --system --shell /usr/sbin/nologin --group --disabled-password --home /home/git git
2$ id git
3uid=110(git) gid=118(git) groups=118(git)
```

> l’IUD: 110 et le GID: 118 seront utilisés dans le docker-compose de Gitea

J’ajoute un dossier pour contenir le fichier docker-compose de Gitea:

```
1$ sudo mkdir /opt/gitea && cd "$_"
2$ touch docker-compose.yaml
```

J’édite le fichier `docker-compose.yaml`

```
 1version: '3'
 2services:
 3  gitea:
 4    container_name: gitea
 5    image: gitea/gitea:1.18.0
 6    restart: unless-stopped
 7    environment:
 8      - USER_UID=110
 9      - USER_GID=118
10      - RUN_MODE=prod
11      - APP_NAME="My forge!"
12      - GITEA__server__SSH_PORT=2222
13      - GITEA__server__SSH_LISTEN_PORT=22
14      - GITEA__server__HTTP_PORT=3000
15      - GITEA__server__ROOT_URL=https://git.domain.tld
16      - GITEA__database__DB_TYPE=sqlite3
17      - GITEA__service__DISABLE_REGISTRATION=true
18      - GITEA__service__REQUIRE_SIGNIN_VIEW=true
19      - GITEA__service__REGISTER_EMAIL_CONFIRM=true  
20      - GITEA__mailer__ENABLED=true
21      - GITEA__mailer__SMTP_ADDR=smtp.domain.tld
22      - GITEA__mailer__SMTP_PORT=587
23      - GITEA__mailer__PROTOCOL=smtp+starttls
24      - GITEA__mailer__USER=admin@domain.tld
25      - GITEA__mailer__PASSWD=V3ryS3cur3
26      - GITEA__mailer__FROM=noreply-gitea@domain.tld
27    expose:
28      - "3000"
29      - "22"
30    networks:
31      - traefik_lan
32    volumes:
33      - data:/data
34      - /etc/timezone:/etc/timezone:ro
35      - /etc/localtime:/etc/localtime:ro
36    labels:
37      - "traefik.enable=true"
38      - "traefik.http.services.gitea.loadbalancer.server.port=3000"
39      - "traefik.docker.network=traefik_lan"
40      - "traefik.http.routers.gitea.rule=Host(`git.domain.tld`)"
41      - "traefik.http.routers.gitea.entrypoints=websecure"
42      - "traefik.http.routers.gitea.middlewares=secured@file"
43      - "traefik.http.routers.gitea.tls=true"
44      - "traefik.http.routers.gitea.tls.certresolver=letsencrypt"
45      - "traefik.tcp.routers.gitea-ssh.rule=HostSNI(`*`)"
46      - "traefik.tcp.routers.gitea-ssh.entrypoints=ssh"
47      - "traefik.tcp.routers.gitea-ssh.service=gitea-ssh-svc"
48      - "traefik.tcp.services.gitea-ssh-svc.loadbalancer.server.port=22"
49networks:
50  traefik_lan:
51    external: true
52volumes:
53  data:
```

Dans cette configuration:

- les inscriptions sont désactivées (l’administrateur doit créer les comptes)
- l’accès est soumis à une authentification systématique (pas d’accès libre à un dépôt public)
- le courriel d’enregistrement doit être validé pour accéder à la forge
- les données sont persistées dans un volume nommé Docker (`/var/lib/docker/volume/gitea_data/`)
- la redirection HTTP vers HTTPS est gérée en amont directement dans Traefik
- les connexions TCP sont redirigées du point d’entrée `ssh` de Traefik vers le port TCP 22 du conteneur

> Gitea propose de nombreux paramètres de configurations. Vous devriez trouver votre bonheur dans [la documentation](https://docs.gitea.io/fr-fr/config-cheat-sheet/) en respectant la convention de nommage des variables d’environnement : [GITEA__SECTION__KEY-NAME](https://github.com/go-gitea/gitea/tree/main/contrib/environment-to-ini)

Il ne reste plus qu’à démarrer le conteneur :

```
1$ docker-compose -f /opt/gitea/docker-compose.yaml up -d
```

et à finaliser l’installation dans l’interface web :

![Gitea Install](https://blog.levassb.ovh/images/gitea-install-sc.png)

> Il faut impérativement créer le compte administrateur puisque les inscriptions sont fermées dans cette configuration.

A ce stade, Gitea est opérationnel. Vous pouvez maintenant:

- créer un compte utilisateur
- ajouter une clé publique SSH
- créer un premier dépôt

et tester les connexions SSH :

```
 1$ mkdir ~/projet && cd "$_"
 2$ touch README.md
 3$ git init
 4$ git checkout -b main
 5$ git add README.md
 6$ git commit -m "first commit"
 7$ git remote add origin ssh://git@git.domain.tld:2222/johndoe/demo.git
 8$ git push -u origin main
 9$ git push origin main
10Énumération des objets: 3, fait.
11Décompte des objets: 100% (3/3), fait.
12Écriture des objets: 100% (3/3), 219 octets | 219.00 Kio/s, fait.
13Total 3 (delta 0), réutilisés 0 (delta 0), réutilisés du pack 0
14remote: . Processing 1 references
15remote: Processed 1 references in total
16To ssh://git.domain.tld:2222/johndoe/demo.git
17 * [new branch]      main -> main
```

![Gitea repo](https://blog.levassb.ovh/images/gitea-repo.png)

## Conclusion

Gitea offre une solution clé en main élégante et facile à mettre en oeuvre. Peu gourmande en ressources (moins de 200Mo de RAM et une image à 257Mo), elle est particulièrement bien adaptée aux contraintes de l’auto-hébergement.

## Ressources

- [documentation Gitea](https://docs.gitea.io/fr-fr/)
- [config Cheat Sheet (en)](https://docs.gitea.io/fr-fr/config-cheat-sheet/)
- [forum Traefik](https://community.traefik.io/t/ssh-gitea-with-traefik-v2/14525)
- [tutorial DigitalOcean (en)](https://www.digitalocean.com/community/tutorials/how-to-install-gitea-on-ubuntu-using-docker)
- [blog de Richard Dern](https://www.richard-dern.fr/blog/2021/09/12/deployer-hugo-via-gitea-et-drone-ci/)
- [blog de Romain Boulanger](https://blog.filador.fr/gitea-le-gestionnaire-de-code-source-leger-et-simple-a-mettre-en-oeuvre-pour-votre-raspberry/)

