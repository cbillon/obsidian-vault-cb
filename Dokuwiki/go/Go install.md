---
tags:
  - go
  - install
---

[[https://go.dev/doc/install]]

  * télécharger la bonne version x86_64
copy avec rsync sur le site
    * decompression
    * mise à jour de PATH

  sudo rm -rf /usr/local/go &&sudo tar -C /usr/local -xzf go1.21.4.linux-amd64.tar.gz
  
  export PATH=$PATH:/usr/local/go/bin
 
 
    * Vérification : go version

