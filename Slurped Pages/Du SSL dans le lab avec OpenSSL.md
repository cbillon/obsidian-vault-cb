---
link: https://j.hommet.net/openssl/
byline: Julien HOMMET
site: J.HOMMET.NET
date: 2025-01-02T09:30
excerpt: Le petit cadenas vert à portée de clics.
slurped: 2025-02-20T16:25
title: Du SSL dans le lab avec OpenSSL
tags:
  - pki
  - ssl
---

Il est toujours intéressant d'avoir la possibilité de faire du chiffrement partout, dès que possible. Dans les faits, les outils exploités seront OpenSSL et Hashicorp Vault. L'outil "[xca](https://hohnstaedt.de/xca/?ref=j.hommet.net)" est aussi un bon moyen de gérer efficacement votre autorité de certificat de manière déconnectée et sans installer de serveur. Avec les technologies qui sont à notre disposition aujourd'hui, je ne m'embête plus trop et j'utilise Ollama avec les modèles Llama3.3 et Phi3 pour initier mes commandes OpenSSL.

Plutôt que de réécrire ou de copier des articles, je vous suggère la lecture de ce document provenant de la société Okta, qui présente le principe de PKI, ses fonctions et ses besoins :
[okta](https://www.okta.com/fr/identity-101/public-key-infrastructure/)
## Autorité racine (root CA)

Il est important de noter que les certificats autosignés ne sont pas reconnus publiquement et ne conviennent pas pour des sites web publics. Ce premier certificat est le plus critique, veillez à protéger la clé et le certificat le mieux possible !

1. Générez la clé privée de l'AC racine : `openssl ecparam -genkey -name prime256v1 -out root-ca.key` (_Prime256v1_ est une courbe elliptique largement utilisée et recommandée par le **NIST** (National Institute of Standards and Technology) pour des clés de 256 bits. C'est une bonne option en terme de sécurité et de performance. Cependant, vous pouvez utiliser d’autres courbes (comme `secp384r1` ou `secp521r1`) pour une sécurité accrue au prix de performances légèrement inférieures).
2. Créez le certificat autosigné de l'AC racine : `openssl req -x509 -new -nodes -key root-ca.key -sha256 -days 3650 -out root-ca.pem -subj "/C=FR/ST=Region/L=City/O=Organization/CN=Root CA" -addext "basicConstraints=critical,CA:TRUE" -addext "keyUsage=critical,digitalSignature,cRLSign,keyCertSign"`. Le certificat est valide pour 10 ans et comporte des extensions (les arguments `-addext`) qui spécifient quelles sont les capacités du certificat (entre autres).

Désormais, vous avez un certificat racine qui sera la base pour tous les autres certificats qui en découleront.

L'autorité intermédiaire doit être le second certificat généré, juste après la racine. Ce certificat se génère comme la racine, avec sa clé et ses extensions.

1. Générez la clé privée de l'AC intermédiaire : `openssl ecparam -genkey -name prime256v1 -out intermediate-ca.key`
2. Créez une demande de signature de certificat (CSR) pour l'AC intermédiaire : `openssl req -new -key intermediate-ca.key -out intermediate-ca.csr -subj "/C=FR/ST=Region/L=City/O=Organization/CN=Intermediate CA"`
3. Signez le certificat de l'AC intermédiaire avec l'AC racine : `openssl x509 -req -in intermediate-ca.csr -CA root-ca.pem -CAkey root-ca.key -out intermediate-ca.pem -days 1825 -sha256 -addext "basicConstraints=critical,CA:TRUE,pathlen:0" -addext "keyUsage=critical,digitalSignature,cRLSign,keyCertSign" -addext "authorityKeyIdentifier=keyid,issuer"`. Ce certificat sera valide cinq ans.

Si toutes les étapes sont validées, vous devriez avoir le résultat suivant :

```
Certificate request self-signature ok
subject=C=FR, ST=Region, L=City, O=Organization, CN=Intermediate CA
```

Pour créer une chaîne de certification plus facile à importer dans les navigateurs et même dans vos configurations, effectuez la commande `cat intermediate-ca.pem root-ca.pem > ca-chain.pem`, qui est la concaténation des certificats intermédiaire et racine. Pour vérifier la chaîne de certification : `openssl verify -CAfile root-ca.pem intermediate-ca.pem`. Cette commande devrait retourner "OK" si la chaîne est valide.

