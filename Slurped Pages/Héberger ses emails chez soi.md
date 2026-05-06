---
link: https://www.linkedin.com/pulse/h%C3%A9berger-ses-emails-chez-soi-en-2026-une-tr%C3%A8s-id%C3%A9e-que-beaufrere-gilie/?trackingId=f2CrQOi4SCKLVw%2FdxPkeuA%3D%3D
byline: Cyril Beaufrere
site: "@LinkedInEditors"
date: 2026-04-13T11:30
excerpt: "J'ai eu maintes fois l'occasion d'administrer des serveurs de
  messagerie. Professionnellement, historiquement, dans des contextes très
  différents : mutualisé, dédié, on-prem, cloud, avec des appliances, avec du
  Postfix à la main, avec des solutions clés en main, avec des contraintes de
  conformité, d"
twitter: https://twitter.com/@LinkedInEditors
slurped: 2026-04-18T14:59
title: "Héberger ses emails chez soi, en 2026 : une très mauvaise idée… que j'ai
  quand même voulu essayer"
---

J'ai eu maintes fois l'occasion d'administrer des serveurs de messagerie. Professionnellement, historiquement, dans des contextes très différents : mutualisé, dédié, on-prem, cloud, avec des appliances, avec du Postfix à la main, avec des solutions clés en main, avec des contraintes de conformité, de sécurité, de volumétrie... Pourtant, jamais, en plus de 25 ans de vie pro je n'avais réellement hébergé mes propres emails à la maison, sur ma propre connexion Internet, avec mon propre nom de domaine, sans prestataire mail en bout de chaîne.

Et avec le recul, je peux le dire sans détour : si j'avais su à quel point ce serait pénible, chronophage, et parfois complètement wtf… je ne me serais probablement jamais lancé.

Cela étant dit, c'est précisément pour cette raison que l'expérience est intéressante.

Je précise d'emblée deux choses. D'abord, certains lecteurs bien plus pointus que moi sur le sujet souriront probablement à la lecture de certaines approximations, raccourcis ou décisions discutables. Je leur demande par avance un peu d'indulgence : cela fait longtemps que je ne suis plus hands-on sur le mail, et ce retour d'expérience est volontairement sincère, boulettes comprises. Ensuite, tout ce qui suit repose sur une architecture fonctionnelle, éprouvée, utilisée réellement, pas sur un montage théorique.

L'objectif est simple : recevoir ses emails chez soi, envoyer ses emails sans finir systématiquement en spam, comprendre pourquoi chaque brique existe, et assumer les compromis.

---

Le problème fondamental : pourquoi héberger du mail chez soi est devenu si difficile

Techniquement, recevoir des emails reste étonnamment simple. SMTP, malgré son âge vénérable - le protocole date de 1982, RFC 821 - fonctionne encore très bien. Ouvrir un port 25, annoncer un MX, accepter des connexions entrantes, stocker des messages : rien de révolutionnaire. Le protocole est bavard, texte clair, profondément rétrocompatible. C'est l'une des rares technos d'Internet qui n'a pas fondamentalement changé en quarante ans.

Ce qui a profondément changé, en revanche, c'est l'envoi.

Depuis une quinzaine d'années, l'écosystème mail s'est radicalement durci. Les grands acteurs (Google, Microsoft, Yahoo etc) ont imposé des règles de plus en plus strictes pour lutter contre le spam, le phishing et l'usurpation d'identité. Ces règles ne sont pas arbitraires : elles sont documentées, normées, souvent basées sur des RFC, mais leur implémentation réelle est impitoyable. Et le paradoxe cruel, c'est que les règles elles-mêmes sont publiques et accessibles à tous, mais leur pondération, leur seuil d'acceptation, leur système de scoring interne, ça, c'est une boîte noire, on joue un jeu dont on ne connaît pas toutes les règles.

Aujourd'hui, envoyer un email "qui arrive" nécessite au minimum :

- une adresse IP avec une réputation correcte - ce qui sous-entend une IP qui n'a jamais été utilisée pour du spam, jamais listée sur une blacklist, idéalement "vieillie" avec un historique propre,
- un reverse DNS cohérent - c'est-à-dire que l'IP doit résoudre vers un nom d'hôte qui lui-même résout vers cette IP,
- un SPF strict et aligné - la déclaration DNS doit correspondre exactement à l'IP émettrice,
- une signature DKIM valide - chaque message doit être signé cryptographiquement,
- une politique DMARC crédible - le domaine doit indiquer ce qu'il faut faire en cas de non-conformité SPF ou DKIM,
- une cohérence totale entre le serveur qui parle SMTP et l'identité qu'il revendique - banner SMTP, EHLO, From, Return-Path : tout doit s'aligner.

