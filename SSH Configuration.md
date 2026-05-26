---
tags:
  - ssh
  - tunnel
  - configuration
---
Vous faites `ssh user@server`. Vous êtes dedans. Vous pensez que vous connaissez SSH. En réalité, vous utilisez 5 % de ce que SSH peut faire. SSH n'est pas juste un protocole de connexion distante, c'est un outil de tunneling réseau complet. Vous pouvez accéder à une base de données distante sans exposer son port, traverser plusieurs pare-feux avec une seule commande, ou créer un proxy SOCKS entier depuis votre terminal. Ce guide couvre tout ça, avec des exemples concrets, pas de théorie creuse.

#### Prérequis

- SSH client installé (par défaut sur macOS et Linux, PuTTY ou Windows SSH sur Windows 10+)
- Au moins un serveur SSH accessible
- Connaissance de base de SSH (`ssh`, `scp`, `ssh-keygen`)
- Pour les parties avancées : un ou plusieurs serveurs avec des rôles différents (bastion, app server, DB server)

#### Fondation : ~/.ssh/config

Avant tout le reste, configurez votre fichier `~/.ssh/config`. C'est la colonne vertébrale de tout ce qui suit. Si vous n'utilisez pas ce fichier, chaque commande SSH devient une ligne de 200 caractères que vous devez retaper à chaque fois.

```bash
# ~/.ssh/config

# Serveur de développement
Host dev
    HostName 203.0.113.10
    User     deploy
    Port     2222
    IdentityFile ~/.ssh/id_ed25519_dev

# Bastion / Jump host
Host bastion
    HostName bastion.example.com
    User     ops
    IdentityFile ~/.ssh/id_ed25519_bastion

# Serveur app — accessible uniquement via le bastion
Host app-server
    HostName 10.0.1.50
    User     app
    ProxyJump bastion
    IdentityFile ~/.ssh/id_ed25519_internal

# Serveur DB — même réseau interne
Host db-server
    HostName 10.0.1.60
    User     dbadmin
    ProxyJump bastion
    IdentityFile ~/.ssh/id_ed25519_internal

# Règle par défaut pour tous les serveurs
Host *
    IdentityFile        ~/.ssh/id_ed25519
    AddKeysToAgent      yes
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

Avec cette config, la connexion à votre app server en dehors du réseau interne devient simplement :

```bash
ssh app-server
```

Au lieu de :

```bash
ssh -i ~/.ssh/id_ed25519_bastion -o ProxyCommand="ssh -W %h:%p -i ~/.ssh/id_ed25519_bastion ops@bastion.example.com" app@10.0.1.50
```

#### Clés Ed25519 : pourquoi et comment

RSA est toujours utilisé par beaucoup de gens. Ed25519 est plus court, plus rapide, plus sécurisé.

```bash
# Générer une clé Ed25519
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_production -C "user@production"

# Copier la clé publique sur le serveur distant
ssh-copy-id -i ~/.ssh/id_ed25519_production user@server
```

```
# Vérifier l'empreinte de la clé
ssh-keygen -lf ~/.ssh/id_ed25519_production.pub
256 SHA256:xYz... (user@production) ED25519 (SSH2)
```

#### SSH Agent : ne plus entrer votre passphrase

Si vous utilisez des clés avec passphrase (vous devriez), l'agent SSH vous évite de la retaper à chaque connexion.

```bash
# Démarrer l'agent (souvent fait automatiquement par votre shell)
eval "$(ssh-agent -s)"

# Ajouter votre clé à l'agent
ssh-add ~/.ssh/id_ed25519_production

# Vérifier les clés chargées
ssh-add -l
```

```
256 SHA256:xYz... user@production (ED25519)
```

**Agent forwarding** : quand vous êtes connecté sur un serveur distant via SSH, vous pouvez, depuis ce serveur, vous connecter à un autre serveur en utilisant votre agent local. Aucune clé privée ne transit sur le réseau.

```bash
# Connexion avec forwarding de l'agent
ssh -A bastion

# Depuis bastion, vous pouvez maintenant faire :
ssh app-server    # utilise votre agent local
```

**Attention** : n'activez `-A` que sur des serveurs auxquels vous faites confiance. Un serveur compromis avec agent forwarding peut se connecter à n'importe quel serveur auquel vous avez accès.

#### Tunneling local : exposer un port distant en local

C'est le cas d'usage le plus courant après la connexion distante. Vous voulez accéder à un service qui tourne sur un serveur distant, mais qui n'est pas exposé publiquement.

##### Exemple : accéder à une base de données PostgreSQL distante

```bash
# La DB tourne sur db-server:5432, mais le port n'est pas ouvert vers l'extérieur
# Vous créez un tunnel local : localhost:5433 → db-server:5432

ssh -L 5433:db-server:5432 bastion

# Dans un autre terminal, connectez-vous normalement :
psql -h localhost -p 5433 -U dbuser -d mydb
```

##### Exemple : accéder à un interface admin web interne

```bash
# Monitoring interne sur app-server:9090, pas exposé
ssh -L 9090:app-server:9090 bastion

# Ouvrez votre navigateur : http://localhost:9090
```

##### Rendre un tunnel permanent avec `-N` et `-f`

```bash
# -N : pas de shell distant (on veut juste le tunnel)
# -f : mettre en arrière-plan
ssh -N -f -L 5433:db-server:5432 bastion
```

Le tunnel tourne en arrière-plan. Pour le tuer :

```bash
# Trouver le processus
ps aux | grep "ssh.*5433"

# Tuer par PID
kill <PID>
```

#### Tunneling distant : exposer un port local vers le monde extérieur

C'est l'inverse du tunnel local. Vous êtes derrière un pare-feu et vous voulez exposer un service qui tourne sur votre machine.

```bash
# Vous êtes sur votre machine locale avec un serveur web sur le port 8080
# Vous voulez l'exposer via votre serveur public sur le port 443