Ce fichier est essentiel dans toutes vos machines et navigateur web de votre réseau local, car il vous permettra de s'assurer que les demandes vers l'URL choisie sont valides et certifiées.

## Certificat wildcard

Il est important de souligner que l'utilisation d'un certificat wildcard n'est pas recommandée en dehors d'un réseau local, parce qu'il permet de valider n'importe quel sous-domaine du nom de domaine spécifié. Bien qu'il permette de faciliter la gestion des certificats (un certificat pour tous vos services), ce type de certificat peut être bloqué par certaines applications ou certains organismes, qui est parfois considéré comme "pas assez sécurisé".

Voici comment procéder avec OpenSSL :

1. Générez une clé privée : `openssl ecparam -genkey -name prime256v1 -out wildcard.key`
2. Créez une demande de signature de certificat (CSR) : `openssl req -new -key wildcard.key -out wildcard.csr -subj "/C=FR/ST=Region/L=City/O=Organization/CN=*.local.hommet.net"`. Notez que j'ai ajouté dans la partie "CN" le nom de domaine que je souhaite utiliser dans mon réseau local. N'oubliez pas de le changer en "***.**votredomaine.com ".
3. Générez le certificat autosigné : `openssl x509 -req -days 365 -in wildcard.csr -CA intermediate-ca.pem -CAkey intermediate-ca.key -CAcreateserial -out wildcard.pem -sha256 -addext "subjectAltName=DNS:*.local.hommet.net,DNS:local.hommet.net" -addext "keyUsage=critical,digitalSignature,keyEncipherment" -addext "extendedKeyUsage=serverAuth,clientAuth"`. Cette commande crée un certificat wildcard valide pour un an.

Voilà, nous avons désormais une chaîne de certification complète (autorités racine et intermédiaire, certificat _chaîne_ (intermédiaire+racine dans le même fichier) et certificat wildcard) utilisable partout dans le réseau local.

Pour que tout fonctionne dans votre réseau, vous devez ajouter le certificat _chaîne_ dans tous vos systèmes (serveurs et navigateurs).

### Exemple de certificat pour un service

Comme pour le certificat wildcard, il faut créer une clé privée, générer une CSR puis signer le certificat via l'autorité intermédiaire. Pour cet exemple, je souhaite avoir un certificat valide pour le domaine "traefik.dc.local.hommet.net" :

```
$ openssl ecparam -genkey -name prime256v1 -out traefik.dc.local.hommet.net.key
$ openssl req -new -key traefik.dc.local.hommet.net.key -out traefik.dc.local.hommet.net.csr -subj "/C=FR/ST=Region/L=City/O=Organization/CN=traefik.dc.local.hommet.net"
$ openssl x509 -req -days 365 -in traefik.dc.local.hommet.net.csr -CA intermediate-ca.pem -CAkey intermediate-ca.key -CAcreateserial -out traefik.dc.local.hommet.net.pem -sha256 -addext "subjectAltName=DNS:traefik.dc.local.hommet.net" -addext "keyUsage=critical,digitalSignature,keyEncipherment" -addext "extendedKeyUsage=serverAuth,clientAuth"
```

## Récapitulatif

