---
link: https://dolys.fr/forums/topic/installer-pi-hole-avec-dns-over-https-doh-et-dnscrypt-proxy-sous-docker-bloquer-les-publicites-et-interroger-des-resolveurs-souverains/
byline: Auteur
site: l'Almanet doLys de nam1962 et ses amis
excerpt: "Introduction Je vous propose ici une alternative à la résolution DNS récursive locale avec Unbound : une stack 100% DNS-over-HTTPS, souveraine, chiffrée, et sans fuite. Grâce à Pi-hole et dnscrypt-proxy, je peux résoudre toutes mes requêtes DNS via des serveurs DoH de confiance (ni Google, ni Cloudflare, ni FAI...), tout…"
twitter: https://twitter.com/https://twitter.com/e_dolys
slurped: 2026-03-29T16:43
title: "Installer Pi-hole avec DNS-over-HTTPS (DoH) et dnscrypt-proxy sous Docker : bloquer les publicités et interroger des résolveurs souverains | l'Almanet doLys de nam1962 et ses amis"
tags:
  - pi-hole
  - dns
  - docker
---

### Introduction

Je vous propose ici une alternative à la résolution DNS récursive locale avec Unbound : une stack 100% DNS-over-HTTPS, souveraine, chiffrée, et sans fuite.

Grâce à Pi-hole et dnscrypt-proxy, je peux résoudre toutes mes requêtes DNS via des serveurs DoH de confiance (ni Google, ni Cloudflare, ni FAI…), tout en profitant du blocage publicitaire offert par Pi-hole. Le tout encapsulé dans Docker, pour garder un système propre, léger et portable.

Contrairement au DNS traditionnel ou même au DoT (DNS over TLS), le DoH (DNS over HTTPS) fait passer vos requêtes DNS dans le même flux que votre trafic web normal. C’est comme si vos requêtes DNS se déguisaient en requêtes web ordinaires. Pour votre FAI ou n’importe quel observateur sur le réseau, impossible de distinguer quand vous naviguez et quand vous effectuez une résolution DNS – tout se fond dans le même trafic chiffré. C’est l’arme parfaite contre la censure et la surveillance.

