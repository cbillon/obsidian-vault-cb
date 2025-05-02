---
link: https://colinfo.fr/comment-deployer-des-machines-virtuelles-dans-proxmox-avec-terraform/
byline: Thomas COLIN
date: 2024-03-11T16:33
excerpt: Découvrez comment déployer efficacement des machines virtuelles dans Proxmox à l'aide de Terraform, un outil open-source pour la gestion de l'infrastructure en tant que code (IaC). Suivez les étapes détaillées d'installation de Terraform, de configuration de la connexion à Proxmox, et de déploiement des machines virtuelles à partir d'un modèle personnalisé. Profitez d'une approche reproductible et simplifiée pour gérer vos environnements de cloud computing avec Terraform.
slurped: 2025-02-28T18:32
title: Comment déployer des machines virtuelles dans Proxmox avec Terraform -
tags:
  - Proxmox
  - template
  - VM
  - Cloud-Init
  - Terraform
---

- [Introduction](https://colinfo.fr/comment-deployer-des-machines-virtuelles-dans-proxmox-avec-terraform/#Introduction "Introduction")
- [Installation](https://colinfo.fr/comment-deployer-des-machines-virtuelles-dans-proxmox-avec-terraform/#Installation "Installation")
- [Configuration de la connexion à proxmox](https://colinfo.fr/comment-deployer-des-machines-virtuelles-dans-proxmox-avec-terraform/#Configuration_de_la_connexion_a_proxmox "Configuration de la connexion à proxmox")
- [Création d’un utilisateur Terraform](https://colinfo.fr/comment-deployer-des-machines-virtuelles-dans-proxmox-avec-terraform/#Creation_dun_utilisateur_Terraform "Création d’un utilisateur Terraform")
- [Configuration SSH](https://colinfo.fr/comment-deployer-des-machines-virtuelles-dans-proxmox-avec-terraform/#Configuration_SSH "Configuration SSH")
- [Deploiement avec Terraform](https://colinfo.fr/comment-deployer-des-machines-virtuelles-dans-proxmox-avec-terraform/#Deploiement_avec_Terraform "Deploiement avec Terraform")
- [Conclusion](https://colinfo.fr/comment-deployer-des-machines-virtuelles-dans-proxmox-avec-terraform/#Conclusion "Conclusion")

## Introduction

Dans les précédents articles, nous avons exploré la mise en place d’un [serveur Proxmox](https://colinfo.fr/installation-et-configuration-de-proxmox/) en configurant à la fois le réseau et le stockage. De plus, nous avons examiné le déploiement d’un [pare-feu pfSense](https://colinfo.fr/installation-et-configuration-de-pfsense/) ainsi que la création d’un [template](https://colinfo.fr/creation-dun-template-de-debian-12-sur-proxmox/) pour Debian 12 avec cloud-init.

  
Dans cet article, nous allons utiliser Terraform pour déployer nos machines virtuelles à partir de ce template.

Terraform est un outil open-source développé par HashiCorp qui permet la gestion de l’infrastructure en tant que code (IaC). Il permet de définir et de provisionner l’infrastructure cloud de manière programmable à l’aide de fichiers de configuration. Cette solution facilite la création, la modification et la gestion des ressources d’infrastructure sur divers fournisseurs cloud tels que AWS, Azure, Google Cloud, vSphere, ou encore Proxmox. Elle permet donc de provisionner efficacement et automatiquement des environnements de cloud computing.

Nous disposons donc d’un template prêt au déploiement et nous allons déployer toute les machines virtuelles Debian à partir de ce Template.

## Installation

Terraform peut être installé sur plusieurs systèmes, comme Windows et Linux.

Nous allons commencer par l’installation des pré-requis et la mise à jour du système.

```
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```

Installation de la clé GPG de hashicorp

```
wget -O- https://apt.releases.hashicorp.com/gpg | \

gpg --dearmor | \

sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
```

Verification de l’empreinte de la clé

```
gpg --no-default-keyring \ 
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \ 
--fingerprint
```

Ensuite, on va ajouter les dépôts officiel de Hashicorp.

```
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \

https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \

sudo tee /etc/apt/sources.list.d/hashicorp.list
```

Et on télécharge puis installe Terraform depuis le dépôt que l’on vient d’ajouter.

```
sudo apt update

sudo apt-get install terraform
```

Vous pouvez vérifier la version de terraform installé avec la commande terraform -v

![Une image contenant Police, texte, capture d’écran
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-police-texte-capture-decran.png)

## 

Configuration de la connexion à proxmox  

### Création d’un utilisateur Terraform  

Ensuite, nous allons créer puis allouer des droits à notre utilisateur terraform afin qu’il possède des droits pour gérer le serveur !

Nous allons commencer par ajouter un rôle en ligne de commande :

```
pveum role add TerraformProv -privs "Datastore.AllocateSpace Datastore.Audit Pool.Allocate Sys.Audit Sys.Console Sys.Modify VM.Allocate VM.Audit VM.Clone VM.Config.CDROM VM.Config.Cloudinit VM.Config.CPU VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Migrate VM.Monitor VM.PowerMgmt SDN.Use"
```

Ensuite on crée un utilisateur. Saisissez le password que vous désirez.

```
pveum user add terraform-prov@pve --password <password>
```

Et on lie cette utilisateur au rôle.

```
pveum aclmod / -user terraform-prov@pve -role TerraformProv
```

Ensuite, nous allons créer un token. Ce jeton permettra à l’utilisateur Terraform de se connecter à Proxmox plutôt que par mot de passe !

```
pveum user token add terraform-prov@pve terraform -expire 0 -privsep 0 -comment "Terraform token"
```

Un résultat apparait, veuillez bien noter la valeur de la clé de votre token, vous ne pourrez plus la revoir !

![Une image contenant texte, capture d’écran, Police, ligne
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-42.png)

Il est possible de tester la connexion avec le token avec l’utilisateur « curl » travers cette commande :

```
curl -X GET 'https://192.168.1.200:8006/api2/json' -H 'Authorization: PVEAPIToken=terraform-prov@pve!terraform=YOUR_TOKEN_ID' --insecure
```

![](https://colinfo.fr/wp-content/uploads/2024/03/word-image-29606-3.png)

### Configuration SSH

Sur l’hôte où Terraform s’exécutera, nous pouvons générer une clé publique SSH en utilisant la commande suivante :

```
ssh-keygen -b 4096
```

Si vous le souhaitez, vous pouvez choisir une passphrase pour renforcer la sécurité de votre clé publique. Cela fournira une protection supplémentaire au cas où quelqu’un récupérerait votre clé publique.

La clé générée sera stockée dans le fichier id_rsa.pub situé dans le répertoire /home/colin/.ssh.

Cette clé sera nécessaire pour vous connecter aux serveurs déployés, car l’authentification SSH par mot de passe de l’image cloud-init est désactivée par défaut !

  
Vous pouvez consulter la documentation du provider terraform ici : https://registry.terraform.io/providers/Telmate/proxmox/latest/docs

## Deploiement avec Terraform

Terraform utilise des fichiers de configuration. Nous aurons 4 fichiers qui seront utilisés :

![Une image contenant texte, Police, capture d’écran, ligne
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-police-capture-decran-3.png)

main.tf : Ce fichier spécifiera comment les ressources doivent être créées, configurées et gérées dans Terraform

providers.tf : Utilisé pour définir le provider (Proxmox) à travers l’API, le token id et le secret du token

variables.tf : Définir les variables utilisées dans le fichier main.tf

variables.tfvars : Contiendra les valeurs réelles des variables définies dans variables.tf.

Voici le contenu des fichiers :

main.tf

—

```
resource "proxmox_vm_qemu" "proxmox" {

count = var.instance_count

name = element(var.name,count.index)

target_node = var.target_node

clone = var.clone

#cpu

cores = 1

sockets = 1

cpu = "host"

#memory

memory = "2048"

#network

network {

bridge = element(var.network_bridge,count.index)

model = "virtio"

}

ipconfig0 = element(var.ip,count.index)

nameserver = var.server_dns

#disk

scsihw = "virtio-scsi-pci"

cloudinit_cdrom_storage = var.storage

disks {

virtio {

virtio0 {

disk {

size = var.size

storage = var.storage

iothread = true

}

}

}

}

#cloud init config

os_type = "cloud-init"

ciuser = var.ciuser

cipassword = var.cipwd

sshkeys = <<EOF

${var.ssh_key}

EOF

}
```

—

providers.tf

—

```
terraform {

required_providers {

proxmox = {

source = "Telmate/proxmox"

version = "3.0.1-rc1"

}

}

}

provider "proxmox" {

pm_api_url = var.pm_api_url

pm_api_token_id = var.pm_api_token_id

pm_api_token_secret = var.pm_api_token_secret

pm_tls_insecure = true

# pm_log_enable = true

# pm_log_file = "terraform-plugin-prox-vm.log"

# pm_log_levels = {

# _default = "debug"

# _capturelog = ""

# }

}
```

—

Varibles.tf

—

```
variable "pm_api_url" {

type = string

}

variable "pm_api_token_id" {

type = string

}

variable "pm_api_token_secret" {

type = string

sensitive = true

}

variable "clone" {

type = string

}

variable "name" {

type = list(string)

}

variable target_node {

type = string

}

variable instance_count {

type = number

}

variable network_bridge {

type = list(string)

}

variable ip {

type = list(string)

}

variable server_dns {

type = string

}

variable size {

type = string

}

variable storage {

type = string

}

variable ciuser {

type = string

}

variable cipwd {

type = string

}

variable ssh_key {

type = string

}
```

—

Variables.tfvars

—

```
pm_api_url = "https://192.168.1.200:8006/api2/json"

pm_api_token_id = "terraform-prov@pve!terraform"

pm_api_token_secret = "8fed….. "

instance_count = 3

name = ["docker-sec","docker-app","docker-rp"]

clone = "template-debian"

target_node = "pve"

#vmbr0 = LAN / vmbr1 = VM_Network / vmbr2 = DMZ

network_bridge = ["vmbr1","vmbr1","vmbr2"]

ip = ["ip=192.168.100.203/24,gw=192.168.100.254","ip=192.168.100.204/24,gw=192.168.100.254","ip=10.0.0.10/24,gw=10.0.0.254"]

server_dns = "192.168.1.1"

size = "20"

storage = "vg_proxmox"

ciuser = "your_user"

cipwd = "your_password"

ssh_key = "ssh-rsa AAA…
```

—

Ce que je trouve particulièrement avantageux avec Terraform, c’est la possibilité de personnaliser la configuration des clients selon nos besoins. Il suffit simplement de remplacer les valeurs des lignes indiquées ci-dessous. Il y a plusieurs façons de faire mais c’est celle-ci que j’ai retenu.

Par exemple, si je souhaite avoir 4 hôtes, il me suffit de modifier la valeur de « instance_count » à 4 et d’ajouter les éléments supplémentaires aux autres variables telles que « network_bridge », « ip » et « name » en fonction de l’ordre que je souhaite !

Exemple avec 4 machines :

—

---

```
instance_count = 4

network_bridge = ["vmbr0","vmbr1","vmbr2","vmbr1"]

ip = ["ip=192.168.100.203/24,gw=192.168.100.254","ip=192.168.100.204/24,gw=192.168.100.254","ip=10.0.0.10/24,gw=10.0.0.254","ip=192.168.100.240/24,gw=192.168.100.254"]

name = ["docker-sec","docker-app","docker-rp","docker-new"]
```

—

Pour exécuter Terraform, il y a quelques commandes à lancer :

Il faut d’abord initialiser le provider :

```
terraform init
```

Ensuite, on peut valider qu’il n’y ait pas d’erreurs et visualiser les changements que Terraform va apporter à notre infrastrure. L’option var-file variables.tfvars permet de spécifier un fichier contenu des variables.

```
terraform plan -var-file variables.tfvars
```

Et enfin, la commande pour lancer l’approvisionnement :

```
terraform apply -var-file variables.tfvars
```

Un message indiquant un succès va apparaitre si tous ce passe bien.

On peut également détruire ce que l’on vient de créer :

```
terraform destroy -var-file variables.tfvars
```

On voit bien sur notre hôte proxmox que Terraform à bien déployé mes 3 machines virtuelles :

![](https://colinfo.fr/wp-content/uploads/2024/03/image-1.png)

## Conclusion

En conclusion, nous avons vu comment déployer des machines virtuelles dans Proxmox avec Terraform à partir d’un modèle avec une configuration personnalisé. Cette solution offre une approche efficace et reproductible pour la gestion de l’infrastructure en tant que code. L’utilisation de Terraform permet de définir et provisionner les ressources d’infrastructure de manière programmable, ce qui simplifie la création, la modification et la gestion des environnements de cloud computing.

Pour aller plus loin : la documentation du provider proxmox de terraform : https://registry.terraform.io/providers/Telmate/proxmox/latest/docs