---
tags:
  - linux
  - tools
---

=== Tmux ===

Les raccourcis :

[[https://github.com/gpakosz/.tmux|github du projet]]

[[https://tmuxcheatsheet.com/|commandes tmux]]


tmux est un multiplexeur de terminal, c'est à dire qu'il est capable de scinder plusieurs terminaux au sein d'un seul et même affichage. C'est un processus indépendant qui se lance dans un shell, et si votre terminal préféré plante, tmux conserve votre session et vous pouvez la réutiliser.

== Installation ==

Bon, j'imagine que vous vous dites que ça a l'air pas mal sur le papier, et vous aimeriez donc l'installer ! Allons-y !
Pour votre distribution GNU/Linux (ubuntu):

  sudo apt-get install tmux
  
Pour Windows via Bash on Windows :

  sudo apt-get install tmux
  
Pour macOS via brew:

  brew install tmux
  
== Utilisation ==

Après votre installation, vous pouvez ouvrir votre terminal préféré et lancer la commande

tmux

Image montrant tmux avec une séparation verticale de la fenêtre

Pour ceux qui utilisent ohmyzsh, vous pouvez adopter la configuration suivante qui lancera tmux au démarrage. Il suffit d'ajouter tmuxdans les plugins et d'ajouter la variable ZSH_TMUX_AUTOSTART à true

  ZSH_TMUX_AUTOSTART='true'
  plugins=(
  ...
  tmux
  ...
  )
  
Si vous quittez votre terminal et que vous souhaitez retrouver votre session tmux, rien de plus simple !

  tmux attach
  
Vous avez ouvert votre première session tmux ! Il y a maintenant plusieurs notions à appréhender :

  * session est un ensemble de window, de plus, quand on ferme le terminal, la session n'est pas supprimée, ce qui permettra de la reprendre en cas de besoin ;
  * window, ou fenêtre, est ce que vous voyez à l'écran, elle est un ensemble de panes;
pane, ou carreau, est une partie d'une fenêtre quand on la découpe, en la scindant horizontalement ou verticalement.

== Commandes basiques ==

L'ensemble des raccourcis tmux débutent par une combinaison de touches spécifique <CTRL>+B, je ne suis pas très fan de cette combinaison par défaut (pas très pratique pour l'exécuter à une main). Si vous voulez la modifier, vous éditez la configuration de tmux qui se trouve dans un fichier nommé .tmux.conf. Si ce fichier n'existe pas, créez-le à la racine de votre répertoire utilisateur.

Dans ma configuration ci-dessous, j'utilise la combinaison <CTRL>+q, <CTRL> étant représenté par C, mais vous pouvez utiliser n'importe quelle touche. Enfin, sur la seconde ligne, on dissocie la précédente combinaison.

  set -g prefix C-q
  unbind C-b
  
Une fois dans tmux, et en effectuant la combinaison <CTRL>+q, vous serez capable d'effectuer ces quelques actions :

, : Renommer le titre de sa session
c : Créer une nouvelle fenêtre dans la session tmux active
n : Basculer entre les différentes fenêtres de la session
X : Choisir une fenêtre spécifique (ou X est le numéro de la fenêtre)
w : Afficher la liste des fenêtres disponibles
t : Afficher l’heure dans une fenêtre
& : Supprimer la fenêtre courante

Passons aux choses sérieuses, le but étant de scinder sa fenêtre avec plusieurs panes à l'écran :

% : Scinder verticalement
" : Scinder horizontalement
ESPACE : Changer l'affichage courant
o : Basculer d'un pane à l'autre
; : Basculer au dernier pane utilisé
q : Afficher le numéro des panes durant un court instant
x : Supprimer le pane courant

Par défaut, pour naviguer dans votre fenêtre et sur vos différents panes, vous devez exécuter la combinaison de touches et appuyer sur la flèche directionnelle pour accéder au terminal de votre choix. Je ne trouve pas cela très pratique, et je vous invite à modifier votre configuration (.tmux.conf) pour y insérer :

bind -n M-Left select-pane -L
bind -n M-Right select-pane -R
bind -n M-Up select-pane -U
bind -n M-Down select-pane -D

Vous pouvez maintenant naviguer dans vos terminaux en ayant la touche <ALT> (le Mdans la configuration correspond à cette touche) enfoncée et en utilisant une flèche directionnelle.

Pour aller plus loin

== La gestion de la souris ==

Il est possible d'activer la souris dans les panes pour activer le défilement et remonter l'historique d'un shell, pour cela, il suffit d'ajouter dans le fichier .tmux.conf :

  set -g mouse on
  
== Le copier coller via la sélection de texte ==

Pour macOS
Il est nécessaire d'installer un paquet (ici via brew) pour faire communiquer le presse papier système avec celui de tmux

brew install reattach-to-user-namespace
et ajouter ceci dans notre configuration .tmux.conf

bind-key -T copy-mode MouseDragEnd1Pane send -X copy-pipe-and-cancel "pbcopy"
Pour GNU/Linux (et Bash on Windows)

Vérifier l'installation du paquet xclip par rapport à votre distribution et ajouter dans le .tmux.conf

bind-key -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel "xclip -se c -i"

Éditer sa configuration tmux à la volée
Si vous souhaitez éditer et recharger la configuration de tmux en liant des touches, vous pouvez faire comme ci-dessous où la touche M (liée à la combinaison de touches tmux) est liée à l'édition du fichier .tmux.conf et la touche r permet de recharger la configuration et d'afficher le message "Configuration rechargée" dans la barre du bas.

bind-key M split-window -h "vim ~/.tmux.conf"
bind-key r source-file ~/.tmux.conf \; display-message "Configuration rechargée..."
Écrire sur l’ensemble des panes d’une fenêtre
Quand on crée plusieurs panes afin d’ouvrir plusieurs accès SSH sur différentes machines et que l’on souhaite effectuer les mêmes actions, il y a une option bien pratique dans tmux.

Pour cela il suffit d’effectuer la combinaison de touche <CTRL>+q et de taper :setw synchronize-panes on, vous pouvez maintenant écrire vos commandes sur l’ensemble des panes. Passez l’option à off quand vous souhaitez arrêter la synchronisation des panes.

== Personnaliser la barre de tmux ==

Il existe beaucoup d'options pour personnaliser la barre accompagnant tmux, voici un extrait de ma configuration, toujours dans le .tmux.conf, avec le détail :

set -g status-position bottom
set -g status-bg colour237
set -g status-fg colour137
set -g status-right-length 100
set -g status-left-length 70
status-position : position de la barre par rapport au terminal
status-bg : couleur associée au background
status-fg : couleur associée au foreground
status-right-length : longueur de l'affichage à droite
status-left-length : longueur de l'affichage à gauche

Conclusion

tmux c’est la vie ! Quand vous commencez à l’utiliser, vous ne pouvez plus vous en passer !

J’espère que cet article vous aura permis de mettre les mains dedans en ayant un premier aperçu fonctionnel de l’outil. Il existe beaucoup de tutoriels pour le personnaliser au maximum, je vous laisse entre les mains d’internet pour poursuivre votre initiation !

