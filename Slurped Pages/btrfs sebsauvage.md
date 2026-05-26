---
link: https://sebsauvage.net/wiki/doku.php?id=btrfs
excerpt: btrfs est un système de fichiers moderne, bien plus avancé qu'ext4. btrfs est examiné ici dans le cadre d'une utilisation sur un ordinateur personnel.
tags:
  - linux
  - btrfs
slurped: 2026-05-23T10:14
title: btrfs [Wiki de sebsauvage.net]
"tags:":
---

### Table des matières

btrfs est un système de fichiers moderne, bien plus avancé qu'ext4. btrfs est examiné ici dans le cadre d'une utilisation sur un ordinateur personnel.

J'ai migré ma machine personnelle d'**ext4** vers **btrfs**, aussi bien pour le système (/) que pour mon /home. Avantages:

- Gain de place:
    
    - / et /home peuvent partager l'espace disponible.
        
    - Compression.
        
    - Déduplication.
        
    - Meilleure gestion des petits fichiers qu'ext4.
        
- Meilleures performances:
    
    - Avec la compression, il devrait y avoir globalement moins d'I/O disque (je n'ai pas un SSD, mais un disque dur à plateaux).
        
    - En btrfs, les répertoires sont indexés.
        
    - La duplication d'un fichier ou dossier existant est instantané, et n'occupe pas d'espace disque supplémentaire (non ce n'est pas de la magie).
        
- Facilité d'administration:
    
    - Snapshots quotidiens du système (/) (pratique en cas de mise à jour ou bidouillage du système qui se passe mal).
        
    - Avant toute manipulation importante sur mes données (dans /home), au lieu de faire un backup (long), je peux faire un snapshot (instantané).
        