Et c'est là que la connexion Internet résidentielle devient un enfer. L'IP est dynamique ou semi-statique : elle change, ou peut changer, et la réputation qui lui est attachée n'est pas celle qu'on a construite. Le reverse DNS est impossible à modifier : c'est l'opérateur qui le gère, et il pointe en général vers un hostname générique du style 1-2-3-4.dyn.example-isp.fr ce qui est un signal quasi universel de "connexion résidentielle suspecte". Les plages IP résidentielles sont parfois massivement listées ou simplement dépriorisées par les grands opérateurs. Le port 25 sortant est souvent filtré par défaut chez les FAI grand public, sans possibilité de dérogation simple, et l'IPv6, même quand il est disponible, est souvent mal configuré côté reverse DNS, créant des incohérences supplémentaires.

C'est précisément pour cette raison qu'un VPS entre en jeu.

---

Pourquoi un VPS est indispensable dans cette architecture

Le VPS n'est pas là pour faire "plus propre". Il est là pour rendre l'envoi possible - point.

Dans mon architecture finale, le VPS joue un rôle extrêmement précis : il est MX frontal sortant et entrant, point d'entrée public du monde SMTP, garant de la réputation IP, détenteur du reverse DNS, et intermédiaire de confiance entre Internet et mon réseau domestique. Toute la valeur de la délivrabilité repose sur lui.

Le serveur mail "complet", celui qui stocke les boîtes, expose IMAP, gère l'authentification, le webmail, les filtres, tourne à la maison - plus précisément sur une VM Proxmox dédiée. Mais il ne parle jamais directement à Gmail, à Outlook, ou à quiconque.

Le flux est volontairement simple :

Internet
   |
   | SMTP (25)
   v
[VPS MX frontal - Postfix]
   |
   | SMTP authentifié / IP restreinte
   v
[VM Proxmox - Mailcow]        

Ce découplage permet plusieurs choses fondamentales.

Le VPS a une IP propre, dans un datacenter, avec un reverse DNS entièrement sous contrôle. C'est une IP "professionnelle", pas résidentielle. Elle peut être choisie en fonction de son historique, vérifiée avant utilisation sur les principales blacklists.

Le VPS est le seul serveur autorisé à émettre vers l'extérieur. Il est le seul déclaré dans le SPF. C'est lui qui relaie les signatures DKIM faites en amont par Mailcow.

Le serveur à la maison n'accepte les emails entrants que depuis le VPS. On verrouille l'accès par IP ou par authentification SASL, ce qui évite qu'une mauvaise configuration expose directement la machine domestique à Internet.

En cas de panne domestique (coupure, reboot, maintenance de Proxmox) le VPS peut temporairement mettre en file d'attente les messages entrants. Postfix tente de relivrer jusqu'à cinq jours par défaut. Une fenêtre de maintenance raisonnable devient soudain beaucoup plus sereine.

C'est une architecture vieille comme le mail lui-même - on l'appelait autrefois "smart host" ou "relay host". Ce qui est nouveau, c'est qu'elle est devenue quasi obligatoire pour un particulier, là où elle était autrefois optionnelle et réservée aux organisations sans IP fixe propre.

---

Le DNS : colonne vertébrale invisible mais impitoyable

Avant même d'installer quoi que ce soit, avant de toucher à un fichier de configuration, avant d'ouvrir le moindre port : le DNS conditionne tout. C'est lui qui dit au monde qui vous êtes, qui parle en votre nom, et si ce que vous prétendez être est cohérent avec ce que vous faites.

Le domaine (appelons-le example.com) doit annoncer clairement qui reçoit le mail, qui a le droit d'en envoyer, et comment vérifier que personne n'usurpe l'identité.

Le premier enregistrement est le MX :

example.com.    IN MX 10 mx1.example.com.        

Ce MX pointe exclusivement vers le VPS. Jamais vers la maison. La maison n'existe pas dans le DNS public - elle est invisible depuis Internet, et c'est voulu.

Ensuite, mx1.example.com doit résoudre vers l'IP publique du VPS :

