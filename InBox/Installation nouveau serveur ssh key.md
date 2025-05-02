---
tags: 
type: tuto
---
#Compte utilisateur
Le compte existe sur le nouveau serveur 
On peut se connecter avec identifiant mot de passe cb sesame
Ajout du compte dans le groupe sudo
```
```
  usermod -aG sudo cb
```
```
Donner utilisation sudo sans mot de passe aux membres du groupe sudo


```
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL) NOPASSWD:ALL
```
Génération de la clé 
Les versions ubuntu n'acceptent plus par defaut _rsa preferer _ed25519
se positionner dans ~/.ssh

```
ssh-keygen -t ed25519 -C "claude.billon@gmail.com"

```
Recopie de la clé sur le nouveau serveur

```
ssh-copy-id -i ~/.ssh/id_ed25519.pub cb:sesame@192.168.1.102
```

Ceci ajoute sue le nouveau serveur la clé .pub dans .ssh/autorizedkeys

Mettre à jour le fichier ~/.ssh/config

```
Host asus
  Hostname 192.168.1.102
  User cb
  IdentityFile ~/.ssh/id_ed25519
```

Vérifier la connexion

```
ssh asus
```

On peur recopier les clés sur le nouveau serveur ou utiliser l'option ForwardAgent

Recopie des clés

```
rsync -av ~/.ssh/id_ed25519 cb:sesame@192.168.1.102:/home/cb/id_ed25519
rsync -av ~/.ssh/id_ed25519.pub cb@192.168.1.102:/home/cb/id_ed25519.pub

```

