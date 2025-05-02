---
link: https://zonetuto.fr/intelligence-artificielle/les-logiciels-pour-utiliser-lintelligence-artificielle-en-local/
byline: MrTuto
site: ZoneTuto
date: 2025-01-04T06:18
excerpt: Quels sont les logiciels pour lancer les grands modèles de langage LLM en local sur Windows, macOS et Linux et profiter de l'IA sur son PC
slurped: 2025-02-22T09:25
title: Les logiciels pour utiliser l'intelligence artificielle en local
tags:
  - ia
  - local
  - intelligence-artificielle
---

Que le temps passe vite, cela faisait bien longtemps que je ne vous avais pas écrit un petit article sur l’intelligence artificielle. Ce n’est pas par manque d’envie, mais malheureusement encore une fois par manque de moments disponible pour vous écrire ici. Rassurez-vous je ne suis jamais très loin et j’essaie de garder le rythme pour vous raconter des choses intéressantes. On va donc parler d’IA et de LLM, car les choses continuent de bouger et c’est loin d’être fini même si le concept commence à montrer ses limites pour plein de raisons, mais ce sera l’occasion de faire un autre article sur le sujet.

Dans cet article, nous allons voir toutes les solutions pour faire tourner des grands modèles de langage que je vais plutôt appeler LLM pour « _large language model_ » dans la suite de cette page. Il est difficile d’être passé à côté si vous vous intéressez au domaine de l’intelligence artificielle, surtout depuis la révolution ChatGPT pour le grand public. Pour enfoncer le clou, les performances des LLMs ont très fortement progressé au cours de l’année. C’est surtout le cas pour les modèles d’intelligence artificielle qui sont librement accessibles et que vous pouvez télécharger puis utiliser sans restriction en local avec les bons logiciels sur votre machine. En parlant de performance, n’oubliez pas qu’il existe un [classement des meilleurs LLM](https://zonetuto.fr/intelligence-artificielle/llm-classement-meilleur-modele-ia/) qui est constamment mis à jour, histoire d’avoir la meilleure expérience avec l’intelligence artificielle en local.

Alors que les performances intrinsèques des LLM progressent, les programmes pour les faire tourner et les exploiter en local s’améliorent eux aussi. Avant, il fallait de très grosses machines bien puissantes pour faire tourner de petits LLM, à présent avec les nombreuses optimisations des logiciels, il faut beaucoup moins de puissance pour faire la même chose. Avec un ordinateur relativement basique, vous allez pouvoir utiliser des LLM en local sans trop de difficulté. Il y a toutefois quelques prérequis et limites, mais c’est facilement utilisable.

## Quelle configuration de PC pour utiliser l’IA en local ?

Avant de passer à la suite, je préfère quand même vous le préciser, selon les logiciels et le LLM que vous voulez utiliser, il faudra tout de même que votre ordinateur propose un minimum de puissance. Il n’y a pas de configuration type pour faire tourner un grand modèle de langage en local directement sur votre ordinateur, mais je préfère tout de même préciser quelques points. Voici mes recommandations concernant la configuration de votre machine basée sur mes expériences personnelles sont les suivantes :

- **CPU** : si vous voulez faire de l’inférence directement sur le processeur, il faut qu’il soit relativement récent avec les bonnes extensions. Pas besoin non plus d’une bête de course si vous avez une carte graphique correcte. Si vous n’avez pas une bonne carte graphique qui est adaptée à l’intelligence artificielle, mais que vous avez tout de même un bon processeur d’installé sur votre machine, rassurez, vous pourrez quand même faire tourner des modèles d’intelligence artificielle en local. Pas de manière optimale, mais ça marche avec des petits LLM. Si vous ne connaissez pas le modèle de votre processeur, vous pouvez utiliser des [commandes sous Linux pour avoir les informations sur votre CPU](https://zonetuto.fr/hardware/afficher-les-informations-de-son-processeur-sur-ubuntu-debian/) ou utiliser [CPU-X / CPU-Z](https://zonetuto.fr/hardware/cpu-x-alternative-libre-cpu-z-gpu-z-linux/).
- **RAM** : d’une manière générale, aujourd’hui, on considère que 16 Go de mémoire vive est vraiment la quantité minimale à avoir. Surtout si vous utilisez votre machine dans un cadre professionnel et de manière intensive. En dessous, cela risque d’être vite compliqué et vous ne pourrez pas charger le LLM en mémoire. Sur ma machine dédiée à l’intelligence artificielle, j’ai préféré avoir directement 32 Go pour que des questions de confort. Je peux charger des LLM dedans si je n’utilise pas la carte graphique et en plus, elle n’est pas saturée ce qui laisse de la place pour les autres applications de mon ordinateur sous Linux Ubuntu. Si vous avez une plateforme avec de la DDR4, vous pouvez même pousser plus loin pour un maximum de confort à condition que la mémoire vive soit rapide. C’est donc au moins 3200 MHz ou mieux 3600 MHz concernant la DDR4, sur la DDR5 le problème se pose moins.
- **Stockage** : il est primordial de prendre en compte la vitesse et la capacité de votre solution de stockage, car les modèles d’IA et les ensembles de données peuvent rapidement occuper un espace considérable. Un SSD NVMe performant vous fera gagner un temps précieux lors du chargement des modèles et des données, notamment pour les inférences réalisées directement sur le CPU ou lors du transfert entre le stockage et [la mémoire de la carte graphique](https://zonetuto.fr/hardware/test-jai-achete-une-carte-graphique-doccasion-sur-amazon/). Cependant, si vous prévoyez de conserver plusieurs modèles sur le long terme, prévoyez un espace de stockage conséquent, quitte à opter en complément pour un disque dur classique afin d’y archiver les modèles que vous n’utilisez pas au quotidien. Cette organisation permet de [préserver la durée de vie de votre SSD](https://zonetuto.fr/windows/comment-verifier-etat-sante-usure-disque-dur-ssd-crystaldiskinfo/), qui sera moins sollicité, tout en garantissant une fluidité optimale lors de vos travaux d’IA en local.
- **GPU :** C’est sans doute le point le plus critique. Pour des questions de confort et surtout de compatibilité, je vous conseille une carte graphique Nvidia. Beaucoup de logiciels sont compatibles avec les cartes AMD, mais pas tous, car ces dernières reposent sur ROCm et non CUDA. Globalement, Nvidia, c’est plus cher, mais cela reste à ce jour la solution la plus polyvalente et largement supportée dans l’écosystème de l’intelligence artificielle. Au-delà du choix de la marque, la quantité de mémoire vidéo, la VRAM disponible sur la carte graphique est primordiale : plus vous disposez de GDDR, plus vous pourrez charger de modèles complexes et lourds pour obtenir des performances optimales dans les réponses. Notez également que d’autres solutions commencent à émerger, comme certaines alternatives basées sur des chipsets spécialisés ou sur des architectures moins courantes, visant à offrir davantage de flexibilité et de puissance à moindre coût dans le domaine de l’IA.

Si votre configuration est assez solide, vous pourrez sans problème faire tourner des LLM en local que ce soit sur votre processeur ou avec votre carte graphique. La tendance est de toute façon à la réduction de la taille des modèles librement accessibles. Il y a de plus en plus de modèles plus petits, mais qui ont été entraîné avec des données de meilleur qualité et avec des techniques différentes. Ainsi, alors qu’ils ont beaucoup moins de paramètres et qu’ils pèsent donc beaucoup moins lourd, on retrouve des performances similaires aux modèles des précédentes générations qui sont pourtant beaucoup plus gros. C’est de toute façon un gros enjeu de la technologie actuelle des LLM, il n’est pas possible à l’heure actuelle de faire toujours plus gros, car le matériel et la consommation en énergie ne suis pas la demande grandissante.

## Comment utiliser un grand modèle de langage LLM en local sur Linux, macOS et Windows

Globalement, tous les logiciels que je vais vous présenter ici sont d’office compatibles avec la plupart des distributions Linux. C’est bien normal, car il s’agit du meilleur environnement pour travailler avec ce genre de technologie et tout est librement accessible. De toute façon, même si certains logiciels apportent de bonnes améliorations, le nerf de la guerre reste la qualité intrinsèque des modèles d’intelligence artificielle. C’est valable peu importe le logiciel que vous allez utiliser pour exploiter un LLM. C’est juste que certains sont plus confortable à utiliser que d’autres et surtout plus facile d’utilisation pour ceux qui débutent.

### Ollama

![](https://zonetuto.fr/wp-content/uploads/2024/12/ollama-run-llm-local-ia-intelligence-artificielle-1024x547.png)

Parmi tous les outils que je vais vous présenter dans la suite de cet article pour utiliser l’IA en local, Ollama est de loin mon préféré. Il est très simple d’utilisation et est compatible avec Windows 10 & 11, macOS et bien évidemment Linux.

Il est possible de récupérer les différents modèles directement à partir du logiciel Ollama donc vous ne vous vous compliquez pas la vie à aller les chercher partout sur Internet. Ollama fonctionne avec une interface graphique, mais si vous adepte comme moi de la ligne de commande, il est possible de l’utiliser seulement avec le terminal.

Je ferais un article plus complet sur Ollama en particulier, mais si vous êtes débutants dans le domaine de l’intelligence artificielle, c’est vraiment celui que je vous conseille ! Pour le télécharger vous pouvez passer par [le site officiel de Ollama](https://ollama.com/).

### GPT4All

![](https://zonetuto.fr/wp-content/uploads/2024/12/nomic-gpt4all-run-llm-local-ia-intelligence-artificielle-logiciel-1024x558.jpg)

Je vous avais déjà parlé il y a longtemps de [GPT4All dans cet article](https://zonetuto.fr/intelligence-artificielle/gpt4all-une-ia-avec-interface-graphique-a-installer-sur-son-pc/) et depuis les choses ont bien évolué. Le logiciel a depuis reçu beaucoup de mise à jour et il s’est bien amélioré. Il supporte bien évidemment la plupart des modèles de LLM et dans GPT4All, il est aussi possible de les récupérer directement en quelques clics.

GPT4All fonctionne lui aussi sur Windows, macOS et Linux ce qui permettra à tout le monde d’utiliser l’intelligence artificielle en local et ce peu importe son système d’exploitation préféré. Il peut utiliser votre processeur ou carte graphique selon votre configuration et supporte très bien les processeurs ARM Apple Silicon du M1 au M4. Si vous avez une configuration suffisante, ce sera donc très simple d’utiliser l’IA en local avec GPT4All.

GPT4All vous permet aussi de facilement utiliser l’IA avec vos propres documents et bien évidemment toujours en local. Très utile si pour vous la vie privée est importante ou surtout s’il s’agit de document avec des informations confidentielles comme c’est souvent le cas en entreprise. Si vous voulez tester GTP4All, [rendez-vous sur leur site officiel](https://www.nomic.ai/gpt4all).

### LM Studio

![](https://zonetuto.fr/wp-content/uploads/2024/12/lm-studio-run-llm-local-ia-inteliigence-artificielle-1024x387.png)

Je ne l’ai pas précisé pour Ollama et GPT4All, mais comme eux, LM Studio est lui aussi totalement open source et gratuit. Dans ce domaine de l’intelligence artificielle, c’est encore une fois très agréable de retrouver autant de logiciels avec des sources ouverte et donc beaucoup de partage. Cela permet à toute la communauté d’avancer encore plus vite, car il reste tellement à faire. Pour tous ces logiciels qui permettent de faire de l’inférence en local, vous pouvez facilement retrouver les [dépôts git](https://zonetuto.fr/linux/deployer-un-projet-heberge-dans-gitlab-sur-ubuntu-avec-cle-ssh-et-git/) sur GitHub très facilement à partir de [leur site officiel](https://lmstudio.ai/). C’est aussi le cas sur le site de présentation de LM Studio.

Comme ses « concurrents » il permet d’utiliser des LLM en local et de pratiquer l’intelligence artificielle à la maison sur sa propre machine. Attention ils précisent tout de même que si vous voulez faire de l’inférence sur le CPU, il faudra un processeur avec au moins l’[extension AVX2](https://fr.wikipedia.org/wiki/Advanced_Vector_Extensions). Rassurez-vous, c’est disponible sur la plupart des processeur grand public, car elle est sortie il y a une dizaine d’années. LM Studio est aussi compatible avec l’ensemble des processeurs ARM Apple Silicon : M1, M2, M3 et plus récemment M4.

Du côté de LM Studio, on retrouve les fonctionnalités habituelles telles que l’inférence, la possibilité de récupérer à partir du logiciel les modèles les plus populaires de Hugging Face et depuis récemment, la possibilité d’analyser et discuter avec vos documents en local. Vous pouvez changer de thème et surtout, il y a beaucoup de langues supportés pour l’interface, agréable si vous n’êtes pas à l’aise avec l’anglais.

### Jan

![](https://zonetuto.fr/wp-content/uploads/2025/01/jan-llm-local-private-ia-intelligence-artificielle-925x1024.jpg)

Jan est un autre outil assez connu pour exécuter des modèles LLM en local et encore une fois, c’est très facile à utiliser. Vous pouvez facilement récupérer des modèles directement à partir du logiciel. Je sais qu’il a ses adeptes avec son interface épurée et claire. Si vous allez faire un tour [sur leur GitHub](https://github.com/janhq/jan), vous verrez que c’est un projet très suivi qui bénéfice de beaucoup de mise à jour. Au moment où j’écris ces lignes, le dernier commit a été fait il y a seulement 4 h, c’est très vivant. Ce n’est pas celui que j’ai le plus utilisé, mais il fait bien le travail et s’appuie comme les autres sur des briques solides avec notamment llama.cpp.

Pour information, il est écrit en TypeScript donc si c’est un langage de programmation que vous maîtrisez bien, ce projet pourrait vous intéresser plus que les autres. Rassurez-vous n’avez pas besoin de maîtriser TypeScript pour l’utiliser. Sinon il fait globalement la même chose et donc on est plutôt dans les goûts et les couleurs. Enfin dernière information, il est compatible directement avec les distributions Linux dérivées de Debian, Windows et macOS sous Intel et Apple Silicon.

### vLLM

![](https://zonetuto.fr/wp-content/uploads/2025/01/vllm-inference-llm-local-cpu-gpu-cuda-rocm.png)

Pour ce logiciel, c’est un peu plus technique et il devrait intéresser ceux qui sont déjà familiers avec la manipulation des LLMs en local. vLLM est un outil open source qui offre des performances impressionnantes pour l’inférence et le « serving » de modèles LLM. Il utilise une technologie appelée PagedAttention qui gère la mémoire de manière plus souple, un peu comme le fait un système d’exploitation avec la pagination, ce qui permet de stocker et de manipuler plus efficacement les fameuses clés et valeurs générées lors de l’inférence. Cette approche évite le gaspillage de ressources et permet de servir plus de requêtes simultanément, tout en gardant des temps de réponse très rapides.

En pratique, ça peut multiplier les performances par plusieurs dizaines par rapport à d’autres bibliothèques d’IA tout aussi populaires. Avec vLLM, même des tâches d’échantillonnage ou de génération parallèle peuvent être menées sans surcharger votre matériel, grâce au partage mémoire intégré. C’est en partie ce qui rend possible l’utilisation de grands modèles de langage de façon abordable et efficace, notamment pour des projets de recherche ou des déploiements qui ne bénéficient pas de ressources illimitées. Pour ceux qui souhaitent en savoir plus, [le projet vLLM est disponible sur GitHub](https://github.com/vllm-project/vllm) et continue d’évoluer rapidement.