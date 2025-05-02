---
link: https://moodle.org/mod/forum/discuss.php?d=461215
excerpt: Bonjour,
tags:
  - moodle
  - plugin
  - install
slurped: 2025-04-03T10:18
title: "Moodle en français: Installation multiple de plugins tiers | Moodle.org"
---

- [◄ [Devoirs] Travail envoyé mais resté en brouillon](https://moodle.org/mod/forum/discuss.php?d=462232)
- [Contact et achèvement d'activités par groupement ►](https://moodle.org/mod/forum/discuss.php?d=462080)

Bonjour,

Quelqu'un a-t-il une solution simple, voir un plugin spécifique, permettant d'installer d'un seul coup un "wagon" de plugins tiers sur une plateforme en construction ?

Une solution interne à Moodle, et non par un accès direct aux dossiers de l'arborescence fichiers ([modules](https://moodle.org/mod/glossary/showentry.php?eid=10247&displayformat=dictionary "Terminologie francophone de Moodle : Modules"), [blocs](https://moodle.org/mod/glossary/showentry.php?eid=4071&displayformat=dictionary "Terminologie francophone de Moodle : Blocs"),local...). Que je saurais faire par ailleurs,mais qui ne me convient pas dans le cas présent.

Merci d'avance et bonne fin de journée !

Daniel  

Moyenne des évaluations  -

"voir**e** un plugin spécifique".  
Excusez l'erreur...

Moyenne des évaluations  -

Bonjour Daniel,  
Je n'ai jamais trouvé de solution toute faite à ce sujet.  
Je suppose que ce qui te pose problème, c'est de [déposer](https://moodle.org/mod/glossary/showentry.php?eid=376&displayformat=dictionary "Terminologie francophone de Moodle : Déposer") chaque plugin tiers dans le bon sous dossier de Moodle.  
La seule solution que je vois si tu veux installer les mêmes plugins tiers qu'un Moodle existant, c'est de copier sur ton ordinateur le dossier complet de ton Moodle qui contient les plugins tiers et de copier ailleurs sur ton ordinateur la même version de Moodle téléchargée directement depuis moodle.org (autrement dit sans plugins tiers).  
Ensuite, tu utilises un programme qui recherche et supprime les doublons. Et tu supprimes d' un seul coup tous les fichiers du Moodle de base dans ton dossier qui contient le Moodle complet.  
Ainsi, il ne te reste que les différents plugins tiers avec l'arborescence. En une seule manip, tu n'as plus qu'à copier toute l'arborescence dans ton nouveau Moodle tout propre. Et tu te retrouves avec tous les plugins installés en une seule fois.  
Je n'ai pas trouvé plus simple.  
Jean-Gabriel

Moyenne des évaluations  -

Merci Jean-Gabriel,

J'espérais un plugin pour cela.

Mais effectivement, je procède à peu près ainsi:

1. Ayant une liste complète de plugins préalablement téléchargés, je les classe par [catégories](https://moodle.org/mod/glossary/showentry.php?eid=342&displayformat=dictionary "Terminologie francophone de Moodle : Catégories") dans les dossiers (modes, blocks, local...)
2. Puis, avec mon [gestionnaire](https://moodle.org/mod/glossary/showentry.php?eid=11208&displayformat=dictionary "Terminologie francophone de Moodle : Gestionnaire") de fichiers de cPanel, je dépose ces dossiers compressés dans le dossier désiré et je décompresse le dossier, puis chacun des plugins. Heureusement c'est assez rapide.
3. Enfin je rafraîchi ma page Moodle et la procédure d'installation débute pour le lot concerné.
4. Par précaution, je ne procède que par lot à chaque fois. Par exemple, les blocs ensemble. Etc...

Il est dommage que lors de l'installation de plugins préalablement téléchargés, on ne puisse pas déposer l'ensemble des fichiers au lieu de un à un. Un peu comme dans la [ressource](https://moodle.org/mod/glossary/showentry.php?eid=3921&displayformat=dictionary "Terminologie francophone de Moodle : Ressource") dossier par exemple... 

Bonne fin de journée

Daniel

Moyenne des évaluations  -

Bonjour Daniel,

Je l'indique ici parce que la mention « solution simple » est très relative d'un individu à l'autre …

Il existe une commande [MOOSH](https://moosh-online.com/commands/) pour cela : 

moosh plugin-install --release 20160101 mod_quickmail

À bientôt,  
Patrick

Moyenne des évaluations Utile (1)

Bonjour Daniel,

Sinon, dans la [catégorie](https://moodle.org/mod/glossary/showentry.php?eid=342&displayformat=dictionary "Terminologie francophone de Moodle : Catégorie") "combine à Nanard" (toujours dans le cas où tu ne peux pas accéder à ton répertoire d'installation via FTP ou autre), tu peux toujours gagner quelques clics en ouvrant plusieurs onglets sur la page _Installer des plugins_ ([admin](https://moodle.org/mod/glossary/showentry.php?eid=4734&displayformat=dictionary "Terminologie francophone de Moodle : Admin")/tool/installaddon/index.php) et charger un fichier zip par [onglet](https://moodle.org/mod/glossary/showentry.php?eid=11184&displayformat=dictionary "Terminologie francophone de Moodle : Onglet")...

Ça évite de passer à chaque fois par la page _Information sur la version actuelle_ (admin/index.php?cache=0&confirmplugincheck=0).  
Tu cliques ensuite sur le bouton _Continuer,_ seulement une fois tous les plugins chargés...

On est d'accord, ça vaut ce que ça vaut, mais c'est toujours ça de pris...

Stéphane

Moyenne des évaluations  -

Bonjour Stéphane,

J'utilise évidemment systématiquement ta première solution, avec un nombre important d'onglets ouverts, que je [parcours](https://moodle.org/mod/glossary/showentry.php?eid=12647&displayformat=dictionary "Terminologie francophone de Moodle : Parcours") en boucle jusqu'à la fin.  
On peut aussi déposer tous ces plugins, classés et compressés par catégorie, dans les dossiers correspondants de l'arborescence de  l'hébergement, puis les décompresser.L'installation se déroule alors globalement.Mais je n'aime pas trop cette méthode.

Je préfère contrôler la situation plugin par plugin. Pour le cas ou l'on d'eux produirait une erreur.

L'idéal serait que lors du dépôt du fichier zip de la page "installer un plugin", on puisse, comme dans l'[activité](https://moodle.org/mod/glossary/showentry.php?eid=340&displayformat=dictionary "Terminologie francophone de Moodle : Activité") dossier, [intégrer](https://moodle.org/mod/glossary/showentry.php?eid=10130&displayformat=dictionary "Terminologie francophone de Moodle : Intégrer") toute une palanquée de plugins et lancer l'installation en une seule fois.

Car le faire un à un est vraiment un peu archaïque.

Daniel  

Moyenne des évaluations  -

Bonjour Daniel,

Il pourrait effectivement être intéressant de suggérer une amélioration, afin de permettre de déposer d'un coup plusieurs fichier ZIP de plugins, qui auraient été préalablement téléchargés localement, afin de les installer d'un coup sur une instance.

Séverin

Moyenne des évaluations Utile (2)

Bonjour Séverin,  
Comme je ne sais pas faire ce genre de suggestion, j'espère qu'un(e) moodleu(se) jeune, dynamique et expérimenté(e), ce que je ne suis plus guère...,  proposera cette amélioration? 😉

Daniel  

Moyenne des évaluations  -

Bonjour Daniel,

Le problème : il manque à la liste "ayant du temps disponible" afin de bien formuler la demande...

Séverin

Moyenne des évaluations  -

Bonjour Séverin,  
Pas compris 🤔

Moyenne des évaluations  -

Bonjour Daniel,

Je voulais signifier que je manquais de temps pour m'en occuper correctement.

Par contre, Frédéric a indiqué [dans ce message](https://moodle.org/mod/forum/discuss.php?d=457869#p1855543) un plugin permettant de copier les plugins non standards d'un Moodle à un autre. Cela pourrait être une solution assez simple lorsque les mêmes plugins sont utilisés sur plusieurs Moodle (de version identique).

Séverin

Moyenne des évaluations Utile (1)

Bonjour Daniel et Séverin,  
Il y a encore **git**plugin [https://moodle.org/plugins/view.php?id=1963](https://moodle.org/plugins/view.php?id=1963)  
_The gitplugins.conf file is a list of all the plugins you want to manage (install, upgrade...)_

Moyenne des évaluations  -

Merci Alexandre,  
Cela semble assez intéressant en effet.  
Mais la mise en place et l'utilisation n'en semble pas simple et immédiate.  
Prudent, il faudrait que je teste cela sur une plateforme de [test](https://moodle.org/mod/glossary/showentry.php?eid=358&displayformat=dictionary "Terminologie francophone de Moodle : Test")...  
Ou disposer d'un mode d'emploi en français pour l'[administrateur](https://moodle.org/mod/glossary/showentry.php?eid=4734&displayformat=dictionary "Terminologie francophone de Moodle : Administrateur") simpliste que je suis !  
Daniel

Moyenne des évaluations  -

- [◄ [Devoirs] Travail envoyé mais resté en brouillon](https://moodle.org/mod/forum/discuss.php?d=462232)
- [Contact et achèvement d'activités par groupement ►](https://moodle.org/mod/forum/discuss.php?d=462080)