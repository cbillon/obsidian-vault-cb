---
tags:
  - Proxmox
  - Cloud-Init
  - template
  - VM
---

 ![Christophe Casalegno](https://www.youtube.com/watch?v=jiV4R4-DyMY)

# Création des templates

Les images 
	- [Debian](https://cloud.debian.org/images/cloud/)
	- [Ubuntu](https://cloud-images.ubuntu.com/)

sélectionner une image correspondant à l(architecture du processeur (amd-64)) au format qcow2

sur Proxmox crééer une VM
- detach disk , remove disk
- noter le n° de la VM ici 103

Ouvrir le shell 
- wget <image-cloud>
  - qm importdisk 103  <image-cloud> vmstorage --format qcow2   
  - qm set 103 --serial0 socket --vga serial0
  - exit

nota: le nom du pooll est vmstorage

sur la VM :
- Hardware select unused Disk0  
- -> Edit -> cocher Discard si disk SSD -> Add
- Disk Action Resize
- Add CloudInit selectionner vmstorage
- Options : chnager boot order sélectionner le disque ajouté
- -Editer les options cloud-init
	- nom user cb password sesame
	- dns 1.1.1.1 8.8.8.8 9.9.9.9
	- recopier la clé public id25519
- clone (full)
Mzttre à jour la config mémoire, cpu, disk, puis regenerate
- Start
pour vérifier la connexion : ssh -l cb 192.168.1.83
