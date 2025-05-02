---
link: https://zonetuto.fr/astuce/ovh-activer-la-boite-mail-gratuite-avec-un-nom-de-domaine/
byline: MrTuto
site: ZoneTuto
date: 2024-02-20T14:32
excerpt: OVH possède des pages de documentation bien référencées sur les moteurs de recherche sur ce thème, mais je trouve que ce n’est forcément pas très clair et bien expliqué. Difficile d’obtenir facilement l’information qui nous intéresse ici et si on ne connaît pas bien leur interface, c’est encore pire ensuite. Je vous propose donc ici ... Lire la suite
slurped: 2025-02-22T09:40
title: "OVH : activer la boite mail gratuite avec un nom de domaine"
tags:
  - ovh
  - mail
---

OVH possède des pages de documentation bien référencées sur les moteurs de recherche sur ce thème, mais je trouve que ce n’est forcément pas très clair et bien expliqué. Difficile d’obtenir facilement l’information qui nous intéresse ici et si on ne connaît pas bien leur interface, c’est encore pire ensuite. Je vous propose donc ici un tutoriel pour vous y retrouver facilement et profiter de votre boite mail gratuit de 5 Go quand vous prenez un nom de domaine OVH. Je prends souvent des noms de domaines chez eux, mais pas l’hébergement, car j’administre [des serveurs VPS](https://zonetuto.fr/linux/comment-creer-gratuitement-un-serveur-vps-sur-digitalocean/) qui sont ailleurs.

Sauf que pour certains services, je n’ai pas envie de m’occuper moi-même de la partie mail et posséder une adresse mail gratuite ça me va très bien et c’est suffisant. Si vous avez besoin de quelque chose plus complet concernant les mails comme beaucoup d’adresses email sur votre nom de domaine, il faudra bien évidemment passer sur une offre supérieure qui n’est pas gratuite.

J’espère vraiment que cette politique d’une adresse mail offerte avec un hébergement de 5 Go ne changera pas dans le futur … Quand on voit ce qu’il s’est passé avec Gandi, c’est un vrai carnage pour certains comptes client qui avaient beaucoup d’adresses emails qui jusqu’ici ne coûtaient pas cher. Je fais confiance à OVH sur ce point et sinon vous avez aussi excellent Infomaniak qui est très réputé depuis plusieurs années. Pour information, [Infomaniak propose aussi une adresse mail gratuite liée à votre nom de domaine](https://zonetuto.fr/infomaniak-home) si vous le prenez chez eux. Dans ce tutoriel, on va se concentrer sur OVH et je vais vous donner la petite technique à connaître pour débloquer facilement votre adresse mail gratuite liée à votre nom de domaine.

## Comment profiter d’un compte mail gratuit avec votre nom de domaine chez OVH ?

Vous avez acheté votre beau nom de domaine, vous êtes content, il est bien, il est classe, il est promis à un bel avenir. Je suis sûr que vous allez vivre une belle aventure ensemble. Pour ma part, je suis très attaché à certains et ce serait un déchirement de les perdre. Vous avez configuré vos DNS vers l’IP de votre serveur ou vous avez par exemple [installé WordPress sur votre serveur VPS](https://zonetuto.fr/linux/installer-wordpress-sur-un-serveur-ubuntu-avec-nginx-php-8-2-et-mariadb/), mais il vous manque quelque chose.

Vous n’avez toujours pas d’adresse mail « propre » rattachée à ce nom de domaine. C’est le moment d’utiliser cette petite fonction cachée d’OVH qui je pense est volontairement cachée pour que ce ne soit pas disponible par défaut pour tout le monde et donc faire des économies d’échelle. Bon, on peut les comprendre, mais il faut la petite technique que je vais vous montrer.

Une fois connecté à votre compte rendez-vous sur « Web Cloud » et sélectionnez votre nom de domaine. Vous arrivez alors sur le tableau de bord de votre nom de domaine qui devrait ressembler à ça au moment où j’écris ces lignes :

![](https://zonetuto.fr/wp-content/uploads/2024/02/ovh-tableau-de-bord-web-cloud-nom-de-domaine-1024x456.jpg)

Allez sur la section « Hébergement Web et e-mail gratuit » et cliquez sur les trois petits points juste à côté :

![](https://zonetuto.fr/wp-content/uploads/2024/02/ovh-nom-de-domaine-activer-hebergement-gratuit-mail.png)

Enfin, vous pouvez cliquer sur « Activer ». Vous arrivez alors sur un nouveau menu avec une étape qui est importante :

![](https://zonetuto.fr/wp-content/uploads/2024/02/ovh-configuration-hebergement-gratuit-1024x327.png)

Je vous conseille de laisser OVH configurer les enregistrements MX de votre zone DNS pour qu’il ajoute les bons si ce n’est pas le cas. Par contre pour les records DNS de type A ça dépend. Comme l’indique OVH, si vous avez un site web lié à ce nom de domaine qui est hébergé ailleurs, ne cochez pas cette case pour ne pas écraser la configuration actuelle. La suite est classique et vous devriez vous en sortir sans encombre. Une fois que c’est validé, il faut attendre un petit peu comme l’indique de message de OVH : L’activation de l’hébergement est en cours, cela peut prendre quelques minutes.

Après avoir patienté un peu, vous pouvez maintenant aller dans la catégorie « E-mails » de « Web Cloud » et cliquer sur le nom de domaine ou vous avez activé la boite mail et l’hébergement gratuit. Vous avez maintenant la possibilité de créer une adresse mail de votre choix avec votre nom de domaine :

![](https://zonetuto.fr/wp-content/uploads/2024/02/ovh-boite-mail-gratuite-1024x468.png)

Une fois que votre adresse mail gratuite est créée, vous pouvez directement utiliser l’interface web d’OVH pour commencer à envoyer et recevoir des mails. Cependant, je vous conseille plutôt d’[ajouter votre adresse mail dans le client Thunderbird](https://zonetuto.fr/outils/comment-ajouter-ses-adresses-mail-dans-thunderbird/) qui récupérera automatiquement la configuration SMTP d’OVH. C’est très facile à configurer avec ce logiciel de Mozilla. Si vous voulez utiliser un autre client mail en local, je vous conseille d’aller voir ce [guide des configurations SMTP](https://zonetuto.fr/outils/liste-smtp-imap-pop-serveur-adresse-boite-mail/) pour ajouter votre boite mail dans un autre client mail.