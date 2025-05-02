---
tags:
  - vs
  - ssh
  - remote
---

## Visual Studio Connection Remote ssh 

[Site](https://www.youtube.com/watch?v=lKXMyln_5q4)

## Environemment

  * Windows10
  * Visual Studio 

Vérifier que que openssh est installé sur le serveur windows 

  cd c:\windows\system32\OpenSSH
  dir

On doit installer l'extension Microsoft ssh Remote
Dans notre cas on veut se connecter sur le serveur dev hebergé chez OVH :** 164.13.161.115**

Vérification de la connexion ssh

  ssh -p 50635 cb@164.132.161.115

  saisir le mot de passe  

## Génération de clés ssh 

en ligne de commande 

  ssh-keygen -t rsa -b 4096
  
La paire de clés est créée dans  c:\Users\cb\    
  keygen  et keygen.pub  

Recopie de la clé publique sur le serveur remote
 
  scp -P 50635 c:\Users\cb\keygen.pub cb@164.132.161.115:~/key.pub

Se connecter sur le serveur remote et ajouter la clé publique

  cat ~/key.pub >> ~/.ssh/authorized_keys
  chmod 0600 ~/.ssh/authorized_keys
  
Vérification de la connexion

  ssh -i c:\Users\cb\keygen -p 50635 cb@164.132.161.115
  
===== Configuration de Visual Studio =====
 
Dans Visual Studio ouvrir Open Remote SSH en cliquant sur l'icone en bas à gauche

select Connect to Host

Le fichier de configuration : **c:\Users\cb\.ssh\config**

Crer une entrée pour le serveur dev

  Host dev
      Hostname 164.132.161.115
      User cb
      Port 50635
      IdentityFile c:\Users\cb\keygen
      
 Sélectioner  dev pour se connecter




