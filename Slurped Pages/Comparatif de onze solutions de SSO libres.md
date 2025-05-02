---
link: https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/
site: La Contre-Voie
date: 2024-12-06T09:45
excerpt: "Technique, partie 2 : En quête du SSO parfait"
slurped: 2024-12-19T09:07
title: Comparatif de onze solutions de SSO libres
---

## Technique, partie 2 : En quête du SSO parfait

Table des matières

1. [Avertissement : nous n’avons pas la réponse à tout](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#avertissement-nous-n-avons-pas-la-reponse-a-tout)
2. [Nos critères](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#nos-criteres)
3. [Un logiciel maison](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#un-logiciel-maison)
4. [Keycloak](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#keycloak)
5. [Authelia](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#authelia)
6. [Authentik](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#authentik)
7. [Ory](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#ory)
8. [Casdoor](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#casdoor)
9. [Zitadel](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#zitadel)
10. [Canaille](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#canaille)
11. [LemonLDAP](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#lemonldap)
12. [SSOwat](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#ssowat)
13. [Rauthy](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#rauthy)
14. [Hiboo](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#hiboo)
15. [Finalement… Keycloak](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#finalement-keycloak)
    1. [Un rouage manquant : le captcha](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#un-rouage-manquant-le-captcha)
    2. [L’espoir vain du multi-realm](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#l-espoir-vain-du-multi-realm)
    3. [Moins d’autonomie pour les organisations](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#moins-d-autonomie-pour-les-organisations)
    4. [Pas de gestion des droits d’accès dans Keycloak](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#pas-de-gestion-des-droits-d-acces-dans-keycloak)
16. [Une dernière solution pour la route…](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#une-derniere-solution-pour-la-route)
17. [Conclusion](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#conclusion)

_Dans ce deuxième article consacré à la gestion de notre infrastructure technique, nous allons vous présenter nos travaux de recherche sur des solutions d’authentification unifiée, dans une démarche **d’essaimage** de connaissances et de transparence._

_Comme vous pouvez vous en douter, ces articles sont exceptionnellement destinés à un public expert, mais vous pouvez également retrouver notre article sur [les accompagnements numériques](https://lacontrevoie.fr/blog/2024/accompagnements-numeriques-notre-mode-operatoire/) ou [notre nouvelle offre de services](https://lacontrevoie.fr/blog/2024/la-contre-voie-vous-prepare-sa-nouvelle-offre-de-services/) si vous préférez lire du contenu plus accessible._

Cet article fait suite à [la présentation de notre matériel informatique](https://lacontrevoie.fr/blog/2024/plongee-dans-l-infrastructure-technique-de-la-contre-voie/) dans lequel nous montrions les serveurs que nous autohébergeons, et sera complété par un article sur l’industrialisation de nos services « à la carte ».

Vous aussi, **vous recherchez une solution de SSO** pour vos services numériques, mais vous ne savez pas quels logiciels tester en premier ? Ça tombe bien, **nous avons perdu énormément de temps** sur la question, vous allez donc pouvoir profiter de notre retour d’expérience !

**Le SSO** (Single Sign-On), c’est l’outil que nous souhaitions mettre en place pour **centraliser l’authentification sur tous nos outils** à travers une seule plateforme, de façon à ce que les utilisateur·ices n’aient à se connecter qu’une seule fois, **sur un seul compte**, pour accéder aux services ; à l’image d’un compte Google qui permet d’accéder à tous les services Google (Drive, Docs, Forms, Agenda…).

Il émerge d’un besoin que nous avions **depuis 2019** avec nos membres : une fois leur compte créé sur l’espace membre, nos adhérent·es doivent ensuite encore **créer un compte** sur Nextcloud, puis **encore un nouveau compte** sur Gitea… une procédure longue et rébarbative en somme.

Nous avions mis en place des dispositifs pour faciliter la création de ces comptes, mais il s’agissait essentiellement **de bidouillage** sur l’API d’administration de ces logiciels ; en bref, le compromis n’était pas satisfaisant, et nous a dissuadé d’ajouter de nouveaux services pendant longtemps.

Après quelques années de fonctionnement, nous avons donc entrepris de rechercher des solutions de SSO.

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#avertissement-nous-n-avons-pas-la-reponse-a-tout)Avertissement : nous n’avons pas la réponse à tout

Cet article **n’a pas pour but d’être un référentiel** de SSO libres. Il reflète notre propre expérience de recherche, qui s’est présentée comme un **travail de fond** que nous avons collectivement réalisé à partir du **début de l’année 2023**. Nous avons mis **plus d’un an** pour tester, contribuer, améliorer, puis choisir notre solution.

[![Mème : un personnage perdu dans une grotte trouve enfin le parchemin de la vérité dans un coffre au trésor après 15 ans de recherche, à son grand enthousiasme. Le parchemin indique « le SSO parfait n’existe pas ». Dans le déni et la colère de cette information insatisfaisante, le personnage jette le parchemin en criant « NYEHHH ». C’est littéralement moi, là. (neil)](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/meme-sso.jpg)](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/meme-sso.jpg)

Vous n’imaginez pas un seul instant à quel point nous en avons bavé pour parvenir à cette conclusion…

Certaines solutions **se sont améliorées** depuis nos tests, et ont intégré des fonctionnalités qui étaient importantes pour nous et qui n’étaient pas là au moment où nous avons testé le logiciel. **Nous ne mettrons pas à jour cet article** au fil des améliorations de ces solutions.

Si toutefois vous trouvez que disposer d’un **comparatif actualisé** de SSO libres serait pertinent, nous vous encourageons à **créer votre propre comparatif de SSO** qui sera sans doute d’une grande aide pour les structures qui les recherchent ; libre à vous de vous baser sur nos retours d’expérience à cette fin.

Et enfin, nous écrivons cet article avec des biais évidents de jeunes passionné·es par les solutions innovantes que nous sommes :

- un penchant pour la **sobriété** (mémoire, CPU) ;
- pas trop d’envie de maintenir un annuaire LDAP ;
- une appétence pour les **interfaces graphiques modernes** et épurées ;
- et une **petite préférence pour les langages modernes** (Go, Rust…) plutôt que les anciens (Java, Perl), sans pour autant que cela pèse trop dans notre jugement, et sans vouloir relancer une guerre de langages.

**Nous ne prétendons pas rendre un avis impartial**, prenez donc nos retours avec un grain de sel !

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#nos-criteres)Nos critères

Nous avions une idée assez précise de ce que nous souhaitions, c’est peut-être aussi pour ça que les recherches se sont avérées peu concluantes.

**Nos contraintes majeures :**

- La solution doit évidemment être libre et open-source : cela élimine un bon nombre d’options, à commencer par **Microsoft Active Directory** et **[Okta/Auth0](https://auth0.com/)**.
- Il doit être possible **d’importer notre base de données existante** : les mots de passe de notre base initiale sont hachés avec `bcrypt`, et nous ne souhaitons pas demander à tous nos membres de réinitialiser leur mot de passe ;
- La solution doit supporter le protocole **OpenID Connect (OIDC)** en tant que **fournisseur**, pas nécessairement en tant que client. Nous souhaitons réaliser l’authentification avec tous nos outils via OIDC.
- Le logiciel doit être activement maintenu ou développé ;
- **L’empreinte mémoire** de la solution doit être **légère** :
    - pas plus de CPU qu’une simple application web ;
    - pas plus de 500 Mo de RAM pour le SSO et le support OIDC, et moins de 150 Mo dans l’idéal.

**Nos contraintes secondaires :**

- Gestion complète du **flow d’authentification** : connexion, oubli de mot de passe, changement d’adresse mail et de nom d’utilisateur·ice ;
- Gestion complète du **flow d’inscription** : création du compte, validation de l’adresse mail, **captcha libre** ;
- Authentification à **deux facteurs** (TOTP) ;
- Fonctionne en autonomie avec sa base de données : ne nécessite **pas de LDAP** (PostgreSQL nous convient) ;
- Traduit au minimum **en anglais et en français** ;
- Système de **permissions / restriction d’accès** à certains services en fonction de l’appartenance à des groupes : par exemple, les utilisateur·ices d’un groupe « admin » pourraient se connecter sur certains services inaccessibles pour les autres.

**Nos bonus :**

- **Champs personnalisables** à l’inscription : on peut ajouter des champs pour chaque utilisateur·ice et en désactiver certains (notamment les champs nom et prénom)
- Espace membre intégré (mais désactivable) avec des options personnalisables : portail d’accès aux services, possibilité d’afficher le statut de l’adhésion avec des bouts de code personnalisés…
- Fonctionne en autonomie totale : un seul conteneur pour tout, ou à la rigueur deux conteneurs : un pour le SSO et un pour OIDC, par exemple. La base de données peut être également séparée.

Nous verrons par la suite que certains critères secondaires se sont avérés plus importants, et d’autres critères majeurs ont finalement été dépriorisés.

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#un-logiciel-maison)Un logiciel maison

Au début, nous avons brièvement envisagé l’idée de développer un **logiciel maison** dédié à l’authentification. Nous avons des connaissances en [Rust](https://www.rust-lang.org/), nous avons plusieurs projets qui utilisent le framework web [Actix](https://actix.rs/) et l’authentification demande de concevoir assez peu de pages web − nous _aurions pu_ partir sur notre propre outil.

Mais deux réserves nous ont fait changer d’avis…

**La gestion de OpenID Connect.** Il existe un paquet en Rust pour créer un serveur OIDC nommé [oxide-auth](https://github.com/HeroicKatora/oxide-auth/), mais son développement n’est pas actif et il manque de fonctionnalités cruciales liées à l’implémentation du protocole OIDC ; en bref, il est assez incomplet et peu maintenu. Nous aurions pu **réimplémenter OIDC** nous-mêmes, mais ça aurait représenté une charge de travail supplémentaire.

**Le flow d’authentification et d’inscription.** En **formalisant nos critères**, nous avons réalisé qu’écrire cet outil ne serait pas une mince affaire si l’on souhaite qu’il soit complet : l’inscription demande d’envoyer un e-mail de confirmation, l’authentification pourrait demander l’implémentation de TOTP, on pourrait vouloir implémenter un captcha… il faut ajouter à cela **la localisation** (traduction), la gestion de l’oubli de mot de passe, les permissions… et ça commence à faire beaucoup pour nos épaules.

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#keycloak)Keycloak

![Logo de Keycloak](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/keycloak.png)

Nous connaissions déjà [Keycloak](https://www.keycloak.org/) − c’est la solution open-source la plus connue quand on parle de SSO. On ne peut pas passer à côté.

Leader dans son domaine, Keycloak est une **solution tout-en-un**, supporte tous les standards essentiels, est bourré de fonctionnalités… Mais c’est aussi l’un de ses défauts : il y a tellement de fonctionnalités dans le logiciel qu’il en devient une usine à gaz. Mais cela ne nous a pas empêché d’essayer l’outil.

En 2023, nous avons testé Keycloak en version 22.0.1, avec l’image Docker officielle pesant **441 Mo** − beaucoup trop à notre goût.

Démarrer une instance de développement de Keycloak (en s’inspirant des instructions données sur le [README](https://github.com/keycloak/keycloak)) annonce la couleur :

```
NAME       CPU %   MEM USAGE / LIMIT   NET I/O       BLOCK I/O       PIDS
keycloak   0.64%   469.3MiB / 800MiB   1.05kB / 0B   165MB / 179MB   45
```

À peine lancé, le conteneur consomme déjà **470 Mo de RAM**, ce qui nous semble outrageusement élevé.

Le logiciel est écrit en Java, et ça ne nous donnait pas du tout envie. On souhaitait **une solution légère**, dans un langage moderne, avec juste assez de fonctionnalités pour nos besoins. Keycloak, c’est **l’exact opposé** de cette description.

Nous sommes donc parti·es à la recherche d’alternatives à Keycloak… et nous vous expliquerons à la fin de l’article pourquoi nous sommes finalement **revenu·es sur ce choix**.

|Critère|Keycloak|
|---|---|
|Libre / open-source|✅ Apache 2.0|
|Légèreté (RAM)|❌ 500 Mo environ|
|Légèreté (image)|❌ 500 Mo environ|
|Légèreté (CPU)|✅ À peu près OK|
|Support OIDC|✅|
|Backend fiable autre que LDAP|✅|
|Import MDP bcrypt possible|✅|
|Formulaire de connexion|✅|
|Formulaire d’inscription|✅|
|Validation par mail|✅|
|Oubli de mot de passe|✅|
|2FA TOTP|✅|
|Captcha libre|❌ Pas de captcha libre|
|Localisation (FR/EN)|✅|
|Personnalisation du CSS|✅ Dans une certaine mesure…|
|RBAC (permissions)|⚠️ Ne permet pas de restreindre l’accès aux apps|
|Espace membre intégré|✅|
|Simple à déployer|✅|
|Champs personnalisables|✅ Très personnalisables|
|Développement actif|✅|
|Réputation du logiciel|✅ 23.6k étoiles, leader|
|Langage|❌ Java|

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#authelia)Authelia

![Logo d’Authelia](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/authelia.png)

Nous avons rapidement trouvé **[Authelia](https://www.authelia.com/)**.

Écrit en Go − langage moderne et disruptif − le logiciel consomme **moins d’une centaine de Mo** en RAM. Son image Docker pèse à peine **20 Mo** ; son beau design et sa légèreté l’honorent.

Malgré tout, nous y relevons deux éléments bloquants…

**Il nécessite un annuaire LDAP.** En dépit de ses airs de modernité, Authelia ne supporte que deux types de bases de données pour authentifier les utilisateur·ices :

- **[LDAP](https://www.authelia.com/configuration/first-factor/ldap/)**, pour lequel [OpenLDAP](https://www.openldap.org/) peut être utilisé, mais que notre frilosité à l’idée de maintenir un annuaire LDAP nous défend de choisir ;
- **[un fichier de configuration](https://www.authelia.com/configuration/first-factor/file/)** en YAML, dans lequel on ajoute manuellement les utilisateur·ices (autant dire que c’est pratique pour des tests, mais pas du tout pour de la production).

**Pas de formulaire d’inscription.** Pour se créer un compte, un·e admin doit intervenir, les comptes ne peuvent pas être créés par les utilisateur·ices en autonomie. Il n’y a même pas [d’interface d’administration](https://github.com/authelia/authelia/issues/303), il faut nécessairement modifier le LDAP ou le fichier YAML.

Mais nous avons persisté avec Authelia. En avril 2023, il nous est venue l’idée folle de créer [un petit logiciel](https://git.lacontrevoie.fr/lacontrevoie/authelia-manager) pour servir de **formulaire d’inscription** et **d’interface d’administration**, qui ajouterait et modifierait des entrées au fichier YAML des utilisateur·ices que nous projetions d’utiliser en production. Denise et Monique étaient sur le projet, mais nous avons fini par abandonner cette idée un peu tarabiscotée.

Nous avons également observé dans les réponses aux [tickets](https://github.com/authelia/authelia/issues) du projet que les développeur·euses sont très frileux·ses à l’idée d’accepter des modifications de leur code (notamment, des backends d’authentification autres que LDAP) « pour des raisons de sécurité », nous laissant peu d’espace à la contribution en cas de besoin.

|Critère|Authelia|
|---|---|
|Libre / open-source|✅ Apache 2.0|
|Légèreté (RAM)|✅ Moins de 100 Mo|
|Légèreté (image)|✅ Moins de 50 Mo|
|Légèreté (CPU)|✅ Rapide et sobre|
|Support OIDC|✅|
|Backend fiable autre que LDAP|❌ YAML ne compte pas !|
|Import MDP bcrypt possible|✅|
|Formulaire de connexion|✅|
|Formulaire d’inscription|❌|
|Validation par mail|✅|
|Oubli de mot de passe|✅|
|2FA TOTP|✅ WebAuthn aussi|
|Captcha libre|❌ Pas de captcha|
|Localisation (FR/EN)|✅ [Oui](https://www.authelia.com/reference/guides/server-asset-overrides/#locales)|
|Personnalisation du CSS|❌ Nécessite une recompilation du binaire|
|RBAC (permissions)|✅ [Oui](https://www.authelia.com/configuration/security/access-control/)|
|Espace membre intégré|❌|
|Simple à déployer|✅|
|Champs personnalisables|❌|
|Développement actif|✅|
|Réputation du logiciel|✅ 21.8k étoiles|
|Langage|✅ Go|

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#authentik)Authentik

![Logo d’Authentik](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/authentik.png)

**[Authentik](https://goauthentik.io/)** s’est mis en valeur dans nos recherches, notamment parce qu’il [se compare](https://goauthentik.io/#comparison) ostensiblement à Keycloak et Authelia sur sa page d’accueil, en vantant ses nombreuses fonctionnalités.

Et il y a de quoi séduire : cette solution de SSO dispose d’un **formulaire de création de compte** et semble même pouvoir se passer complètement d’un annuaire LDAP en utilisant sa propre base de données. Il permet aussi de modifier assez facilement les pages servies − l’ajout de feuilles CSS est très facile.

Écrit en Python, ce programme semblait avoir tout pour plaire. Seul point noir à première vue : il intègre un système de captcha, mais ne supporte que des **captchas propriétaires** (hCaptcha, reCAPTCHA…). Il pourra être envisagé de développer notre propre intégration en cas de besoin.

Nous avons donc testé de le mettre en place. Son installation nécessite une base de données PostgreSQL et un cache Redis. Il se déploie nécessairement en deux conteneurs : un serveur et un worker.

La taille de l’image Docker `ghcr.io/goauthentik/server` est d’abord assez surprenante : **802 Mo** dans sa version `2023.6.1`, à ce demander ce qui peut prendre autant de place dedans.

Le déploiement révèle alors la vraie nature de Authentik :

```
NAME               CPU %     MEM USAGE / LIMIT    NET I/O           BLOCK I/O         PIDS
authentik-worker   0.05%     401.8MiB / 500MiB    582kB / 516kB     115kB / 0B        9
authentik-server   1.11%     473.2MiB / 500MiB    390kB / 244kB     25.2MB / 0B       21
authentik-redis    0.27%     4.258MiB / 300MiB    8.58MB / 3.54MB   4.71MB / 86kB     5
```

Malgré la limite de RAM à **500 Mo** sur les deux conteneurs, ils se réservent chacun a minima 400 Mo de RAM **à vide** : les conteneurs viennent d’être redémarrés et aucune page web n’a encore été consultée.

Comme quoi, une application peut être gourmande sans être écrite en Java : ce logiciel consomme au minimum deux fois plus de ressources que Keycloak.

|Critère|Authentik|
|---|---|
|Libre / open-source|✅ [MIT](https://github.com/goauthentik/authentik/blob/main/LICENSE) (pour la version gratuite)|
|Légèreté (RAM)|❌ 800+ Mo à vide : monstrueux|
|Légèreté (image)|❌ 800 Mo : mammouth|
|Légèreté (CPU)|❓ Pas testé|
|Support OIDC|✅|
|Backend autre que LDAP|✅ PostgreSQL|
|Import MDP bcrypt possible|❓ Pas testé|
|Formulaire de connexion|✅|
|Formulaire d’inscription|✅|
|Validation par mail|✅|
|Oubli de mot de passe|✅|
|2FA TOTP|✅|
|Captcha libre|❌ reCaptcha / hCaptcha|
|Localisation (FR/EN)|✅|
|Personnalisation du CSS|✅|
|RBAC (permissions)|✅|
|Espace membre intégré|❓|
|Simple à déployer|✅|
|Champs personnalisables|❓|
|Développement actif|✅|
|Réputation du logiciel|✅ 13.6k étoiles|
|Langage|✅ Python|

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#ory)Ory

![Logo d’Ory](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/ory.png)

Une autre solution populaire se nomme [Ory](https://github.com/ory/), avec ses composants [Hydra](https://github.com/ory/hydra/) et [Kratos](https://github.com/ory/kratos/).

Écrite en Go et intégrant un **formulaire d’inscription**, elle se présente comme naturellement séduisante… jusqu’au moment du déploiement en local.

Denise, membre de notre Comité de Contribution, a déployé [une instance fonctionnelle](https://git.lacontrevoie.fr/denise/ory-account-experience) du logiciel. Sa consommation en RAM est assez faible.

Elle nécessite **quatre conteneurs** :

- `ory-kratos`, le serveur d’identité, qui gère l’annuaire des utilisateur·ices et s’interface avec la base de données PostgreSQL (qui vous nécessitera de fait un conteneur de plus).
- `ory-kratos-ui`, l’interface graphique du serveur d’identité.
- `ory-hydra`, le module OpenID Connect qui utilise Kratos comme backend d’authentification.
- `ory-hydra-consent`, le module de Hydra qui affiche une interface pour demander le consentement des utilisateur·ices lors de l’accès à une application.

La mise en place de ces conteneurs **est une plaie**, Denise a dû recompiler chaque module et réécrire un Dockerfile pour parvenir à une configuration fonctionnelle. Les fichiers de configuration ne sont pas évidents à prendre en main (notamment [le fichier d’identité](https://www.ory.sh/docs/kratos/manage-identities/identity-schema)).

Et cerise sur le gâteau : au moment où nous réalisions nos tests, il n’était [pas possible](https://github.com/ory/elements/issues/105) de traduire le logiciel sans un fork d’un autre module, [ory/elements](https://github.com/ory/elements/).

En bref : même si elle consomme peu de ressources, cette solution est **une véritable usine à gaz**, et il semblerait que toute la procédure d’installation est là pour nous dissuader de l’autohéberger et partir sur la solution payante, hébergée par l’entreprise. C’est la conclusion à laquelle nous sommes parvenu·es en mars 2023.

|Critère|Ory / Kratos / Hydra|
|---|---|
|Libre / open-source|✅ Oui (pour la version gratuite)|
|Légèreté (RAM)|✅ Oui (moins de 150 Mo)|
|Légèreté (image)|✅|
|Légèreté (CPU)|✅|
|Support OIDC|✅|
|Backend autre que LDAP|✅|
|Import MDP bcrypt possible|✅|
|Formulaire de connexion|✅|
|Formulaire d’inscription|✅|
|Validation par mail|✅|
|Oubli de mot de passe|✅|
|2FA TOTP|✅|
|Captcha libre|❌ Pas de captcha|
|Localisation (FR/EN)|❌ Inexistante au moment des tests|
|Personnalisation du CSS|❌ Recompilation nécessaire|
|RBAC (permissions)|❓|
|Espace membre intégré|✅|
|Simple à déployer|❌ L’Enfer sur Terre|
|Champs personnalisables|✅|
|Développement actif|✅|
|Réputation du logiciel|✅ 15.6k étoiles|
|Langage|✅ Go|

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#casdoor)Casdoor

![Logo de Casdoor](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/casdoor.png)

Pendant nos recherches, nous sommes tombé·es sur la page [awesome-auth](https://github.com/casbin/awesome-auth) qui répertorie les solutions d’authentification libres. Casdoor est systématiquement listé en premier dans de nombreuses catégories.

Nous avons rapidement réalisé que **[Casdoor](https://casdoor.org/)**, un SSO open-source, est un projet lié à [Casbin](https://casbin.org/), et que les développeur·euses de ces deux logiciels **sont les propriétaires du dépôt awesome-auth**. Ils se sont donc eux-mêmes positionnés en première alternative partout, pour promouvoir leur solution…

Malgré ce que nous qualifions comme manque d’honnêteté de leur part, nous nous avons essayé leur solution. Leur documentation est assez mal rédigée, apparemment mal traduite du chinois, ce qui nous a causé des difficultés pour configurer Casdoor.

Mais une fois configuré, le logiciel **faisait rêver** : il y avait presque toutes les fonctionnalités dont nous avions besoin ! Et les solutions manquantes semblaient facilement intégrables, car le logiciel est **hautement modulaire**.

La consommation en RAM du logiciel est négligeable, et des fonctionnalités particulièrement intéressantes qu’on ne trouve pas partout ailleurs étaient bien travaillées (paiements intégrés, gestion multi-organisations…). Les nouvelles versions sont fréquentes et les fonctionnalités ajoutées sont pertinentes.

Neil a même fini par réaliser une présentation de Casdoor au Camp CHATONS 2023 en la présentant comme une excellente solution.

Parmi ce qui manquait :

- **La traduction française** était très mauvaise (`Home` traduit en `Maison` plutôt que `Accueil`…), ce pour quoi nous nous sommes mobilisé·es pour contribuer à la traduction sur leur [Crowdin](https://crowdin.com/project/casdoor-site).
- **Il y a un captcha libre, mais peu sécurisé** ; nous avons souhaité intégrer [mCaptcha](https://github.com/mCaptcha/mCaptcha/) en contribuant au code, ce qui semble plutôt accessible car le code est très modulaire et que leur logiciel supporte déjà de nombreux captchas.

Puis, peu à peu, nous avons commencé à découvrir les points négatifs de cette solution.

Le premier point négatif : les nouvelles versions sont **[beaucoup trop fréquentes](https://github.com/casdoor/casdoor/releases)**. Il y a globalement **une nouvelle version par commit**. Le versionnage est chaotique, il ne s’agit pas d’un [versionnage sémantique](https://semver.org/) mais d’une variante un peu abstraite qui ne semble pas avoir trop de sens.

En trois mois, le logiciel change tellement qu’il y a pas mal de travail de reconfiguration à faire, et les mises à jour cassent assez régulièrement l’installation.

Par ailleurs, nous avons identifié plusieurs problèmes de sécurité sur Casdoor, à commencer par sa politique de conservation des mots de passe : le logiciel supporte de nombreux algorithmes de dérivation de mot de passe (dont `bcrypt`), mais stocke les **mots de passe en clair par défaut**, au moment de nos tests.

Par ailleurs, Casdoor **diffuse énormément d’informations à travers son API**, parfois des informations sensibles. Hannaeko, membre de notre Comité de Contribution, a repéré une faille assez sérieuse : la réponse de tous les capatchas du logiciel est [divulguée via l’API](https://github.com/casdoor/casdoor/pull/2471)…

Le bug a été résolu un mois et demi plus tard, mais nous étions déjà parti·es sur une autre solution car la qualité du code et la stabilité des fonctionnalités ne nous ont pas entièrement convaincu·es.

|Critère|Casdoor|
|---|---|
|Libre / open-source|✅ Oui (pour la version gratuite)|
|Légèreté (RAM)|✅ Oui (moins de 150 Mo)|
|Légèreté (image)|✅ Oui (45 Mo)|
|Légèreté (CPU)|✅|
|Support OIDC|✅|
|Backend autre que LDAP|✅|
|Import MDP bcrypt possible|✅|
|Formulaire de connexion|✅|
|Formulaire d’inscription|✅|
|Validation par mail|✅|
|Oubli de mot de passe|✅|
|2FA TOTP|✅|
|Captcha libre|✅ Oui, mais assez facile à casser|
|Localisation (FR/EN)|⚠️ Oui, mais doit être retravaillée|
|Personnalisation du CSS|✅|
|RBAC (permissions)|✅ Assez complexe à prendre en main|
|Espace membre intégré|✅|
|Simple à déployer|✅ Oui, malgré la documentation de mauvaise qualité|
|Champs personnalisables|✅|
|Développement actif|⚠️ Oui, mais la publication des nouvelles versions est chaotique|
|Réputation du logiciel|✅ 10.2k étoiles|
|Langage|✅ Go|
|Points bloquants|❌ Sécurité prise à la légère, peu de stabilité entre les mises à jour|

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#zitadel)Zitadel

![Logo de Zitadel](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/zitadel.png)

Nous avons également testé [Zitadel](https://zitadel.com/), qui se présente comme une alternative émergente aux autres SSO et qui semble s’améliorer (très lentement) avec le temps.

**Les très bons points :** il intègre un formulaire d’inscription et fonctionne avec PostgreSQL, et il est relativement léger.

**Ce qui nous a dissuadé de l’utiliser :** le manque de personnalisation des champs utilisateur·ices (nom et prénom obligatoire) et l’absence totale de support pour un captcha.

Pour les champs personnalisables, [un ticket est ouvert](https://github.com/zitadel/zitadel/issues/6433) depuis 2023 et sera sans doute résolu d’ici la publication de **Zitadel V3**, mais c’est toujours pareil : on ne sait jamais quand ça sortira, impossible de se projeter, et en attendant la fonctionnalité n’est pas là.

|Critère|Zitadel|
|---|---|
|Libre / open-source|✅ Apache 2.0|
|Légèreté (RAM)|✅|
|Légèreté (image)|✅ 130 Mo|
|Légèreté (CPU)|✅|
|Support OIDC|✅|
|Backend autre que LDAP|✅ PostgreSQL|
|Import MDP bcrypt possible|✅|
|Formulaire de connexion|✅|
|Formulaire d’inscription|✅|
|Validation par mail|✅|
|Oubli de mot de passe|✅|
|2FA TOTP|✅|
|Captcha libre|❌ [Pas de captcha](https://github.com/zitadel/zitadel/issues/6658)|
|Localisation (FR/EN)|✅ [Oui](https://zitadel.com/docs/guides/manage/customize/texts#internationalization--i18n)|
|Personnalisation du CSS|✅|
|RBAC (permissions)|❓|
|Espace membre intégré|❓|
|Simple à déployer|✅|
|Champs personnalisables|❌ Pas tellement [(1)](https://github.com/zitadel/zitadel/issues/3965), [(2)](https://github.com/zitadel/zitadel/issues/6433)|
|Développement actif|✅|
|Réputation du logiciel|✅ 9k étoiles|
|Langage|✅ Go|

_Pour les logiciels suivants, nous ne pourrons pas vous fournir de tableau récapitulatif car nous ne les avons pas suffisament testés, ou cela remonte à trop longtemps._

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#canaille)Canaille

![Logo de Canaille](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/canaille.png)

[Canaille](https://canaille.yaal.coop/), c’est un petit logiciel développé par un membre du [collectif CHATONS](https://www.chatons.org/), [Yaal Coop](https://yaal.coop/) (le membre CHATONS en question étant Nubla, développé par cette coopérative).

Canaille est écrit en **Python** et utilise un **backend LDAP**, mais il semble avoir récemment intégré le support d’un [backend SQL](https://gitlab.com/yaal/canaille/-/issues/30).

Au départ, le logiciel n’intégrait pas de formulaire d’inscription, et il s’agissait d’un point bloquant pour nous. Denise a donc essayé de [contribuer directement au code](https://gitlab.com/yaal/canaille/-/merge_requests/133) pour ajouter cette fonctionnalité, mais l’implémentation n’était pas entièrement satisfaisante pour les développeur·euses qui souhaitaient d’abord ajouter d’autres fonctionnalités qui pourraient en dépendre.

Pour éviter de créer à nouveau de l’inertie dans nos recherches, nous avons passé notre chemin et testé d’autres solutions.

Aujourd’hui, Canaille semble avoir intégré le formulaire d’inscription et pourrait être une potentielle solution décente, mais semble peiner à trouver sa communauté et son public.

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#lemonldap)LemonLDAP

Nous n’avons pas testé [LemonLDAP](https://lemonldap-ng.org/), mais c’est une solution couramment suggérée.

Il s’agit d’un serveur OpenID Connect qui utilise un annuaire LDAP intégré. Il dispose de nombreuses fonctionnalités recherchées dont le **formulaire de création de compte** et un captcha libre. A priori, il s’agit d’une solution assez éprouvée, le logiciel semble avoir été créé en 2006 et est maintenu par des français·es.

Le fait que ce logiciel soit **écrit en Perl** et qu’il fonctionne avec **LDAP** nous a en revanche un peu refroidi. Ce n’était pas forcément un argument éliminatoire, mais le fait est que nous avons finalement testé d’autres solutions car nous avions un temps limité et nous ne pouvions pas tester tous les logiciels ; nous ne nous sommes pas attardé·es sur celui-ci.

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#ssowat)SSOwat

[SSOwat](https://github.com/YunoHost/SSOwat), c’est la solution d’authentification intégrée au **[projet Yunohost](https://yunohost.org/)**, que nous avons très brièvement examinée et aussi vite écartée.

Il s’agit d’une **extension de Nginx** écrite en **Lua** et utilise un **backend LDAP**.

Il se trouve que son fonctionnement est assez **intrinsèquement lié** à Yunohost, et qu’il semble assez difficile de l’en détacher − d’autant plus que nous utilisons [Caddy](https://caddyserver.com/), et pas Nginx, comme reverse-proxy pour nos services.

Le logiciel a pas mal changé avec [la version 12](https://forum.yunohost.org/t/yunohost-12-0-bookworm-release-sortie-de-yunohost-12-0-bookworm/31673) de Yunohost, le module SSO a été décomposé en trois modules distincts, ce qui ne va pas pour autant faciliter son installation dans d’autres environnements que dans Yunohost lui-même.

Il reste une solution sans doute viable, à condition que votre infrastructure tourne **entièrement sous Yunohost**.

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#rauthy)Rauthy

Dans la catégorie des petits projets sympathiques, nous avons examiné [Rauthy](https://github.com/sebadob/rauthy), un SSO écrit en Rust.

Le logiciel semble disposer d’un certain nombre de fonctionnalités dont nous avions besoin, dont le formulaire d’inscription. Il dépend d’un backend PostgreSQL.

Il a l’avantage d’être **extrêmement léger** et rapide. Pour un SSO, il est très sobre et efficace.

Cela dit, le projet est encore beaucoup trop jeune à notre goût : **maintenu par une seule personne**, pas entièrement abouti en termes de design et de fonctionnalités, il s’agit encore d’une solution un peu bancale, bien qu’elle s’améliore d’année en année.

Le logiciel n’a **pas (encore) de communauté**, donc très peu de support et de retours d’expérience également, et cela joue inévitablement en sa défaveur dans la balance.

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#hiboo)Hiboo

Toujours avides des projets expérimentaux, nous avons regardé le projet [Hiboo](https://acides.org/docs/hiboo/) du CHATONS [TeDomum](https://tedomum.net/).

Nous ne sommes pas les premièr·es à avoir songé à réinventer la roue : [TeDomum présente Hiboo en 2019](https://write.tedomum.net/tedomum/hiboo-re-inventons-la-roue-de-lauthentification) et tente d’adapter à leur sauce les méthodes d’authentification unifiée.

Le projet, écrit en Python, est expérimental mais globalement fonctionnel, selon TeDomum qui l’utilise en production.

Fin 2023, ses développeur·euses [dressent le bilan](https://forge.tedomum.net/acides/hiboo/-/issues/119) de cette expérimentation et envisagent deux possibilités :

1. d’abandonner le développement de Hiboo et préparer une migration de leur base vers des alternatives (Authelia, Keycloak, Authentik) ;
2. de se donner les moyens d’avoir un modèle de développement soutenable : avoir une communauté active, payer un·e dev…

Ici s’arrête notre suivi du projet, mais il semble encore vivoter, vous pouvez [consulter leur dépôt](https://forge.tedomum.net/acides/hiboo) pour en savoir plus.

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#finalement-keycloak)Finalement… Keycloak

Nos recherches ont commencé par Keycloak, et se terminent sur Keycloak. C’était bien la peine…

C’est notre solution de repli, qui n’est pas franchement si mauvaise car extrêmement répandue, ce qui présente un **avantage majeur** : si l’on rencontre un problème, nous pouvons à peu près être sûr·es que **quelqu’un d’autre l’a déjà rencontré** et qu’il existe une solution.

Concernant sa consommation en RAM : nous avons constaté, et corroboré avec d’autres témoignages, que même si Keycloak consomme **un minimum irréductible 500 Mo de RAM**, il ne semble pas monter bien au-delà :

[![Un graphique tiré du logiciel de supervision Grafana pour visualiser la consommation de RAM de Keycloak. Il stagne à 700 Mo en moyenne, puis redescend à 600 Mo après un redémarrage.](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/keycloak-ram.png)](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/keycloak-ram.png)

La consommation en RAM de notre Keycloak sur une semaine. Il traite une trentaine de connexions différentes par jour.

Nous avons redémarré manuellement le service le 20 novembre.

Avec une limite de 1 Go de RAM, le logiciel devrait tourner sans problème et consommer assez peu de CPU en moyenne. **1 Go, c’est vraiment beaucoup** pour nous (un quart de la RAM pour la plupart de nos machines…), mais nous avons fini par accepter ce douloureux compromis.

Mais ce n’était pas le seul sacrifice que nous avons dû accepter.

#### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#un-rouage-manquant-le-captcha)Un rouage manquant : le captcha

Nous ne pouvons pas encore déployer Keycloak en remplacement de l’espace membre le temps que nous développions un **captcha** pour Keycloak, outil de protection qui nous semble **absolument indispensable** pour protéger notre service.

Actuellement, Keycloak supporte officiellement reCaptcha (Google), et la communauté a développé un plugin pour supporter hCaptcha (Cloudflare). Nous aimerions y ajouter le support de [mCaptcha](https://mcaptcha.org/) (peu maintenu, mais c’est notre solution pour le moment), ce qui nous nécessiterait de **coder un plugin en Java** (_là, neil s’arrache les cheveux_).

Sans cette brique, nous utilisons Keycloak **uniquement pour nos accompagnements numériques** puisqu’ils n’incluent pas de formulaire de création de compte. Nous pourrons l’utiliser pour remplacer notre espace membre une fois que cette brique sera écrite.

#### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#l-espoir-vain-du-multi-realm)L’espoir vain du multi-realm

Un « realm », ou _royaume_, c’est un terme de Keycloak pour désigner un environnement dans lequel une organisation peut créer ses propres **applications**, avoir ses propres **utilisateur·ices**, ses propres **groupes**…

Keycloak permet de créer **plusieurs realms**, ce qui permet d’héberger de nombreuses organisations différentes qui ont chacune leurs applications et leurs utilisateur·ices. C’est utile pour fournir des services à beaucoup d’organisations différentes.

Mais le plus grand atout de ce fonctionnement est également son plus gros défaut : les applications et les utilisateur·ices ne peuvent pas être **partagées entre plusieurs realms**.

Par exemple, si une personne est membre de l’organisation « Le Bêta », mais **aussi** de l’organisation « Maison des Peuples et de la Paix », elle devra créer **deux comptes différents** pour accéder aux services de ces organisations respectives. Et si elle devient membre de La Contre-Voie, il lui faudra un **troisième compte**. Quel est l’intérêt d’avoir une authentification unifiée s’il faut créer de nombreux comptes par personne dessus ?

Mais le plus problématique, c’est sans doute pour les **applications** : imaginons que nous souhaiterions mettre à disposition **un service partagé entre toutes les organisations**, par exemple un outil de visioconférence ou un service de pads… comment devrions nous procéder dans cet environnement ?

La réponse est simple : il faudrait déployer **autant d’instances de cet outil que de realms existants**, car presque tous les logiciels qui supportent OpenID Connect ne peuvent se connecter qu’à un seul serveur OIDC par instance. Pourtant, dans ce cas d’usage précis, il s’agirait d’un immense **gâchis de ressources**.

Nous avons donc fait le choix − à mi-chemin − **d’héberger tout le monde sur un seul realm**.

#### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#moins-d-autonomie-pour-les-organisations)Moins d’autonomie pour les organisations

Avoir tout le monde sur un même realm impose une sérieuse contrainte : la gestion des membres par des tiers (ex.: des administrateur·ices parmi les structures que nous hébergeons) n’est **plus possible**.

Lorsque chaque organisation possède son realm, nous n’avons aucun problème à **donner des droits d’administration sur Keycloak** à des responsables au sein de ces organisations, car ces droits sont **cloisonnés** au sein de leur realm. Une personne « référente » pouvait ainsi ajouter, modifier ou supprimer des membres **en toute autonomie**, et même leur donner ou retirer des **droits d’accès** pour des services au sein de leur organisation, sans nous solliciter.

Si tout le monde se situe dans le même realm, donner des droits d’administration à un·e référent·e revient à lui donner les droits sur **toutes les organisations** que nous hébergeons, y compris nos utilisateur·ices, ce qui n’est absolument pas envisageable.

Pour l’instant, la solution palliative que nous avons choisie consiste à demander à ce que chaque organisation **nous envoie un e-mail** avec la **liste des membres** qui sont censés avoir accès à leurs services, avec leurs **droits respectifs**, et ce à chaque modification de cette liste… puis nous reportons cette liste nous-mêmes en production. On ne va pas vous raconter le travail de support que cela incombe…

À terme (et assez rapidement), il nous faudra sans doute **concevoir un middleware** qui fera l’interface entre les membres référent·es de chaque organisation et l’API d’administration de Keycloak, leur permettant de gérer leurs membres et leurs droits de manière **encadrée et restreinte**.

#### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#pas-de-gestion-des-droits-d-acces-dans-keycloak)Pas de gestion des droits d’accès dans Keycloak

Aussi incroyable que cela puisse paraître, Keycloak **ne permet pas de gérer des droits d’accès** à une application pour certain·es utilisateur·ices ou certains groupes. Tous les membres d’un realm ont **accès** à toutes les applications.

Les développeur·euses de Keycloak ont décrété que cela ne fait pas partie du périmètre du logiciel, **car Keycloak s’occupe uniquement de l’authentification, pas de l’autorisation** [(1)](https://keycloak.discourse.group/t/allowing-keycloak-login-to-clients-depending-on-a-keycloak-group-belonging/16684), [(2)](https://keycloak.discourse.group/t/restrict-login-to-specific-group-for-one-client/24105) ; que l’autorisation de l’utilisateur·ice relève donc du rôle de l’application.

Il existe bien [un plugin](https://github.com/sventorben/keycloak-restrict-client-auth) à cet usage, mais il peut être assez facilement contourné dans certains cas et n’est donc **pas du tout** une solution sécurisée.

Or, les applications ne permettent **pas toutes** de restreindre leur accès en fonction des groupes des utilisateur·ices (ex.: [un plugin de Nextcloud](https://github.com/nextcloud/user_oidc/pull/884) ne le supportait pas jusqu’à… la semaine dernière !), ce qui ajoute une sérieuse contrainte dans la sélection des applications que nous proposons.

En solution palliative plutôt efficace, nous utilisons le plugin de Caddy [caddy-security](https://github.com/greenpau/caddy-security) qui permet de créer un **mur d’authentification** sur n’importe quelle application au sein du reverse-proxy, en fonction des groupes / rôles attribués à chaque utilisateur·ice.

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#une-derniere-solution-pour-la-route)Une dernière solution pour la route…

Il existe une solution tout à fait valable pour proposer un ensemble de services avec OpenID Connect sans trop de prise de tête : **utiliser l’une des applications comme fournisseur d’authentification**.

Par exemple, si vous proposez un Nextcloud, sachez que Nextcloud lui-même peut agir comme **serveur d’identité OIDC**. Il en va de même pour la plupart des applications connues, comme WordPress par exemple. Ainsi, les membres utiliseront leurs identifiants pour ces applications-là pour se connecter aux autres applications.

Cela implique toutefois que vos membres **aient obligatoirement un compte** sur l’application qui agit comme fournisseur (par exemple, Nextcloud ou WordPress) ; ce n’est peut-être pas toujours souhaitable.

Aussi, le respect des autres critères que vous pourriez avoir (captcha, gestion des groupes, gestion d’organisations, personnalisation des champs…) peut s’avérer assez compliqué.

### [🔗](https://lacontrevoie.fr/blog/2024/comparatif-de-onze-solutions-de-sso-libres/#conclusion)Conclusion

En bref : **on a adopté Keycloak**, même si nous n’en sommes pas très satisfait·es en raison de fonctionnalités manquantes et de cas d’usage assez contraignants.

Mais ne vous donnez pas la peine de nous suggérer d’autres solutions de SSO ! Maintenant que nous sommes sur Keycloak, **nous ne comptons pas changer tout de suite**, comme il s’agit d’une opération très lourde et chronophage et que nous avons déjà passé deux bonnes années à rechercher la solution idéale sans la trouver.

Cependant, nous espérons que cet article vous éclairera sur **l’éventail des possibilités** et vous permettra de faire votre choix plus aisément ! Gardez toutefois en tête que les logiciels que nous vous avons présenté ont, pour certains, sensiblement évolué depuis que nous les avons examinés.

Pour des fournisseurs de services numériques éthiques, faire de l’authentification unifiée représente pour nous un enjeu majeur, pour plusieurs raisons :

**La recherche de facilité.** Pour un public non-initié, habitué au « compte tout-en-un » de Google, Apple ou Microsoft, une authentification unifiée pour des services libres n’est pas dépaysant, elle offre un confort qui facilitera énormément la prise en main, surtout si vous proposez plusieurs logiciels différents.

**Les services ouverts sans inscription** sont de plus en plus difficiles à maintenir. Les services comme [Framaforms](https://framaforms.org/) sont débordés de spam, les pads partagés servent à des fins de phishing, même [notre raccourcisseur de liens](https://lacontrevoie.fr/services/liens/) nous coûte du [temps de modération](https://lacontrevoie.fr/blog/2022/service-liens-trois-ans-de-lutte-contre-le-spam/) chaque semaine. À la fin de l’année 2023, **de nombreuses instances Jitsi** (outil de visioconférence) [ont dû fermer](https://plume.deuxfleurs.fr/~/Deuxfleurs/Abus%20sur%20Jitsi,%20notre%20r%C3%A9action%20%C3%A0%20chaud) à cause d’un sursaut d’usages contraires à la loi et à l’éthique.

Nous espérons que ce long résumé de nos recherches sera utile à certain·es de nos lecteur·ices !

Si vous estimez que ce genre de retour est précieux, vous pouvez **nous soutenir financièrement** pour nous permettre de continuer à [documenter notre travail](https://docs.lacontrevoie.fr/) :

[

### Posez votre étoile

#### et faites un don à La Contre-Voie !

](https://www.helloasso.com/associations/la-contre-voie/formulaires/1)