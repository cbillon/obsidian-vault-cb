---
tags:
  - sudo
---
Ajout du compte dans le groupe sudo```
```
  usermod -aG sudo $USER
```
```
Donner utilisation sudo sans mot de passe aux membres du groupe sudo


```
sudo visudo

%sudo   ALL=(ALL:ALL) NOPASSWD:ALL