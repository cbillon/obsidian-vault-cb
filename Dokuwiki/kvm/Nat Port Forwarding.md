---
tags:
  - kvm
  - network
---

====== NAT Port forwarding ======

[[https://wiki.libvirt.org/Networking.html#host-configuration-bridged]]

Creer un un fichier dans /etc/libvirt/hooks/qemu (or add the following to an already existing hook script)

  
  #!/bin/bash
  # IMPORTANT: Change the "VM NAME" string to match your actual VM Name.
  # In order to create rules to other VMs, just duplicate the below block and configure
  # it accordingly.  
  if [ "${1}" = "VM NAME" ]; then
     # Update the following variables to fit your setup
     GUEST_IP=
     GUEST_PORT=
     HOST_PORT=
     if [ "${2}" = "stopped" ] || [ "${2}" = "reconnect" ]; then
      /sbin/iptables -D FORWARD -o virbr0 -p tcp -d $GUEST_IP --dport $GUEST_PORT -j ACCEPT
      /sbin/iptables -t nat -D PREROUTING -p tcp --dport $HOST_PORT -j DNAT --to $GUEST_IP:$GUEST_PORT
     fi
     if [ "${2}" = "start" ] || [ "${2}" = "reconnect" ]; then
      /sbin/iptables -I FORWARD -o virbr0 -p tcp -d $GUEST_IP --dport $GUEST_PORT -j ACCEPT
      /sbin/iptables -t nat -I PREROUTING -p tcp --dport $HOST_PORT -j DNAT --to $GUEST_IP:$GUEST_PORT
     fi
  fi
  
Si il y a d'autres port forwarding ajouter le code dupliqué dans le fichier pour chaque environnement