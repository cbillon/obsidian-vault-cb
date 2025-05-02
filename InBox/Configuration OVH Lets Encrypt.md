---
tags:
  - ovh
  - Lets-Encrypt
  - api
---
Pour générer une application   ouvrir le lien

https://api.ovh.com/createToken/?GET=/domain/zone/cbillon.ovh/*&POST=/domain/zone/cbillon.ovh/*&PUT=/domain/zone/cbillon.ovh/*&GET=/domain/zone/cbillon.ovh&DELETE=/domain/zone/cbillon.ovh/record/*

Pour ouvrir la console API

eu.api.ovh.com

sélectionner /me

applications liste applications-id

Select get (or delete tu supress application)
applications/{applicationID}  renvoie applicationKey

credentials {applicationId}  reboie les droits d'accés

![[Pasted image 20240430083258.png]]
cd4d90a40814717a
81fe9ddb47b434700f6339559255d923
8ed862485ddd80597454604e489d1852



```
OVH_ENDPOINT="ovh-eu"  
OVH_APPLICATION_KEY="cd4d90a40814717a"  
OVH_APPLICATION_SECRET="f20fe0cef02fb73d2a39a5fd11b2e279"  
OVH_CONSUMER_KEY="8ed862485ddd80597454604e489d1852"
```