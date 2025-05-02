====== LFTP ======

===== Connexion OVH =====

===== Créer un fichier lftp.cnf =====


  #debug 3
  set ftp:ssl-allow no
  set ssl:verify-certificate no
  set net:socket-bind-ipv4 51.91.104.76
  open ns3036570.ip-164-132-161.eu:iVtnpSzriw@ftpback-rbx7-592.mybackup.ovh.net

Pour vérifier la connexion lftp -f lftp.cnf

==== Pour créer un bookmark ====

créer un fichier add_bookmark
  source lftp.cnf
  bookmark add save

executer lftp -f add_bookmark

Vérificacation lftp save

==== Commande Mirror ====

Cette commande permet de synchroniser  2 répertoires :
on a : mirror <source> <destination>

  mirror  <remote> <local>  get   mise à jour du dépôt local 

  mirror -R <local> <remote>  put  sauvegarde du dépôt local


==== Liste des commandes ====

[[https://www.tutorialspoint.com/unix_commands/lftp.htm]]










