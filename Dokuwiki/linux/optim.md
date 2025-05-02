---
tags:
  - linux
  - optimisation
---

====== Optimisation du temps de démarrage Ubuntu Linux ======

Depuis quelques temps, je trouve que le temps de démarrage de mon PC sous Ubuntu est bien long... alors que le disque principal est un SSD.

===== Identifier le temps total de démarrage =====

Sur les O.S. qui utilisent systemd, il existe une commande permettant de connaître le temps nécessaire au démarrage : systemd-analyze.

Sans option, cette commande donne juste le temps total nécessaire au démarrage, ainsi que le temps après lequel on arrive à l'écran de connexion. Exemple :

  systemd-analyze 
 

===== Identifier les services les plus lents =====

La commande system-analyze utilisée avec l'option blame donne la liste des services en cours, triés par ordre de temps nécessaire à leur initialisation (ordre décroissant).

Exemple :

  systemd-analyze blame
        
Note : le temps de démarrage d'un service peut être long car il attend la fin d'un autre service.
 

On peut utiliser l'option critical-chain pour identifier toute la chaine des services qui sont sur le "chemin critique", ce qui permet de mieux cibler les pistes d'amélioration.
Exemple :

  systemd-analyze critical-chain

Comme l'indique la commande, le temps de démarrage du service (unit dans la terminologie systemd) est donné après le caractère '+'.

===== Optimisation =====

La liste des services "lents" doit ensuite être analysée, en fonction du contexte. Sur mon PC personnel, je n'ai pas forcément besoin de lancer à chaque démarrage :

un serveur nginx
les daemon libvirtd et qemu (pour des VM KVM)
le service ModemManager (mais pourquoi donc)
un serveur postfix
On peut donc désactiver les différents services au démarrage :

sudo systemctl disable nginx
sudo systemctl disable libvirtd
sudo systemctl disable postfix
sudo systemctl disable ModemManager
Sachant qu'on pourra toujours démarrer les services manuellement, par exemple :

# démarrage nginx
sudo systemctl start nginx 
On peut ensuite rebooter pour vérifier l'impact :

  systemd-analyze 

On a gagné environ 30 secondes sur le temps de démarrage. Par contre l'écran de connexion ne s'affiche qu'une seconde plus tôt.
On a aussi désactivé des processus et donc potentiellement amélioré l'utilisation de la mémoire et de la CPU.