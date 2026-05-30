---
tags:
  - docker
  - dockerfile
---
Si vous êtes familier avec Docker, il vous est sûrement déjà arrivé de vouloir arrêter un conteneur avec la commande `docker stop` et d'avoir à attendre plus de 10 secondes avant qu'il ne s'arrête. Ou même de ne pas réussir à le stopper en appuyant sur `CTRL+C`. Bien souvent, cela s'explique par une mauvaise utilisation des instructions `ENTRYPOINT` et `CMD` du Dockerfile.

En effet, pour définir le point d'entrée du conteneur, il existe deux syntaxes :

```dockerfile
# Syntaxe Shell
ENTRYPOINT npm start

# Syntax Exec
ENTRYPOINT ["npm", "start"]
```

Y a-t-il une syntaxe meilleure que l'autre ? Si oui, laquelle faut-il privilégier ? Pour répondre à ces questions, regardons de plus près les spécificités de chacune.

## Quelques Rappels

Avant de commencer, rappelons rapidement la différence entre les instructions `ENTRYPOINT` et `CMD`. En effet, ce sont ces deux instructions qui sont concernées par ces syntaxes (ainsi que l'instruction `RUN`, mais nous y reviendrons plus tard).

Tout d'abord, l'instruction `ENTRYPOINT` permet de définir le point d'entrée de notre conteneur, c'est-à-dire le programme à lancer au démarrage. L'instruction `CMD` permet quant à elle de définir des arguments par défaut au point d'entrée du conteneur, qui peuvent être facilement surchargés avec la commande `docker run`.

Le point d'entrée du conteneur correspond au processus principal qui y sera exécuté, c'est-à-dire celui de PID 1.

💡

Dans un environnement Unix, chaque processus peut avoir un ou plusieurs processus enfants, et chacun possède un seul processus parent. La seule exception est le tout premier processus, celui de PID 1 (également appelé _init_), qui n'a pas de parent. Tous les autres processus seront rattachés au processus _init_, directement ou indirectement.

Ce processus joue un rôle essentiel. En effet, l'une de ses missions est extrêmement importante : **transmettre les signaux reçus par le système d'exploitation à ses processus enfants**.

💡

Les signaux sont un moyen de communication entre les différents processus d'un système. C'est une sorte de notification envoyée par un processus à un autre pour lui _signaler_ l'apparition d'un événement. Par exemple, lorsque vous appuyez sur `CTRL+C` pour quitter un programme, le signal `SIGINT` est envoyé à ce dernier par votre shell. Le programme réagit à ce signal en s'arrêtant.

Dans le cas de Docker, les signaux importants sont `SIGINT`, `SIGTERM` et `SIGKILL`.

- `SIGINT` est une manière douce d'indiquer au programme qu'il doit s'arrêter.
    
- `SIGTERM` est une manière un peu plus brutale de lui dire de s'arrêter.
    
- `SIGKILL` permet de mettre fin au processus sans lui demander son avis.
    

Ces signaux sont envoyés au processus principal du conteneur. Les deux premiers signaux, `SIGINT` et `SIGTERM`, laissent le temps au processus d'exécuter des éventuelles fonctions de clean-up avant de s'arrêter. `SIGINT` est le signal envoyé lorsqu'un conteneur est démarré en mode intéractif et qu'on appuie sur `CTRL+C`. `SIGTERM` est le signal qui est envoyé au conteneur lorsque l'on exécute la commande `docker stop`.

Et qu'en est-il du signal `SIGKILL` ? Ce dernier est envoyé à l'exécution de la commande `docker kill`. Mais il est également envoyé **automatiquement** par Docker, 10 secondes après avoir executé `docker stop` si le processus ne s'est toujours par arrêté. On veut limiter le plus possible ce signal car il empêche le processus de lancer ces éventuelles fonctions de nettoyage, ce qui peut aboutir à une corruption des données.

💡

Exemples de fonctions de nettoyage : déconnexion de la base de données, _flush_ du contenu d'un fichier stocké en mémoire vers le disque, désallocation d'espace mémoire, ou encore débloquage d'un _lock_.

En situation normale, c'est-à-dire lorsqu'on utilise la bonne syntaxe _shell_ ou _exec_ et qu'on exécute la commande `docker stop` pour arrêter un conteneur, le processus principal reçoit ce signal, le gère et le transmet à ses enfants. Ses enfants répondent alors avec le signal `SIGCHLD`, et lorsque tous les enfants ont répondu, le processus principal s'arrête.

En revanche, lorsque l'on n'utilise pas la bonne syntaxe, c'est là que les problèmes surviennent. C'est notamment ce qui se passe lorsque le processus principal du conteneur reçoit le signal `SIGTERM`, **mais qu'il ne sait pas le gérer et qu'il ne le transmet pas à ses enfants**. C'est par exemple le cas des scripts _bash_, qui par défaut ne transmettent pas les signaux systèmes à leurs enfants (c'est-à-dire aux commandes qu'ils exécutent eux-mêmes). De ce fait, les enfants ne reçoivent jamais ces signaux, et ne s'arrêtent donc jamais car personne ne leur a dit de le faire. C'est pour cela que Docker est obligé d'envoyer lui-même le signal `SIGKILL` afin de forcer l'arrêt du conteneur.

## Impacts d'une Mauvaise Utilisation

C'est bien beau tout ça, mais quels sont les réels problèmes d'une mauvaise utilisation de la propagation des signaux ?

Tout d'abord, comme nous l'avons déjà vu dans la partie précédente, cela empêche le programme d'exécuter les fonctions de clean-up qu'il pourrait avoir besoin de lancer. Ces fonctions peuvent être primordiales, et si elles ne s'exécutent pas, cela peut amener à une altération des données (contenu stocké en RAM non vidé vers le disque) ou à un bloquage de l'exécution d'autres applications (à cause d'un _lock_ qui n'aura pas été débloqué par le programme).

En plus de cela, il est particulièrement ennuyant de devoir attendre 10 secondes (le temps que Docker envoie le signal `SIGKILL` au conteneur) après avoir exécuté `docker stop` pour que le conteneur s'arrête. C'est d'autant plus énervant de ne pas pouvoir arrêter un conteneur avec `CTRL+C` est de devoir le faire en passant par un autre terminal.

Mais ce n'est pas tout, car dans un environnement **Kubernetes**, le comportement par défaut n'est pas d'attendre 10 secondes, mais **30 secondes** ! Si l'application met quelques secondes à s'arrêter en temps normal, cela revient à durer presque 10 fois plus longtemps par rapport à une bonne gestion des signaux. **Pour rien**.

💡

Imaginez que vous avez un déploiement Kubernetes de 5 pods. Vous déployez une nouvelle version en utilisant la stratégie _RollingUpdate_, et les nouveaux pods se créent un par un tandis que les pods actuels s'arrêtent un par un. Avec une bonne gestion des signaux, la nouvelle version sera effective au bout de quelques secondes. En revanche, avec une mauvaise gestion des signaux, il faudra environ 2 minutes et 30 secondes.

Ceci étant dit, nous pouvons maintenant nous intéresser aux différences entre les syntaxes _shell_ et _exec_.

## Syntaxe Shell

Cette syntaxe est la plus lisible des deux. Pour une application Node que l'on démarre avec la commande `npm start`, on l'utilise de cette façon :

```dockerfile
ENTRYPOINT npm start
```

C'est la plus lisible et la plus intuitive, oui. Mais...

Si l'on se réfère à la documentation de Docker sur l'instruction `ENTRYPOINT`, voici ce que l'on apprend :

> The _shell_ form prevents any `CMD` or `run` command line arguments from being used, but has the disadvantage that your `ENTRYPOINT` will be started as a subcommand of `/bin/sh -c`, which does not pass signals. This means that the executable will not be the container's `PID 1` - and will _not_ receive Unix signals - so your executable will not receive a `SIGTERM` from `docker stop <container>`.

Tout d'abord, on nous explique qu'en utilisant la syntaxe _shell_ pour l'instruction `ENTRYPOINT`, cela empêche les arguments passés à l'instruction `CMD` (Dockerfile) ou à `run` (`docker run`) d'être utilisés. Dans la plupart des cas, ce n'est pas le comportement que l'on souhaite, car c'est là tout leur intérêt : pouvoir passer des arguments par défaut au point d'entrée de notre conteneur, que l'on peut surcharger facilement si besoin.

Mais on y apprend surtout que la commande que l'on renseigne à **l'instruction** `ENTRYPOINT` **ne sera pas exécutée comme programme principal du conteneur**, mais comme une sous-commande de `/bin/sh -c`, **qui ne transmet pas les signaux !** On retrouve donc le problème que nous avons vu dans la première partie de cet article : notre programme ne recevra jamais de signaux, et sera terminé de manière forcée.

Pour illustrer tout ça, je vous propose une petite démo. J'ai écrit un programme très simple en Bash, `time.sh`, qui affiche toutes les secondes `Hello World!` suivi du nombre d'itérations.

```bash
#!/bin/bash

trap "echo Bye; exit 0" SIGINT SIGTERM

count=0
while true; do
  count=$((count+1))
  echo "Hello world! $count"
  sleep 1
done
```

Mon programme ne casse pas trois pattes à un canard, certes. Notez cependant que j'ai ajouté une fonction permettant de gérer les signaux `SIGINT` (`CTRL+C`) et `SIGTERM` (`docker stop`) en affichant le message `Bye`, avant de quitter le programme. Cette instruction est importante, nous y reviendrons dans quelques instants.

Je peux démarrer mon programme en local, et le stopper au bout de quelques secondes en appuyant sur `CTRL+C`.

Tout va pour le mieux dans le meilleur des mondes, je suis vraiment un développeur très talentueux. Je suis désormais prêt à conteneuriser mon script. Pour cela, je commence par rédiger le Dockerfile :

```dockerfile

FROM ubuntu:23.10

COPY time.sh time.sh

ENTRYPOINT ./time.sh
```

Toujours rien d'impressionnant, le strict minimum permettant de conteneuriser mon script. L'instruction `ENTRYPOINT` utilise la syntax _shell_ car c'est plus agréable à lire. Je peux maintenant construire l'image et démarrer un conteneur :

```bash
docker build . -t time-shell
docker run --rm time-shell
```

Après quelques secondes, je décide de stopper mon conteneur avec `CTRL+C` comme je l'ai fait précédemment. Enfin, j'essaie...

![Tentative d'arrêt du conteneur avec CTRL+C](https://cdn.hashnode.com/res/hashnode/image/upload/v1694863561327/04309d6a-2f02-4019-bcd7-e5cc3d02cd5c.gif)

Même problème avec la commande `docker stop`.

![Tentative d'arrêt du conteneur avec docker stop](https://cdn.hashnode.com/res/hashnode/image/upload/v1694863752559/bfeff4fe-42aa-410d-a437-19aafd97095d.gif)

**Que se passe-t-il ?** 🤯

En appuyant sur `CTRL+C`, le signal `SIGINT` est envoyé au processus principal du conteneur. Pareil pour le signal `SIGTERM` avec la commande `docker stop`. Cependant, `/bin/sh` (processus principal du conteneur) **ne gère pas ce signal** **et ne le propage donc pas à ses processus enfants** (en l'occurrence, mon script _bash_). Ce processus enfant continue de s'exécuter car il n'a pas reçu le signal que devait lui transmettre son parent, qui continue également de s'exécuter.

Avec la commande `docker stop`, le signal `SIGTERM` est envoyé. Ce dernier n'est pas transmis non plus au script _bash_, qui continue donc de s'exécuter. Au bout de 10 secondes, Docker détecte que le conteneur ne s'est toujours pas terminé (car le processus de PID 1 s'exécute toujours), et force alors son arrêt avec le signal `SIGKILL`.

## Syntaxe Exec

La syntaxe _exec_ a un comportement différent de la syntaxe _shell_. Toujours dans le cas d'une application Node démarrée avec la commande `npm start`, elle s'utilise de la façon suivante :

```dockerfile
ENTRYPOINT ["npm", "start"]
```

La commande à exécuter ainsi que ses arguments doivent être fournis sous forme d'une liste JSON. Pour savoir si l'utilisation que vous en faites est correcte, dites-vous que **chaque élément entre guillemets de la liste apparaîtra également avec des guillemets dans la commande finale**. Par exemple, si l'on considère cet `ENTRYPOINT` :

```dockerfile
ENTRYPOINT ["ls", "-al", "$HOME"]
```

La commande qui sera exécutée correspondra à ça :

```bash
'ls' '-al' '$HOME'
```

💡

Les guillements simples sont importantes, car avec la syntaxe _exec_, les variables ne sont pas interpétées.

Il n'est donc pas possible de fournir votre commande entière comme seul élément de l'instruction `ENTRYPOINT`, bande de petits malins 😉. Pour vous en convaincre, vous pouvez essayer d'utiliser l'`ENTRYPOINT` suivant :

```dockerfile
ENTRYPOINT ["ls -al $HOME"]
```

Et vous constaterez que ça ne fonctionne pas, car la commande exécutée sera la suivante (qui n'existe pas dans votre `$PATH`) :

```bash
'ls -al $HOME'
```

💡

Vous pouvez également essayer de la lancer dans votre terminal pour constater l'erreur générée : `command not found`.

Contrairement à la syntaxe _shell_, la syntaxe _exec_ n'invoque pas de shell. Comme nous venons de la voir avec l'exemple précédent, cela signifie qu'aucune interprétation n'a lieu, comme la substitution de variables ou l'exécution de commandes. Par exemple, la redirection des sorties n'est pas possible non plus, et cet `ENTRYPOINT` est donc invalide :

```dockerfile
ENTRYPOINT ["echo", "hello", ">", "out.txt"]
```

💡

Là encore, vous pouvez vous en convaincre entre exécutant cette commande dans votre terminal : `'echo' 'hello' '>' 'out.txt'`.

Mais cela signifie **surtout** que la commande renseignée sera exécutée **comme processus principal du conteneur** ! Et donc que les signaux qui seront envoyés au conteneur seront reçus par notre programme.

Reprenons l'exemple de la partie précédente. Le programme `time.sh` reste inchangé, mais le Dockerfile va subir une légère transformation, subtile mais majeure :

```dockerfile
FROM ubuntu:23.10

COPY time.sh time.sh

ENTRYPOINT ["./time.sh"]
```

L'instruction `ENTRYPOINT` utilise désormais la syntaxe _exec_. On peut alors effectuer à nouveaux les tests d'arrêt, d'abord avec `CTRL+C` :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694872601512/facdc25f-687c-4b36-b1ca-0d0ace3e91f5.gif)

Puis avec `docker stop` :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1694872613094/a2bdd311-7f75-4b1b-9ed0-8539ff016142.gif)

Le résultat est sans appel : **l'arrêt du conteneur est désormais immédiat.**

Il reste cependant un cas où le conteneur pourrait prendre 10 secondes à s'arrêter avec `docker stop`, même en utilisant la syntaxe _exec_ : si le programme ne gère pas le signal `SIGTERM`.

Dans les exemples ci-dessus, le signal `SIGTERM` était géré correctement, et donc tout s'est bien passé lorsqu'on a arrêté le conteneur. Mais si votre programme n'a pas défini de handler pour ce signal, le problème des 10 secondes surviendra à nouveau. De la même façon, si aucun handler n'est défini pour le signal `SIGINT`, il ne sera pas possible de l'arrêter en appuyant sur `CTRL+C`.

Pour solutionner ce problème, il suffit donc de définir des handlers pour gérer ces signaux de terminaison comme je l'ai fait avec la commande `trap`.

🚨

Attention à bien les transmettre aux processus enfants s'il y en a, sinon ils seront tués de manière brutale, bande de barbares ⚔️.

## Quelle Syntaxe Utiliser ?

La **recommandation de la documentation officielle** de Docker est **d'utiliser la syntaxe _exec_ par défaut**. Dans la plupart des cas, c'est cette syntaxe que l'on souhaitera adopter pour éviter les problèmes que nous avons évoqués. En effet, avec la syntaxe _shell_, il n'est pas possible de fournir des arguments par défaut au programme principal, et les signaux ne lui sont pas transmis.

Si vous souhaitez utiliser la syntaxe _shell_, c'est que vous avez une bonne raison de le faire et que vous savez ce que vous faites. Dans la grande majorité des cas, la syntaxe _exec_ sera la bonne.

💡

Les syntaxes _shell_ et _exec_ s'appliquent aux instructions `ENTRYPOINT`, `CMD`, mais aussi `RUN`. Dans le cas de l'instruction `RUN`, on préfèrera la syntaxe _shell_ car elle est plus lisible, permet l'interprétation des variables et des commandes, et la gestion des signaux n'a aucune importance (car cette instruction est évaluée au moment de la construction de l'image, pas lors du démarrage du conteneur). La syntaxe _exec_ pourra tout de même être utilisée au besoin, par exemple pour utiliser un _shell_ différent de `/bin/sh`.

Dans le cas où vous utilisez l'instruction `CMD` pour founir des arguments par défaut à l'instruction `ENTRYPOINT`, ces deux instructions devront utiliser la syntaxe _exec_.

## Conclusion

Les syntaxes _shell_ et _exec_ n'ont désormais plus aucun secret pour vous. Vous êtes en mesure de les utiliser de la bonne façon en fonction des situations. Comme souvent, il n'y a pas de solution meilleure que l'autre, mais une solution qui est adaptée à chaque cas d'utilisation.

Pour conclure, je vous propose ce tableau récapitulatif de ces deux syntaxes :

||Syntaxe _shell_|Syntaxe _exec_|
|---|---|---|
|**Utilisation**|`ENTRYPOINT npm start`|`ENTRYPOINT ["npm", "start"]`|
|**Processus principal**|`/bin/sh -c "npm start"`|`npm start`|
|**Transmission des signaux**|❌|✅|
|**Interprétation du shell (variables, commandes)**|✅|❌|

---

J'espère vous avoir appris des choses au travers de cet article, et si vous avez des questions ou des remarques, n'hésitez-pas à vous exprimer dans l'espace commentaires ou à me contacter ! 💬