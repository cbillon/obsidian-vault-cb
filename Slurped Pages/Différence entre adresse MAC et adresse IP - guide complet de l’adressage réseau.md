---
link: https://networkpulse.fr/difference-entre-adresse-mac-et-adresse-ip/
byline: Guillaume REYNAUD
site: Network Pulse
date: 2026-01-04T18:07
excerpt: "Adresses MAC et IP : deux identifiants réseau essentiels, complémentaires, avec des rôles distincts pour connecter et identifier les appareils."
slurped: 2026-01-21T17:42
title: "Différence entre adresse MAC et adresse IP : guide complet de l’adressage réseau"
tags:
  - network
---

Dans le monde des réseaux informatiques, deux types d'adresses sont essentiels pour permettre la communication entre les appareils : l'adresse MAC et l'adresse IP.

Bien que ces deux identifiants puissent sembler similaires au premier abord, ils jouent des rôles distincts et complémentaires dans l'architecture réseau. Ce tutoriel vous aidera à comprendre leurs différences fondamentales et leur utilisation pratique.

## **L'adresse MAC : L'identité physique de votre appareil**

### Définition et caractéristiques

L'adresse MAC (Media Access Control) est un identifiant unique attribué à chaque carte réseau (NIC - Network Interface Card) par son fabricant. Pensez-y comme l'équivalent du numéro de châssis d'une voiture : chaque appareil possède son propre identifiant gravé dans le matériel.

Cette adresse se compose de _6 paires de chiffres hexadécimaux_ (donc 6 octets ou 48 bits au total), séparés par des deux-points ou des tirets, comme par exemple : 6C:83:75:B8:22:1A.

Mais cette structure n'est pas aléatoire : elle obéit à une organisation précise qui révèle des informations importantes sur l'appareil.

### Exemple d'une adresse MAC : Comprendre sa structure

![Exemple d'une adresse MAC : Comprendre sa structure](https://networkpulse.fr/content/images/2025/12/adresse-mac-adresse-ip-2.webp)

Une adresse MAC de 6 octets est divisée en deux parties distinctes de 3 octets chacune :

Les 3 premiers octets - OUI (Organizationally Unique Identifier) : Cette première moitié identifie le fabricant de la carte réseau. Par exemple, dans l'adresse 6C:83:75:B8:22:1A, la séquence 6C:83:75 est attribuée par l'IEEE (Institute of Electrical and Electronics Engineers) à un fabricant spécifique. Chaque constructeur de cartes réseau (Intel, Realtek, Broadcom, etc.) possède un ou plusieurs identifiants OUI qui lui sont propres. C'est grâce à cette partie que vous pouvez déterminer qui a fabriqué votre carte réseau.

Les 3 derniers octets - NIC (Network Interface Controller Specific) : La seconde moitié, dans notre exemple B8:22:1A, est attribuée par le fabricant lui-même pour identifier de manière unique chaque interface réseau qu'il produit. Cette portion garantit qu'aucune carte réseau du même fabricant ne possède le même identifiant complet.

### Les bits spéciaux du premier octet

Le premier octet de l'adresse MAC contient deux bits particulièrement importants qui définissent le type d'adresse :

**Le bit U/L (Universal/Local)** : C'est le 7ème bit (en comptant de gauche à droite) du premier octet.

- Si ce bit vaut 0, l'adresse est globalement unique (attribuée par l'IEEE)
- Si ce bit vaut 1, l'adresse est localement administrée (modifiée manuellement)

**Le bit I/G (Individual/Group)** : C'est le 8ème bit (le dernier) du premier octet.

- Si ce bit vaut 0, l'adresse est de type unicast (destinée à un seul appareil)
- Si ce bit vaut 1, l'adresse est de type multicast (destinée à un groupe d'appareils)

Par exemple, si le premier octet est 6C en hexadécimal, cela correspond à 01101100 en binaire. Le 7ème bit est 0 (adresse universelle) et le 8ème bit est 0 (unicast).

### L'adresse de broadcast MAC

