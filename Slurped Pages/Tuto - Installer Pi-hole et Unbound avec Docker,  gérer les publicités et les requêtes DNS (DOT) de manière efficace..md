---
link: https://dolys.fr/forums/topic/tuto-installer-pi-hole-et-unbound-avec-docker-gerer-les-publicites-et-les-requetes-dns-de-maniere-efficace/
byline: Auteur
site: l'Almanet doLys de nam1962 et ses amis
excerpt: Introduction Je vais vous montrer ici comment mettre en place Pi-hole, un serveur DNS qui bloque les publicités, en utilisant Docker, tout en configurant Unbound comme résolveur DNS récursif local avec DNS-over-TLS (DoT). Cette configuration vous permettra de naviguer sur Internet sans être dérangé par les publicités indésirables. DNS-over-TLS (DoT)…
twitter: https://twitter.com/https://twitter.com/e_dolys
slurped: 2026-03-29T16:32
title: "Tuto: Installer Pi-hole et Unbound avec Docker,  gérer les publicités et les requêtes DNS (DOT) de manière efficace. | l'Almanet doLys de nam1962 et ses amis"
tags:
  - pi-hole
  - unbbound
  - docker
  - dns
---

### Introduction

Je vais vous montrer ici comment mettre en place Pi-hole, un serveur DNS qui bloque les publicités, en utilisant Docker, tout en configurant Unbound comme résolveur DNS récursif local avec DNS-over-TLS (DoT). Cette configuration vous permettra de naviguer sur Internet sans être dérangé par les publicités indésirables.

DNS-over-TLS (DoT) est une méthode qui chiffre vos requêtes DNS en utilisant le protocole TLS sur un port dédié (853). C’est comme si vos requêtes DNS voyageaient dans une enveloppe scellée que seul le destinataire peut ouvrir. Cette méthode offre un excellent équilibre entre sécurité et contrôle, car vous gérez vous-même toute la chaîne de résolution DNS de manière chiffrée.

Mais ce n’est pas tout ! En contrôlant votre propre infrastructure DNS, vous devenez souverain de vos données. Vous évitez ainsi plusieurs types de menaces courantes :

– Les indiscrétions : Vos requêtes DNS sont souvent interceptées et analysées par des tiers. Avec votre propre serveur, vous gardez le contrôle de ces informations sensibles.  
– Les DNS menteurs : Les FAI sont sous contrôle des administrations et censurent, redirigent, surveillent les requêtes. Votre serveur DNS personnel préserve l’intégrité de vos connexions.  
– Les scripts indiscrets : De nombreux sites web chargent des scripts aux actions diverses et jamais utiles pour vous. Pi-hole peut bloquer ces scripts, renforçant ainsi votre confidentialité en ligne.

Cette configuration donne donc un triple avantage : elle bloque les publicités, améliore votre sécurité et renforce votre souveraineté numérique.

Astuce : si vous hésitez entre DoT et [DoH (que je présente dans un autre tuto)](https://dolys.fr/forums/topic/tuto-installer-une-stack-dns-over-https-avec-pihole-et-dnscrypt-proxy/), voici un [petit switch pour passer de l’un à l’autre](https://dolys.fr/forums/topic/astuce-un-petit-script-pour-alterner-entre-vos-deux-stacks-dns/).

### Prérequis

Avant de commencer, assurez-vous d’avoir :

– Un système Linux à jour (Manjaro, EndeavourOS, Debian, Linux Mint…, etc.)  
– Docker et Docker Compose installés  
– Accès à un terminal avec des privilèges sudo

Ps, je ne parle pas d’Ubuntu, ni même de Xubuntu que j’ai longtemps utilisé car snapd est banni de mes installs)

### Étape 1 : Installer Docker et Docker Compose

Si vous n’avez pas encore installé Docker et Docker Compose, voici comment procéder :

#### Pour Arch Linux et dérivés (Manjaro, EndeavourOS, Garuda)

