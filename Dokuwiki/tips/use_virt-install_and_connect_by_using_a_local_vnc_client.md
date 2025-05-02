---
tags:
  - kvm
  - vnc
---

====== Use virt-install and connect by using a local VNC client ======

[[https://docs.openstack.org/image-guide/virt-install.html]]

If you do not wish to use virt-manager (for example, you do not want to install the dependencies on your server, you do not have an X server running locally, the X11 forwarding over SSH is not working), you can use the virt-install tool to boot the virtual machine through libvirt and connect to the graphical console from a VNC client installed on your local machine.

Because VNC is a standard protocol, there are multiple clients available that implement the VNC spec, including TigerVNC (multiple platforms), TightVNC (multiple platforms), RealVNC (multiple platforms), Chicken (Mac OS X), Krde (KDE), Vinagre (GNOME).

The following example shows how to use the qemu-img command to create an empty image file, and virt-install command to start up a virtual machine using that image file. As root:

  # qemu-img create -f qcow2 /tmp/centos.qcow2 10G
  # virt-install --virt-type kvm --name centos --ram 1024 \
    --disk /tmp/centos.qcow2,format=qcow2 \
    --network network=default \
    --graphics vnc,listen=0.0.0.0 --noautoconsole \
    --os-type=linux --os-variant=centos7.0 \
    --location=/data/isos/CentOS-7-x86_64-NetInstall-1611.iso
  # remplacemen location by cd-rom (cb)
  Starting install...
  Creating domain...                     |    0 B     00:00
  Domain installation still in progress. You can reconnect to
  the console to complete the installation process.