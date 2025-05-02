---
tags:
  - ssh
---

====== Audit Tool ======

[[https://www.cyberciti.biz/tips/how-to-audit-ssh-server-and-client-config-on-linux-unix.html]]

ssh-audit is a tool for ssh server and client auditing. For example, you can use this tool:

  * Scan for OpenSSH server and client config for security issues
  * Make sure the correct and recommended algorithm is used by your Linux and Unix boxes
  * Check for OpenSSH banners and recognize device or software and operating system
  * Lookup for ssh key exchange, host-keys, encryption, and message authentication code algorithms
  * Liste à puceAlert developers and sysadmin about config issues, weak/legacy algorithms, and features used by SSH
  * Historical information from OpenSSH, Dropbear SSH, and libssh
  * Policy scans to ensure adherence to a hardened/standard configuration

===== To install from Dockerhub =====

  docker pull positronsecurity/ssh-audit

Then run it as follows:

  docker run -it -p 2222:2222 positronsecurity/ssh-audit {ssh-server-ip}
  docker run -it -p 2222:2222 positronsecurity/ssh-audit 192.168.2.17
