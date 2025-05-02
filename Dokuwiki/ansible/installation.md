---
tags:
  - ansible
  - install
---

====== Installation Ansible ======

===== Installation environnement =====

  * Installation de direnv [[direnv| ici]]
  * créer un réportoire du projet
    dans ce répertoire créer un répertoire .envrc
    
    export PUB_KEY=$(cat ~/.ssh/id_rsa.pub)
    layout_python3
    export PATH="$HOME/.local/bin:$PATH"

    

  * installation python

    sudo apt install python3-pip 
    sudo apt install python3.8-venv
    

  * installation ansible

    sudo apt install ansible-core


