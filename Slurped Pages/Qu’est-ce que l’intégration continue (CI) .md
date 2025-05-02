---
link: https://enix.io/fr/blog/integration-continue-ci/
byline: Paul Laffitte
site: "@enixsas"
excerpt: Les avantages d'une chaîne d'intégration continue ou pipeline de
  continuous integration (CI), notre retour d'expérience, nos recommandations !
twitter: https://twitter.com/@enixsas
slurped: 2024-11-24T12:29
title: Qu’est-ce que l’intégration continue (CI) ?
---

Dans cet article, nous introduisons le concept **d’intégration continue** (ou _**continuous integration**_).

Nous allons voir **ce que c’est**, la notion de **chaîne d’intégration continue** et **pourquoi l’utiliser** pour le développement de vos applications. Nous terminerons par des **recommandations** issues de notre expérience sur nos projets clients et Open Source.

## L’intégration continue, c’est quoi ?

L'**intégration continue**, souvent abrégée en **CI** pour _Continuous Integration_, est une pratique DevOps qui consiste à régulièrement intégrer les modifications faites dans un dépôt de code. Le principe consiste à **automatiser** les tâches habituellement effectuées à la main, telles que l’exécution des tests, le build de l’application ou encore lancer un scan de vulnérabilités. L’objectif est d’industrialiser ces étapes afin d'**éviter les erreurs humaines** et de pouvoir **vérifier la validité d’une application au fur et à mesure des évolutions du code**.

Le fonctionnement est simple : une fois votre outil d’intégration continue connecté à votre dépôt de code (via un webhook par exemple), il va pouvoir exécuter des tâches sur les différents commits de votre dépôt, qui se déclencheront sous certaines conditions.

