---
link: https://enix.io/fr/blog/gitops-definition/
byline: Alexandre Buisine
site: "@enixsas"
excerpt: Notre définition et l'essentiel sur GitOps, pour une gestion centralisée et automatisée de vos infrastructures Kubernetes dans git.
twitter: https://twitter.com/@enixsas
slurped: 2024-11-24T12:21
title: "GitOps : les principes et pourquoi l'adopter ?"
tags:
  - git
  - gitops
---

Cet article s’adresse à ceux qui découvrent ou qui n’ont pas encore d’avis fort sur **la pratique GitOps**.

Il est particulièrement difficile d’y voir clair entre le phénomène [**DevOps**](https://enix.io/fr/expert-devops/), les outils d’**Infrastructure-as-code**, et la **méthode GitOps**.

L’objectif de cet article est de défricher tout ce joyeux petit monde. Vous devriez sortir de cette lecture avec **un avis sur le sujet**, et pourquoi pas même une préférence sur l’approche GitOps qui vous correspond le plus.

Je vais rester très court sur le côté DevOps : on essaie d’unifier le Dev et l’Ops au travers de l’automatisation, du monitoring et autres bonnes pratiques, dans le but d’optimiser et d’accélérer toute la chaîne. L’un des gains notables est de pouvoir livrer son application plus souvent et avec moins d’embûches, mais il y en a de nombreux autres… (plus de détails dans un futur billet de blog !).

Le DevOps est donc une pratique (ou un métier), et il faut, comme partout, s’outiller et appliquer les bonnes méthodes !

Commençons avec les concepts bien en vogue…

## Les bases de l’Infrastructure-as-code

L’infrastructure-as-code est une pratique qui cherche à faciliter la gestion des infrastructures IT :

La **mise en place d’une infrastructure** est une tâche longue et fastidieuse, avec pas mal de dépendances entre les objets que l’on manipule (machines, réseau, stockage, sécurité…). Il en résulte qu’une **infrastructure est difficilement duplicable ou ré-instanciable** (dans le cas d’une grosse panne par exemple).

C’est d’autant plus regrettable quand on cherche à mettre en place des environnements identiques :

- Une production avec sa pré-production la plus ressemblante possible
- Une production dupliquée sur 3 _datacenters_
- …

Le principe est donc de coucher ~~sur papier~~ **dans un fichier** les informations relatives à l’environnement que l’on souhaite produire ou reproduire. L’envie est d’autant plus grande depuis que l’on peut utiliser les APIs des cloud providers pour décrire une infrastructure (ou utiliser des solutions on-premises avec des APIs). On peut enfin **décrire une machine**, **du réseau**, **du stockage** … de façon normée.

A partir de là, les outils permettant d’**atteindre un état décrit dans un fichier** n’ont pas mis longtemps à apparaître (on parle de modèle déclaratif).

Si une partie de l’infrastructure est déjà en place, charge à l’outil de rajouter / supprimer les ressources manquantes ou inadaptées.

Vous connaissez probablement déjà certains d’entre eux, au moins de nom :

- Les génériques multi-cloud : [Terraform](https://www.terraform.io/), [Pulumi](https://www.pulumi.com/), …
- Cloud formation pour Amazon
- Azure Resource Management, ou même [Bicep](https://docs.microsoft.com/fr-fr/azure/azure-resource-manager/bicep/overview)
- Google Cloud Deployment manager
- …

On parle de **déclarer des infrastructures dans des fichiers**… Il est donc temps de stocker tout ça dans Git ! Appliquons donc la **méthode GitOps**…

## Définition du GitOps

![Logo Gitops](https://enix.io/fr/blog/gitops-definition/gitops-logo.png)

Je vous la fais en version courte !

Sans même parler d’infrastructure, vous utilisez certainement divers outils pour décrire un état recherché en production :

- Si vous utilisez **[Kubernetes](https://enix.io/fr/kubernetes-devops-expert/)**, un **Helm** Chart peut représenter votre stack applicative de production sous forme de fichier ;
- Si vous configurez vos machines avec **Ansible**, vous décrivez ce qui va tourner à l’intérieur d’une infrastructure via plusieurs fichiers ;
- Il existe de nombreux autres cas d’usage …

On y est ! Le **GitOps** est là pour organiser tout ça.

On peut résumer le concept en une phrase : **Organiser les opérations liées à la modification d’un environnement (généralement la production, mais pas que) sur la base d’un (ou plusieurs) repository git**.

En langage technique, cela revient à combiner une **_source of truth_** avec de l’**automatisation**.

### Source of truth

Lorsque plusieurs personnes travaillent sur un objectif de production, et qu’en plus il existe plusieurs visions ou versions de l’objectif … il devient fondamental de savoir où est la référence actuelle. Vous l’aurez deviné, avec la méthode GitOps, on **stocke tous les fichiers déclaratifs de notre état recherché dans Git**.

Les **bénéfices** sont immédiats :

- Tant d’un point de vue traçabilité
    - Historique des modifications
    - Qui a changé quoi ?
    - Gestion des droits de lecture et d’édition
- … que d’un point de vue méthodologie
    - Plusieurs branches/visions concurrentes
    - Possibilité d’éditer en parallèle
    - Possibilité de merger/concilier plusieurs visions concurrentes

Bref, vous pouvez **ré-utiliser toutes les bonnes méthodes utilisées lors du développement applicatif** et les appliquer à la gestion de votre infrastructure.

### Automatisation

Sur la base d’un **repository Git**, et à l’image de ce qui se fait habituellement pour du code applicatif ; on cherche à construire un **pipeline d’intégration continue**. Quel que soit l’outil, par exemple Gitlab CI, Github Actions, Jenkins / Jenkins X, ou autre …

En d’autres termes, on cherche à vérifier la validité et la qualité du code déclaratif, et on peut en profiter pour en tester certaines parties.

A ce stade, on se sent bien prêt pour déployer en production ! Alors c’est parti !

Que vous souhaitiez augmenter la fréquence des déploiements ou éviter les actions manuelles hasardeuses, **la méthode GitOps vous invite à automatiser l’exécution du script**, avec un déclencheur qui peut rester humain.

Attention toutefois, deux stratégies s’opposent pour le déploiement continu On parle d’approches Push ou Pull. Voyons ça de plus près !

## GitOps Push : artisanat ou solution entièrement customisée

Si vous considérez que des fichiers déclaratifs avec un outil et un pipeline de CI/CD sont déjà bien assez de composants. Ou si vous avez le concept de _source of truth_ déjà en place. L’**approche _push_ du GitOps** aura votre préférence.

Le concept ici est simple : on utilise son pipeline CI/CD pour le _last mile,_ à savoir pour déployer en production.

En termes techniques, il s’agît de lancer, au même titre que l’intégration continue, l’outil choisi sur la base des fichiers stockés dans git et dans le but de modifier l’environnement de destination.

La plupart du temps vous asservirez le pipeline à une action manuelle de déclenchement histoire de rester le patron.

Les **points positifs** sont nombreux :

- Maintenance simplifiée puisque similaire à l’intégration continue
- Pas besoin d’outil supplémentaire
- Conserver la maîtrise/compréhension de chaque action
- Customisation sans limite, sur la base d’évènements git ou manuels
- Assez facile à re-jouer en local sur son poste pour débugger

Il faut parler des **inconvénients** aussi :

- On couple un peu plus le Dev et l’Ops
- L’outil de CI/CD devra avoir accès d’une façon ou d’une autre à la production (ce qui peut s’avérer inacceptable pour certains)
- Vous devez avoir un niveau de compétence certain à propos des outils et des concepts de déploiement afin de mettre en place le système

## GitOps Pull : Flux ou ArgoCD

![Logos de Flux et de Argo](https://enix.io/fr/blog/gitops-definition/gitops-flux-argo.png)

Terminé les extensions de pipeline de CI/CD ! Ici on fait place à des spécialistes : des solutions qui poussent le concept beaucoup plus loin.

Deux champions se partagent le gâteau pour l'**approche _Pull_ du GitOps** : **Flux** et **Argo**.

Si à un moment de l’histoire les deux projets ont envisagé de merger pour former une unique solution (sans succès), c’est qu’ils ont une approche assez différente du modèle GitOps (Pull). Encore une fois, je ne rentre pas dans le détail car d’autres billets de blog dédiés vous donneront bien plus d’informations.

D’un côté, [**Flux**](https://fluxcd.io/) est une solution cluster centrique, i.e. à installer sur chaque cluster et qui se comporte de façon autonome.

De l’autre côté, [**ArgoCD**](https://argoproj.github.io/cd/) est plus orienté _workflow_, avec notablement une interface graphique qui permet de redonner la main facilement à l’humain. ArgoCD est plus naturellement orienté multi-environnement (et donc multi-cluster Kubernetes la plupart du temps).

Dans les deux cas vous aurez à adapter votre organisation autour des outils, car ils vont challenger les vieilles habitudes de déploiement … soit par prise de décision automatique, soit parce que le fonctionnement interne de la solution n’est pas très lisible / compréhensible facilement. Il faudra donc faire confiance à l’outil.

Notez bien ! On parle de **GitOps pull** car les deux solutions vont **se connecter à vos repositories Git** et ensuite appliquer des modifications sur vos environnements.

Les deux solutions sont aujourd’hui assez matures pour être utilisées de façon industrielle, pas d’inquiétude sur ce point !

Des **avantages** :

- Méthode normalisée / guidée / documentée de mise en production
- Vous n’avez plus à maintenir un pipeline de déploiement
- Réconciliation automatique entre l’état cible demandé et l’état actuel
- Si vous considérez que votre plateforme git ne devrait pas avoir accès à votre production, vous êtes au bon endroit :)

Quelques **Inconvénients** :

- Manque de flexibilité. Impossibilité d’intégrer certains outils (ex: Ansible).
- Les deux solutions dépendent de Kubernetes.
- Les changements en direct sur un cluster sont chaotiques, ils seraient écrasés en quelques secondes par le système.

## GitOps et Kubernetes

![Logos de GitOps et de Kubernetes](https://enix.io/fr/blog/gitops-definition/gitops-kubernetes.webp)

Je réponds à une question fréquente. On est resté volontairement générique jusqu’à présent, et on voit bien que cette approche GitOps est **applicable à tout type d’infrastructure** dont on souhaite automatiser le déploiement.

Mais c’est bien **avec l’évolution vers les conteneurs et l’utilisation de Kubernetes que le GitOps s’est démocratisé**. C’est logique quand on connaît la nature déclarative et centralisée de Kubernetes. Et les conteneurs applicatifs sont eux aussi souvent déployés via un pipeline de déploiement continu s’appuyant sur Git (Gitlab CI, Github Actions, etc.). On peut utiliser le même repo Git centralisé pour stocker les données utiles pour déployer aussi bien l’infrastructure que les conteneurs applicatifs.

## Conclusions sur le GitOps

On ne se pose pas beaucoup de questions avant d’adopter la méthode GitOps. Et c’est tant mieux.

Les **gains sont importants** :

- Réversibilité
- Suivi des modifications de façon nominative, avec authentification
- Facilite la gestion dans le cadre d’équipes larges
- Le repo git peut être considéré comme une _source of truth_ et représente correctement les divers environnements dont la production

Cependant, on se pose beaucoup de questions quand il s’agît de la mettre en place. J’ai envie de **synthétiser** de la façon suivante la situation et pour faire court :

- Pour une utilisation simple : privilégiez l’approche push
- Pour en faire la pierre angulaire de votre organisation : privilégiez la méthode pull, en fonction de l’organisation de la société. Mais attention à bien former les équipe opérationnelles

Un petit avis très personnel en sus : Attention au jusqu’au-boutisme qui peut rendre le système très … trop ? complexe, rendant finalement l’ensemble difficile à maintenir et source de frustration pour les équipes au lieu de leur simplifier la vie.

Le **GitOps, un must have** dans tous les cas !