mx1.example.com. IN A 203.0.113.42        

Et le reverse DNS de cette IP doit, de son côté, pointer vers mx1.example.com. Ce reverse DNS (appelé PTR record) est géré par l'hébergeur du VPS, pas par le registrar du domaine. C'est une subtilité que beaucoup ratent au premier passage. La plupart des hébergeurs VPS permettent de le configurer depuis leur panel. C'est une exigence implicite, non écrite dans une RFC unique, mais appliquée partout sans exception. Sans reverse cohérent, la réputation chute immédiatement et certains serveurs rejettent même la connexion sans aller plus loin.

Une autre subtilité souvent négligée : le TTL des enregistrements DNS mail. Pendant la phase de mise en place, on est tenté de faire des changements fréquents. Un TTL bas (300 secondes) permet ça mais une fois en production, il faut le remonter. Un TTL trop bas peut être interprété comme un signe de configuration instable ou éphémère, ce qui n'est pas bon pour la réputation.

---

SPF : dire explicitement qui a le droit de parler pour votre domaine

SPF (Sender Policy Framework, RFC 7208) est souvent mal compris, et cette incompréhension génère des configurations bancales. Ce n'est pas un mécanisme de sécurité cryptographique. C'est une déclaration d'intention, publiée dans le DNS, lisible par n'importe qui.

Quand un serveur reçoit un email prétendant venir de @example.com, il va interroger le DNS pour savoir si l'IP émettrice est autorisée à parler au nom de ce domaine. Ce n'est pas le contenu du message qui est vérifié - c'est l'IP de connexion SMTP, celle qui apparaît dans l'enveloppe, pas forcément dans le header From visible par l'utilisateur. C'est le MAIL FROM de la session SMTP, aussi appelé "envelope sender" ou "Return-Path".

Dans notre cas, c'est simple : seul le VPS a le droit d'envoyer.

example.com. IN TXT "v=spf1 ip4:203.0.113.42 -all"        

Le -all est important. Il signifie : toute autre IP doit être rejetée. Beaucoup préfèrent ~all (softfail) par prudence - les messages d'IP non autorisées sont marqués suspects mais pas rejetés. C'est une position défensive compréhensible en phase de transition, mais à terme, ~all envoie un signal de faiblesse. -all dit clairement : je sais exactement qui envoie en mon nom.

À ce stade, le serveur mail à la maison ne doit surtout pas apparaître dans le SPF. S'il envoyait directement depuis son IP résidentielle - par erreur de configuration, par boucle, par test mal calibré - il serait rejeté ou marqué, et cela impacterait la délivrabilité globale du domaine.

Une erreur courante : multiplier les include: dans le SPF. Chaque include: déclenche une requête DNS supplémentaire au moment de la vérification. La RFC limite à 10 lookups DNS. Au-delà, le SPF est considéré comme invalide - avec les mêmes conséquences qu'un SPF qui échoue. Garder le SPF minimal n'est pas qu'une question d'hygiène : c'est une contrainte technique réelle.

---

DKIM : signer cryptographiquement chaque message

SPF dit "qui a le droit d'envoyer". DKIM (DomainKeys Identified Mail, RFC 6376) dit "ce message n'a pas été modifié en transit et provient bien d'une entité contrôlant ce domaine".

DKIM repose sur une paire de clés asymétriques. La clé privée reste sur le serveur qui signe, dans mon cas, Mailcow sur la VM Proxmox à la maison. La clé publique est publiée dans le DNS, accessible à tous les serveurs de réception.

Un exemple d'enregistrement DKIM :

dkim._domainkey.example.com. IN TXT "v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFA..."        

Le préfixe dkim est le sélecteur - un identifiant arbitraire qui permet d'avoir plusieurs clés DKIM actives simultanément (utile lors d'une rotation de clés). Chaque email sortant est signé avec la clé privée. Le serveur de réception récupère la clé publique via DNS et vérifie la signature. Si la signature est valide, il sait que le message n'a pas été altéré entre la signature et la réception.

Ce qui est signé, concrètement : un sous-ensemble des headers (From, To, Subject, Date…) et le corps du message, hashé. Toute modification, même l'ajout d'un ****** d'espace invalide la signature. C'est pour ça que certains équipements anti-spam qui modifient les messages en transit peuvent casser les signatures DKIM, ce qui crée des problèmes bien connus dans les architectures mail en chaîne.

