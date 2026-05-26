---
tags:
  - Proxmox
  - Cloud-Init
---
Provisionner un serveur manuellement, c'est accepter que chaque machine soit un flocon de neige : unique, fragile, et impossible à reproduire à l'identique. Cloud-init résout ce problème structurel en transformant la configuration d'un serveur en un processus déclaratif, reproductible et entièrement automatisé.

Concrètement, cela signifie que chaque VM ou instance cloud démarre avec exactement la même base, les mêmes paquets, les mêmes utilisateurs et les mêmes règles de sécurité. Pas d'oubli, pas de dérive de configuration. Pour bien comprendre ce mécanisme, commençons par les fondamentaux.

#### Qu'est-ce que cloud-init ?

Cloud-init est un outil open source développé par Canonical, devenu le standard de facto pour l'initialisation d'instances cloud. Il est supporté nativement par tous les grands fournisseurs cloud (AWS, Azure, GCP, OVH, Scaleway) ainsi que par les hyperviseurs privés comme [Proxmox VE](https://www.shpv.fr/blog/proxmox-vs-vmware/).

Trois éléments sont à retenir :

1. **Il s'exécute au premier démarrage** : cloud-init intervient avant même que vous ne vous connectiez à la machine
2. **Il est déclaratif** : vous décrivez l'état souhaité, cloud-init se charge de l'atteindre
3. **Il est idempotent** : relancer cloud-init sur une machine déjà configurée ne casse rien

#### Les phases de démarrage de cloud-init

Pour bien comprendre ce mécanisme, il faut savoir que cloud-init s'exécute en cinq phases distinctes lors du boot :

##### 1. Generator

Cloud-init détermine s'il doit s'exécuter ou non. Il vérifie la présence d'un fichier de désactivation et l'existence d'une datasource valide.

##### 2. Local (cloud-init-local)

Cette phase s'exécute avant le réseau. Elle récupère la configuration réseau depuis la datasource locale (par exemple, un lecteur CD-ROM virtuel sur Proxmox) et l'applique. C'est ce qui permet à la machine d'obtenir une IP statique dès le premier boot.

##### 3. Network (cloud-init)

Une fois le réseau disponible, cloud-init récupère les métadonnées complètes depuis la datasource distante et exécute les modules de configuration initiaux : création d'utilisateurs, déploiement de clés SSH, configuration du hostname.

##### 4. Config (cloud-config)

Les modules de configuration avancés s'exécutent : installation de paquets, écriture de fichiers, montage de volumes, configuration NTP.

##### 5. Final (cloud-final)

La dernière phase exécute les scripts utilisateur, les commandes `runcmd` et les tâches qui nécessitent que le système soit pleinement opérationnel.

#### La datasource : d'où viennent les données ?

La datasource est la source d'information que cloud-init interroge pour récupérer la configuration. Selon votre environnement, elle prend différentes formes :

|Environnement|Datasource|Mécanisme|
|---|---|---|
|AWS|`Ec2`|Service de métadonnées HTTP (169.254.169.254)|
|Azure|`Azure`|Endpoint IMDS + WALinuxAgent|
|GCP|`GCE`|Serveur de métadonnées HTTP|
|Proxmox VE|`NoCloud`|Lecteur CD-ROM virtuel (ISO)|
|OpenStack|`OpenStack`|Service de métadonnées ou config drive|

Pour un environnement Proxmox, la datasource NoCloud est la plus courante. Elle utilise un fichier ISO monté comme lecteur CD-ROM contenant les fichiers `user-data`, `meta-data` et éventuellement `network-config`.

#### Anatomie d'un fichier user-data

Le fichier `user-data` est le cœur de la configuration cloud-init. Il utilise le format YAML avec le header `#cloud-config`. Voici un exemple complet et commenté :

```yaml
#cloud-config

# Configuration du hostname
hostname: web-prod-01
fqdn: web-prod-01.infra.local
manage_etc_hosts: true

# Création d'utilisateurs
users:
  - name: deploy
    groups: sudo, docker
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... deploy@infra

# Mise à jour et installation de paquets
package_update: true
package_upgrade: true
packages:
  - curl
  - wget
  - htop
  - git
  - fail2ban
  - unattended-upgrades

# Écriture de fichiers de configuration
write_files:
  - path: /etc/ssh/sshd_config.d/hardening.conf
    content: |
      PermitRootLogin no
      PasswordAuthentication no
      MaxAuthTries 3
      ClientAliveInterval 300
      ClientAliveCountMax 2
    permissions: '0644'

  - path: /etc/sysctl.d/99-hardening.conf
    content: |
      net.ipv4.ip_forward = 0
      net.ipv4.conf.all.rp_filter = 1
      net.ipv4.conf.default.rp_filter = 1
    permissions: '0644'

# Commandes exécutées en fin de provisioning
runcmd:
  - systemctl enable fail2ban
  - systemctl start fail2ban
  - sysctl --system
  - systemctl restart sshd
```

Concrètement, cela signifie qu'en une seule déclaration YAML, vous obtenez un serveur avec un utilisateur de déploiement, des paquets essentiels installés, SSH durci et fail2ban actif. Le tout sans toucher un clavier.

#### Configuration réseau avec network-config

Le fichier `network-config` permet de définir une configuration réseau statique au format Netplan (version 2). C'est particulièrement utile pour les serveurs de production où le DHCP n'est pas souhaitable :

```yaml
version: 2
ethernets:
  eth0:
    addresses:
      - 10.120.20.50/24
    routes:
      - to: default
        via: 10.120.20.1
    nameservers:
      addresses:
        - 10.120.20.1
        - 1.1.1.1
      search:
        - infra.local
```

#### Cloud-init sur Proxmox VE : mise en pratique

Proxmox VE intègre nativement le support de cloud-init, ce qui en fait un outil redoutable pour automatiser le provisioning de VMs dans un environnement on-premise. La méthode recommandée repose sur la création d'un template.

##### Étape 1 : créer un template cloud-init

```bash
# Télécharger une image cloud Ubuntu
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img

# Créer la VM (ID 9000 = template)
qm create 9000 --name ubuntu-24-template --memory 2048 --cores 2 \
  --net0 virtio,bridge=vmbr0

# Importer le disque
qm importdisk 9000 noble-server-cloudimg-amd64.img local-lvm

# Attacher le disque
qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-9000-disk-0

# Ajouter le lecteur cloud-init
qm set 9000 --ide2 local-lvm:cloudinit

# Configurer le boot
qm set 9000 --boot order=scsi0

# Activer le serial console et l'agent QEMU
qm set 9000 --serial0 socket --vga serial0
qm set 9000 --agent enabled=1

# Convertir en template
qm template 9000
```

##### Étape 2 : configurer cloud-init via Proxmox

Proxmox permet de passer les paramètres cloud-init directement via l'interface web ou la CLI :

```bash
# Configuration de base
qm set 9000 --ciuser deploy
qm set 9000 --sshkeys ~/.ssh/authorized_keys
qm set 9000 --ipconfig0 ip=10.120.20.50/24,gw=10.120.20.1
qm set 9000 --nameserver 10.120.20.1
qm set 9000 --searchdomain infra.local
```

##### Étape 3 : cloner et démarrer

```bash
# Cloner le template
qm clone 9000 110 --name web-prod-01 --full

# Personnaliser l'IP
qm set 110 --ipconfig0 ip=10.120.20.51/24,gw=10.120.20.1

# Démarrer
qm start 110
```

En quelques secondes, vous avez un serveur prêt à l'emploi avec la bonne IP, le bon utilisateur et les bonnes clés SSH. Chez SHPV, c'est exactement cette approche que nous utilisons pour provisionner les serveurs clients sur notre infrastructure Proxmox.

#### Intégration avec Terraform et Ansible

Cloud-init prend toute sa puissance quand il s'intègre dans une chaîne d'automatisation complète. Le pattern classique est le suivant :

1. **[Terraform](https://www.shpv.fr/blog/terraform-iac/)** crée la VM et injecte la configuration cloud-init
2. **Cloud-init** effectue le provisioning initial (utilisateurs, réseau, paquets de base)
3. **[Ansible](https://www.shpv.fr/blog/automatiser-configuration-ansible/)** prend le relais pour la configuration applicative

##### Exemple Terraform avec le provider Proxmox

```hcl
resource "proxmox_vm_qemu" "web_server" {
  name        = "web-prod-01"
  target_node = "pve-node-01"
  clone       = "ubuntu-24-template"
  full_clone  = true

  cores   = 4
  memory  = 8192

  disk {
    storage = "local-lvm"
    size    = "50G"
    type    = "scsi"
  }

  network {
    model  = "virtio"
    bridge = "vmbr0"
  }

  # Configuration cloud-init
  os_type    = "cloud-init"
  ciuser     = "deploy"
  sshkeys    = file("~/.ssh/id_ed25519.pub")
  ipconfig0  = "ip=10.120.20.51/24,gw=10.120.20.1"
  nameserver = "10.120.20.1"
}
```

Cette approche s'intègre parfaitement dans un workflow [Infrastructure as Code complet](https://www.shpv.fr/blog/terraform-ansible-cloud/) où Terraform gère le cycle de vie des VMs et Ansible s'occupe de la configuration post-provisioning.

#### Snippets personnalisés avec cloud-init

Pour les cas plus avancés, cloud-init supporte l'injection de snippets personnalisés. Sur Proxmox, cela permet de passer un fichier `user-data` complet au lieu des paramètres basiques de l'interface :

```bash
# Stocker le snippet sur le stockage Proxmox
cat > /var/lib/vz/snippets/web-server.yml << 'USERDATA'
#cloud-config
package_update: true
packages:
  - nginx
  - certbot
  - python3-certbot-nginx

write_files:
  - path: /etc/nginx/sites-available/default
    content: |
      server {
          listen 80;
          server_name _;
          root /var/www/html;
          index index.html;
      }

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
  - ufw allow 'Nginx Full'
  - ufw --force enable
USERDATA

# Associer le snippet à la VM
qm set 110 --cicustom "user=local:snippets/web-server.yml"
```

#### Bonnes pratiques en production

Trois éléments sont à retenir pour un usage de cloud-init en production :

##### 1. Séparer la configuration de base et la configuration applicative

Cloud-init doit se limiter au socle : utilisateurs, réseau, paquets système, durcissement SSH. La configuration applicative (Nginx, Docker, bases de données) relève d'[Ansible](https://www.shpv.fr/blog/ansible-roles-galaxy/) ou d'un autre outil de gestion de configuration.

##### 2. Versionner les templates et les snippets

Vos fichiers `user-data` et vos templates Proxmox doivent être versionnés dans Git. Un template non versionné est un template qui dérivera silencieusement.

##### 3. Automatiser les mises à jour de sécurité

Combinez cloud-init avec le paquet [unattended-upgrades](https://www.shpv.fr/blog/unattended-upgrades/) pour que chaque nouveau serveur applique automatiquement les correctifs de sécurité dès son premier démarrage :

```yaml
#cloud-config
packages:
  - unattended-upgrades

write_files:
  - path: /etc/apt/apt.conf.d/20auto-upgrades
    content: |
      APT::Periodic::Update-Package-Lists "1";
      APT::Periodic::Unattended-Upgrade "1";
      APT::Periodic::AutocleanInterval "7";

runcmd:
  - dpkg-reconfigure -plow unattended-upgrades
```

#### Dépannage : les erreurs courantes

Quand cloud-init ne fonctionne pas comme prévu, voici les commandes de diagnostic essentielles :

```bash
# Vérifier le statut global
cloud-init status --long

# Consulter les logs détaillés
cat /var/log/cloud-init-output.log

# Voir les données d''instance
cloud-init query userdata

# Réinitialiser cloud-init (utile pour les tests)
cloud-init clean --logs
```

Les erreurs les plus fréquentes sont :

- **YAML invalide** : une indentation incorrecte dans le fichier `user-data` fait échouer silencieusement la configuration. Validez toujours avec `cloud-init schema --config-file user-data.yml`
- **Datasource introuvable** : sur Proxmox, vérifiez que le lecteur cloud-init (ide2) est bien attaché à la VM
- **Réseau non configuré** : si la VM n'obtient pas d'IP, vérifiez que le fichier `network-config` est bien présent dans l'ISO NoCloud
- **Timeout au boot** : cloud-init attend la datasource pendant 120 secondes par défaut. Si la datasource est absente, le boot sera lent

#### Cloud-init vs alternatives

|Critère|Cloud-init|Ignition (CoreOS)|Kickstart/Preseed|
|---|---|---|---|
|Écosystème|Universel|Fedora CoreOS, Flatcar|RHEL, Debian|
|Format|YAML|JSON|Propriétaire|
|Réseau|Oui|Oui|Limité|
|Post-install|Oui (runcmd)|Limité|Oui|
|Cloud support|Natif|Limité|Non|
|Proxmox|Natif|Manuel|Non|

Pour un environnement hétérogène mêlant cloud public et infrastructure privée, cloud-init reste le choix le plus polyvalent. C'est d'ailleurs pourquoi SHPV l'utilise systématiquement pour le provisioning de VMs clients, que ce soit sur Proxmox ou sur les offres [cloud managé](https://www.shpv.fr/services/cloud/).

#### Pour aller plus loin

Cloud-init est la première brique d'une stratégie d'automatisation complète. Une fois maîtrisé, il s'intègre naturellement dans un pipeline plus large : [Terraform](https://www.shpv.fr/blog/terraform-iac/) pour l'orchestration, [Ansible](https://www.shpv.fr/blog/automatiser-configuration-ansible/) pour la configuration, et [Proxmox Backup Server](https://www.shpv.fr/blog/proxmox-backup-server/) pour la sauvegarde.

L'objectif final est simple : qu'un serveur puisse être détruit et recréé à l'identique en quelques minutes, sans intervention humaine. C'est la définition même de l'infrastructure immuable, et cloud-init en est la pierre angulaire.

#### Sources

- [Documentation officielle cloud-init 26.1](https://docs.cloud-init.io/en/latest/)
- [Red Hat : Introduction to cloud-init (RHEL 9)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_and_managing_cloud-init_for_rhel_9/introduction-to-cloud-init_cloud-content)
- [Proxmox Wiki : Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support)