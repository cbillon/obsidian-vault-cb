---
link: https://www.metal3d.org/blog/2025/livebox-limit%C3%A9e-aidons-la-avec-dnsmasq/
site: Metal3d
date: 2025-08-11T13:14
excerpt: |-
  Même la dernière Livebox ne fait pas le job coté DNS, alors on va la soulager de ce fardeau et
  laisser un bon vieux Raspberry Pi (ou n'importe quel machine sous Linux) faire le travail à sa place.
tags:
  - livebox
  - dns
slurped: 2026-03-28T10:15
title: Livebox limitée, Aidons-la avec Dnsmasq
---

![[Pasted image 20260328102004.png]]


Même la dernière Livebox ne fait pas le job coté DNS, alors on va la soulager de ce fardeau et laisser un bon vieux Raspberry Pi (ou n'importe quel machine sous Linux) faire le travail à sa place.

Depuis des années, les utilisateurs avancés[1](https://www.metal3d.org/blog/2025/livebox-limit%C3%A9e-aidons-la-avec-dnsmasq/#fn:1) râlent, mais Orange s’en soucie autant que de la durée de vie d’un poil pubien. Après tout, si ça ne concerne pas 90% des utilisateurs, pourquoi s’en soucier ? Pourtant, c’est un vrai casse-tête pour ceux qui veulent avancer dans le paramétrage de leur réseau local.

En gros, **impossible de manipuler les DNS des Livebox**. Pas de noms de machines supplémentaires, pas de sous-domaines, et encore moins de DNS autres que ceux d’Orange. Pas de filtrage, pas de sécurisation pour la famille. Heureusement, on peut contourner le problème en déléguant ce travail à une autre machine, comme un Raspberry Pi ou un vieux PC.

Dans cet article, on va utiliser **dnsmasq** sur une Fedora (mais n’importe quelle distribution fera l’affaire). Il sera notre serveur DNS et DHCP. Voici ce qu’on va faire :

- Configurer une machine (un Raspberry Pi) en IP statique
- En faire un serveur DNS et DHCP
- Couper le DHCP de la Livebox et ne plus utiliser son service DNS (elle ne servira plus que de routeur)

Trois étapes, mais il faudra un peu jongler pour ne pas se retrouver dans le noir.

> Pas de panique, suivez les explications et tout ira bien.

Un Raspberry Pi 3 ou 4 suffira amplement pour faire tourner dnsmasq.

Et oui, je connais Pi-Hole, une solution très intéressante, mais j’ai eu pas mal de soucis avec. Configurer DNSMasq demande un peu plus d’efforts, mais ça a marché sans problème. J’ai mis seulement 20 minutes à tout configurer sur mon Raspberry Pi, surtout parce que j’avais les traces de mes erreurs à chaque test.

> Faire les choses soi-même, c’est aussi apprendre, comprendre, et réaliser à quel point on peut aller plus loin.

## Prérequis

Ayez une machine dédiée, comme un Raspberry Pi ou un vieux PC, connectée en permanence au réseau, de préférence en Ethernet. Vous devez pouvoir vous y connecter à distance via SSH ou physiquement avec un clavier/écran.

Un éditeur de texte comme `vim`, `nano`, ou même `emacs` si vous êtes un peu fou, est nécessaire pour corriger les fichiers.

Mon setup : un Raspberry Pi 4 avec Fedora 41, branché sur une Livebox 6. Adaptez les instructions à votre environnement.

Assurez-vous d’avoir `nmcli` (client NetworkManager) sur votre machine dédiée. Sinon, adaptez la configuration de l’IP statique.

**Toutes les commandes doivent être exécutées avec les droits administrateur**, donc utilisez `sudo` ou connectez-vous en tant que `root`.

Prenez un café ou un remontant pour rester concentré.

## Avant de vous lancer

**Ne vous lancez pas sans savoir à quoi servent ces configurations.** Vous devez comprendre :

- ce qu’est un serveur DNS,
- et ce qu’est un serveur DHCP,
- et que vous allez perdre la connexion réseau sur certaines machines pendant la configuration.

**Premier point** : On va désactiver le DHCP de la Livebox. Pendant quelques minutes, votre réseau local va être perturbé. Votre machine dédiée (un Linux toujours allumé, un Raspberry Pi chez moi) prendra le relais.

**Donc, assurez-vous de pouvoir vous connecter à votre machine DHCP/DNS** soit avec un clavier et un écran, soit en ajustant le paramétrage réseau.

Pas de panique, je vais vous donner les clés pour éviter de rester dans le noir trop longtemps.

> Mais avant, quelques définitions…

### C’est quoi un DHCP ?

_Dynamic Host Configuration Protocol_ : Un service qui automatise la configuration de votre réseau local. Quand une machine rejoint le réseau, elle n’a pas d’adresse IP, pas de DNS, etc. Elle envoie un message en Broadcast pour obtenir ces informations. Votre Livebox (ou Freebox) fait office de serveur DHCP… pour le moment.

![DHCP](https://www.metal3d.org/blog/2025/livebox-limit%C3%A9e-aidons-la-avec-dnsmasq/dhcp-simple.png)

### C’est quoi un serveur DNS ?

_Domain Name Server_ : Un serveur qui traduit les noms de domaine en adresses IP. Votre Livebox, comme beaucoup de routeurs, fait office de serveur DNS. Elle résout les noms de domaine internet et les noms de machines de votre réseau.

![Principe du DNS](https://www.metal3d.org/blog/2025/livebox-limit%C3%A9e-aidons-la-avec-dnsmasq/dns.png)

### Les deux ensemble, c’est mieux ?

Le DNS et le DHCP peuvent être séparés, mais les avoir ensemble est très pratique. Dnsmasq, par exemple, enregistre le nom de la machine dans le DNS quand il lui attribue une IP. Actuellement, c’est votre box internet qui gère les deux.

## Configuration réseau de la machine dédiée

Votre machine qui va servir à faire office de DNS et de DHCP doit être configurée pour avoir une adresse IP statique et ne plus utiliser `systemd-resolved`.

### Première chose, passer la machine dédiée en IP statique

Pourquoi une IP statique ? Parce que cette machine deviendra le DNS du réseau local. Si son adresse change, vous risquez une désynchronisation des adresses IP sur les machines du réseau. On fige donc l’IP.

Je pars du principe que vous avez connecté cette machine dédiée en Ethernet directement sur la Livebox. C’est plus stable, plus rapide et ça évite bien des problèmes.

C’est une étape délicate, car vous allez perdre la connexion à cette machine lorsque vous changerez son IP.

On va utiliser l’adresse IP 192.168.1.250. Normalement, la Livebox n’assigne pas d’adresse IP au-delà de 192.168.1.150. Vérifiez quand même que cette adresse n’est pas déjà utilisée par une autre machine. Pour le moment, on garde le DNS de la Livebox.

La manipulation est simple, mais si vous faites cela sur un shell SSH, vous allez perdre la connexion à votre Raspberry Pi, ce qui est normal. Il faudra se reconnecter avec la nouvelle adresse IP.

On garde la Livebox comme serveur DNS et le domaine “home”. Pour le moment, on continue de l’utiliser.

On va utiliser `nmcli` (client NetworkManager). Si votre machine est configurée différemment, utilisez l’outil adéquat.

Pour trouver le nom de la connexion réseau, utilisez :

```
nmcli connection --show
NAME                UUID                                  TYPE      DEVICE
Wired connection 1  9b63ca3f-bbf2-3567-8c24-39cc956e92fe  ethernet  end0
lo                  f5674142-e3f6-468d-b8a7-b89bb8e609c0  loopback  lo
podman1             f27ed4fc-043e-4671-aab8-f3f1b294342d  bridge    podman1
Livebox-0730        440b56e8-7407-4ff1-84b3-d78d76722594  wifi      --

```

Le nom “`Wired connection 1`” est le nom de la connexion réseau à changer chez moi, elle peut être différente chez vous. Gardez aussi en tête le nom de l’interface utilisée, chez moi est `end0`, ça peut être un truc différent chez vous.

> Vous remarquez que j’ai Podman sur cette machine, ça induit de faire attention quand on va paramétrer le serveur DNS de Dnsmasq, car Podman utilise aussi un DNS interne. On va le voir plus tard.

```
## nom de la connexion réseau
CONNECTION="Wired connection 1"
## votre nouvelle IP pour le raspberry pi
NEW_IP=192.168.1.250
## on va garder le même domaine que la Livebox
DOMAIN=home

## on met la connexion en statique
nmcli connection modify "$CONNECTION" \
  ipv4.addresses $NEW_IP/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns  192.168.1.1 \
  ipv4.dns-search $DOMAIN \
  ipv4.method manual
systemctl restart NetworkManager
```

Donc voilà, on a mis une nouvelle adresse IP, une passerelle (la Livebox), un DNS (la Livebox) et un domaine (“home”). On a bien spécifié que la méthode est “manuelle”, donc que l’on ne veut pas une IP assignée par le DHCP de la Livebox.

Là, vous allez certainement perdre la connexion SSH. Donc, reconnectez-vous sur 192.168.1.250 et vérifiez que tout va bien. Faites un `ping` sur “google.com”, une machine du réseau, etc…

### Suppression de systemd-resolved

Vous devriez avoir retrouvé la connexion SSH sur votre machine dédiée (le Raspberry Pi chez moi). On continue la configuration.

On va supprimer `systemd-resolved`, c’est un outil sympa sur les postes clients, mais sur un serveur avec réseau statique, qui plus est un serveur DNS, c’est une plaie. Il va nous faire des misères de conflits de port, ça va nous énerver plus que nous rendre service. Allez ça dégage !

Supprimer `systemd-resolved` ne suffit pas, il faut forcer le système à régénérer un fichier `/etc/resolv.conf` qui ne soit pas un lien symbolique vers `/run/systemd/resolve/stub-resolv.conf`.

```
sudo systemctl stop systemd-resolved
systemctl disable --force systemd-resolved

## et on supprime /etc/resolv.conf qui est un lien symbolique
rm -f /etc/resolv.conf
```

**Attention** à partir de maintenant, vous n’avez plus de serveur DNS**, relancez la connexion et vérifiez que le fichier `/etc/resolv.conf` est bien créé.

```
systemctl restart NetworkManager
cat /etc/resolv.conf
```

S’il est vide ou manquant, vérifiez que le service `systemd-resolved` est bien arrêté et désactivé. Regardez les erreurs avec `journalctl -e -u NetworkManager`. Bref, trouvez pourquoi…

### Checkpoint de votre config réseau

On résume, votre petit serveur qui va faire DNS / DHCP, est à présent maitre de sa configuration réseau.

À ce stade, son adresse IP n’est plus assignée par la Livebox. La Livebox sert encore de routeur et de serveur DNS, mais ça ne va pas durer.

## Configurer DNSMasq pour faire le job

On va enfin installer l’outil magique qui va remplacer ce que la Livebox fait très mal : **dnsmasq**.

### DNSMasq, partie DNS

Pour le moment, c’est votre Livebox qui fait office de serveur DNS.

On va maintenant faire en sorte que cette machine dédiée devienne un serveur DNS.

Vous commencez par installer `dnsmasq` et deux ou trois outils pour tester notre configuration (`dig`, `ping`…).

```
dnf install -y dnsmasq bind-utils iputils
## utilisateurs de Debian/Ubuntu, débrouillez vous avec apt...
```

**Très important** : il faut accepter d’être contacté pour les requêtes DNS et DHCP, donc ouvrir le firewall sur les bons ports ; on peut donner les noms de services, plus clair :

```
## je vous rappelle que je suis sur Fedora, donc firewall-cmd
## est l'outil de gestion du firewall
firewall-cmd --add-service=dns --permanent
firewall-cmd --add-service=dhcp --permanent
firewall-cmd --reload

## si vous utilisez une Debian/like, vous pouvez utiliser ufw
ufw allow dns
ufw allow dhcp
ufw reload
```

La chose bien foutue sur Fedora (je ne sais pas pour les autres distributions) est que `dnsmasq` propose un fichier de configuration _de base_ complètement commenté dans `/etc/dnsmasq.conf`. Et je vous conseille de ne pas le modifier. Il doit servir de référence. La configuration perso, on la place dans des fichiers à l’intérieur du répertoire `/etc/dnsmasq.d/`. C’est une bonne pratique, car ça permet de séparer les configurations.

> En tout cas, chez Ubuntu, [leur doc](https://doc.ubuntu-fr.org/configuration_serveur_dns_dhcp), c’est la fête de la saucisse, édition à la bourrin dans le fichier de base ! Ils me tuent…

On va créer un fichier de configuration dans `/etc/dnsmasq.d/main.conf` pour les trucs de base :

```
# interface réseau à écouter (end0, eth0 ...)
INTERFACE=$(ip r | awk '/default/{print $5; exit}')

cat 1>/etc/dnsmasq.d/main.conf << EOF
## port à écouter
port=53
## quelles interfaces écouter
bind-interfaces
interface=$INTERFACE

## Ajoutez des DNS externes pour le web
## on va prendre les DNS de Cloudflare et de Quad9
server=1.1.1.1
server=9.9.9.9
EOF
```

> Juste un détail, mais tout à l’heure, je vous ai dit que j’ai Podman, donc je suis obligé de spécifier l’interface `end0` (détectée avec ma commande “`ip r...`”). Sinon, Dnsmasq voudrait écouter sur toutes les interfaces réseau, y-compris “`podman1`”. Et comme un serveur DNS est déjà présent ici, DNSMasq plantera.

Vous pouvez lancer `dnsmaq` :

```
sudo systemctl enable --now dnsmasq
```

**Si vous avez un message d’erreur, allez voir `journalctl -e -u dnsmasq` pour comprendre ce qui ne va pas.** Ne continuez pas tant que dnsmasq ne fonctionne pas !

Si tout va bien, alors on va changer le DNS utilisé sur cette machine pour qu’elle utilise `dnsmasq` comme serveur DNS.

```
## nom de la connexion réseau
CONNECTION="Wired connection 1"
NEW_IP=192.168.1.250
## on change le DNS de la connexion réseau
nmcli connection modify "$CONNECTION" \
  ipv4.dns  $NEW_IP
nmcli device reapply "$CONNECTION"

# je ne sais plus si c'est utile :
systemctl restart NetworkManager
```

Vous remarquez que je mets l’adresse de DNS sur le Raspberry pi sur “lui-même”. C’est normal ! parce que le Raspberry Pi **est le serveur DNS**.

**Testons !** C’est important, il faut absolument que `dnsmasq` fonctionne proprement en tant que serveur DNS.

```
## On regarde si NetworkManager a bien pris en compte
## la nouvelle configuration
cat /etc/resolv.conf
## doit ressembler à ça:
## Generated by NetworkManager
## search home
## nameserver 192.168.1.250

## on teste une résolution avec dnsmasq...
dig @127.0.0.1 +short google.com

## et faites le sans dire quel DNS utiliser
dig google.com | grep SERVER:
## répondra quelque chose comme:
##;; SERVER: 192.168.1.250#53(192.168.1.250) (UDP)
```

**Si aucune adresse IP n’est retournée ou que ce n’est pas l’adresse de votre machine qui sert de DNS, c’est que dnsmasq ne fonctionne pas. Vous devez corriger ça !**

Si tout se passe bien, d’abord “bravo à vous”, et on passe à la suite !

### DNSMasq, partie DHCP

> C’est le moment de dire adieu au DHCP de la Livebox.

Maintenant, on va dire à notre DNSMasq qu’il va faire office de DHCP. C’est donc lui qui répondra à toutes les machines qui vont arriver sur le réseau.

Il faut qu’il leur fournisse une adresse IP, un DNS, une passerelle.

- l’adresse IP, on va lui dire qu’on ne dépassera pas 192.168.1.249, on va même dire qu’on veut des adresses entre 192.168.1.10 à 192.168.1.249. Ça nous permet de se réserver les adresses IP que personne ne peut avoir à moins de le faire à la main,
- le nouveau DNS, donc l’IP de la machine dédiée,
- la passerelle, c’est l’adresse IP de la Livebox (192.168.1.1 par défaut)

Pour garder la même logique que ce que faisait la Livebox, on va utiliser un suffixe DNS en “.home”. On l’a spécifié tout à l’heure pour configurer notre connexion statique.

Donc, configurons :

```
DOMAIN=home
IP_ADDR=$(hostname -I | awk '{print $1}')

cat 1> /etc/dnsmasq.d/dhcp.conf <<EOF
log-dhcp
## rang d'ip qu'on peut donner aux machines, on
## se réserve les IPs de 0 à 10 et au delà de 249
dhcp-range=192.168.1.10,192.168.1.249,12h
dhcp-option=option:netmask,255.255.255.0
dhcp-option=option:router,192.168.1.1
dhcp-option=option:dns-server,$IP_ADDR
dhcp-option=option:domain-name,$DOMAIN
dhcp-option=option:domain-search,$DOMAIN
enable-ra

## et on est dans un domaine
domain=$DOMAIN
local=/$DOMAIN/

# et on notre DHCP c'est un peu comme Sauron
dhcp-authoritative
EOF
```

En gros (et en simplifiant) :

- On assigne un intervalle d’adresses IP de 192.168.1.10 à 192.168.1.249, nos machines auront une adresse IP pendant 12 heures (12h). Si elles ne sont plus là pendant ce laps de temps, alors elles perdent la réservation. Une nouvelle IP pourra leur être assignée. Augmentez la durée si vous voulez, mais 12 heures est une valeur assez commune.
- Quand on configure une machine, on lui assigne un masque de sous-réseau, le DNS (notre raspberry Pi) et la passerelle (la Livebox)
- `domain=home` : on dit à dnsmasq que le domaine de notre réseau est “home”
- `local=/home/` : on dit à dnsmasq de considérer que les machines du réseau sont dans le domaine “home” (donc il retrouvera les noms de machines avec un suffixe .home)
- `dhcp-authoritative` : on dit à dnsmasq qu’il est le seul serveur DHCP du réseau, et qu’il n’y a pas de conflit possible avec la Livebox.
- `enable-ra` : on active les annonces de routeur (Router Advertisements) pour que les machines puissent découvrir la passerelle et le DNS (optionnel, mais recommandé)

**Pourquoi mettre un domaine ?** Pour deux raisons…

- Parce que c’est plus propre, qu’on va éviter des conflits avec les noms de domaines internet si une machine fait n’importe quoi,
- et puis `systemd-resolved` va nous râper les raisins à ne pas accepter de faire des résolutions de noms sans domaine. **En conséquence, on est obligé de lui en donner un.**

> Notez que c’est la raison qui a fait que j’ai rendu mon routeur TP-Link qui n’assigne aucun suffixe DNS…

Si vous voulez forcer une IP pour une machine en particulier, faitez un fichier de configuration qui s’appelle par exemple `/etc/dnsmasq.d/reserved.conf` :

```
## dhcp-host=<MAC>,<IP> ou encore...
## dhcp-host=<MAC>,<IP>,<NOM>,<TEMPS>
## par exemple, pour mon poste fixe:
dhcp-host=a6:db:13:b0:df:45,192.168.1.98
```

Et enfin, on configure le fichier `/etc/dnsmasq.d/home.conf` pour les options de base du DNS:

```
cat 1> /etc/dnsmasq.d/home.conf <<EOF
no-resolv
no-negcache
no-hosts
domain-needed
bogus-priv

## mon Raspberry Pi sert aussi de NAS
address=/nas/192.168.1.250
address=/nas.home/192.168.1.250
EOF
```

Houlla… Alors, je vous explique ce que ça veut dire:

- no-resolv: on ne va pas utiliser le fichier `/etc/resolv.conf` pour les requêtes DNS
- no-negcache: on ne va pas mettre en cache les réponses négatives (celles qui disent que le nom n’existe pas)
- no-hosts: on ne va pas utiliser le fichier `/etc/hosts` pour les résolutions de noms
- domain-needed: on ne va pas répondre aux requêtes DNS sans domaine (c’est important)
- bogus-priv: si aucune adresse IP n’est trouvée, on ne va pas répondre aux requêtes pour les adresses privées (192.168.x.x, 10.x.x.x, etc.)

Pourquoi j’ajoute `address=/nas/...` et `address=/nas.home/...` ?

Mon Raspberry pi qui sert de DHCP/DNS est configuré en IP statique. Il ne s’enregistre pas tout seul dans le DNS.

**Donc, je dois le déclarer manuellement.** Parce que le DHCP ne s’est pas occupé de lui-même.

Évidemment que vous pouvez utiiliser un autre nom. Mon Raspberry Pi est aussi un NAS, je l’appelle comme ça. Vous pouvez l’appeler “rpi”, “domain” ou “jean-michel”, c’est vous qui voyez.

**Maintenant, on passe aux choses sérieuses !**

On redémarre le service `dnsmasq` :

```
systemctl restart dnsmasq
```

Il ne faut **aucune erreur** !

**À partir de maintenant, la machine dédiée est le serveur DNS et le serveur DHCP de votre réseau local.**

> C’est le moment fatidique, on coupe le DHCP de la Livebox.

![Coupure le DHCP de la Livebox](https://www.metal3d.org/blog/2025/livebox-limit%C3%A9e-aidons-la-avec-dnsmasq/stop-dhcp.png)

Attention, toutes les machines vont avoir un souci pendant quelques secondes, voir plusieurs minutes. Leur configuration réseau ne sera pas à jour, donc il faut relancer la phase de négociation DHCP. En gros, couper le réseau des machines et le redémarrer.

> Pour le moment, votre réseau est aussi bordélique qu’une chambre d’ado élevé avec la méthode Montessori.

C’est la phase “critique”, tant que vos machines ne se reconnectent pas, elles n’ont pas reçu les informations du DHCP.

## On teste sur une machine du réseau

Sur mon laptop (ouais, mon “ordinateur portable”), j’ai coupé le wifi et je l’ai relancé. Puis je vérifie :

```
ip r get 1                                                         metal3d@patrice-laptop
## m'affiche
## 1.0.0.0 via 192.168.1.1 dev wlo1 src 192.168.1.98 uid 1000
```

J’ai bien une IP et la passerelle par défaut, ouf !

Et sinon, coté DNS:

```
resolvectl status
## affiche:
##...
Link 3 (wlo1)
    Current Scopes: DNS LLMNR/IPv4 LLMNR/IPv6
         Protocols: +DefaultRoute LLMNR=resolve -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 192.168.1.250
       DNS Servers: 192.168.1.250
        DNS Domain: home
     Default Route: yes
```

Tout va bien ! Le DHCP m’a assigné le DNS qu’on a configuré, et le domaine “home”.

**Vous pouvez être fiers de vous, tout est OK !**

## Quelques vérifications supplémentaires

Sur le raspberry Pi ou la machine dédié qui fait DHCP/DNS, vous pouvez vérifier que les machines sont bien enregistrées :

```
 cat /var/lib/dnsmasq/dnsmasq.leases
1754853901 20:4e:f6:30:2b:2e 192.168.1.213 Roomba-DB09C7BAA7C54FC881662B9B88592317 *
1754855117 d8:6c:63:5f:34:e8 192.168.1.94 Chromecast *
1754853936 6a:07:a8:80:6e:19 192.168.1.90 patrice-desktop 01:6a:07:a8:80:6e:19
1754853936 50:14:79:1b:dc:7a 192.168.1.62 iRobot-85DB52B2707343789E585364D0C28099 *
1754867719 a6:db:13:b0:df:45 192.168.1.98 patrice-laptop 01:a6:db:13:b0:df:45
1754854162 66:e0:43:eb:2b:e9 192.168.1.26 Z-Flip6-de-Patrice 01:66:e0:43:eb:2b:e9
 #... etc
```

On peut aussi vérifier que le DNS fonctionne, en prenant un nom de machine :

```
ping -c 2 patrice-desktop
PING patrice-desktop.home (192.168.1.90) 56(84) bytes of data.
64 bytes from patrice-desktop.home (192.168.1.90): icmp_seq=1 ttl=64 time=3.31 ms
64 bytes from patrice-desktop.home (192.168.1.90): icmp_seq=2 ttl=64 time=3.15 ms
```

On voit par ailleurs que le nom de domaine “.home” est bien utilisé et que ça répond.

> Et maintenant, on peut aller beaucoup plus loin !

## Ce que Livebox ne permettait pas, on peut le faire

D’abord, on utilise désormais les DNS qu’on veut. On a mis 1.1.1.1 et 9.9.9.9 mais vous pouvez mettre ceux de Orange, de Google, de votre FAI…

Et dorénavant, vous pouvez aussi ajouter des noms de machines, des alias, des domaines, etc.

Reprenons l’exemple initial, j’ai une machine “patrice-desktop” qui est une machine sur laquelle j’ai installé pas mal de trucs. Je veux pouvoir y accéder avec plusieurs noms.

Un truc intéressant avec un _vrai_ serveur DNS, c’est qu’au lieu de dire “alors ce nom correspond à telle adresse IP”, on peut lui dire “ce nom, il est un alias de tel nom”. Je vulgarise un peu hein… C’est ce qu’on appelle un `CNAME` (Canonical Name).

Donc, on va créer un fichier de configuration pour des alias, par exemple `/etc/dnsmasq.d/alias.conf` :

```
cname=ai.home,patrice-desktop.home
```

Relancez le service `dnsmasq` :

```
systemctl restart dnsmasq
```

Et sur un poste…

```
dig ai.home
##...
;; ANSWER SECTION:
ai.home.                0       IN      CNAME   patrice-desktop.home.
patrice-desktop.home.   0       IN      A       192.168.1.90
```

Comprenez bien, je peux désormais accéder à mon serveur avec le nom `ai.home`.

J’ai ensuite configuré ma machine pour que `ai.home` pointe sur le conteneur [OpenWebUI](https://github.com/open-webui/open-webui) pour utiliser des modèles locaux d’IA. J’ai aussi ajouté `ollama.home` qui pointe sur le même serveur, mais qui est un alias pour aller sur mon conteneur Ollama.

Et vous pouvez faire autant de bordel que vous voulez.

Tiens, un bonus : [https://github.com/KnightmareVIIVIIXC/AIO-Firebog-Blocklists](https://github.com/KnightmareVIIVIIXC/AIO-Firebog-Blocklists) contient des fichiers à placer dans votre `/etc/dnsmasq.d/` pour bloquer des domaines, des publicités, etc.

J’ai plutôt utilisé [la liste simple de OISD](https://oisd.nl/setup/dnsmasq):

```
curl https://small.oisd.nl/dnsmasq2 > /etc/dnsmasq.d/oisd-blocklist.conf
systemctl restart dnsmasq
```

Mais vous pouvez tout à fait en faire autant avec n’importe quel domaine :

```
## par exemple dans /etc/dnsmasq.d/blacklist.conf
##ipv4 et ipv6
local=/foo.com/
```

Pourquoi “local” ? En fait, vous dites à DNSMasq de tenter de résoudre le nom de domaine localement. Et comme aucune entrée locale n’existe, il ne trouvera pas le nom de domaine. Donc, il ne répondra pas. Ça rend donc la résolution de ce domaine impossible. C’est un moyen hyper simple de bloquer un domaine.

Résultat, par exemple sur le site Futura Sciences, réputé blindé de pubs…

![Futura Sciences sans pub](https://www.metal3d.org/blog/2025/livebox-limit%C3%A9e-aidons-la-avec-dnsmasq/blocked.png)

Bref, gérer son DNS c’est cool.

## Alors, heureux ?

La plupart des box internet du marché, pour les particuliers, ne sont pas à la hauteur en termes de gestion réseau. Ce sont de très bons routeurs internet pour la majorité, mais elles limitent vraiment les utilisateurs avancés.

Configurer son propre DHCP/DNS peut faire peur. Ça demande un peu de patience, de temps et le courage d’affronter la crainte de casser le réseau. Mais tout est rapidement réversible.

DNSMasq ne m’a jamais déçu, je m’en suis servi à mainte reprise en local (pour réduire les latences de résolution de nom ou avec Docker/Podman), je l’avais réglé pour supprimer les publicités sur une de mes machines, et je l’avais même intégré dans un cluster pour corriger un gros souci de résolution interne.

Il est léger, et donc tourne sans accroc sur un tout petit Raspberry Pi. Il est simple à configurer (c’est très lisible).

Là, je l’utilise pour mon réseau local depuis des semaines. Aucun souci.

Donc oui, moi, je suis heureux d’avoir la maitrise de mon DNS, de pouvoir faire du cache, de mieux contrôler mon domaine et de pouvoir faire des alias, couper les pubs, etc.