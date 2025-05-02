---
link: https://linuxembedded.fr/2020/11/namespaces-la-brique-de-base-des-conteneurs
byline: Jérémy Rosen
excerpt: Introduction aux Namespaces L'une des utilisations les plus fréquentes
  de Linux consiste à monter des conteneurs. Notre OS préféré est à la fois un
  très bon hôte, mais également un très bon invité pour conteneurs. Pourtant,
  les conteneurs sont une notion inconnue du kernel. Ce sont les outils haut
  niveau (docker, nspawn, lxc) qui parlent de conteneurs. Pour le kernel, on
  parle de namespaces. Les namespaces ne sont pourtant pas des conteneurs. Les
  namespaces sont une brique utilisée pour construire des conteneurs. Nous
  allons les découvrir ensemble.
slurped: 2024-11-22T13:23
title: "Namespaces : La brique de base des conteneurs"
---

##  Introduction aux Namespaces

L'une des utilisations les plus fréquentes de Linux consiste à monter des conteneurs. Notre OS préféré est à la fois un très bon hôte, mais également un très bon invité pour conteneurs.

Pourtant, les conteneurs sont une notion inconnue du kernel. Ce sont les outils haut niveau (docker, nspawn, lxc) qui parlent de conteneurs. Pour le kernel, on parle de namespaces. Les namespaces ne sont pourtant pas des conteneurs. Les namespaces sont une brique utilisée pour construire des conteneurs. Nous allons les découvrir ensemble.

## Pourquoi les namespaces.

Dans un système Unix traditionnel, il y a un grand nombre de notions uniques, ou de valeurs qui sont communes dans tout le système. Tous les processus voient la même chose et y accèdent de la même façon, voici quelques exemples :

- le hostname est le même pour tout le monde ;
- chaque processus a un PID unique ;
- la configuration réseau (IP, ports ouverts) est commune à tous les processus sur le système ;
- tout le monde voit les mêmes montages sur les mêmes répertoires.

Si l'on tente de créer un conteneur, le fait que ces notions soient uniques va empêcher le système invité de fonctionner correctement, en effet :

- le système invité ne peut pas changer son hostname sans changer celui de l'hôte. Il faut donc que le système invité soit prudent ;
- le processus init de l'invité n'aura pas le PID 1 :
    - la plupart des systèmes d'init (sysV, systemd) refusent de démarrer s'ils ne sont pas PID 1 ;
    - comme ils ne sont pas PID 1, ils ne recevront pas les processus orphelins.
- le système invité ne peut pas changer la configuration réseau et si un port est utilisé par l'hôte, l'invité ne pourra pas l'ouvrir.

On se rend compte que faire un conteneur dans ces conditions est impossible. Les namespaces vont nous permettre de contourner ce problème en offrant des "visions" différentes de ces ressources uniques selon le processus qui en fait la demande.

## Les namespaces, comment ça marche...

Un namespace est donc une vision d'une ressource unique du noyau. Nous allons prendre le hostname comme exemple

- le hostname est le "nom" de la machine ;
- il peut être obtenu par la commande _hostname_ ;
- il peut être modifié par la commande _hostname <nouveau nom>_ à condition d'être root.

Par défaut, tous les processus de notre système voient le même hostname et si un processus modifie le hostname, il sera modifié pour tous les processus.

Pour pouvoir contourner cette limitation nous allons créer un namespace de type UTS ( pour Unix Time-Sharing ) et nous allons mettre des processus à l'intérieur. Un namespace est donc un regroupement de processus qui ont une _vision commune_ d'une ressource partagée. 

Il existe plusieurs types de namespaces correspondant aux ressources que l'on va isoler. Les namespaces UTS regroupent un certain nombre de variables globales du kernel comme le hostname, mais également les espaces de nommage des socket unix, les sémaphores posix et quelques autres.

Pour commencer, nous allons créer un namespace UTS et lancer un shell à l'intérieur :