La taille de clé recommandée est aujourd'hui 2048 bits minimum. Les clés RSA 1024 bits sont progressivement refusées ou ignorées par les grands opérateurs. Certains commencent à expérimenter l'Ed25519 pour DKIM, plus compact et plus moderne, mais le support universel n'est pas encore garanti.

DKIM est aujourd'hui absolument indispensable. Un mail sans DKIM valide est suspect par défaut. Chez Gmail notamment, l'absence de signature DKIM est un signal fort qui aggrave significativement le score de spam.

---

DMARC : l'arbitre final

DMARC (Domain-based Message Authentication, Reporting and Conformance, RFC 7489) ne fait rien tout seul. Il s'appuie sur SPF et DKIM, et ajoute deux choses essentielles que ces deux mécanismes ne fournissent pas seuls : l'alignement et la politique.

L'alignement, c'est la vérification que le domaine du From visible par l'utilisateur correspond bien au domaine qui a passé SPF ou DKIM. Sans DMARC, on peut avoir un email qui passe SPF (parce qu'il vient d'une IP autorisée pour evil.com) mais qui affiche From: victim@example.com. DMARC bouche cette faille.

La politique, c'est ce que le serveur de réception doit faire si l'alignement échoue : ne rien faire (none), mettre en quarantaine (quarantine), ou rejeter (reject).

Un enregistrement DMARC minimal pour commencer :

_dmarc.example.com. IN TXT "v=DMARC1; p=none; rua=mailto:dmarc@example.com; ruf=mailto:dmarc@example.com"        

Le p=none est volontairement permissif au début. Il permet d'observer, de collecter des rapports agrégés (rua) et des rapports forensiques (ruf), sans casser la délivrabilité pendant la phase de rodage. Ces rapports sont envoyés en XML - souvent illisibles bruts, mais des outils comme dmarcian, MXToolbox ou mail-tester.com les parsent proprement.

Ce que les rapports DMARC révèlent est souvent surprenant : des services tiers qui envoient en votre nom sans que vous le sachiez (newsletters, outils CRM, notifications applicatives), des tentatives d'usurpation depuis des IP inconnues, des incohérences dans votre propre configuration. Passer quelques semaines en p=none n'est pas de la lâcheté c'est de la rigueur.

Une fois sûr de son setup, on peut évoluer vers quarantine puis reject. Mais pas avant. Passer directement à reject sans avoir lu ses rapports DMARC, c'est s'exposer à perdre des emails légitimes sans même s'en rendre compte.

---

Pourquoi Mailcow sur une VM Proxmox, malgré tout

Arrive alors la question du serveur mail "interne", celui qui tourne à la maison.

J'aurais pu assembler moi-même Postfix, Dovecot, Amavis, Rspamd, OpenDKIM, une base SQL pour la gestion des comptes et des alias, un webmail (Roundcube, Rainloop), de la supervision, des certificats Let's Encrypt avec renouvellement automatique, des scripts de maintenance, de la rétention des logs, une gestion fine des quotas, mais ça m'aurait pris des semaines, ça aurait fonctionné, et ça aurait cassé dès qu'une dépendance aurait changé de comportement entre deux mises à jour.

Le choix de Proxmox comme hyperviseur s'est imposé naturellement, c'est ce qui tourne déjà chez moi pour d'autres services. [Mailcow](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fmailcow%2Eemail&urlhash=onVl&trk=article-ssr-frontend-pulse_little-text-block) a donc sa VM dédiée : 4 vCPU, 8 Go de RAM, un disque virtuel sur un SSD NVMe. L'isolation est propre, les snapshots sont possibles avant chaque mise à jour majeure, et si quelque chose explose côté Mailcow, le reste de l'infra domestique n'est pas impacté. C'est l'un des avantages souvent sous-estimés de virtualiser ce genre de service : la capacité à revenir en arrière en trente secondes via un snapshot Proxmox vaut tous les scripts de sauvegarde du monde.

Mailcow est donc un compromis assumé. Ce n'est pas magique, ce n'est pas simple à comprendre en profondeur. Ce n'est pas adapté à des ressources très limitées - 6 Go de RAM est un minimum raisonnable pour tourner confortablement avec ClamAV actif. Mais c'est cohérent, documenté, maintenu activement.

Par contre je n'ai jamais vu une interface aussi riche et confuse à la fois, il y a des menus et sous-menus partout, tout le temps.

