---
link: https://www.glukhov.org/fr/app-architecture/integration-patterns/idempotency-in-distributed-systems/
byline: À propos de Rost Glukhov
site: Rost Glukhov | Site personnel et blog technique
excerpt: L’idempotence n’est pas une astuce HTTP. Découvrez comment éviter les écritures dupliquées, les messages rejoués et les double facturations entre les API, les files d’attente, les webhooks et les workflows.
slurped: 2026-05-10T16:55
title: L'idempotence dans les systèmes distribués qui fonctionne réellement
tags:
  - idempotency
---

L’idempotence dans les systèmes distribués est la propriété qui vous sauve lorsque le réseau ment, la file d’attente effectue des tentatives de relecture, le client panique et l’opérateur lance une rejoue. Dans les systèmes de production, la livraison en double est normale. Les effets de bord en double constituent le bug.

Le protocole HTTP définit une méthode idempotente comme une méthode où plusieurs requêtes identiques ont le même effet prévu sur le serveur qu’une seule requête. C’est pourquoi les méthodes PUT, DELETE et les méthodes sûres sont idempotentes dans la sémantique du protocole et peuvent être relancées automatiquement après une défaillance de communication.

![flux de message d’intégration : idempotence](https://www.glukhov.org/img/integration-message-flow_w678.jpg "flux de message d’intégration : idempotence")

Cette définition est utile, mais elle ne suffit pas. Dans les architectures réelles, l’idempotence n’est pas une simple question de culture générale HTTP. C’est une garantie métier. Si un client clique sur « payer » une fois, vous ne devez pas le facturer deux fois simplement parce qu’une expiration de délai (timeout) s’est produite entre l’engagement (commit) et la réponse. Si un travailleur met à jour l’inventaire et s’arrête avant d’acquitter le message, vous ne devez pas décrémenter le stock deux fois parce que le courtier a redistribué le message. Tel est le niveau d’exigence.

L’erreur que je vois se répéter sans cesse consiste à traiter l’idempotence comme une caractéristique de transport plutôt que comme une propriété du système. La déduplication des files d’attente, les verbes HTTP et les tentatives de relecture du client aident, mais aucun d’eux ne sauve une conception qui laisse un même intention métier créer un second effet de bord. Si vous souhaitez une vue d’ensemble sur la manière dont ces décisions d’intégration s’inscrivent dans les limites des services et les compromis d’accès aux données, commencez par [Architecture d’application en production : modèles d’intégration, conception de code et accès aux données](https://www.glukhov.org/fr/app-architecture/ "Pilier d’architecture d’application pratique pour les systèmes de production : modèles d’intégration basés sur le chat avec Slack et Discord, modèles de conception d’architecture propre en Python, et compromis d’accès aux données en Go avec GORM, Ent, Bun et sqlc.").

## D’où viennent les doublons en production

Les doublons n’apparaissent pas parce que les équipes sont négligentes. Ils apparaissent parce que les systèmes distribués relancent, réordonnent et rejouent.

Un client peut envoyer une requête de création, le serveur peut l’engager, et la réponse peut quand même disparaître sur le réseau. C’est exactement pourquoi HTTP distingue les méthodes idempotentes et pourquoi les API de paiement telles que Stripe et PayPal exposent des mécanismes d’idempotence explicites pour les méthodes non sûres comme POST.

Les courtiers de messages rendent le problème encore plus évident. La livraison « au moins une fois » signifie qu’un consommateur peut être invoqué à plusieurs reprises pour le même message, et qu’un gestionnaire peut mettre à jour la base de données avec succès mais échouer avant l’acquittement, ce qui amène le courtier à redistribuer le même message.

Les webhooks ne sont pas différents. GitHub indique que les livraisons de webhooks peuvent arriver dans le désordre, que les livraisons échouées ne sont pas automatiquement redistribuées, et que chaque livraison porte un GUID `X-GitHub-Delivery` unique que vous devez utiliser pour vous protéger contre les rejoues. Pour une vue architecturale pratique des points de terminaison de chat en tant que limites d’interaction, consultez [Les plateformes de chat comme interfaces système dans les systèmes modernes](https://www.glukhov.org/fr/app-architecture/integration-patterns/chat-platforms-as-system-interfaces/ "Explorez comment Slack et Discord agissent comme interfaces système pour les flux de travail d’alerte et le contrôle humain dans la boucle dans les architectures distribuées modernes.").

Même les systèmes qui publient des garanties plus fortes vous laissent encore du travail à faire. Kafka peut empêcher les entrées dupliquées dans les journaux Kafka avec des producteurs idempotents et peut fournir une livraison « exactement une fois » pour les flux lecture-traitement-écriture qui restent à l’intérieur de Kafka avec des transactions et des consommateurs `read_committed`. Mais la documentation de conception de Kafka est claire : les systèmes externes nécessitent toujours une coordination avec les offsets et les sorties. La livraison « exactement une fois » de Google Cloud Pub/Sub est limitée aux abonnements de tirage (pull), au sein d’une région cloud, et exige toujours que les clients suivent la progression du traitement jusqu’à ce que l’acquittement réussisse.

Mon résumé, teinté d’opinion, est simple. Supposez que le transport relancera. Supposez que les opérateurs rejouera. Supposez que les webhooks arriveront en retard. Concevez le chemin d’écriture de sorte qu’une intention répétée ne puisse pas créer un second effet métier.

## Le contrat API auquel je fais réellement confiance

Le seul contrat API auquel je fais confiance pour les opérations de mutation est l’intention fournie par l’appelant plus la persistance côté serveur.

AWS recommande un identifiant de requête fourni par l’appelant et met en garde que le service doit enregistrer atomiquement le jeton d’idempotence conjointement avec le travail de mutation. Stripe stocke le premier code d’état et le corps de la réponse pour une clé, compare les paramètres ultérieurs avec la requête originale, et retourne le même résultat pour les relances. PayPal utilise `PayPal-Request-Id` sur les API POST prises en charge et retourne le dernier statut pour la requête précédente avec ce même en-tête.

Cela conduit à un contrat pratique :

1. Le client génère une clé d’idempotence pour une opération métier.
2. Le serveur scope cette clé par locataire (tenant) et nom d’opération.
3. Le serveur stocke un hachage de requête afin que la même clé ne puisse pas être réutilisée pour une charge utile différente.
4. Le serveur enregistre l’état tel que `pending` (en attente), `completed` (terminé) ou `failed` (échoué).
5. Les relances avec la même clé retournent soit le résultat stocké, soit un pointeur stable vers celui-ci.
6. Les relances avec la même clé et une charge utile différente échouent de manière explicite.

Il existe un projet de brouillon d’en-tête IETF `Idempotency-Key`, mais au 09-05-2026, il est toujours répertorié dans le Datatracker IETF comme un Internet-Draft expiré plutôt que comme une RFC publiée. En pratique, le nom de l’en-tête reste largement utile comme convention de facto, mais vous devez documenter le contrat dans votre propre API plutôt que de prétendre que la norme est terminée.

Que doit représenter la clé ? L’intention. Pas une tentative HTTP. Pas une connexion TCP. Pas un compteur de relance. Si l’utilisateur signifie « créer la commande 123 une fois », chaque relance pour cette même commande doit réutiliser la même clé. Si l’utilisateur signifie « passer une deuxième commande », cela doit utiliser une clé différente.

Un ID de requête sert au traçage. Une clé d’idempotence sert à la correction. Si vous mélangez les deux, vos tableaux de bord auront l’air propre pendant que votre argent bougera deux fois.

### Pourquoi PUT ne suffit pas

Non, HTTP PUT ne suffit pas pour rendre une opération idempotente.

Oui, la RFC 9110 donne à PUT des sémantiques idempotentes. Mais si votre gestionnaire PUT émet un nouvel événement en aval, envoie un e-mail à chaque tentative de relecture ou facture à nouveau un fournisseur externe, alors votre implémentation a violé le contrat métier, même si le nom de votre route semble respectable.

Le choix du verbe aide les clients à comprendre l’intention. Il n’implémente pas l’intention pour vous.

Utilisez PUT lorsque le modèle de ressource correspond véritablement à une opération de remplacement complet ou de type upsert. Utilisez POST lorsque vous créez des commandes ou des actions. Mais pour toute mutation qui pourrait être relancée au-delà des limites du réseau, documentez un contrat d’idempotence explicite. Si vos actions de mutation sont déclenchées à partir de flux de travail de chat, le même contrat s’applique dans [Modèles d’intégration Slack pour les alertes et les flux de travail](https://www.glukhov.org/fr/app-architecture/integration-patterns/slack/ "Analyse approfondie des webhooks et applications Slack pour les alertes, les approbations et l’automatisation des flux de travail. Boutons Block Kit, vérification de signature, exemples Go et Python.") et [Modèle d’intégration Discord pour les alertes et les boucles de contrôle](https://www.glukhov.org/fr/app-architecture/integration-patterns/discord/ "Analyse approfondie des webhooks et bots Discord pour les alertes, les approbations et le contrôle humain dans la boucle. Exemples Go et Python, sécurité, idempotence et routage."). Les effets de bord cachés sont l’endroit où l’architecture va mourir.

### Combien de temps une clé d’idempotence doit-elle être stockée

Plus longtemps que votre équipe de transport ne le souhaite.

Stripe indique que les clés peuvent être éliminées après au moins 24 heures. PayPal indique que la rétention est spécifique à l’API et donne des exemples qui peuvent durer jusqu’à 45 jours. Amazon SQS FIFO déduplique uniquement dans une fenêtre de 5 minutes. GitHub conserve les livraisons récentes pendant 3 jours pour la redistribution manuelle. Ces chiffres sont très différents parce que la période de rétention correcte est une décision métier, pas une valeur par défaut du protocole.

Si vous ne conservez les clés que pendant cinq minutes parce que votre file d’attente le fait, vous ne concevez pas l’idempotence. Vous copiez une limitation de transport dans votre couche métier.

Conservez les enregistrements d’idempotence pendant au moins le maximum de ces fenêtres :

- horizon de relance du client
- horizon de redrive de la file d’attente
- horizon de rejoue des webhooks
- horizon de rejoue de l’opérateur
- horizon de règlement ou de compensation pour les opérations de transfert d’argent

Pour les paiements, les réservations et l’approvisionnement, cela signifie souvent des heures ou des jours, pas des minutes.

AWS signale également deux anti-modèles avec lesquels je suis entièrement d’accord. N’utilisez pas les horodatages comme clé, car le décalage d’horloge et les collisions les rendent peu fiables. N’enregistrez pas aveuglément l’intégralité des charges utiles de requête comme enregistrement de déduplication pour chaque requête, car cela nuit aux performances et à l’évolutivité. Stockez un hachage de requête normalisé plus l’état de réponse minimum dont vous avez besoin pour rejouer en toute sécurité. Si vous devez reproduire la première réponse byte pour byte, stockez le corps de la réponse canonique comme le fait Stripe.

## Les modèles de base de données qui rendent l’idempotence réelle

L’idempotence devient réelle lorsque la couche de persistance peut gagner une course une seule fois.

PostgreSQL vous donne deux primitives critiques ici. Les contraintes d’unicité imposent l’unicité sur une ou plusieurs colonnes, et `INSERT ... ON CONFLICT` vous permet de définir une action alternative au lieu d’échouer sur une violation d’unicité. PostgreSQL documente également que `ON CONFLICT DO UPDATE` garantit un résultat d’insertion ou de mise à jour atomique sous concurrence.

Cela signifie que votre couche d’idempotence devrait généralement commencer par une table comme celle-ci :

```
create table api_idempotency (
    tenant_id text not null,
    operation text not null,
    idempotency_key text not null,
    request_hash text not null,
    state text not null,
    status_code integer,
    response_body jsonb,
    resource_type text,
    resource_id text,
    created_at timestamptz not null default now(),
    expires_at timestamptz not null,
    primary key (tenant_id, operation, idempotency_key)
);
```

Et le flux de traitement devrait ressembler à ceci :

```
begin transaction

try insert (tenant_id, operation, idempotency_key, request_hash, state='pending')
on conflict do nothing

load row for (tenant_id, operation, idempotency_key) for update

if row.request_hash != incoming_request_hash
    fail with conflict or validation error

if row.state = 'completed'
    return stored response

if row.state = 'pending' and row was created by another live request
    either wait briefly, or fail fast with a retryable response

perform local business mutation

store stable result in idempotency row
set state = 'completed'

commit
return result
```

La partie importante n’est pas la syntaxe. La partie importante est l’atomicité. L’enregistrement de la clé et l’exécution de la mutation doivent réussir ou échouer ensemble. AWS le dit explicitement pour l’idempotence des API, et la même règle s’applique aux services basés sur SQL.

Ne faites pas une séquence naïve de vérification puis d’action comme « sélectionner la clé ; si manquante alors insérer la commande ». Sous concurrence, deux requêtes peuvent passer la vérification et toutes deux créer l’effet de bord. Une contrainte d’unicité n’est pas optionnelle. C’est le mécanisme qui transforme votre architecture d’un folklore optimiste en quelque chose que vous pouvez prouver sous charge.

Voici la règle que j’utilise dans les revues. Si la décision de déduplication n’est pas protégée par la même limite transactionnelle que la mutation, vous n’avez pas l’idempotence. Vous avez de l’espoir.

## Les messages, événements et webhooks ont besoin de leur propre limite

Pour les consommateurs de messages, le modèle classique est toujours le bon. Enregistrez les ID de message traités dans la même transaction de base de données que la mise à jour métier. Chris Richardson décrit directement l’approche de la table `PROCESSED_MESSAGES`, en utilisant une clé primaire sur l’abonné et l’ID de message afin que les doublons échouent proprement et puissent être ignorés.

De nombreuses équipes appellent ce magasin explicite `processed_messages` une table de boîte de réception. L’étiquette importe moins que la règle. Le récepteur doit persister la preuve qu’il a déjà traité le message avant qu’une tentative de relecture ne puisse faire rien en toute sécurité.

Une forme minimale ressemble à ceci :

```
create table processed_messages (
    subscriber_id text not null,
    message_id text not null,
    processed_at timestamptz not null default now(),
    primary key (subscriber_id, message_id)
);
```

Et le flux du consommateur est aussi strict que le flux HTTP :

```
begin transaction

insert into processed_messages (subscriber_id, message_id)
values (?, ?)
on conflict do nothing

if no row inserted
    rollback
    ack and ignore duplicate

apply business mutation

commit
ack message
```

Ce modèle est ennuyeux. Tant mieux. L’idempotence devrait être ennuyeuse.

C’est aussi généralement mieux que d’essayer de s’appuyer sur les termes marketing des courtiers. Le support « exactement une fois » de Kafka est excellent lorsque vous restez dans le modèle transactionnel de Kafka, mais la documentation de Kafka met toujours en garde que les destinations externes ont besoin de coopération. SQS FIFO réduit les envois en double uniquement dans sa fenêtre de déduplication de 5 minutes. Pub/Sub « exactement une fois » s’attend toujours à ce que l’abonné suive la progression et évite le travail en double lorsque les acquittements échouent.

« Exactement une fois » est généralement une optimisation locale. Les effets de bord idempotents sont la garantie du système.

### Associez la déduplication avec le modèle de boîte de sortie

Si votre service met à jour l’état local et publie également un événement, la consommation idempotente seule ne suffit pas. Vous avez également besoin d’un moyen sûr de sortir l’événement après que la transaction locale soit engagée.

C’est pourquoi le modèle de boîte de sortie transactionnelle (transactional outbox) est important. Chris Richardson décrit l’idée de base comme l’écriture de l’événement dans une table de boîte de sortie dans la même transaction que la mise à jour métier, puis sa publication asynchrone. Debezium indique que le modèle de boîte de sortie évite les incohérences entre l’état interne d’un service et les événements consommés par d’autres services. NServiceBus va plus loin et montre comment le traitement de la boîte de sortie déduplique les messages entrants et évite les enregistrements zombies et les messages fantômes.

Voici l’architecture que je recommande pour les services qui possèdent des données et publient des événements d’intégration :

1. Validez et persistez la commande sous une clé d’idempotence.
2. Écrivez l’état métier et l’événement de boîte de sortie dans une seule transaction locale.
3. Laissez le CDC ou un dispatcher de boîte de sortie publier l’événement.
4. Rendez les consommateurs en aval également idempotents.

La boîte de sortie n’élimine pas le besoin de consommateurs idempotents. Elle élimine le besoin de prétendre qu’un engagement de base de données et une publication de courtier peuvent être une transaction distribuée magique unique, ce qui n’est généralement pas possible.

### Les webhooks ne sont que des messages avec un meilleur branding

Traitez les webhooks entrants exactement comme des messages provenant d’une limite de réseau non fiable.

GitHub documente que les livraisons peuvent arriver dans le désordre, recommande d’utiliser `X-Hub-Signature-256` pour vérifier l’authenticité, et fournit `X-GitHub-Delivery` comme identifiant de livraison unique. Il note également que les redistributions réutilisent le même ID de livraison.

L’architecture est donc simple :

- vérifiez la signature en premier
- utilisez le GUID de livraison comme clé de déduplication
- persistez la réception avant les effets de bord
- rendez les gestionnaires conscients de l’ordre plutôt que de supposer l’ordre d’arrivée
- mettez en file d’attente le travail lourd et revenez rapidement

Si votre gestionnaire de webhook écrit directement dans les tables métier avant d’enregistrer la réception, il n’est pas prêt pour la production. Il fait juste plus vite des erreurs en double.

## Les sagas et moteurs de workflow ont encore besoin d’idempotence

Les sagas et les moteurs de workflow durables n’éliminent pas le problème. Ils le rendent visible.

Temporal recommande d’écrire des Activités pour qu’elles soient idempotentes car les Activités peuvent être relancées après des échecs ou des expirations de délai. Sa documentation souligne même le cas limite où un travailleur complète avec succès un effet de bord externe mais s’arrête avant de signaler l’achèvement, ce qui provoque l’exécution de l’Activité à nouveau. Temporal suggère également d’utiliser une combinaison de l’ID d’exécution du Workflow et de l’ID d’Activité comme clé d’idempotence stable lors de l’appel des services en aval. Si vous appliquez cela à l’orchestration de services, [Microservices Go pour l’orchestration IA/ML](https://www.glukhov.org/fr/app-architecture/integration-patterns/go-microservices-for-ai-ml-orchestration-patterns/ "Explorez les modèles éprouvés pour construire des systèmes d’orchestration IA/ML évolutifs en utilisant des microservices Go. Apprenez les architectures orientées événements, les moteurs de workflow et les meilleures pratiques pour les déploiements de production.") couvre les compromis de workflow plus larges.

C’est exactement le bon modèle mental. Un moteur de workflow peut préserver l’historique d’exécution et coordonner les relances. Il ne peut pas rétroactivement annuler la facturation d’une carte ou annuler l’envoi d’un e-mail à moins que votre application ne lui fournisse des étapes idempotentes et des compensations idempotentes.

Il en va de même pour les sagas. Les propres conseils de saga de Temporal décrivent des actions de compensation qui s’exécutent lorsqu’une étape échoue. Ces compensations doivent également être idempotentes. Si « rembourser le paiement » s’exécute deux fois, vous pouvez avoir résolu le bug d’origine en en créant un nouveau.

Ma règle ici est brutale et simple. Chaque Activité, chaque gestionnaire de commande et chaque compensation qui touche le monde extérieur doit soit être naturellement idempotent, soit porter une véritable clé d’idempotence vers le système en aval.

La plupart des équipes testent les chemins heureux et s’étonnent ensuite lorsque des relances ont lieu. Cela ne suffit pas.

Vous devriez avoir des tests automatisés pour au moins ces cas :

- le serveur engage la mutation mais la réponse n’atteint jamais le client
- deux requêtes identiques sont en concurrence avec la même clé d’idempotence
- la même clé est réutilisée avec une charge utile différente
- un consommateur engage son travail de base de données et s’arrête avant l’acquittement
- un webhook est rejoué avec le même ID de livraison
- un dispatcher de boîte de sortie publie le même événement plus d’une fois
- une Activité de workflow complète l’appel externe et s’arrête avant que l’achèvement ne soit signalé
- un enregistrement d’idempotence expire et une relance tardive légitime arrive

AWS recommande explicitement des suites de tests complètes qui incluent des requêtes réussies, des requêtes échouées et des requêtes en double. Ce conseil est banal et absolument correct.

J’ajouterais un exercice de défaillance supplémentaire. Vérifiez que la réponse rejouée est sémantiquement équivalente au premier résultat. AWS discute des relances arrivant tard et plaide pour des réponses qui préservent la signification originale même après que l’état sous-jacent a changé. C’est la différence entre « aucun effet de bord supplémentaire ne s’est produit » et « l’appelant a toujours un contrat cohérent ».

## Des règles d’opinion qui sauvent les systèmes réels

Voici les règles que je ferai respecter lors d’une revue d’architecture.

Premièrement, les clés d’idempotence appartiennent à l’intention métier, pas aux tentatives de transport.

Deuxièmement, scopez chaque clé par locataire et opération. Les espaces de clés globaux sont la manière dont des requêtes non liées entrent en collision.

Troisièmement, persistez la décision de déduplication atomiquement avec la mutation. Si ce n’est pas vrai, la conception est fausse.

Quatrièmement, rejetez les relances avec la même clé mais une charge utile différente. Stripe et AWS font cela pour de bonnes raisons.

Cinquièmement, conservez les clés pour l’horizon de rejoue complet du processus métier, pas pour la fenêtre de file d’attente la plus courte.

Sixièmement, associez les producteurs à une boîte de sortie et les consommateurs au suivi des ID de message. Un côté sans l’autre est une conception à moitié faite.

Septièmement, propagez la même identité d’opération en aval lorsque l’action métier est la même. AWS recommande explicitement de passer le jeton d’idempotence le long de la chaîne de traitement.

Huitièmement, ne supposez jamais que le marketing « exactement une fois » élimine le besoin d’effets de bord idempotents.

Si cela semble strict, tant mieux. L’idempotence est l’endroit où l’architecture optimiste rencontre la réalité de la production. Vous n’avez pas besoin de complexité partout. Mais partout où des effets de bord en double nuiraient à l’argent, à l’état ou à la confiance, l’idempotence devrait être une partie de premier plan du contrat.

## Liens utiles

- [RFC 9110, Sémantique HTTP](https://www.rfc-editor.org/rfc/rfc9110.html)
- [Google Cloud Pub/Sub, Livraison exactement une fois](https://docs.cloud.google.com/pubsub/docs/exactly-once-delivery)
- [Documentation GitHub, Redistribution des webhooks](https://docs.github.com/en/webhooks/testing-and-troubleshooting-webhooks/redelivering-webhooks)
- [Documentation Temporal, Cas d’utilisation et modèles de conception](https://docs.temporal.io/evaluate/use-cases-design-patterns)
- [IETF Datatracker, Le champ d’en-tête HTTP Idempotency-Key](https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/)