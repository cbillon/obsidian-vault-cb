---
tags:
  - ssh
---


====== SSH Rebond ======

Dans notre cas la machine à atteindre à partir d'une machine locale est une machine virtuelle installée sur un serveur  accessible depuis internet 
Le serveur a une adresse Ip publique : **ns3036570.cbillon.ov**h
La machine virtuelle a comme adresse 192.168.122.XX, donc non accessible directement

Nous allons parametrer une machine locale qui va se connecter sur le serveur puis à partir de la sur la VM



===== Parametrage de la machine locale =====

Mettre à jour .ssh/config
La clé d'accus (public key) a été déposée sur le serveur

Acces au serveur:

  Host dev
    HostName ns3036570.cbillon.ovh
    User cb
    Port 50635
    ForwardX11 yes
    ForwardAgent yes
    IdentityFile ~/.ssh/id_rsa
  #  LogLevel DEBUG3  
  

Acces à la VM 

  Host 192.168.122.*
    User cb
    ProxyJump cb@dev
  #  LogLevel DEBUG3



===== Vérification de la connexion =====


  ssh 192.168.122.20
  

