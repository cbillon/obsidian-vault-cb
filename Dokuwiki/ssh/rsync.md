---
tags:
  - rsync
---

====== RSYNC SSH ======

La syntaxe de connexion  rsync (options] <source> <destination>

Cas usage 1 : envoi d'un fichier pdf sur VM

Le fichier pdf est récupéré sur la machine locale

On doit l'envoyer dans le répertoire **/srv/dokuwiki/data/media**

Il sera accessible sur dokuwiki en utilisant la commande **{{:your.pdf|}}**
 

===== Commande rsync =====

  rsync -azv ~/Téléchargements/mon.pdf  dokuwiki:/srv/dokuwiki/data/media
