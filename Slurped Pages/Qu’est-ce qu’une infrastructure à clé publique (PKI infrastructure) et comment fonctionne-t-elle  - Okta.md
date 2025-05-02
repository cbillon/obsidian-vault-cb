---
link: https://www.okta.com/fr/identity-101/public-key-infrastructure/
excerpt: Apprenez-en plus sur l'infrastructure à clé publique (PKI infrastructure) avec Okta et découvrez comment gérer le chiffrement à clé publique.
tags:
  - pki
  - infrastructure
slurped: 2025-02-20T16:01
title: Qu’est-ce qu’une infrastructure à clé publique (PKI infrastructure) et comment fonctionne-t-elle ? | Okta
---

## L'importance et l'utilisation des certificats électroniques

Les certificats électroniques jouent un rôle crucial dans la sécurisation des communications numériques. Ils servent à authentifier l'identité des utilisateurs et des appareils, assurant ainsi la confidentialité et l'intégrité des échanges en ligne. En utilisant des protocoles tels que TLS/SSL, les certificats permettent le chiffrement des données transmises, offrant une protection efficace contre les cyberattaques et les interceptions malveillantes.

Pour rendre possible l'utilisation et la gestion de ces certificats, une infrastructure à clé publique (PKI) est essentielle.

Une infrastructure à clé publique, ou PKI (Public Key Infrastructure), regroupe tous les éléments utilisés pour établir et gérer le chiffrement à clé publique. Elle englobe les logiciels, le matériel, ainsi que les politiques et procédures utilisés pour créer, distribuer, gérer, stocker et révoquer les certificats numériques. 

