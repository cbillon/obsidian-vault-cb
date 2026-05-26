---
tags:
  - Proxmox
  - backup-server
---
#### Introduction

La sauvegarde est le pilier de toute infrastructure résiliente. Pour les environnements **Proxmox VE**, une solution native s'impose : **Proxmox Backup Server (PBS)**. Cet outil open-source offre une approche moderne aux sauvegardes (incrémentales, dédupliquées et chiffrées) sans surcoût de licence.

Que vous hébergiez quelques VMs ou gériez une flotte entière de conteneurs LXC, PBS propose une architecture qui passe à l'échelle, avec **déduplication à grain variable**, **chiffrement client AES-256-GCM** et **intégration transparente** avec votre hyperviseur Proxmox.

---

#### Plan

1. [Qu'est-ce que Proxmox Backup Server](https://www.shpv.fr/blog/proxmox-backup-server/#quest-ce-que-proxmox-backup-server)
2. [Installation de PBS](https://www.shpv.fr/blog/proxmox-backup-server/#installation-de-pbs)
3. [Configuration des datastores](https://www.shpv.fr/blog/proxmox-backup-server/#configuration-des-datastores)
4. [Intégration avec Proxmox VE](https://www.shpv.fr/blog/proxmox-backup-server/#int%C3%A9gration-avec-proxmox-ve)
5. [Planifier les sauvegardes](https://www.shpv.fr/blog/proxmox-backup-server/#planifier-les-sauvegardes)
6. [PBS vs Veeam vs Bacula](https://www.shpv.fr/blog/proxmox-backup-server/#pbs-vs-veeam-vs-bacula)
7. [Perspectives](https://www.shpv.fr/blog/proxmox-backup-server/#perspectives)

---

#### Qu'est-ce que Proxmox Backup Server

##### Architecture et principes

**Proxmox Backup Server** repose sur **trois piliers architecturaux** :

- **Déduplication à grain variable** : PBS découpe les données en chunks de taille variable (4 Ko–128 Ko) et ne stocke que les blocs uniques. Réduction typique : 50–70 % selon votre charge.
- **Chiffrement client** : AES-256-GCM, les données sont chiffrées côté client, jamais exposées en clair sur le réseau.
- **Incrémentalité** : après une base complète, seuls les blocs modifiés sont sauvegardés.

Le backend stockage utilise **ZFS**, offrant snapshots, compression et intégrité des données via checksums.

##### Supports sauvegardés

PBS supporte trois types de sources :

|Type|Source|Cas d'usage|
|---|---|---|
|**VM**|Proxmox VE (qemu)|Sauvegardes VMs Linux/Windows|
|**Container**|LXC|Sauvegardes conteneurs LXC|
|**Host**|Systèmes de fichiers|Sauvegardes fichiers, bases données|

##### API et interface

- **Web UI** : console d'administration ergonomique
- **REST API** : automatisation et intégration via scripts
- **Datastore** : unité logique de stockage, liée à un chemin ZFS/disque

---

#### Installation de PBS

##### Environnement prérequis

- Debian 11+ (ou compatible)
- 4 GB RAM minimum (8 GB recommandé)
- Stockage dédié (disque/partition externe)
- Accès réseau vers votre Proxmox VE

##### Étapes d'installation

**1. Ajouter le dépôt PBS**

```bash
echo "deb [arch=amd64] http://download.proxmox.com/debian/pbs bookworm pbs-release" \
  | sudo tee /etc/apt/sources.list.d/pbs-release.list

curl https://enterprise.proxmox.com/debian/proxmox-release-bookworm.gpg \
  | sudo apt-key add -

sudo apt update
```

**2. Installer PBS**

```bash
sudo apt install proxmox-backup-server
```

**3. Accéder à la Web UI**

```
https://<ip-pbs>:8007
```

Par défaut, login : `root@pam` avec le mot de passe root du système.

##### Configuration réseau initiale

Vérifiez que le firewall autorise le port **8007** (HTTPS) et **10050** (communication Proxmox VE).

```bash
sudo ufw allow 8007/tcp
sudo ufw allow 10050/tcp
```

---

#### Configuration des datastores

Un **datastore** est le conteneur logique où PBS stocke les sauvegardes. Il est lié à un chemin disque ZFS ou standard.

##### Créer un datastore

Via la Web UI : **Configuration** → **Datastores** → **Create**.

Ou en CLI :

```bash
proxmox-backup-manager datastore create <name> <path> \
  --comment "Description" \
  --chunks-per-file 32 \
  --disable-pitr false
```

Exemple :

```bash
proxmox-backup-manager datastore create backup-primary /mnt/pbs-storage \
  --comment "Stockage principal des VMs"
```

##### Tunage du datastore

- **chunks-per-file** : nombre de chunks par fichier (32–256 selon la RAM/charge)
- **PITR** (Point-in-Time Restore) : snapshot de chaque backup pour restauration granulaire
- **Maintenance** : garbage collection (GC) pour récupérer l'espace des old chunks

Lancer une GC manuelle :

```bash
proxmox-backup-manager datastore gc <datastore-name>
```

---

#### Intégration avec Proxmox VE

##### Ajouter PBS comme storage

Sur **Proxmox VE**, via **Datacenter** → **Storage** → **Add** → **Proxmox Backup Server**.

Configuration :

- **ID** : nom local du storage (ex. `pbs-primary`)
- **Server** : IP/hostname PBS
- **Username** : utilisateur PBS (ex. `root@pam`)
- **Password** : token d'authentification
- **Datastore** : nom du datastore sur PBS
- **Fingerprint** : empreinte SSL du serveur PBS

Une fois configuré, ce storage apparaît dans votre **Proxmox VE** et vous pouvez l'utiliser pour les sauvegardes.

##### Vérification

Depuis la CLI Proxmox :

```bash
pvesm list pbs-primary
```

Vous verrez la capacité et l'utilisation du datastore PBS intégré.

---

#### Planifier les sauvegardes

##### Créer un job de sauvegarde

Via **Proxmox VE** : sélectionnez une VM → **Backup** → **New**.

Paramètres clés :

- **Enabled** : activer le job
- **Storage** : datastore PBS
- **Schedule** : Cron (ex. `0 2 * * *` pour 2h du matin quotidiennement)
- **Retention** : garder N backups ou N jours (ex. `keep-last=7`)
- **Compression** : `zstd` (par défaut, bon ratio)
- **Verification** : **activer** pour valider chaque backup après création

##### Vérifier l'intégrité

PBS vérifie automatiquement les chunks dédupliqués. Vous pouvez aussi forcer une vérification manuelle :

```bash
proxmox-backup-manager verify <datastore> <snapshot-id>
```

##### Rétention et archivage

Définir une rétention multi-niveaux :

- **Quotidien** : conserver 7 jours
- **Hebdomadaire** : conserver 4 semaines
- **Mensuel** : conserver 12 mois

Exemple Cron :

```
# Sauvegarde quotidienne à 2h
0 2 * * * pbs-daily

# Sauvegarde hebdomadaire (dimanche) à 3h
0 3 * * 0 pbs-weekly
```

---

#### PBS vs Veeam vs Bacula

|Critère|PBS|Veeam|Bacula|
|---|---|---|---|
|**Licence**|Open-source (gratuit)|Propriétaire (payant)|Open-source (gratuit)|
|**Déduplication**|Variable-size chunking|Inline + post-process|Basique|
|**Chiffrement**|AES-256-GCM client|AES-256 (réseau)|Support optionnel|
|**Proxmox VE**|Intégration native|Plugin (limité)|Intégration faible|
|**Interface**|Web UI moderne|Veeam Backup & Replication|CLI/GUI archaïque|
|**Courbe d'apprentissage**|Faible|Moyenne|Élevée|
|**Enterprise support**|Payant (Proxmox)|Inclus|Communauté/payant|

**Verdict** : Pour un environnement **Proxmox VE pur**, PBS offre le meilleur ROI, intégration native, zéro licence, architecture moderne.

---

#### Perspectives

Pour approfondir la sauvegarde en infrastructure :

- [Velero et la sauvegarde Kubernetes](https://www.shpv.fr/blog/velero-k8s-backup/) : sauvegardez vos clusters K8s avec la même philosophie incrémentale
- [Stratégie 3-2-1 : la golden rule du backup](https://www.shpv.fr/blog/strategie-sauvegarde-321/) : mise en place opérationnelle
- [Proxmox VE vs VMware : architecture de sauvegarde](https://www.shpv.fr/blog/proxmox-vs-vmware/) : comparaison infrastructurelle

---

#### Sources

- [Proxmox Backup Server, Documentation officielle](https://pbs.proxmox.com/docs/)
- [Proxmox VE, PBS Integration](https://pve.proxmox.com/wiki/Backup_and_Restore)
- [ZFS, Advanced File System and Volume Manager](https://zfsonlinux.org/)
- [AES-256-GCM, NIST FIPS 140-2](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf)

---

#### Conclusion

**Proxmox Backup Server** incarne une philosophie de sauvegarde moderne : **efficacité, sécurité et transparence**. La déduplication réduit vos coûts de stockage, l'incrémentalité accélère les backups, et le chiffrement client garantit la confidentialité.

Chez **SHPV**, nous déployons PBS dans les infrastructures Proxmox pour assurer une **résilience de classe entreprise** sans surcoût licenciel. Combiné à une stratégie **3-2-1** rigoureuse et à des vérifications régulières, vous avez une solution de sauvegarde **éprouvée et auditable**.