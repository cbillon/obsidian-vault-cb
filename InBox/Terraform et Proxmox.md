---
tags:
  - Terraform
  - Proxmox
  - VM
---


Cet article est un peu le préquel de celui sur [Ansible](https://blog.levassb.ovh/post/ansible/). Cette fois, on va s’attarder sur l’étape précédente avec le déploiement d’une dizaine de VM sous Promox avec Terraform.

Terraform est un outil libre d’orchestration développé par [HashiCorp](https://www.hashicorp.com/) à qui on doit déjà les excellents Vault, Packer et Consul. Il est utilisé pour provisionner des infrastructures complètes (serveurs, équipements réseaux, DNS, Firewall…) en utilisant un langage déclaratif (HCL Hashicop Configuration Language). Terraform est aujourd’hui un standard de fait de l’IaC (Infrastructure as Code).

## Configurer Proxmox

## Création d’un jeton d’API[](https://blog.levassb.ovh/post/terraform/#cr%C3%A9ation-dun-jeton-dapi)

Je commence par déclarer un rôle dédié avec les permissions nécessaires pour créer des VMs.

En SSH sur le serveur Proxmox :

- créer un rôle dédié `Terraform`

```
1pveum role add Terraform -privs "VM.Allocate VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Monitor VM.Audit VM.PowerMgmt Datastore.AllocateSpace Datastore.Audit"
```

bash

- ajouter un utilisateur dédié `terraform` dans le royaume `pve` (voir [gestion des utilisateurs](https://pve.proxmox.com/wiki/User_Management))

```
1pveum user add terraform@pve -password <password> -comment "Terraform account"
```

bash

- affecter le rôle `Terraform` à l’utilisateur `terraform@pve`

```
1pveum aclmod / -user terraform@pve -role Terraform
```

bash

- créer le jeton pour accéder à l’API (à conserver précieusement)

```
 1pveum user token add terraform@pve terraform -expire 0 -privsep 0 -comment "Terraform token"
 2┌──────────────┬──────────────────────────────────────────────────────────┐
 3│ key          │ value                                                    │
 4╞══════════════╪══════════════════════════════════════════════════════════╡
 5│ full-tokenid │ terraform@pve!terraform                                  │
 6├──────────────┼──────────────────────────────────────────────────────────┤
 7│ info         │ {"comment":"Terraform token","expire":"0","privsep":"0"} │
 8├──────────────┼──────────────────────────────────────────────────────────┤
 9│ value        │ c9995d9f-72d9-4adc-8ddb-80a0ccf470cc                     │
10└──────────────┴──────────────────────────────────────────────────────────┘
```

bash

L’option `-privsep 0` permet l’héritage des permissions de l’utilisateur.

Je peux maintenant interroger directement l’[API Proxmox](https://pve.proxmox.com/pve-docs/api-viewer/) en utilisant ce jeton (la liste des noeuds dans cet exemple):

Depuis un poste client:

```
1curl -X GET 'https://pve-01.exemple.tld:8006/api2/json/nodes' -H 'Authorization: PVEAPIToken=terraform@pve!terraform=c9995d9f-72d9-4adc-8ddb-80a0ccf470cc'
2
3{"data":[{"status":"online","id":"node/pve-01","node":"pve-01","ssl_fingerprint":"C9:3B:55:FD:15:4F:35:46:75:85:B8:D7:CD:8E:F9:0C:75:39:3D:99:29:EF:56:F3:6C:F2:7C:21:70:08:95:1A","level":"","type":"node"}]}%
```

bash

## Création d’un modèle de VM[](https://blog.levassb.ovh/post/terraform/#cr%C3%A9ation-dun-mod%C3%A8le-de-vm)

Terraform ne réalise pas de miracle, il se contente de cloner un modèle de VM et plus précisément une image “cloud” pour bénéficier de la personnalisation apportée par [cloud-init](https://cloud-init.io/).

La plupart des distributions fournissent des images au format `qcow2` pour KVM:

- [Debian](https://cloud.debian.org/images/cloud/)
- [Ubuntu](https://cloud-images.ubuntu.com/focal/current/)
- [Rocky Linux](https://download.rockylinux.org/pub/rocky/8/images/)

Exemple avec Debian 11 (toujours en SSH sur le serveur Proxmox):

- télécharger l’image debian

```
1cd /tmp
2wget https://cdimage.debian.org/cdimage/cloud/bullseye/latest/debian-11-genericcloud-amd64.qcow2
```

bash

- créer une VM avec un VMID libre (1000 dans l’exemple), adapter le nom du bridge à votre configuration réseau (vmbr1 dans mon cas pour le traffic réseau des VMs)

```
1qm create 1000 --memory 1024 --core 1 --name debian11-temp --net0 virtio,bridge=vmbr1 --description "Debian 11 cloud template"
```

bash

- importer l’image Debian dans l’espace de stockage des VMs (“local-lvm” dans l’exemple)

```
1qm importdisk 1000 /tmp/debian-11-genericcloud-amd64.qcow2 local-lvm
2importing disk '/tmp/debian-11-genericcloud-amd64.qcow2' to VM 1000 ...
3  Logical volume "vm-1000-disk-0" created.
4Successfully imported disk as 'unused0:local-lvm:vm-1000-disk-0'
```

bash

- attacher le disque au contrôleur de la VM

```
1qm set 1000 --scsihw virtio-scsi-pci --scsi0 local-lvm:vm-1000-disk-0
2update VM 1000: -scsi0 local-lvm:vm-1000-disk-0 -scsihw virtio-scsi-pci
```

bash

- définir le disque comme périphérique de boot par défaut

```
1qm set 1000 --boot c --bootdisk scsi0
```

bash

- ajouter le CDrom pour cloud-init

```
1qm set 1000 --ide2 local-lvm:cloudinit
```

bash

- ajouter un port serie et l’utiliser pour afficher la console

```
1qm set 1000 --serial0 socket --vga serial0
```

bash

- convertir la VM en modèle (template)

- supprimer l’image Debian téléchargée

```
1rm -f /tmp/debian-11-genericcloud-amd64.qcow2
```

bash

Si tout c’est bien passé, vous devriez obtenir ce résultat:

![VM template](https://blog.levassb.ovh/images/vm-template.png)

Figure 2: VM template

## Installer Terraform

Terraform est un binaire écrit en Go. Hashicop fournit un dépôt pour la majorité des distributions ([https://www.terraform.io/downloads](https://www.terraform.io/downloads)) mais le plus simple reste encore de télécharger directement le binaire compilé pour votre plateforme.

exemple pour Mac OSX Intel (sur un poste client):

```
1wget https://releases.hashicorp.com/terraform/1.1.7/terraform_1.1.7_darwin_amd64.zip
2unzip terraform_1.1.7_darwin_amd64.zip
3sudo mv terraform /usr/local/bin/
4sudo chmod +x /usr/local/bin/terraform
5
6terraform -version
7Terraform v1.1.7
8on darwin_amd64
```

bash

## Utiliser Terraform

Le cycle de vie d’un projet Terraform est décomposé en 4 phases:

![Cycle Terraform](https://blog.levassb.ovh/images/terraform-cycle.svg)

Figure 3: Cycle Terraform

- **Init** : initialise le répertoire de travail et installe les fournisseurs (providers) nécessaires
- **Plan** : calcule les opérations à effectuer pour obtenir l’état souhaité
- **Apply** : applique les opérations pour atteindre l’état souhaité
- **Destroy** : supprime les ressources

Terraform fournit un [registre](https://registry.terraform.io/browse/providers) qui référence les `provider` pour lui permettent de communiquer avec une multitude de plateformes et d’équipements. Pour Proxmox, le plus populaire est celui proposé par [Telmate](https://github.com/Telmate/terraform-provider-proxmox).

Sur un poste client, je vais créer un dossier pour mon projet et 3 fichiers de configuration:

```
1mkdir Proxmox-Terraform && cd Proxmox-Terraform
2touch provider.tf main.tf variables.tfvars
3tree
4.
5├── main.tf
6├── provider.tf
7└── variables.tfvars
```

bash

- **provider.tf** : pour installer et configurer le provider Proxmox (mettre la dernière version disponible)

```
 1terraform {
 2    required_providers {
 3        proxmox =  {
 4        source = "Telmate/proxmox"
 5        version = "2.9.6"
 6        }
 7    }
 8}
 9
10provider "proxmox" {
11    pm_api_url = var.pm_api_url
12    pm_api_token_id= var.pm_api_token_id
13    pm_api_token_secret= var.pm_api_token_secret
14    #pm_tls_insecure = true  //for Proxmox self-signed certificate WUI
15}
```

bash

- **variables.tfvars** : pour contenir les variables nécéssaires

Je reprends ici le jeton API et le nom du modèle de VM créés précédemment ainsi que le nom du noeud Proxmox qui va exécuter les VMs.

```
 1variable  "pm_api_url" {
 2    default =  "https://pve-01.exemple.tld:8006/api2/json"
 3}
 4variable  "pm_api_token_id" {
 5    default =  "terraform@pve!terraform"
 6}
 7variable  "pm_api_token_secret" {
 8    default =  "c9995d9f-72d9-4adc-8ddb-80a0ccf470cc"
 9}
10variable  "target_node" {
11    default =  "pve-01"
12}
13variable "clone" {
14    default =  "debian11-temp"
15}
16variable  "ssh_key" {
17    default =  "ssh-rsa AAAAB3NzaC1y............c2EAAAABIwAAA== user@destop.exemple.tld"
18}
```

bash

- **main.tf** : pour décrire l’état souhaité (ajout de 10 VMs)

Dans cette configuration, j’utilise une itération sur la valeur “count” pour construire le nom de la VM et son adresse IP (test-vm-1 à test-vm-10).

```
 1resource "proxmox_vm_qemu" "tp_servers" {
 2  desc = "Deploiement 10 VM Debian sur Proxmox"
 3  count = 10
 4  name = "test-vm${count.index + 1}"
 5  target_node = var.target_node
 6  clone = var.clone
 7
 8  os_type = "cloud-init"
 9  cores = 2
10  sockets = 1
11  cpu = "host"
12  memory = 2048
13  scsihw = "virtio-scsi-pci"
14  bootdisk = "scsi0"
15
16  disk {
17    slot = 0
18    size = "2G"
19    type = "scsi"
20    storage = "local-lvm"
21    iothread = 1
22  }
23
24  network {
25    model = "virtio"
26    bridge = "vmbr1"
27  }
28
29  ipconfig0 = "ip=192.168.1.${count.index + 1}/24,gw=192.168.1.1"
30  nameserver = 192.168.1.254
31  searchdomain = exemple.tld
32}
```

...

bash

### terraform init[](https://blog.levassb.ovh/post/terraform/#terraform-init)

La première commande à lancer est un `terraform init` pour initialiser le répertoire et travail mais aussi installer le provider Proxmox.

```
1terraform init
2Initializing the backend...
3Initializing provider plugins...
4- Finding telmate/proxmox versions matching "2.9.6"...
5- Installing telmate/proxmox v2.9.6...
6- Installed telmate/proxmox v2.9.6 (self-signed, key ID A9EBBE091B35AFCE)
7
8Terraform has been successfully initialized!
```

bash

### terraform plan[](https://blog.levassb.ovh/post/terraform/#terraform-plan)

Avec la commande `terraform plan`, Terraform va determiner les actions à accomplir pour atteindre l’objectif que j’ai défini dans le `main.tf`. Le résultat de la commande est relativement verbeuse (tronqué dans l’exemple) et permet de connaitre précisément les opérations qui vont être réalisées.

```
 1terraform plan
 2 + create
 3
 4Terraform will perform the following actions:
 5
 6 # proxmox_vm_qemu.test_server[0] will be created
 7 + resource "proxmox_vm_qemu" "test_server" {
 8     + clone                     = "debian11-temp"
 9     + cores                     = 2
10     + cpu                       = "host"
11     + full_clone                = true
12     + ipconfig0                 = "ip=192.168.1.1/24,gw=192.168.1.1"
13     + memory                    = 2048
14     + name                      = "test-vm-1"
15     + nameserver                = "192.168.1.254"
16     + os_type                   = "cloud-init"
17     + scsihw                    = "virtio-scsi-pci"
18     + searchdomain              = "exemple.tld"
19     + sockets                   = 1
20     + tablet                    = true
21     + target_node               = "pve-01"
22
23     + disk {
24         + size         = "2G"
25         + storage      = "local-lvm"
26         + type         = "scsi"
27       }
28
29     + network {
30         + bridge    = "vmbr1"
31         + macaddr   = (known after apply)
32         + model     = "virtio"
33         + queues    = (known after apply)
34         + rate      = (known after apply)
35         + tag       = -1
36       }
37   }
```

...

bash

### terraform apply[](https://blog.levassb.ovh/post/terraform/#terraform-apply)

C’est maintenant que la magie s’opère avec la commande `terraform apply --auto-approve` (l’option `--auto-approve` permet d’éviter le prompt de confirmation) qui va provisionner les VMs sur le serveur Proxmox.

```
 1terraform apply --auto-approve
 2proxmox_vm_qemu.test_server[8]: Creation complete after 1m59s [id=pve-01/qemu/107]
 3proxmox_vm_qemu.test_server[3]: Still creating... [2m0s elapsed]
 4proxmox_vm_qemu.test_server[9]: Still creating... [2m10s elapsed]
 5proxmox_vm_qemu.test_server[9]: Creation complete after 2m11s [id=pve-01/qemu/108]
 6proxmox_vm_qemu.test_server[7]: Creation complete after 2m12s [id=pve-01/qemu/109]
 7proxmox_vm_qemu.test_server[3]: Still creating... [2m20s elapsed]
 8proxmox_vm_qemu.test_server[3]: Still creating... [2m30s elapsed]
 9proxmox_vm_qemu.test_server[3]: Creation complete after 2m34s [id=pve-01/qemu/110]
10proxmox_vm_qemu.test_server[6]: Still creating... [2m40s elapsed]
11proxmox_vm_qemu.test_server[6]: Creation complete after 2m47s [id=pve-01/qemu/111]
12
13Apply complete! Resources: 10 added, 0 changed, 0 destroyed.
```

bash

Et voilà: **2m47s** pour créer 10 VMs prêtes à l’emploi !

```
 1qm list
 2      VMID NAME                 STATUS     MEM(MB)    BOOTDISK(GB) PID
 3       102 test-vm-2            running    2048               2.00 536444
 4       103 test-vm-1            running    2048               2.00 536524
 5       104 test-vm-3            running    2048               2.00 536638
 6       105 test-vm-6            running    2048               2.00 536795
 7       106 test-vm-5            running    2048               2.00 537133
 8       107 test-vm-9            running    2048               2.00 537308
 9       108 test-vm-10           running    2048               2.00 537533
10       109 test-vm-8            running    2048               2.00 537529
11       110 test-vm-4            running    2048               2.00 537743
12       111 test-vm-7            running    2048               2.00 537898
13      1000 debian11-temp        stopped    1024               2.00 0
```

bash

Contrairement à Ansible, Terraform fonctionne en mode conservation d’état (stateful) et garde localement le résultat de la commande `apply` ( fichier `terraform.tfstate`). En cas de modification du fichier `main.tf`, Terraform n’exécutera que le delta entre la configuration en place et la nouvelle. Si je souhaite 5 VMs de plus, il suffit de passer la valeur `count` à 15. La commande `terraform plan` va m’indiquer `Plan: 5 to add, 10 to change, 0 to destroy.` avec 5 nouvelles VMs dont les noms qui commenceront bien à partir de `test-vm-11` à `test-vm-15`.

### terraform destroy[](https://blog.levassb.ovh/post/terraform/#terraform-destroy)

La commande `terraform destroy` va comme son nom l’indique supprimer toutes les ressources créées

```
 1terraform destroy
 2proxmox_vm_qemu.test_server[1]: Destruction complete after 5s
 3proxmox_vm_qemu.test_server[3]: Destruction complete after 5s
 4proxmox_vm_qemu.test_server[2]: Destruction complete after 5s
 5proxmox_vm_qemu.test_server[5]: Destruction complete after 7s
 6proxmox_vm_qemu.test_server[6]: Destruction complete after 9s
 7proxmox_vm_qemu.test_server[7]: Destruction complete after 9s
 8proxmox_vm_qemu.test_server[0]: Destruction complete after 9s
 9proxmox_vm_qemu.test_server[9]: Destruction complete after 11s
10proxmox_vm_qemu.test_server[4]: Destruction complete after 14s
11proxmox_vm_qemu.test_server[8]: Destruction complete after 14s
12
13Destroy complete! Resources: 10 destroyed.
```

bash

## Conclusion

Terraform est un outil puissant qui remplace avantageusement les solutions à base de PXE et de Preseed. Il est le compagnon idéal d’Ansible. L’article ne couvre que la création de VMs mais sachez que le provider de Telmate permet aussi de créer des [conteneurs LXC](https://github.com/Telmate/terraform-provider-proxmox/blob/master/examples/lxc_example.tf) avec la même facilité déconcertante.

## Ressources

- [how to deploy vms in proxmox with terraform](https://austinsnerdythings.com/2021/09/01/how-to-deploy-vms-in-proxmox-with-terraform/)
- [terraformer son server](https://blog.zedas.fr/posts/terraformer-son-server/)
- [how to create proxmox cloud templates](https://blog.f2h.cloud/how-to-create-proxmox-cloud-templates/)
- [Cloud images in Proxmox](https://gist.github.com/chriswayg/b6421dcc69cb3b7e41f2998f1150e1df)
- [cloud init cloud image](https://techno-tim.github.io/posts/cloud-init-cloud-image/)
- [utiliser une image cloud init dans proxmox](https://atlanticaweb.fr/p/utiliser-une-image-cloud-init-dans-proxmox/)
- [homelab with proxmox and terraform](https://cloudalbania.com/posts/2022-01-homelab-with-proxmox-and-terraform/)
- [automating proxmox with terraform ansible](https://vanmieghem.io/automating-proxmox-with-terraform-ansible/)
- [provision proxmox vms with terraform](https://vectops.com/2020/05/provision-proxmox-vms-with-terraform-quick-and-easy/)
- [Doc Proxmox - Cloud-Init_Support](https://pve.proxmox.com/wiki/Cloud-Init_Support)
- [Terraform for beginners](https://geekflare.com/terraform-for-beginners/)