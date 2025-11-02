---
link: https://mcorbin.fr/posts/2025-10-17-genai-dev/
site: "@_mcorbin"
excerpt: mcorbin Tech Blog
twitter: https://twitter.com/@_mcorbin
slurped: 2025-10-26T16:32
title: La GenAI pour les développeurs, un réel gain de productivité?
tags:
  - ai
  - genai
---

Tout le monde parle de GenAI en ce moment, notamment pour les développeurs. Fin des dev, remplacés par des AI, ou au contraire génération de dette technique en masse, on entend de tout sur ce sujet. Qu’en est-il réellement?

Fait rare, je vais commencer cet article par une petite introduction.  
Je code aujourd’hui depuis plus de 18 ans, dont plus de 10 ans de manière pro. Même lors de mes passages côté infrastructure (j’ai longtemps travaillé comme sysadmin/SRE), le développement était essentiel que ce soit pour construire de l’outillage interne (platform engineering) ou pour aider les équipes de développement sur des problématiques liées au code ou à l’architecture en général.

J’ai également toujours travaillé (et je travaille toujours) dans des contextes que je qualifierai d’exigeant sur la qualité attendue, et où un défaut de qualité peut avoir des conséquences importantes pour l’entreprise et ses clients. C’est un point important que je développerai plus loin dans cet article.

Quand l’AI a débarqué dans le dev, j’étais assez sceptique. Je trouvais que c’était surtout de l’autocomplétion améliorée et le code produit était souvent pas terrible, donc je n’avais pas vraiment l’impression de gagner en productivité.

Ma vision a maintenant changée et j’utilise de plus en plus l’AI au quotidien, pour le dev ou autre.

## Mon utilisation de l’AI pour le développement.

J’utilise aujourd’hui exclusivement Claude Code comme assistant de code, dans le terminal.  
Claude Code m’a vraiment fait changer d’avis sur l’AI. J’ai toujours connu la course à l’échalote sur les IDE, qui incluaient de plus en plus de fonctionnalités (et tant mieux!).

Si on m’avait dit que je pourrai développer depuis un simple terminal en langage natural il y a 5 ans, j’aurai bien rigolé. Comment une interface aussi simple (taper du texte sans fioritures) pourrait être suffisante pour bosser sur quelque chose d’aussi complexe qu’une base de code? Et pourtant c’est le cas. Ca m’a également beaucoup fait réfléchir sur les produits grands publics de demain et à quoi ressemblera leurs UX, mais on en parlera dans un autre article.

J’ai tenté de classifier en trois catégories mon utilisation de l’AI aujourd’hui, que je vais maintenant présenter.

### Vide coding

Et oui, contrairement à toute attente, je code parfois en laissant l’AI faire tout le travail et en ne regardant même pas le code généré.

Il m’arrive parfois de devoir écrire des scripts non liées à la production. Par exemple, j’ai récemment dû faire pas mal de scripting pour fusionner selon certains critères des fichiers csv et JSON que j’avais en local, pour ensuite produire d’autres fichiers. Je me rappelle également d’un autre use case où j’avais téléchargé des traces opentelemetry depuis Grafana Tempo en JSON et où j’avais ici aussi besoin d’écrire des scripts pénibles pour extraire certaines données.