Mailcow regroupe dans un seul docker-compose :

- Postfix pour le SMTP (réception et envoi),
- Dovecot pour IMAP, les boîtes, l'authentification,
- Rspamd pour l'antispam, avec une interface d'administration dédiée,
- SOGo pour le webmail, le calendrier et les contacts (CardDAV/CalDAV),
- ClamAV pour l'antivirus,
- une PKI intégrée avec Let's Encrypt,
- une UI d'administration correcte pour gérer domaines, comptes, alias, DKIM.

Il existe des alternatives sérieuses : [Mail-in-a-Box](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fmailinabox%2Eemail&urlhash=PUu8&trk=article-ssr-frontend-pulse_little-text-block) est plus simple mais moins flexible, [iRedMail](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fwww%2Eiredmail%2Eorg&urlhash=CHNW&trk=article-ssr-frontend-pulse_little-text-block) est robuste mais l'édition communautaire est moins maintenue, [Stalwart](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fstalw%2Eart&urlhash=wo7N&trk=article-ssr-frontend-pulse_little-text-block) est une option intéressante tout-en-un en Rust, mais encore jeune. Mailcow a été choisi ici pour une raison simple : il permet d'aller loin sans perdre totalement le contrôle, et sans passer sa vie à coller des briques ensemble.

---

Le piège classique : Mailcow ne doit PAS envoyer directement sur Internet

C'est ici que beaucoup d'installations échouent : les emails partent, Mailcow ne remonte pas d'erreur, mais les messages n'arrivent jamais. Ou arrivent en spam. Ou sont rejetés en différé, des heures plus tard, dans un bounce que personne ne lit.

Mailcow, par défaut, essaie d'envoyer directement les emails vers les MX distants. Il fait une résolution DNS du MX de destination, établit une connexion SMTP sur le port 25, et tente la livraison. Or, depuis une connexion résidentielle, cela échoue presque toujours : port 25 filtré par le FAI, IP résidentielle rejetée ou blacklistée en entrée chez Gmail ou Outlook, IPv6 mal configuré générant des tentatives de connexion sur des adresses sans reverse DNS valide.

La solution consiste à forcer Mailcow à utiliser le VPS comme smarthost - un relais intermédiaire vers lequel tout est délégué.

Un sous-menu donne accès à un autre sous-menu pour le relay.

Dans l'interface Mailcow (ou directement dans la configuration Postfix qu'il expose), on configure un relayhost pointant vers le VPS, sur le port 587 (submission avec STARTTLS), avec authentification SASL. Cela implique de créer sur le VPS un compte SMTP dédié, avec un mot de passe fort, et de restreindre ce compte pour qu'il ne puisse être utilisé que depuis l'IP de la maison.

Concrètement, cela revient à dire à Mailcow : "Quoi qu'il arrive, tu n'essaies pas d'être malin, tu envoies tout au VPS, sur le port 587, avec tes credentials. Lui se débrouille avec le reste du monde."

C'est ce point précis et souvent ce point seulement qui a débloqué toute la chaîne.

## Recommandé par LinkedIn

---

Le VPS : Postfix minimaliste

Sur le VPS, pas de Mailcow. Pas de webmail. Pas d'IMAP. Pas de base de données. Pas d'interface d'administration graphique.

Juste Postfix. Un Postfix installé depuis les paquets de la distribution, configuré à la main, et maintenu aussi simple que possible.

Ce Postfix est configuré pour faire exactement quatre choses, et rien d'autre :

1. Accepter les mails entrants pour le domaine depuis Internet,
2. Les relayer vers la VM Mailcow à la maison,
3. Accepter les mails sortants authentifiés (via SASL) depuis Mailcow,
4. Les livrer vers Internet en parlant proprement SMTP.

Un extrait de main.cf :

myhostname = mx1.example.com
mydomain = example.com
myorigin = example.com

inet_interfaces = all
inet_protocols = all

mydestination =
relay_domains = example.com
transport_maps = hash:/etc/postfix/transport

smtpd_recipient_restrictions =
    permit_mynetworks,
    permit_sasl_authenticated,
    reject_unauth_destination        

Et dans /etc/postfix/transport :

example.com smtp:[adresse-ip-mailcow]        

Le crochet autour de l'adresse est important : il indique à Postfix de ne pas faire de résolution MX sur cette destination, mais de se connecter directement. Sans les crochets, Postfix chercherait un enregistrement MX pour le hostname, ce qui ne fonctionnerait pas pour un serveur interne.

