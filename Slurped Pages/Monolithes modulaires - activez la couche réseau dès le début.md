---
link: https://medium.com/@tpierrain/monolithes-modulaires-activez-la-couche-r%C3%A9seau-d%C3%A8s-le-d%C3%A9but-45285eb84000
byline: Thomas Pierrain. (υѕe caѕe drιven)
site: Medium
date: 2024-07-12T19:48
excerpt: TLDR; Pour garantir une séparation de modules efficace dans un
  monolithe modulaire, activez le transport réseau au runtime et en production
  dès le début, et ce même si tous les modules sont déployés…
twitter: https://twitter.com/@tpierrain
slurped: 2024-08-26T18:52
title: "Monolithes modulaires : activez la couche réseau dès le début"
---
**_TLDR;_** _Pour garantir une séparation de modules efficace dans un monolithe modulaire, activez le transport réseau au runtime et en production dès le début, et ce même si tous les modules sont déployés dans le même processus. Cela permet de détecter et d’améliorer les interactions inefficaces dès le début, évitant des problèmes de performance et de communication lors d’un futur déploiement séparé des modules._

## Le problème

Travailler dans un monolithe modulaire est passionnant et très utile, mais cela exige une grande rigueur. En effet, garantir la possibilité de séparer les modules à l’exécution est loin d’être simple. De nombreux pièges peuvent survenir quand on travaille dans un tel environnement (comme l’utilisation d’une même base de données par plusieurs modules par exemple).

Aujourd’hui, je souhaite me concentrer sur un autre piège courant : celui d’avoir des interactions in-proc excessivement bavardes (chatty) entre les modules, ou pire, l’adoption d’un style RPC (Remote Procedure Call) entre modules.

Ces interactions inefficaces peuvent compromettre les performances et remettre même en question la séparation des modules que l’on souhaiterait réaliser ultérieurement.

## Burn the “RPC witch”!

