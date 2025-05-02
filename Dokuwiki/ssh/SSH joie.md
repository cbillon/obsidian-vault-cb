---
tags:
  - ssh
---

====== Amusons-nous dans la joie et l’allégresse ======

Ça c’est un peu la solution simple dans un monde simple. Par contre, si vous devez faire exécuter les commandes avec un autre utilisateur que celui avec lequel vous vous connectez, ça peut rapidement devenir un cauchemar. Par exemple, disons que vous travaillez localement avec un utilisateur userlocal, vous vous connectez avec un utilisateur habilité à se connecter user1, mais que sur cette machine, vous devez exécuter une commande avec les droits de l’utilisateur user2, lui ne pouvant être utilisé pour la connexion SSH. Bienvenue dans l’enfer des échappements, là où vont les mauvais sysadmins finissent leur vie. Voici quelques petits exemples facile à reproduire chez vous si vous avez au moins deux hôtes à dispo :

  ssh user1@host sudo -u user2 echo $USER
  > userlocal # Faute de débutant, le $USER est résolu par votre propre shell localement

  ssh user1@host sudo -u user2 'echo $USER'
  > user1 # Zut, le $USER est résolu par mon deuxième shell distant avant sudo

  ssh user1@host sudo -u user2 'echo \$USER'

  > $USER # La variable n'est même plus interprêtée. Elle est bien exécutée par user2 mais pas substituée. Il nous faut un shell !

  ssh user1@host sudo -u user2 sh -c 'echo $USER'
  >      # Résultat vide ! Si avec ça vous n'avez l'impression de reculer plutôt qu'avancer,
         # vous avez les nerfs solides ! C'est dû au fait que 'sh -c' prend une chaine de caractère
         # en argument, et non pas plusieurs. Rien à voir avec SSH, essayez sh -c 'echo test' en local, 
    vous verrez.
         # Vous allez me dire que c'en est une, mais vos simple quotes ici disparaissent une fois arrivés 
         # sur votre hôte distant. Il faut les échapper aussi localement !

  ssh user1@host sudo -u user2 sh -c 'echo $USER'
  > userlocal # Mouahah. C'est à ce moment là que vous vous demandez pourquoi vous faites pas de 
              # l'élèvage de brebis dans le Cantal plutôt. Oui, c'est à devenir chèvre... (pardon)

  ssh user1@host sudo -u user2 sh -c \'echo \$USER\'
  > user2 # Victoire ! Mais à quel prix... Sinon vous pouvez échapper plus globalement mais ça ne réduit 
  pas la 
          # complexité, même si vous allez sûrement préféré cette solution :
  ssh user1@host "sudo -u user2 sh -c 'echo \$USER'"
  > user2

Ok c’est un peu l’enfer, mais on se débrouille pour un simple test comme cela. Par contre, ça peut devenir carrément violent pour les neurones si on doit appliquer ça à un script plus touffu avec moult variables et caractères spéciaux. Si vous avez une solution bien plus simple qui respecte ce besoin, je suis intéressé par vos retours d’expérience !

Dans ce cas là je préconiserais plutôt une méthode abordée plus haut qui est de construire un script localement, l’envoyer (via scp) et l’exécuter. C’est bien plus simple à debugger et ça fait le taff. Par contre ça crée plusieurs connexions SSH à l’occasion, ce qui peut être un peu lourd, surtout sur les réseaux kerberisés.