```
# Toute les commandes doivent être lancées en tant que root
root:~$ hostname
original
root:~$ unshare --uts
root:~$ hostname
original
root:~$ hostname newname
root:~$ hostname
newname

# Dans un autre terminal
root:~$ hostname
original
```

La commande _unshare --uts_ permet de créer un nouveau namespace de type UTS et de lancer un shell à l'intérieur. Cela n'a pas changé notre hostname, mais nous pouvons le changer à _newname_.

C'est lorsque nous utilisons un autre terminal que l'on constate l'effet des namespaces. En effet le nouveau terminal est créé hors du namespace, il conserve donc le hostname d'origine.

Nous pouvons rejoindre le namespace que nous venons de créer grâce à la commande _nsenter_ :

```
# Dans le shell avec le namespace
root:~$ hostname
newname
# Récupération de notre PID 
root:~$ echo $BASHPID
500384


# Dans un autre terminal
root:~$ nsenter -a -t 500384
root:~$ hostname
newname
```

Par défaut, tous les enfants sont créés dans les même namespaces que leurs parents et les namespaces disparaissent lorsque leur dernier utilisateur se termine.

Pour connaître tous les namespaces sur notre système, nous utilisons la commande _lsns_ :

```
# Dans le shell avec le namespace
root:~$ lsns -t uts
        NS TYPE NPROCS    PID USER             COMMAND
4026531838 uts     299      1 root             /lib/systemd/systemd --system --deserialize 60
4026532233 uts       1 366509 root             /lib/systemd/systemd-udevd
4026532325 uts       1 189196 systemd-timesync /lib/systemd/systemd-timesyncd
4026532380 uts       1    701 root             /lib/systemd/systemd-logind
4026532579 uts       2 500384 root             -bash
4026532603 uts       1 436270 root             /lib/systemd/systemd-machined
```

- la première colonne est un identifiant unique du namespace ;
- la deuxième colonne est le type du namespace (nous nous sommes limités aux namespaces de type UTS) ;
- la troisième colonne indique le nombre de processus contenus dans le namespace ;
- la quatrième indique le PID du processus qui a créé le namespace ;
- la cinquième est l'utilisateur de ce processus créateur ;
- la cinquième est la commande de ce processus créateur.

Nous retrouvons bien notre bash, dans son propre namespace (le deuxième processus dans le namespace est la commande lsns elle même) et nous trouvons également le namespace racine, créé par le PID1 et qui contient la plupart des processus.

Les autres namespaces UTS sont créés par des daemons systemd qui s'auto-isolent par mesure de sécurité.

**Deux petites note de précaution à propos de lsns**

- la commande lsns obtient ses informations en lisant le contenu de /proc/_pid_/ns/, or, seul le propriétaire d'un processus a accès à ce répertoire. Il faut donc lancer cette commande en tant que root si vous voulez avoir accès à toutes les informations ;
- certains namespaces limitent la visibilité des namespaces eux mêmes. Ce n'est pas le cas avec notre exemple, mais si vous lancez cette commande dans un conteneur, vous ne verrez que les namespaces créés par ce conteneur. Si vous lancez cette commande sur l'hôte, vous verrez tous les namespaces, dont ceux des conteneurs.

## Les namespaces de PID

Maintenant que nous avons vu les namespaces avec quelque chose de simple, intéressons nous à quelque chose d'un peu plus complexe : les PID

## Rappel sur les PID

Sur un système Unix, chaque processus a un identifiant numérique unique, le PID (Processus Identifier). Le PID est fourni par le kernel au moment de la création du processus et n'est jamais modifié dans la vie du processus. Les PID sont utilisés dans de nombreux appels systèmes pour indiquer au kernel sur quel processus on veut agir. La commande kill, par exemple, utilise les PID pour identifier à quel processus envoyer un signal. Il est donc nécessaire que, à un instant donné, un PID soit assigné à un unique processus.

Parmi tous les processus du système, il y en a un particulier, celui dont le PID vaut 1

