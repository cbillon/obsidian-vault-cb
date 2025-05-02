---
link: https://www.vrchr.fr/posts/2023/2023-12-18-demo-magic
excerpt: Minimisez l'effet démo avec les outils demo-magic et asciinema !
slurped: 2024-05-13T13:35:14.738Z
title: Minimisez l'effet démo avec demo-magic et asciinema !
---

Super, j'ai été sélectionné pour présenter un sujet à l'[Open Source Experience 2023](https://www.opensource-experience.com/) ! Je connais mon sujet, l'ai déjà présenté au meetup DevOps Aix-Marseille, ça devrait aller... Ah, mais la présentation ne dure que 20 minutes, et je vois dans la FAQ des speakers qu'il n'y a pas de micro cravates, et donc potentiellement pas les 2 mains libres pour faire les 2 démos, ça va être sport!

Un coup de stress, je dois drastiquement réduire le temps de ma présentation. Pour la partie théorique ça peut aller, je vais zapper des choses pas forcément essentielles, mais j'ai toujours mes démos à faire, et ça je ne peux pas faire l'impasse, mon sujet c'est justement la démo de 2 outils, et le moindre grain de sable peut faire exploser le timing déjà très court !

Je me rappelle alors du [Devoxx France 2022](https://www.vrchr.fr/posts/2022/2022-04-21-devoxxfr-jeudi/) où [Antoine Ceol](https://twitter.com/boombaprealm) avait utilisé une application pour sa démo sur [GIT](https://youtu.be/QCsFFAc5ffc). Pour une fois que mon blog sert :P

Je vais alors vous présenter 2 outils pour accélérer vos démos, et minimiser les erreurs pendant vos présentations.

## Demo-magic

[Demo-magic](https://github.com/paxtonhare/demo-magic) est un script shell assez simple qui vous permet d'automatiser la frappe au clavier de commandes, un peu comme du playback. Le concept est simple, on écrit les commandes à l'avance, et lorsque vient le temps de la démo, on appelle `demo-magic` qui fera le travail presque à votre place.

### Installation

En pré-requis, il faut installer la commande `pv` pour simuler la tape au clavier (ici installation sous Debian). Ensuite, il vous faut récupérer le [script](https://raw.githubusercontent.com/paxtonhare/demo-magic/master/demo-magic.sh) :

```
1$ sudo apt install pv
2$ curl -LO https://raw.githubusercontent.com/paxtonhare/demo-magic/master/demo-magic.sh
```

Puis, il suffit de "sourcer" ce script dans votre propre script / prompt bash, qui contiendra les commandes à exécuter / simuler :

```
1#!/bin/bash
2########################
3# include the magic
4# https://github.com/paxtonhare/demo-magic
5########################
6. demo-magic.sh
7
8p "apt install cowsay"
```

### Utilisation

Une fois l'outil inclus, vous pouvez écrire les différentes étapes de votre démo. Le script propose plusieurs fonctions :

- `p` : Affiche simplement la commande, sans l'exécuter. Il faudra appuyer sur `Entrée` pour l'affichage
- `pe` : Affiche et exécute la commande. Il faudra appuyer sur `Entrée` pour l'exécution
- `pei` : Affiche et exécute la commande immédiatement.

Vu la complexité du script, vous pouvez l'étendre sans trop de problèmes ;)

D'ailleurs, j'ai rajouté de mon côté `pi`, je vous laisse deviner sa fonction.

À l'appel du script, il est possible également de spécifier 2-3 paramètres comme le fait d'attendre entre les actions ou pas.

Voici ci-après un exemple d'utilisation :

```
 1p "# Clone, Python venv and install requirements"
 2pi "python3 -m venv krr-env"
 3pei "source krr-env/bin/activate"
 4
 5# Clone & Install Krr
 6pi "git clone https://github.com/robusta-dev/krr"
 7pei "cd krr"
 8pi "pip install -r requirements.txt"
 9
10# Use Krr
11pe "python krr.py simple -n kube-system"
```

### Paramétrage

Il est possible de "customiser" le script via quelques variables d'environnement, comme la vitesse d'écriture, la couleur du prompt, etc. Il faut simplement surcharger les variables :

```
1# Set demo-magic options
2TYPE_SPEED=50 # Accelerate typing
3DEMO_CMD_COLOR="" # No bold
4DEMO_PROMPT="${PURPLE}$ ${COLOR_RESET}"
5DEMO_COMMENT_COLOR=$CYAN
```

Et voilà, il n'y a plus qu'à lancer votre démo, et par magie tout s'exécute (presque) tout seul ! Voici le résultat de cette [démo](https://gitlab.com/opsrel.io/requests-limits-demo/-/blob/d3286c8260936dccab6103eb2d9d1f99b8d9448c/requests-limits-demo-krr.sh) :

Super, j'ai "automatisé" ma démo, mais je ne suis pas serein à 100%... Si jamais j'ai un problème de réseau, que ma plateforme est en vrac, je peux scripter autant que je veux mes actions, ça ne marchera pas ! Vient alors le deuxième outil à la rescousse : asciinema !

## Asciinema

[Asciinema](https://asciinema.org/) est un logiciel super pratique, qui va enregistrer ce qui se passe dans votre terminal. Vous pourrez ensuite sauvegarder et rejouer l'enregistrement en local, ou bien le télécharger et le partager en ligne sur le site [https://asciinema.org/](https://asciinema.org/).

### Installation

Toujours sous Debian, un coup d'apt et c'est réglé. On peut ensuite s'authentifier pour pouvoir télécharger le résultat pour partage.

```
1$ sudo apt install asciinema
2$ asciinema auth
3Open the following URL in a web browser to link your install ID with your asciinema.org user account:
4
5https://asciinema.org/connect/xxxx-xx-xxx-xxxx
6
7This will associate all recordings uploaded from this machine (past and future ones) to your account, and allow you to manage them (change title/theme, delete) at asciinema.org.
```

### Utilisation

Plusieurs arguments possibles :

- `rec` : enregistre la session du terminal
- `play` : joue une session enregistrée précédemment
- `cat` : affiche le contenu du fichier enregistré
- `upload` : télécharge sur le site la session enregistrée

Dans notre cas, on va enregistrer l'exécution du script (ici `bonjour.sh`)

```
1❯ asciinema rec -c ./bonjour.sh
2asciinema: recording asciicast to /tmp/tmp00kyltyr-ascii.cast
3asciinema: exit opened program when you're done
4Bonjour !
5asciinema: recording finished
6asciinema: press <enter> to upload to asciinema.org, <ctrl-c> to save locally
7asciinema: asciicast saved to /tmp/tmp00kyltyr-ascii.cast
8~
```

Vous pouvez ensuite rejouer la commande avec `asciinema play`.

On retrouvera alors des exemples de ma démo super rapide ici (je ne sais pas jusqu'à quand elles seront en ligne, les enregistrements ne restant pas indéfiniment sur le site) :

- [Krr demo](https://asciinema.org/a/DJPEsCuSf5yNJnyZfQHnyb0gI)
- [Goldilocks demo](https://asciinema.org/a/GKNmnzPTjpvjoCbc6UlAQnnx5)

Vous aurez remarqué que la vidéo du paragraphe précédent est de l'asciinema, non ?

## Conclusion

Et voilà ! On a maintenant de quoi jouer notre démo sans accrocs :

- Une démo automatisée avec `demo-magic`
- Une démo enregistrée avec `asciinema`

Et pour l'anecdote, mes démos se sont très bien passées, dans les temps ! Et je ne suis pas le seul à avoir du faire un travail de synthèse pour l'OXSP, 20 minutes c'est vraiment court ;)