---
link: https://www.antoinefi.net/index.php/2024/09/22/laboratoire-infra-linux-ha-4/
excerpt: Cet article est le quatrième opus de la suite d’articles sur la mise en
  œuvre d’une infrastructure Linux visant à servir le blog « lab07.lab » en
  haute dispo.
slurped: 2024-12-19T09:21
title: "Laboratoire infra Linux HA #4 – NFS avec DRBD et Heartbeat – La fin des K7 📼"
---

## lab07-nfs-01 & lab07-nfs-02

Cet article est le quatrième opus de la suite d’articles sur la mise en œuvre d’une infrastructure Linux visant à servir le blog « lab07.lab » en haute dispo.

Dans le précédent article nous avons activé la réplication de la bdd wordpress entre les serveurs `lab07-db-01` et `lab07-db-02`.  
Dans cet article on va découpler le stockage du code et son exécution. Notre serveur FRONT `lab07-front-01` va continuer d’interpréter le PHP et servir nos pages web mais ne portera plus le code.  
On va monter dans un premier temps le serveur `NFS` `lab07-nfs-01` qui partagera les fichiers puis on utilisera `DRBD` pour répliquer les données de notre NFS sur un second serveur `lab07-nfs-02` et, `Heartbeat` pour orchestrer les services et l’attribution d’une IP virtuelle, une `VIP`.

L’intérêt de ces manipulations est double :

- pouvoir utiliser plusieurs serveurs FRONT pour servir rigoureusement le même code et donc avoir un emplacement unique pour mettre à jour le code (un seul `git pull` si le code est versionné),
- mettre en place une tolérance de panne du NFS en redondant le stockage.

Assez bavassé, l’infra ne va pas se monter toute seule.

## Notre premier serveur NFS

Vu qu’on est au 4ème article sur le sujet, je me doute que tu dois commencer à oublier des détails… N’est-ce pas Alzheimer !?  
Donc tu démarres `lab07-nfs-01`, tu paramètres son hostname, son adresse IP, tu adaptes `/etc/hosts` pour référencer les hôtes du lab, tu purges `nano` et, tu fais un `apt update && apt ugrapde`. C’est bon, on peut avancer maintenant ? Oui, cool !

On va installer le serveur NFS, créer un dossier, lui attribuer les droits adaptés et le partager. Ceci fait, on va copier le dossier de WordPress depuis `lab07-front-01` dans notre dossier partagé.

ShellSession

```
root@lab07-nfs-01:~# apt install nfs-kernel-server
root@lab07-nfs-01:~# mkdir -p /opt/nfsshare
root@lab07-nfs-01:~# chown nobody:nogroup /opt/nfsshare
root@lab07-nfs-01:~# chmod -R 755 /opt/nfsshare
root@lab07-nfs-01:~# echo "/opt/nfsshare   192.168.50.0/24(rw,sync,no_subtree_check)" >> /etc/exports
root@lab07-nfs-01:~# exportfs
root@lab07-nfs-01:~# rsync -avz lab07-front-01:/var/www/lab07.lab /opt/nfsshare
```

On a un serveur, il nous faut un client. On va installer le client NFS sur `lab07-front-01` et paramétrer le point de montage de façon persistante dans le `fstab`.

ShellSession

```
root@lab07-front-01:~# apt install nfs-common
root@lab07-front-01:~# vi /etc/fstab
```

VimL

```
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda2 during installation
UUID=305806a9-4e9d-4844-b3e8-702d6549ce5e /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda1 during installation
UUID=2f641569-e15b-43db-ac82-7ee8639b36ed none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
# Montage NFS de WordPress depuis lab07-nfs-01
192.168.50.27:/opt/nfsshare /var/www nfs4 defaults,user,exec,_netdev 0 0
```

On va supprimer le contenu du dossier `/var/www` de `lab07-front-01` pour ne pas se mélanger les pinceaux et on jouera à nouveau le `fstab`.

ShellSession

```
root@lab07-front-01:~# rm -rf /var/www/*
root@lab07-front-01:~# systemctl daemon-reload
root@lab07-front-01:~# mount -a
```

