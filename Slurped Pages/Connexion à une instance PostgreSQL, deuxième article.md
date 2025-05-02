---
link: https://www.loxodata.com/post/connexions2/
excerpt: Loxodata vous présente les différentes méthodes de connexion à une
  instance PostgreSQL
slurped: 2025-02-22T09:11
title: Connexion à une instance PostgreSQL, deuxième article
---

## Certificats de connexion

Ce deuxième article de la série présente une fonctionnalité sous-exploitée de PostgreSQL : l’authentification par certificat SSL.

Beaucoup d’administrateurs de bases de données et de développeurs, n’étant pas experts en sécurité, se contentent d’une authentification par mot de passe. Si la version 10 de PostgreSQL introduit l’algorithme [SCRAM](https://www.postgresql.org/docs/10/static/auth-methods.html#AUTH-PASSWORD), améliorant l’authentification par mot de passe, le mot de passe reste stocké et connu par l’application, avec tous les problèmes que ça peut poser en matière de sécurité.

Quel que soit le mode d’authentification, il est possible de chiffrer tout échange entre le client et le serveur, ce qui limite les risques d’interception des données, à l’image de ce qu’est https pour http. L’échange chiffré des données n’est pas optionnel pour l’authentification basée sur des certificats.

![](https://www.loxodata.com/images/post/connexions-2/4652a9e59641cea9f841feaca11bf59e.jpg)

## Autorité de certification

La première étape consiste donc à créer une [autorité de certification](https://en.wikipedia.org/wiki/Certificate_authority) qui pourra délivrer des certificats décrivant l’identité du serveur ou des clients. Dans le contexte d’un échange un client et un serveur PostgreSQL, il n’est généralement pas nécessaire de recourir à une autorité de certification publique, car le client est très souvent connu du serveur à l’avance (et inversement), contrairement à un serveur Web.

Les commandes suivantes, effectuées en tant que `root`, vont créer une clé privée et le certificat signé :

```
openssl genrsa -des3 -out /etc/ssl/private/trustly-ca.key
chown root:root /etc/ssl/private/trustly-ca.key
chmod 400 /etc/ssl/private/trustly-ca.key
```

La clé privée n’est ici accessible qu’à l’utilisateur `root`.

```
openssl req -new -x509 -days 3650 \
  -subj '/C=FR/ST=BZH/L=Nantes/O=Loxodata/CN=trustly' \
  -key /etc/ssl/private/trustly-ca.key \
  -out /usr/local/share/ca-certificates/trustly-ca.crt
```

L’option `-days` permet d’indiquer la durée de validité du certificat ainsi signé. La valeur par défaut est 30 jours.

Le mot de passe demandé doit bien sûr être solide et secrètement conservé.

Cette clé et ce certificat peuvent être utilisés pour signer les clés de plusieurs serveurs et de plusieurs clients, de telle sorte qu’une organisation peut n’avoir qu’un seul jeu de fichier.

Ces fichiers n’ont pas besoin d’être distribués dans tous les systèmes concernés, mais seulement dans le ou les systèmes ou l’on signe les clés.

Dans cet article, le serveur et le client PostgreSQL fonctionnent dans le même système, pour simplifier la démonstration. Lors d’une utilisation réelle, il suffit de copier les fichiers dans le bon système pour créer les certificats et les fichiers racine (`root.crt`).

## Certificats coté serveur

La connexion du client nécessitant une communication chiffrée, la première chose à faire est de créer une [clé](https://fr.wikipedia.org/wiki/Cryptographie_asym%C3%A9trique) et un certificat pour le serveur :

```
su - postgres
mkdir ssl
cd ssl/
openssl genrsa -des3 -out server.key
openssl rsa -in server.key -out server.key
```

Le deuxième appel permet de créer une clé sans mot de passe, ce qui évite de devoir saisir ce mot de passe à chaque démarrage de l’instance PostgreSQL.

### Demande

Ensuite, à partir de cette clé, on crée un fichier de demande de certificat :

```
openssl req -new -nodes -key server.key -out server.csr \
    -subj '/C=FR/ST=BZH/L=Nantes/O=Loxodata/CN=tolva'
```

L’option `CN` est le nom qui va effectivement être utilisé par le client lors de la connexion au serveur.

Il est possible d’examiner le contenu du fichier produit avec la commande :

```
openssl req -in server.csr -noout -text
```

### Signature

Puis, la demande de certification est signée à partir de l’autorité de certification, effectuée en tant que `root` :

```
openssl req -x509 -key /etc/ssl/private/trustly-ca.key -days 360 \
    -in ~postgres/ssl/server.csr -out ~postgres/ssl/server.crt
chown postgres: ~postgres/ssl/server.crt
```

L’option `-days` permet d’indiquer la durée de validité du certificat ainsi signé. La valeur par défaut est 30 jours.

Il est possible d’examiner le contenu du fichier produit avec la commande :

```
openssl x509 -in ~postgres/ssl/server.crt -noout -text
```

### Racine

Enfin, les certificats doivent être ajouté au fichier [`root.crt`](https://en.wikipedia.org/wiki/Root_certificate) :

```
cat /usr/local/share/ca-certificates/trustly-ca.crt >> ~postgres/ssl/root.crt
chown postgres: ~postgres/ssl/root.crt
su - postgres
cat ~postgres/ssl/server.crt >> ~postgres/ssl/root.crt
```

### Configuration de PostgreSQL

Les fichiers créés doivent être renseignés dans le fichier de configuration `postgresql.conf`, tout en activant le mode `ssl` si ça n’est pas déjà fait :

```
ssl = true
ssl_cert_file = '/var/lib/postgresql/ssl/server.crt'
ssl_key_file = '/var/lib/postgresql/ssl/server.key'
ssl_ca_file = '/var/lib/postgresql/ssl/root.crt'
```

Ces changements nécessitent un redémarrage de l’instance.

Ensuite, la règle d’accès autorisant la connexion depuis un certificat doit être ajoutée au fichier `pg_hba.conf` :

```
hostssl all slardiere 192.168.1.0/32 cert clientcert=1
```

Comme pour toute manipulation de ce fichier, il faut être attentif à l’ordre des lignes et à la hiérarchie induite des règles.

![](https://www.loxodata.com/images/post/connexions-2/89ceee013c1c67a1dcdb4ec85b80bb0b.jpg)

## Certificats coté client

Le certificat coté client doit être placé dans le répertoire `~/.postgresql/`. La clé privée ayant un mot de passe, celui-ci devra être saisi à chaque connexion, mais n’est jamais connu pas le serveur ni ne transite sur le réseau :

```
openssl genrsa -des3 -out ~/.postgresql/postgresql.key
openssl req -new -key ~/.postgresql/postgresql.key -out ~/.postgresql/postgresql.csr  \
        -subj '/C=FR/ST=BZH/L=Nantes/O=Loxodata/CN=slardiere'
```

L’option `CN` correspond au rôle de connexion connu par la base de données.

### Signature

En tant que `root`, la demande de certification est signée :

```
openssl x509 -req -in /home/slardiere/.postgresql/postgresql.csr \
                  -CA /usr/local/share/ca-certificates/trustly-ca.crt \
                  -CAkey /etc/ssl/private/trustly-ca.key -days 360 \
                  -out /home/slardiere/.postgresql/postgresql.crt -CAcreateserial
chown postgres: ~postgres/ssl/postgresql.crt
```

L’option `-days` permet d’indiquer la durée de validité du certificat ainsi signé. La valeur par défaut est 30 jours.

### Racine

Comme côté serveur, on ajoute les certificats au fichier `root.crt`, en tant que `root` :

```
cat /usr/local/share/ca-certificates/trustly-ca.crt >> ~slardiere/.postgresql/root.crt
chown slardiere: ~slardiere/.postgresql/root.crt
```

puis, en tant qu’utilisateur :

```
cat ~postgres/ssl/server.crt >> ~postgres/ssl/root.crt
```

### Connexion

La connexion peut alors se faire, en précisant le mode `ssl`, parmi :

- `require` : si le fichier `root.crt` est présent, vérifie que le certificat coté serveur vient bien de l’autorité de certification ;
- `verify-ca` : vérifie que le certificat coté serveur vient bien de l’autorité de certification ;
- `verify-full` : en plus que ce que fait `verify-ca`, et que le champ CN correspond au nom utilisé.

```
$ psql -h tolva "sslmode=verify-full"
Enter PEM pass phrase:
psql (10.5 (Debian 10.5-1.pgdg90+1))
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

tolva:5432 slardiere@postgres=>
```

### Services

Il est possible de définir plusieurs connexions, en utilisant des fichiers différents que ci-dessus, avec les [options de connexions](https://www.postgresql.org/docs/10/static/libpq-connect.html#LIBPQ-CONNECT-SSLCERT) : `sslcert` `sslkey` et `sslrootcert`. Pour cela, le plus simple est de définir un service dans le fichier `~/.pg_service.conf` :

```
[tolva]
host=tolva
sslmode=verify-full
sslcert=/home/slardiere/.postgresql/postgresql.crt
sslkey=/home/slardiere/.postgresql/postgresql.key
sslrootcert=/home/slardiere/.postgresql/root.crt
```

puis d’utiliser le service :

```
$ psql service=tolva
Enter PEM pass phrase:
psql (10.5 (Debian 10.5-1.pgdg90+1))
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
Type "help" for help.

tolva:5432 slardiere@postgres=>
```

Il est donc possible de créer autant de services que nécessaire, en indiquant les chemins vers les fichiers correspondants.

## Conclusion

En maitrisant les processus de création des clés et des certificats, il n’est pas compliqué de mettre en place des connexions sécurisées, évitant ainsi de voir des mots de passe en clair dans des fichiers, tout en contrôlant la création des certificats pour son organisation.