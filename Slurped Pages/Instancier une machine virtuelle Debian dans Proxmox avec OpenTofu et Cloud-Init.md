---
link: https://xieme-art.org/post/instancier-une-machine-virtuelle-debian-dans-proxmox-avec-opentofu-et-cloud-init/
byline: ephase
site: Xieme-Art
date: 2024-11-07T23:04
excerpt: Installer une machine virtuelle dans Proxmox VE n’est pas une tâche très sympa. Mais heureusement il est possible de l’automatiser en...
twitter: https://twitter.com/@ephase
tags: []
slurped: 2024-12-19T07:47
title: Instancier une machine virtuelle Debian dans Proxmox avec OpenTofu et Cloud-Init
---
#Proxmox #OpenTofu #Cloud-Init

Installer une machine virtuelle dans [Proxmox VE](https://www.proxmox.com/) n’est pas une tâche très sympa. Mais heureusement il est possible de l’automatiser en utilisant [OpenTofu](https://opentofu.org/) et [Cloud-Init](https://cloud-init.io/). C’est ce que nous allons faire dans cet article en installant une machine Debian.

## Préparer OpenTofu

Je pars du principe que vous avez déjà une installation fonctionnelle de [Proxmox VE](https://www.proxmox.com/) et d’OpenTofu.

Pour permettre à OpenTofu de communiquer avec notre instance, nous utiliserons le provider écrit par _Pavel Boldyrev_ dont le code est disponible [sur GitHub](https://github.com/bpg/terraform-provider-proxmox)

Commençons par créer un dossier pour notre projet OpenTofu et à l’intérieur un fichier `provider.tf` avec le contenu suivant:

```
terraform {
    required_version = "> 1.6.0" 
    required_providers {
        proxmox = {
            source  = "bpg/proxmox"
            version = "0.64.0"
        }
    } 
}  

provider "proxmox" {
    endpoint    = var.pve_endpoint
    username    = var.pve_username
    password    = var.pve_password
    insecure    = var.pve_insecure 
}
```

Le premier bloc nous permet de déclarer les _providers_ utilisés, bon ici nous en avons un seul…

Le second est là pour configurer notre _provider Proxmox_ en lui donnant les informations utiles pour qu’il se connecte à l’API. Nous utilisons des variables que nous définirons… Maintenant!

## Définir nos variables

Maintenant que notre _provider_ est configuré, nous pouvons passer à la définition de nos variables dans le fichier `variables.tf`:

```
variable "pve_insecure" {
  type        = bool
  description = "Enable insecure connexion"
  default     = true
}  
variable "pve_endpoint" {
  type        = string
  description = "API endpoint URL"
}
variable "pve_password" {
  type        = string
  description = "Password"
}
variable "pve_username" {
  type        = string
  description = "Username"
}
variable "pve_node" {
  type        = string
  description = "Node where install elements"
  default     = ""
}
variable "debian_image_url" {
  type        = string
  description = "The URL for the latest Debian 12 Bookworm qcow2 image"            default     = ""
}
variable "debian_image_checksum_algorithm" {
  type        = string
  description = "Checksum algo used by image"
  default      = "sha512"
}
variable "debian_image_checksum" {
  type        = string
  description = "SHA Digest of the image"
  default     = ""
}
```

Afin de donner des valeurs à nos variables, ajoutons le fichier `terraform.tfvars` qui sera lu automatiquement par _OpenTofu_:

```
# PVE informations 
pve_endpoint = "https://192.168.0.200"
pve_username = "root" 
pve_password = "Mysup3rPassW0rd" 
pve_node     = "pve01" pve_insecure = true
# Debian images 
debian_image_url = "https://cloud.debian.org/images/cloud/bookworm/20241004-1890/debian-12-generic-amd64-20241004-1890.qcow2" 
debian_image_checksum = "da84d609d7ec5645dae1df503ea72037b2a831401d1b42ce2e7ec2a840b699f07ca8aea630853a3d5430839268c2bd337be45d89498264c36a9b5e12872c59ee"

Pour la partie informations d’identifications au serveur PVE, remplacez les valeurs données par les vôtres.

Pour l’image, il suffit de regarder sur le dépôt [Debian relatif aux images cloud](https://cloud.debian.org/images/cloud/). Pour le choix de la _somme de contrôle_ à utiliser, il suffit de regarder dans le fichier `SHA512SUMS` situé dans le dossier des images et de choisir la bonne ligne.

## Petit détour par Cloud-Init

Laisson de côté _OpenTofu_ un moment pour passer un peu de temps sur _Cloud Init_. Cloud-Init permet **d’automatiser l’initialisation d’instance cloud**. Il est utilisé par de nombreuses distribution Linux et FreeBSD.

Cette phase d’initialisation comprend entre autres la création d’utilisateur, la mise à jour des dépôts logiciels, l’installation de paquets. Les ordres sont donnés au service _Cloud-Init_ (lancé dans notre machine virtuelle) par l’intermédiaire de fichiers disponibles dans un lecteur amovible à l’intérieur de notre machine virtuelle.

Pour notre exemple, nous allons:

- créer un utilisateur et installer sa clé publique SSH
- définir le nom d’hôte de la machine
- mettre à jour les dépôts logiciel
- installer `qemu-guest-agent` et activer le service associé

Le dernier point nous permettra s’activer les intégrations de notre machine virtuelle de test avec notre hyperviseur.

### Vendor config

Cloud-Init utilise (entre autres) des fichers YAML pour gérer les configurations à appliquer. Le fichier `vendor-config` sert à appliquer les configurations relative **au fournisseur de l’infrastructure**: dans notre cas _Proxmox VE_.

Ce `vendor-config` nous permettra d’installer et configurer _l’agent Qemu_. Cet agent permet une meilleure intégration de notre machine virtuelle dans l’hyperviseur. Par exemple _OpenTofu_ sera capable de récupérer et afficher l’adresse IP de la machine créée.

Pour cela, ajoutons le répertoire `cloudinit` à notre projet et à l’intérieur le fichier `vendor-config.yaml` avec le contenu suivant:


```
#cloud-config 
  package_update: true
  packages:
  - qemu-guest-agent 
  runcmd:
  - systemctl enable --now qemu-guest-agent

```


Ce fichier parle de lui même:

- mise a jour des dépôts;
- installation du paquet `qemu-guest-agent`;
- activation du service associé via la directive `runcmd`;

### User config

Le fichier `user-config` est destiné à appliquer les configurations voulues par l’utilisateur **qui instancie la machine**. Nous allons nous en servir pour ajouter un utilisateur à notre machine.

Ajoutons le fichier `user-config.yaml` dans le répertoire `cloudinit/` avec le contenu suivant:

```
   `#cloud-config
   hostname: test-debian-12
   users: 
   - name: ephase
    gecos: My simple user
    ssh_authorized_keys:
     - ssh-ed25519 AAbB...   
     lock_passwd: false   
     sudo: ['ALL=(ALL) NOPASSWD:ALL']
      shell: /bin/bash`

```
La plupart des éléments (si ce n’est tous) doivent vous parler, `hostname` permet de définir le nom de la machines et `users` les comptes utilisateurs. Il faut bien entendu personnaliser ce fichier en fonction de vos besoins.

La documentation de Cloud-Init donne un [exemple complet](https://cloudinit.readthedocs.io/en/latest/reference/examples.html) des paramètres disponibles.

## Notre script OpenTofu

Nous allons construire notre `main.ft` pas à pas dans cette partie. Vous pouvex cependant télécharger [le script complet](https://xieme-art.org/post/instancier-une-machine-virtuelle-debian-dans-proxmox-avec-opentofu-et-cloud-init/files/main.tf) si vous voulez avoir une vue plus globale.

### Téléchargement de l’image Debian

Il nous reste maintenant notre script `main.tf` à créer. Commençons par le téléchargement de l’image Debian:


```
    resource "proxmox_virtual_environment_download_file" "debian_12" {
       content_type          = "iso"
       datastore_id          = "local"
       file_name             = "debian-12-generic-amd64.img"
       node_name             = var.pve_node       
	   url                   = var.debian_image_url  
	   checksum              = var.debian_image_checksum  
	   checksum_algorithm    = var.debian_image_checksum_algorithm  
	   overwrite             = true   
	   overwrite_unmanaged   = true 
    }	
  `
```
Ce bloc permet le téléchargement de l’image dans le stockage défini par `datastore_id` dans le fichier donné par le paramètre `file_name`.

Le paramètres `overwrite` défini le comportement de PVE si le fichier image existe déjà dans `datastore_id`:

- `false`: aucune action se sera effectuée, le fichier reste tel quel;
- `true`: si la taille du fichier local (sur PVE) et celle du fichier distant (donnée par l’entête HTTP`Content-Length`) sont différentes, alors l’image sera téléchargée à nouveau et le fichier local écrasé.

`overwrite_unmanaged` permet de remplacer l’image disque si elle existe sur `datastore_id` et n’est pas gérée par OpenTofu.

### Les fichiers Cloud-Init

Pour permettre à PVE de stocker les fichiers Cloud-Init envoyés par OpenTofu, nous devons activer le stockage des _snippets_ depuis l’interface web de Proxmox VE. Il nous suffit d’aller dans _Datacenter_ puis _Stockage_ et de double cliquer sur _local_.

![Capture d'écran de l'interface PVE montrant les paramètres du stockage local](https://xieme-art.org/post/instancier-une-machine-virtuelle-debian-dans-proxmox-avec-opentofu-et-cloud-init/imgs/pve_snippets.png)

Ouvrons ensuite la liste déroulante _Content_ et ajoutons _Snippets_ puis validons avec _OK_.

Une fois le stockage des snippets activés, ajoutons les instructions suivantes à notre fichier `main.tf`:


```
  resource "proxmox_virtual_environment_file" "user_config" {
    content_type  = "snippets"
    datastore_id  = "local"
    node_name     = var.pve_node
    source_raw {    
        data        = file("cloud-init/user-config.yaml")
        file_name   = "user-config.yaml"   
    }
   }
   resource "proxmox_virtual_environment_file" "vendor_config" {
      content_type  = "snippets"
      datastore_id  = "local"
      node_name     = var.pve_node
      source_raw {
           data        = file("cloud-init/vendor-config.yaml")
           file_name   = "vendor-config.yaml"
       }
   }`
```
Nous utilisons pour cela la ressource `proxmox_virtual_environment_file`:

- `content_type` permet de définir le type d’objet à stocker (images disque, image de conteneur, snippet);
- `source_raw` permet de définir la source, nous aurions pu utiliser la notation suivante pour l’option `data`[1](https://xieme-art.org/post/instancier-une-machine-virtuelle-debian-dans-proxmox-avec-opentofu-et-cloud-init/#fn:data_option):
    
    `raw {   data = <<-EOF   #cloud-config   users:     - name: ephase       # [...]   EOF }`
    

### Définition de notre machine virtuelle

Maintenant nous pouvons définir notre machine virtuelle par la définition d’une ressource `proxmox_virtual_environment_vm`.

`resource "proxmox_virtual_environment_vm" "debian_12_test" {     depends_on          = [          proxmox_virtual_environment_file.user_config,         proxmox_virtual_environment_file.vendor_config      ]     name                = "debian-12"     description         = "Debian 12 created with Terraform"     tags                = ["terraform", "debian"]     node_name           = var.pve_node      cpu {         cores   = 2     }     memory {         dedicated = 2048         floating  = 2048     }      disk {         datastore_id    = "local-lvm"         file_id         = proxmox_virtual_environment_download_file.debian_12.id         interface       = "virtio0"         iothread        = true         discard         = "on"         ssd             = true     }     network_device {         bridge          = "vmbr0"         model           = "virtio"     }     operating_system {         type            = "l26"     }     agent {         enabled         = true     }     initialization {         ip_config {             ipv4 {                 address     = "dhcp"             }         }         # Link cloud-init yaml files         user_data_file_id   = proxmox_virtual_environment_file.user_config.id         vendor_data_file_id = proxmox_virtual_environment_file.vendor_config.id     }`

Première chose importante, l’option `depends_on` qui permet à OpenTofu d’attendre que les définitions des deux fichiers _Cloud-Init_ soient effectuée avant de définir notre machine. Sans cette option, il arrive que la machine démarre sans que les fichiers Cloud-Init ne soient prêt, ainsi les personnalisations ne sont pas appliquées.

Ensuite nous trouvons un ensemble d’options qui définissent les paramètres de notre machine:

- les blocs `cpu {}` et `memory {}` pour les options relative au calcul comme le nombre de cœurs CPU ou la taille de la mémoire;
- le bloc `disk {}` permet de définir un périphérique de stockage;
- le bloc `network_device` permet de définir un périphérique réseau.

Pour `disk` et `network_device`, il est possible de définir **plusieurs blocs** afin d’instancier plusieurs périphériques. Je vous invite à aller voir la documentation [du module](https://github.com/bpg/terraform-provider-proxmox/blob/main/docs/resources/virtual_environment_vm.md) sur son dépôt GitHub.

#### Agent Qemu

**Le bloc `agent` est important**, il permet d’activer le **service d’intégration du côté _Proxmox VE_**. Invité et hôte pourront échanger des informations et activer des fonctionnalités avancées.

Côté invité, c’est les instructions données dans le fichier `vendor-config` qui s’occupe de l’installation de l’agent comme nous l’avons vu.

#### Paramétrage de Cloud-Init

C’est dans le bloc `initialization` que se font les paramétrages des éléments de _Cloud-Init_ relatifs à notre machine virtuelle.

Le bloc `ip_config` permet de définir les paramètres réseau. Le paramétrage se fait comme les autres blocs, via des valeurs affectées à des options. Mais il est aussi possible d’utiliser des fichiers YAML comme ce que nous avons définis plus haut. Pour `ip_config`, nous aurions pu utiliser `network_data_file_id` et donner un identifiant de fichier.

C’est ce que nous faisons pour `vendor_data_file_id` et `user_data_file_id` qui prennent comme valeurs les identifiants des fichiers définis dans les blocs `proxmox_virtual_environment_file`:

`# Link cloud-init yaml files         user_data_file_id   = proxmox_virtual_environment_file.user_config.id         vendor_data_file_id = proxmox_virtual_environment_file.vendor_config.id`

Nous passons en paramètre les _id_ de fichiers.

### Output

Afin de connaitre l’adresse IP de la machine qui va être créée, ajoutons un bloc `output` à la fin de notre fichier `main.tf`:

`output "debian_12_test_ip_address" {     value = proxmox_virtual_environment_vm.debian_12_test.ipv4_addresses }`

## Application de notre script OpenTofu

Maintenant que tout est en place nous pouvons nous lancer dans l’exécution de notre script. D’abord il faut initialiser Opentofu, c’est à cette étape qu’il va télécharger les modules:

`tofu init Initializing the backend...  Initializing provider plugins... - Reusing previous version of bpg/proxmox from the dependency lock file - Installing bpg/proxmox v0.64.0... - Installed bpg/proxmox v0.64.0 (signed, key ID F0582AD6AE97C188)  Providers are signed by their developers. If you'd like to know more about provider signing, you can read about it here: https://opentofu.org/docs/cli/plugins/signing/  OpenTofu has been successfully initialized!  You may now begin working with OpenTofu. Try running "tofu plan" to see any changes that are required for your infrastructure. All OpenTofu commands should now work.  If you ever set or change modules or backend configuration for OpenTofu, rerun this command to reinitialize your working directory. If you forget, other commands will detect it and remind you to do so if necessary.`

Vous pouvez lancer `tofu plan` afin de vérifier les actions effectuées par notre script, mais passons ici directement à l’action pour notre configuration:

`tofu apply # [...] Plan: 4 to add, 0 to change, 0 to destroy.  Changes to Outputs:   + debian_12_test_ip_address = (known after apply)  Do you want to perform these actions?   OpenTofu will perform the actions described above.   Only 'yes' will be accepted to approve.    Enter a value:`

Puis entrer _yes_ afin de valider, les actions seront alors:

`proxmox_virtual_environment_file.vendor_config: Creating... proxmox_virtual_environment_file.user_config: Creating... proxmox_virtual_environment_download_file.debian_12: Creating... proxmox_virtual_environment_file.vendor_config: Creation complete after 1s [id=local:snippets/vendor-config.yaml] proxmox_virtual_environment_file.user_config: Creation complete after 1s [id=local:snippets/user-config.yaml] proxmox_virtual_environment_download_file.debian_12: Creation complete after 8s [id=local:iso/debian-12-generic-amd64.img] proxmox_virtual_environment_vm.debian_12_test: Creating... proxmox_virtual_environment_vm.debian_12_test: Still creating... [10s elapsed] proxmox_virtual_environment_vm.debian_12_test: Still creating... [20s elapsed] proxmox_virtual_environment_vm.debian_12_test: Still creating... [30s elapsed] proxmox_virtual_environment_vm.debian_12_test: Still creating... [40s elapsed] proxmox_virtual_environment_vm.debian_12_test: Still creating... [50s elapsed] proxmox_virtual_environment_vm.debian_12_test: Still creating... [1m0s elapsed] proxmox_virtual_environment_vm.debian_12_test: Still creating... [1m10s elapsed] proxmox_virtual_environment_vm.debian_12_test: Creation complete after 1m15s [id=100]  Apply complete! Resources: 4 added, 0 changed, 0 destroyed.  Outputs:  debian_12_test_ip_address = tolist([   tolist([     "127.0.0.1",   ]),   tolist([     "192.168.0.112",   ]), ])`

Notre machine est en place avec l’adresse `192.168.0.112`, c’est grâce à **l’intégration de l’agent Qemu** qu’OpenTofu obtient cette information et nous l’affiche. Maintenant vérifions que notre utilisateur existe et que la clé publique que nous avons fournie dans `user-config.yaml` fonctionne:

`ssh ephase@192.168.0.112 -i ~/.ssh/id_ed25519 # [...] ephase@debian-12-test:~$ hostnamectl hostname test-debian-12 ephase@debian-12-test:~$ sudo id uid=0(root) gid=0(root) groups=0(root)`

Notre connexion fonctionne, notre utilisateur existe bien, le nom d’hôte est bien celui voulu et le paramétrage de `sudo` fonctionne aussi.

## En conclusion

En partant d’une image Debian minimale, nous avons réussi à créer une machine virtuelle paramétrée en automatisant autant que possible les actions à effectuer à l’aide de briques logicielles open-sources. Bien sûr nous avons juste effleuré les possibilités offertes par tous ces éléments, mais j’espère vous avoir donné l’envie d’aller plus loin.

Merci à [Heuzef](https://heuzef.com/) et [RavenRamirez](https://aloisbouny.fr/) pour les relectures, corrections et conseils.