`lab07-front-01` sert désormais le code stocké sur `lab07-nfs-01` sans rechigner ! Vaudrait mieux quand même t’en assurer en désactivant la carte réseau de `lab07-nfs-01`.

> C’est pas forcément bien mais il sera peut-être nécessaire d’exporter notre partage à l’aide du paramètre `no_root_squash` pour que root@lab07-front soit root sur le partage NFS.

T’as testé ? Tout fonctionne ? Ok, on continue dans ce cas.

Si tu t’arrêtes à ce moment de l’histoire, voici ta situation :

![](https://www.antoinefi.net/wp-content/uploads/2024/09/lab_1nfs-2.png)

## Second serveur NFS

La haute disponibilité de notre partage NFS va être assurée par le couple formé par `DRBD` et `Heartbeat`.

`DRBD` est comparé à du RAID1 en réseau. Il prend en charge deux missions principales :

- la réplication des données d’un volume de stockage primaire (node primaire) vers un ou plusieurs serveurs secondaires (node secondaire) en mode bloc,
- contrôler le rôle du node, le volume n’est monté que sur le node primaire.

Bien qu’obsolète et remplacé par `CoroSync` (que je n’ai pas encore étudié), on va utiliser `Heartbeat` pour gérer la bascule. `Heartbeat` va s’occuper de plusieurs tâches pour nous :

- le montage du volume `DRBD`,
- la distribution de l’adresse IP virtuelle ; la `VIP`,
- l’export du partage de notre serveur `NFS`.

Dans l’absolu, tu prépares `lab07-nfs-02` comme tout les autres serveurs, tu installes ensuite `nfs-kernel-server` sans paramétrer `/etc/exports`.

### Installation et paramètres de base sur les NFS

Bon on est parti de VM identiques du coup on a une seule partoche qui prend tout le disque. `DRBD` fonctionne avec des volumes donc on va ajouter un disque secondaire sur nos deux serveurs NFS. Je te détaille pas cette partie c’est du clicodrome. Moi j’ajoute à froid un disque dur de 5 Go de chaque côté, je rallume les VMs et je créé une partition sur chacune des VM.

ShellSession

```
root@lab07-nfs-01:~# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.38.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0xae961fea.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759):

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

**Ne formate pas la partition – pas de `mkfs` !**

Ça y est, on est prêt pour l’installation et la configuration de `DRBD`. Les actions suivantes seront menées sur les deux serveurs NFS (je te montre avec `lab07-nfs-01`).

ShellSession

```
root@lab07-nfs-01:~# apt install drbd-utils
root@lab07-nfs-01:~# vi /etc/drbd.d/global_common.conf
```

Je te montre la version simplifiée sans les commentaires.

VimL

```
common {
	handlers {
		outdate-peer "/usr/lib/heartbeat/drbd-peer-outdater -t 5";
	}
	startup {
		become-primary-on lab07-nfs-01;
	}
}
```

Maintenant qu’on a décrété comment DRBD allait bosser, on va lui dire avec quoi bosser et on va le détailler dans son fichier de ressource.

ShellSession

```
root@lab07-nfs-01:~# vi /etc/drbd.d/r0.res
```

VimL

```
resource r0 {
	on lab07-nfs-01 {
		device /dev/drbd0;
		disk /dev/sdb1;
		meta-disk internal;
		address 192.168.50.27:7788;
	}

	on lab07-nfs-02 {
		device /dev/drbd0;
		disk /dev/sdb1;
		meta-disk internal;
		address 192.168.50.28:7788;
	}
}
```

Et on oublie pas de redémarrer le service pour prendre en compte les fichiers de configuration que l’on vient de modifier/créer.

ShellSession

```
root@lab07-nfs-01:~# systemctl restart drbd
```

### Création de la ressource partagée

On n’a pas ajouté notre second disque pour faire des colliers de nouilles! On va initialiser les métadonnées de stockage sur les deux serveurs.

ShellSession

```
root@lab07-nfs-01:~# drbdadm create-md r0
WARN:
  You are using the 'drbd-peer-outdater' as fence-peer program.
  If you use that mechanism the dopd heartbeat plugin program needs
  to be able to call drbdsetup and drbdmeta with root privileges.

  You need to fix this with these commands:
  dpkg-statoverride --add --update root haclient 4750 /lib/drbd/drbdsetup-84

  dpkg-statoverride --add --update root haclient 4750 /usr/sbin/drbdmeta

initializing activity log
initializing bitmap (160 KB) to all zero
Writing meta data...
New drbd meta data block successfully created.
```

La création des métadonnées nous indique des modifications à opérer pour pouvoir utiliser `drbd-peer-outdater`. On effectue ces deux commandes sur les deux NFS. On redémarre DRBD et on démarre le volume.

ShellSession

```
root@lab07-nfs-01:~# dpkg-statoverride --add --update root haclient 4750 /lib/drbd/drbdsetup-84
root@lab07-nfs-01:~# dpkg-statoverride --add --update root haclient 4750 /usr/sbin/drbdmeta
root@lab07-nfs-01:~# systemctl restart drbd
root@lab07-nfs-01:~# drbdadm up r0

root@lab07-nfs-02:~# dpkg-statoverride --add --update root haclient 4750 /lib/drbd/drbdsetup-84
root@lab07-nfs-02:~# dpkg-statoverride --add --update root haclient 4750 /usr/sbin/drbdmeta
root@lab07-nfs-02:~# systemctl restart drbd
root@lab07-nfs-02:~# drbdadm up r0
```

Normalement nos serveurs devraient avoir fait connaissance et commencer à se sentir le… je m’égare. On va faire parler le processus de `DRBD` plutôt que moi.

ShellSession

```
root@lab07-nfs-01:~# cat /proc/drbd
version: 8.4.11 (api:1/proto:86-101)
srcversion: 19D914EA50F713FCCE48607 
 0: cs:Connected ro:Secondary/Secondary ds:Inconsistent/Inconsistent C r-----
    ns:0 nr:0 dw:0 dr:0 al:8 bm:0 lo:0 pe:0 ua:0 ap:0 ep:1 wo:f oos:5241660
```

### Synchronisation des ~~montres~~ volumes

![](https://blog.antoinefi.net/wp-content/uploads/2024/08/parkerlewis.jpg)

En effet, les serveurs sont connectés mais pour l’instant chacun est secondaire et les données inconsistantes. Qu’à cela ne tienne. On va déclarer notre serveur `lab07-nfs-01` en tant que serveur primaire et écraser les éventuelles données de 02 et surveiller la quiche du process `DRBD`. Après seulement, on formatera le volume.

ShellSession

```
root@lab07-nfs-01:~# drbdadm -- --overwrite-data-of-peer primary r0
root@lab07-nfs-01:~# watch -n0.5 'cat /proc/drbd'
root@lab07-nfs-01:~# mkfs.ext4 /dev/drbd0
```

![](https://blog.antoinefi.net/wp-content/uploads/2024/08/drbd_sync.gif)

L’animation ci-dessus est accélérée au cas où tu décides de la regarder entièrement, je voulais pas que t’y passes une heure (à moins de la regarder plusieurs fois).  
Tiens pour le coup on va changer d’outil, plutôt que de faire parler le processus, on va utiliser la directive `status` de la commande `drbdadm`.

ShellSession

```
root@lab07-nfs-01:~# drbdadm status
r0 role:Primary
  disk:UpToDate
  peer role:Secondary
    replication:Established peer-disk:UpToDate
```

Tu vois, notre serveur `lab07-nfs-01` à le rôle de serveur primaire, son pair `lab07-nfs-02` est serveur secondaire et la réplication est à jour : `UpToDate`.

C’est déjà pas mal comme situation mais bon on va pas reconnecter notre partage NFS sur notre front à la main à chaque fois que le volume DRBD change de serveur. Puis il faut bien qu’on justifie d’avoir installé `Heartbeat`.

## Attribution d’une VIP au DRBD primaire mais pas que…

`Heartbeat` est un sous-système de haute disponibilité Linux (je frime mais en fait c’est la description du paquet). Tu as remarqué la quantité de paquets supplémentaires qui se sont installés lors du dernier `apt install` qu’on a fait ensemble ? La grande majorité était ajouté pour fonctionner conjointement à `Heartbeat`.

La configuration de `Heartbeat` repose sur trois fichiers :

- `/etc/ha.d/ha.cf` – Le fichier de conf globale,
- `/etc/ha.d/haresources` – La déclaration de la (des) ressource(s) partagée(s),
- `/etc/ha.d/authkeys` – Le fichier des clés d’authentification des communications.

### Le fichier de conf globale

On commence avec la création du fichier de configuration globale sur les deux serveurs. Je te montre sur `lab07-nfs-01`. Tu adapteras la ligne `ucast` sur `lab07-nfs-02`.

ShellSession

```
root@lab07-nfs-01:~# vi /etc/ha.d/ha.cf
```

VimL

```
# Heartbeat logging configuration
#logfacility daemon
logfile /var/log/heartbeat.log

# Heartbeat cluster members
node lab07-nfs-01
node lab07-nfs-02

# Heartbeat communication timing
keepalive 2
warntime 10
deadtime 20
initdead 60

# Heartbeat communication paths
udpport 694
ucast enp0s3 192.168.50.28

# fail back automatically
auto_failback On

# Monitoring of network connection to default gateway
ping 192.168.50.1   # ping to gateway for network testing
#respawn hacluster /usr/lib/heartbeat/ipfail
```

On a déclaré un fichier de journal, les nœuds, les contraintes temporelles, le moyen de communication (ici on fait de l’`unicast` vers notre pair ; on aurait pu faire du `broadcast` mais si plusieurs `Heartbeat` discutent sur un même réseau, ça fout le zbeul) et enfin on se basera sur la connectivité au routeur.

> L’`auto_failback` est une directive dépréciée et j’ai vraiment voulu te montrer une jolie infra toute belle alors j’ai lu et lu et lu et ça m’a bien cassé le zouk-machine. Alors bon utiliser une commande dépréciée sur un soft obso… On n’est pas à ça près non ?

### Le fichier de ressource(s)

Une ligne de commentaire, une ligne de configuration… Mais alors quelle ligne bien velue !

ShellSession

```
root@lab07-nfs-01:~# vi /etc/ha.d/haresources
```

VimL

```
# host          network                         resource        resource_storage                        daemon
lab07-nfs-01    IPaddr::192.168.50.32/24/enp0s3 drbddisk::r0    Filesystem::/dev/drbd0::/opt/drbd::ext4 nfs-kernel-server
```

Pour le host c’est facile. Mais le reste est assez imbitable mais je peux te décrypter ça au risque d’écrire des idioties parce que je trouve bien des tutos mais pas de `manpage` :

- `IPaddr::192.168.50.32/24/enp0s3` – Notre adresse IP virtuelle 192.168.50.32 sur l’interface enp0s3,
- `drbddisk::r0` – La ressource partagée par notre VIP,
- `Filesystem::/dev/drbd0::/opt/drbd::ext4` – Le stockage de la ressource c’est notre `/dev/drbd0` monté dans `/opt/drbd` avec un volume au format `ext4`,
- `nfs-kernel-server` – le service que va manipuler `Heartbeat`.

### Chiffrement des communications

Le fichier `authkeys` défini le chiffrement utilisé par `Heartbeat` et la clé partagée.

ShellSession

```
root@lab07-nfs-01:~# echo Jean-Claude | md5sum
8c9e41ed2d0f5cd6c3b4ed983c77854f  -
root@lab07-nfs-01:~# vi /etc/ha.d/authkeys
```

VimL

```
auth 1
1 md5 8c9e41ed2d0f5cd6c3b4ed983c77854f
```

On va utiliser la première clé de chiffrement pour chiffrer nos paquets `UDP 694` `Heartbeat` envoyés en utilisant `md5` et notre clé à nous c’est la somme md5 de la chaîne « Jean-Claude ». On peut paramétrer jusqu’à 15 clés en réception.

Sécurité oblige, le fichier `authkeys` ne doit présenter que les droits de lecture et d’écriture par `root`. Je ne fais pas le `chown` parce qu’il appartient déjà à root.

ShellSession

```
root@lab07-nfs-01:~# chmod 600 /etc/ha.d/authkeys
```

## Mise en route du bouzin

On a presque fini mais, ya un truc que je t’ai pas encore dit. C’est que si `Heartbeat` orchestre tout, bah **on ne veut pas que notre serveur NFS soit autorisé à démarrer par défaut**. C’est `Heartbeat` qui mettra ses gros doigts dedans pour s’en servir de marionnette avec ce que l’on a configuré dans `/etc/ha.d/haresources`. Alors on le désactive sur les deux serveurs NFS avant de redémarrer `Heartbeat`.

ShellSession

```
root@lab07-nfs-01:~# systemctl disable nfs-kernel-server
root@lab07-nfs-01:~# systemctl restart heartbeat
```

Si tout se passe bien, et que tu as bien collé le même mot de passe du premier coup (pas comme moi) ton serveur `lab07-nfs-01` devrait disposer de la VIP.

ShellSession

```
root@lab07-nfs-01:~# ip a
[...]
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:f1:3e:3d brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.27/24 brd 192.168.50.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet 192.168.50.32/24 brd 192.168.50.255 scope global secondary enp0s3:0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fef1:3e3d/64 scope link 
       valid_lft forever preferred_lft forever
```

Et si on éteint `Heartbeat` sur `lab07-nfs-01`, elle devrait migrer sur 02.

ShellSession

```
root@lab07-nfs-01:~# systemctl stop heartbeat
root@lab07-nfs-01:~# ip a
[...]
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:f1:3e:3d brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.27/24 brd 192.168.50.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fef1:3e3d/64 scope link 
       valid_lft forever preferred_lft forever
```

ShellSession

```
root@lab07-nfs-02:~# ip a
[...]
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:37:5f:a7 brd ff:ff:ff:ff:ff:ff
    inet 192.168.50.28/24 brd 192.168.50.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet 192.168.50.32/24 brd 192.168.50.255 scope global secondary enp0s3:0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe37:5fa7/64 scope link 
       valid_lft forever preferred_lft forever
```

## Adaptation de lab07-front-01

Magique ! Ça bascule correctement (en relançant `heartbeat` sur `lab07-nfs-01`, il est bien redevenu primaire). On va basculer les données de `/opt/nfsshare` vers `/opt/drbd` sur `lab07-nfs-01` vu que c’est le serveur `DRBD` primaire et donc qu’il a le volume de monté (il faut t’en assurer avant, pas comme moi qui me suis retrouvé avec `/opt/drbd` vide quand le volume était monté… Il passe au-dessus et oui, j’ai eu une suée pendant 2 minutes).

ShellSession

```
root@lab07-nfs-01:~# mv /opt/nfsshare/lab07.lab /opt/drbd/
```

On remplace l’ancien chemin dans le fichier d’exports NFS sur les deux serveurs `lab07-nfs-01` et `lab07-nfs-02` pour partager le nouvel emplacement (t’auras compris je te le montre que sur un) et on redémarre `Heartbeat` pour qu’il redémarre `nfs-kernel-server`.

ShellSession

```
root@lab07-nfs-01:~# sed -i 's/\/opt\/nfsshare/\/opt\/drbd/g' /etc/exports
```

Le délimiteur de `sed` c’est le slash, alors on l’échappe en le faisant précéder d’un antislash ; c’est pas hyper lisible mais on s’habitue je te rassure…

T’as pas commencé à t’habituer ? Non? Ça tombe bien, suite au [commentaire de Valentin](https://www.antoinefi.net/index.php/2024/09/22/laboratoire-infra-linux-ha-4/#comment-10) j’ai testé de remplacer le délimiteur de `sed` et non seulement ça fonctionne mais c’est surtout vachement plus lisible. Le principe c’est de prendre un caractère qui ne soit pas présent dans les chaînes que tu souhaites modifier. Voici plusieurs exemples.

ShellSession

```
root@lab07-nfs-01:~# sed -i 's,/opt/nfsshare,/opt/drbd,g' /etc/exports
root@lab07-nfs-01:~# sed -i 's=/opt/nfsshare=/opt/drbd=g' /etc/exports
root@lab07-nfs-01:~# sed -i 's!/opt/nfsshare!/opt/drbd!g' /etc/exports
```

On va pouvoir adapter le montage de notre client NFS sur `lab07-front-01`.

ShellSession

```
root@lab07-front-01:~# vi /etc/fstab
```

VimL

```
[...]
# Montage NFS de WordPress depuis la VIP des serveurs NFS lab07-nfs-01 et lab07-nfs-02
192.168.50.32:/opt/drbd /var/www nfs4 defaults,user,exec,_netdev 0 0
```

Il va falloir recharger la configuration du `fstab`, démonter et remonter le partage.

ShellSession

```
root@lab07-front-01:~# systemctl daemon-reload
root@lab07-front-01:~# umount /var/www
root@lab07-front-01:~# mount /var/www
root@lab07-front-01:~# mount | grep 32
192.168.50.32:/opt/nfsshare on /var/www type nfs4 (rw,nosuid,nodev,relatime,vers=4.2,rsize=131072,wsize=131072,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.50.23,local_lock=none,addr=192.168.50.32,user,_netdev)
```

Notre infrastructure commence à s’étoffer et j’aime bien faire des schémas alors voilà où l’on fini ce volet.

![](https://www.antoinefi.net/wp-content/uploads/2024/09/lab_2nfs-1.png)

Ça surf, notre blog WordPress fonctionne quelque soit le serveur NFS qui porte la `VIP` et le volume `DRBD` :

ShellSession

```
root@lab07-front-01:~# curl http://lab07.lab | head -n 10
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html lang="fr-FR">
<head>
	<meta charset="UTF-8" />
	<meta name="viewport" content="width=device-width, initial-scale=1" />
<meta name='robots' content='noindex, nofollow' />
<title>wordpress</title>
<link rel="alternate" type="application/rss+xml" title="wordpress » Flux" href="http://lab07.lab/?feed=rss2" />
<link rel="alternate" type="application/rss+xml" title="wordpress » Flux des commentaires" href="http://lab07.lab/?feed=comments-rss2" />
<script>
100 10129    0 10129    0     0   155k      0 --:--:-- --:--:-- --:--:--  157k
```

## La fritte sur l’macdo

On peut même regarder notre Heartbeat taper la causette.

ShellSession

```
root@lab07-nfs-02:~# tcpdump -i enp0s3 'host 192.168.50.27 and udp'
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on enp0s3, link-type EN10MB (Ethernet), snapshot length 262144 bytes
08:31:40.636823 IP lab07-nfs-01.42517 > lab07-nfs-02.694: UDP, length 305
08:31:42.629396 IP lab07-nfs-01.42517 > lab07-nfs-02.694: UDP, length 305
08:31:44.632552 IP lab07-nfs-01.42517 > lab07-nfs-02.694: UDP, length 305
[...]
root@lab07-nfs-02:~# tcpdump -i enp0s3 'host 192.168.50.28 and udp'
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on enp0s3, link-type EN10MB (Ethernet), snapshot length 262144 bytes
08:34:06.688603 IP lab07-nfs-01.42517 > lab07-nfs-02.694: UDP, length 305
08:34:08.692756 IP lab07-nfs-01.42517 > lab07-nfs-02.694: UDP, length 305
08:34:10.695719 IP lab07-nfs-01.42517 > lab07-nfs-02.694: UDP, length 305
```

![fin de la cassette](https://blog.antoinefi.net/wp-content/uploads/2024/05/oa0qcp6gow0-1024x683.jpg)

Fin de la bande