- c'est le seul processus que le kernel crée spontanément au boot, après qu'il ait fini de s'initialiser. PID1 est ensuite chargé du démarrage de tous les autres processus du système ;
- si un processus est orphelin (son parent s'est terminé avant lui), le processus est réaffecté à PID1. PID1 doit donc savoir gérer les SIGCHLD de processus qu'il n'a pas lui-même créé ;
- PID1 n'a pas le droit de se terminer. Si PID1 se termine, cela cause un kernel-panic ;
- lorsqu'un processus effectue l'appel système _reboot()_, le kernel envoie un signal SIGTERM au PID1.

Dans le cadre d'un conteneur, il est nécessaire de démarrer l'OS invité, ce qui implique d'exécuter le programme qui lui servira de PID1, mais le PID1 existe déjà, vu qu'il s'agit du PID1 de l'hôte. Ainsi, Linux fournit un type de namespace particulier, les namespaces de pid pour contourner ce problème

## Dans la pratique

Commençons par créer un namespace de PID :

```
root:~$ unshare --pid --fork --mount-proc
root:~$ echo $BASHPID
1
root:~$ bash -c 'echo $BASHPID'
4

```

Nous utilisons deux nouvelles options de unshare :

- l'option _--fork_ force unshare à créer un nouveau processus au lieu de réutiliser le sien. Cette option est nécessaire avec les pid-namespaces pour permettre à notre bash d'avoir un PID dans le nouveau namespace ;
- l'option _--mount-proc_ demande à unshare de faire le nécessaire pour que /proc soit cohérent avec le namespace de PID.

Le namespace se comporte comme prévu. Notre nouveau shell a bien le PID1 et quand nous exécutons des commandes, elles sont bien exécutées avec une nouvelle numérotation de PID. Notre shell étant le PID1 du nouveau namespace, le noyau le traite de façon un peu spéciale :

- les processus orphelins appartenant à notre namespace seront reparentés au PID1 du namespace et non pas au PID1 de l'hôte (c'est à dire Bash) ;
- si notre PID1 se termine, tous les processus appartenant à son namespace de PID seront immédiatement tués par le kernel ;
- si un processus appartenant au namespace exécute l'appel système _reboot()_, le SIGTERM sera envoyé à notre PID1 et non au PID1 de l'hôte.

Pour terminer, vérifions ce qui se passe avec lsns :

```
# Dans notre nouveau namespace de PID
root:~$ lsns --type pid
        NS TYPE NPROCS PID USER COMMAND
4026532580 pid       2   1 root -bash
```

Nous avons bien notre namespace, créé par notre processus bash, mais le namespace racine a disparu. Que se passe-t-il si nous exécutons lsns hors de notre namespace de pid :

```
#Dans le namespace racine
root:~$ lsns --type pid
        NS TYPE NPROCS    PID USER   COMMAND
4026531836 pid     258      1 root   /lib/systemd/systemd --system --deserialize 60
4026532288 pid      54 426124 jerros /opt/google/chrome/chrome --type=zygote
4026532514 pid       1 426127 jerros /opt/google/chrome/nacl_helper
4026532580 pid       1 556909 root   -bash
```

Nous retrouvons notre namespace racine (celui créé par PID1) et quelques namespaces créés par Chrome afin de protéger ses onglets. Notre namespace jouet est également là, mais le PID du processus bash n'est plus 1. Notre processus a un PID différent lorsqu'on le regarde depuis l'extérieur du namespace. Attention, pour avoir la liste de tous les namespace, il vous faudra probablement lancer la commande lsns avec sudo.

Afin de garder l'unicité des PID, les processus de notre namespace ont donc deux PID, le "faux PID" visible de l'intérieur et qui est utilisé par les processus à l'intérieur du namespace et un "vrai PID" visible et utilisé par les processus à l'extérieur. Un appel à _kill_ visant notre bash ne prendra donc pas le même PID cible selon qu'il est invoqué à l'intérieur ou à l'extérieur du namespace. Le kernel s'occupe de garder la cohérence de tout cela.

