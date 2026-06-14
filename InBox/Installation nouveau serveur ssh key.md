---
tags:
  - ssh
  - SSH-key
  - server
type: tuto
---
## pre requis
en local 

On se connect en local  avec identifiant mot de passe:  cb sesame




un script permet de creer les cles et d installer la clé publique sur le server distant




```
rsync -av ~/.ssh/id_ed25519 cb:sesame@192.168.1.102:/home/cb/id_ed25519
rsync -av ~/.ssh/id_ed25519.pub cb@192.168.1.102:/home/cb/id_ed25519.pub

```