```
# autorité racine
$ openssl ecparam -genkey -name prime256v1 -out root-ca.key
$ openssl req -x509 -new -nodes -key root-ca.key -sha256 -days 3650 -out root-ca.pem -subj "/C=FR/ST=Region/L=City/O=Organization/CN=Root CA" -addext "basicConstraints=critical,CA:TRUE" -addext "keyUsage=critical,digitalSignature,cRLSign,keyCertSign"

# autorité intermédiaire
$ openssl ecparam -genkey -name prime256v1 -out intermediate-ca.key
$ openssl req -new -key intermediate-ca.key -out intermediate-ca.csr -subj "/C=FR/ST=Region/L=City/O=Organization/CN=Intermediate CA"
$ openssl x509 -req -in intermediate-ca.csr -CA root-ca.pem -CAkey root-ca.key -CAcreateserial -out intermediate-ca.pem -days 1825 -sha256 -addext "basicConstraints=critical,CA:TRUE,pathlen:0" -addext "keyUsage=critical,digitalSignature,cRLSign,keyCertSign" -addext "authorityKeyIdentifier=keyid,issuer"

# chaîne de certification
$ cat intermediate-ca.pem root-ca.pem > ca-chain.pem
$ openssl verify -CAfile root-ca.pem intermediate-ca.pem

# certificat wildcard
$ openssl ecparam -genkey -name prime256v1 -out wildcard.key
$ openssl req -new -key wildcard.key -out wildcard.csr -subj "/C=FR/ST=Region/L=City/O=Organization/CN=*.local.hommet.net"
$ openssl x509 -req -days 365 -in wildcard.csr -CA intermediate-ca.pem -CAkey intermediate-ca.key -CAcreateserial -out wildcard.pem -sha256 -addext "subjectAltName=DNS:*.local.hommet.net,DNS:local.hommet.net" -addext "keyUsage=critical,digitalSignature,keyEncipherment" -addext "extendedKeyUsage=serverAuth,clientAuth"
```

Lors de la génération et de l'utilisation de fichiers avec OpenSSL, il est important de prendre certaines précautions pour assurer la sécurité :

