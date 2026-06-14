---
tags:
  - docker
  - dockerfile
  - bash
---
Une pratique commune est d’utiliser un script shell en tant qu’ENTRYPOINT, qui va effectuer des opérations quelconques (injection de secrets, initialisation de base de données…), puis démarrer le service, sous forme d’un ou de plusieurs processus.

## Entrypoint qui lance un unique processus

Dans ce cas, le Dockerfile spécifie :

Une instruction ENTRYPOINT associée à un script shell
Une instruction CMD, passée en arguments de l’entrypoint
L’entrypoint est exécuté par un shell comme /bin/sh ou /bin/bash. C’est le premier processus exécuté dans le conteneur, il a donc le PID 1.

Traditionnellement, ce script lance le service désigné par CMD.
Probléme
Les scripts ne transmettent pas les signaux à leurs enfants (voir explications ici).
Le processus désigné par CMD ne reçoit donc jamais les signaux qui sont envoyés au conteneur. Or, ces signaux peuvent être importants : signal de terminaison, signal de rechargement de configuration… De plus, un processus tué brutalement peut créer des situations indésirables, comme la corruption d’une base de données. Il est donc très important qu’il reçoive les signaux qui lui sont destinés.



Comment faire ?

Utiliser exec
Le comportement désiré serait d’exécuter les instruction de l’entrypoint, puis de remplacer l’entrypoint par son processus enfant. C’est exactement le but de la commande exec.

On remplace l’entrypoint par :

snippet.bash
#!/bin/sh
 ...
exec $@

$@ designe l'ensemble des arguments passés passés à l'ENTRYPOINT c'est à dire le contenu de CMD du Dockerfile

L’entrypoint a été remplacé par CMD. Il reçoit correctement les signaux.