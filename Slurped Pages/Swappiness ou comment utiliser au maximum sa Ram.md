---
link: https://geekeries.de-labrusse.fr/?p=1806
byline: by stephane de labrusse
excerpt: Published on décembre 8, 2012
slurped: 2025-02-10T15:45
title: Swappiness ou comment utiliser au maximum sa Ram
---

Maintenant nous allons voir le [SWAPPINESS.](http://en.wikipedia.org/wiki/Swappiness)

En fait le système est simple puisque on dit a Linux quand il doit commencer a écrire sur le disque dur dans l’espace d’échange appelé SWAP afin de délester la Ram. Il peut paraître évident qu’avec des temps d’accès en millisecondes pour les HD et en nanoseconde pour la mémoire vive, il est préférable d’écrire dans cette dernière.

Et on ne parle même pas des vitesses d’écriture qui sont immensément plus rapides que celles des HD

En Clair :

- `vm.swappiness = 0` – Linux utilisera le HD en dernière limite pour éviter un manque de RAM.
- `vm.swappiness = 60` – Valeur par défaut de Linux : à partir de 40% d’occupation de Ram, le noyau écrit sur le disque.
- `vm.swappiness = 100` – tous les accès se font en écriture dans la SWAP.

Pour s’informer des réglages de swappiness

```
cat /proc/sys/vm/swappiness
```

Pour mon ordinateur (16G de Ram), j’estime qu’un swappiness à 90 est suffisant puisque la swap sera utilisée lorsqu’il ne restera plus que 1,6G :’)

Du coup il faut éditer le fichier suivant `/etc/sysctl.conf en root et ajouter cette Ligne`

```
vm.swappiness = 10
```

Lancer ces commandes en ROOT qui vont désactiver la SWAP pour la relancer avec les nouveaux réglages.

```
swapoff -a
swapon -a
```

Vous pouvez aussi changer les variables par cette commande en root, mais elles auront normalement disparu au reboot

```
sysctl vm.swappiness=10
```

That’s all folks, Enjoy.