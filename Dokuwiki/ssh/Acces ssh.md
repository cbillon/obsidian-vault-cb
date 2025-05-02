---
tags:
  - ssh
---

====== Un accès SSH pour les contrôler tous ======

Essayons d’aller du cas le plus simple au plus complexe. Première chose à garder en tête: tout caractère spécial des commandes que vous exécutez dans votre shell local sera interprêté, c’est très important de garder ça en tête à chaque moment. Donc bien sûr si vous écrivez :

  ssh user@host echo $USER
  
Cela vous donnera l’utilisateur local et non distant car votre shell local substituera la variable avant même de l’avoir envoyée sur votre hôte distant. Seule solution, échapper ! On échappe habituellement par des single quotes (’) mais on peut également utiliser le backslash. Je recommande tout de même la solution du single quote car elle résoud pas mal d’autres problèmes d’un coup

  # Meh...
  ssh user@host echo \$USER
  # Mieux !
  ssh user@host 'echo $USER'

Ok, facile. Maintenant, comment faire pour exécuter plusieurs commandes sur un hôte distant de façon propre et dans un ordre précis ? Si vous avez tout de suite pensé Ansible/SaltStack/Chef/Puppet (rayez la mention inutile), alors vous avez parfaitement raison mais ce n’est pas le sujet d’aujourd’hui :D

On peut en effet séparer les commandes par des point-virgules (;) ou des double esperluettes (&&) mais ça peut vite devenir fouilli, à garder seulement si la taille de la ligne résultante reste raisonnable. On peut également écrire un script localement au besoin, l’envoyer puis l’exécuter à distance. Ce n’est pas vraiment problématique en soi, c’est facile et rapide à comprendre, mais ça reste tout de même laborieux et un peu moche, avouons-le (et ça exécute bien plus de commandes que nécessaires).

La méthode préférée sera alors le Here-Document, fonctionnalité merveilleuse de bash s’il en est.

  ssh user@host -T << "EOF"
    var=text
    commande2 $var
    commande3 || commande4
  EOF
  
Le -T est juste là pour éviter le warning qui va popper irrémédiablement vu qu’aucun terminal n’est alloué au stdin de ssh de cette façon. Si vous ne connaissez pas le Here-Doc, le mot de fin, ici EOF est spécifié en argument après les doubles chevrons pour indiquer que tout le document entre la deuxième ligne et le même mot (EOF) sera envoyé à l’entrée standard de la commande appelante, ici ssh.

Attention, j’ai utilisé ici “EOF” et non pas EOF, ce qui fait une grande différence. Entouré de double quotes, “EOF” indique à bash de ne pas interprêter les variables et certains caractères spéciaux du document utilisé. L’inverse peut-être voulu et est très utile par exemple pour créer un fichier selon les valeurs de variables de l’environnement courant :

  TEMP=$(mktemp) # Je crée un fichier temporaire
  cat > $TEMP << EOF
  # fichier généré le $(date +%F)
  [env]
  user=$USER
  shell=$SHELL
  EOF

C’est un exemple très simpliste qui fonctionnera avec tout le monde mais avec de vraies données ça peut être très robuste. Donc on ignore les double quotes si on veut interpréter les variables, et on les ajoute si on veut que les “$” restent tel quels.