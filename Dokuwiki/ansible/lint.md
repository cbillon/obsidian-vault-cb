---
tags:
  - ansible
---

## Ansible lint

[blog Stéphane Robert](https://blog.stephane-robert.info/post/ansible-check-lint/)

Pour afficher toutes les règles utilisées par défaut, il suffit de taper la commande suivante :

  ansible-lint -v -T
  
Pour limiter les tests sur certaines règles, il suffit d’utiliser l’option -t. Par exemple pour lancer un contrôle sur juste l’idempotence :

  ansible-lint -t idempotency playbook.yml

De la même manière pour exclure certaines règles :

  ansible-lint -x formatting,metadata playbook.yml

On peut plutôt que de les ignorer les afficher en warning:


  ansible-lint -w experimental playbook.yml
  
### Ansible-lint apporte certaines corrections tout seul 


Introduit avec la version 6.0.0, ansible-lint permet désormais d’apporter certaines corrections à votre place avec l’option --write :

  * L’utilisation des FQCN : fqcn
  * La première lettre de la description en capitale : name
  * les valeurs booléenes à true ou false : yaml

Cette option s’enrichit au fil de la sortie des versions.

### Utilisation de l’option –write

Cette option prend une valeur qui peut être all, none ou une liste des id de règles à corriger séparés par des virgules.

### Écrire ses propres règles Ansible-Lint 

Il est possible de créer ses propres règles et de les utiliser avec l’option -r /path/to/custom-rules.

Pour la syntaxe des règles il suffit de se rendre sur cette page : custom-rules

Si on veut les utiliser conjointement avec les règles par défaut, on remplace -w par -W.