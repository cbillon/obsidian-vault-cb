---
link: https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique
byline: gUI
excerpt: "IA : mon parcours initiatique"
tags:
  - ai
slurped: 2026-06-23T07:47
title: "IA : mon parcours initiatique"
---

## Sommaire

- [Niveau 0 : ChatGPT dans le browser](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#toc-niveau-0--chatgpt-dans-le-browser)
- [Niveau 1 : LLM local](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#toc-niveau-1--llm-local)
- [Niveau 2 : Agent IA](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#toc-niveau-2--agent-ia)
    - [Claude code](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#toc-claude-code)
    - [CLAUDE.md](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#toc-claudemd)
    - [Open code avec Ollama](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#toc-open-code-avec-ollama)
- [Niveau 3 : Skills](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#toc-niveau-3--skills)
- [Niveau 4 : MCP](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#toc-niveau-4--mcp)
- [Conclusion](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#toc-conclusion)


Je m'intéresse de près à l'IA depuis peu, et il m'a fallu un certain temps pour comprendre ce que je devais chercher en fait. Le fameux "claude code" ou les "agents" étaient un peu nébuleux, mais j'avais bien compris que c'était autre chose que de faire ses requêtes dans un browser web.

Si vous êtes comme moi il y a quelques mois, voici l'article que j'aurais aimé lire à l'époque.

## Niveau 0 : ChatGPT dans le browser

J'ai commencé par [ChatGPT](https://chatgpt.com/) à ses débuts. Comme beaucoup je suppose. Au tout début je rigolais de le voir me citer "les 3 capitales des États-Unis" sans broncher. Bon, ça n'a pas trop duré, rapidement il a su me répondre qu'il n'y en a qu'une mais que d'autres villes importantes _blablabla_, veux-tu que je te liste _blablabla_.

En matière de code, j'ai pas mal essayé de le faire bosser à ma place, avec des résultats mitigés. Un copier/coller de quelques centaines de lignes de log et il me pointe exactement le souci en 10 secondes, c'est arrivé. Une correction de bug en le décrivant et en lui donnant le code actuel aussi. Mais ça ne l'a pas empêché d'être souvent en dehors de la plaque, de me donner des corrections fausses, ou de tourner en rond en étant persuadé qu'il avait raison.

J'appelle ça le niveau 0 parce que ça permet de se donner une idée de la puissance du machin tout de même, mais ça une utilisation vraiment restreinte. C'est adapté au LLM (parler avec une machine en langage naturel) mais pas trop à la génération de code.

## Niveau 1 : LLM local

Ensuite de par la curiosité d'une part mais aussi l'envie de dépendre le moins possible de service extérieurs (payants ou pas), j'ai regardé du côté des IA qu'on peut faire tourner localement.

D'abord sur mon CPU, puis en achetant une carte graphique AMD avec pas mal de RAM mais dont les joueurs ne veulent pas (une AMD avec 24Go de VRAM que j'ai payé 600€).

Honnêtement je n'ai pas encore beaucoup creusé cela, mais je sais déjà faire le minimum :

- utiliser un modèle proposé par [Ollama](https://ollama.com/) ou [Hugging Faces](https://huggingface.co/)
- le paramétrer légèrement (taille du contexte, température)

Ça m'a permis facilement de mettre rapidement le doigt sur ces deux paramètres fondamentaux. En gros la [fenêtre de contexte](https://fr.wikipedia.org/wiki/Fen%C3%AAtre_de_contexte) est la taille des informations que le modèle peut traiter en même temps. Il contient les instructions, mais aussi les données à traiter. Plus le contexte est gros moins il va halluciner et raconter des conneries, mais bien évidemment plus ce sera consommateur en ressources.

Pour la [température](https://www.ibm.com/fr-fr/think/topics/llm-temperature) c'est un peu à quel point il peut être "farfelu" et "créatif". Par exemple si on fournit une capture d'écran effectuée lors d'un test automatique et qu'on demande 10x à un même LLM de nous dire ce qu'il y voit dedans, il nous dira en gros la même chose, mais avec des mots différents, parfois les détails diffèreront etc. Par contre si on met une température à 1 (qui veut dire "aucune imagination", en opposé à 0 qui veut dire "t'es no limit !", et toutes les valeurs intermédiaires étant des nuances), on ne sait toujours pas à l'avance ce qu'il sortira, mais par contre si on lui demande 10x avec la même source il ressortira 10x exactement le même texte.

Bon, j'ai également pu voir l'intérêt de payer pour un service extérieur, les performances sont sans commune mesure en matière de rapidité, mais aussi de qualité. Et notamment parce que les modèles qui tournent dans les services externes peuvent avoir une fenêtre de contexte de l'ordre de 10 ou 100x supérieures à ce que je peux faire en local.

Pour du LLM (parler à une machine en langage naturel) la solution locale est vraiment pas mal. Mais pour du code, passé le _boilerplate_ d'un script bash ou python, c'est très limité.

J'appelle ça le niveau 1 parce que c'est tout aussi peu utilisable, mais au moins on comprend 2 ou 3 trucs, on devient (un peu) acteur.

## Niveau 2 : Agent IA

C'est clairement là où j'ai compris le potentiel de l'IA dans le domaine du développement logiciel.

Un "agent" c'est un programme (codé en python, Rust, C++… tout ce qu'il y a d'habituel) qui fait l'intermédiaire entre nous, pauvres humains, et l'IA. C'est lui qui transmet notre demande, et c'est lui qui va recevoir la réponse.

La différence avec le browser web ? C'est que l'échange est un peu plus formalisé, sous forme de fichier JSON généralement. Ainsi on peut gérer plus finement le contexte (fichiers joints, prompts de différents niveaux…) mais aussi la réponse peut contenir des actions comme écrire dans un fichier, exécuter une commande `git` etc.

Toutes ces actions sont concrètement exécutées par l'agent lui-même. Il faut donc veiller à ce qu'il n'accepte pas de faire n'importe quoi. En général l'agent va demander les autorisations au fur et à mesure, en laissant la possibilité d'accepter par défaut toutes les prochaines actions du même type.

### Claude code

J'ai commencé à utiliser `claude code` comme agent. Par défaut il utilise [Claude](https://claude.ai/), l'IA dans le cloud comme LLM. Et là oui, c'est impressionnant. Sur une base de code existante où on veut rajouter une fonctionnalité il va d'abord lire tout le code (évidemment, faut que ça rentre dans la fenêtre de contexte, si le code est vraiment trop gros il ne lira pas _tout_). Ensuite selon ce qu'on lui a demandé de faire, il peut juste proposer la modification, mais aussi la faire, la tester, faire le commit git… Tout cela en quelques minutes et quelques aller/retour entre l'agent et le modèle IA. Ça se déroule sous nos yeux, c'est réellement fascinant.

### CLAUDE.md

C'est aussi la découverte du fameux CLAUDE.md. Ce fichier est par défaut lu par l'agent claude code, et ajouté dans le contexte du LLM. C'est un peu un prompt par défaut. En voici un bref exemple juste pour avoir une idée:

```
# langage
tout doit être réalisé en python3

# langue
parle-moi en français mais rédige le code (classes, fonctions, cariables, commentaires) en anglais

# règles de codage
application stricte de PEP-8
```

Ensuite dans l'outil si on dit simplement "fais-moi un programme des tours de Hanoï" ce sera donc écrit en python3, le code en anglais, et il vérifiera bien que les recommandations PEP-8 sont appliquées.

Il y a plein de littérature sur ces fichiers (au pluriel, parce que dans ce même fichier on peut par exemple écrire "lis GIT.md pour savoir comment rédiger les commit git" et donc dédier un fichier à la rédaction des commits git), et c'est clairement une nouvelle compétence de développeur de savoir l'écrire et le gérer. Entre "chaque codeur le sien", et à l'opposé "un seul qui est commun pour tout le projet", toutes les saveurs existent !

### Open code avec Ollama

Toujours dans l'iée de ne pas être accroc à un abonnement (mais ça devient de plus en plus dur… ) je me suis amusé à utiliser `open code` au lieu de `claude code` (un autre outil d'agent IA, très ressemblant), et mon instance locale ollama au lieu de Claude. J'ai fais quelques essais avec le LLM [ministral](https://ollama.com/library/ministral-3) (cocorico !) ou [qwen3](https://ollama.com/library/qwen3.6) (celui d'Alibaba, le chinois qui devient connu également pour ses LLM). Concrètement ça marche certes, mais évidemment c'est moins rapide, moins pertinent…

J'appelle ça le niveau 2 parce que clairement, on a passé un niveau ! Un niveau en matière d'utilisabilité, oui on peut coder, trouver des bugs, refactorer… mais aussi on est un peu plus impliqué dans le résultat final, de par la qualité des prompts demandés.

## Niveau 3 : Skills

Comme dit précédemment, la fenêtre de contexte est assez stratégique. C'est en gros ce qui coûte en ressources, il faut la gérer au mieux. Un "skill" ça pourrait être caricaturé par "oublie tout ce que j'ai dis jusqu'à présent et concentre-toi sur un point particulier".

Concrètement, c'est un nouveau fichier markdwon écrit à destination de l'agent (Claude code par exemple). C'est un nouveau prompt, mais spécialisé sur un point particulier.

Par exemple on va créer un skill de relecteur de code. En lui disant quelque chose comme "tu vérifies les best practices, les règles de codage. tu vérifies qu'il n'y a pas de code dupliqué" il va relire le code que le même LLM a généré et le voir d'un autre œil. Selon le contexte et la situation, il arrive qu'il corrige ainsi son propre code !

Idem pour un agent spécialisé cybersécurité par exemple, il peut écrire un rapport par exemple en suggérant des améliorations de code.

J'appelle ça le niveau 3 parce que ça permet de comprendre réellement les limites des LLM. Vraiment, tu n'as que ce que tu demandes. On le sait depuis le début, c'est un évidence, mais là ça formalise l'exercice de parler différemment au même LLM pour avoir différents comportements, complémentaires souvent, mais parfois même contradictoires.

## Niveau 4 : MCP

Quand on utilise un agent IA, comme Claude code, le LLM sait faire certaines choses basiques : lire un fichier, le modifier, exécuter une commande shell (comme `grep`, `sed` ou `git`) mais pas grand-chose de plus. On peut lui ajouter des fonctionnalités via le protocole MCP ([Model Context Protocol](https://fr.wikipedia.org/wiki/Model_Context_Protocol)).

Un exemple qu'on peut citer ce serait un LLM qui remplace le SAV d'une entreprise de VPC. En codant un MCP permettant d'accéder à la base de donnée, le LLM pourra répondre à M. Toutlemonde "oui, je vous confirme que votre paquet a été posté hier". Ce n'est pas un LLM entraîné pour, c'est un LLM tout à fait standard qu'on prend tel quel et auquel on explique simplement comment faire telle ou telle action.

Concrètement il s'agit de code python (qui utilise un SDK dédié) et qui fournit quelques fonctions. Ensuite encore et toujours via un fichier markdown on décrira l'utilisation. Par exemple "pour accéder à la base de donnée tu dois utiliser le MCP my_bdd.". Les docstrings de chaque fonction suffisent souvent au LLM à comprendre leur but et leur mode d'emploi. Mais si on veut détailler des choses, il "suffit" de les ajouter dans ce markdown.

Dans mon cas, j'ai commencé à écrire un MCP utilisant `labgrid`, un [framework d'accès à des machines physiques](https://labgrid.readthedocs.io/en/latest/#) en vue de tests automatiques. Je le fais tourner en local sur ollama avec un LLM standard (ministral), et du coup je peux lui dire "dis-moi la RAM libre au moment du boot sur les produits de type X, Y et Z". Il saura réserver chacun des bancs de tests, le booter, lancer une commande SSH simple et me donner le résultat. Pour l'instant ça ne me sert à rien (j'ai déjà des tests pytests qui le font formellement) mais je voulais tenter l'expérience, et il faut reconnaître que c'est assez troublant de voir avec quelle facilité on peut permettre à une IA d'accéder au monde réel.

J'appelle ça le niveau 4 parce qu’on apprend à donner encore plus de possibilités à son agent LLM, et on peut ainsi sortir du cadre imposé par l'outil agent.

## Conclusion

Voilà, c'est là où j'en suis, et j'ai déjà quelques pistes pour la suite.

- RAG : il s'agit en gros de prendre un modèle et de l'entraîner un peu plus sur ses propres données (typiquement de la doc) (EDIT en fait ça pas ça, voir [ce fil de commentaires](app://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#comment-2022053) pour plus de détails). J'en sais pas plus, et maintenant que je sais utiliser MCP je ne vois pas vraiment les cas d'usage que ça pourrait combler
- Gas Town : quelques collègues ont commencé à jouer avec [Gas Town](https://github.com/gastownhall/gastown). Si j'ai bien compris, il s'agit d'une organisation complète de différents agents/skills qui s'organisent et se supervisent entre eux

J'espère que ce journal a permis de mieux faire comprendre les quelques termes techniques de base qui gravitent autour de l'utilisation des LLM dans le contexte du développement logiciel, qu'il permettra de mieux comprendre les prochaines publications sur LinuxFR.org qui deviendront fatalement de plus en plus techniques sur les sujets d'IA, et évidemment axées sur le développement logiciel. En tous cas j'ai passé pas mal de temps à voir passer des infos que je ne comprenais pas du tout, tout en me disant que je passais à côté de quelque chose.

Bref, si vous jugez l'IA pour le développement logiciel en étant resté simplement à copier/coller du code dans l'interface web d'un LLM, je vous engage à essayer d'aller plus loin, c'est réellement une autre dimension.

## RAG

Le point vraiment différent, de ce que je comprends (je ne suis pas expert du tout), c'est que pour le RAG il faut passer tes données dans une moulinette pour peupler une vector database qui va permettre d'interroger tes documents de manière assez efficace. Il va falloir fine-tuner les paramètres de cette moulinette avec des valeurs qui dépendent assez largement de la nature des documents traités.

J'avais commencé à regarder aussi, ça semblait assez efficace, mais complexe à mettre en place et paramétrer.

En fait je trouve que ça fait beaucoup d'overhead et que ça limite les cas d'utilisation. Notamment, ça me semble plus adaptés à des corpus documentaires qui ne changent pas (parceque sinon, te devrais mettre à jour ta vector database, et je ne crois pas que l'on puisse faire ça par delta).

Au final, depuis que j'ai découvert les MCP, je trouve ça plus flexible d'un point de vue utilisation.

[Répondre](https://linuxfr.org/nodes/143430/comments/2022057/answer#new_comment)

- ## [[^]](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#comment-2022057 "Remonter au commentaire parent") [#](https://linuxfr.org/users/gbetous/journaux/ia-mon-parcours-initiatique#comment-2022062 "Lien direct vers ce commentaire") [Re: RAG](https://linuxfr.org/nodes/143430/comments/2022062)
    
    
    
    Tout a fait.
    
    ![](https://img.linuxfr.org/img/68747470733a2f2f75706c6f61642e77696b696d656469612e6f72672f77696b6970656469612f636f6d6d6f6e732f312f31342f5241475f6469616772616d2e737667/RAG_diagram.svg "Source : https://upload.wikimedia.org/wikipedia/commons/1/14/RAG_diagram.svg")
    
    ([source](https://fr.wikipedia.org/wiki/G%C3%A9n%C3%A9ration_%C3%A0_enrichissement_contextuel))
    
    En amont tu génères les vecteurs sémantiques ("embedding") de tes documents en utilisant un modèle spécialisé pour ça ("embedding model") et tu fous tout ça dans une base de donnée.
    
    Puis au runtime, tu fais une requête en langage naturel qui est elle aussi transformé en vecteur sémantique avec le modèle spécialisé que tu compares aux vecteurs de documents pour voir lesquels sont les plus proches.
    
    > En fait je trouve que ça fait beaucoup d'overhead et que ça limite les cas d'utilisation. Notamment, ça me semble plus adaptés à des corpus documentaires qui ne changent pas (parceque sinon, te devrais mettre à jour ta vector database, et je ne crois pas que l'on puisse faire ça par delta).
    
    Tu n'as besoin de mettre à jour que les vecteurs des documents qui ont changés, pas recalculer toute la base de donnée

Le RAG permets plus de borner la réponse pour éviter les hallucinations.

Le résultat des documents matchés est réinjecté dans le prompt si tu fais bien les choses.

Il y a donc 2 étapes dans le RAG : on cherche des parties de documents liées au prompt, puis on utilise une template pour reformuler le prompt en injectant le résultat de l'étape précédente.

Comme dit Barmic, c'est différent du fine tuning, c'est très économique (si tu caches tes vecteurs en db) …

Comme tu le mentionnes, j'ai l'impression qu'il y a vraiment cette notion de "parcours initiatique". Et tant qu'on ne le suit pas d'une manière où une autre, on peut lire plein d'articles, mais ça reste flou.

Quelques petits commentaires :

- je n'aurais pas classé skills et MCP, parceque ce sont 2 choses à mon avis complémentaires
- je n'aurais pas défini Claude Code comme un agent IA, mais comme un outil permettant faire de l'IA agentique, au sens où il fournit une infrastructure modulaire avec des skills, des MCP, et surtout des agents (tu peux aussi en définir toi même) qu'il va orchestrer pour réaliser des tâches complexes.
- au sein de Claude Code, j'aurais même rajouté une étape : l'utilisation du mode "plan" - en planifiant le travail à venir, on obtient des résultats beaucoup plus pertinents avec moins d'itérations coûteuses

Mon expérience :

J'utilise au travail Claude Code, mais à titre perso, j'essaie de fonctionner avec Mistral Vibe branché sur un modèle devstral-small. C'est clairement plus rustique, mais ça répond à pas mal de mes besoins, avec une consommation beaucoup raisonnable que des modèles plus grands. Ces 2 configurations fonctionnent pour moi parcequ'elles correspondent à 2 objectifs complètement différents :

- au travail : je ne produis quasi plus de code, et l'enjeu est de réussir à affiner skills, utilisation de MCP, stratégies de prompting pour réduire les itérations nécessaires
- en perso : je veux rester rester auteur de mon code, j'utilise principalement l'IA pour sa rapidité d'analyse de code, pour rapidement avoir une idée de comment marche telle ou telle API, pour investiguer des erreurs complexes, et pour confronter mes choix et éventuellement découvrir des alternatives

