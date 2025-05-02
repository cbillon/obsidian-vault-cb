---
link: https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739
byline: Yann Barrault
site: DEV Community
date: 2024-12-14T08:00
excerpt: Cet article s'adresse aux professionnels de la data, aux développeurs
  et aux responsables de projets...
twitter: https://twitter.com/@
tags:
  - slurp/adventoftech2024
  - slurp/dataengineering
  - slurp/airbyte
  - slurp/dbt
  - slurp/software
  - slurp/coding
  - slurp/development
  - slurp/engineering
  - slurp/inclusive
  - slurp/community
slurped: 2025-03-01T08:56
title: "Explorer l'API de 360Learning : de l'agilité de Power Query à la
  robustesse de la Modern Data Stack"
---

Cet article s'adresse aux professionnels de la data, aux développeurs et aux responsables de projets numériques qui cherchent à optimiser l'exploitation des données via des solutions modernes et agiles. En explorant l'**API de 360Learning**, nous partageons notre expérience de la transition de **Power Query** à une **Modern Data Stack** robuste, intégrant **Airbyte** et **DBT** , pour répondre aux exigences de **reporting** de **France Num** .

**LinkedIn** : [France Num](https://www.linkedin.com/company/francenum) [360 Learning](https://www.linkedin.com/company/360learning/) [Cerfrance Brocéliande](https://www.linkedin.com/company/cerfrance-broc%C3%A9liande) [Cerfrance Côtes d'Armor](https://www.linkedin.com/company/cerfrance-cotes-darmor/)

#### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#auteurs)Auteurs

[Florian SEURRE](https://www.linkedin.com/in/florian-seurre) - [Gildas GUILLEMOT](https://www.linkedin.com/in/gildas-guillemot-a522752/) - [Serge-François PATOUT](https://www.linkedin.com/in/serge-fran%C3%A7ois-patout/) - [Yann BARRAULT](https://www.linkedin.com/in/yann-barrault-onepoint/)

## [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#table-des-mati%C3%A8res)Table des matières

1. 📊[Quelques éléments de contexte !](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#contexte)
2. [Nos objectifs et besoins](https://dev.toobjectifs/)
3. 🔍[Retour d’expérience : Utilisation de Power Query pour exploiter l'API de 360Learning](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#retour-dexp%C3%A9rience--utilisation-de-power-query-pour-exploiter-lapi-de-360learning-)
    - 🌟[Les opportunités offertes par Power Query](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#les-opportunit%C3%A9s-offertes-par-power-query-)
    - 🔧[Retour d’expérience avec Power Query](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#retour-dexp%C3%A9rience-avec-power-query-)
    - 🚧[Les limites rencontrées avec Power Query](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#les-limites-rencontr%C3%A9es-avec-power-query-)
4. 💡[De Power Query à Airbyte pour exploiter l’API de 360Learning](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#de-power-query-%C3%A0-airbyte-pour-exploiter-lapi-de-360learning-)
    - [Pourquoi Airbyte ?](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#pourquoi-airbyte--)
    - [Comparaison avec d’autres solutions : Fivetran et Rivery](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#comparaison-avec-dautres-solutions--fivetran-et-rivery-)
    - [Les limites rencontrées avec Airbyte](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#les-limites-rencontr%C3%A9es-avec-airbyte)
5. 🚀[Les perspectives de DBT pour faciliter l’exploitation et la visualisation des données](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#les-perspectives-de-dbt-pour-faciliter-lexploitation-et-la-visualisation-des-donn%C3%A9es-)
    - [DBT : Transformer les données brutes en modèle analytique](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#dbt--transformer-les-donn%C3%A9es-brutes-en-mod%C3%A8le-analytique)
    - [Connexion avec Tableau ou Power BI](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#connexion-avec-tableau-ou-power-bi)
6. 🏁[Conclusion et perspectives](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#conclusion-et-perspectives-)
    - [Perspectives à explorer](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#perspectives-%C3%A0-explorer-)
7. 🔗[Références](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#r%C3%A9f%C3%A9rences-)

## [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#quelques-%C3%A9l%C3%A9ments-de-contexte-)Quelques éléments de contexte !

Chez **Nexcer**, nous avons créé une synergie unique entre Onepoint et les cabinets comptables [**Cerfrance Brocéliande**](https://www.cerfrance-broceliande.fr/) et [**Cerfrance Côtes d’Armor**](https://cerfrance22.fr/) pour développer des solutions SaaS sur mesure à destination de la profession.

**[Kinexo](https://www.kinexo.fr/)** , une PSA (Plateform Services Automation) est au cœur de la digitalisation des cabinets comptables et de leurs clients. Elle optimise la gestion des processus de vente et de production grâce à une GRC (Gestion de la Relation Client), simplifiant ainsi la gestion des devis, des lettres de mission avec signature électronique, le suivi de la production et la facturation automatisée.

**MyKinexo**, de son côté, offre aux clients un accès direct et sécurisé à leurs documents importants, tels que les factures et lettres de mission, stockés dans un coffre-fort numérique. Cela leur permet de suivre l'avancement des prestations et de structurer leur patrimoine documentaire, en conformité avec la réglementation (**Facture-X**).

Ensemble, Kinexo et MyKinexo facilitent une **transformation numérique** fluide et efficace pour les cabinets comptables et leurs clients. **[Onepoint](https://www.groupeonepoint.com/fr/)** , cabinet de conseil et architecte des grandes transformations, en sa qualité de partie prenante dans la joint-venture Nexcer, joue un rôle technologique majeur dans la transformation numérique des cabinets comptables.

Pour soutenir cette transformation, Nexcer utilise la solution LMS **[360Learning](https://360learning.com/)** pour mutualiser les plateformes e-Learning des cabinets Cerfrance. Initialement, cette plateforme a formé **1500 collaborateurs** sur la solution Kinexo en 2019. Elle a ensuite évolué pour devenir un "Campus de formations e-Learning" pour les formations internes.

Plus récemment, elle a été intégrée au projet France Num, visant à sensibiliser **3000 entreprises bretonnes** aux enjeux numériques, à les accompagner dans leur numérisation, et à inciter celles éloignées du numérique à entamer leur transformation. Ces TPE/PME couvrent divers secteurs, de l'artisanat au commerce, en passant par les services, l'agriculture et les métiers de la mer.

Le dispositif France Num est riche en diversité, impliquant des acteurs clés tels que les **assistants digitaux** pour la gestion des inscriptions et du phoning, les **relais digitaux** pour les diagnostics, et les **coachs digitaux** pour l'animation des formations. **Formateurs**, **auteurs** de contenu, **administrateurs** de la plateforme et un **comité de pilotage** collaborent pour garantir l'efficacité du dispositif. **_La décentralisation_** est cruciale pour assurer la proximité avec les clients, avec des sessions de formation dans les **50 agences du groupement** .

En tant qu'administrateur de la plateforme 360Learning, je peux attester de ses nombreux atouts : la gestion de groupes de formations publiques/privés distincts pour offrir un contenu spécifique aux apprenants, la possibilité d'ajouter des champs personnalisés sans contraintes, et une API pour accéder aux données. Dans le cadre du projet France Num, nous avons exploité ces fonctionnalités pour isoler les clients des collaborateurs internes des cabinets.

Cependant, les capacités de reporting de la plateforme ont montré leurs limites face aux exigences strictes de France Num.

## [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#nos-objectifs-et-besoins)Nos objectifs et besoins

Bien que la plateforme 360Learning offre de nombreux avantages, ses capacités de reporting se sont révélées insuffisantes pour répondre aux exigences strictes de France Num. Pour piloter efficacement le projet, nous devons suivre plusieurs indicateurs clés de performance (KPI) :

- **Nombre de clients** ayant suivi le parcours de formation, par cabinet (historique)
- **Nombre d’inscrits** par cabinet et par région ou agence (prévisions)
- **Taux de remplissage des sessions** de formation
- **Répartition des apprenants** selon divers critères (géographique, typologie, secteur d’activité, taille, nombre de salariés, etc.)

En outre, France Num impose un **reporting très détaillé** pour procéder aux règlements. Ce reporting doit inclure des informations sur l’apprenant, son entreprise, son dirigeant, le temps passé en formation par module, les feuilles de présence, ainsi que ses appréciations et recommandations.

Dans une démarche agile, nous avons choisi d'exploiter les données de l’API de 360Learning pour générer des statistiques et des analyses sur nos dispositifs de formation.

[Microsoft Power Query](https://learn.microsoft.com/fr-fr/power-query/power-query-what-is-power-query) s’est rapidement imposé comme un choix naturel pour la centralisation des données de l’API grâce à sa simplicité et son intégration dans des outils comme **Excel** ou **Power BI** . Cela en fait une solution accessible, même pour des utilisateurs non-développeurs.

Cette approche nous a permis de tester l’API, valider les statistiques et soumettre le reporting aux équipes de pilotage pour itérer sans nécessiter de développement spécifique ni de monter une infrastructure.

De plus, le format de reporting exigé par France Num est un fichier Excel, ce qui s'intègre parfaitement avec notre utilisation de Power Query.

## [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#retour-dexp%C3%A9rience-utilisation-de-power-query-pour-exploiter-lapi-de-360learning)Retour d’expérience : Utilisation de Power Query pour exploiter l'API de 360Learning

[![Image description](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fst570y90x4k15e6m3cq4.jpg)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fst570y90x4k15e6m3cq4.jpg)

### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#les-opportunit%C3%A9s-offertes-par-power-query)Les opportunités offertes par Power Query

- **Découverte rapide des données** : Power Query permet d'interroger une API sans nécessiter de développement complexe. Une simple configuration suffit pour effectuer des appels REST et transformer les résultats JSON en tables exploitables.
    
- **Flexibilité pour les ajustements** : Grâce à son éditeur visuel, il est possible d'ajouter ou de modifier des colonnes, de regrouper des données ou d'appliquer des filtres sans écrire de code.
    
- **Rafraîchissement à la demande** : Une fois le flux configuré, les données peuvent être mises à jour régulièrement, offrant une solution dynamique pour suivre l'évolution des indicateurs clés.
    

### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#retour-dexp%C3%A9rience-avec-power-query)Retour d’expérience avec Power Query

- **Simplicité d'utilisation** : Power Query facilite la consommation de données à partir de plusieurs sources, mais l'accès à certaines bases de données nécessite l'installation de drivers sur le poste, ce qui n'est pas toujours simple ni possible pour les utilisateurs métier.
    
- **Complexité des requêtes** : Lorsque des requêtes ou manipulations avancées sont nécessaires, cela peut devenir complexe et nécessiter l'utilisation de l'éditeur avancé.
    
- **Assistance IA** : Que ce soit via l'interface utilisateur ou l'éditeur de requêtes avancé, une IA comme ChatGPT ou notre solution sécurisée interne Neo chez Onepoint se révèle être un allié précieux.
    
- **Limites de l'éditeur avancé** : L'éditeur avancé présente des lacunes, telles que l'absence de complétion automatique, de recherche et de remplacement, et la nécessité de doubler systématiquement les guillemets.
    
- **Augmentation de la complexité** : Bien que la complexité des requêtes ait augmenté, la solution reste sous contrôle. Cependant, le temps de traitement pour rafraîchir les données a augmenté de manière linéaire avec le nombre de formations, révélant les limitations des appels HTTP avec Power Query (gestion des erreurs, catch/retry, throttling).
    
- **Nécessité d'une solution plus robuste** : En raison de la fiabilité décroissante des statistiques et des temps de traitement trop longs, une solution plus robuste et fiable s'est avérée nécessaire.
    

### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#les-limites-rencontr%C3%A9es-avec-power-query)Les limites rencontrées avec Power Query

Bien que Power Query ait été efficace initialement, certaines contraintes sont apparues à mesure que les besoins devenaient plus complexes :

- **Gestion de la volumétrie** : Power Query peut rencontrer des difficultés avec des jeux de données volumineux, notamment lorsque les appels API retournent des résultats paginés. Avec 214 sessions, 675 dates de formation en présentiel, 2980 apprenants et 118 905 champs personnalisés, le temps de rafraîchissement est devenu excessivement long.
    
- **Limitations des appels API** : Les quotas d'appels ou les erreurs intermittentes (timeouts, authentifications expirées) ne sont pas toujours bien gérés, rendant le processus instable.
    
- **Complexité croissante** : Au fil du temps, les transformations (jointures, pivots...) deviennent plus complexes, et l'interface visuelle atteint ses limites.
    
- **Absence de connecteurs natifs** : Contrairement à certaines solutions avancées, Power Query ne propose pas de connecteurs prêts à l'emploi pour simplifier l'intégration avec certaines API spécifiques.
    
- **Installation locale de drivers** : Bien que cela n'ait pas été une contrainte dans notre cas, il est important de noter que Power Query nécessite des drivers locaux pour accéder à certaines bases de données.
    

## [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#de-power-query-%C3%A0-airbyte-pour-exploiter-lapi-de-360learning)De Power Query à Airbyte pour exploiter l’API de 360Learning

Pour surmonter les limites rencontrées avec Power Query, nous avons exploré plusieurs solutions capables d'automatiser et de fiabiliser l’ingestion des données depuis l’API de 360Learning. Airbyte s'est imposé comme la meilleure alternative grâce à sa facilité de prise en main, de mise en place et son architecture open source.

### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#pourquoi-airbyte-)Pourquoi Airbyte ?

Airbyte est une solution moderne d’ingestion, permettant de synchroniser des données depuis diverses sources (API, bases de données, applications SaaS) vers des destinations telles qu’un entrepôt de données.

- **Gestion des synchronisations incrémentales** : Contrairement à Power Query, Airbyte permet de synchroniser uniquement les nouvelles données, réduisant ainsi la charge sur l’API et la base de données cible. Et bien sûr le délais de rafraîchissement.
    
- **Résilience face aux erreurs** : En cas d’échec d’une synchronisation, Airbyte peut redémarrer à partir du point d’échec, assurant ainsi une continuité sans faille.
    
- **Support des quotas API et pagination** : Airbyte gère automatiquement la pagination des résultats API, ce qui était l'une des principales limites de Power Query.
    
- **Personnalisation des connecteurs** : Pour l’API de 360Learning, nous avons pu créer un connecteur sur mesure afin de répondre à des besoins spécifiques, notamment pour les données des formations en présentiel (nombre d’inscrits, feuilles de présence).
    

### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#comparaison-avec-dautres-solutions-fivetran-et-rivery)Comparaison avec d’autres solutions : Fivetran et Rivery

Nous avons également considéré des alternatives comme Fivetran et Rivery, mais elles présentaient des limitations spécifiques :

- **Fivetran** propose un connecteur 360Learning, mais celui-ci n’inclut pas certaines données essentielles, comme les formations en présentiel.
- **Rivery**, bien que très flexible, ne permet pas la création de connecteurs sur mesure.

Ces solutions restent intéressantes lorsque les connecteurs natifs couvrent tous les besoins. Cependant, dans notre cas, la personnalisation offerte par Airbyte a fait la différence, nous permettant d'adapter précisément l'intégration à nos exigences spécifiques.

### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#les-limites-rencontr%C3%A9es-avec-airbyte)Les limites rencontrées avec Airbyte

- **Complexité technique** : Airbyte nécessite des compétences techniques pour l'installation et la configuration, notamment la maîtrise de Docker.
    
- **Prérequis matériels sur son poste de travail** : Airbyte peut nécessiter des ressources matérielles significatives, un poste de 16Go de RAM est clairement insuffisant.
    
- **Documentation limitée** : La documentation est à notre avis insuffisante, pas suffisamment détaillée et manque d’exemples, rendant l'apprentissage et la résolution de problèmes plus difficiles.
    
- **Personnalisation limitée** : Bien que Airbyte soit open-source, la personnalisation des connecteurs ou des fonctionnalités nécessite des compétences en développement tout comme l’éditeur avancé de Power Query.
    
- **Déploiement** : Si vous n’utilisez pas la version payante en mode SaaS, il vous faudra gérer le provisionning et le déploiement sur un de vos clusters Kubernetes. Heureusement, Airbyte est nativement CaC (configuration as code) ce qui ravira les Devops.
    

## [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#les-perspectives-de-dbt-pour-faciliter-lexploitation-et-la-visualisation-des-donn%C3%A9es)Les perspectives de DBT pour faciliter l’exploitation et la visualisation des données

Avec Airbyte, les données sont synchronisées dans une base **PostgreSQL**, mais elles restent brutes et souvent difficiles à exploiter directement. Pour transformer ces données en un modèle analytique prêt à être utilisé par des solutions de reporting BI comme **Tableau** ou **Power BI**, nous avons opté pour **DBT (Data Build Tool)**.

[![Image description](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwif9p13hzy5c0uktl467.jpg)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwif9p13hzy5c0uktl467.jpg)

### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#dbt-transformer-les-donn%C3%A9es-brutes-en-mod%C3%A8le-analytique)DBT : Transformer les données brutes en modèle analytique

DBT permet de créer des modèles de transformations en SQL directement dans la base de données cible :

- **Simplification des modèles** : Les données initialement fragmentées ou redondantes sont regroupées dans des tables synthétiques, prêtes pour l’analyse.
- **Enrichissement des données** : DBT facilite l’ajout de colonnes calculées, comme des taux de complétion ou des moyennes d’évaluation.
- **Versioning et traçabilité** : Toutes les transformations sont versionnées et documentées, offrant une grande transparence.
- **Qualité de la donnée** : La possibilité de faire des tests unitaires sur les modèles de données permet de garantir une qualité de la donnée d’une version à une autre.

Exemple concret : **Analyser les formations en présentiel**

Avec DBT, nous avons pu regrouper les données des sessions en présentiel, des participants et des feuilles de présence pour créer une table unique, contenant :

- Le nombre d’inscrits et le taux de participation.
- Les évaluations moyennes par session.
- Les indicateurs de satisfaction consolidés par groupe ou région.

### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#connexion-avec-tableau-ou-power-bi)Connexion avec Tableau ou Power BI

Une fois les données transformées, elles sont prêtes à être connectées à des outils BI pour une visualisation avancée. Avec Tableau ou Power BI, nous avons pu :

- Créer des tableaux de bord dynamiques pour suivre les KPI.
- Offrir des capacités d’exploration en temps réel.
- Automatiser le rafraîchissement des données pour garantir leur actualité.

## [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#conclusion-et-perspectives)Conclusion et perspectives

Le passage de Power Query à Airbyte, enrichi par DBT, a profondément transformé notre approche de l’analyse des données issues de l’API de 360Learning. Cette chaîne de solutions constitue une plateforme robuste, automatisée et évolutive, nécessitant toutefois une infrastructure pour l’exécution des traitements et le stockage des données. Heureusement, Nexcer dispose d'une infrastructure en mode SaaS qui permet de déployer rapidement ce type de Modern Data Stack grâce à l'Infrastructure as Code.

### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#perspectives-%C3%A0-explorer)Perspectives à explorer

- **Automatisation complète avec un orchestrateur de workflow** : Envisager l'orchestration de l’ensemble du pipeline, de l’extraction API à la mise à jour des tableaux de bord, via un outil comme Airflow. Cela permettrait de gérer la fréquence des traitements pour mettre à jour et transformer les données de manière fluide et de pouvoir monitorer efficacement le bon déroulement du pipeline.
    
- **Standardisation des KPI avec DBT** : Développer des modèles de données optimisés pour garantir une cohérence analytique et une qualité de données irréprochable, facilitant ainsi l'exploitation des KPI.
    
- **Déploiement automatisé et Configuration as Code** : Gérer et versionner le déploiement de l'infrastructure sous-jacente et le paramétrage des flux Airbyte sous forme de code, assurant ainsi une flexibilité et une reproductibilité accrues.
    

En intégrant Airbyte, Airflow et DBT, nous avons la possibilité de construire un Modern Data Stack à la fois puissant et adaptable, facilitant une exploitation des données plus efficace et une prise de décision plus éclairée.

Cette approche, moins coûteuse, plus modulaire et flexible que les solutions traditionnelles, marque une avancée significative dans la manière dont les entreprises peuvent gèrer et valoriser leurs données.

Cet article fait partie du "Advent of Tech 2024 Onepoint", une série d'articles tech publiés par Onepoint pour patienter jusqu'à Noël.  
[Voir tous les articles](https://dev.to/onepoint/advent-of-tech-2024-onepoint-le) du Advent of Tech 2024 d'ici Noël.

Merci à @GaelPoupard pour cette superbe idée, à [@bmarsteau](https://dev.to/bmarsteau) et [@michaelmaillot](https://dev.to/michaelmaillot) pour l’organisation de cet [Advent of Tech](https://dev.to/onepoint/advent-of-tech-2024-onepoint-le), ainsi qu’à @onepoint et à nos relecteurs. Un spécial big up 🙌 à [**Florian**](https://www.linkedin.com/in/florian-seurre) qui nous a fait découvrir à cette occasion Airbyte et DBT cf Data Landscape & Data Tech Radar ci-dessous !

Bonnes lectures et joyeuses fêtes de fin d'année à vous tous !

### [](https://dev.to/onepoint/explorer-lapi-de-360learning-de-lagilite-de-power-query-a-la-robustesse-de-la-modern-data-stack-5739#r%C3%A9f%C3%A9rences)Références

[The 2024 MAD (ML, AI & Data) Landscape](https://mad.firstmark.com/)

[Airbyte - Open-Source Data Movement](https://airbyte.com/)

[dbt Labs – Transform Data](https://www.getdbt.com/)

[Airbyte - Define your infrastructure through configuration files with Terraform](https://airbyte.com/blog/terraform-provider-launched-for-airbyte-cloud)

[Apache Airflow® - An Open-Source platform for developing, scheduling, and monitoring batch-oriented workflows as Code](https://airflow.apache.org/docs/apache-airflow/stable/index.html)

[Airbyte - How-to-use-airflow-and-airbyte-together](https://airbyte.com/tutorials/how-to-use-airflow-and-airbyte-together)

[Rivery – An all-in one Modern Data Stack (+200 connecteurs)](https://rivery.io/fr/)

[Fivetran – Automated (ELT) Data Movement Platform (+660 connecteurs)](https://www.fivetran.com/)

[Tech Radar](https://data-ai.theodo.com/tech-radar-data1)