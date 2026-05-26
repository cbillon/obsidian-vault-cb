---
link: https://blog.stephane-robert.info/docs/admin-serveurs/linux/references-complementaires/btrfs/
byline: Stéphane Robert
site: Stéphane ROBERT - DevSecOps Website
date: 2025-04-22T02:00
excerpt: Comprendre les concepts de Btrfs et ses usages possibles sur un système Linux.
twitter: https://twitter.com/@stephane_robert
slurped: 2026-05-23T10:17
title: Découvrir et administrer Btrfs
tags:
  - linux
  - btrfs
---

**Le système de fichiers [Btrfs](https://blog.stephane-robert.info/docs/admin-serveurs/linux/btrfs/) (B-tree File System) est un choix de plus en plus répandu sur les systèmes [Linux](https://blog.stephane-robert.info/docs/admin-serveurs/linux/) modernes. Il se distingue par ses fonctionnalités avancées comme les snapshots, la compression transparente et la gestion native de volumes en RAID. Conçu pour répondre aux besoins des environnements dynamiques et des grandes infrastructures, Btrfs vise à remplacer [ext4](https://blog.stephane-robert.info/docs/admin-serveurs/linux/ext4/) tout en offrant une flexibilité bien supérieure.**

Dans ce guide, on va apprendre à **créer**, **monter** et **administrer** un système de fichiers **Btrfs**, en tirant parti de ses outils spécifiques et de ses capacités de tuning. Si tu n’as pas encore lu le [guide d’introduction aux systèmes de fichiers Linux](https://blog.stephane-robert.info/docs/admin-serveurs/linux/stockage/choisir-systeme-fichiers/), je te recommande de le faire pour bien saisir les bases (blocs, journalisation, extents…).

Le **système de fichiers Btrfs** (B-tree File System) est né chez Oracle en 2007 avec un objectif clair : proposer un système **moderne**, **fiable** et **adapté aux gros volumes**. Il est désormais intégré dans le noyau Linux et soutenu par plusieurs distributions comme openSUSE, Fedora et Ubuntu (pour certains usages). Contrairement à ext4, Btrfs n’est pas une évolution d’un système existant, mais une refonte complète pensée pour les besoins actuels des administrateurs.

Le cœur de Btrfs repose sur le mécanisme **Copy-on-Write (CoW)** : au lieu de modifier directement les blocs, il en crée de nouveaux à chaque changement. Ce fonctionnement garantit une **meilleure intégrité des données**, permet des **snapshots** ultra-rapides et rend possible des fonctionnalités avancées comme la **déduplication**.

Ce principe a un coût en performance pour certains workloads (bases de données, VM…), mais il permet des opérations qu’ext4 ne sait pas faire sans outils tiers.

Voici ce qui distingue **Btrfs** des autres systèmes de fichiers :

- **RAID natif** : Btrfs gère le RAID 0, 1, 10 (et expérimentalement RAID 5/6) sans [LVM](https://blog.stephane-robert.info/docs/admin-serveurs/linux/lvm/) ou mdadm.
    
- **Compression transparente** (`zlib`, `lzo`, `zstd`) : active au montage.
    
- **Défragmentation en ligne** avec `btrfs filesystem defragment`.
    
- **Checksum sur les données et métadonnées** : détection des corruptions silencieuses.
    
- **Balayage automatique** (`scrub`) : vérifie régulièrement l’intégrité du disque.
    
- **Subvolumes** : chaque sous-volume peut être géré comme un FS indépendant.
    
- **Redimensionnement à chaud** : on peut agrandir ou réduire le FS monté.
    
- **Support natif des quotas** par sous-volume.
    
- **Snapshots** : capture instantanée et sans interruption d’un sous-volume.
    

- Gain d’espace avec la **compression automatique**,
- **Sauvegardes rapides** via les snapshots et `btrfs send/receive`,
- Adapté aux **containers** et aux environnements dynamiques,
- Fonctionnalités **intégrées** sans besoin d’empiler des couches (RAID, LVM…).

Aucune solution n’est parfaite. Voici les principales limites actuelles :

- Les modes **RAID 5/6** sont toujours expérimentaux (bugs non corrigés),
- Le **CoW** peut ralentir certaines écritures séquentielles intensives,
- Moins robuste qu’ext4 pour les systèmes critiques sans tuning adapté,
- **Outils d’administration** encore en évolution, moins nombreux que pour ext4.

Btrfs est idéal pour :

- les **serveurs de fichiers** avec snapshots fréquents,
- les **postes de développement** avec plusieurs containers,
- les **systèmes de sauvegarde**,
- les **disques SSD**, grâce à la compression et au TRIM.

**Btrfs**, c’est un système de fichiers qui allie **flexibilité**, **intégrité** et **fonctionnalités avancées** sans nécessiter d’empilements complexes. Mais il demande une prise en main plus rigoureuse qu’ext4. Prêt à le mettre en place ? Voyons comment le créer proprement.

Avant de profiter des **fonctionnalités avancées** de Btrfs, il faut d’abord **formater le support de stockage** avec le bon système de fichiers. Cette opération peut se faire sur un **disque entier** ou une **partition**. Btrfs n’a pas besoin de [**LVM**](https://blog.stephane-robert.info/docs/admin-serveurs/linux/stockage/lvm/) : il gère directement les volumes multiples et le RAID.

On peut créer un système **Btrfs** sur :

- un **disque brut** (`/dev/sdb`),
- une **partition** (`/dev/sdb1`),

Pour lister les disques disponibles :

S’il s’agit d’un nouveau disque, on crée une table de partitions avec `fdisk`, `parted` ou `gdisk` :

Ensuite, on crée une partition comme `/dev/sdb1`, à formater avec Btrfs.

La commande de base est :

```
sudo mkfs.btrfs /dev/sdb1
```

Quelques options utiles :

- `-L nom` : attribue un **label** au système de fichiers,
- `-d type` : définit le mode de gestion des **données** (`single`, `raid1`, etc.),
- `-m type` : définit le mode de gestion des **métadonnées**,
- `-f` : force le formatage, même si le disque est non vide.

Exemple complet :

```
sudo mkfs.btrfs -L data_btrfs -m single -d single /dev/sdb1
```

Pour plusieurs disques :

```
sudo mkfs.btrfs -L pool_btrfs /dev/sdb1 /dev/sdc1
```

Ici, Btrfs les assemble dans un volume unique avec **gestion intégrée** (comme un RAID 0 par défaut).

Pour afficher les infos sur le nouveau FS :

```
sudo btrfs filesystem show
```

Pour voir l’espace disponible et la structure du système :

```
sudo btrfs filesystem df /mnt
```

Créer un **système de fichiers Btrfs**, c’est rapide, mais les choix initiaux (compression, mode RAID, nombre de disques) influencent fortement les performances et la fiabilité. On passe maintenant à l’étape suivante : **le montage**.

Une fois le système de fichiers **Btrfs** créé, il faut le **monter** pour pouvoir l’utiliser. Le montage permet d’intégrer la partition ou le volume dans l’arborescence du système, typiquement dans `/mnt` ou `/data`. Btrfs étant orienté gestion de volumes et sous-volumes, quelques spécificités s’appliquent par rapport à ext4.

1. Créer un point de montage :

2. Monter le système de fichiers :

```
sudo mount /dev/sdb1 /mnt/data
```

3. Vérifier que tout est en place :

ou :

Ce montage est **temporaire** : il disparaît au prochain redémarrage.

Il est plus sûr de monter un volume Btrfs via son **label** ou son **UUID**, surtout si plusieurs disques sont présents.

Pour trouver ces identifiants :

Exemples :

- Monter par label :

```
sudo mount -L data_btrfs /mnt/data
```

- Monter par UUID :

```
sudo mount UUID=abcd-1234 /mnt/data
```

Pour rendre un montage **persistant**, on l’ajoute dans `/etc/fstab`. Exemple :

```
UUID=abcd-1234 /mnt/data btrfs defaults,compress=zstd 0 0
```

Explication :

- **UUID** : identifiant stable du système,
- **/mnt/data** : point de montage,
- **btrfs** : type de FS,
- **defaults,compress=zstd** : options (ici, compression activée),
- **0 0** : pas de sauvegarde avec `dump`, pas de vérification avec `fsck`.

Tester la configuration sans redémarrer :

Btrfs permet de monter un **sous-volume** spécifique (plutôt que la racine). Exemple :

```
sudo mount -o subvol=@home /dev/sdb1 /home
```

Dans `/etc/fstab`, cela donne :

```
UUID=abcd-1234 /home btrfs defaults,subvol=@home 0 0
```

Créer un sous-volume :

```
sudo btrfs subvolume create /mnt/data/@home
```

Avec Btrfs, on pense souvent en **sous-volumes** plutôt qu’en partitions : c’est un paradigme plus souple, surtout pour les sauvegardes et les snapshots. On verra dans le chapitre suivant les options de montage **spécifiques à Btrfs** pour affiner son comportement.

Le système de fichiers **Btrfs** propose de nombreuses **options de montage** permettant de configurer finement son comportement : **compression**, **défragmentation automatique**, **gestion des sous-volumes**, ou encore **optimisations pour SSD**. Bien les connaître permet d’adapter le FS à son usage réel et d’en tirer les meilleures performances.

- **`compress=zstd | lzo | zlib`** Active la **compression transparente** des fichiers. Recommandé : `zstd`, qui allie rapidité et bon taux de compression.
    
    Exemple dans `/etc/fstab` :
    
    ```
    UUID=abcd-1234 /mnt/data btrfs defaults,compress=zstd 0 0
    ```
    
- **`autodefrag`** Active la **défragmentation automatique à l'écriture**, utile pour les fichiers fréquemment modifiés (bases [SQLite](https://blog.stephane-robert.info/docs/services/bdd/relationnelles/sqlite/), VMs).
    
    Exemple :
    
    ```
    UUID=abcd-1234 /mnt/data btrfs defaults,compress=zstd,autodefrag 0 0
    ```
    
- **`noatime`** Empêche la [mise](https://blog.stephane-robert.info/docs/outils/systeme/mise/) à jour du champ d’accès des fichiers (`atime`), ce qui réduit les écritures.
    
    Exemple :
    
    ```
    UUID=abcd-1234 /mnt/data btrfs defaults,noatime 0 0
    ```
    
- **`space_cache=v2`** Accélère les montages successifs en stockant un cache d’espace libre. `v2` est recommandé pour les nouveaux systèmes.
    

- **`ssd`** Informe le FS qu’il est sur un disque SSD, ce qui active des optimisations spécifiques (alignement, TRIM...).
    
- **`discard`** Active la commande TRIM en ligne pour libérer les blocs inutilisés.
    
    Exemple :
    
    ```
    UUID=abcd-1234 /mnt/data btrfs defaults,ssd,discard 0 0
    ```
    

Ou, plus efficacement, déclencher TRIM périodiquement via `cron` :

- **`nodev`** : interdit les fichiers de type périphérique sur la partition montée.
- **`nosuid`** : empêche les bits SUID/SGID, renforçant la sécurité.
- **`noexec`** : interdit l'exécution de programmes sur la partition (utile sur `/tmp`, `/var/log`, etc.).

Exemple combiné pour `/tmp` :

```
UUID=abcd-1234 /tmp btrfs defaults,nodev,nosuid,noexec 0 0
```

- **`subvol=nom`** Permet de monter un **sous-volume spécifique**.
    
    Exemple :
    
    ```
    UUID=abcd-1234 /home btrfs defaults,subvol=@home 0 0
    ```
    
- **`subvolid=id`** Alternative par identifiant du sous-volume, peu utilisée manuellement.
    

- **`nodatacow`** Désactive le mécanisme Copy-on-Write pour certains workloads spécifiques (bases de données, VM). À utiliser avec précaution, car il désactive aussi la journalisation.
    
    Il faut le définir **au moment de la création** du fichier ou répertoire avec l’attribut `chattr` :
    
    ```
    sudo chattr +C /mnt/data/vm
    ```
    

|Option|Utilité principale|Recommandé pour|
|---|---|---|
|`compress=zstd`|Réduction de l’espace disque|Tous les systèmes|
|`autodefrag`|Réduction de la fragmentation|Fichiers modifiés fréquemment|
|`noatime`|Réduction des écritures inutiles|Systèmes fortement sollicités|
|`ssd`, `discard`|Optimisation pour SSD|Disques SSD|
|`nodev,nosuid`|Renforcement de la sécurité|`/tmp`, `/home`, disques externes|
|`subvol=@home`|Montage d’un sous-volume spécifique|Systèmes avec snapshots|

**Bien choisir ses options de montage**, c’est garantir un système Btrfs performant, stable et adapté à son usage. Dans le chapitre suivant, on verra comment **vérifier et réparer** un système Btrfs avec les outils `btrfs check` et `scrub`.

Même si **Btrfs** intègre de nombreux mécanismes de détection d’erreurs, un **système de fichiers reste vulnérable** aux coupures d’alimentation, pannes matérielles ou bugs. Il est donc essentiel de connaître les outils permettant de **vérifier** et **réparer** un système Btrfs. Les commandes clés sont `btrfs check`, `btrfs scrub` et `btrfs device`.

- Après un **crash du système** ou un redémarrage brutal,
- En cas de **comportement anormal** : fichiers manquants, montages impossibles,
- Pour une **vérification manuelle périodique** (rare mais possible),
- Si le système refuse de monter en raison d’une corruption détectée.

**Attention** : `btrfs check` ne s’utilise que sur un système de fichiers **démonté**. Ne jamais l’exécuter sur une partition montée.

Pour vérifier un système sans le réparer :

```
sudo btrfs check /dev/sdb1
```

Cela analyse la structure du système de fichiers et signale les erreurs détectées.

Si des erreurs sont signalées et que le système ne peut être monté, on peut tenter une **réparation** :

```
sudo btrfs check --repair /dev/sdb1
```

**Important** : cette commande est **destructive** et à utiliser en dernier recours. Il est fortement conseillé de faire une **sauvegarde** avant.

Pour vérifier les blocs de données **pendant que le système est monté**, on utilise `scrub`. Ce processus lit chaque bloc, vérifie les checksums et tente de corriger les erreurs en utilisant les redondances (RAID).

Lancer un scrub :

```
sudo btrfs scrub start /mnt/data
```

Suivre l’état d’avancement :

```
sudo btrfs scrub status /mnt/data
```

Planifier un scrub régulier (par exemple avec `cron`) est une bonne pratique pour détecter les erreurs silencieuses.

1. **Démarrer sur un live CD** Linux avec support Btrfs.
2. Identifier la partition avec `lsblk`.
3. Vérifier ou réparer :

```
sudo btrfs check /dev/sdb1sudo btrfs check --repair /dev/sdb1  # en dernier recours
```

4. Monter avec des options minimales ou en lecture seule si nécessaire :

```
sudo mount -o ro /dev/sdb1 /mnt
```

Btrfs n’est pas un simple système de fichiers à formater et oublier. Il offre une **palette d’outils intégrés** pour surveiller son état, **gérer l’espace**, **défragmenter**, ou encore **ajuster dynamiquement** ses paramètres selon l’usage. Voici comment garder un œil sur son bon fonctionnement et l’optimiser au quotidien.

Pour voir les volumes Btrfs montés :

```
sudo btrfs filesystem show
```

Pour afficher l’utilisation réelle de l’espace :

```
sudo btrfs filesystem df /mnt/data
```

Exemple de sortie :

```
Data, single: total=10.00GiB, used=7.50GiBMetadata, single: total=1.00GiB, used=512.00MiBSystem, single: total=32.00MiB, used=16.00MiB
```

Cela montre que les **métadonnées** et les **données** sont gérées séparément.

**Btrfs** fragmente les fichiers, surtout quand le **Copy-on-Write** est actif. Pour analyser le taux de fragmentation :

```
sudo filefrag -v /mnt/data/fichier
```

Pour défragmenter un fichier ou un répertoire :

```
sudo btrfs filesystem defragment -r /mnt/data
```

Ajouter `-clzo` ou `-czstd` applique la **compression à la volée** pendant la défragmentation :

```
sudo btrfs filesystem defragment -r -czstd /mnt/data
```

Btrfs logge ses erreurs dans le noyau. Pour les consulter :

Ou avec `journalctl` :

```
journalctl -k | grep btrfs
```

Lancer un scrub régulier est une bonne habitude pour détecter les erreurs silencieuses :

```
sudo btrfs scrub start -Bd /mnt/data
```

Option `-Bd` = bloquant (attend la fin), verbeux.

Planifier un `scrub` mensuel :

```
echo "0 3 1 * * root btrfs scrub start -Bd /mnt/data" | sudo tee -a /etc/crontab
```

Btrfs permet d’agrandir ou réduire l’espace **à chaud** (si monté) :

- Agrandir :

```
sudo btrfs filesystem resize +10G /mnt/data
```

- Réduire :

```
sudo btrfs filesystem resize -5G /mnt/data
```

- Taille absolue :

```
sudo btrfs filesystem resize 100G /mnt/data
```

Activer les quotas Btrfs :

```
sudo btrfs quota enable /mnt/data
```

Voir l’utilisation par sous-volume :

```
sudo btrfs qgroup show /mnt/data
```

On peut ainsi fixer des **limites d’usage par sous-volume**, très utile pour isoler `/home`, `/var`, etc.

L’un des atouts majeurs de **Btrfs**, c’est la possibilité de créer des **snapshots** en un clin d'œil. Un snapshot est une **copie instantanée** d’un sous-volume à un moment donné, qui ne prend de l’espace que lorsqu’il y a des modifications. C’est l’outil idéal pour les **sauvegardes**, les **tests**, ou les **restaurations rapides**.

Avant tout, assure-toi d’avoir un **sous-volume** source. Exemple :

```
sudo btrfs subvolume create /mnt/data/@home
```

Pour créer un **snapshot en lecture/écriture** :

```
sudo btrfs subvolume snapshot /mnt/data/@home /mnt/data/@home_snap
```

Pour un snapshot en **lecture seule** (idéal pour une sauvegarde) :

```
sudo btrfs subvolume snapshot -r /mnt/data/@home /mnt/data/@home_ro_snapshot
```

Pour voir tous les sous-volumes (y compris les snapshots) :

```
sudo btrfs subvolume list /mnt/data
```

Chaque snapshot est un **sous-volume indépendant**, que tu peux monter, sauvegarder ou supprimer.

Tu peux monter un snapshot comme n’importe quel sous-volume :

```
sudo mount -o subvol=@home_ro_snapshot /dev/sdb1 /mnt/snap_home
```

Attention, cette opération est irréversible. Pour supprimer un snapshot :

```
sudo btrfs subvolume delete /mnt/data/@home_snap
```

Tu peux exporter un snapshot pour le transférer ou le sauvegarder :

```
sudo btrfs send /mnt/data/@home_ro_snapshot > backup-home.btrfs
```

Et le réimporter ailleurs :

```
sudo btrfs receive /mnt/backup < backup-home.btrfs
```

Idéal pour des **sauvegardes différentielles**, avec `btrfs send -p`.

---

- Avant une **mise à jour système** : `snapshot -r /`
- Pour un **backup quotidien** de `/home`,
- Avant un **test de configuration** ou un script,
- Pour un **retour rapide à l’état précédent** (restauration en quelques secondes).

Même si **Btrfs** est un système de fichiers moderne et riche en fonctionnalités, sa **bonne configuration initiale** et sa **gestion au quotidien** font toute la différence. Voici une série de **bonnes pratiques** pour améliorer sa fiabilité, ses performances et sa résilience sur le long terme.

Plutôt que de tout stocker dans la racine du FS, crée des **sous-volumes** pour `/home`, `/var`, `/srv`, etc.

Exemple :

```
sudo btrfs subvolume create /mnt/data/@homesudo btrfs subvolume create /mnt/data/@var
```

Avantages :

- Permet de **monter indépendamment** chaque répertoire,
- Facilite la création de **snapshots ciblés**,
- Simplifie les **sauvegardes incrémentales** avec `btrfs send`.

La compression réduit l’espace disque utilisé et améliore les performances (moins d’I/O).

Ajoute l’option `compress=zstd` dans `/etc/fstab` :

```
UUID=abcd-1234 /mnt/data btrfs defaults,compress=zstd 0 0
```

Recompresse les anciens fichiers :

```
sudo btrfs filesystem defragment -r -czstd /mnt/data
```

Le **scrub** détecte les erreurs de données en arrière-plan. Lance-le régulièrement pour éviter les surprises :

```
sudo btrfs scrub start -Bd /mnt/data
```

Tu peux aussi l’automatiser via `cron` ou un [`timer systemd`](https://blog.stephane-robert.info/docs/admin-serveurs/linux/exploiter/planification/timers/).

Pour certains cas (bases de données, VMs), le **Copy-on-Write** peut nuire aux performances. Utilise alors `nodatacow` :

1. Crée un répertoire dédié :

2. Désactive le CoW avec `chattr` :

```
sudo chattr +C /mnt/data/vm
```

3. Monte ce répertoire sur un sous-volume si besoin.

**Attention** : `nodatacow` désactive aussi les checksums.

Les **groupes de quotas** permettent de contrôler la consommation de chaque sous-volume :

```
sudo btrfs quota enable /mnt/datasudo btrfs qgroup show /mnt/data
```

Tu peux aussi fixer une limite par sous-volume (`limit`).

Note les éléments suivants dans un fichier (`/etc/btrfs-setup.txt`, par exemple) :

- UUID et labels,
- Sous-volumes créés,
- Options de montage (`compress`, `autodefrag`, etc.),
- Politique de snapshots (script, fréquence),
- Quotas définis.

Cela aide énormément lors des restaurations ou migrations.

Btrfs journalise dans `dmesg` et `journalctl`. Vérifie régulièrement :

```
dmesg | grep btrfsjournalctl -k | grep btrfs
```

Tu pourras ainsi repérer à temps les débuts de corruption ou de déséquilibres RAID.

Tu as lu tout le guide sur **Btrfs** ? Parfait ! C’est maintenant le moment de vérifier ce que tu as retenu. Ce petit **contrôle de connaissances** te permet de tester ta compréhension des concepts clés : création, montage, options, réparation et bonnes pratiques. L’objectif : t’assurer que tu es prêt à utiliser Btrfs en production ou en environnement personnel, en confiance.

Validez vos connaissances avec ce quiz interactif

#### Informations

- Le chronomètre démarre au clic sur _Démarrer_
- Questions à choix multiples, vrai/faux et réponses courtes
- Vous pouvez naviguer entre les questions
- Les résultats détaillés sont affichés à la fin

Lance le quiz et démarre le chronomètre

- **10 questions** aléatoires issues de la banque de questions,
- **5 minutes** pour répondre,
- **80 %** requis pour réussir.

Ce quiz est une manière rapide de consolider tes acquis avant de mettre les mains dans le cambouis.

Savoir c’est bien, pratiquer c’est mieux. Si tu maîtrises ces questions, tu es prêt à **administrer un système Btrfs** avec assurance. Prochainement, on pourra comparer Btrfs à d’autres systèmes comme **ZFS** ou **[XFS](https://blog.stephane-robert.info/docs/admin-serveurs/linux/xfs/)**, pour mieux choisir selon les usages.