---
tags:
  - Proxmox
  - lxc
---
Si vous utilisez Proxmox uniquement pour faire tourner des VMs, vous passez à côté de la moitié de sa puissance. C'est un peu comme si vous achetiez une console next-gen pour jouer uniquement aux jeux rétro. Sympa, mais vous n'exploitez pas tout le potentiel de la machine.

Les conteneurs LXC (Linux Containers) sont le secret le mieux gardé de Proxmox VE. Là où Docker a conquis le monde des microservices, LXC joue dans une catégorie différente : celle des **conteneurs système**. Un LXC, c'est un Linux complet, avec son init, ses services, ses utilisateurs, mais sans le poids d'une machine virtuelle.

On ne va pas se mentir : la plupart des admins connaissent LXC de nom, mais rares sont ceux qui l'exploitent vraiment en production. Pourtant, pour des services comme Nginx, PostgreSQL ou Pi-hole, c'est souvent le choix le plus judicieux.

#### LXC, c'est quoi exactement ?

Un conteneur LXC, c'est une instance Linux isolée qui partage le noyau de l'hôte. Contrairement à une VM qui embarque son propre kernel, son BIOS virtuel et tout le tralalà, un LXC démarre directement sur le kernel Proxmox.

Concrètement, ça donne :

- **Démarrage en 2 à 5 secondes** (contre 30 à 60 secondes pour une VM KVM)
- **Consommation mémoire de 100 à 500 Mo** (contre 2 à 4 Go pour une VM équivalente)
- **Overhead CPU de 1 à 3 %** (contre 5 à 10 % pour KVM)

C'est un peu comme la différence entre lancer un jeu natif et le faire tourner dans un émulateur. Les deux fonctionnent, mais l'un est nettement plus réactif.

##### LXC vs Docker : deux philosophies

Attention, LXC et Docker ne jouent pas dans la même cour. Docker encapsule **une application** (un processus, un service). LXC encapsule **un système complet** (systemd, cron, SSH, plusieurs services).

|Critère|LXC|Docker|
|---|---|---|
|Granularité|Système complet|Application unique|
|Init system|systemd / OpenRC|Pas d'init (PID 1 = app)|
|Gestion réseau|IP propre, bridge natif|Réseau overlay, NAT|
|Persistance|Filesystem complet|Volumes montés|
|Cas d'usage|Services infra|Microservices applicatifs|