Le VPS ne stocke rien. Il transporte. C'est volontaire : moins il y a de données sur le VPS, moins une compromission de ce serveur serait catastrophique.

---

Tester la configuration : ne rien supposer, tout vérifier

Une fois l'architecture en place, la tentation est de "tester en vrai" - envoyer un email depuis SOGo et voir ce qui se passe. C'est une erreur. Quand quelque chose ne fonctionne pas, on ne sait pas où ça a échoué, il faut donc tester chaque maillon de la chaîne indépendamment, du plus bas niveau vers le plus haut.

Vérification DNS en premier

Avant même de lancer le moindre serveur SMTP, valider le DNS, pas dans la tête mais avec des outils.

# Vérifier le MX
dig MX example.com

# Vérifier que le MX résout vers la bonne IP
dig A mx1.example.com

# Vérifier le reverse DNS de l'IP du VPS
dig -x 203.0.113.42

# Vérifier SPF
dig TXT example.com | grep spf

# Vérifier DKIM
dig TXT dkim._domainkey.example.com

# Vérifier DMARC
dig TXT _dmarc.example.com        

Chaque réponse doit correspondre exactement à ce qu'on a configuré. Une divergence ici, c'est une heure de debug inutile plus tard. MXToolbox propose également une interface web qui agrège tous ces contrôles, super utile pour avoir une vue d'ensemble en un coup d'œil.

Vérifier que les ports écoutent correctement sur le VPS

sudo ss -lntp | egrep ':(25|587)\b'        

Le port 25 doit être ouvert pour recevoir depuis Internet. Le port 587 doit l'être pour que Mailcow puisse s'y authentifier. Si l'un des deux n'apparaît pas, Postfix n'est pas correctement configuré ou le firewall bloque.

Activer proprement le port 587 (submission)

Sur le VPS, le port 587 est souvent présent dans master.cf mais commenté par défaut. Il faut l'activer explicitement avec les bonnes options :

sudo nano /etc/postfix/master.cf        

La section submission doit ressembler à ceci :

submission inet n       -       y       -       -       smtpd
  -o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject        

smtpd_tls_security_level=encrypt force STARTTLS - pas optionnel, obligatoire. smtpd_relay_restrictions=permit_sasl_authenticated,reject garantit que seuls les clients authentifiés peuvent relayer. Sans ça, on court le risque d'avoir un open relay, ce qui est catastrophique pour la réputation IP.

Après modification :

sudo systemctl restart postfix
sudo ss -lntp | egrep ':(25|587)\b'        

Tester l'envoi avec swaks

[swaks (Swiss Army Knife for SMTP)](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fjetmore%2Eorg%2Fjohn%2Fcode%2Fswaks%2F&urlhash=43uR&trk=article-ssr-frontend-pulse_little-text-block) est l'outil indispensable pour tester une chaîne SMTP sans passer par un client mail, il simule exactement ce que ferait Mailcow lors d'un envoi.

swaks \
  --server localhost \
  --port 587 \
  --auth LOGIN \
  --auth-user relaymail \
  --auth-password 'password' \
  --to test@gmail.com \
  --from test@example.com        

Ce test s'effectue directement depuis le VPS pour valider que le port 587 fonctionne, que l'authentification SASL est opérationnelle, et que Postfix accepte bien de relayer vers l'extérieur. Si la commande retourne un 250 OK, le message est accepté par Postfix local. Ce n'est pas encore une garantie de délivrabilité mais c'est la première étape validée.

Pour tester la chaîne complète depuis Mailcow jusqu'au VPS, on lance le même swaks depuis la VM Proxmox, en pointant cette fois vers l'IP du VPS :

swaks \
  --server mx1.example.com \
  --port 587 \
  --auth LOGIN \
  --auth-user relaymail \
  --auth-password 'password' \
  --to test@gmail.com \
  --from test@example.com \
  --tls        

Si ça passe : la connexion authentifiée entre Mailcow et le VPS fonctionne. Si ça échoue : soit le firewall bloque, soit les credentials sont mauvais, soit STARTTLS n'est pas correctement configuré.

Suivre les logs en temps réel

L'outil de diagnostic le plus sous-utilisé reste tail -f sur les logs Postfix. Sur le VPS :

