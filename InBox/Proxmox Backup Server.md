Oct 14, 2022 · 8 min read · [proxmox](https://blog.levassb.ovh/tags/proxmox "proxmox") [backup](https://blog.levassb.ovh/tags/backup "backup")  ·

![Proxmox Backup Server](https://blog.levassb.ovh/images/pbs.jpg)

Proxmox Backup Server est une solution de sauvegarde de classe entreprise particulièrement adaptée (mais pas que) à la sauvegarde des machines virtuelles. Une sorte de Veeam Backup libre…

Cet outil relativement jeune (1er stable en 2020) est basé sur Debian 11 (PBS 2.x) et écrit principalement en Rust pour garantir de bonnes performances. Il est proposé en version communautaire ou avec du support éditeur via des souscriptions.

- [forum de support communautaire](https://forum.proxmox.com/)
- [bugtracker](https://bugzilla.proxmox.com/)
- [support entreprise avec souscription](https://www.proxmox.com/en/proxmox-backup-server/pricing)

Dans ce rapide guide, je vais privilégier l’intégration avec Proxmox Virtual Environment (PVE) et l’utilisation de la ligne de commande, mais sachez que tout est réalisable depuis l’interface Web. Je ferai également un focus sur l’utilisation des `namespaces` introduits avec la version 2.x.

## Fonctionnalités principales[](https://blog.levassb.ovh/post/pbs/#fonctionnalit%C3%A9s-principales)

- sauvegarde incrémentale pour machines virtuelles, conteneurs et serveurs physiques.
- intégration étroite avec Proxmox Virtual Environment (PVE)
- chiffrement (AES-256) et compression (Zstandard) à la source
- déduplication côté serveur (par VM et entre les VMs d’un même datastore)
- synchronisation sur d’autres serveurs distants (pour respecter la règle des 3.2.1)
- restauration au niveau fichier sans devoir restaurer toute la VM
- somme de contrôle pour vérifier l’intégrité des sauvegardes (SHA-256)
- logiciel libre (GNU AGPL,v3) avec support entreprise possible (souscription)
- architecture client-serveur avec les communications chiffrées en TLS
- interface Web d’administration
- support de [QEMU dirty bitmaps](https://www.qemu.org/docs/master/interop/bitmaps.html) (ressemble à VMware CBT)
- API RESTfull
- contrôle d’accès basé sur des rôles (RBAC)
- OpenID, 2FA

## Fonctionnement[](https://blog.levassb.ovh/post/pbs/#fonctionnement)

Dans PBS, les données sont découpées en blocs (chunks) identifiés par un condensat (SHA-256) dans un index. La déduplication est basée sur la réutilisation des blocs (plusieurs index peuvent pointer vers un même bloc pour réduire le stockage nécessaire).

A chaque nouvelle sauvegarde, le client télécharge la liste des blocs déjà présents sur le serveur et ne transfère que les blocs modifiés pour reduire la bande passante réseau et la fenêtre de sauvegarde ([voir fonctionnement détaillé](https://pbs.proxmox.com/docs/technical-overview.html)).

## Installation[](https://blog.levassb.ovh/post/pbs/#installation)

Je ne m’attarde pas sur l’installation avec l’[ISO fournit par Proxmox](http://download.proxmox.com/iso/proxmox-backup-server_2.2-1.iso) qui ne présente pas de difficulté particulière.

Vous pouvez choisir d’installer PBS en machine virtuelle (si vous aimez le coté “Inception”) avec du stockage distant en NFS par exemple ou plus logiquement directement sur un serveur physique. Dans les 2 cas, il faudra provisionner la volumétrie nécessaire pour sauvegarder toutes vos VMs avec la rétention souhaitée et en tenant compte de la déduplication (ratio 2:1 minimum).

À l’issue de l’installation, vous devriez pouvoir vous connecter à l’interface web à l’adresse: `https://<fqdn ou IP>:8007`

Par défaut, le gestionnaire de paquets est configuré pour pointer vers la version `enterprise`. Si vous souhaitez passer sur la version communautaire, il faut modifier le fichier `pbs-enterprise.list`:

```
1root@pbs-server:~# cat /etc/apt/sources.list.d/pbs-enterprise.list
2
3#deb https://enterprise.proxmox.com/debian/pbs bullseye pbs-enterprise
4# PBS pbs-no-subscription repository provided by proxmox.com,
5# NOT recommended for production use
6deb http://download.proxmox.com/debian/pbs bullseye pbs-no-subscription
7
8root@pbs-server:~# apt update && apt update
```

bash

Si vous n’utilisez pas NFS, vous pouvez supprimer le service `rpcbind` qui est lancé par défaut :

```
1root@pbs-server:~# apt-get purge rpcbind
```

bash

Le fonctionnement repose sur 2 services :

```
1root@pbs-server:~# systemctl list-unit-files | grep proxmox
2proxmox-backup-proxy.service           enabled         enabled
3proxmox-backup.service                 enabled         enabled
4
5root@pbs-server:~# lsof -i -P -n
6COMMAND     PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
7proxmox-b 14519  backup   21u  IPv6  20682      0t0  TCP *:8007 (LISTEN)
8proxmox-b 27152    root   17u  IPv4  16907      0t0  TCP 127.0.0.1:82 (LISTEN)
9proxmox-b 27152    root   18u  IPv4  16907      0t0  TCP 127.0.0.1:82 (LISTEN)
```

bash

- le service `proxmox-backup-proxy` expose l’API PBS en HTTPS sur le port 8007 (tcp). Il tourne avec des permissions restreintes (utilisateur `backup`) et transmet les opérations qui nécéssitent des permissions étendues au service `proxmox-backup`
    
- le service `proxmox-backup` expose l’API de management sur le port 82 uniquement en localhost.
    

## Ajout d’un Datastore en ZFS[](https://blog.levassb.ovh/post/pbs/#ajout-dun-datastore-en-zfs)

Un `datastore` est l’emplacement où sont stockées les sauvegardes. Un serveur PBS doit avoir au moins un `datastore` configuré. Il peut être constitué d’une grappe RAID formatée en XFS ou EXT4, un point de montage NFS ou mieux encore : un pool ZFS !

Si vous n’êtes pas familiarisé avec ZFS, pas de panique: PBS permet de gérer la création du pool directement depuis l’interface WEB. Vous pouvez aussi vous référer à mes [précédents articles](https://blog.levassb.ovh/post/zfs/).

Pour cette prise en main, je vais juste créer un pool ZFS avec deux disques de 1TB en miroir (RAID-1). En production, Proxmox recommande l’ajout du SSD en [special device ZFS](https://pbs.proxmox.com/docs/sysadmin.html#local-zfs-special-device) pour améliorer les performances.

- Afficher la liste des disques disponibles

```
 1root@pbs-server:~# proxmox-backup-manager disk list
 2┌──────┬────────┬─────┬───────────┬───────────────┬───────────────┬─────────┬────────┐
 3│ name │ used   │ gpt │ disk-type │          size │ model         │ wearout │ status │
 4╞══════╪════════╪═════╪═══════════╪═══════════════╪═══════════════╪═════════╪════════╡
 5│ sda  │ lvm    │   1 │ hdd       │   34359738368 │ QEMU_HARDDISK │       - │ passed │
 6├──────┼────────┼─────┼───────────┼───────────────┼───────────────┼─────────┼────────┤
 7│ sdb  │ unused │   0 │ hdd       │ 1099511627776 │ QEMU_HARDDISK │       - │ passed │
 8├──────┼────────┼─────┼───────────┼───────────────┼───────────────┼─────────┼────────┤
 9│ sdc  │ unused │   0 │ hdd       │ 1099511627776 │ QEMU_HARDDISK │       - │ passed │
10└──────┴────────┴─────┴───────────┴───────────────┴───────────────┴─────────┴────────┘
```

bash

- Création du pool ZFS. L’argument `local-zfs` est à la fois le nom du pool ZFS et celui du datastore PBS.

```
 1root@pbs-server:~# proxmox-backup-manager disk zpool create local-zfs --devices sdb,sdc --raidlevel mirror --add-datastore
 2create Mirror zpool 'local-zfs' on devices 'sdb,sdc'
 3# "zpool" "create" "-o" "ashift=12" "-m" "/mnt/datastore/local-zfs" "local-zfs" "mirror" "sdb" "sdc"
 4# "zfs" "set" "relatime=on" "local-zfs"
 5
 6Chunkstore create: 1%
 7Chunkstore create: 2%
 8---8<--
 9Chunkstore create: 98%
10Chunkstore create: 99%
11TASK OK
```

...

bash

- Vérification du pool ZFS et du datastore PBS

```
 1root@pbs-server:~# zpool status
 2  pool: local-zfs
 3 state: ONLINE
 4config:
 5	NAME        STATE     READ WRITE CKSUM
 6	local-zfs   ONLINE       0     0     0
 7	  mirror-0  ONLINE       0     0     0
 8	    sdb     ONLINE       0     0     0
 9	    sdc     ONLINE       0     0     0
10
11root@pbs-server:~# proxmox-backup-manager datastore list
12┌───────────┬──────────────────────────┬─────────┐
13│ name      │ path                     │ comment │
14╞═══════════╪══════════════════════════╪═════════╡
15│ local-zfs │ /mnt/datastore/local-zfs │         │
16└───────────┴──────────────────────────┴─────────┘
```

...

bash

A ce stade, vous devriez obtenir quelque chose comme ça :

![Datastore PBS](https://blog.levassb.ovh/images/pbs-datastore.png)

Figure 2: Datastore PBS

## Ajout des namespaces:[](https://blog.levassb.ovh/post/pbs/#ajout-des-namespaces)

Les `namespaces` ont été introduits avec PBS 2.x et permettent de structurer un datastore en plusieurs éléments sur lesquels vous pouvez positionner des politiques de rétentions et des permissions d’accès différentes. Cela permet de prendre en charge 2 cas de figure particuliers :

- la sauvegarde de plusieurs serveurs Proxmox isolés ou plusieurs clusters en évitant les conflits (vmid).
- la délégation de l’espace d’un datastore à d’autres PBS pour faire de la synchronisation (2 PBS d’extrémités qui se synchronisent sur un PBS central avec 2 namespaces).

L’intérêt des namespaces est de conserver la déduplication entre les VMs d’un même datastore (sauf si vous activez le chiffrement à la source : dans ce cas, les sommes de contrôles d’un même bloc sont différentes).

Pour la création des namespaces, il va falloir passer par l’interface Web puisque pour le moment, il n’existe pas d’outil en ligne de commande (j’ai cherché un moment : voir flèche rouge) :

![Namespace PBS](https://blog.levassb.ovh/images/pbs-namespace.png)

Figure 3: Namespace PBS

Ici, j’ai créé un namespace pour chacun des serveurs Proxmox VE (pve-01 à pve-03) à sauvegarder.

## Configurer la rétention (prune)[](https://blog.levassb.ovh/post/pbs/#configurer-la-r%C3%A9tention-prune)

La rétention (délais de conservation des sauvegardes) peut être définie globalement niveau `datastore` ou plus finement au niveau `namespace` pour avoir des politiques différentes en fonction des clients de sauvegardes.

Pour planifier au mieux vos rétentions, le plus simple est d’utiliser le [simulateur](https://pbs.proxmox.com/docs/prune-simulator/index.html) fourni.

![Namespace PBS](https://blog.levassb.ovh/images/pbs-prune.png)

Figure 4: Namespace PBS

Dans cet exemple, PBS conserve les sauvegardes sur 1 an avec:

- planification quotidienne décalée des purges (20:00 pour pve-01 à 00:00 pour pve-03)
- 2 semaines de sauvegardes quotidiennes
- 2 mois de sauvegardes hebdomadaires
- 1 an de sauvegardes mensuelles.

## Purger les données (garbage collector)

Dans PBS, l’opération de `prune` traite uniquement les index et pas les blocs de données du datastore. Cette tâche revient au `garbage collector` qu’il faut planifier régulièrement.

Voir l’exemple dans la capture ci-dessus avec une planification chaque samedi à 18h15 (l’opération peut être longue sur un gros `datastore`).

## Créer des utilisateurs dédiés

L’utilisation du compte `root@pam` du serveur PBS pour la connexion des clients de sauvegarde est évidemment à proscrire. Je vais créer un utilisateur distinct pour chaque serveur à sauvegarder et l’associer son namespace (pve-01 dans l’exemple).

```
 1root@pbs-server:~# proxmox-backup-manager user create user-pve-01@pbs --password VerySecret1 --comment "backup user for pve-01"
 2
 3root@pbs-server:~# proxmox-backup-manager user list
 4┌─────────────────┬────────┬────────┬───────────┬──────────┬───────────────────┬────────────────────────┐
 5│ userid          │ enable │ expire │ firstname │ lastname │ email             │ comment                │
 6╞═════════════════╪════════╪════════╪═══════════╪══════════╪═══════════════════╪════════════════════════╡
 7│ root@pam        │      1 │  never │           │          │ admin@domain.tld  │                        │
 8├─────────────────┼────────┼────────┼───────────┼──────────┼───────────────────┼────────────────────────┤
 9│ user-pve-01@pbs │      1 │  never │           │          │                   │ backup user for pve-01 │
10├─────────────────┼────────┼────────┼───────────┼──────────┼───────────────────┼────────────────────────┤
11│ user-pve-02@pbs │      1 │  never │           │          │                   │ backup user for pve-02 │
12├─────────────────┼────────┼────────┼───────────┼──────────┼───────────────────┼────────────────────────┤
13│ user-pve-03@pbs │      1 │  never │           │          │                   │ backup user for pve-03 │
14└─────────────────┴────────┴────────┴───────────┴──────────┴───────────────────┴────────────────────────┘
```

...

bash

Et accorder le rôle [DatastoreBackup](https://pbs.proxmox.com/docs/user-management.html#access-roles) sur leurs namespaces respectifs (pve-01 dans l’exemple):

```
 1root@pbs-server:~# proxmox-backup-manager acl update /datastore/local-zfs/pve-01 DatastoreBackup --auth-id user-pve-01@pbs
 2
 3root@pbs-server:~# proxmox-backup-manager acl list
 4┌─────────────────┬─────────────────────────────┬───────────┬─────────────────┐
 5│ ugid            │ path                        │ propagate │ roleid          │
 6╞═════════════════╪═════════════════════════════╪═══════════╪═════════════════╡
 7│ user-pve-01@pbs │ /datastore/local-zfs/pve-01 │         1 │ DatastoreBackup │
 8├─────────────────┼─────────────────────────────┼───────────┼─────────────────┤
 9│ user-pve-02@pbs │ /datastore/local-zfs/pve-02 │         1 │ DatastoreBackup │
10├─────────────────┼─────────────────────────────┼───────────┼─────────────────┤
11│ user-pve-03@pbs │ /datastore/local-zfs/pve-03 │         1 │ DatastoreBackup │
12└─────────────────┴─────────────────────────────┴───────────┴─────────────────┘
```

...

bash

## Configurer Proxmox VE avec PBS[](https://blog.levassb.ovh/post/pbs/#configurer-proxmox-ve-avec-pbs)

Pour commencer, il faut récupérer l’empreinte de la clé publique du certificat TLS du serveur PBS (attention, si vous décidez d’utiliser autre chose que les certificats auto-signés générés à l’installation)

```
1root@pbs-server:~# proxmox-backup-manager cert info | grep Fingerprint
2Fingerprint (sha256): 1c:3b:0e:80:9a:da:dc:26:31:92:2e:76:f6:2d:0b:52:85:0e:e0:6f:b7:cd:af:e7:25:55:ef:c2:5c:d4:e9:f9
```

bash

et ajouter le serveur PBS sur les serveurs Proxmox VE (pve-01 dans l’exemple):

```
1root@pve-01:~# pvesm add pbs pbs-server --datastore local-zfs --server <IP server PBS> --fingerprint 1c:3b:0e:80:9a:da:dc:26:31:92:2e:76:f6:2d:0b:52:85:0e:e0:6f:b7:cd:af:e7:25:55:ef:c2:5c:d4:e9:f9 --username user-pve-01@pbs --password --namespace pve-01
2Enter Password: **********
```

bash

Dans l’interface Web de votre serveur Proxmox VE :

![PBS on PVE](https://blog.levassb.ovh/images/pbs-pve.png)

Figure 5: PBS on PVE

## Planifier une sauvegarde[](https://blog.levassb.ovh/post/pbs/#planifier-une-sauvegarde)

La planification des sauvegardes se configure directement depuis l’interface Web de Proxmox VE:

![backup schedule on PVE](https://blog.levassb.ovh/images/pbs-pve-schedule.png)

Figure 6: backup schedule on PVE

Dans cette exemple, le serveur Proxmox VE va faire une sauvegarde de toutes les VMs tous les jours à partir de 21h00 sur le serveur PBS en mode snapshot (sans interruption de service).

## Restauration[](https://blog.levassb.ovh/post/pbs/#restauration)

La restauration est toute aussi simple, avec la possibilité de restaurer l’ensemble de la VM / conteneur ou simplement un fichier en parcourant l’arborescence.

![file level recovery](https://blog.levassb.ovh/images/pbs-file-level-recovery.png)

Figure 7: file level recovery

## Conclusion[](https://blog.levassb.ovh/post/pbs/#conclusion)

PBS est une solution incontournable pour la sauvegarde des environnements Proxmox VE. Les planifications proposées dans cet article doivent être adaptées à vos besoins. Cet article ne couvre pas la gestion des vérifications, les sauvegardes de serveurs en mode fichier et la synchronisation sur un second serveur distant…bref encore un peu de grain à moudre à l’occasion de la mise en service de l’infrastructure de production.

## Ressources:[](https://blog.levassb.ovh/post/pbs/#ressources)

- [PBS Documentation (EN)](https://pbs.proxmox.com/docs/index.html)
- [Blog Notamax (FR)](https://notamax.be/proxmox-backup-server-presentation-et-premiers-pas/)
- [Simulateur (EN)](https://pbs.proxmox.com/docs/prune-simulator/index.html)
- [Wiki Ordinoscope (FR)](https://www.ordinoscope.net/index.php/Informatique/Softwares/Proxmox/PBS)

Photo by [Jason Pofahl](https://unsplash.com/@jasonpofahlphotography) on [Unsplash](https://unsplash.com/photos/zLtXrNXJpKM)