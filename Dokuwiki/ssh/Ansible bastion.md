---
"auteur:": Aurelien Maury
"tags:": "[ansible, ssh, bastion]"
lien: "[site](https://blog.wescale.fr/gerer-les-bastions-avec-ansible)"
tags:
  - ansible
  - bastion
---

## Gérer les bastions avec Ansible 

Il arrive souvent que les machines de votre parc ne soient pas toutes accessibles directement via SSH depuis votre poste de travail. Dans ce genre de cas, il existe souvent une machine bastion, qui est l’unique point d’entrée vers cette zone réseau. Voici une astuce bien pratique pour atteindre vos machines non-exposées en passant au travers d’un bastion de façon transparente.

![[ssh-bastion-1.webp]]

Ansible s’appuie fortement sur SSH et permet de faire des paramétrages fins sur les paramètres SSH à appliquer lors de la connexion de Ansible à une machine cible. Il est notamment possible de passer un fichier de configuration standard SSH en jouant sur les variables du groupe ssh_connection.

La façon de faire la plus pratique est de poser un fichier ansible.cfg à l’endroit d’où vous comptez lancer vos commandes Ansible. Ce fichier de configuration sera pris en compte automatiquement, simplement du fait de sa présence dans le répertoire courant au lancement d’Ansible.

===== ansible.cfg =====

  [ssh_connection]
  ssh_args = -F ssh.cfg
  control_path = ~/.ssh/mux-%r@%h:%p
  
Avec cette configuration, Ansible passera le fichier de configuration ssh.cfg en paramètre à SSH pour toute connexion. L’astuce est de forger votre configuration SSH avec les directives permettant d’adresser les IP internes de votre zone privée en utilisant le bastion comme proxy.

===== ssh.cfg =====

  # Connexion directe avec le bastion.
  # Pensez à adapter le User et le IdentityFile selon vos besoins.
  Host bastion
  Hostname 84.39.41.33
  User admin
  IdentityFile /home/you/.ssh/your_key.pem
  # Pour toutes les machines de la zone privée :
  # Vous pouvez renseigner un range d’IPs ou une zone dns, exemple:
  #    *.eu-west-1.compute.amazonaws.com
  Host 192.168.47.*
  # Proxifier la connexion au travers du bastion.
  ProxyCommand ssh -F ssh.cfg -W %h:%p bastion
  # A adapter à votre cas : le User et la clé pour les connexions aux machines privées.
  User admin
  IdentityFile /home/you/.ssh/your_key.pem
  # Directives de multiplexing SSH
  Host *
    ControlMaster   auto
    ControlPath     ~/.ssh/mux-%r@%h:%p
    ControlPersist  15m

Voilà ! C’est fait ! Maintenant, à supposer que votre inventaire d’hôtes Ansible ressemble à ceci :

===== inventory =====

  [bastion]
  84.39.41.33
  [private_servers]
  192.168.47.10
  192.168.47.11
  192.168.47.12
  192.168.47.13

Il ne reste plus qu’à tester qu’Ansible arrive bien à se connecter aux machines grâce au module Ansible ping qui va effectuer une connexion SSH, vérifier la disponibilité d’un environnement Python adapté à Ansible et répondre pong en cas de succès.


  ansible -i inventory -m ping all
  192.168.47.11 | success >> {
      "changed": false,
      "ping": "pong"
  }
  192.168.47.12 | success >> {
      "changed": false,
      "ping": "pong"
  }
  bastion | success >> {
      "changed": false,
      "ping": "pong"
  }
  192.168.47.13 | success >> {
      "changed": false,
      "ping": "pong"
  }
  192.168.47.10 | success >> {
      "changed": false,
      "ping": "pong"
  }

Toutes les machines de l’inventaire ont été contactées et ont répondu pong ? C’est gagné, vous pouvez piloter votre zone privée depuis votre poste de travail au travers de votre bastion.