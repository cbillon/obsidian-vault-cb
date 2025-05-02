---
tags:
  - nas
  - backup
  - synology
---

====== Sauvegarde d'un serveur Linux sur un NAS Synology ======

[[https://www.vincentliefooghe.net/content/sauvegarde-dun-serveur-linux-sur-un-nas-synology]]

J'ai acquis récemment un NAS Synology DS215J , afin de m'en servir comme serveur de fichiers à la maison.

Vu la capacité installée (2 x 2 To en RAID 1), je me suis dit que je pouvais aussi l'utiliser pour la sauvegarde externalisée des VPS utilisés pour mon activité d'hébergeur HebInWeb.com

J'avais déjà un script shell que je lançais sur mon PC perso, qui allait récupérer par rsync les données sur les VPS.

Comme le coeur du NAS est un linux, la mise en place a été assez simple. Pour cela il a juste fallu :

  * Générer une paire de clé RSA (privée / publique) sur le NAS
  * Copier la clé publique sur le serveur distant
  * Adapter le script de sauvegarde
  * Tester...
  * Créer une tâche planifiée sur le NAS

===== Génération de la paire de clé =====

Une fois connecté sur le serveur NAS, avec le compte admin, on peut commencer les manipulations.

J'ai déjà décrit dans un article (https://www.vincentliefooghe.net/content/configuration-ssh-pour-ladmin) comment mettre en place un accès ssh sur des machines. Nous sommes dans le même cas de figure.

  ssh-keygen -t rsa 

Je n'ai pas mis de mot de passe, car les échanges se font de machine à machine. Par défaut, cela me crée une paire de fichiers :

  * id_rsa : la clé privée
  * id_rsa.pub : la clé publique.

==== Copie de la clé publique sur le serveur distant ====

Une fois la clé publique disponible, on l'ajoute dans le fichier $HOME/.ssh/authorized_keys du compte que l'on va utiliser sur le serveur distant.

On peut alors tenter la connexion, depuis le compte sur le NAS. J'ai également ajouté une entrée dans mon fichier .ssh/config (sur le NAS) de la forme :

  host vps1
    hostname monbeauvps.hebinweb.com
    user operator
    port 22
    
A ce stade, on peut se connecter simplement via :

  ssh vps1
  
Ceci doit nous amener directement sur le serveur, sans saisir de mot de passe. A la première connexion, on doit valider l'ajout du serveur dans le fichier .ssh/known_hosts.

=== Script de sauvegarde ===

J'ai adapté le script de sauvegarde très simplement, uniquement remplacé /bin/bash par /bin/sh.

J'utilise sur mes VPS une structure de répertoires identiques, avec des fichiers de configuration donnant le chemin vers les répertoires, les connexions (éventuelles) à la base de données.

Le script comment par récupérer ces fichiers de configurations, puis va synchroniser (via rsync) les répertoires et si nécessaire, les dump de base. Au final, ça se présente comme cela :

  #!/bin/sh
  #
  # Recupération des backups des Serveurs, via rsync
  #
  # Step 1 : connexion au remote, recuperation des fichiers de configuration
  # Step 2 : parcours des fichiers de configuration, et rsync des fichiers
  # Step 3 : récupération des dumps de base de données
  # Step 4 : récupération des fichiers tar.gz (etc, vmail, etc)
  #=================================================================================================
  if [ -z $1 ] ; then
   exit
  else
   REMOTE=$1
  fi

  LOGDIR=/volume1/VPS/logs
  LOGFILE=${LOGDIR}/log-${REMOTE}-$(date "+%Y-%m-%d_%H-%M")
  Log () {
    ladate=$(date "+%d/%m/%Y %H:%M:%S")
    echo "[$ladate] - $@" >> ${LOGFILE}
    echo "[$ladate] - $@"
  }

  date "+%d/%m/%Y %H:%M:%S"
  Log "Debut de la recuperation des backups de $REMOTE"
  #
  # Define rsync command
  #
  RSYNC="rsync -az --protocol=26"
  # Remote server             
  REMOTEDIR=/backups     
  DUMPDIR=$REMOTEDIR/last     
  OTHERDIR=$REMOTEDIR/other   
  CONFDIR=/backups/conf.d     
  REMUSER=root

  # local machine
  BCKDIR=/volume1/VPS/backup/$REMOTE
  ARCHDIR=$BCKDIR/ARCH        
  FILESDIR=$BCKDIR/FILES
  DBDIR=$BCKDIR/DUMP
  cd $BCKDIR

  # Step 1
  Log "(Step 1) Récupération des fichiers de configuration .conf"
  rm $BCKDIR/config/*.conf > /dev/null 2>&1
  ${RSYNC} ${REMUSER}@${REMOTE}:$CONFDIR/*.conf ${BCKDIR}/config

  # Step 2
  Log "(Step 2) Récupération des directories depuis le serveur $REMOTE dans la directory $REMOTEDIR"
  lesConfs=`ls $BCKDIR/config/*.conf`
  for myConf in $lesConfs ; do
    source $myConf
    Log "Synchro Directory $SYNC_DIR"
    ${RSYNC} ${REMUSER}@${REMOTE}:$ROOT_PATH/$ROOT_DIR $FILESDIR/$SYNC_DIR/
  done

  Log "(Step 3) Récupération des dumps de base"
  for DUMP in $(rsh $REMOTE "ls $DUMPDIR/dmp*.sql.gz")
  do
    Log "Envoi du dump $DUMP"
    ${RSYNC} ${REMUSER}@${REMOTE}:$DUMPDIR/ $DBDIR/
  done

  Log "(Step 4) Récupération des autres sauvegardes"
  rsync -az ${REMUSER}@${REMOTE}:$OTHERDIR/ $BCKDIR/

  Log "Fin de la récupération des fichiers de $REMOTE"

== Création de la tâche planifiée ==

Une fois que l'on a testé en ligne de commande, il reste juste à créer une tâche planifiée via l'interface Web du Synology
 
Désormais, la sauvegarde s'effectuera tous les jours.

La première sauvegarde sera assez longue, puisqu'il faut tout transférer, mais par la suite on ne copie que les delta