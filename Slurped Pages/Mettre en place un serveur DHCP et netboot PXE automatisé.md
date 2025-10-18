---
link: https://www.shpv.fr/blog/netboot-dhcp-pxe/
byline: SHPV
site: SHPV
date: 2025-07-11T09:00
excerpt: Apprenez à installer et configurer un serveur DHCP et un service PXE pour automatiser le démarrage réseau et le déploiement d'OS sur votre parc machine.
twitter: https://twitter.com/@SHPVFR
tags:
  - pxe
slurped: 2025-10-12T18:51
title: Mettre en place un serveur DHCP et netboot PXE automatisé
---

Publié le 11 juillet 2025

Le boot réseau via PXE (Preboot Execution Environment) permet d’automatiser l’installation et le démarrage d’OS sur des machines sans support local. En associant un serveur DHCP à un service PXE, vous facilitez la gestion de vos déploiements à grande échelle (bureaux, salles machines, datacenters…).

## Prérequis

- 1 serveur Linux (Debian, Ubuntu, Rocky, etc.)
- Droits root/sudo
- Réseau local dédié ou VLAN pour les tests
- Image ISO d’installation ou environnement Live

## Installation du serveur DHCP

### 1. Installer isc-dhcp-server

```
sudo apt update
sudo apt install isc-dhcp-server -y
```

### 2. Exemple de configuration `/etc/dhcp/dhcpd.conf`

```
subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.100 192.168.1.200;
  option routers 192.168.1.1;
  option broadcast-address 192.168.1.255;
  next-server 192.168.1.10;
  filename "pxelinux.0";
}
```

- `next-server` : IP du serveur PXE/TFTP

## Mise en place du service PXE (TFTP)

### 1. Installer tftpd-hpa

```
sudo apt install tftpd-hpa syslinux-common -y
```

### 2. Copier les fichiers de boot

```
sudo cp /usr/lib/PXELINUX/pxelinux.0 /var/lib/tftpboot/
sudo mkdir /var/lib/tftpboot/pxelinux.cfg
sudo cp <chemin_iso>/initrd.img /var/lib/tftpboot/
sudo cp <chemin_iso>/vmlinuz /var/lib/tftpboot/
```

### 3. Exemple de menu PXE `/var/lib/tftpboot/pxelinux.cfg/default`

```
DEFAULT install
LABEL install
  KERNEL vmlinuz
  INITRD initrd.img
  APPEND root=/dev/ram0 init=/linuxrc rw
```

## Démarrage d’un client en PXE

1. Configurer le BIOS/UEFI pour démarrer en réseau (option PXE/Network Boot)
2. Vérifier que la machine obtient une IP et lance “pxelinux.0” via TFTP

## Astuces & bonnes pratiques

- **Séparer DHCP et PXE** si plusieurs serveurs DHCP sont présents (option 66/67)
- Protéger l’accès TFTP (firewall, VLAN dédié)
- Automatiser la génération de menus via scripts ou templates

## Avantages

- Installation rapide et centralisée de plusieurs machines
- Zéro support amovible requis (USB/CD)
- Gain de temps massif pour les déploiements en série

## Limites

- Risque d’écraser des machines en production si mal configuré
- Sensible aux coupures réseau ou erreurs de config
- Nécessite une bonne gestion des BIOS/UEFI clients

## Conclusion

La combinaison DHCP + PXE modernise et automatise vos déploiements de systèmes d’exploitation. Elle est idéale pour l’industrialisation et l’administration de grandes flottes de machines.

###### Besoin d'aide sur ce sujet ?

Notre équipe d'experts est là pour vous accompagner dans vos projets.

[Contactez-nous](https://www.shpv.fr/demander-un-devis)

---

##### Articles similaires qui pourraient vous intéresser