1. **Stockez les clés privées** dans un endroit sécurisé, idéalement sur un **support chiffré et déconnecté**.
2. **Évitez** d'inclure les mots de passe directement dans les lignes de commande.
3. **Vérifiez** régulièrement les dates d'expiration des certificats
4. **Mettez en place un processus de renouvellement** des certificats avant leur expiration.
5. **Effectuez des sauvegardes** régulières des fichiers importants (clés privées)
6. Stockez les sauvegardes dans un endroit sécurisé, séparé de l'environnement de production.
7. **Limitez l'accès** aux fichiers sensibles uniquement aux utilisateurs autorisés : restreignez les permissions d'accès aux fichiers de clés privées (comme les fichiers .key) avec un `chmod 0600 *.key` et un `chown $USER: *.key`.
8. **Chiffrez** les fichiers contenant des informations sensibles lorsqu'ils ne sont pas utilisés.
9. Nettoyez de manière sécurisée les fichiers temporaires et intermédiaires après utilisation (via l'outil `shred` sous Linux par exemple).

## Créer et gérer une CRL

Une liste de révocation de certificat (Certificat Revocation List, CRL) est un fichier qui contient une liste de certificats révoqués avant leur date d'expiration. Ce fichier est essentiel pour s'assurer que les certificats invalides ou compromis ne sont pas utilisés. La CRL est signée par l'autorité de certification intermédiaire (ou racine si vous n'avez pas d'intermédiaire) afin de garantir son intégrité et son authenticité.

#### Pourquoi générer une CRL ?

La CRL permet aux serveurs et clients d'être informés des certificats révoqués et de prendre les mesures appropriées pour ne pas les accepter. Par exemple, un certificat peut être révoqué parce que la clé privée est compromise, ou parce qu'il y a une coquille dans les informations du certificat. Sans une CRL mise à jour, il est presque impossible d'identifier les certificats invalides, ce qui peut nuire à la sécurité de votre SI.

#### Génération d'une CRL avec OpenSSL

Assurez-vous que la AC racine est configurée avec un fichier (par exemple `index.txt`) qui contient la liste des certificats émis et révoqués, ainsi qu'un fichier de configuration (`openssl.cnf`) adapté.

**Vérifier la CRL** (remplacer `crl.pem` par votre fichier) :

```
openssl crl -in crl.pem -text
```

Ça affichera le contenu de la CRL, y compris les certificats révoqués, leur numéro de série et la date de révocation.

**Générer la CRL** : Une fois les certificats révoqués, vous pouvez générer la CRL en utilisant la commande suivante :

```
openssl ca -gencrl -out crl.pem
```

Cette commande crée un fichier `crl.pem` contenant la liste de tous les certificats révoqués jusqu'à ce moment. Vous pouvez ensuite distribuer cette CRL à tous les systèmes qui doivent vérifier la validité des certificats.

**Révoquer un certificat** : Avant de générer une CRL, vous devez révoquer un certificat spécifique. Pour ce faire, utilisez la commande suivante, en remplaçant le nom "`certificat.pem`" par le certificat à révoquer :

```
openssl ca -revoke certificat.pem
```

#### Recommandations pour la CRL

Il est recommandé de générer la CRL périodiquement, surtout dans les environnements dans lesquels des certificats sont régulièrement révoqués.

Pour que la CRL soit effective, elle doit être accessible par les systèmes qui valident les certificats. Cela peut être fait en publiant la CRL sur un serveur HTTP ou via un autre mécanisme de distribution. Le certificat signé de l'autorité de certification peut contenir un champ `crlDistributionPoints` pointant vers l'URL de la CRL, ce qui permettra aux clients de la récupérer automatiquement.

## Comparaison avec d'autres solutions

### OpenSSL vs XCA

XCA est plus facile à utiliser grâce à son interface graphique et sa compatibilité avec de nombreux systèmes d'exploitation. Cependant, il dispose de moins d'options de personnalisation face à OpenSSL.

### OpenSSL vs Hashicorp Vault

Vault propose une gestion centralisée des secrets (mots de passe) et des certificats, idéale en entreprise, alors qu'OpenSSL ne s'occupe que des certificats. Vault dispose d'autant d'options pour la gestion et la création de certificat comme OpenSSL. Que ce soit via l'interface web ou en ligne de commande, Vault saura être un allié de taille pour la gestion des certificats. Son accessibilité est toutefois un peu plus lourde qu'OpenSSL, une prise en main est nécessaire pour bien comprendre et exploiter l'outil.

Il existe d'autres solutions comme [EJBCA](https://www.ejbca.org/?ref=j.hommet.net), [StepCA](https://smallstep.com/docs/step-ca/?ref=j.hommet.net), [mkcert](https://github.com/FiloSottile/mkcert?ref=j.hommet.net)…

Voici une liste de commandes OpenSSL couramment utilisées, accompagnées d'exemples pratiques pour divers cas d'utilisation. OpenSSL est un outil puissant pour gérer les certificats, les clés, les CSR (Certificate Signing Requests) et bien plus encore.

## Quelques commandes utiles

- Afficher les détails de la CSR pour vérifier les informations avant de l'envoyer : `openssl req -in request.csr -noout -text`.
- Convertir un certificat PEM (texte lisible) vers DER (binaire) : `openssl x509 -in cert.pem -outform der -out cert.der`.
- Convertir un certificat DER vers PEM : `openssl x509 -in cert.der -inform der -out cert.pem`.
- Vérifier si un certificat est valide par rapport à une autorité de certification donnée : `openssl verify -CAfile chain.pem serveur.crt`.
- Afficher les détails d'un certificat, y compris sa date d'expiration et son émetteur : `openssl x509 -in serveur.crt -text -noout`.
- Obtenir l'empreinte SHA256 d'un certificat pour le vérifier ou l'importer dans des applications : `openssl x509 -in serveur.crt -noout -fingerprint -sha256`.
- Concaténer les certificats intermédiaire et racine pour former une chaîne de certification : `cat intermediate.pem root.pem > chain.pem`.
- Vérifier les certificats et la configuration SSL/TLS d'un serveur web : `openssl s_client -connect exemple.com:443 -servername exemple.com`.
- Protéger un fichier avec un chiffrement AES-256 :
    - **Chiffrement** : `openssl enc -aes-256-cbc -salt -in` fichier.txt -out fichier.txt.enc
    - **Déchiffrement** :`openssl enc -aes-256-cbc -d -in` fichier.txt.enc -out fichier.txt
- Générer une chaîne aléatoire hexadécimale de 16 octets pour une clé ou un nonce : `openssl rand -hex 16`.
- Obtenir l'empreinte SHA256 d'un fichier : `openssl dgst -sha256 fichier.txt`.
- Signer un fichier et vérifier son authenticité avec une paire de clés publique/privée :
    - **Création** : `openssl dgst -sha256 -sign private.key -out signature.bin fichier.txt`.
    - **Vérification** : `openssl dgst -sha256 -verify public.key -signature signature.bin fichier.txt`.

## Et maintenant ?

Diffusez et importez le certificat "chaine" dans vos systèmes, créer les certificats pour vos services qui en ont besoin, notez la date d'expiration des certificats et créez des rappels pour les renouveler (au moins 10 jours avant l'expiration).