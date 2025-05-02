---
tags:
  - linux
  - ldap
---

====== LDAP ======

===== Installtion ldapsearch =====

Sous linux

    sudo apt install ldap-utils

==== Verification de la connexion ====

  ldapsearch -x -H ldap://212.99.95.157 -D uid=ldaps.moodle,ou=people,o=accounts,dc=groupe-igs,dc=net -w 
  861KYL -b uid=claude.billon,ou=people,o=accounts,dc=groupe-igs,dc=net

=== Moodle configuration ===

Se connecter : cbi-test-41.cbillon.ovh

Distinguished name : uid=ldaps.moodle,ou=people,o=accounts,dc=groupe-igs,dc=net
Mot de passe : 861KYL

Contexts: ou=people,o=accounts,dc=groupe-igs,dc=net

Appariements des données

  User attriute: uid
  Prenom: prenom
  Nom: sn
  Adresse courriel: mailPro
  Numero d'identification: idnumber
  
 
== Test connexion ==

Sur la ligne de parametarge de la méthode d'autentification : test settings
  