ssh -R 443:localhost:8080 user@public-server

# Le port 443 du public-server pointe maintenant vers votre localhost:8080
```

**Attention importante** : par défaut, le port distant 443 n'écoutera que sur 127.0.0.1 (localhost du serveur), pas sur toutes les interfaces (0.0.0.0). Cela signifie que seules les connexions locales au serveur peuvent accéder au tunnel. Pour que le port soit accessible depuis l'extérieur, le serveur distant doit avoir `GatewayPorts yes` ou `GatewayPorts clientspecified` dans sa configuration sshd_config. Vérifiez avec votre administrateur ou mettez à jour `/etc/ssh/sshd_config` sur le serveur public.

Cas d'usage concret : vous développez localement et vous voulez que quelqu'un puisse tester votre appli via une URL publique. Sans ngrok, sans rien, juste SSH.

#### Tunneling dynamique : SOCKS proxy SSH

C'est ici où SSH devient vraiment puissant. Un tunnel dynamique crée un proxy SOCKS sur votre machine. Tout le trafic qui passe par ce proxy sort via le serveur SSH distant.

```bash
# Créer un proxy SOCKS5 sur le port 1080, via votre serveur distant
ssh -D 1080 -N user@remote-server

# Configurez votre navigateur ou vos outils pour utiliser localhost:1080 comme proxy SOCKS5
```

##### Utiliser curl avec le proxy SOCKS

```bash
# Vérifier votre IP vue depuis le serveur distant
curl --socks5 localhost:1080 https://ifconfig.me
```

##### Utiliser avec git

```bash
# Clone un repo via le proxy SOCKS
GIT_PROXY_COMMAND="ssh -W %h:%p user@remote" git clone git@github.com:user/repo.git
```

#### ProxyJump : chaîner des serveurs SSH

`ProxyJump` (ou `-J` en ligne de commande) vous permet de traverser plusieurs serveurs SSH en une seule commande. C'est la réponse moderne à `ProxyCommand`.

```bash
# Connexion directe via un bastion
ssh -J bastion app-server

# Chaîne de deux bastions
ssh -J bastion1,bastion2 app-server

# Équivalent dans ~/.ssh/config (déjà configuré plus haut)
ssh app-server
```

##### Architecture typique : bastion multi-hop

```
Internet ──► bastion (IP publique) ──► app-server (10.0.1.50) ──► db-server (10.0.1.60)
```

```bash
# Accès direct à la DB depuis votre machine locale
ssh -J bastion,app-server db-server
```

Une seule commande. Trois hops. Aucune clé privée ne quitte votre machine.

#### Exemple complet : accéder à une DB distante sans exposer le port

Scénario réel : vous avez un cluster avec un bastion, un app server, et une base PostgreSQL. La DB n'a aucun port ouvert vers l'extérieur, elle n'écoute que sur localhost ou sur le réseau interne.

```bash
# Étape 1 : créer un tunnel local vers la DB via le bastion et l'app server
ssh -L 5433:localhost:5432 -J bastion app-server &

# Étape 2 : connecter-vous à la DB via le tunnel
psql -h localhost -p 5433 -U appuser -d production

# Étape 3 : pour arrêter le tunnel
kill %1
```

La DB n'est jamais exposée publiquement. Le tunnel existe uniquement pendant votre session. Zéro risque d'exposition.

#### Hardening rapide : les bonnes pratiques

Pour un hardening SSH complet, consultez le guide [sécuriser SSH](https://www.shpv.fr/blog/ssh-securisation/).

```bash
# /etc/ssh/sshd_config — les paramètres essentiels

# Désactiver l'authentification par mot de passe
PasswordAuthentication no

# Désactiver le login root
PermitRootLogin no

# Changer le port (ça n'est pas de la sécurité, c'est du bruit réduit dans les logs)
Port 2222

# Limiter les utilisateurs autorisés
AllowUsers deploy ops

# Timeout des sessions inactives (en secondes)
ClientAliveInterval 300
ClientAliveCountMax 3
```

```bash
# Redémarrer le service après modification
sudo systemctl restart sshd
```

#### Tableau récapitulatif

|Fonctionnalité|Commande|Usage|
|---|---|---|
|Tunnel local|`ssh -L local_port:host:remote_port server`|Accéder à un service distant depuis votre machine|
|Tunnel distant|`ssh -R remote_port:host:local_port server`|Exposer un service local vers l'extérieur|
|Proxy SOCKS|`ssh -D port -N server`|Proxy réseau complet via un serveur distant|
|ProxyJump|`ssh -J hop1,hop2 target`|Chaîner plusieurs serveurs SSH|
|Agent forwarding|`ssh -A server`|Utiliser votre agent SSH depuis un serveur distant|
|Config host|`~/.ssh/config`|Simplifier toutes les connexions|

#### Conclusion

SSH fait beaucoup plus que vous ne le pensez. Un tunnel local pour toucher une DB qui n'est pas exposée, un SOCKS proxy pour faire passer tout votre trafic par un serveur distant, un ProxyJump pour traverser trois hops en une commande, tout ça est natif, sans installer quoi que ce soit.

Pour les déploiements en production, enrichissez cette fondation avec des [certificats OpenSSH](https://www.shpv.fr/blog/openssh-ca/) pour une gestion centralisée ou un [bastion multi-facteurs](https://www.shpv.fr/blog/ssh-bastion-mfa/).

La prochaine étape ? Automatisez ces tunnels avec systemd user services ou avec des scripts d'entrée. Et si vous gérez une infra multi-datacenter, regardez du côté de WireGuard, mais SSH reste votre premier outil d'urgence, partout, toujours.