====== Exporter ou sauvegarder une VM ======

[[https://www.vincentliefooghe.net/content/exporter-ou-sauvegarder-une-vm-kvm]]

En dehors des containers LXC que j'utilise souvent pour isoler mes environnements de test, j'utilise aussi des "vraies" machines virtuelles avec KVM, notamment lorsque j'ai besoin d'une VM windows ou d'un autre Linux ; 

Pour le stockage des données KVM, j'utilise des volumes logiques avec LVM.

Pour exporter ou sauvegarder une VM, il faut donc récupérer la définition de la VM, ainsi que les données.

===== Sauvegarde manuelle =====

==== Fichier de définition ====

Le fichier de définition se trouve dans /etc/libvirt/qemu/. Il est au format xml, avec le nom de la machine. Par exemple :

  /etc/libvirt/qemu/Ubuntu-LXD.xml décrit la configuration de la machine Ubuntu-LXD.

On peut également exporter avec la commande virsh dumpxml NOM_DE_LA_VM. Exemple :

  virsh dumpxml Ubuntu-LXD > /mnt/backup/Ubuntu-LXC.xml
  
==== Données ====

Dans le fichier de définition, on trouve le paramétrage des disques. La source peut être de type dev (volume logique) ou file (fichier):

    <disk type='block' device='disk'>
      <driver name='qemu' type='raw'/>
      <source dev='/dev/vmvg/Ubuntu-LXD'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
    </disk>
    
Pour un fichier :

    <disk type='file' device='disk'>
      <driver name='qemu' type='raw'/>
      <source file='/var/lib/libvirt/images/CentOS6.img'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
    </disk>
    
Pour sauvegarder ces données, on peut donc utiliser la commande dd :

  dd if=/dev/vmvg/Ubuntu-LXD of=/mnt/backup/Ubuntu-LXD.img bs=512k  # cas d'un dev

  dd if=/var/lib/libvirt/images/CentOS6.img of=/mnt/backup/CentOS6.img bs=512k  # cas d'un file

===== Via un script =====

Pour me simplifier la vie, j'ai développé un script qui fait toutes ces étapes, et vérifie également l'espace disque disponible.


  #!/bin/bash
  #
  # Liste des domaines
  #
  #-----------------------------------------------------------------
  if [ "$(id -u)" != "0" ]; then
    echo "Merci de lancer ce script en tant que root"
    exit 1
  fi
  #
  # Repertoire par defaut
  #
  BACKUPDIR=/media/vl/backup/KVM

  # Liste des machines virtuelles (domain KVM)
  virsh list --all

  echo -n "Machine / domaine : "
  read DOM

  if [ -z $DOM ] ; then
    echo "Pas de machine saisie..."
    exit
  fi

  echo -n "Backup directory : [$BACKUPDIR] "
  read BACKDIR

  if [ -z $BACKDIR ] ; then
    BACKDIR=$BACKUPDIR
  fi

  if [ ! -d $BACKDIR ] ; then
    echo "Repertoire $BACKDIR inexistant ! "
    exit 1
  fi

  # Export de la configuration
  virsh dumpxml $DOM > ${BACKDIR}/$DOM.xml
  if [ $? -ne 0 ] ; then
    echo "Erreur dans l'export XML"
    exit 1
  fi

  # Recherche des partitions ou fichiers de la VM
  IMGFILE=$(grep 'source dev' ${BACKDIR}/${DOM}.xml | cut -d "'" -f2)
  # premier test avec source dev
  if [ "$IMGFILE" = "" ] ; then
    echo "Test avec source file"
    IMGFILE=$(grep 'source file' ${BACKDIR}/${DOM}.xml | cut -d "'" -f2)
    if [ "$IMGFILE" = "" ] ; then
      echo "Impossible de trouver l'image de la VM"
      exit
    fi
  fi

  for rawfile in $IMGFILE ; do
    echo
    echo -n "Espace occupé par $rawfile : "
    VMSIZE=$(virsh vol-info $rawfile|grep Allocation|cut -c '15-25'|cut -d ',' -f1 )
    echo $VMSIZE
    echo
    DFSIZE=$(df -h $BACKDIR --output=avail |tail -1|cut -d ',' -f1)
    echo "Espace dispo sur $BACKDIR : $DFSIZE "
    if [ $VMSIZE -gt $DFSIZE ] ; then
      echo "Il n'y a pas assez de place disponible sur le répertoire cible $BACKDIR"
     exit
    fi
    echo -n "Sauvegarde de $rawfile ? "
    read rep
    if [ "$rep" = "o" ] ; then
      filename=$(basename $rawfile)
      echo dd if=${rawfile} of=${BACKDIR}/${DOM}-${filename}.img bs=512K
      dd if=${rawfile} of=${BACKDIR}/${DOM}-${filename}.img bs=512K
    fi
  done