Pour aller plus loin sur les runtimes de conteneurs, jetez un œil à notre article sur [containerd, le runtime qui propulse Kubernetes](https://www.shpv.fr/blog/containerd-runtime/).

#### VM vs LXC : quand choisir quoi ?

C'est LA question que tout admin Proxmox se pose. La réponse courte : **LXC par défaut, VM quand c'est nécessaire**.

##### Choisir LXC quand :

- Le service tourne sous **Linux** (pas de Windows, pas de BSD)
- Vous n'avez **pas besoin de pass-through GPU/USB**
- Le service ne nécessite **pas un kernel spécifique**
- Vous voulez **maximiser la densité** sur votre hyperviseur
- Le service est un **composant d'infrastructure** (DNS, reverse proxy, base de données, monitoring)

##### Choisir une VM quand :

- Vous avez besoin de **Windows ou BSD**
- Le workload nécessite un **kernel custom** ou des modules spécifiques
- Vous avez besoin de **pass-through matériel** (GPU, carte réseau SR-IOV)
- L'isolation de sécurité est **critique** (multi-tenant, environnement hostile)
- Vous voulez faire tourner **Docker** dedans (recommandation officielle Proxmox)

Sur ce dernier point, Proxmox est clair dans sa documentation : Docker doit tourner dans une VM, pas dans un LXC. C'est techniquement possible en LXC avec le nesting, mais c'est un nid à problèmes en production.

Pour une comparaison détaillée entre Proxmox et les autres hyperviseurs, notre article [Proxmox vs VMware](https://www.shpv.fr/blog/proxmox-vs-vmware/) creuse le sujet.

#### Sécurité : privileged vs unprivileged

C'est LE point critique quand on parle de LXC en production. Proxmox propose deux modes :

##### Conteneurs non privilégiés (par défaut)

C'est le mode recommandé. Le root du conteneur (UID 0) est mappé sur un UID élevé côté hôte (typiquement 100 000+). Même si un attaquant obtient root dans le conteneur, il n'a aucun privilège sur l'hôte.

En plus du mapping UID/GID, Proxmox empile les couches de protection :

- **AppArmor** : profils de confinement par conteneur
- **Seccomp** : filtrage des appels système dangereux
- **Namespaces Linux** : isolation PID, réseau, montages, IPC

##### Conteneurs privilégiés

Le root du conteneur correspond au root de l'hôte. C'est plus simple pour certains cas (NFS, montages spéciaux), mais la surface d'attaque est nettement plus étendue.

La règle en production : **unprivileged par défaut, privileged uniquement si vous savez exactement pourquoi**, et idéalement sur un réseau isolé.

Pour approfondir les mécanismes de cgroups et d'isolation qui sous-tendent LXC, notre article sur les [cgroups et systemd pour la QoS](https://www.shpv.fr/blog/cgroups-systemd-qos/) détaille le fonctionnement interne.

#### Déployer des services en LXC : cas concrets

Passons à la pratique. Voici trois cas d'usage classiques qui tirent parfaitement parti de LXC.

##### Nginx en reverse proxy

Un reverse proxy Nginx en LXC, c'est le cas d'usage parfait. Le service est léger, stateless (ou presque), et n'a besoin que de réseau et de CPU.

**Configuration recommandée :**

```
Cores : 2
RAM : 512 Mo
Disque : 4 Go
Réseau : bridge vmbr0, IP statique
```

L'avantage par rapport à une VM : vous économisez 1,5 Go de RAM et le démarrage est quasi instantané. Sur un cluster Proxmox avec 10 services derrière un reverse proxy, ça fait une vraie différence.

**Points d'attention :**

- Activez le `nesting` si vous utilisez certains modules Nginx (pas toujours nécessaire)
- Montez les certificats TLS via un bind mount depuis l'hôte ou utilisez un volume partagé
- Pensez à limiter les IOPS disque via les cgroups Proxmox pour éviter qu'un pic de logs ne sature le stockage

##### PostgreSQL

Spoiler alert : PostgreSQL en LXC, ça tourne comme sur des roulettes. Les benchmarks montrent un overhead d'environ 2 % par rapport au bare metal, ce qui est négligeable pour la plupart des workloads.

**Configuration recommandée :**

```
Cores : 4
RAM : 4 Go (ajustable selon shared_buffers)
Disque : 32 Go+ sur stockage rapide (SSD/NVMe)
Réseau : bridge dédié ou VLAN
```

**Bonnes pratiques :**

- Utilisez un stockage **ZFS** ou **LVM-thin** pour les snapshots natifs Proxmox
- Configurez `shared_buffers` à 25 % de la RAM allouée au conteneur
- Planifiez vos sauvegardes avec `pg_dump` ET les snapshots Proxmox (ceinture et bretelles)
- Isolez le trafic base de données sur un VLAN dédié

Pour la sauvegarde de vos bases PostgreSQL, notre guide sur [BorgBackup et PostgreSQL](https://www.shpv.fr/blog/borgbackup-postgresql/) couvre les stratégies avancées.

##### Pi-hole (DNS + filtrage)

Pi-hole en LXC, c'est le classique du homelab qui fonctionne aussi très bien en entreprise pour le filtrage DNS interne.

**Configuration recommandée :**

```
Cores : 1
RAM : 256 Mo
Disque : 2 Go
Réseau : IP statique obligatoire (c''est un serveur DNS)
```

Un Pi-hole en VM consommerait 8 à 10 fois plus de ressources pour le même résultat. En LXC, vous pouvez même en déployer deux (primaire + secondaire) pour la redondance, sans impact notable sur l'hyperviseur.

#### Réseau et stockage : les fondamentaux

##### Réseau

Chaque conteneur LXC dans Proxmox obtient une interface réseau virtuelle connectée à un bridge (typiquement `vmbr0`). Les bonnes pratiques réseau :

- **VLAN** : isolez vos conteneurs par fonction (front, back, management)
- **MTU** : assurez la cohérence des MTU sur toute la chaîne (hôte, bridge, conteneur, switch)
- **Firewall** : activez le firewall Proxmox au niveau du conteneur, pas seulement au niveau du datacenter

##### Stockage

Le choix du stockage backend impacte directement les performances LXC :

- **ZFS** : snapshots atomiques, compression, déduplication. Idéal pour la production.
- **LVM-thin** : snapshots possibles, bonne performance, moins gourmand en RAM que ZFS
- **Directory** : simple mais pas de snapshots natifs. À réserver au dev/test.

Pour les sauvegardes de vos conteneurs LXC, Proxmox Backup Server s'intègre nativement. On en parle en détail dans notre article sur [Proxmox Backup Server](https://www.shpv.fr/blog/proxmox-backup-server/).

#### Templates et automatisation

Proxmox fournit des templates LXC prêts à l'emploi (Debian, Ubuntu, Alpine, CentOS, Fedora) téléchargeables depuis l'interface web. Mais en production, vous voudrez probablement créer vos propres templates.

##### Créer un template personnalisé

```bash
# Créer un conteneur de base
pct create 9000 local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \
  --hostname template-base \
  --memory 512 \
  --cores 2 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --storage local-lvm

# Démarrer et configurer
pct start 9000
pct enter 9000

# (installer paquets, configurer SSH, hardening...)

# Convertir en template
pct stop 9000
pct template 9000
```

##### Automatisation avec l'API Proxmox

L'API REST de Proxmox permet de scripter la création de conteneurs LXC. Combiné avec Ansible ou Terraform, vous pouvez déployer des dizaines de conteneurs en quelques minutes.

```bash
# Cloner depuis un template via l'API
curl -k -X POST "https://pve:8006/api2/json/nodes/pve/lxc/9000/clone" \
  -H "Authorization: PVEAPIToken=user@pam!token=UUID" \
  -d "newid=200&hostname=nginx-prod&full=1"
```

Pour ceux qui s'intéressent aux alternatives de conteneurisation système, notre article sur [systemd-nspawn pour l'administration avancée](https://www.shpv.fr/blog/systemd-nspawn-advanced-admin/) explore une approche complémentaire.

#### Performance : les chiffres qui comptent

On ne va pas se mentir : les chiffres bruts parlent d'eux-mêmes.

|Métrique|LXC|VM KVM|Ratio|
|---|---|---|---|
|Temps de boot|2 à 5 s|30 à 60 s|10x plus rapide|
|RAM idle|100 à 500 Mo|2 à 4 Go|5 à 8x moins|
|Overhead CPU|1 à 3 %|5 à 10 %|3x moins|
|Densité par hôte|50 à 100+|10 à 20|5x plus|
|I/O disque|Quasi natif|Via virtio|LXC avantage|

Sur un serveur avec 128 Go de RAM, vous pouvez faire tourner confortablement 50 à 80 conteneurs LXC de services d'infrastructure. En VMs, vous plafonnez à 15 ou 20 pour les mêmes services.

Chez SHPV, cette densité est un levier concret pour nos clients en [infogérance](https://www.shpv.fr/services/infogerance/) : plus de services par serveur physique, c'est moins de matériel, moins de consommation électrique, et un meilleur PUE dans nos datacenters partenaires.

#### Les pièges à éviter

Parce qu'un bon article, c'est aussi vous éviter de perdre un vendredi soir en debug :

1. **Docker dans LXC** : techniquement possible avec le nesting activé, mais source de bugs subtils en production. Utilisez une VM dédiée pour Docker.
    
2. **Kernel modules** : un LXC partage le kernel de l'hôte. Si votre service a besoin d'un module kernel non chargé sur l'hôte, ça ne fonctionnera pas.
    
3. **NFS dans un unprivileged** : le mapping UID rend les montages NFS problématiques. Soit vous passez en privileged (risque sécurité), soit vous utilisez un montage bind depuis l'hôte.
    
4. **Ressources non limitées** : par défaut, un LXC peut consommer toute la RAM de l'hôte. Configurez TOUJOURS les limites mémoire et swap.
    
5. **Snapshots sans ZFS/LVM** : sur du stockage directory, pas de snapshot natif. Vos sauvegardes passent par un dump complet, ce qui est plus lent et plus lourd.
    

#### Le mot de la fin

LXC dans Proxmox, c'est le couteau suisse de l'admin système qui veut maximiser ses ressources sans sacrifier l'isolation. Pour les services d'infrastructure Linux (DNS, reverse proxy, bases de données, monitoring, VPN), c'est souvent le meilleur choix.

La règle d'or : 80 % de LXC pour l'efficacité, 20 % de VMs pour les cas spécifiques. Et si vous avez besoin d'un coup de main pour architecturer votre infrastructure Proxmox, les équipes SHPV sont là pour ça, que ce soit en [hébergement dédié](https://www.shpv.fr/services/hebergement/) ou en [infogérance](https://www.shpv.fr/services/infogerance/).

Spoiler alert pour le prochain épisode : on parlera de [Firecracker et les micro-VMs](https://www.shpv.fr/blog/firecracker-microvms/), le meilleur des deux mondes entre conteneurs et VMs. Stay tuned.

#### Sources

- [Linux Container, documentation officielle Proxmox VE](https://pve.proxmox.com/wiki/Linux_Container)
- [VM vs LXC in Proxmox: Performance, Use Cases and Setup Guide](https://www.propelrc.com/vm-vs-lxc-in-proxmox/)
- [Proxmox VE: Performance of KVM vs LXC, IKUS Software](https://ikus-soft.com/en_CA/blog/techies-10/proxmox-ve-performance-of-kvm-vs-lxc-75)
- [Optimizing LXC Container Performance in Proxmox VE 9, Dataplugs](https://www.dataplugs.com/en/optimizing-lxc-container-performance-in-proxmox-ve-9/)