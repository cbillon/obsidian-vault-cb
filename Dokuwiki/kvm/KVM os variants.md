---
tags:
  - kvm
  - os
---


When creating a guest with **virt-install** you need to specify the **–os-variant**.

To get a list of acceptable values, first install the **libosinfo-bin** package

  sudo apt install libosinfo-bin

before running the command below:

  osinfo-query os