Aujourd’hui, j’utilise quasi exclusivement l’AI pour générer ce type de scripts, et le gain de productivité est énorme. En quelques minutes (voir moins), je sors des scripts que j’aurai mis énormément de temps à écrire à la main (probablement en heures). Claude Code sait de lui même analyser le format et la structure des données en entrée, et produire le script qui va bien, généralement en Python. Encore une bonne raison [pour ne plus avoir à apprendre Bash](https://mcorbin.fr/posts/2022-04-28-bash-administration-2022/), l’AI le générera pour vous si besoin ;)

Je vibe code également aujourd’hui tout ce qui est interface.  
J’ai eu des cas récents où j’avais, encore une fois, des fichiers JSON complexes en local que je voulais pouvoir montrer "de manière jolie". et même partager avec des non-tech. Là aussi, j’ai utilisé Claude Code pour me générer des HTML statiques embarquant les données (généralement je lui demande de me faire un script qui génère ensuite un HTML depuis un fichier en entrée). Cela m’a permis de produire très rapidement des trucs corrects et d’avoir des visualisations de données à montrer.

J’ai régulièrement des cas où je veux créer des interfaces simples pour des API, pour de l’outillage interne où je me fiche un peu de la qualité du code du front. J’utilise énormément l’AI pour ça et cela marche très bien.  
Une interface simple avec un menu, des formulaires et des pages pour afficher ou intéragir avec des données, l’AI le génèrera très bien depuis la spec OpenAPI du backend.

Ce dernier exemple m’inspire une nouvelle réflexion: l’AI nous permet d’imaginer un monde "Proof of Concept first" où en quelques heures on peut sortir un produit certe dégueulasse et non maintenable sur le long terme mais qui marche, avec une interface et donc des interactions possibles. A voir si ça vaut toujours le coup mais je trouve l’idée de rapidement itérer sur des interfaces qui tapent de vrais APIs (avec de la vraie donnée) assez intéressante.

### Mode chirurgien

J’ai eu récemment à écrire un slackbot en Go. Franchement, le SDK Slack est d’une complexité inimaginable, notamment sur le payload reçu par le bot (des structs Go géantes avec énorméments de champs et d’imbrications où on ne sait pas exactement ce qu’on doit extraire). J’ai donc demandé à mon ami Claude de m’extraire tout ce dont j’avais besoin en fonction du code qui générait l’interaction utilisateur dans Slack, et il m’a one-shot le truc.  
Bref, pour des questions très précises, c’est très puissant. Auparavant j’aurai cherché sur Google/Stackoverflow, maintenant l’AI permet d’aller beaucoup plus vite.

L’AI marche également bien pour du bugfix, et j’ai l’exemple parfait (true story).  
On utilise Clickhouse pour les traces dans ma boîte. Il y avait un bug dans le plugin Grafana de clickhouse qui empêchait l’affichage des "span events" sur les traces, et c’était super handicapant lorsqu’on investiguait des problèmes en production. Un collègue a donc [créé](https://github.com/grafana/clickhouse-datasource/issues/1262) une issue sur le Github du plugin.  
Lors de l’investigation d’un incident en Août, je me reprends le bug et je craque: je clone le plugin, je lance Claude Code, je lui mets un prompt similaire à `This is the clickhouse datasource for Grafana, fix this bug: [https://github.com/grafana/clickhouse-datasource/issues/1262](https://github.com/grafana/clickhouse-datasource/issues/1262)` (c’était vraiment ça, pas plus) et …​ en une minute, il me [fix le bug](https://github.com/grafana/clickhouse-datasource/pull/1345).  
On parle ici d’une base de code non triviale, assez complexe et avec une logique particulière (écrire un plugin Grafana, on a déjà connu plus fun). Pourtant, l’AI s’en est sorti du premier coup juste en partant de l’issue Github. Je ne sais franchement pas combien de temps j’aurai mis à trouver ce problème moi même, ou même si j’aurai réussi.

### Pour du code de production

J’écris énormément de Python en ce moment, pour des services backend et des produits liés à la GenAI.  
Quand vous avez le traditionnel "database / service / API layer" à écrire, avec les tests associés, l’AI permet de gagner du temps. Je fais moi même la partie SQL (modeling des données, définition des tables SQL…​) et les classes représentant ces données, puis je demande à Claude de me générer le "repository" (les fonctions du type create, get by ID, list, update, delete…​).  
Une fois la partie repository terminée, j’utilise toujours l’AI comme assistant pour faire la partie service/domaine puis API, ainsi que pour m’aider à écrire les tests. Comme vous pouvez le voir, je ne génère jamais toute la feature d’un coup, j’itère sur les layers de l’application ou des fichiers spécifiques jusqu’à avoir ce que je veux.

Je relis toujours le code produit et je n’hésite bien sûr pas à faire des ajustements, à la main ou via l’AI selon ce qui est le plus efficace. Mais je suis généralement satisfait de ce que me génère l’AI et j’ai également ici un bon gain de productivité.

L’AI peut aussi aider pour les reviews de pull request, ou même pour explorer une base de code. Il ne faut pas hésiter à l’utiliser comme canard pour comprendre comment quelque chose fonctionne.  
Claude Code permet également d’écrire des agents pouvant réaliser des tâches spécifiques. Par exemple, j’ai écrit récemment un agent qui, une fois invoqué, récupère les commentaires sur la pull request associée à ma branche locale, puis modifie le code en fonction.  
Si vous (ou vos collègues) avez des tâches répétitives à faire, écrire un agent est également très intéressant.

## Est ce que ça va remplacer les dev?

Certaines boîtes annoncent licensier des développeurs car "remplacés par l’AI" (mon opinion sur ça: 100 % bullshit, mais je présume que ça passe mieux dans les médias que "on a beaucoup trop recruté pendant le covid et maintenant on veut écrémer").  
C’est un peu comme l’exemple de Google qui, si je me souviens bien, comptait l’autocomplétion (la touche tab de l’IDE) dans ses "25 % de code généré par AI", alors que c’est pas du tout ce qu’on imagine quand on parle de code généré par AI.  
Bref, attention aux chiffres et aux discours de certaines entreprises ou C-level sur le sujet, tout le monde met du charbon dans la chaudière à hype mais pas forcément avec beaucoup d’honnêteté.

Néanmoins, Je vois clairement le gain de productivité apporté par l’AI et ce serait dommage de s’en passer, et je pense que chaque développeur doit commencer à s’équiper. Mais est ce que le métier de développeur va disparaître?

Il est déjà important de comprendre de quoi on parle quand on parle du métier de développeur.

Comme dit précédemment, j’ai la chance de bosser pour des boîtes où la tech est un élément essentiel de l’entreprise, et où les pratiques liées au dev ou l’infra sont assez avancées (voir dans ce qui se fait de meilleur en France sur certains sujets).  
Dans ces contextes, écrire du code (= taper du texte aevc une syntaxe spécifique pour produire un résultat) est clairement la partie facile du métier qui est de construire un produit.

Avant de développer, il y a toujours une phase de réflexion: qu’est ce qu’on veut construire, quelles sont les "success metrics" du produit, comment ce nouveau produit va s’insérer dans l’existant, quelles sont les difficultés techniques et autres edge cases que l’on va rencontrer pendant sa construction.  
Pendant cette phase, l’humain est essentiel et il y a d’ailleurs même un aspect social et organisationnel lorsqu’on construit un logiciel: comment on gère les dépendances aux autres équipes, comment on priorise et comment on s’organise…​

L’expertise technique est également indispensable, encore plus dans les entreprises d’une certaine taille où le SI est un système distribué composé de nombreux services et où il faut s’intégrer avec l’existant que ce soit en terme de gestion de données (base de données, event stream, eventual consistency…​), de tolérance aux pannes, scalabilité…​  
Construire une application robuste et correctement monitorée (traces, métriques, logs, alerting, cycle de vie de l’application…​) et sécurisée demande énormément de connaissances. L’AI peut aider sur certains points mais seul l’expérience permet de construire des produits maintenables dans le temps.

Pour être honnête, j’ai du mal à écrire cette dernière partie car c’est assez dur de transcrire tout ce que j’ai en tête sur le sujet des bonnes pratiques tech sans partir dans tous les sens. Je pourrai tripler la taille de cet article en continuant de citer des sujets ou points d’attention à avoir lorsqu’on construit un produit, et tout cela ne vient qu’avec l’expérience.  
D’ailleurs, l’AI est demandeuse de cette expérience. Avec Claude Code, vous pouvez donner des instructions à l’AI dans un fichier `CLAUDE.md`: mais indiquer à l’AI les bonnes pratiques à suivre demande déjà de les connaitre soit même.

Bref, le métier d’ingénieur logiciel ne va pas disparaître, loin de là. L’AI nous permet de nous libérer du temps en faisant pour nous les choses les moins intéressantes (écrire le code), et j’en suis très content.

On peut imaginer également un monde où des non-tech peuvent commencer à contribuer à un projet via de l’AI. Ca reste un sujet pas simple (notamment hors framework "no code" où faut quand même installer des outils type git, l’écosystème du langage etc en local, et où il faut une revue à la fin) mais je prévois de faire quelques experimentations sur le sujet dans l’année à venir.

En conclusion, si vous êtes tech, je vous recommande de vous y mettre aussi, et vous verrez que cela ne vous remplace mais mais vous donne des super pouvoirs.