```
# Installation de Docker et Docker Compose
sudo pacman -S docker docker-compose

# Démarrage et activation du service Docker
sudo systemctl start docker
sudo systemctl enable docker

# Ajout de votre utilisateur au groupe docker (évite de devoir utiliser sudo)
sudo usermod -aG docker $USER
```

#### Pour Debian/Linux Mint

```
# Installation des prérequis
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Ajout de la clé GPG et du dépôt Docker
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Pour Mint basé sur Ubuntu, remplacez la ligne ci-dessus par celle-ci :
# echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu focal stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Installation des packages Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

# Démarrage et activation du service Docker
sudo systemctl start docker
sudo systemctl enable docker

# Ajout de votre utilisateur au groupe docker
sudo usermod -aG docker $USER
```

Après avoir ajouté votre utilisateur au groupe docker, déconnectez-vous et reconnectez-vous pour que les changements prennent effet.

### Étape 2 : Créer la structure de répertoires

Je vais créer un répertoire pour mon projet et y ajouter les fichiers nécessaires :

```
mkdir -p ~/docker/dns-dot/unbound
cd ~/docker/dns-dot
```

### Étape 3 : Créer le fichier docker-compose.yml

Dans le répertoire ~/docker/dns-dot, je crée un fichier docker-compose.yml :

```
nano docker-compose.yml
```

### Validation de la syntaxe YAML

