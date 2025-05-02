====== Install vncserver ======


[[https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-22-04]]

Sur le site remote :

  * Install xfxe4 xfxe4-goodies
  * Install tightvncserver

  * lancer vncserver
  * créer un mot de passe

Commandes utiles

  pgrep vnc
  
Affiche le pid 

  sudo kill <pid>

Pour lancer

  vncserver : 1
  
Pour arretet 
   
  vncserver -kill :1 

Configuration du fichier ~/.vnc/startup

                                                                                              

  set -xv
  xrdb $HOME/.Xresources
  xsetroot -solid grey
  #x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
  #x-window-manager &
  # Fix to make GNOME work
  export XKL_XMODMAP_DISABLE=1
  /etc/X11/Xsession /usr/bin/startxfce4
  #xrdb $HOME/.Xresources
  #startxfce4 &


     