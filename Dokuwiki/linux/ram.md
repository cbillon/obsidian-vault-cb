---
tags:
  - linux
  - tips
---

====== Récupérer la consommation mémoire d'un process sous linux ======

[[https://www.vincentliefooghe.net/content/r%C3%A9cup%C3%A9rer-la-consommation-m%C3%A9moire-dun-process-sous-linux]]

Il est assez facile de récupérer la consommation totale de mémoire sur un serveur Linux/Unix, via la commande free par exemple :

  free -m
               total       used       free     shared    buffers     cached
  Mem:          5963       3516       2446          0        194       1161
  -/+ buffers/cache:       2160       3802
  Swap:         4095          7       4088

On voit ici que l'on utilise 3516 Mo sur 5963 disponibles, mais que sans la mémoire utilisée par les buffers et caches, on n'en prend que 2160 (la deuxième colonne de la deuxième ligne).

Idem pour un seul processus, un coup de "ps aux" donne, dans la colonne %RSS, la mémoire réservée pour chaque process :

   ps aux
  USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  root         1  0.0  0.0  10348   700 ?        Ss   Feb24   0:01 init [4]
  .../...
  root     25639  0.0  0.0  44268   644 ?        Ss   05:30   0:00 /usr/sbin/vsftpd 
  /etc/vsftpd/vsftpd.conf
  nobody   25656  0.0  0.0   4576  1316 ?        S    05:30   0:00 /product/ibmhttpserver/V6.1/bin/httpd - 
  d /product/ibmhttpserver/V6.
  nobody   25659  0.0  0.0 288004  4284 ?        Sl   05:30   0:00 /product/ibmhttpserver/V6.1/bin/httpd - 
  d /product/ibmhttpserver/V6.
  nobody   25661  0.0  0.0 287964  4236 ?        Sl   05:30   0:00 /product/ibmhttpserver/V6.1/bin/httpd - 
  d /product/ibmhttpserver/V6.
  cdirect  25730  0.0  0.0   3200  1700 ?        S    05:30   0:00 ftdaemon stf

===== Comment faire alors pour récupérer le total de tous les processus ibmhttpserver ? =====


Un script combinant shell + awk permet d'avoir le total des processus.

  #!/bin/bash

  if [ "$1" = "" ] ; then
    echo -n "Nom du process : "
    read process
  else
    process=$1
  fi

  ps aux | grep $process | grep -v grep | awk 'BEGIN { sum=0 } {sum=sum+$6; } END {printf("Taille RAM 
  utilisée: %s Mo\n",sum / 1024)}'

Par exemple :

  ./memProc ibmhttpserver
  Taille RAM utilisée: 15.9492 Mo

Simple, mais efficace.