sudo tail -f /var/log/mail.log        

Chaque transaction SMTP génère des lignes de log détaillées : connexion entrante, authentification, résolution MX de destination, tentative de livraison, réponse du serveur distant. Quand Gmail rejette un message, la raison apparaît ici - souvent avec un code d'erreur et un lien vers une page d'aide Google. Ces messages d'erreur sont bien plus informatifs qu'on ne le pense.

Tester la réception

Pour la réception, le test le plus simple est d'envoyer un email depuis un compte Gmail externe vers votre domaine, puis de vérifier dans les logs du VPS que le message arrive bien, qu'il est correctement relayé vers Mailcow, et qu'il apparaît dans SOGo ou dans un client IMAP.

sudo tail -f /var/log/mail.log | grep "from=<votre-gmail>"        

Si le message arrive sur le VPS mais pas dans Mailcow, le problème est dans le relayage interne (le transport map). Si le message n'arrive pas du tout sur le VPS, le problème est DNS ou réseau.

Le test de délivrabilité global : mail-tester.com

Une fois la chaîne validée brique par brique, [mail-tester.com](https://www.linkedin.com/redir/redirect?url=https%3A%2F%2Fwww%2Email-tester%2Ecom&urlhash=1xNb&trk=article-ssr-frontend-pulse_little-text-block) donne une vue synthétique de la délivrabilité réelle. Le service génère une adresse temporaire, on lui envoie un email depuis son compte, et il retourne un score sur 10 avec le détail de chaque vérification : SPF, DKIM, DMARC, réputation IP, contenu du message, structure des headers.

Un score de 10/10 est l'objectif. En dessous de 8, il reste des problèmes sérieux. L'interface détaille précisément ce qui échoue et c'est souvent là que des erreurs de configuration subtiles apparaissent pour la première fois.

---

Sécuriser le VPS : parce qu'un Postfix exposé sur Internet est une cible

Un Postfix ouvert sur le port 25, sur un VPS avec une IP propre, c'est immédiatement visible par les scanners automatiques. En quelques heures, les tentatives de connexion commencent. La sécurisation n'est pas optionnelle - c'est une partie intégrante de l'architecture.

Firewall : n'exposer que ce qui est strictement nécessaire

Le principe est simple : seuls les ports utiles sont ouverts, et certains d'entre eux sont restreints à des IPs spécifiques.

Avec ufw (ou nftables selon les préférences) :

# SSH : restreint à votre IP fixe ou un bastion
sudo ufw allow from VOTRE_IP_FIXE to any port 22

# SMTP entrant : ouvert à Internet (réception des emails)
sudo ufw allow 25/tcp

# Submission : restreint à l'IP de Mailcow uniquement
sudo ufw allow from IP_MAILCOW_MAISON to any port 587

# Tout le reste : refusé
sudo ufw default deny incoming
sudo ufw enable        

L'idée clé ici : le port 587 ne doit jamais être ouvert à tout Internet. Seule l'IP de la VM Mailcow à la maison a besoin d'y accéder. Si cette IP est dynamique chez vous (ce qui est le cas avec la plupart des FAI français) il faudra soit mettre en place un DNS dynamique et adapter les règles, soit accepter de mettre à jour la règle manuellement lors des changements d'IP.

Fail2ban : bannir automatiquement les tentatives répétées

Les bots scannent en permanence le port 25 à la recherche d'open relays ou de comptes à brute-forcer. Fail2ban surveille les logs Postfix et banne automatiquement les IPs qui génèrent trop d'erreurs d'authentification ou de tentatives de relay non autorisées.

sudo apt install fail2ban        

Dans /etc/fail2ban/jail.local :

[postfix]
enabled  = true
port     = smtp,submission
filter   = postfix
logpath  = /var/log/mail.log
maxretry = 5
bantime  = 3600
findtime = 600

[postfix-sasl]
enabled  = true
port     = submission
filter   = postfix-sasl
logpath  = /var/log/mail.log
maxretry = 3
bantime  = 86400
findtime = 300        

Le filtre postfix-sasl est particulièrement important : il ban rapidement les tentatives de brute-force sur les credentials SASL du port 587. Trois tentatives échouées en cinq minutes, et l'IP est bannie 24 heures.

Ne jamais être un open relay

C'est la règle cardinale. Un open relay, soit un serveur qui accepte de relayer des emails pour n'importe qui, sans authentification, est blacklisté en quelques heures. Les conséquences sont immédiates et durables : l'IP est listée sur les principales blacklists, et la réputation construite patiemment disparaît.

Pour vérifier qu'on n'est pas un open relay, on peut utiliser MXToolbox (outil "Open Relay Test") ou tenter manuellement :

telnet mx1.example.com 25
EHLO test.com
MAIL FROM:<attacker@evil.com>
RCPT TO:<victim@gmail.com>
# Doit recevoir : 554 Relay access denied        

Garder le système à jour

Postfix est un logiciel mature et rarement vulnérable, mais le système sous-jacent l'est potentiellement. Les mises à jour de sécurité automatiques (unattended-upgrades sous Debian/Ubuntu) sont une bonne pratique sur un serveur exposé. Ce n'est pas glamour, mais c'est le genre de chose qui évite des nuits de gestion de crise.

Monitoring minimal : savoir quand quelque chose casse

Un serveur mail silencieux peut casser silencieusement. Un monitoring même basique - vérification périodique que le port 25 répond, alerte si la file d'attente Postfix grossit anormalement - évite de découvrir trois jours après qu'on n'a plus reçu de mail.

# Vérifier la file d'attente Postfix
sudo mailq | tail -1

# Vérifier les messages en attente
sudo postqueue -p | grep -E "^[A-F0-9]"        

Si la file grossit et ne se vide pas, quelque chose est cassé dans le relayage vers Mailcow : panne de la VM, coupure réseau à la maison, erreur d'authentification. Les logs Postfix diront précisément pourquoi les livraisons échouent.

---

La délivrabilité : le moment de vérité

Une fois tout en place, les tests sont impitoyables et Gmail ne pardonne rien.

Les premiers emails arrivent souvent en spam, c'est "normal" et c'est décourageant. La réputation d'une IP se construit lentement, empiriquement, sans feedback direct. Les Postmaster Tools de Google et de Microsoft permettent d'observer comment vos emails sont perçus mais uniquement si vous avez un volume suffisant, ce qui pour un usage personnel n'est jamais garanti.

Il faut envoyer peu, régulièrement, proprement. Ne pas tester dix fois par minute depuis la même IP. Ne pas modifier le DNS toutes les heures. Ne pas envoyer depuis une boîte fraîche vers des listes qui vont bouncer massivement.

Et un jour, sans prévenir, les emails arrivent en boîte principale.

C'est à ce moment précis qu'on comprend pourquoi tant d'entreprises externalisent ce problème à des ESPs. Ce n'est pas parce qu'elles ne pourraient pas le faire elles-mêmes. C'est parce que le coût d'opportunité est réel, le risque de délivrabilité est permanent, et la maintenance est continue.

---

Conclusion : une expérience enrichissante… et profondément dissuasive

Héberger ses emails chez soi en 2026 est possible. Techniquement, fonctionnellement, proprement, même.

Mais ce n'est ni simple ni confortable. Chaque brique existe pour une raison, chaque RFC est écrite avec le sang de ceux qui ont essayé avant, et chaque approximation est immédiatement sanctionnée par les géants du mail (pas avec un message d'erreur clair, mais avec du silence, des dossiers spam, des bounces cryptiques)...

Ce qui m'a le plus frappé dans cette expérience, c'est à quel point le mail est devenu un oligopole de fait. Les règles sont théoriquement ouvertes, documentées, accessibles, mais la réalité de la délivrabilité est dominée par quelques acteurs qui font et défont les standards d'usage en dehors de tout processus de normalisation formel. Envoyer un email à quelqu'un qui utilise Gmail, c'est convaincre Google que vous êtes légitime, pas l'inverse.

Cette expérience m'a aussi rappelé une chose essentielle : le mail est l'un des derniers protocoles réellement décentralisés dans sa conception, n'importe qui peut, techniquement, tenir son bout de la chaîne. Mais cette liberté a un prix, qui s'est considérablement apprécié au fil des années. La décentralisation existe sur le papier, mais dans les faits, sans une IP propre, un nom de domaine ancien, une réputation construite, et une connaissance fine des mécanismes, vous êtes en marge.

Et honnêtement ? Si je devais donner un conseil à quelqu'un qui hésite encore : faites-le pour apprendre, pas pour le confort, parce que le confort, dans cette histoire, il est ailleurs, chez les prestataires que vous aurez mieux compris après avoir fait ça une fois.