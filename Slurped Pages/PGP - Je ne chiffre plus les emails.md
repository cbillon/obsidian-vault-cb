---
link: https://9x0rg.com/posts/tech/pgp-je-ne-chiffre-plus-les-emails/
byline: Olivier Falcoz
site: 9x0rg
date: 2024-12-10T23:03
excerpt: >-
  Public key cryptography1 - xkcd

  Contexte [Spoiler]: Avant que le communauté des utilisateurs irréductibles de
  PGP2 ne me saute dessus à bras raccourcis, laissez-moi vous donner quelques
  éléments de contexte.

  J&rsquo;utilise le chiffrement PGP depuis bientôt 20 ans, une des rares
  solutions qui permettait de se soustraire à la curiosité envahissante de nos
  amis Chinois quand j&rsquo;habitais en Chine3. C&rsquo;est à la même période
  d&rsquo;ailleurs et pour des raisons similaires que j&rsquo;abandonne Windows
  et bascule mon environnement de travail sous Linux.
tags:
  - tech
  - encryption
  - PGP
  - data-privacy
slurped: 2024-12-19T09:00
title: PGP - Je ne chiffre plus les emails
---

![Posting my PGP key for 15 years](app://obsidian.md/images/xkcd-pgp-public-key.png "I guess I should be signing stuff, but I've never been sure what to sign. Maybe if I post my private key, I can crowdsource my decisions about what to sign.") Public key cryptography[1](#fn:1) - [xkcd](https://www.explainxkcd.com/wiki/index.php/1553:_Public_Key)

## Contexte

> [Spoiler]: Avant que le communauté des utilisateurs irréductibles de PGP[2](#fn:2) ne me saute dessus à bras raccourcis, laissez-moi vous donner quelques éléments de contexte.

J’utilise le chiffrement PGP depuis bientôt 20 ans, une des rares solutions qui permettait de se soustraire à la curiosité envahissante de _nos amis_ Chinois quand j’habitais en Chine[3](#fn:3). C’est à la même période d’ailleurs et pour des raisons similaires que j’abandonne Windows et bascule mon environnement de travail sous Linux.

Au sein de la société qui m’emploie dans les années 2010 en Asie du Sud-Est, le recours au chiffrement des pièces jointes est systématique - au format `.txt`, pas MS Word bien sûr. Expliquer à nos client comment installer et utiliser un logiciel de chiffrement prend parfois du temps mais on y parvient. Tout le monde jongle avec les clés de chacun - celles dédiées aux déplacements, celles qui sont égarées, celles dont les mots de passe sont perdus, celles qui sont créées sans prévenir… Mais ça fonctionne. Non sans quelques frictions et grognements.

L’apparition des messageries instantanées, WhatsApp, Telegram brièvement puis [Signal](https://signal.org/) révolutionne les échanges et sonne - enfin - le glas des pièces jointes chiffrées envoyées par email. J’entretiens néanmoins mon petit parc de clés PGP et utilise des clients PGP-compatibles tels que [Thunderbird](https://www.thunderbird.net/) (desktop) et [OpenKeyChain](https://www.openkeychain.org/) avec K9 puis [Fairemail](https://email.faircode.eu/) (Android).

Bon élève, je suis les bonne pratiques[4](#fn:4) à la lettre: je bidouille le fichier `gpg.conf`, sélectionne Curve25519 et Ed25519, je deviens un virtuose de la CLI `gpg2 --expert --full-gen-key` pour séparer la clé de certification de ses sous-clés de chiffrement et signature, j’exporte la clé vers le mobile en ôtant la clé principale, choisis des durées de validité courtes, je les renouvelle, les publie sur des [serveurs de clé](https://wiki.archlinux.org/title/OpenPGP#Keyserver), conserve soigneusement les certificats de révocation dans un container Veracrypt, etc… Bref, les utilisateurs avertis de PGP se reconnaîtront, tout ceci est une vraie plaie à gérer.

Puis, au fur et à mesure des années, abstraction faite de quelques irréductibles, le volume des _emails utiles_ chiffrés en PGP diminue pour tout simplement tomber à **zéro**. Je ne pense pas avoir reçu une seul email spontanément chiffré _à dessein_ par son émetteur depuis 3 ou 4 ans.

Comme personne ne les utilise plus, j’arrête moi aussi de chiffrer et/ou de signer les emails en PGP.

En conséquence, si vous tombez sur les clés `ed25519/0xCAAD364477DA43C8` et `ed25519/0xF509244B5E236B2C`, sachez qu’elles sont désormais révoquées.

## PGP - une impasse technologique?

Le chiffrement PGP existe depuis 20 ans et il est toujours aussi complexe et pénible à utiliser. Quand bien même y parviendrait-on, GPG présente des défauts structurels qui n’ont pas été surmontés depuis sa création. Le sujet est assez bien documenté…

**Moxie Marlinspike** (le fondateur de Signal) note[5](#fn:5) que _la technologie elle-même, accusant son âge (les années 90) sur le plan de la philosophie de conception, est obsolète et que le protocole PGP ne laisse pas non plus de place à des concepts aujourd’hui essentiels tels que la [forward secrecy](https://en.wikipedia.org/wiki/Forward_secrecy)._

_Les clés PGP sont nulles voire dangereuses, leur gestion manuelle est une hérésie, le format OpenPGP et les valeurs par défaut craignent, les clients de messagerie sont horribles et la _forward secrecy_ n’existe pas_ assène non sans malice[6](#fn:6) **Matthew Green**, ajoutant qu’_un détracteur de PGP n’est qu’un utilisateur de PGP qui a utilisé le logiciel pendant un certain temps_.

_Il ne s’agit pas de l’outil PGP lui-même, ni des outils en général mais du modèle de clé PGP à long terme qui m’a déçu_ constate[7](#fn:7) **Filippo Valsorda**, très fin connaisseur du protocole. _Tout d’abord, il y a la question de l’adoption. Ensuite, il y a le problème de l’interface utilisateur, des erreurs faciles à commettre, ainsi que des listes de serveurs de clés désordonnées qui remontent à plusieurs années. Mais la vraie question est celle de la sécurité des clés à long terme. Une clé à long terme n’est aussi sûre que le plus petit dénominateur commun de vos pratiques de sécurité au cours de sa durée de vie. C’est le maillon faible_.

_Aucun ingénieur compétent en cryptographie ne concevrait un système qui ressemblerait à PGP aujourd’hui, ni ne tolérerait la plupart de ses défauts dans une autre application_, relève[8](#fn:8) le Blog de **Latacora**, soulignant _sa complexité absurde, son design de couteau suisse faisant un tas de choses mais aucune correctement, son aspect embourbé dans la rétrocompatibilité, son authentification défaillante et une gestion des identités incohérente, des clés lourdingues, un code farfelu et des négociation hasardeuses entre des algos pléthoriques_. Et de rajouter que _PGP fait fuiter les métadonnées et est démuni de forward secrecy_.

N’en jetez plus la coupe est… Ah, on continue?

_L’email n’est pas sûr et ne peut pas être sécurisé. Les outils dont nous disposons aujourd’hui pour chiffrer les emails sont très imparfaits. Même si ces défauts étaient corrigés, l’email resterait dangereux. Il n’est pas possible de résoudre ces problèmes de manière satisfaisante_ répète[9](#fn:9) le Blog de **Latacora**. _Les métadonnées sont aussi importantes que le contenu et le courrier électronique les fait fuir. Tous les messages archivés finissent par fuir. Tout secret à long terme finit par être dévoilé_.

Et pour couronner le tout, des divergences de _standards_ PGP sont en train d’apparaître à la suite d’un schisme[10](#fn:10) entre OpenPGP et GnuPG comme l’explique ArchLinux dans son Wiki[11](#fn:11):

> **Avertissement** : GnuPG a débuté comme une implémentation du format OpenPGP. Cependant, ces dernières années, son mainteneur s’est activement écarté de l’effort de standardisation d’OpenPGP et étend séparément le format d’une manière spécifique à GnuPG (voir draft-koch-librepgp). Ces changements entraînent des problèmes de compatibilité avec d’autres implémentations depuis la version 2.4.

CQFD/QED, j’adhère et plussoie.

Désormais, je vais privilégier des protocoles qui garantissent la _forward secrecy_ des communications sensibles, utiliser des clés courtes et solides en Ed25519/ChaCha20-Poly1305, à la durée de vie réduite, aux algorithmes récents et tant pis pour _Quid de la rétro-compatibilité pour lire les vieux emails chiffrés en 2010_.

## Alternative à PGP

### Chiffrer les emails

Don’t. Nope. Nada. Fini. J’ai dit non.

[Contactez-moi](https://9x0rg.com/about/#contact) par le moyen de votre choix (plaintext email, Matrix, XMPP) et je vous fournirai mon Signal _username_.

### Signer des données/packages

Utilisez [Minisign](https://github.com/jedisct1/minisign) de Franck Denis, le type derrière Libsodium ou [Signify](https://man.openbsd.org/signify) par Ted Unangst, celui d’[OpenBSD](app://obsidian.md/(https://www.openbsd.org/papers/bsdcan-signify.html)).

### Chiffrer des fichiers/dossiers

Utilisez [age encryption](https://github.com/FiloSottile/age) un outil de chiffrement simple écrit en Go, moderne et sécurisé avec des clés courtes et explicites (ChaCha20-Poly1305), pas d’options de configuration. Voir aussi [Awesome age](https://github.com/FiloSottile/awesome-age), un écosystème construit autour de age encryption (implémentations, plugins, CLI, GUI, outils).

Utilisez [Tomb](https://github.com/dyne/tomb), un outil minimaliste en ligne de commande basé sur `dm-crypt` et [LUKS](https://gitlab.com/cryptsetup/cryptsetup/-/blob/main/README.md) (Linux Unified Key Setup) pour créer des containers à la Veracrypt.

Utilisez [Picocrypt](https://github.com/Picocrypt/Picocrypt) un tout petit utilitaire tout simple de chiffrement (XChaCha20 cipher et Argon2id pour la dérivation des clés) qui fonctionne sous Linux, macOS, Windows.

### Transmettre des fichiers

Utilisez [Magic Wormhole](https://github.com/warner/magic-wormhole). Les [clients](https://magic-wormhole.readthedocs.io/en/latest/ecosystem.html#end-user-client-applications) Magic Wormhole utilisent la méthode _password-authenticated key exchange_ (PAKE) pour chiffrer les fichiers transmis entre destinataires. C’est facile (du moins pour les _nerds_), sécurisé et cool (Android, iOS, desktop).

Utilisez le moyen de votre choix après avoir zippé et chiffré les données avec [age encryption](https://github.com/FiloSottile/age) ou [Picocrypt](https://github.com/Picocrypt/Picocrypt).

### Chiffrer des sauvegardes

Utilisez [BorgBackup](https://github.com/borgbackup/borg), les données sont chiffrées côté client en 256-bit AES et l’intégrité et l’authenticité des données vérifiées à l’aide de HMAC-SHA256.

### Chiffrer des données d’application

Utilisez la librairie d’applications [libsodium](https://github.com/jedisct1/libsodium). Voir aussi [les projets](https://doc.libsodium.org/libsodium_users) utilisant libsodium.

## Addendum

Il se peut que je conserve une clé PGP au cas où un _irréductible des moyens de transmission électroniques décentralisés et libres (l’email en bref) qui soit **et** réfractaire à Signal **et** aux smartphone_ (si si, j’en connais) veuille ab-so-lu-ment me contacter en chiffré, mais je ne promets rien.