PS: Je ne me sers pas des fonctionnalités de btrfs (snapshot distants, etc.) pour faire mes [backups](https://sebsauvage.net/wiki/doku.php?id=disque_externe "disque_externe"). Je me sers d'un autre outils ([BorgBackup](https://sebsauvage.net/wiki/doku.php?id=borgbackup "borgbackup")).

Je note ici en vrac des infos utiles ou mes interrogations.

Je ne vais pas parler ici que de certaines fonctionnalités de btrfs, mais sachez qu'on peut aller beaucoup plus loin, comme faire du RAID logiciel, faire un miroir quasi-temps réel d'un disque vers une autre machine, ou encore _remplacer à chaud un disque dur sans arrêter le système_ (!)

---

## Concepts de btrfs

Il faut voir btrfs comme un énorme gestionnaire de blocs de données (de tailles variables, jusqu'à 128 ko par défaut) gérés par un arbre binaire (d'où le nom de btrfs : "Binary Tree") permettant une recherche très rapide de ces blocs. Un fichier est décrit comme l'enchaînement d'un nombre de blocs de données. Cette particularité permet à btrfs un certain nombre de choses assez cools, comme les snapshots, la répartitions sur plusieurs supports de stockage (RAID) ou le "CoW" (Copy-on-Write).

Je ne vais pas vous énumérer ici _toutes_ les autres particularités de btrfs (support des très gros fichiers, gestion des volumes logiques, checksum des blocs de données et méta-données, indexation des répertoires, possibilité d'envoyer les snapshot sur une autre machine, etc.). Pour cela, je vous laisser regarder le [Wiki officiel](https://btrfs.wiki.kernel.org/index.php/Main_Page#Features "https://btrfs.wiki.kernel.org/index.php/Main_Page#Features").

Je ne vais ici aborder que les particularités majeurs de btrfs : Le Copy-on-Write (CoW) et les snapshots.

### CoW (Copy-on-write)

Le Copy-on-write (ou "CoW") est une particularité bien spécifique de certains systèmes de fichiers récents comme btrfs ou ZFS.

#### Fonctionnement

Avec le CoW (actif par défaut dans btrfs), toute modification d'un fichier écrit les données modifiées à côté du fichier original. Ce mécanisme permet d'assurer l'intégrité des données à tout moment. Imaginons que nous ayons un fichier constitué de blocs de données A, B, C et D:

![btrfs:cow-1.png](https://sebsauvage.net/wiki/lib/exe/fetch.php?media=btrfs:cow-1.png)
![btrfs:cow-1.png|142](app://obsidian.md/wiki/lib/exe/fetch.php?media=btrfs:cow-1.png)
Maintenant on écrit de nouvelles données dans une partie du fichier: 
  
![btrfs:cow-2.png](https://sebsauvage.net/wiki/lib/exe/fetch.php?media=btrfs:cow-2.png)

La plupart des systèmes de fichiers vont écrire les données de E par dessus C, sur place.

Avec le Copy-on-Write, btrfs va copier ces nouvelles données dans une nouvelle zone (un "extend") et _une fois la copie terminée_, il va mettre à jour les méta-données du fichier pour dire que cette partie du fichier pointe sur le nouvel extend. 

  
![btrfs:cow-3.png](https://sebsauvage.net/wiki/lib/exe/fetch.php?media=btrfs:cow-3.png)

Ainsi, il ne peut pas y avoir d'état où les données de E sont à moitié écrites dans le fichier. Votre fichier est forcément dans un état correct: Soit dans son ancienne version (A,B,C,D) soit dans la nouvelle (A,B,E,D).

Le bloc C sera ultérieurement libéré automatiquement par un processus en tâche de fond de btrfs.

#### (dé)fragmentation

Inconvénient: Plus un fichier est modifié, plus cela génère de fragmentation.

Il y a trois manières de lutter contre cette fragmentation:

1. Montez votre système de fichiers avec l'option `autodefrag` pour une défragmentation automatique.
    
2. Défragmentez manuellement des fichiers ou répertoires:
    
    sudo btrfs filesystem defragment -r /
    
3. Pour les _gros_ fichiers modifiés souvent (machines virtuelles, bases de données, conteneurs docker…), vous pouvez désactiver sélectivement CoW sur un fichier ou un répertoire.
    
    sudo chattr +C fichier
    
    - Notez que si vous désactivez CoW sur un répertoire, tout nouveau fichier créé dans ce répertoire aura également CoW désactivé.
        
    - Un fichier marqué en "NOCOW" ne pourra pas bénéficier de la compression (voir plus loin).
        

Le fait de faire une défragmentation casse les liens de déduplication. Cela n'a aucun impact sur l'intégrité des données, mais si vous aviez des fichiers dédupliqués (par exemple avec jdupes), avec la défragmentation chaque fichier se retrouve avec sa copie des données. L'occupation de l'espace disque sera donc supérieure. Il sera nécessaire de refaire une déduplication après la défragmentation pour récupérer l'espace.

#### atime

_atime_ est probablement l'une des plus mauvaises inventions d'Unix/Linux. Il enregistre la date de dernier accès à un fichier ou un répertoire. Ce qui veut dire que tout fichier **lu** va générer une **écriture disque**.

Dans la plupart des systèmes récents, l'option de montage `relatime` est active par défaut et permet de limiter ces écritures, mais dans le cas d'un système de fichier travaillant en Copy-on-Write, il vaut mieux désactiver totalement cela en utilisant les options de montage `noatime`/`nodiratime` dans votre `/etc/fstab`.

Attention: Certains outils (serveurs de mail, certains outils de backup, audit) ont besoin de la date de dernier accès au fichier. `noatime` peut donc potentiellement nuire à leur fonctionnement. D'une manière générale, sur un serveur, évitez `noatime` sur `/var/spool` et `/tmp`. (Là dans le cadre de ma machine perso, avec aucun serveur de mail qui tourne dessus, j'ai activé `noatime` partout. Quant à /tmp, il est en tmpfs (en mémoire) et non dans la partition btrfs).

#### Copie de fichier rapide

Astuce: Quand vous copiez des fichiers avec `cp` à l'intérieur de votre partition btrfs, ajoutez l'option `--reflink=auto` : Cela va utiliser CoW. Non seulement la copie du fichier sera **instantanée**, mais elle ne **consommera aucun espace disque supplémentaire** (Bien sûr cela ne fonctionne qu'à l'intérieur d'un même système de fichiers btrfs).

Exemple: `cp --reflink=auto fichier1 fichier2`

À la différence des hardlinks, modifier `fichier2` ne modifiera pas `fichier1`.

Des données ne commenceront à être écrites sur disque que si vous modifiez l'un des deux fichiers. Avant cela, ils partagent leurs blocs de données.

### Sous-volumes et snapshots

L'autre truc extrêment cool avec CoW, c'est la possibilité de faire des snapshots (des "images") instantanées de votre système de fichiers.

"sous-volumes" et "snapshots" sont en fait les mêmes entités. Sauf qu'un sous-volume peut être créé vide, alors qu'un snapshot est la copie d'un autre sous-volume ou snapshot. On peut utiliser indifféremment l'un ou l'autre.

Imaginons que vous ayez dans votre sous-volume @home (que vous avez monté en `/home`) un Fichier1:
![[Pasted image 20260525160411.png]]


Maintenant on créé une copie (un snapshot) de @home en **@backupHome**:

```
  sudo btrfs subvolume snapshot @home @backupHome
```

Le snapshot est **immédiat**, car aucune donnée n'est copiée. On se contente de dire que dans ce nouveau snapshot @backupHome, Fichier1 pointe sur les mêmes blocs de données: 
  
![btrfs:snapshot-2.png](https://sebsauvage.net/wiki/lib/exe/fetch.php?media=btrfs:snapshot-2.png)

Mais que se passe-t-il quand dans **@home** on écrit dans Fichier1 ? Et bien avec CoW, il va créer un _extend_ qui contient les nouvelles données, et faire pointer cette partie du fichier sur le nouveau bloc E, mais _uniquement dans ce sous-volume @home_. 
![[Pasted image 20260525161354.png]]

Vous pouvez donc accéder à l'ancienne version de Fichier1 dans @backupHome, et la nouvelle dans @home, sans pour autant avoir besoin d'avoir deux copies complètes des données du fichier sur disque.

Ainsi les blocs de données partagés entre sous-volumes et snapshots n'occupent pas plus de place sur disque. Ce qui prend de la place sur disque, ce sont les modifications de données.

Encore plus cool: Vous avez besoin de revenir à l'ancienne version de votre "home" ? C'est désarmant de simplicité:

```
sudo mv @home @home.old
sudo mv @backupHome @home
```

(oui, c'est aussi simple que cela !)

Bien entendu, à force d'accumuler des snapshots, cela prend de la place. Il convient de ne pas accumuler de manière inutile des snapshots sous peine de remplir son disque dur. Mais cela permet de prendre à n'importe quel moment des instantanés de votre système de fichiers, et d'y revenir très facilement en cas de problème.

Voici quelques commandes pour gérer les sous-volumes et snapshots:

- **Lister les sous-volumes et snapshots**:
    
    sudo btrfs subvolume list /
    
- Sur un Linux Mint installé en btrfs, voici les sous-volumes créés:
    
    ID 257 gen 5525 top level 5 path @
    ID 258 gen 5525 top level 5 path @home
    ID 271 gen 5434 top level 5 path timeshift-btrfs/snapshots/2019-10-28_16-47-34/@
    ID 272 gen 5525 top level 5 path timeshift-btrfs/snapshots/2019-10-29_11-11-18/@
    
- On retrouve:
    
    - La racine `@` (montée sur `/`)
        
    - Le sous-volume `@home` (monté sur `/home`)
        
    - `timeshift-btrfs/` : Les snapshots de `@` créés par Timeshift.
        
- Pour voir les **points de montage des sous-volumes**:
    
    > mount | grep /dev/sda
    /dev/sda1 on / type btrfs (rw,relatime,space_cache,subvolid=257,subvol=/@)
    /dev/sda1 on /home type btrfs (rw,relatime,space_cache,subvolid=258,subvol=/@home)
    
    - On voit bien le point de montage des différents sous-volumes (`@subvol=/…`)
        

**Manipulation des volumes et snapshots**:

- Commencez par monter votre partition btrfs:
    
    > sudo mkdir -p /mnt/sda1
    > sudo mount /dev/sda1 /mnt/sda1
    > ls -l /mnt/sda1
    total 0
    drwxr-xr-x 1 root root 242 oct.  29 15:08 @
    drwxr-xr-x 1 root root   8 oct.  29 13:24 @home
    drwxr-xr-x 1 root root 210 oct.  29 14:59 timeshift-btrfs
    
- Vous retrouvez là les sous-volumes et snapshots affichés par la commande `sudo btrfs subvolume list /`
    
- Vous pouvez voir ici vos volumes (`@` et `@home`) ainsi que les snapshot Timeshift.
    
- Vous pouvez créer un répertoire pour stocker vos snapshots:
    
    > cd /mnt/sda1
    > sudo mkdir -p snapshots
    
- (Libre à vous d'organiser votre arborescence de snapshots comme vous l'entendez.)
    
- Ensuite:
    
    - Créer un snapshot :
        
        > sudo btrfs subvolume snapshot @home snapshots/@home-2019-10-29
        
    - Créer un snapshot non modifiable: ajouter `-r` :
        
        > sudo btrfs subvolume snapshot -r @home snapshots/@home-2019-10-29
        
    - Vous pouvez vous balader dans le snapshot: c'est un simple répertoire.
        
    - Restaurer votre /home depuis le snapshot:
        
        > mv @home @home.old
        > mv snapshots/@home-2019-10-29 @home
        
    - Copier un snapshot:
        
        > sudo btrfs subvolume snapshot snapshots/@home-2019-10-29 @autreCopie
        
    - Supprimer un snapshot:
        
        > sudo btrfs subvolume delete snapshots/@home-2019-10-29
        
    - Créer un volume vide:
        
        > sudo btrfs subvolume create @kiki
        
    - Si vous ne savez plus si vous avez fait des snapshots d'un volume, faites:
        
        > sudo btrfs subvolume show @home
        @home
        	Name: 			@home
        	UUID: 			eee8706b-7178-e14b-a567-dfe5041eac1e
        	Parent UUID: 		-
        	Received UUID: 		-
        	Creation time: 		2019-10-29 13:21:37 +0100
        	Subvolume ID: 		258
        	Generation: 		5540
        	Gen at creation: 	10
        	Parent ID: 		5
        	Top level ID: 		5
        	Flags: 			-
        	Snapshot(s):
        				snapshots/@home-2019-10-29
        
        - (On voit la liste des snapshots)
            
- Savoir combien de place occupe un snapshot:
    
    - Montez votre partition btrfs (par exemple en /mnt/sda1).
        
    - > sudo btrfs filesystem du -s /mnt/sda1/timeshift-btrfs/snapshots/2019-10-30_15-38-24/@
             Total   Exclusive  Set shared  Filename
           6.53GiB   100.36MiB     4.60GiB  /mnt/sda1/timeshift-btrfs/snapshots/2019-10-30_15-38-24/@
        
    - _Exclusive_ : Quantité de données exclusives à ce snapshot (données qu'on ne retrouve pas dans d'autres snapshots).
        
    - _Set shared_ : Quantité de données de ce snapshot communes avec d'autres snapshots.
        
    - ⇒ Si vous supprimez ce snapshot, vous gagnerez donc 100.36 Mo sur disque.
        

---

## Compression

- La compression se fait par _extend_ (par blocs de données). Elle peut être active sur certains fichiers et pas d'autres.
    
- Vous pouvez compresser des fichiers et répertoires:
    
    - à la demande, avec une ligne de commande.
        
    - à l'écriture (option de montage à ajouter): Tout fichier écrit sera automatiquement compressé.
        
- Quel que soit votre choix, btrfs décompresse de manière transparente. (Vous n'avez pas besoin d'activer l'option de compression dans les options de montage pour lire les fichiers compressés).
    
- Il existe 3 algorithmes de compression : lzo (le plus rapide, impacte minimal sur le CPU), zstd et zlib (plus puissant, mais beaucoup plus gourmand en CPU).
    
- Même si vous n'avez pas activé la compression dans les options de montage, vous pouvez sans problème lire et écrire les partitions btrfs contenant des données compressées.
    

Il y a donc 2 endroits où vous pouvez compresser:

- **Dans les options de montage de la partition btrfs** (en ajoutant `compress=…` ou `compress-force=…` dans `fstab`). Cela activera la compression des données à l'écriture.
    
- **En compressant manuellement des fichiers/répertoire** (`sudo btrfs filesystem defragment -r -v -czstd /répertoire`). Vous pouvez faire cela même si l'option de compression n'est pas active dans les options de montage. btrfs saura lire et manipuler ces fichiers compressés sans problème.
    

Mon expérience:

- ~~J'ai choisi d'activer la compression lzo à l'écriture sur tout le disque.~~ Je suis passé à la compression zstd, très performante. (Et je ne vois toujours pas d'impact sur le CPU)
    
- Je ne constate aucun ralentissement, ni en lecture ni en écriture.
    
- Le disque "gratte" moins qu'en ext4.
    
- Je suis passé de 60 Go de libre (en ext4) à 242 Go de libre (sur une partition de 1 To).
    
- L'espace libre dans `/` et `/home` est désormais partagé, ce qui me laisse plus de place libre.
    

Donc en ce qui me concerne, ce n'est que du positif.

- Voir occupé/libre:
    
    btrfs filesystem usage /
    
- Compresser tous les fichiers:
    
    sudo btrfs filesystem defragment -r -v -czstd /
    
    - Exemple pratique: Sur un système Linux Mint 19.2 fraîchement installé (dans une VM):
        
        - Avant compression: _Used: 9.03 GiB / Free (estimated) : 29.06 GiB_
            
        - Après compression: _Used: 5.04 GiB / Free (estimated) : 33.11 GiB_
            
        - On gagne 4 Go d'un coup !
            
- Pour que les nouveaux fichiers créés soient compressés automatiquement, modifier `/etc/fstab` en ajoutant dans les options de montage `compress-force=zstd`.
    
- La compression a un peu d'intelligence: Si le début d'un fichier se compresse mal, btrfs ne compressera pas le fichier (même si la compression est active dans les options). btrfs ne devrait donc pas perdre de temps à essayer de compresser les archive 7z/zip, les vidéos mp4, etc.
    
- La compression s'effectue au niveau blocs et pas au niveau fichiers.
    
- La compression est parfaitement compatible avec les snapshots et la déduplication.
    
- Une fois un fichier compressé, on ne peut pas "décompresser" le fichier. On peut juste demander à btrfs de ne plus compresser les nouveaux _extends_ de ce fichier.
    
- La compression ne fonctionne pas pour les fichiers ou répertoires qui sont en _NOCOW_.
    
- Pour savoir ce que vous gagnez comme place grâce à la compression, vous pouvez utiliser `compsize` (`sudo apt install btrfs-compsize`).
    

Type       Perc     Disk Usage   Uncompressed Referenced  
TOTAL       65%      1.4G         2.1G         4.0G  

- Ce dossier contient 4 Go de données.
    
- Certains fichiers (dans différents sous-répertoires) ont un contenu identique. Avec la déduplication, il n'y a donc que 2,1 Go de données uniques dans ce répertoire.
    
- Avec la compression, ces 2,1 Go se compressent en 1,4 Go.
    
- Donc ce dossier contenant 4 Go de fichiers n'occupe que 1,4 Go sur disque.
    

---

## Déduplication

Contrairement à ZFS, btrfs ne fait pas (encore) de déduplication en temps réel. Il n'est également pas fourni avec des outils officiels pour faire la déduplication. Il faut lancer des outils tiers pour dédupliquer les données. Il en existe plusieurs: [https://btrfs.wiki.kernel.org/index.php/Deduplication#Dedicated_btrfs_deduplicators](https://btrfs.wiki.kernel.org/index.php/Deduplication#Dedicated_btrfs_deduplicators "https://btrfs.wiki.kernel.org/index.php/Deduplication#Dedicated_btrfs_deduplicators")

Cette déduplication ne peut être effectuée que sur une partition montée.

Certains outils de déduplication fonctionnent au niveau **fichier** (les fichiers identiques _en entier_ seront dédupliqués), d'autres au niveau **blocs de données** (les blocs de données identiques, même dans des fichiers différents, seront dédupliqués).

La déduplication au niveau blocs est en théorie plus efficace, mais j'ai constaté de bien meilleurs résultats avec _jdupes_ (déduplication au niveau fichiers) qu'avec _duperemove_ (déduplication au niveau blocs). Je ne me l'explique pas. Donc de manière générale, parmis les solutions proposées, préférez _jdupes_.

Voici différents outils:

---

## Fiabilité

On trouve beaucoup d'histoires de corruptions et de problème avec btrfs. Mais il semblerait qu'elles soient toutes liées à d'anciennes versions (<2016) de btrfs. Or le développement de btrfs est très actif. Afin d'éviter les problèmes, il est donc conseillé d'utiliser des noyaux Linux récents.

Facebook utilise [massivement](https://engineering.fb.com/open-source/linux/ "https://engineering.fb.com/open-source/linux/") btrfs sur ses serveurs. Je ne pense pas qu'ils soient très fan de la corruption de données.  
Les NAS de Synology utilisent aussi btrfs. La distribution Linux _Fedora_ utilise [btrfs par défaut](https://9to5linux.com/fedora-33-beta-released-with-btrfs-by-default-gnome-3-38-and-linux-5-8 "https://9to5linux.com/fedora-33-beta-released-with-btrfs-by-default-gnome-3-38-and-linux-5-8").

Ceci dit, _certaines_ fonctionnalités de btrfs ne sont pas considérées comme stable et ne devraient pas encore être utilisées (par exemple RAID56) : [https://btrfs.wiki.kernel.org/index.php/Status](https://btrfs.wiki.kernel.org/index.php/Status "https://btrfs.wiki.kernel.org/index.php/Status")

Quoi qu'il en soit, la règle suivante est toujours valable: **FAITES DES BACKUPS ![:!:](https://sebsauvage.net/wiki/lib/images/smileys/exclaim.svg)![:!:](https://sebsauvage.net/wiki/lib/images/smileys/exclaim.svg)![:!:](https://sebsauvage.net/wiki/lib/images/smileys/exclaim.svg)** 

Je ne manquerai pas d'indiquer dans cette page si j'ai des soucis avec btrfs.

---

## Performances

On trouve des avis contradictoires.

En performances pures, ext4 semble meilleur que btrfs, mais je n'ai jamais vu de comparaison avec un btrfs _compressé lzo_ (données compressées, donc en moins d'I/O disque, donc en principe plus rapide).

---

## Gotchas

- Quand vous effacez des fichiers, la libération des blocs de données est faite en arrière-plan. L'espace peut donc être libéré une minute plus tard (voir plus).
    
- Avec btrfs, il n'existe pas de moyen de connaître vraiment combien il reste d'**espace libre** (donc un `df` ne donnera pas la place libre _exacte_). Des esprits brillants se sont penchés sur le problème, et il n'existe pas de manière totalement fiable d'effectuer ce calcul à l'heure actuelle.
    
    - La commande `btrfs filesystem usage /` vous donnera sans doute des résultats plus précis que `df`.
        
- btrfs **fragmente**. Conseils:
    
    - Montez les partitions avec l'option `autodefrag`.
        
    - Désactivez CoW (`chattr +C …`) sur les gros fichiers qui sont modifiés souvent (bases de données, machines virtuelles, conteneurs docker)
        
    - Montez vos partition avec l'option `noatime`/`nodiratime`
        
    - De temps en temps, défragmentez manuellement (`sudo btrfs filesystem defragment -r /`)
        
- btrfs ne supporte pas nativement le **chiffrement**.
    
    - Utilisez LUKS/dmcrypt/VeraCrypt.
        

- Le **`commit`** par défaut en ext4 est généralement de 5 secondes. btrfs a un `commit` par défaut de 30 secondes.
    
- ~~Ne pas utiliser btrfs sur des **périphériques USB** (disque dur externe, clés USB…). La cause n'est pas vraiment btrfs lui-même mais la qualité désastreuse des pilotes USB.~~ Cette recommandation a disparu du wiki officiel de btfs, je présume donc que l'utilisation de btrfs sur périphériques USB ne pose plus problème.
    
- Quand vous faites un snapshot d'un sous-volume, cela ne fait pas de snapshot des sous-volumes qui sont à l'intérieur. (Si vous avez `@` monté en `/`, `@home` monté en `/home`, faire un snapshot de `@` ne fera **pas** un snapshot de `@home`. Le répertoire `/home` apparaîtra comme un répertoire vide dans le snapshot de `@`).
    

---

## Migration et utilisation dans Linux (Ubuntu / Linux Mint)

Le noyau Linux (et Ubuntu/Linux Mint) supporte nativement btrfs. Avantages:

- Permet d'utiliser btrfs dès l'installation.
    
- Linux Mint est fourni avec Timeshift qui prend de manière automatique des snapshots btrfs (quotidiens/hedbo/etc.)
    
    - (Dans Ubuntu, _timeshift_ peut être installé manuellement)
        
- En cas de problème, je peux booter sur la clé USB et accéder/réparer le système de fichiers (Je peux directement accéder aux snapshots btrfs pour restaurer).
    

L'**énorme** avantage des snapshots systèmes Timeshift en btrfs, par rapport à ext4, c'est qu'ils sont _instantanés_ (ça prend moins d'une seconde) et qu'ils prennent moins de place que les snapshots rsync.

---

### Migration depuis ext4

Il existe un outils de migration ext4 vers btrfs, mais il n'est absolument pas stable. J'ai donc choisi de déplacer temporairement mes fichiers sur un disque externe, reformater, et re-transférer les fichiers.

Il est fortement déconseillé d'utiliser l'outils de migration ext4➡btrfs. Il faut mieux: sortir les fichiers sur un autre support, reformater en btrfs, puis recopier les fichiers. Vous pouvez vous aider de _fsarchiver_ est extrêmement efficace pour stocker temporairement des fichiers sur un support externe.

### Partitionnement

![btrfs:btrfs-migration.png](app://obsidian.md/wiki/lib/exe/fetch.php?media=btrfs:btrfs-migration.png)

Notes:

- Quelques personnes m'ont recommandé de mettre `/` dans un `@rootfs` au lieu de la racine btrfs `@`. J'envisage éventuellement ça par la suite, mais pour le moment je laisse tel quel.
    
- La partition VeraCrypt est mise dans sa propre partition, car _a priori_ les multiples modification du fichier conteneur risquent de créer plein de données CoW pour rien (gaspillage de place et et d'I/O). Autant lui mettre sa propre partition.
    
- Je n'ai pas de partition swap, j'ai 12Go de RAM et le swap est en zram seul.
    

Planning de la migration:

1. Backup habituel de mon /home avec [borg](https://sebsauvage.net/wiki/doku.php?id=borgbackup "borgbackup") (parce que ceintures et bretelles)
    
2. Récupération d'un HD externe 1 To temporairement (le disque de l'ordinateur à migrer fait 1 To)
    
3. Formatage d'une partition en ext4 sur ce HD externe.
    
    - ![:!:](https://sebsauvage.net/wiki/lib/images/smileys/exclaim.svg) Gotcha: Formattez - _bien à l'avance de votre backup_ - votre partition de 1 To en ext4 et _laissez le disque branché_, car la création des inodes en tâche de fond prend une éternité. Je me suis fait avoir. J'ai perdu beaucoup de temps à cause de ça.
        
4. Sauvegarde de /home sur ce disque avec fsarchiver
    
    sudo fsarchiver -v -Z 1 savedir /mnt/sdb1/home.fsa /home
    
    - `savedir`=sauvegarder un répertoire, `-v` pour verbose, `-Z 1` pour utiliser la compression ztsd (très rapide).
        
    - Je ne sauvegarde pas '/' car je ré-installe le système (Je change de version de Mint).
        
    - Pourquoi _fsarchiver_ ? Il est fiable, _très_ rapide, les archives sont checksummées, et surtout si une partie de l'archive est corrompue, on ne perd QUE les fichiers de la partie corrompue, pas toute l'archive. Et puis le chiffrement me permet de ne pas laisser mes données de la partition VeraCrypt en clair sur le disque externe.
        
5. Sauvegarde du contenu de ma partition VeraCrypt avec fsarchiver (en ajoutant l'option `-c -` pour chiffrer l'archive)
    
6. Repartionnement en btrfs et ré-installation de l'OS.
    
7. Activation des options de montage des volumes btrfs dans `/etc/fstab` (`noatime,nodiratime,autodefrag,compress-force=zstd`) + reboot pour prise en compte.
    
8. Recopie des données du dd externe vers la machine (`fsarchiver restdir`)
    
9. (plus tard) Déduplication des données.
    

---

### À l'installation de Mint

À l'installation de Mint:

- Dans l'installeur, choisir le partitionnement manuel. Créer:
    
    - Une partition primaire btrfs, point de montage /
        
    - Une seconde partition primaire en ext2 (qu'on utilisera par la suite pour VeraCrypt).
        

Après installation:

- Comme on peut le voir dans `/etc/fstab`:
    
    UUID=e3dc85e3-0d2b-40e3-803b-7c2cb0bf543a /               btrfs   defaults,subvol=@ 0       1
    UUID=e3dc85e3-0d2b-40e3-803b-7c2cb0bf543a /home           btrfs   defaults,subvol=@home 0       2
    
- `/` est monté depuis la racine btrfs `@`.
    
- `/home` est monté depuis le sous-volume `@home` de btrfs.
    

L'installeur de Linux Mint met automatiquement `/home` dans un sous-volume btrfs `@home` ce qui est une excellente idée. Cela permet de faire des snapshots séparés du système et des fichiers utilisateurs. (On peut donc par exemple restaurer un snapshot du système (`@`) sans impacter les données des utilisateurs (`@home`) ; C'est d'ailleurs ce que vous proposera Timeshift).

#### Options de montage

Je monte mes sous-volumes btrfs avec ces options:

UUID=e3dc85e3-0d2b-40e3-803b-7c2cb0bf543a /               btrfs   defaults,noatime,nodiratime,autodefrag,compress-force=zstd,subvol=@ 0       1
UUID=e3dc85e3-0d2b-40e3-803b-7c2cb0bf543a /home           btrfs   defaults,noatime,nodiratime,autodefrag,compress-force=zstd,subvol=@home 0       2

- `noatime` : Ne pas enregistrer la date de dernier accès aux fichiers.
    
- `nodiratime` : Ne pas enregistrer la date de dernier accès aux répertoires.
    
- `compress-force=zstd` : Activer la compression à l'écriture.
    
- `autodefrag` : Activer la défragmentation automatique.
    

---

### Timeshift

Linux Mint est fourni avec Timeshift et encourage fortement à son utilisation.

Par défaut, Timeshift fera un backup automatisé du système (tout sauf /home) en utilisant btrfs. C'est une excellente idée, et permet de revenir en arrière en cas de mise à jour ou bidouillage du système qui pose problème, sans que cela impact les données utilisateur.

Timeshift utilisant les fonctionnalités btrfs, les snapshots sont instantanés.

Timeshit a une GUI pour simplifier la configuration automatisée des snapshots et permet aussi de les consulter facilement.

---

## Migration de version de Linux Mint par ré-installation

Quand je change de version de Linux Mint/Ubuntu, je procède par ré-installation. Cependant, même si Linux Mint supporte btrfs, l'installeur de Mint (Ubiquity) ne permet pas de spécifier des sous-volumes précis pour l'installation.

L'installeur Ubiquity supporte malgré tout btrfs, et créé automatiquement dans la partition btrfs:

- Un sous-volume `@` qui sera le `/`
    
- Un sous-volume `@home` qui sera le `/home`
    
- Il ne touchera pas aux autres sous-volumes présents qu'il ne connaît pas.
    

Voici comment procéder pour upgrader Mint par ré-installation sans perdre votre **@home**:

- Bootez sur clé USB.
    
- **Avant installation**:
    
    - Dans l'explorateur de fichiers, cliquez sur votre disque dur: Vous allez voir vos sous-volume btrfs **@** et **@home**.
        
    - Ouvrez un terminal dans ce dossier.
        
    - Renommez-les:
        
        sudo mv @ @.ancien
        sudo mv @home @home.ancien
        
    - démontez le disque.
        
- Procédez à l'installation de Linux Mint. Dans l'installeur, choisissez "Autre chose". Sélectionnez votre partition btrfs et cliquez sur le bouton "Modifier" et utilisez votre partition btrfs pour l'installation (point de montage: `/`), **mais ![:!:](https://sebsauvage.net/wiki/lib/images/smileys/exclaim.svg) ne cochez surtout pas la case _Formatter_**. L'installeur va ainsi créer des sous-volumes **@** et **@home** sans toucher aux sous-volumes que vous avez sauvegardés (_.ancien_). [![](https://sebsauvage.net/wiki/lib/exe/fetch.php?media=btrfs:btrfs-ubiquity-reuse.png)](https://sebsauvage.net/wiki/lib/exe/fetch.php?media=btrfs:btrfs-ubiquity-reuse.png "btrfs:btrfs-ubiquity-reuse.png")
    
- À l'installation, créez l'utilisateur système avec le même nom (pour ne pas avoir à retoucher aux droits dans /home)
    
- **Après la fin de l'installation**, **![:!:](https://sebsauvage.net/wiki/lib/images/smileys/exclaim.svg) ne rebootez pas**.
    
- Remontez votre partition btrfs, et renommez les sous-volumes:
    
    sudo mv @home @home.new
    sudo mv @home.ancien @home
    
- Rebootez: Vous devriez avoir votre `/home` en l'état.
    
- Pour gagner de la place, supprimer ensuite les sous-volumes @.ancien et @home.new.
    

---

## Questions

- Quelle est la maintenance régulière à faire sur un système btrfs, et à quelle fréquence ? (defrag, scrub, balance… ?)
    
    - En dehors, bien sûr, de ne pas garder inutilement plein de snapshots (mais Timeshift est bien paramétrable de ce côté là).
        
- Pour nettoyer l'espace libre (écrire des zéros), je présume qu'avec le CoW, un `sfill -fllvz /` n'est pertinent ? Par quoi remplacer ? (il y a des outils internes btrfs ?). Ou c'est juste impossible ?
    

---

## Liens, sources

## Après 7 mois sous btrfs

Voici donc un petit bilan après avoir passé ma machine personnelle d'ext4 en btrfs il y a 7 mois (j'ai migré en novembre 2019, nous sommes en juin 2020).

- **Espace libre** : J'ai beaucoup **beaucoup** plus d'espace libre qu'avant. Mais **vraiment beaucoup**. Je parle de dizaines de giga-octets gagnés sur un disque de 1 To. Grâce:
    
    - Au fait que `/` et `/home` partagent l'espace libre (fini de me dire "_aannn j'ai besoin de 20 Go en plus dans /home que je n'ai pas. Il y a 30 Go libres dans ma partition système, mais je ne peux pas l'utiliser._")
        
    - La compression (écriture/lecture transparente de données compressées)
        
    - La déduplication (les portions de fichiers identiques sont fusionnés au niveau du stockage disque)
        
- **Accès au données** : Je n'ai constaté aucun ralentissement. La consommation CPU dûe à la compression n'est pas perceptible. Et la compression réduisant les I/O disque pour les mêmes données, je suis gagnant.
    
- **Accès aux méta-données**: btrfs est bien plus efficace qu'ext4 quand il s'agit d'accéder aux méta-données. L'indexation des fichiers et répertoires est beaucoup plus rapide. Un `chmod -R` sur un énorme répertoire prend une seconde en btrfs là où ext4 mettait plus de 12 secondes.
    
- N'ayant qu'un Core-i3, j'avais choisi la compression lzo (la plus rapide). La consommation CPU n'est pas perceptible. À tel point que je vais passer à la compression zstd (meilleure compression).
    
- J'ai fait attention, tel que recommandé, à désactiver le Copy-on-write sur les gros fichiers souvent modifiés (bases de données, VM). Comme ces fichiers subissent énormément de petites écritures, cela évite la fragmentation dûe au Copy-on-write.
    
- **Maintenance**: Je réalise une fois par mois (voir moins souvent) la maintenance suivante:
    
    - Défragmentation (avec recompression éventuellement des blocs qui ne seraient pas compressés).
        
        - (Car oui, btrfs fragmente un peu à cause du Copy-On-Write, même si c'est modéré par l'option de montage _autodefrag_)
            
    - Déduplication (avec _duperemove_)
        
    - Ces deux opérations me ramènent _systématiquement_ un gain de place de plusieurs giga-octets, voir **plusieurs dizaines de giga-octets** (gain que je n'aurais pas en ext4).
        
        - (il faut dire que Steam est partagé entre plusieurs comptes sur ma machine, et que même avec un répertoire d'installation Steam partagé, ce dernier s'épanche très largement en fichiers redondants).
            
- Je n'ai eu absolument aucun incident.
    
- J'en suis tellement content que j'ai finalement passé ma machine de travail également en btrfs. Là encore le gain de place sur disque a été phénoménal.
    
    - À cette occasion, j'ai également booté sur ma clé USB pour modifier ma partition btrfs avec GParted qui l'a géré sans problème (création+déplacement+agrandissement).
        
    - J'ai la tranquillité d'esprit de savoir que je peux booter sur ma clé USB pour manipuler, sauvegarder, restaurer ma partition btrfs ainsi que les snapshots qu'elle contient (il est extrêmement facile de passer d'un snapshot à l'autre depuis la clé USB, ou même d'aller chercher un fichier précis dans un ancien snapshot).
        

Alors c'est un gros **oui**. Je continue sous btrfs. Je gagne énormément de la place, je réduis les I/O disque, et j'ai des outils de snapshot extrêmement pratiques.

Comme avec la compression la quantité globales de données écrites et lues sur disque est moindre:

- C'est un gain de performances pour les disques durs traditionnels.
    
- C'est un gain de durée de vie pour les SSD puisque cela réduit les écritures.
    

Histoire de vous donner un ordre d'idée, sur ma machine de travail:

- Mon système (`/`) fait 10 Go, mais il n'occupe que 5,8 Go sur disque grâce à la compression. C'est un gain net de 4,2 Go.
    
- Le `/home` contient 427 Go de données, mais n'occupe que 314 Go sur disque. C'est un gain de **113 Go** sans effort.
    

**PS**: Puisqu'on me pose des questions:

- **Swap** : Dans les versions récentes, btrfs supporte un fichier de swap, mais je n'ai pas pris le temps d'essayer. J'ai mis une petite partition de swap. Mais qui n'est jamais utilisée car:
    
    - J'ai activé zram (`sudo apt install zram-config` et rebootez. Constatez l'utilisation du swap zram par rapport au swap disque avec un `cat /proc/swaps`). Résultat: Cela fait bien longtemps que je n'ai plus eu d'écriture dans le swap disque, même sur ma machine qui n'avait que 4 Go de RAM. Je vous recommande chaudement l'activation de zram, même si vous avez beaucoup de RAM. Dans la pratique, cela **élimine les écritures dans le swap disque**.
        
- Comme les SSD n'aiment pas les écritures, quelques moyens de réduire les écritures:
    
    - Mettre `/tmp` et `/var/tmp` en ram avec tmpfs.
        
        - Ces répertoires ne contiennent pas beaucoup de données, mais subissent des écritures très fréquentes.
            
        - à ceux qui me diraient "_il ne faut pas mettre /var/tmp en RAM_", je dirais que tout le monde dit qu'il ne faut pas, mais sans apporter de raison solide. J'ai Googlé, je n'en ai trouvé aucune concrète à part des pages qui disent "il ne faut pas". Cela fait des mois que je tourne avec /var/tmp en tmpfs, et aucun problème à signaler.
            
    - Passer le _swappiness_ à 10 au lieu de 60.
        
    - Vous pouvez aussi regarder du côté de log2ram pour réduire la fréquence des écritures dans /var/log.