Un certificat numérique associe une [clé publique](https://www.okta.com/identity-101/public-key-encryption/) cryptographique au terminal ou à l’utilisateur qui la détient. Cela permet d’authentifier les utilisateurs et les terminaux, et de sécuriser les communications numériques. 

L’infrastructure à clé publique est l’une des formes les plus courantes du chiffrement Internet, et permet de sécuriser et d’authentifier le trafic entre les navigateurs et les serveurs web. Elle peut également servir à sécuriser l’accès aux terminaux connectés et les communications internes au sein d’une organisation. 

L’infrastructure à clé publique a été développée pour sécuriser et authentifier les communications numériques avec [un double objectif](https://www.computerweekly.com/feature/Why-public-key-infrastructure-is-a-good-idea) : permettre la confidentialité du message envoyé et vérifier que l’expéditeur est bien celui qu’il prétend être.

Une infrastructure PKI joue un rôle important dans la sécurité d’Internet. Elle comprend une série de technologies et processus constituant un framework de chiffrement destiné à protéger et authentifier les communications numériques.

Une PKI utilise des clés publiques cryptographiques associées à un certificat numérique, lequel authentifie le terminal ou l’utilisateur qui envoie la communication numérique. Les certificats numériques sont émis par une source fiable, l’autorité de certification, et font fonction de carte d’identité numérique permettant de vérifier et confirmer l’identité de l’expéditeur.

L’infrastructure PKI protège et authentifie les communications entre les serveurs et les utilisateurs, par exemple entre votre site web (hébergé sur votre serveur web) et vos clients (les utilisateurs tentant de se connecter via leur navigateur). Elle permet également de sécuriser les communications au sein d’une organisation et de s’assurer que les messages sont uniquement visibles par l’expéditeur et le destinataire et qu’ils n’ont pas été falsifiés en transit. 

Une infrastructure PKI comprend plusieurs composants importants, dont les suivants :

- **Autorité de certification (CA) :** l’autorité de certification est l’entité de confiance qui émet, stocke et signe le certificat numérique. Elle signe le certificat numérique avec sa propre clé privée, puis publie la clé publique qui est, elle, accessible sur demande.
- **Autorité d’enregistrement (RA) :** l’autorité d’enregistrement vérifie l’identité de l’utilisateur ou du terminal demandant le certificat numérique. Il peut s’agir d’une tierce partie, mais aussi de l’autorité de certification. 
- **Base de données de certificats :** celle-ci stocke les certificats numériques et leurs métadonnées, y compris la durée de validités des certificats.
- **Référentiel central :** il s’agit de l’emplacement sécurisé où les clés cryptographiques sont indexées et stockées.  
- **Système de gestion des certificats :** ce système gère la distribution des certificats ainsi que leur accès. 
- **Politique de certification :** cette politique décrit les procédures de l’infrastructure PKI. Elle peut être consultée par des utilisateurs externes pour déterminer la fiabilité de la PKI.

## Principe de fonctionnement d’une infrastructure PKI

Une infrastructure à clé publique (PKI) utilise des méthodes de chiffrement asymétrique pour assurer la confidentialité des messages, mais aussi pour authentifier le terminal ou l’utilisateur à l’origine de la transmission. 

Le chiffrement asymétrique prévoit l’utilisation d’une clé publique et d’une clé privée. Une clé cryptographique est une longue chaîne de bits utilisés pour chiffrer les données. 

La clé publique est accessible à quiconque la demande et est émise par une autorité de certification de confiance. Cette clé publique vérifie et authentifie l’expéditeur du message chiffré.

La seconde partie d’une paire de clés cryptographiques utilisée dans une infrastructure PKI est la clé privée, ou secrète. Le destinataire est le seul à posséder cette clé, qui est utilisée pour déchiffrer la transmission. 

Le chiffrement et le déchiffrement des paires de clés publique/privée emploient des algorithmes complexes. La clé publique authentifie l’expéditeur du message électronique tandis que la clé privée garantit que seul le destinataire peut l’ouvrir et le lire.

## Certificats PKI

Une infrastructure PKI repose avant tout sur la confiance. Pour une entité destinataire, il est important de n’avoir aucun doute quant à l’identité de l’expéditeur du certificat numérique. 

Les autorités de certification tierces de confiance peuvent se porter garantes de l’expéditeur et donner la preuve qu’il est effectivement celui qu’il prétend être. Les certificats numériques servent à vérifier les identités numériques. 

On les appelle aussi _certificats PKI_ ou _certificats X.509_. Un certificat PKI apporte la preuve de l’identité à l’entité demandeuse, ce certificat étant vérifié par une tierce partie et fonctionnant comme une carte d’identité électronique. 

Le certificat PKI contient les informations suivantes :

- Nom unique du propriétaire
- Clé publique du propriétaire
- Date d’émission
- Date d’expiration
- Nom unique de l’autorité de certification émettrice
- Signature numérique de l’autorité de certification émettrice

## À quoi sert une infrastructure PKI ?

Une infrastructure PKI est très souvent utilisée avec le protocole TLS/SSL (Transport Layer Security/Secure Socket Layer), qui sécurise les communications HTTP (Hypertext Transfer Protocol) chiffrées. 

Les propriétaires de sites web obtiennent un certificat numérique auprès d’une autorité de certification de confiance. Pour le recevoir, le propriétaire du site web doit prouver son identité. Une fois celle-ci vérifiée, le propriétaire du site web peut acheter un certificat SSL à installer sur le serveur web. Ce certificat indique au navigateur que le site auquel il tente d’accéder est bien le site web légitime. 

Le protocole TLS/SSL s’appuie sur une chaîne de confiance, où l’utilisateur doit faire confiance à l’autorité chargée de l’émission du certificat racine. Une autre solution consiste à établir un réseau de confiance qui utilise des certificats autosignés, validés par un tiers. Ce réseau de confiance est souvent utilisé au sein de plus petites communautés d’utilisateurs, par exemple au sein du réseau autonome d’une organisation.

Une infrastructure PKI peut être également utilisée à d’autres fins :

- Chiffrement des e-mails et authentification de l’expéditeur
- Signature de documents et de logiciels
- Utilisation des serveurs de bases de données pour sécuriser les communications internes
- Sécurisation des communications web, par exemple pour l’e-commerce
- Authentification et chiffrement de documents
- Sécurisation des réseaux locaux et authentification par carte à puce
- Chiffrement et déchiffrement de fichiers
- Limitation de l’accès aux VPN et intranets d’entreprise
- Communication sécurisée entre des terminaux de confiance, par exemple des terminaux IoT (Internet des Objets)

## Types d'infrastructures PKI open source

Une infrastructure PKI accessible au public est dite « open source ». En voici quelques exemples :

- [EJBCA Enterprise](https://www.primekey.com/products/ejbca-enterprise/) : développée en langage Java pour les entreprises et capable d’implémenter une autorité de certification aux fonctionnalités complètes, la solution peut configurer une autorité de certification en tant que service ou pour une utilisation interne.
- [OpenSSL](https://www.openssl.org/) : toolkit commercial riche en fonctionnalités, il est inclus dans toutes les distributions majeures de Linux et développé en langage C. Il peut offrir des fonctionnalités PKI aux applications et être utilisé pour créer une autorité de certification simple.
- [CFSSL](https://github.com/cloudflare/cfssl) : il s’agit du toolkit PKI/SSL de CloudFlare, destiné à signer, vérifier et regrouper des certificats TLS et à créer des outils PKI TLS personnalisés.
- [XiPKI](https://github.com/xipki/xipki) : très performant et évolutif, ce répondeur OCSP et CA est implémenté en Java avec prise en charge de SHA-3.
- [Dogtag Certificate System](https://www.dogtagpki.org/wiki/PKI_Main_Page) : il s’agit d’une autorité de certification pour entreprises, dotée de fonctionnalités très complètes et prenant en charge tous les aspect de la gestion du cycle de vie des certificats.

## Ressources supplémentaires

Les principaux navigateurs et systèmes d’exploitation, par exemple [Apple](https://support.apple.com/en-us/HT209143) et [Microsoft](https://docs.microsoft.com/en-us/security/trusted-root/participants-list), publient des magasins de confiance qui fournissent une liste de certificats racine de confiance. Un certificat racine de confiance est nécessaire pour que les utilisateurs fassent confiance au certificat numérique fourni et à son autorité de certification (CA). Une CA de confiance constitue un aspect essentiel de l’infrastructure à clé publique. 

Celle-ci utilise la cryptographie asymétrique pour chiffrer et déchiffrer les messages électroniques. Pour toute information ou question sur le chiffrement asymétrique et l’utilisation des clés cryptographiques publiques et privées, [Okta](https://www.okta.com/identity-101/asymmetric-encryption/) est là pour vous aider. N’hésitez pas à nous contacter.

## Références

[Why Public Key Infrastructure Is a Good Idea](https://www.computerweekly.com/feature/Why-public-key-infrastructure-is-a-good-idea). Mars 2001. _Computer Weekly_.

[EJBCA Enterprise](https://www.primekey.com/products/ejbca-enterprise/). 2022. PrimeKey AB.

[OpenSSL](https://www.openssl.org/). 2021. The OpenSSL Project Authors.

[Cloudflare/CFSSL](https://github.com/cloudflare/cfssl). 2022. GitHub, Inc.

[Xipki/xipki](https://github.com/xipki/xipki). 2022. GitHub, Inc.

[PKI Main Page](https://www.dogtagpki.org/wiki/PKI_Main_Page). Dogtag PKI.

[Available Trusted Root Certificates for Apple Operating Systems](https://support.apple.com/en-us/HT209143). 2022. Apple, Inc.

[List of Participants – Microsoft Trusted Root Program](https://docs.microsoft.com/en-us/security/trusted-root/participants-list). Décembre 2021. Microsoft Build.

[Chiffrement asymétrique : définition, architecture et utilisation](https://www.okta.com/identity-101/asymmetric-encryption/). 2022. Okta.

[Public vs. Private: Unlocking the Full Potential of Public Key Infrastructure](https://www.forbes.com/sites/forbestechcouncil/2021/12/08/public-vs-private-unlocking-the-full-potential-of-public-key-infrastructure/?sh=6dcfbe725849). Décembre 2021. _Forbes_.