Le fichier `docker-compose.yml` doit avoir une indentation correcte (2 espaces par niveau, pas de tabulations) pour éviter les erreurs avec Docker Compose. Si vous copiez le code depuis ce tutoriel, l’affichage du forum peut déformer l’indentation. Vérifiez la syntaxe avec [YAML Lint](http://www.yamllint.com/) en ligne ou installez `yamllint` localement.

Pour installer `yamllint` sur Debian/Ubuntu :

```
sudo apt update
sudo apt install yamllint
yamllint ~/docker/dns-dot/docker-compose.yml
```

Pour installer `yamllint` sur Arch Linux :

```
sudo pacman -S yamllint
yamllint ~/docker/dns-dot/docker-compose.yml
```

Si des erreurs sont détectées, ajustez l’indentation dans un éditeur comme `nano` ou VS Code, puis revalidez. Consultez [la spécification YAML](https://yaml.org/spec/1.2/spec.html#id2760395) pour plus de détails sur l’indentation.

Contenu à copier :

```
services:
  unbound:
    container_name: unbound
    image: mvance/unbound:latest
    restart: unless-stopped
    hostname: unbound
    volumes:
      - './unbound:/opt/unbound/etc/unbound/custom.conf.d'
    networks:
      pihole_net:
        ipv4_address: 172.22.0.2
    ports:
      - "5335:53/tcp"
      - "5335:53/udp"
    healthcheck:
      test: ["CMD-SHELL", "drill @172.22.0.2 pi.hole || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
    mem_limit: 1024m
    cpus: '1.0'
    mem_reservation: 512m
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    restart: unless-stopped
    hostname: pihole
    depends_on:
      - unbound
    dns:
      - 172.22.0.2
    env_file:
      - .env
    volumes:
      - './etc-pihole:/etc/pihole'
      - './etc-dnsmasq.d:/etc/dnsmasq.d'
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    networks:
      pihole_net:
        ipv4_address: 172.22.0.3
    cap_add:
      - NET_ADMIN
    healthcheck:
      test: ["CMD-SHELL", "dig @127.0.0.1 pi.hole || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3
    mem_limit: 1024m
    cpus: '2.0'
    mem_reservation: 512m
networks:
  pihole_net:
    ipam:
      config:
        - subnet: 172.22.0.0/24
```

### Étape 4 : Créer le fichier .env

Je vais maintenant créer un fichier .env pour stocker mes variables d’environnement :

```
nano .env
```

Contenu à copier (remplacez votremotdepasse par votre mot de passe à vous !) :

```
WEBPASSWORD=votremotdepasse
TZ=Europe/Paris
DNS1=172.22.0.2#53
DNS2=no
```

### Étape 5 : Configurer Unbound avec DNS-over-TLS

Je vais maintenant configurer Unbound pour utiliser DNS-over-TLS, ce qui chiffre les requêtes DNS et aide à contourner la censure des FAI. J’ouvre le fichier de configuration d’Unbound :

```
nano ~/docker/dns-dot/unbound/custom.conf
```

J’ajoute les lignes suivantes pour configurer Unbound :

```
server:
  # Paramètres réseau
  interface: 0.0.0.0
  interface: ::0
  port: 53 # Port interne du conteneur, mappé à 5335 sur l'hôte
  do-ip4: yes
  do-ip6: yes # Conservé pour contourner les restrictions FAI via IPv6
  do-udp: yes
  do-tcp: yes

  # Contrôle d'accès
  access-control: 127.0.0.0/8 allow
  access-control: ::1/128 allow
  access-control: 172.22.0.0/24 allow # Réseau Docker pihole_net
  access-control: 192.168.0.0/16 allow
  access-control: 10.0.0.0/8 allow

  # Paramètres de base
  verbosity: 1
  num-threads: 4
  outgoing-range: 16384
  so-rcvbuf: 8m
  so-sndbuf: 8m
  edns-buffer-size: 1472
  msg-cache-size: 400m
  rrset-cache-size: 800m
  cache-min-ttl: 60
  cache-max-ttl: 86400
  cache-max-negative-ttl: 60
  do-not-query-localhost: no

  # Confidentialité et sécurité
  hide-identity: yes
  hide-version: yes
  harden-glue: yes
  harden-dnssec-stripped: yes
  harden-algo-downgrade: yes
  harden-short-bufsize: yes
  use-caps-for-id: yes
  qname-minimisation: yes
  qname-minimisation-strict: yes
  minimal-responses: yes

  # Paramètres DNSSEC
  root-hints: root.hints
  trust-anchor-file: trusted-key.key
  auto-trust-anchor-file: "root.key"

  # Paramètres TLS pour DNS chiffré
  tls-cert-bundle: "/etc/ssl/certs/ca-certificates.crt"
  tls-ciphersuites: "TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256"
  tls-ciphers: "HIGH:!aNULL:!MD5:!3DES"

  # Optimisations de performance
  prefetch: yes
  prefetch-key: yes
  serve-expired: yes
  aggressive-nsec: yes
  ratelimit: 1000

  # Configuration des modules
  module-config: "iterator"

# Zone de transfert pour Pi-hole
forward-zone:
  name: "pi.hole"
  forward-addr: 172.22.0.3@53

# Zone de transfert principale avec serveurs DNS-over-TLS (DoT) uniquement
forward-zone:
  name: "."
  forward-tls-upstream: yes

  # Europe (Luxembourg, Suisse - neutres et respectueux de la vie privée)
  forward-addr: 45.67.222.155@853#dns0.eu        # dns0.eu (Luxembourg, vie privée stricte, no-log)
  forward-addr: 95.216.161.189@853#doh-ch.blahdns.com  # BlahDNS (Suisse, neutre, no-log)
  forward-addr: 185.213.26.187@853#dns.digitale-gesellschaft.ch  # Digitale Gesellschaft (Suisse, ONG vie privée)
  forward-addr: 185.169.152.15@853#dns2.digitalcourage.de # Artikel10 (relay NL, neutre, pas de filtrage)

  # Danemark (hors 14-Eyes)
  forward-addr: 89.233.43.71@853#unicast.uncensoreddns.org    # UncensoredDNS (Danemark, non filtré, no-log)

  # Asie (hors influence occidentale)
  forward-addr: 101.101.101.101@853#dns.twnic.tw    # TWNIC (Taiwan, neutre)

  # Europe du Sud (hors 14-Eyes)
  forward-addr: 116.202.176.26@853#dot.libredns.gr  # LibreDNS (Grèce, neutre, no-log, stabilité variable)
```

Cette configuration utilise exclusivement DNS-over-TLS (DoT) pour chiffrer les requêtes DNS, ce qui permet de contourner efficacement la censure des FAI, d’éviter les fuites de données et d’améliorer considérablement la confidentialité de vos requêtes DNS.

### Étape 6 : Optimisation de dnsmasq pour Pi-hole

Pour éviter les problèmes de limitations de requêtes DNS, créons un fichier de configuration personnalisé :

```
# Créer un répertoire pour les configurations personnalisées
mkdir -p ~/docker/dns-dot/etc-dnsmasq.d

# Augmenter la limite de requêtes DNS simultanées
echo "dns-forward-max=300" > ~/docker/dns-dot/etc-dnsmasq.d/99-custom.conf
```

### Étape 7 : Démarrer les conteneurs

Maintenant que tout est configuré, je vais démarrer les conteneurs. Je m’assure d’être dans le répertoire dns-dot, puis j’exécute :

```
cd ~/docker/dns-dot
docker compose up -d
```

Cela va démarrer Pi-hole et Unbound en arrière-plan, et ils démarreront automatiquement au redémarrage du système.

### Étape 8 : Configuration de votre système pour utiliser Pi-hole

Pour que votre système utilise Pi-hole pour les requêtes DNS, plusieurs méthodes sont possibles :

#### Méthode 1 : Configuration manuelle de resolv.conf (toutes distributions)

```
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
# Pour éviter que le fichier ne soit écrasé
sudo chattr +i /etc/resolv.conf
```

#### Méthode 2 : Pour les distributions basées sur Arch utilisant NetworkManager

```
# Configurer NetworkManager pour ne pas modifier resolv.conf
sudo mkdir -p /etc/NetworkManager/conf.d/
echo -e "[main]\ndns=none" | sudo tee /etc/NetworkManager/conf.d/dns.conf

# Redémarrer NetworkManager
sudo systemctl restart NetworkManager

# Configurer resolv.conf
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
```

#### Méthode 3 : Pour les distributions basées sur Debian utilisant resolvconf

```
# Installer resolvconf si ce n'est pas déjà fait
sudo apt install -y resolvconf

# Configurer les nameservers
echo "nameserver 127.0.0.1" | sudo tee /etc/resolvconf/resolv.conf.d/head

# Appliquer les changements
sudo resolvconf -u
```

### Étape 9 : Vérification de l’installation

Pour vérifier que tout fonctionne correctement, je teste la résolution DNS :

```
dig kernel.org
```

Si vous obtenez une réponse, tout fonctionne correctement !

Je peux aussi accéder à l’interface web de Pi-hole à l’adresse [http://localhost/admin](http://localhost/admin) et me connecter avec le mot de passe défini dans le fichier .env.

### Étape 10 : Test de validation DNSSEC

Une des fonctionnalités importantes d’Unbound est sa capacité à valider DNSSEC, qui protège contre les attaques d’empoisonnement DNS. Vérifions si cela fonctionne correctement :

```
dig +dnssec dnssec-deployment.org @127.0.0.1
```

Vous devriez voir des enregistrements RRSIG dans la réponse, ce qui confirme que DNSSEC fonctionne.  

### Étape 11 : Ajouter des listes de blocage recommandées

Par défaut, Pi-hole est configuré avec une liste de blocage de base. Pour améliorer le blocage des publicités et des traqueurs, vous pouvez ajouter des listes supplémentaires via l’interface web :

1. Accédez à [http://localhost/admin](http://localhost/admin)  
2. Allez dans « Group Management » > « Adlists »  
3. Ajoutez ces listes recommandées :

```
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
https://raw.githubusercontent.com/crazy-max/WindowsSpyBlocker/master/data/hosts/spy.txt
https://s3.amazonaws.com/lists.disconnect.me/simple_tracking.txt
https://gitlab.com/quidsup/notrack-blocklists/raw/master/notrack-malware.txt
https://adaway.org/hosts.txt
https://v.firebog.net/hosts/Easylist.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/light.txt
https://small.oisd.nl/
```

4. Après avoir ajouté les listes, allez dans « Tools » > « Update Gravity » pour mettre à jour les listes de blocage.

### Bonus : Script de fallback DNS

Pour éviter de perdre la connectivité en cas de panne DNS locale, ajoutez ce script à votre crontab :

```
mkdir -p ~/bin
nano ~/bin/dns-fallback-check.sh
```

```
#!/bin/bash
# Fichier de log
LOGFILE="$HOME/dns-check.log"

# Configuration DNS
LOCAL_DNS="127.0.0.1"
# DNS fallbacks hors 14-Eyes et discrets
FALLBACK_DNS1="193.110.81.0" # dns0.eu (Allemagne)
FALLBACK_DNS2="116.202.176.26" # LibreDNS (Allemagne)

# Test du DNS local
if ! dig +time=3 +tries=1 debian.org @$LOCAL_DNS > /dev/null 2>&1; then
  echo "[dns-fallback] $(date): DNS local indisponible, fallback activé" >> "$LOGFILE"
  echo -e "nameserver $FALLBACK_DNS1\nnameserver $FALLBACK_DNS2" | sudo tee /etc/resolv.conf > /dev/null

  # Notification optionnelle (si notify-send est installé)
  if command -v notify-send > /dev/null; then
    notify-send "DNS Fallback" "Serveurs DNS alternatifs activés" --icon=network-server
  fi
else
  # Vérifier si on est en mode fallback et si le DNS local est revenu en ligne
  if ! grep -q "$LOCAL_DNS" /etc/resolv.conf; then
    echo "[dns-fallback] $(date): DNS local restauré" >> "$LOGFILE"
    echo -e "nameserver $LOCAL_DNS" | sudo tee /etc/resolv.conf > /dev/null

    # Notification optionnelle
    if command -v notify-send > /dev/null; then
      notify-send "DNS restauré" "Service DNS local rétabli" --icon=network-server
    fi
  fi
fi
```

```
chmod +x ~/bin/dns-fallback-check.sh
crontab -e
```

Puis ajoutez cette ligne pour exécuter le script toutes les 5 minutes :

```
*/5 * * * * ~/bin/dns-fallback-check.sh
```

### Conclusion

Voilà, j’ai maintenant un système Pi-hole fonctionnant avec Unbound et DNS-over-TLS, prêt à bloquer les publicités et à gérer mes requêtes DNS de manière sécurisée et souveraine. Cette configuration offre plusieurs avantages :

– Blocage des publicités et des traqueurs  
– Protection de la vie privée grâce aux DNS chiffrés (DNS-over-TLS)  
– Validation DNSSEC pour se protéger contre les attaques d’empoisonnement DNS  
– Contournement de la censure des FAI  
– Contrôle total sur mes requêtes DNS  
– Solution portée par Docker, facile à déployer sur n’importe quelle distribution Linux

Si vous souhaitez comparer cette approche avec la méthode [DNS-over-HTTPS (DoH)](https://dolys.fr/forums/topic/installer-pi-hole-avec-dns-over-https-doh-et-dnscrypt-proxy-sous-docker-bloquer-les-publicites-et-interroger-des-resolveurs-souverains/), n’hésitez pas à consulter mon autre tutoriel dédié, ainsi que le script de basculement qui permet de passer facilement d’une méthode à l’autre.

J’espère que ce tutoriel vous a été utile. N’hésitez pas à partager vos expériences et vos questions dans les commentaires !

---

### Notes supplémentaires

– Documentation officielle de Pi-hole : [https://docs.pi-hole.net/](https://docs.pi-hole.net/)  
– Documentation officielle d’Unbound : [https://www.nlnetlabs.nl/documentation/unbound/](https://www.nlnetlabs.nl/documentation/unbound/)

### Vérification et dépannage

Pour vérifier que votre configuration DNS-over-TLS fonctionne correctement et comprendre comment interpréter les résultats des tests de fuite DNS qui peuvent parfois sembler contradictoires avec votre setup, consultez notre [guide complet de vérification et dépannage DNS](https://dolys.fr/forums/topic/comprendre-les-tests-de-fuite-dns-et-depannage-dns-dot-et-dns-doh/).

Un jeune site que j'aime bien, la ferrari du T-shirt ...bio en plus : [GoudronBlanc](https://goudronblanc.com/)