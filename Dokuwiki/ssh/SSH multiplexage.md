---
tags:
  - ssh
---

====== Multiplexage SSH ======

Pour rendre ça plus rapide, il y a toujours l’option de multiplexage de connexions SSH si vous ne connaissez pas. L’idée est d’ouvrir un agent de multiplexage gardant une connexion ouverte le temps que vous passez toutes les commandes que vous voulez à travers la même connexion TCP, ce qui réduit le temps de chaque commande à exécuter.

Pour l’activer, ça se passe par le fichier de configuration avec les paramètres suivants :

  ControlPath ~/.ssh/%r@%h:%p # Spécifie l'emplacement du fichier socket, dans votre homedir c'est bien
  ControlMaster auto          # Comportement du multiplexage. auto utilise la connexion si déjà ouverte
  ControlPersist 5m           # Combien de temps la connexion sera maintenue ouverte

À noter que cette option utilise un process ssh supplémentaire pendant la durée du ControlPersist. Comme toute configuration cliente SSH, vous pouvez soit inclure ceci globalement via votre fichier /etc/ssh/ssh_config, soit juste pour votre utilisateur et potentiellement par hôte cible via votre fichier ~/ssh/config

  Host mon-serveur
    ControlPath ~/.ssh/%r@%h:%p 
    ControlMaster auto          
    ControlPersist 5m           

Enfin, vous pouvez également l’exécuter seulement pour une connexion indépendante grâce aux options inline:

  opts="-o ControlPath=~/.ssh/%r@%h:%p -o ControlMaster=auto -o ControlPersist=10m"
  ssh $opts user@host env
  ssh $opts user@host "echo merci d\'avoir lu jusqu\'ici !"