Le principe d’intégration continue est largement utilisé par la communauté open-source mais aussi en entreprise. On peut citer par exemple des projets comme [npm](https://github.com/npm/cli/actions) avec _GitHub Actions_ ou encore [Chromium](https://ci.chromium.org/p/chromium) avec _Luci_, un outil développé en open-source pour les besoins du projet.

**L’intégration continue peut être adoptée pour n’importe quel type de projet** de développement. Que ce soit pour des applications déployées de façon classique sur du bare-metal ou des VMs, ou pour des applications cloud native déployées via des containers avec Docker et Kubernetes.

## Pourquoi faire de l’intégration continue ?

L’intégration continue s’inscrit dans la lignée des méthodes agiles et de l’approche DevOps. L’un des premiers avantages de la CI est donc d'**augmenter le confort et l’agilité de vos équipes**, mais c’est loin d’être le seul, en voici une liste non-exhaustive :

### Avantages pour les développeurs

- **Le débogage est facilité** : Étant donné que tous les commits sont testés un à un, il n’y a généralement qu’un seul commit ou au maximum un petit groupe de commits à vérifier lors de l’introduction d’un bug lors du développement.
- **Un nombre réduit de conflits** : Les changements sont intégrés petit à petit, ce qui réduit la surface de conflit et limite leur introduction. Lorsqu’il en apparaît, ils sont généralement facilement résolubles.
- **Meilleure cohérence du code** : Les développeurs sont obligés de régulièrement communiquer entre eux sur leurs avancées via les commits poussés sur le dépôt de code. Les incohérences peuvent donc être identifiées et corrigées au fur et à mesure qu’elles apparaissent.
- **Moins de frictions sur les normes/conventions** : L’utilisation par exemple de _Linter_ ou d’un formateur de code permet d’assurer le respect des normes et conventions, ce qui produit un code plus uniforme et surtout moins de friction pour les développeurs.
- **Un gain de temps** : L’automatisation des tâches répétitives permet de libérer du temps aux développeurs pour se consacrer à des activités à plus forte valeur ajoutée.
- **Un réduction du risque d’erreur humaine** : Certaines tâches étant automatisées, cela réduit drastiquement le risque d’erreurs humaines sur lesdites tâches.

### Avantages pour le chef de projet

- **Des progrès mesurables** : Certains outils permettent de mesurer les progrès effectués d’un commit à l’autre. On peut ainsi observer l’évolution de la couverture des tests, le temps moyen de la correction d’un bug ou encore le temps moyen d’exécution d’un pipeline.
- **Un retour presque immédiat** : On est en mesure de savoir qui a introduit un bug et généralement on s’en rend compte avant de le mettre en production grâce au tests automatisés. La personne ayant introduit le bug est alors en mesure de le corriger rapidement.
- **Une itération plus rapide** : La pression mise sur les petites décisions est minimisée car elles demandent moins d’intervention humaine grâce aux automatisations mises en place.

### Avantages pour les utilisateurs

- **Améliorer le produit et la satisfaction des utilisateurs** : Les mises à jour et corrections de bugs sont plus réactives car les itérations sont plus courtes, ce qui augmente la qualité générale du produit.
- **La boucle de rétroaction avec les utilisateurs est plus courte** : Le fait de pouvoir mettre en production plus rapidement et plus souvent permet d’accélérer la boucle de rétroaction avec les utilisateurs. Des modifications peuvent rapidement être apportées à la production suite à un feedback utilisateur.

In fine, ces différents avantages aident à **augmenter la vélocité de développement** et ainsi **réduire les coûts de développement et de maintenance** tout en **augmentant la satisfaction des utilisateurs**.

Il est cependant important d’avoir à l’esprit que ces avantages dépendent de la bonne mise en place de votre chaîne d’intégration continue.

## La chaîne d’intégration continue

Une **chaîne** (ou _**pipeline**_) **d’intégration continue** correspond à la série d’étapes qui doivent être exécutées afin d’intégrer des changements du code.

Une chaîne d’intégration continue constitue l'**un des éléments d’une chaîne complète de CI/CD** (_**Continuous Intégration**_ & _**Continuous Delivery**_).

Le déploiement continu est une pratique très souvent associée à l’intégration continue puisqu’elle en est la suite logique dans le cycle de vie de l’application. Là où l’intégration continue permet d’intégrer le travail des développeurs au fur et à mesure de sa production, le **déploiement continu** (ou livraison continue) permet de livrer et déployer en production des mises à jour de façon itérative.

![Logo de chaîne d’intégration et de déploiement continu](https://enix.io/fr/blog/integration-continue-ci/chaine-integration-continue.svg)

Lors de la phase d’intégration continue, on trouve des tâches comme la **construction d’un binaire**, l'**exécution de tests** ou l'**analyse statique du code**. Cette phase peut être suivie d’une phase de livraison avec des tâches comme la création d’un tag git pour identifier une version, la publication d’un artefact (images Docker, packages debian, archive, etc), ou encore la mise en production d’une application.

> ⚠️ La mise en place d’une chaîne d’intégration continue ne dispense pas les développeurs de vérifier la validité de leur code en local ! Ils restent responsables du bon fonctionnement du code sur leur périmètre : la CI est là pour assurer un niveau de qualité supplémentaire en automatisant les tests plus globaux sur l’application.

## Choisir parmi les outils d’intégration continue

Il existe de nombreux outils d’intégration continue. Certains sont “généralistes” et permettent de gérer la (quasi) intégralité de la chaîne d’intégration et de déploiement continu, d’autres outils couvrent un périmètre plus restreint ou sont plus spécialisés.

**Choisir son outil d’intégration continue** peut être une tâche fastidieuse étant donné le nombre conséquent d’alternatives disponibles et la variété des fonctionnalités recherchées. **Quelques critères fonctionnels** qui nous semblent fondamentaux pour bien choisir son outil :

- Simplicité d’utilisation par les développeurs
- Intégration avec les autres outils de livraison et de déploiement continu
- Extensibilité (add-ons, plugins) de la chaîne CI/CD pour les évolutions fonctionnelles
- Disponibilité d’outils d’analytics sur l’utilisation de la CI et des habitudes de code
- Adéquation avec le développement d’applications conteneurisées et un déploiement sur Kubernetes
- Intégration avec les plateformes sur lesquelles on souhaite déployer (e.g. Cloud providers, …)
- …

Voici quelques-uns des **outils de CI les plus connus** : GitLab CI, GitHub Actions, Jenkins / Jenkins X, Travis CI, CircleCI, Bamboo, Concourse CI, etc. Nous ferons sous peu un blogpost plus détaillé sur les principaux outils DevOps plus globalement.

Certains de ces outils peuvent être _**self-hosted**_ sur l’environnement de votre choix, ou il existe pour la plupart des déploiements en **mode SaaS**. Chez Enix, nous concevons des **chaînes de CI/CD sur mesure**. La plupart du temps, nous les **hébergeons** et les **opérons** sur nos infrastructures pour en assurer la disponibilité, la maintenance et les évolutions régulières (et oui, une chaîne de CI/CD n’est pas figée une fois qu’elle est mise en place !).

![Logos d’outils d’intégration continue](https://enix.io/fr/blog/integration-continue-ci/outils-integration-continue.webp)

## Retour d’expérience et recommandations

Au travers de nos projets open-source tels que [x509-exporter](https://enix.io/fr/blog/renouveler-certificats-kubernetes-cloud-native/) et [san-iscsi-csi](https://github.com/enix/san-iscsi-csi), ou bien sur les projets de nos clients, nous avons pu acquérir un bagage de bonnes pratiques liées à l’intégration continue.

En règle générale, nous mettons en place un système d’intégration et de déploiement continue (CI/CD) sur notre instance GitLab self-hosted pour les projets internes et sur github.com ou gitlab.com pour nos projets open-source.

Pour nos projets clients, nous mettons en place des chaînes CI/CD sur mesure en utilisant leurs outils préférés (Gitlab, Github, Jenkins, etc.).

### Principales recommandations

Aujourd’hui, nous souhaitons partager avec vous cette expérience, notamment ces principales recommandations (forcément non exhaustives !) :

- **Faites des commits fréquents** : Tout l’intérêt de la CI est d’avoir, comme son nom l’indique, une intégration **continue**. Faire des commits fréquents permet de détecter les anomalies au plus vite et de bénéficier des intérêts des automatisations mises en place. Régulièrement merger le travail sur la branche principale permet ensuite de réduire le nombre de conflits.
- **Testez tous les commits** : Lancer les tests sur toutes les branches permet de détecter les bugs au plus tôt. Ne tester que la branche principale permet de s’assurer du bon fonctionnement du produit fini mais ne permet pas autant de réactivité et peut engendrer des coûts en cas de conflits difficilement résolubles.
- **Échouez rapidement** : Les tests d’intégration sont souvent longs à exécuter, privilégiez donc l’exécution des tests unitaires en premier afin d’éviter d’avoir à lancer des tests d’intégration lorsque les tests unitaires ne passent pas. La même chose est vraie pour les tests E2E (end-to-end). De façon générale, faites bien attention à l’ordre de vos tâches.
- **Optimisez le temps d’exécution de votre pipeline** : Des temps d’exécution courts permettent de connaître plus rapidement la validité d’un commit et de réduire son temps de mise en production ainsi que les coûts d’exécution. Vous pouvez par exemple vous assurer d’installer les dépendances de votre pipeline avant l’exécution de celui-ci plutôt que pendant. On peut pour cela utiliser des images Docker par exemple. Mettre des conditions de déclenchement permet aussi de réduire le temps d’exécution, certaines parties de l’application n’ont pas besoin par exemple d’être reconstruites si le code n’a pas changé entre temps, c’est particulièrement vrai dans le cadre des monorepo. On peut aussi utiliser des outils comme le [DAG](https://docs.gitlab.com/ee/ci/directed_acyclic_graph/) (Direct Acyclic Graph) de Gitlab.
- **Utilisez des variables d’environnement** : La notion de sécurité peut devenir très importante lors de la mise en place d’une CI, on parle d’ailleurs de DevSecOps lorsque les questions de sécurité sont abordées dès la mise en place des chaînes CI/CD. Utiliser des variables d’environnement permet d’éviter d’avoir à mettre les secrets et credentials utilisés directement dans la configuration de la CI, qui généralement est versionné dans le dépôt de code, et ainsi éviter que n’importe qui puisse y accéder.
- **Définissez correctement la gouvernance de la CI** : La CI est une brique sensible et quelques heures d’indisponibilités seulement peuvent mettre à mal toute une entreprise. Il est donc important que les rôles soient bien définis. Généralement, c’est une équipe avec des compétences transversales qui va gérer celle-ci. Il ne faut surtout pas hésiter à mettre les moyens humains nécessaires à son bon fonctionnement car comme pour le code, la CI demande une certaine maintenance suivant sa mise en place.
- **Mettez en place des notifications** : Pour alerter au plus vite d’un problème survenu sur la CI, il peut être une bonne idée de mettre en place un envoi de mail ou encore mieux, de message slack directement dans un canal dédié, voire en message privé au développeur concerné.
- **Gardez des étapes de CI très simples** : Dans la mesure du possible, les différentes étapes de la CI doivent être lançables manuellement afin de faciliter le debugging. Pour cela, simplifiez au maximum la CI en déplaçant la complexité vers des scripts qui pourront être appelés par la CI et lancés localement.

### Adapter votre chaîne d’intégration continue

Après avoir vu ces recommandations, on pourrait croire qu’il suffit de les mettre en pratique pour bénéficier automatiquement de tous les avantages d’une chaîne d’intégration. Mais la CI n’est pas un outil magique !

Quand on conçoit sa chaîne d’intégration continue, il est **important de l’adapter à son métier, son type d’applications, ses environnements de développement et de test, ainsi qu’à son organisation interne**. Il y a de nombreux choix à faire sur l’outil et sur les automatisations à mettre en place qui peuvent dépendre notamment des besoins et des envies exprimées par les différentes équipes.

Il existe également des **pratiques d’intégration continue avancées**. On peut par exemple mettre en place du _semantic versioning_ pour automatiser la création et la sortie de versions, ou encore la génération du _changelog_. Je vous invite d’ailleurs à lire cet article d’Alexandre sur la [gestion automatique des versions de code](https://enix.io/fr/blog/integration-continue-sortir-de-la-gestion-manuelle-des-versions/).

## Conclusions

Nous avons vu dans cet article les nombreux **avantages de la mise en place d’une chaîne d’intégration continue** pour améliorer la vélocité du développement et la qualité du code. Bien mise en place, l’intégration continue est également une vraie source de confort pour les équipes techniques débarrassées de tâches manuelles souvent fastidieuses.

Attention cependant à ne pas abuser des bonnes choses, certaines tâches peuvent nécessiter une attention particulière et donc de conserver une intervention humaine. Sachez donc faire la part des choses et **ne complexifiez pas votre chaîne de CI en automatisant des tâches qui ne devraient pas l’être**.

Enfin, si vous avez besoin de support pour la mise en place ou la gestion opérationnelle de votre chaîne d’intégration continue, n’hésitez pas à nous contacter, nos [experts DevOps](https://enix.io/fr/expert-devops/) se feront un plaisir de vous aider !

---

Ne ratez pas nos prochains articles DevOps et Cloud Native! Suivez Enix sur [Twitter](https://twitter.com/enixsas)!