Depuis des décennies, nous savons que la promesse du RPC ([**Remote Procedure Call**](https://en.wikipedia.org/wiki/Remote_procedure_call)) — donner l’illusion qu’un code appelle une méthode sur un objet local alors qu’il ne fait que proxy pour un objet situé dans un autre processus possiblement lointain — nous mène à une impasse.

Cette “leaky abstraction” nuit très souvent gravement aux performances de nos applications. Sans nous en rendre compte, on risque facilement de multiplier les appels réseau au sein de boucles for ou d’adopter des communications excessivement bavardes entre modules, ce qui dégrade encore plus les performances.

Bref. A la place du paradigme RPC, explicitons plutôt tous les implicites, et rendons visible dans notre code le fait qu’il peut y avoir des I/O et du réseaux.

## Une prise en compte souvent tardive et frustrante

Dans le cas d’un monolithe modulaire que j’évoquais juste au dessus, on peut se rendre compte un peu tard que nos interactions entre modules sont tellement peu adaptées à un mode de communication réseau (http ou même gRPC) qu’il va nous falloir les repenser.

- Regrouper/Chunker nos aller-retour à l’aide de DTO (qui pour rappel servent à réduire le nombre d’A-R réseaux entre 2 systèmes)
- Changer les contrats d’échanges entre modules etc

Je dis « _un peu tard_ », car c’est pile au moment où on va se dire : « _ça aurait du sens qu’on déploie finalement ces 2 modules indépendamment_ » qu’on ne pourra pas le faire facilement, ni sans travail ou efforts supplémentaires.

## **Un pattern que je pensais bien connaitre**

De mon côté, cela faisait des années que je travaille dans des monolithes modulaires en composant des « hexagones » dans le même processus pour cadrer les frontières et préparer ces éventuelles séparations.

C’est un pattern que j’ai appelé the Hive (la ruche) et que nous sommes quelques-uns à avoir expérimenté depuis des années (dont mon ami [**Julien Topçu**](https://twitter.com/JulienTopcu), avec qui on va présenter celui-ci dans un talk très prochainement).

_the hive pattern, bientôt sur vos écrans ;-)_

Jusqu’alors, je pensais que la mise en œuvre de ce pattern était suffisante pour cadrer ces frontières entre modules. On fait émerger des ports (contrats) entre les modules qui marquent les frontières, et on peut au runtime utiliser :

- soit des Adapter In-Proc (c.a.d. Des adaptateurs qui ne font que des appels de méthodes dans le même processus, sur les ports d’autres modules)
- soit des adaptateurs plus classiques, qui font des appels HTTP pour parler aux autres modules, lorsqu’on souhaite déployer ces modules dans des processus différents.

## Sympa la force…

Et bien je viens de me rendre compte tout récemment que c’était autre chose qui m’avait protégé jusqu’à maintenant contre ces problèmes d’interactions inadaptées au second mode de déploiement (i.e.: quand on met du réseau et des I/O entre les modules).

J’avais jusqu’à maintenant tout simplement été…

## Aidé par mes précédents environnements

Dans mon cas, c’est plutôt que mon utilisation des monolithes modulaires a longtemps été cadré par des solutions de messaging low latency qu’on retrouve dans la finance (ex: 29West, renommé Ultra Messaging). Ceux-ci permettaient à la configuration de switcher facilement d’un protocole à l’autre mais sans changer notre code, juste par de la configuration. On pouvait par exemple choisir une couche de transport parmis :

- IPC (via Shared-memory et ring buffer)
- UDP unicast
- UDP multicast (super pour de gros throughput combiné à de la faible latence)
- TCP (pour les infrastructures peu compatibles avec UDP)

La conséquence c’est qu’on savait -lorsqu’on codait- que chaque échange inter-module allait probablement passer par du réseau et tout ce que ça implique. On avait donc fortement conscience de ça et faisions explicitement du message-passing entre nos modules (et surtout pas du RPC).

En d’autres termes, on rendait très explicite dans notre code le fait que l’on parlait à un module possiblement déployé ailleurs (et donc coûteux en termes de latence, soumis à des aléas liés au réseau, etc).

## Mais ça, c’était avant…

Je constate aujourd’hui que ce n’est pas forcément le cas pour des gens qui n’ont connu le monolithe modulaire qu’à travers des architectures à base d’API web ou qui utilisent même du gRPC (qui ne force malheureusement pas les gens à faire du message passing explicite).

Du coup, ma proposition pour vous éviter de devoir justifier des délais supplémentaires et passer des semaines à revoir et retailler certains de vos protocoles d’échanges entre modules, c’est :

> **Activez une couche de transport réseau au runtime (ex: HTTP), Y COMPRIS quand ce n’est pas nécessaire en production et que vous déployez tous vos modules dans le même processus/pod.**

Et par exemple pour nous qui déployons tous nos services sur du K8s, cela signifie également qu’on doit s’assurer :

- De repasser par la partie Load Balancing (LB) sur tous nos appels HTTP internes entre modules dans le même monolithe (ça répartit bien la charge, évite pleins d’erreurs liés au graceful shutdown, mais aussi nous fait bénéficier des stratégies de throttling au niveau de l’infra => nous évite de nous auto-DDoS avec des appels en boucle avec nous-même)
- De désactiver le tcp-keep alive automatique sous-jacent à nos implémentations de client HTTP pour tous les cas où on s’appelle entre services ou modules dans le même cluster K8s. Ça, c’est plutôt lié au mode de fonctionnement de Kube et nous permet justement de repasser systématiquement par le LB, mais surtout d’éviter de maintenir illusoirement des connexions vers des pods qui se sont déjà fait préempter par exemple (élasticité et finops oblige). On s’évite un bon paquet d’erreur 503 et de retry. Attention par contre, à ne pas désactiver le tcp-keep alive si vous consommez un autre module déployé dans une autre région. Parce que dans ce cas précis, vous allez sentir passer le coût du handshake TCP systématique 😄

## Ok mais les performances dans tout ça ?!?

Comme d’hab, il vous suffit de mesurer plutôt que d’imaginer quelque chose (et cette heuristique là -elle — est toujours valable 🙂). En d’autres termes :

Et même si vous déployez ensuite tous vos modules séparément, mais dans le même cluster K8s comme c’est le cas chez nous, l’impact de faire des I/Os sera vraiment fortement limité, voire insensible pour des utilisateurs. En tout cas, le fait de vous être entrainés à passer par le réseau même quand ces modules étaient regroupés dans le même processus, vous aura servi de répétition pour ce déploiement séparé à venir.

## Un feedback nécessaire

Et si ce n’est pas le cas et que vous constatez des problèmes de latence sur certains usages, c’est que vous devriez très vraisemblablement revoir vos échanges et contrats d’interfaces (“port” au sens archi hexa) entre modules.

Mais le fait d’introduire du réseau dès le début entre vos modules comme si vous vouliez dès maintenant les déployer séparément, aura une vertu cardinale : faire du back-pressure très tôt contre vos mauvais design, vos mauvaises idées et vos mauvais contrats d’interactions entre modules au sein de votre monolithe distribué.

Je ne sais pas pour vous, mais en ce qui me concerne : **avoir du feedback le plus vite possible sur des mauvais choix de design est quelque chose qui me réjouit 🙂.**