## Les namespaces réseau

Lorsque vous lancez un conteneur, votre OS invité a sa propre configuration réseau avec ses propres périphériques. Pour arriver à faire cela, votre manager de conteneurs utilise un autre type de namespace : les namespace réseau.

## La création du namespace réseau

Un namespace réseau regroupe des processus qui ont leur propre pile réseau, totalement isolée de la pile réseau du reste du système :

- ils ont leurs propres interfaces réseau ;
- ils ont leurs propres règles iptable ;
- ils ont même leur propre loopback isolé du reste du système.

Voyons comment cela se passe dans la pratique :

```
# Sur notre hôte
root:~$ ip -br link list
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 
enx8cae4cf1499c  UP             8c:ae:4c:f1:49:9c <BROADCAST,MULTICAST,UP,LOWER_UP> 

root:~$ unshare --net
root:~$ ip -br link list
lo               DOWN           00:00:00:00:00:00 <LOOPBACK> 
root:~$ ip link set lo up
root:~$ ip -br link list
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 

```

Nous commençons par regarder les interfaces réseau disponibles sur notre machine hôte :

- l'interface loopback ;
- une interface ethernet standard.

Nous entrons ensuite dans un namespace réseau. La seul interface disponible est _lo_ et elle est down (elle vient d'être créée). Nous la mettons UP immédiatement.

Notre namespace est donc isolé. Il ne peut évidemment pas communiquer avec l'extérieur, puisqu'il n'y a pas d'interface réseau.

## La création des interfaces

Il existe de nombreuses façons de connecter notre namespace au monde extérieur. Nous allons essayer la méthode la plus simple à expliquer : les MACVLAN.

- nous allons demander à notre carte ethernet d'écouter une deuxième adresse MAC ;
- nous allons faire en sorte que les paquets utilisant cette deuxième adresse MAC (entrants ou sortants) soient vu comme utilisant une autre interface réseau ;
- nous allons "débrancher" cette deuxième interface de notre hôte et nous allons la "rebrancher" dans notre conteneur.

Suite à cette manipulation, notre conteneur aura sa propre interface réseau logiquement indépendante de celle de l'hôte mais qui utilisera physiquement la même carte réseau.

```
# Dans notre namespace d'origine
root:~$  ip link add mymacvlan link enx8cae4cf1499c type macvlan mode bridge

root:~$ ip -br link list
lo                              UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 
enx8cae4cf1499c                 UP             8c:ae:4c:f1:49:9c <BROADCAST,MULTICAST,UP,LOWER_UP> 
mymacvlan@enx8cae4cf1499c       DOWN           32:9c:52:14:73:28 <BROADCAST,MULTICAST> 
```

Nous avons créé une nouvelle interface réseau (l'adresse MAC a été créée automatiquement par le noyau). Elle n'a pas d'adresse IP pour l'instant, mais commençons par la transférer vers notre nouveau namespace réseau.

```
# Dans notre namespace réseau
root:~$ echo $BASHPID
2186254

# Dans notre namespace d'origine
root:~$ ip netns attach mynamespace 2186254
root:~$ sudo ip link set mymacvlan netns mynamespace
root:~$ ip -br link list
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 
enx8cae4cf1499c  UP             8c:ae:4c:f1:49:9c <BROADCAST,MULTICAST,UP,LOWER_UP> 


# Dans notre namespace réseau
root:~$ ip -br link list
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP> 
mymacvlan@if31   DOWN           32:9c:52:14:73:28 <BROADCAST,MULTICAST> 
root:~$ dhclient mymacvlan

```

Nous commençons par récupérer le PID du shell dans notre namespace réseau afin de nommer notre espace réseau. La command _ip_ ne peut travailler qu'avec des namespaces nommés, nous commençons donc par cela.

Une fois que c'est fait, il ne nous reste plus qu'à déplacer notre nouvelle interface réseau vers notre nouveau namespace. Nous constatons ensuite que l'interface est déplacée.

Le dernier appel (dhclient) sert simplement à configurer notre interface en lui donnant une adresse IP.

## Les autres façons de partager le réseau

Nous avons utilisé la méthode MACVLAN pour partager notre carte réseau. Cette méthode est sans doute la plus facile à comprendre, mais c'est loin d'être la seul, ni la plus fréquente, en voici quelques autres :

- La méthode IPVLAN est très similaire, sauf qu'au lieu de faire gérer deux adresses MAC à notre carte réseau, nous lui faisons gérer deux adresses IP. Chaque adresse IP devient une interface réseau. Attention, il est également possible de donner deux adresses IP à une unique interface au lieu de créer deux interfaces. Cette méthode n'est pas utilisable pour partager une interface entre conteneurs.
- Lorsque vous voulez utiliser des VLAN, chaque VLAN apparaîtra comme une interface différente. Vous pouvez ensuite transférer ces interfaces vers différents namespaces. Votre conteneur sera ainsi sur un VLAN différent de votre hôte.
- Il est possible de créer une paire d'interfaces VETH. Il s'agit d'une paire d'interfaces (virtuelles) reliées par un cable (virtuel). Lorsque vous transférez l'une de ces interface vers un namespace, ce namespace pourra alors parler à l'autre interface (mais pas au réseau en général).
- Histoire de compliquer les choses, il est possible de créer un switch réseau virtuel (un bridge) et d'y connecter l'une de ces interfaces. Il est ainsi possible de créer des réseau complexes de conteneurs connectés entre eux.
- Si vous êtes vraiment motivés, [cet article](https://www.linuxembedded.fr/2021/01/emulating-wlan-in-linux-part-ii-mac80211hwsim) vous expliquera comment créer des points d'accès wifi et des client wifi virtuels que vous pouvez ensuite faire communiquer par radio (virtuel).

## Et dans les conteneurs, qui fait quoi ?

Lorsque vous utilisez un gestionnaire de conteneur complet (docker, lxc, etc.), c'est le gestionnaire de conteneur qui créera et connectera les interfaces réseau en fonction de la configuration que vous lui donnerez. Ces configurations sont généralement basées sur des bridges, eux même connectés à des interfaces physiques. 

Néanmoins, si c'est toujours le gestionnaire de conteneur qui fait la configuration des connexions physiques (c'est-à-dire les cartes virtuelles, switch virtuels et câbles virtuels), il faut également faire la configuration IP. Votre gestionnaire de conteneur peut faire ça pour vous, mais il peut également laisser les interfaces non-configurées. Dans ce deuxième cas, c'est à l'OS invité de faire le nécessaire pour configurer l'interface (via une requète DHCP par exemple).

## Les user-namespaces

Nous avons vu deux types de namespaces. Nous sommes maintenant suffisamment familiers avec les concepts pour voir un type de namespace un peu plus compliqué; les user-namespaces.

Tout comme les namespaces de PID nous ont permis de donner à certains processus une vision différente des PID des processus, les user-namespaces vont nous permettre d'avoir une vision différente des UID. Nous pourrons donc faire en sorte qu'un processus pense être exécuté en tant que UIDx mais que, pour le kernel, il est exécuté en tant que UIDy. Cela devient particulièrement intéressant lorsque UIDx = 0, c'est-à-dire que le processus pense être exécuté en tant que root, mais est en fait exécuté en tant qu'utilisateur normal.

## Une première expérience

Avant d'expliquer le fonctionnement des user-namespaces, commençons par un petit exemple.

```
# En tant qu'utilisateur normal <user>
user:~$ unshare -r
root:~$ id
uid=0(root) gid=0(root) groupes=0(root),65534(nogroup)
```

La commande _unshare -r_ nous a permis de devenir root alors que notre utilisateur est un utilisateur normal (pas de permissions sudo ou autre permissions administrateur). Mais est-ce vraiment le cas ?

```
# Avec les privilèges root que nous venons d'obtenir

root:~$ ls -l /home
total 4
drwxr-xr-x 43 root root 4096  2 mars  09:51 user

root:~$ cd /root
-bash: cd: /root/: Permission non accordée

root:~$ ls -dl /root
drwx------ 9 nobody nogroup 4096  8 févr. 19:07 /root/
```

Nous constatons que notre compte "root" ne se comporte pas vraiment comme prévu :

- les fichiers de notre utilisateur normal "user" appartiennent maintenant à root ;
- nous n'avons pas le droit d'entrer dans le répertoire /root ;
- le répertoire /root n'appartient pas à root mais à nobody.

À ce stade de notre article, on peut facilement imaginer ce qui se passe :

- nous sommes entrés dans un user-namespace ;
- le kernel "nous dit" que nous sommes root ;
- en vérité nous sommes toujours l'utilisateur "user" ;
- tous les UID qui ne sont pas ceux de "user" sont cachés derrière l'identifiant magique "nobody" mais les vraies permissions sont utilisées.

## Les user-namespaces en détail

La commande _unshare -r_ que nous avons utilisé est une commande de haut niveau qui s'occupe de configurer les namespaces pour nous. Pour configurer cela manuellement, il faut utiliser la commande _unshare -U_

```
# En tant qu'utilisateur normal
user:~$ id
uid=1000(user) gid=1000(user)

user:~$ unshare -U
nobody:~$ id
uid=65534(nobody) gid=65534(nogroup) groupes=65534(nogroup)

nobody:~$ ls -l /home
total 4
drwxr-xr-x 43 nobody nogroup 4096  2 mars  10:34 user
nobody:~$ ls -ld /root
drwx------ 9 nobody nogroup 4096  8 févr. 19:07 /root/
nobody:~$  cd /root
-bash: cd: /root: Permission non accordée
```

À ce stade là, nous sommes dans une situation bizarre. Nous sommes _nobody_, Tous les fichiers et tous les répertoires appartiennent à _nobody_, mais nous ne pouvons pas faire ce que nous voulons. Notre namespace n'est qu'à moitié configuré :

- le kernel a tout masqué (tout est à nobody) ;
- nous pouvons finir la configuration nous-même depuis l'extérieur du namespace.

```
# Depuis notre namespace
nobody:~$ echo $BASHPID
37642

# Depuis un autre shell
user:~$ echo "0       1000          1" >  /proc/37642/uid_map

#À nouveau dans le namespace
nobody:~$ id
uid=0(root) gid=65534(nogroup) groupes=65534(nogroup)
```

En écrivant dans le fichier magique _uid_map,_ nous expliquons au noyau comment nous souhaitons mapper les UID dans ce namespace :

- l'UID 0 (intérieur) est mappé à l'UID 1000 (extérieur) et un seul UID est mappé par cette ligne (il est possible de mapper tout une suite d'UID de cette façon) ;
- l'écriture doit être faite depuis l'extérieur du namespace ;
- on aurait pu écrire plusieurs lignes dans ce fichier ;
- il n'est possible d'écrire qu'une seule fois dans ce fichier ;
- il existe également un fichier _gid_map_ mais comme nous n'avons pas écrit dedans, nous utilisons toujours le groupe _nogroup._

Avec un peu de précautions, nous pouvons utiliser tout un espace d'UID non utilisés de notre hôte pour permettre à notre conteneur d'avoir son propre utilisateur root, ses propres utilisateurs etc. 

## Les vrai pouvoirs des faux root

Nous venons de voir comment l'on peut prétendre être root sans en avoir les pouvoirs, mais c'est un petit peu limité. Il serait intéressant de pouvoir faire un peu plus de choses que cela.

Commençons par nous créer un namespace UTS (pour les hostnames). Cela nous permettra de comprendre ce qui se passe.

```
user:~$ unshare -u
unshare: échec de unshare: Opération non permise
```

Un utilisateur normal n'a pas le droit de créer des namespaces. Mais si l'on fait semblant d'être root ?

```
user:~$ unshare -r
root:~$ hostname
original
root:~$ hostname newname
hostname: you must be root to change the host name
root:~$ unshare -u
root:~$ hostname newname
root:~$ hostname
newname
```

Un utilisateur normal a le droit de créer des user-namespaces et le compte "root" d'un user-namespaces a le droit de créer des namespaces au sein de son user-namespace :

- Nous créons donc un user-namespaces.
- Nous ne pouvons pas changer le _hostname_ (nous sommes un faux root).
- Nous créons un namespace UTS au sein de notre user-namespace.
- Nous pouvons désormais changer le hostname, mais cela ne se voit que dans le namespace UTS.

La création d'un namespace depuis un user-namespace créé un namespace "enfant" dans lequel le faux root récupère de nouveaux privilèges réservés à root, mais applicables uniquement aux namespaces enfants :

- création de hostnames (namespaces UTS) ;
- configuration du réseau (namespaces résea) ;
- d'autres permissions liées aux autres types de namespaces.

L'empilement des namespaces est l'une des clés du fonctionnement des conteneurs. En créant d'abord un user-namespace puis les autres namespaces dépendant, on peut donner à notre conteneurs les permissions dont il a besoin pour pouvoir gérer ses propres ressources et, par exemple, pouvoir configurer son réseau d'après sa propre configuration.

Notons que les règles complètes d'utilisation des user-namespaces sont beaucoup plus complexes que ça. Des règles aussi simples que celles décrites dans cet article permettraient une infinité de vulnérabilités. Nous invitons le lecteur à se référer à la page de man user_namespaces(7), en s'équipant au préalable d'un cachet d'aspirine.

## Les mount-namespaces

Pour finir cet article nous allons voir un dernier type de namespace qui nous permettra d'aller plus loin dans l'empilement de namespaces ; les mount-namespaces.

Les mount-namespaces permettent d'isoler la vision qu'ont les processus des montages et donc, avec un peu d'astuce, de changer complètement l'arborescence de fichiers que nos processus peuvent voir.

Un petit exemple pour commencer, et maintenant que nous connaissons les empilements de namespaces, nous allons nous en servir.

```
user:~$ unshare -r
root:~$ mkdir mymountpoint
root:~$ mount -t tmpfs none mymountpoint
root:~$ mount
[...]
none on /home/user/mymountpoint  type tmpfs (rw,relatime,uid=1000,gid=1000)


# Depuis un autre terminal
user:~$ mount
[...]
# Pas de mymountpoint
user:~$ 
```

Nous avons suffisamment d'expérience avec les namespaces pour comprendre tout ce qui se passe ici :

- nous créons un mount-namespaces ;
- nous montons un filesystem dans ce namespace ;
- ce montage n'est pas visible de l'extérieur ;
- la "vision" des montages n'est donc pas partagée.

Les mount-namespaces ne sont pas, en soi, plus compliqués que ça. Mais leur utilisation peut nous permettre de faire pas mal de choses intéressantes en utilisant certains types de montage particuliers :

- Les mount namespaces sont généralement utilisés avec l'appel système _chroot_ qui permet de changer la racine du filesystem pour un processus et ses enfants.
- Les bind-mounts permettent d'utiliser un répertoire comme source d'un montage. Le contenu du répertoire sera donc visible à deux endroits. En montant un répertoire "extérieur" à l'intérieur d'un chroot, il est possible de faire des répertoire partagés.
- Les bind-mounts permettent également de changer des propriétés des montages pour un sous-répertoire. En particulier la propriété RO/RW :
    - On peut monter un répertoire par dessus lui-même, pour le rendre read-only.
    - On peut ensuite monter un sous-répertoire de ce repertoire par dessus lui-même pour le re-rendre read-write.
- Les overlays permettent de monter un répertoire (généralement vide) par dessus un autre répertoire source :
    - Lorsqu'un fichier est disponible dans les deux répertoires, c'est la version "du dessus" qui sera utilisée.
    - Si un fichier est modifié, la nouvelle version sera sauvegardée dans le répertoire "du dessus".
    - On peut ainsi avoir plusieurs conteneurs utilisant la même arborescence de fichiers mais en ayant des copies différentes des données modifiées.

Tout ceci est amusant mais ce que root peut faire, root peut le défaire. Tous les mounts que je fais pour protéger ne servent à rien si le processus dangereux peut immédiatement appeler unmount. Sauf que, pour pouvoir faire un unmount, il faut être root, et dans le bon namespace.

```
user:~$ mkdir mymountpoint
user:~$ touch mymountpoint/fichiersecret.txt

user:~$ unshare -rm
root:~$ mount -t tmpfs none mymountpoint
root:~$ ls mymountpoint 
#Rien de visible
root:~$ unshare -r
root:~$ mount
none on /home/user/mymountpoint type tmpfs (rw,relatime,uid=1000,gid=1000)
root:~$ umount mymountpoint
umount: /home/user/mymountpoint: seul le superutilisateur peut démonter.



```

En créant un deuxième user-namespace après avoir positionné nos mounts, nous changeons de user-namespace et notre "nouveau faux root" n'est plus "le bon faux root" et il ne peut donc plus affecter les mounts du mount-namespace parent. Nous nous sommes donc protégés de ce type d'attaque.

La technique des "sandwich de user-namespaces" est la base de l'isolation des conteneurs :

- on crée un premier user-namespace pour pouvoir avoir les permissions de configuration ;
- on crée les namespaces secondaires et on positionne toutes les configurations nécessaires ;
- on crée un deuxième user-namespace pour protéger nos configurations ;
- on crée un deuxième jeu de namespaces secondaires que le conteneur pourra administrer lui-même si il le souhaite.

## Les autres types de namespaces

Nous ne détaillerons pas tous les types de namespaces, mais voici une rapide liste de ceux qui ne sont pas traités dans cet article

- **cgroup:** Ce namespace permet de limiter la vision des processus sur les control-groups. Cela permet non seulement de limiter les ressources système accessibles aux processus (les cgroup le permettent déjà) mais également de limiter la vision qu'ont les processus de ces ressources. Les processus ne peuvent donc pas savoir que certaines ressources leur sont interdites ;
- **IPC:** Ce namespace limite l'accès à un certain nombre de resssources unix qui normalement sont partagées dans tout le système :
    - les message-queues ;
    - les sémaphores ;
    - les mémoires partagées ;
    - les sockets anonymes.
- **Time:** Ce namespace isole l'heure système. Il permet donc de changer l'heure système au sein du namespace sans affecter l'hôte.

## Conclusion

Nous venons de voir les namespaces ainsi que les principes permettant aux gestionnaires de conteneurs d'isoler leurs processus de l'hôte. 

La façon de fonctionner des namespaces est à la fois simple et astucieuse, mais réussir à faire un vrai montage complet permettant de lancer un conteneur de façon sécurisée devient très vite complexe. Nous recommandons donc très fortement au lecteur d'utiliser des outils spécialisés comme LXC, systemd-nspawn ou docker. Évitez les scripts shell maisons.

Néanmoins, les namespaces sont également un très bon moyens de sécuriser une application en utilisant uniquement une partie des fonctionnalités des namespaces. De nombreuses applications telles que les navigateurs web ou les services fournis par systemd utilisent ce mécanisme pour limiter les risques en limitant ce qu'un processus donné peut faire

Si ces configurations sont faisables à la main nous recommandons d'utiliser les fonctionnalités de systemd qui vous permettent de facilement décrire ce que vous voulez. Tout cela est détaillé dans la page de man systemd.exec(5)

Enfin, signalons la commande _nsenter_ qui permet de démarrer un shell dans les mêmes namespaces qu'une commande déjà en cours d'exécution. Bien pratique pour la mise au point