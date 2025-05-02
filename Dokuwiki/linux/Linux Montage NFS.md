---
tags:
  - linux
  - nfs
  - tips
---
## Montage NFS coté client


[digital ocean tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)

> [!warning] Firewall
> Sur le client et le serveur penser à mettre à jour le port de communication
> Port 2049

### Installation

installer sudo apt install nfs-common

### Montage

- créer un répertoire de montage :
sudo mkdir  /mnt/backup

###  1 solution : montage temporaire
sudo mount -t nfs 192.168.1.34:/volume1/Backup  /mnt/backup

pour vérifier : df -h

### 2 solution : montage permanent

mettre à jour fstab
sudo nano /etc/fstab

192.168.1.34:/volume1/Backup  /mnt/backup  nfs defaults 0 0

### 3 autofs

utiliser le package autofs
[[autofs]]
[voir]()
### 4 Avec un service systemd

[site ](https://blog.tomecek.net/post/automount-with-systemd/)

créer préalablement le  répertoire de montage
cela est nécessaire car ce nom est repris dans les fichiers à creer

Encode des répertoires /mnt/backup -> mnt-backup
crer 2 fichiers :

$ cat /etc/systemd/system/mnt-backup.automount
[Unit]
Description=Automount Backup

[Automount]
Where=/mnt/backup

[Install]
WantedBy=multi-user.target

$ cat /etc/systemd/system/mnt-backup.mount
[Unit]
Description=Backup

[Mount]
What=192.168.1.34:/volume1/Backup
Where=/mnt/backup
Type=nfs

[Install]
WantedBy=multi-user.target

Now to notify systemd there are some new files available:

$ systemctl daemon-reload

Runtime setup

Feel free to disable the mount, but enable the automount:

$ systemctl is-enabled mnt-scratch.mount
disabled
$ systemctl is-enabled mnt-scratch.automount                                                                                            
enabled
$ systemctl start mnt-scratch.automount                                                                                             
$ ls /mnt/scratch >/dev/null
$ systemctl status mnt-scratch.automount
● mnt-scratch.automount - Automount Scratch
   Loaded: loaded (/etc/systemd/system/mnt-scratch.automount; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2016-04-18 10:49:04 CEST; 4h 33min ago
    Where: /mnt/scratch

Apr 18 10:49:04 oat systemd[1]: Set up automount Automount Scratch.
Apr 18 10:49:14 oat systemd[1]: mnt-scratch.automount: Got automount request for /mnt/scratch, triggered by 20266 (zsh)
$ systemctl status mnt-scratch.mount
● mnt-scratch.mount - Scratch
   Loaded: loaded (/proc/self/mountinfo; disabled; vendor preset: disabled)
   Active: active (mounted) since Mon 2016-04-18 10:49:16 CEST; 4h 33min ago
    Where: /mnt/scratch
     What: nfs.example.com:/export/scratch

Apr 18 10:49:14 oat systemd[1]: Mounting Scratch...
Apr 18 10:49:16 oat systemd[1]: Mounted Scratch.




