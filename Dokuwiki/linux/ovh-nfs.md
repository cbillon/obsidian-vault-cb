---
tags:
  - ovh
  - nfs
---

====== Configuration Backup OVH ======

Acces à OVH cloud pour déclarer l’accès du serveur au backup OVH

[[https://help.ovhcloud.com/csm/en-dedicated-servers-services-backup-storage?id=kb_article_view&sysparm_article=KB0043985]]

L'adresse du serveur ns3158894 à sauvegarder : 51.91.104.76

===== Configuration lftp =====

  debug 3
  set ftp:ssl-allow no
  set ssl:verify-certificate no
  open ns3036570.ip-164-132-161.eu:iVtnpSzriw@ftpback-rbx7-592.mybackup.ovh.net
  
  
le serveur: ftpback-rbx7-592.mybackup.ovh.net
le service: ns3036570.ip-164-132-161.eu
mot de passe: iVtnpSzriw

==== Trouver l'adresse ip du serveur backup ====

  ping ftpback-rbx7-592.mybackup.ovh.net
  
  PING ftpback-rbx7-592.mybackup.ovh.net (145.239.11.86) 56(84) bytes of data.
  64 bytes from ftpback-rbx7-592.ovh.net (145.239.11.86): icmp_seq=1 ttl=54 time=1.93 ms
  
Trouver la conf du serveur distant nfs
  
  showmount -e ftpback-rbx7-592.mybackup.ovh.net

  Export list for ftpback-rbx7-592.mybackup.ovh.net:
  /export/ftpbackup/ns3036570.ip-164-132-161.eu 51.91.104.76/32

on a les 2 éléments
  *  /export/ftpbackup/ns3036570.ip-164-132-161.eu  répertoire partagé
  * 51.91.104.76/32 adresse ip du serveur autorisé

===== Configuration du client nfs =====


  * Installer le client nfs sur le serveur


  sudo apt install nfs-common
  

  * créer le point de montage 
  

  sudo mkdir -p /mnt/nfs/ovh
 
 
  * montage du répertoire distant


  sudo mount -t nfs -v ftpback-rbx7-592.mybackup.ovh.net:/export/ftpbackup/ns3036570.ip-164-132-161.eu 
  /mnt/nfs/ovh


  * Vérifier le fonctionnement

  sudo ls -l /mnt/nfs/ovh

  total 54
  drwxr-xr-x 2 systemd-network users 9 May  8 16:13 iso
  drwxr-xr-x 4 systemd-network users 4 Aug  5  2021 moodle39
  drwxr-xr-x 6 systemd-network users 6 Jan  9  2022 save_prod_39
  drwxr-xr-x 4 systemd-network users 4 Mar  1  2022 scripts

  * pour demonter le point de montage

  sudo umount /mnt/nfs/ovh
  
=====  Installation AUTOFS =====

  * Install autofs

   sudo apt install autofs
   
  * sauvegarde auto.fs

  sudo mv /etc/auto.master  /etc/auto.master.bck

  * créer un nouveau fichier 

  
    sudo nano /etc/auto.master 

    /-    /etc/auto.nfs  --ghost,--timeout=60
    

  * créer le fichier auto.nfs

  #  Adresse ip server backup: 145.239.11.86
  /mnt/nfs/ovh     -fstype=nfs,rw,intr     145.239.11.86:/export/ftpbackup/ns3036570.ip-164-132-161.eu
  
  * recharger la configuration

  sudo systemctl reload autofs
  sudo systemct status autofs
  

  * pour verifier

  sudo ls -l /mnt/nfs/ovh
  
  sudo df -h
  
 

