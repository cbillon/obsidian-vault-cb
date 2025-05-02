---
tags:
  - ansible
  - tips
  - reboot
---

## Reboot server 

[Site](https://blog.wescale.fr/ansible-tips-reboot-continue)

Playbook

  
> [!example] Code
> ---- 
> hosts: pending-reboot  
> become: yes  tasks:    
>    - name: Attempting reboot
>     shell: reboot
>     async: 1200
>     poll: 0        
>   - name: Waiting for resurection
>     wait_for_connection:
>      delay: 60
>      timeout: 300
>    -  name: Is it still you Bob?
>      settup:
      -

  ---
  - h
        
Pour bien comprendre la mécanique, sachez que :

  * l'attribut async est positionnable sur n'importe quelle tâche et indique à Ansible de la lancer en asynchrone. La valeur de async donne la durée maximale d'exécution en secondes.
  * l'attribut poll vient avec async et donne la durée en secondes entre 2 vérifications de fin de tâches par Ansible.
  * La tâche Attempting reboot lance donc une simple commande shell, mais avec un async suffisament long pour qu'Ansible ne se préoccupe pas de couper le sous-processus, et un poll à 0 ce qui indique de ne jamais venir vérifier si le processus s'est bien terminé.

Enfin, la tâche Waiting for resurection va attendre 60 secondes avant de tenter de se connecter à nouveau à nos machines. Cela laisse largement le temps à sshd de s'éteindre et évite qu'Ansible ne valide la connexion comme active alors même que la machine va s'éteindre. Il y aura une tentative de connexion toutes les delay secondes jusqu'au timeout.

Une fois que la connexion est revenue, le flow de tâches continue normalement.