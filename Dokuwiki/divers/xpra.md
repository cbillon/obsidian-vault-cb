---
tags:
  - xpra
  - tools
---

====== Xpra ======

[[https://xpra.org/index.html|Site Officiel]

===== Installation =====


ensure the SSL certificates are up to date: sudo apt install ca-certificates
import the key used for signing the packages (newer Debian releases require a newer key, a newer sources format, etc):
sudo wget -O "/usr/share/keyrings/xpra.asc" https://xpra.org/xpra.asc
(if the file already exists, you may hit some problems )
download the appropriate repository file (see table below) into /etc/apt/sources.list.d as root:
cd /etc/apt/sources.list.d ; wget $REPOFILE
then run apt update ; apt install xpra