![L'adresse de broadcast MAC](https://networkpulse.fr/content/images/2025/12/adresse-mac-adresse-ip-3.webp)

Une adresse MAC spéciale mérite une attention particulière : FF:FF:FF:FF:FF:FF.

Il s'agit de l'adresse de broadcast MAC, utilisée pour envoyer un message à tous les appareils d'un réseau local simultanément. C'est notamment cette adresse qui est utilisée avec le protocole ARP (Address Resolution Protocol) pour découvrir l'adresse MAC correspondant à une adresse IP.

**Exercice pratique** : Pour afficher votre adresse MAC sous Windows, ouvrez l'invite de commandes et tapez _ipconfig /all_.

Sous Linux ou macOS, utilisez la commande _ifconfig ou ip link show_. Vous verrez l'adresse physique de chaque interface réseau de votre machine.

Essayez ensuite de rechercher en ligne l'OUI de votre carte pour identifier son fabricant. Vous pouvez consulter ce site : [https://macaddresslookup.io/fr](https://macaddresslookup.io/fr), pou identifier le constructeur.

### Rôle et fonctionnement

L'adresse MAC opère au niveau de la couche liaison de données (Layer 2) du modèle OSI. Son rôle principal est d'identifier un appareil au sein d'un réseau local (LAN).

Lorsque vous envoyez des données sur votre réseau domestique ou d'entreprise, c'est l'adresse MAC qui permet aux switches et aux routeurs de savoir exactement à quel appareil physique transmettre l'information.

Une caractéristique importante : l'adresse MAC est permanente et gravée directement dans le matériel.

Bien qu'il soit techniquement possible de la modifier par logiciel (spoofing MAC), ce qui changerait notamment le bit U/L à 1, elle reste fondamentalement liée à l'équipement physique.

### Applications pratiques

Dans un contexte professionnel, les adresses MAC sont utilisées pour le filtrage d'accès réseau.

Les administrateurs peuvent configurer des points d'accès Wi-Fi pour n'autoriser que certaines adresses MAC, créant ainsi une liste blanche d'appareils autorisés. Cette méthode, bien que contournable via le MAC spoofing, constitue une première couche de sécurité.

Les adresses MAC sont également utilisées dans les protocoles de découverte réseau.

Par exemple, le protocole DHCP utilise l'adresse MAC du client pour lui attribuer une adresse IP spécifique, permettant ainsi de réserver des adresses IP fixes à certains équipements sur le réseau.

## **L'adresse IP : Votre localisation sur le réseau**

### Définition et structure

L'adresse IP (Internet Protocol) fonctionne comme une adresse postale pour votre appareil sur un réseau. Contrairement à l'adresse MAC, elle est logique et configurable.

Il existe deux versions principales : IPv4, qui utilise un format de 32 bits (par exemple, 192.168.1.1), et IPv6, qui utilise 128 bits pour répondre à l'épuisement des adresses IPv4.

![L'adresse IP : Votre localisation sur le réseau](https://networkpulse.fr/content/images/2025/12/adresse-mac-adresse-ip-4.webp)

### Anatomie d'une adresse IPv4

Une adresse IPv4 est composée de 32 bits, généralement représentée sous forme décimale pointée en quatre octets. Prenons l'exemple de l'adresse 192.168.1.10 :

**Structure en octets** : Chaque nombre séparé par un point représente un octet (8 bits), pouvant prendre une valeur entre 0 et 255.

- Premier octet : 192
- Deuxième octet : 168
- Troisième octet : 1
- Quatrième octet : 10

En binaire, cette adresse s'écrit : 11000000.10101000.00000001.00001010

**Les classes d'adresses** : Historiquement, les adresses IPv4 étaient divisées en classes (A, B, C, D, E) selon le premier octet. Bien que ce système soit aujourd'hui remplacé par le CIDR (Classless Inter-Domain Routing), il reste important de le connaître :

- Classe A (1-126) : pour les très grands réseaux
- Classe B (128-191) : pour les réseaux de taille moyenne
- Classe C (192-223) : pour les petits réseaux
- Classe D (224-239) : pour le multicast
- Classe E (240-255) : réservée à des fins expérimentales

### Le masque de sous-réseau : Diviser pour mieux régner

Le masque de sous-réseau (subnet mask) est un élément crucial qui accompagne toujours une adresse IP.

Il détermine quelle portion de l'adresse IP identifie le réseau et quelle portion identifie l'hôte (l'appareil) sur ce réseau.

**Fonctionnement du masque** : Un masque de sous-réseau est également composé de 32 bits. Il utilise une série de 1 consécutifs suivis de 0 consécutifs.

Par exemple, le masque 255.255.255.0 s'écrit en binaire : 11111111.11111111.11111111.00000000

Les bits à 1 indiquent la partie réseau, les bits à 0 indiquent la partie hôte.

**Exemple pratique** : Prenons l'adresse 192.168.1.10 avec le masque 255.255.255.0 :

- Adresse IP : 11000000.10101000.00000001.00001010
- Adresse IP : 11000000.10101000.00000001.00001010
- Masque : 11111111.11111111.11111111.00000000
- Partie réseau : 192.168.1 (les 24 premiers bits)
- Partie hôte : 10 (les 8 derniers bits)

Cela signifie que tous les appareils dont l'adresse commence par 192.168.1 (de 192.168.1.0 à 192.168.1.255) appartiennent au même sous-réseau et peuvent communiquer directement sans passer par un routeur.

**Notation CIDR** : Le masque peut également être exprimé en notation CIDR (Classless Inter-Domain Routing). Au lieu d'écrire 255.255.255.0, on écrit /24, indiquant que les 24 premiers bits sont réservés au réseau. Ainsi, 192.168.1.10/24 est équivalent à 192.168.1.10 avec le masque 255.255.255.0.

**Adresses spéciales dans un sous-réseau** :

- L'adresse réseau : tous les bits hôtes à 0 (ex : 192.168.1.0)
- L'adresse de broadcast : tous les bits hôtes à 1 (ex : 192.168.1.255)
- Les adresses utilisables : toutes les autres (ex : 192.168.1.1 à 192.168.1.254)

Dans notre exemple avec le masque /24, nous disposons donc de 254 adresses utilisables pour les hôtes (256 moins l'adresse réseau et l'adresse de broadcast).

**Calcul pratique** : Pour déterminer si deux appareils sont sur le même réseau, on effectue une opération ET logique (AND) entre l'adresse IP et le masque. Si le résultat est identique pour les deux appareils, ils sont sur le même réseau.

Supposons que vous ayez deux appareils avec les adresses IP suivantes et le même masque de sous-réseau :

- Appareil A : 192.168.10.25
- Appareil B : 192.168.10.78
- Masque de sous-réseau : 255.255.255.0 (/24)

**Étape 1 : Convertir les adresses IP et le masque en binaire**

- 192.168.10.25 → 11000000.10101000.00001010.00011001
- 192.168.10.78 → 11000000.10101000.00001010.01001110
- Masque 255.255.255.0 → 11111111.11111111.11111111.00000000

**Étape 2 : Effectuer l’opération AND entre chaque adresse IP et le masque**

- Appareil A : 11000000.10101000.00001010.00011001 AND 11111111.11111111.11111111.00000000 = 11000000.10101000.00001010.00000000 → 192.168.10.0
- Appareil B : 11000000.10101000.00001010.01001110 AND 11111111.11111111.11111111.00000000 = 11000000.10101000.00001010.00000000 → 192.168.10.0

**Étape 3 : Comparer les résultats**

Les résultats de l’opération AND sont identiques (**192.168.10.0**).

  
✅ _Conclusion :_ les deux appareils sont **sur le même réseau** et peuvent communiquer directement sans passer par un routeur.

**Variante : Appareil C sur un autre réseau**

- Appareil C : 192.168.11.15 Opération AND : 11000000.10101000.00001011.00001111 AND 11111111.11111111.11111111.00000000 = 11000000.10101000.00001011.00000000 → 192.168.11.0

Résultat différent → ❌ Appareil C **n’est pas sur le même réseau** que A et B.

### Attribution et flexibilité

Les adresses IP sont attribuées par un administrateur réseau, un fournisseur d'accès Internet (ISP), ou automatiquement par un serveur DHCP (Dynamic Host Configuration Protocol).

Cette flexibilité permet aux appareils de changer d'adresse IP selon leur emplacement et leur configuration réseau.

On distingue deux types d'adresses IP : les adresses statiques, qui restent constantes, et les adresses dynamiques, qui peuvent changer à chaque connexion au réseau.

Dans un environnement domestique, votre routeur attribue généralement des adresses dynamiques aux appareils connectés via son serveur DHCP intégré.

### Fonctionnement au niveau réseau

L'adresse IP opère à la couche réseau (Layer 3) du modèle OSI. Son rôle est de permettre le routage des données entre différents réseaux.

Alors que l'adresse MAC vous identifie localement, l'adresse IP permet de vous joindre depuis n'importe où sur Internet.

**Exercice pratique** : Sous Windows, tapez ipconfig pour voir votre adresse IP locale et votre masque de sous-réseau. Pour connaître votre _adresse IP publique_ (celle visible sur Internet), visitez un site comme [https://www.monippublique.com/](https://www.monippublique.com/).

![Pour connaître votre adresse IP publique (celle visible sur Internet), visitez un site comme https://www.monippublique.com/](https://networkpulse.fr/content/images/2025/12/adresse-mac-adresse-ip-6.webp)

Remarquez la différence entre ces deux adresses. Essayez également la commande ipconfig /all pour voir tous les détails de votre configuration réseau, incluant la passerelle par défaut (votre routeur) et les serveurs DNS.

## **L'analogie postale : Comprendre la complémentarité**

L'adresse MAC est comme l'adresse physique de votre maison, fixe et permanente, tandis que l'adresse IP est comme votre adresse postale, qui peut changer si vous déménagez.

Cette comparaison illustre parfaitement pourquoi les deux systèmes coexistent.

Imaginez que vous envoyez un courrier : l'adresse postale (IP) permet au système postal de router votre lettre vers la bonne ville et le bon quartier, tandis que l'adresse physique (MAC) identifie précisément le bâtiment final.

Le masque de sous-réseau, quant à lui, serait comme le code postal qui définit la zone géographique.

### Scénario pratique : Le cheminement d'un paquet de données

Prenons un exemple concret. Lorsque vous accédez à un site web depuis votre ordinateur, voici ce qui se passe :

1. Votre ordinateur vérifie d'abord si l'adresse IP de destination est sur le même sous-réseau en utilisant le masque de sous-réseau
2. Si c'est le cas, il utilise le protocole ARP pour découvrir l'adresse MAC correspondante et communique directement
3. Sinon, il encapsule les données avec votre adresse MAC source et celle de votre routeur (passerelle) comme destination MAC
4. Le routeur reçoit le paquet grâce à l'adresse MAC et examine l'adresse IP de destination
5. Le routeur transmet le paquet à travers Internet en utilisant l'adresse IP
6. À chaque étape du routage, l'adresse MAC change (elle identifie le prochain saut physique), mais l'adresse IP reste la même (elle identifie la destination finale)

**Travaux pratiques** : Utilisez la commande arp -a sous Windows, Linux ou Mac pour voir la [**table ARP de votre système**](https://quick-tutoriel.com/275-utiliser-les-commandes-ping-et-arp-pour-recuperer-ladresse-mac-dun-equipement/). Cette table fait le lien entre les adresses IP et MAC des appareils de votre réseau local, illustrant parfaitement la coopération entre ces deux systèmes d'adressage.

![Utilisez la commande arp -a sous Windows, Linux ou Mac pour voir la table ARP de votre système.](https://networkpulse.fr/content/images/2026/01/commande-arp-1.webp)

Vous pouvez également utiliser la commande getmac sous Windows pour afficher toutes les adresses MAC de vos interfaces réseau avec leurs détails.

Enfin, essayez la commande ping suivie d'une adresse IP de votre réseau local, puis consultez immédiatement la table ARP : vous verrez la correspondance IP-MAC s'ajouter dynamiquement.

##  **Tableau de synthèse MAC vs IP**

![L'adresse IP : Votre localisation sur le réseau](https://networkpulse.fr/content/images/2025/12/adresse-mac-adresse-ip-5.webp)

_Astuce pédagogique : ce tableau permet de retenir rapidement les différences fondamentales entre MAC et IP._

## **Conclusion**

Les adresses MAC et IP sont complémentaires à toutes communications réseaux. La première, avec sa structure en deux parties (OUI et NIC) et ses bits spéciaux, assure l'identification au niveau physique et local.

La seconde, avec sa division en partie réseau et partie hôte définie par le masque de sous-réseau, permet le routage logique à travers des réseaux multiples.

![Les adresses MAC et IP sont complémentaires à toutes communications réseaux.](https://networkpulse.fr/content/images/2025/12/adresse-mac-adresse-ip-1.png)

Comprendre leur différence, leur structure interne, le rôle du masque de sous-réseau et leur interaction est essentiel pour tout étudiant en informatique souhaitant maîtriser les fondamentaux des réseaux.

Cette connaissance vous sera précieuse aussi bien pour le dépannage réseau, la configuration de systèmes complexes, que pour la planification et la segmentation de réseaux d'entreprise.