Astuce : si vous hésitez entre DoH et [DoT (que je vous ai présenté dans un premier tuto)](https://dolys.fr/forums/topic/tuto-installer-pi-hole-et-unbound-avec-docker-gerer-les-publicites-et-les-requetes-dns-de-maniere-efficace/), voici un [petit switch pour passer de l’un à l’autre](https://dolys.fr/forums/topic/astuce-un-petit-script-pour-alterner-entre-vos-deux-stacks-dns/).

### Pourquoi cette méthode ?

– Pour vous protéger des DNS menteurs (coucou les FAI des pays européens…)  
– Pour chiffrer toutes vos requêtes DNS jusqu’à des résolveurs externes de confiance  
– Pour centraliser le filtrage DNS sur un poste de travail ou un réseau local  
– Et, bien sûr, pour gagner en souveraineté sur votre propre infrastructure numérique

### Prérequis

Avant de commencer, assurez-vous d’avoir :

– Un Linux à jour  
– Docker + Docker Compose installés  
– Un terminal et un accès sudo

### Installation de Docker (si pas déjà fait)

#### Pour les distributions basées sur Arch (EndeavourOS, Manjaro, Garuda, etc.)

```
# Installation de Docker et Docker Compose
sudo pacman -S docker docker-compose

# Démarrage et activation du service Docker
sudo systemctl enable --now docker

# Ajout de votre utilisateur au groupe docker (évite de devoir utiliser sudo)
sudo usermod -aG docker $USER
```

#### Pour les distributions basées sur Debian (Debian, Linux Mint, etc.)

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
sudo systemctl enable --now docker

# Ajout de votre utilisateur au groupe docker
sudo usermod -aG docker $USER
```

Après avoir ajouté votre utilisateur au groupe docker, déconnectez-vous et reconnectez-vous pour que les changements prennent effet.

### Mise en place de la stack DNS-DoH

Je vais travailler ici dans un répertoire dédié :

```
mkdir -p ~/docker/dns-doh
cd ~/docker/dns-doh
```

### 1. Télécharger les images Docker

Avant de couper l’ancien DNS, je récupère les images nécessaires :

```
docker pull klutchell/dnscrypt-proxy
docker pull pihole/pihole
```

### 2. docker-compose.yml

Je crée le fichier suivant :

```
nano docker-compose.yml
```

### Validation de la syntaxe YAML

Le fichier `docker-compose.yml` doit avoir une indentation correcte (2 espaces par niveau, pas de tabulations) pour éviter les erreurs avec Docker Compose. Si vous copiez le code depuis ce tutoriel, l’affichage du forum peut déformer l’indentation. Vérifiez la syntaxe avec [YAML Lint](http://www.yamllint.com/) en ligne ou installez `yamllint` localement.

Pour installer `yamllint` sur Debian/Ubuntu :

```
sudo apt update
sudo apt install yamllint
yamllint ~/docker/dns-doh/docker-compose.yml
```

Si des erreurs sont détectées, ajustez l’indentation dans un éditeur comme `nano` ou VS Code, puis revalidez. Consultez [la spécification YAML](https://yaml.org/spec/1.2/spec.html#id2760395) pour plus de détails sur l’indentation.

Voici le contenu du fichier :

```
services:
  dnscrypt-proxy:
    container_name: dnscrypt-proxy
    image: klutchell/dnscrypt-proxy
    restart: unless-stopped
    volumes:
      - ./dnscrypt-proxy:/config
    networks:
      dnsnet:
        ipv4_address: 172.22.0.2
    mem_limit: 512m
    cpus: '1.0'
    mem_reservation: 256m
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    restart: unless-stopped
    depends_on:
      - dnscrypt-proxy
    hostname: pihole
    env_file:
      - .env
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
    networks:
      dnsnet:
        ipv4_address: 172.22.0.3
    cap_add:
      - NET_ADMIN
    mem_limit: 1024m
    cpus: '2.0'
    mem_reservation: 512m
networks:
  dnsnet:
    ipam:
      config:
        - subnet: 172.22.0.0/24
```

### 3. .env

Je crée un fichier `.env` avec les variables nécessaires :

```
nano .env
```

Contenu (à adapter bien sûr) :

```
WEBPASSWORD='votre_mot_de_passe_super_sécurisé'
TZ=Europe/Paris
DNS1=172.22.0.2#53
DNS2=no
```

### 4. Configuration de dnscrypt-proxy

Je prépare le dossier de configuration :

```
mkdir -p dnscrypt-proxy
nano dnscrypt-proxy/dnscrypt-proxy.toml
```

Et je configure dnscrypt-proxy pour utiliser uniquement des serveurs DoH fiables, hors des 14-Eyes autant que possible, et sans conservation de logs :

```
# Liste des serveurs DNS sécurisés utilisés
server_names = ['ibksturm', 'pf-dnscrypt', 'uncensoreddns', 'altername', 'iij', 'libredns', 'dns0.eu-doh', 'digitale-gesellschaft-doh', 'blahdns-ch']

# Adresse d'écoute locale
listen_addresses = ['127.0.0.1:53']

# Types de serveurs à utiliser
ipv4_servers = true
ipv6_servers = false
dnscrypt_servers = true
doh_servers = true

# Politique de sécurité : pas de logs, pas de filtrage
require_nolog = true
require_nofilter = true

# Ignorer la configuration DNS système
ignore_system_dns = true

# Mise en cache DNS
cache = true
cache_size = 8192

# Délai de réponse maximum (en millisecondes)
timeout = 5000
```

Cette configuration utilise des serveurs prédéfinis plutôt que des définitions statiques, ce qui évite les erreurs de syntaxe avec les « stamps ». Les serveurs choisis sont réputés pour leur confidentialité et leur absence de censure.

### 5. Démarrage

Une fois tout en place, je lance la stack :

```
docker compose up -d
```

**Note pour distributions plus anciennes** : Sur les systèmes plus anciens, vous devrez peut-être utiliser la commande avec un tiret :

```
docker-compose up -d
```

### 6. Configuration des DNS du système

Maintenant, je dois configurer mon système pour utiliser Pi-hole comme résolveur DNS.

#### Pour les distributions utilisant systemd-resolved (la plupart des distributions modernes)

```
# Désactiver le service systemd-resolved
sudo systemctl disable --now systemd-resolved

# Configurer le fichier resolv.conf
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
```

#### Pour les distributions basées sur Arch utilisant NetworkManager

```
# Configurer NetworkManager pour ne pas modifier resolv.conf
sudo mkdir -p /etc/NetworkManager/conf.d/
echo -e "[main]\ndns=none" | sudo tee /etc/NetworkManager/conf.d/dns.conf

# Redémarrer NetworkManager
sudo systemctl restart NetworkManager

# Configurer resolv.conf
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf
```

#### Pour les distributions basées sur Debian utilisant resolvconf

```
# Installer resolvconf si ce n'est pas déjà fait
sudo apt install -y resolvconf

# Configurer les nameservers
echo "nameserver 127.0.0.1" | sudo tee /etc/resolvconf/resolv.conf.d/head

# Appliquer les changements
sudo resolvconf -u
```

### 7. Test

Je vérifie simplement avec :

```
dig debian.org @127.0.0.1
```

Je peux aussi faire un `docker logs dnscrypt-proxy` pour voir quels résolveurs ont été détectés et lesquels sont utilisés.

Pour accéder à l’interface web de Pi-hole, ouvrez un navigateur et allez à [http://localhost/admin](http://localhost/admin) ou [http://127.0.0.1/admin](http://127.0.0.1/admin).

### 8. Ajouter des listes de blocage recommandées

Pour améliorer le blocage des publicités et des traqueurs, vous pouvez ajouter des listes de blocage à Pi-hole. Comme les listes recommandées sont identiques pour DoH et DoT, consultez la [section des listes de blocage dans le tutoriel DoT](https://dolys.fr/forums/topic/tuto-installer-pi-hole-et-unbound-avec-docker-gerer-les-publicites-et-les-requetes-dns-de-maniere-efficace/#adlists) pour les instructions détaillées.

### 9. Installation des outils pour le DNS Switcher (optionnel)

Si vous souhaitez installer l’interface graphique pour le DNS Switcher mentionné au début, vous aurez besoin de yad :

#### Sur Arch et dérivés

```
sudo pacman -S yad
```

#### Sur Debian et dérivés

```
sudo apt install -y yad
```

Vous pourrez ensuite suivre le [tutoriel sur le DNS Switcher](https://dolys.fr/forums/topic/astuce-un-petit-script-pour-alterner-entre-vos-deux-stacks-dns/) pour basculer facilement entre vos configurations DoT et DoH.

### Conclusion

Cette solution me permet d’utiliser une résolution DNS chiffrée de bout en bout, sans aucun passage par les serveurs de mon FAI, tout en bénéficiant de la protection publicitaire de Pi-hole.

Elle complète la [stack DNS récursive avec Unbound DoT](https://dolys.fr/forums/topic/tuto-installer-pi-hole-et-unbound-avec-docker-gerer-les-publicites-et-les-requetes-dns-de-maniere-efficace/), selon le degré de contrôle que je veux exercer :

– Résolution récursive locale (DoT) : contrôle total et validation DNSSEC  
– DNS-over-HTTPS souverain : confidentialité maximale et contournement de la censure

Vous n’avez plus d’excuse pour laisser des FAI ou ennemis de la pensée espionner vos résolutions DNS, surtout dans le pays des Lumières 😉

---

### Documentation utile

– [https://dnscrypt.info/](https://dnscrypt.info/)  
– [https://docs.pi-hole.net/](https://docs.pi-hole.net/)  
– [https://github.com/DNSCrypt/dnscrypt-proxy/wiki/Configuration](https://github.com/DNSCrypt/dnscrypt-proxy/wiki/Configuration)

### Vérification et dépannage

Les configurations DNS-over-HTTPS peuvent parfois donner des résultats surprenants lors des tests en ligne. Pour bien comprendre comment vérifier votre installation et interpréter correctement les tests de fuite DNS, référez-vous à notre [guide dédié au dépannage et à la vérification DNS](https://dolys.fr/forums/topic/comprendre-les-tests-de-fuite-dns-et-depannage-dns-dot-et-dns-doh/).

Voilà ! 🙂

