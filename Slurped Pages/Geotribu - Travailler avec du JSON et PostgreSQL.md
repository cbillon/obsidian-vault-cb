---
link: https://geotribu.fr/articles/2025/2025-01-21_travailler-avec-JSON-et-PostgreSQL/#exemple-simple
byline: Thomas SZCZUREK-GAYANT
site: Geotribu
date: 2025-01-21T13:00
excerpt: Stocker des données au format json dans PostgreSQL, les consulter... et tout ça avec les données du recensement de l'INSEE pour l'exemple.
slurped: 2025-02-22T08:57
title: Geotribu - Travailler avec du JSON et PostgreSQL
tags:
  - json
  - postgresql
---

[JSON](app://obsidian.md/tags/#tag:json) [PostgreSQL](app://obsidian.md/tags/#tag:postgresql)

[](https://github.com/geotribu/website/edit/master/content/articles/2025/2025-01-21_travailler-avec-JSON-et-PostgreSQL.md "Editer cette page")[](https://github.com/geotribu/website/raw/master/content/articles/2025/2025-01-21_travailler-avec-JSON-et-PostgreSQL.md "Consulter la source de cette page")

![📆](https://cdn.jsdelivr.net/gh/jdecked/twemoji@15.1.0/assets/svg/1f4c6.svg ":calendar:") Date de publication initiale : 21 janvier 2025

![logo JSON](https://cdn.geotribu.fr/img/logos-icones/programmation/json.png)

Dans le cadre d'un projet personnel, j'ai voulu stocker une bonne partie des données du recensement de l'Insee dans une base PostgreSQL avec des tables multimillésimes. Problème, au sein d'un même jeu de données, les champs peuvent changer au cours des années et cela empêche de pouvoir dégager une structure de table fixe, ce qui est assez gênant vous en conviendrez. La solution ? Passer par des données semi-structurées, soit stocker ces données en JSON dans le champ d'une table. Cet article se veut un condensé de cette expérience.

Obsolescence non programmée

Ces travaux ont été réalisés avant la sortie de PostgreSQL 17 qui ajoute d'importantes fonctionnalités pour le JSON avec les [`JSON_TABLE`](https://doc.postgresql.fr/17/functions-json.html#FUNCTIONS-SQLJSON-TABLE), elles ne seront pas évoquées ici.

Puisque nous allons parler de JSON et de données semi-structurées, je me sens dans l'obligation de commencer cet article par un avertissement.

**Le modèle relationnel, c'est bon, mangez-en, et les contraintes d'intégrités ont été inventées pour de bonnes raisons.**

Cet article ne se veut surtout pas être une invitation à partir en mode YOLO sur la gestion des données "c'est bon ya qu'a tout mettre en JSON" (comme un vulgaire dev qui mettrait tout dans MongoDB diraient les mauvaises langues).

## Le JSON pour les débutant⸱es[#](#le-json-pour-les-debutantes "Permanent link")

Pour celles et ceux qui ne connaissent pas le [JSON](https://www.json.org/json-fr.html) il s'agit d'un format textuel de représentation des données venant du JavaScript fonctionnant en partie sur un système de `clé : valeur` qu'on peut voir comme une sorte d'évolution du XML.

`[](#__codelineno-0-1){"clé_1": "valeur", "clé_2": "valeur", "clé_3": "valeur"}`

Pas besoin de guillemets pour les nombres :

`[](#__codelineno-1-1){"nb_champignons": 42, "nb_tomates": 31, "prenom": "roger"}`

Les valeurs peuvent prendre deux formes :

- soit une valeur unique comme dans l'exemple ci-dessus,
- soit un `array`, une liste, qu'on place entre `[]`, les deux pouvant êtres combinés au sein d'un seul objet JSON.

`[](#__codelineno-2-1){"prenoms": ["elodie", "roger", "fatima"], "nb_champgnons": 42}`

Ce qu'on appelle un objet, c'est tout ce qu'il se trouve entre les `{}` qui servent à déclarer le dit objet. Pour complexifier tout ça, on peut imbriquer les objets et ainsi vous donner un exemple un peu plus parlant que de parler de tomates et de champignons :

|Exemple de JSON des superhéros|   |
|---|---|
|[1](#__codelineno-3-1)<br> [2](#__codelineno-3-2)<br> [3](#__codelineno-3-3)<br> [4](#__codelineno-3-4)<br> [5](#__codelineno-3-5)<br> [6](#__codelineno-3-6)<br> [7](#__codelineno-3-7)<br> [8](#__codelineno-3-8)<br> [9](#__codelineno-3-9)<br>[10](#__codelineno-3-10)<br>[11](#__codelineno-3-11)<br>[12](#__codelineno-3-12)<br>[13](#__codelineno-3-13)<br>[14](#__codelineno-3-14)<br>[15](#__codelineno-3-15)<br>[16](#__codelineno-3-16)<br>[17](#__codelineno-3-17)<br>[18](#__codelineno-3-18)<br>[19](#__codelineno-3-19)<br>[20](#__codelineno-3-20)<br>[21](#__codelineno-3-21)<br>[22](#__codelineno-3-22)<br>[23](#__codelineno-3-23)<br>[24](#__codelineno-3-24)<br>[25](#__codelineno-3-25)<br>[26](#__codelineno-3-26)<br>[27](#__codelineno-3-27)<br>[28](#__codelineno-3-28)<br>[29](#__codelineno-3-29)<br>[30](#__codelineno-3-30)<br>[31](#__codelineno-3-31)<br>[32](#__codelineno-3-32)<br>[33](#__codelineno-3-33)<br>[34](#__codelineno-3-34)<br>[35](#__codelineno-3-35)<br>[36](#__codelineno-3-36)<br>[37](#__codelineno-3-37)<br>[38](#__codelineno-3-38)<br>[39](#__codelineno-3-39)<br>[40](#__codelineno-3-40)<br>[41](#__codelineno-3-41)|`{   "squadName": "Super hero squad",  "homeTown": "Metro City",  "formed": 2016,  "secretBase": "Super tower",  "active": true,  "members": [    {      "name": "Molecule Man",      "age": 29,      "secretIdentity": "Dan Jukes",      "powers": [        "Radiation resistance",        "Turning tiny",        "Radiation blast"      ]    },    {      "name": "Madame Uppercut",      "age": 39,      "secretIdentity": "Jane Wilson",      "powers": [        "Million tonne punch",        "Damage resistance",        "Superhuman reflexes"      ]    },    {      "name": "Eternal Flame",      "age": 1000000,      "secretIdentity": "Unknown",      "powers": [        "Immortality",        "Heat Immunity",        "Inferno",        "Teleportation",        "Interdimensional travel"      ]    }  ] }`|

Exemple issu du site de [Mozilla](https://developer.mozilla.org/fr/docs/Learn/JavaScript/Objects/JSON) qui vous permettra d'approfondir. Vous pouvez aussi consulter l'article [wikipedia](https://fr.wikipedia.org/wiki/JavaScript_Object_Notation) ou encore [l'infâme site de la spécification](https://www.json.org/json-fr.html).

## Les types json de PostgreSQL[#](#les-types-json-de-postgresql "Permanent link")

PostgreSQL est capable de stocker les données/objets au format json dans des champs auxquels on attribuera un type dédié. Il y en a deux car sinon ça ne serait pas rigolo.

- Le type `json` qui est là pour des raisons historiques et laisser fonctionner des bases qui auraient utilisé ce type dans le passé. Il stocke les informations sous forme textuelle, ce qui est peu optimisé pour un ordinateur. Il existe toutefois un intérêt à l'utiliser : il permet de retrouver l'information sur l'ordre des clés. S'il est important pour vous de savoir que `nom` est la clef 1 et `prenom` la clef 2, sans avoir à repasser par le nom de la clef, alors il vous faudra passer par le type `json`.
    
- le type `jsonb`. Le type moderne. Il stocke les informations sous forme binaire et énormément de fonctions sont disponibles en plus de celles du type `json`.
    

## Les index[#](#les-index "Permanent link")

![icône index](https://cdn.geotribu.fr/img/logos-icones/divers/index_pointeur.webp)

Il est possible d'indexer un champ de type `json` / `jsonb` sur ses clés **de premier niveau** et cela se fait avec des index de type `GIN` :

`[](#__codelineno-4-1)CREATE INDEX idx_tb_champjson ON tb USING gin (champ_json);`

Pour indexer des valeurs plus "profondes", il faudra passer par des indexs fonctionnels, index sur des fonctions :

`[](#__codelineno-5-1)CREATE INDEX idx_tb_champjson ON tb USING gin (champ_json -> cle_ou_se_situent_les_valeur_a_indexer);`

On expliquera ce `->` par la suite.

Pour les aventuriers et aventurières, il existe une extension de PostgreSQL `btree_gin` permettant de faire des index multi-champs mixant des champs classiques et `json`. Elle est disponible nativement à l'installation de PostgreSQL et ne vous demandera pas de devenir développeur⸱se C/C++ pour l'installer (coucou les FDW parquet ! Ça va par chez vous ?).

## Création des tables[#](#creation-des-tables "Permanent link")

Je vais éviter de vous spammer du `DDL` ([Data Definition Language](https://fr.wikipedia.org/wiki/Langage_de_d%C3%A9finition_de_donn%C3%A9es), mais vous pourrez retrouver le schéma complet de la base [ici](https://github.com/thomas-szczurek/base_donnees_insee/tree/main/sql/creation_tables).

En voici cependant un schéma succinct pour aider à la compréhension du reste de l'article :

![Diagramme du modèle de la base de données](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2024/postgresql_json/modele.png)

_Ne vous préoccupez pas des tables empilées tout à droite, ce sont des partitions de la table `donnees_communes`. Le sujet ne sera pas évoqué ici et elles ne sont nécessaires pour l'exemple._

Partant d'un schéma nommé `insee`, on va créer deux tables :

- la première contiendra la liste des _bases_ disponibles, les différents volets du recensement ;
- lae seconde permettant de stocker les données.

Pour rester concentré sur le JSON, on va s'épargner 95% du modèle sous-jacent. On ne gérera donc pas ici les codes communes, etc. :

|Script de création des tables|   |
|---|---|
|[1](#__codelineno-6-1)<br> [2](#__codelineno-6-2)<br> [3](#__codelineno-6-3)<br> [4](#__codelineno-6-4)<br> [5](#__codelineno-6-5)<br> [6](#__codelineno-6-6)<br> [7](#__codelineno-6-7)<br> [8](#__codelineno-6-8)<br> [9](#__codelineno-6-9)<br>[10](#__codelineno-6-10)<br>[11](#__codelineno-6-11)<br>[12](#__codelineno-6-12)<br>[13](#__codelineno-6-13)<br>[14](#__codelineno-6-14)<br>[15](#__codelineno-6-15)<br>[16](#__codelineno-6-16)<br>[17](#__codelineno-6-17)<br>[18](#__codelineno-6-18)<br>[19](#__codelineno-6-19)<br>[20](#__codelineno-6-20)<br>[21](#__codelineno-6-21)<br>[22](#__codelineno-6-22)<br>[23](#__codelineno-6-23)<br>[24](#__codelineno-6-24)|`BEGIN; -- table listant les differentes bases Insee disponibles CREATE TABLE insee.bases ( pk_id int2 PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY, nom text NOT NULL ); -- table stockant les données par communes CREATE TABLE insee.donnees_communes ( pk_id int4 PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY, code_commune text NOT NULL CHECK length(code_commune) = 5, -- n'oublions pas les Corses qui ont des codes commune en 2A et 2B d'où le type text fk_base int2 NOT NULL, annee int2 NOT NULL, donnees jsonb NOT NULL, UNIQUE (fk_base, annee, code_commune) CONSTRAINT fk_donnees_bases FOREIGN KEY fk_base ON insee.bases(pk_id) ON UPDATE CASCADE ON DELETE CASCADE ); -- Création des index -- Un index multichamps est déjà créé automatiquement sur (fk_base, annee, code_commune) grace à la clause UNIQUE de la déclaration de table CREATE INDEX idx_donnees_com_donnees ON insee.donnees_communes USING gin (donnees); END;`|

Les plus tatillons des DBA d'entre vous auront remarqué ce `CHECK`. "Mais pourquoi qu'il utilise pas varchar(5) ? Il fait vraiment n'importe quoi !" Tout simplement, car cette forme permet d'utiliser un type de texte aux nombres de caractère **réellement** arbitraire (le type `text`) contrairement à varchar(255), tout en pouvant en contrôler le nombre minimum et maximum avec le prédicat `CHECK` (contrairement à varchar qui ne contrôle que le maximum) comme expliqué sur le [wiki Postgres](https://wiki.postgresql.org/wiki/Don%27t_Do_This#Don.27t_use_varchar.28n.29_by_default).

Et on insère quelques lignes dans notre table insee.bases :

|Script SQL d'insertion de données|   |
|---|---|
|[1](#__codelineno-7-1)<br>[2](#__codelineno-7-2)<br>[3](#__codelineno-7-3)<br>[4](#__codelineno-7-4)<br>[5](#__codelineno-7-5)<br>[6](#__codelineno-7-6)<br>[7](#__codelineno-7-7)|`INSERT INTO insee.bases (nom) VALUES ('rp_population'), ('rp_menages'), ('rp_logement'), ('rp_diplomes'), ('rp_activite'), ('rp_emploi')`|

## Insertion de données et récupération[#](#insertion-de-donnees-et-recuperation "Permanent link")

Pour passer des données textuelles / SQL vers des données encodées en JSON dans le champ dédié qui va bien, PostgreSQL dispose de [_quelques_ fonctions](https://docs.postgresql.fr/17/functions-json.html#FUNCTIONS-JSON-PROCESSING). Nous allons utiliser la fonction `jsonb_object()` qui permet de transformer un `array` sql sous forme `clef1, valeur1, clef2, valeur2 ....` en objet `jsonb` qui n'aura qu'un niveau d'imbrication. D'autres fonctions sont disponibles pour des objets plus complexes (comme `jsonb_build_object()`).

### Exemple simple[#](#exemple-simple "Permanent link")

On va créer une chaine de texte qui contiendra le contenu de notre json dont les valeurs seront séparées par des virgules `clé1, valeur1, clé2, valeur2`. Cette chaine sera passée dans une fonction `string_to_array()` la transformant en `array` avec comme séparateur des `,` pour séparer les éléments de la chaine de texte vers des éléments de liste, caractère passé en second paramètre de la fonction. Cet `array` sera ensuite envoyé dans la fonction `jsonb_object()`.

`[](#__codelineno-8-1)INSERT INTO insee.donnees_communes (code_commune, annee, fk_base, donnees) VALUES [](#__codelineno-8-2)(   [](#__codelineno-8-3)  '99999',  [](#__codelineno-8-4)  2024,  [](#__codelineno-8-5)  1,  [](#__codelineno-8-6)  jsonb_object(    [](#__codelineno-8-7)    string_to_array('tomates,42,melons,12',',')    [](#__codelineno-8-8)    ) [](#__codelineno-8-9))`

Cette requête encodera cet objet json dans le champs `données` :

`[](#__codelineno-9-1){   [](#__codelineno-9-2)  "tomate": 42,  [](#__codelineno-9-3)  "melons": 12 [](#__codelineno-9-4)}`

Maintenant comment récupérer notre nombre de melons pour le code commune 99999 en 2024 ? Celà se fait grace à des opérateurs spéciaux :

- `champ_jsonb -> 'clé'` récupère la valeur d'une clé en conservant son type json
- `champ_jsonb ->> 'clé'` fait de même en transformant la valeur en type sql "classique"

`[](#__codelineno-10-1)SELECT   [](#__codelineno-10-2)  (donnees ->> 'melons')::int4 AS nb_melons [](#__codelineno-10-3)FROM insee.donnees_communes [](#__codelineno-10-4)WHERE   [](#__codelineno-10-5)  donnees ? 'melons'  [](#__codelineno-10-6)  AND code_commune = '99999'  [](#__codelineno-10-7)  AND annee = 2024`

Vous pouvez voir que j'utilise l'opérateur `?`; uniquement valable pour les champs `jsonb` et non ceux en simple `json`. En effet, lorsque l'on requête un champ json/jsonb, un retour vous est fait pour l'ensemble des enregistrements de la table, même ceux ne contenant pas la clé. Comprendre que si votre table contient 100 000 enregistrements, mais que seuls 100 contiennent la clé "melons", ne pas spécifier cette clause WHERE vous renverrait 100 000 lignes, dont 99 900 de `NULL`.`?` est un opérateur json permettant de poser la question "la clé est-elle présente au premier niveau du champ json pour cet enregistrement ?", et on ne récupérerait que nos 100 enregistrements contenant la clé "melons".

Je précise que même si vous êtes encore ici, je suppose que vous savez déjà cela. Cependant, la forme `(quelque_chose)::(type)` est un raccourci de PostgreSQL pour faire un `cast` soit convertir une valeur dans un autre type. Avec `->>` la valeur nous est renvoyée sous forme de texte et nous la convertissons en entier.

### Avec du JSON imbriqué[#](#avec-du-json-imbrique "Permanent link")

Bon, il est bien gentil, mais là son JSON reste du JSON où tout est au premier niveau. Bah oui, pour mon besoin, cela m'a suffi. Mais, je ne vais pas fuir mes responsabilités et on va voir comment cela se passe avec du JSON plus complexe.  
Je repartirai de l'exemple de la partie précédente pour complexifier après cet aparté.

Pour injecter du json complexe dans un champ, deux solutions s'offrent à nous : imbriquer les fonctions dédiées, ou caster une chaine de texte. Imaginons une table "test", dans un schema "test" et dont un champ `jsonb` se nomme "donnees".

`[](#__codelineno-11-1)INSERT INTO test.test (donnees) VALUES [](#__codelineno-11-2)(jsonb_object(  [](#__codelineno-11-3) 'cle1', 'valeur1', [](#__codelineno-11-4) 'cle2', jsonb_array(  [](#__codelineno-11-5)  'foo', 'bar', 'baz')  [](#__codelineno-11-6)  ) [](#__codelineno-11-7) )`

Cette insertion pourrait tout aussi bien s'écrire avec un cast d'une chaine de texte vers du jsonb. Attention, la syntaxe json doit être ici respectée :

`[](#__codelineno-12-1)INSERT INTO test.test (donnees) VALUES [](#__codelineno-12-2)('{"cle1": "valeur1", "cle2": ["foo", "bar", "baz"]}'::jsonb)`

#### Recupération[#](#recuperation "Permanent link")

Pour récupérer une valeur, on utilise la fonction `jsonb_path_query()` qui possède deux paramètres : `le nom du champ` contenant les données json, et le `json_path` vers la valeur à atteindre. Imaginons que nous voulions récupérer la deuxième valeur de la liste contenue dans "cle2" :

`[](#__codelineno-13-1)SELECT  [](#__codelineno-13-2) jsonb_path_query(donnees, '$.cle2[1]') [](#__codelineno-13-3)FROM test.test`

Le `$` désigne le début du chemin json retourné. Nous faisons suivre ce premier symbole par un point pour passer à l'objet suivant. Puis, par le nom de clé suivant et ainsi de suite jusqu'à la clé recherchée, à laquelle nous collons un `[1]` pour la 2ᵉ valeur de la liste (les valeurs commencent à 0). Pour plus d'informations sur les `json_path`, vous pouvez consulter la [documentation](https://www.postgresql.org/docs/current/functions-json.html#FUNCTIONS-SQLJSON-PATH).

## Attaquons nous au recensement[#](#attaquons-nous-au-recensement "Permanent link")

![logo INSEE](https://cdn.geotribu.fr/img/logos-icones/entreprises_association/INSEE.svg)

Bien, maintenant que nous avons essayé d'expliquer tant bien que mal les concepts avec des exemples simples, car il faut bien commencer quelque part, essayons avec quelque chose d'un peu plus volumineux.

Récupérons le dernier millésime du volet population du recensement communal [au format csv sur le site de l'Insee](https://www.insee.fr/fr/information/8183122) (vous voulez les bases des principaux indicateurs). Pour l'exemple, on utilisera le fichier "Evolution et structure de la population". Il faut tout d'abord nettoyer les noms de champ. En effet, l'Insee indique le millésime systématiquement dans ses noms de champ, ce qui fait que ces derniers changent tous les ans pour un même indicateur.

Les noms de champ commencent tous par P ou C, ceci indique _exploitation principale_ (réponses brutes aux questions du recensement) ou _exploitation complémentaire_ (croisement de réponses pour établir un indicateur). Les champs provenant de l'exploitation principale et ceux issus de la complémentaire ne doivent pas être croisés entre eux. Cette information est évidemment à conserver, mais par choix personnel, je préfère la mettre à la fin plutôt qu'au début du nom. On passera ainsi de champs normés comme ceci : `P18_POP` vers une normalisation de ce type `POP_P`.

Vous trouverez [ici](https://github.com/thomas-szczurek/base_donnees_insee/blob/main/sql/import/correction_champs_insee.xlsx) un tableur dont le rôle est de s'occuper de tout ça.

Avant d'insérer les données dans notre table, nous allons passer par une table temporaire afin de rendre les données accessibles dans Postgres. Utiliser `COPY` de Postgresql serait fastidieux car il faudrait indiquer la centaine de champs que contient le volet population du recensement dans la commande. Et, je n'ai pas honte de dire que j'ai un baobab dans la main à cette idée. Nous sortons donc ce merveilleux logiciel qu'est QGIS. On active les panneaux Explorateur et Explorateur2. On crée une connexion vers la base avec les droits de création, et d'un mouvement gracile du poignet, vous glissez le fichier depuis le panneau Explorateur vers la base Postgres dans l'Explorateur2. Laissez la magie opérer.

Maintenant, préparez-vous pour peut-être un des INSERT les plus bizarres de votre vie (en tout cas, ça l'a été pour moi !). Arf. Je me rends compte que si je voulais bien faire, il faudrait aussi que j'explique les [CTE](https://www.postgresql.org/docs/current/queries-with.html). Mais pour ne pas trop alourdir, je vous laisser cliquer sur le lien.

On va utiliser la CTE pour concaténer le nom que l'on veut donner à nos clés avec les valeurs contenues dans notre table temporaire dans une chaine séparée par des `,`. On l'enverra dans une fonction `string_to_array()` puis dans une fonction `jsonb_object()`. On en profitera pour au passage, nettoyer toute tabulation ou retour chariot qui pourrait subsister avec une expression régulière grâce à la fonction `regex_replace()`. (ces caractères se notent `\t`, `\n` et `\r`). Cette dernière fonction prend 3 arguments : la chaine de caractères source, le `pattern` recherché, le texte de remplacement. On y ajoute le _drapeau_ optionnel `g` afin de remplacer toutes les occurrences trouvées.

Notez que si votre table temporaire possède un nom différent de "rp_population_import" il vous faudra modifier la clause FROM de la CTE.

|L'INSERT le plus bizarre de votre vie (avec une CTE)|   |
|---|---|
|[1](#__codelineno-14-1)<br>  [2](#__codelineno-14-2)<br>  [3](#__codelineno-14-3)<br>  [4](#__codelineno-14-4)<br>  [5](#__codelineno-14-5)<br>  [6](#__codelineno-14-6)<br>  [7](#__codelineno-14-7)<br>  [8](#__codelineno-14-8)<br>  [9](#__codelineno-14-9)<br> [10](#__codelineno-14-10)<br> [11](#__codelineno-14-11)<br> [12](#__codelineno-14-12)<br> [13](#__codelineno-14-13)<br> [14](#__codelineno-14-14)<br> [15](#__codelineno-14-15)<br> [16](#__codelineno-14-16)<br> [17](#__codelineno-14-17)<br> [18](#__codelineno-14-18)<br> [19](#__codelineno-14-19)<br> [20](#__codelineno-14-20)<br> [21](#__codelineno-14-21)<br> [22](#__codelineno-14-22)<br> [23](#__codelineno-14-23)<br> [24](#__codelineno-14-24)<br> [25](#__codelineno-14-25)<br> [26](#__codelineno-14-26)<br> [27](#__codelineno-14-27)<br> [28](#__codelineno-14-28)<br> [29](#__codelineno-14-29)<br> [30](#__codelineno-14-30)<br> [31](#__codelineno-14-31)<br> [32](#__codelineno-14-32)<br> [33](#__codelineno-14-33)<br> [34](#__codelineno-14-34)<br> [35](#__codelineno-14-35)<br> [36](#__codelineno-14-36)<br> [37](#__codelineno-14-37)<br> [38](#__codelineno-14-38)<br> [39](#__codelineno-14-39)<br> [40](#__codelineno-14-40)<br> [41](#__codelineno-14-41)<br> [42](#__codelineno-14-42)<br> [43](#__codelineno-14-43)<br> [44](#__codelineno-14-44)<br> [45](#__codelineno-14-45)<br> [46](#__codelineno-14-46)<br> [47](#__codelineno-14-47)<br> [48](#__codelineno-14-48)<br> [49](#__codelineno-14-49)<br> [50](#__codelineno-14-50)<br> [51](#__codelineno-14-51)<br> [52](#__codelineno-14-52)<br> [53](#__codelineno-14-53)<br> [54](#__codelineno-14-54)<br> [55](#__codelineno-14-55)<br> [56](#__codelineno-14-56)<br> [57](#__codelineno-14-57)<br> [58](#__codelineno-14-58)<br> [59](#__codelineno-14-59)<br> [60](#__codelineno-14-60)<br> [61](#__codelineno-14-61)<br> [62](#__codelineno-14-62)<br> [63](#__codelineno-14-63)<br> [64](#__codelineno-14-64)<br> [65](#__codelineno-14-65)<br> [66](#__codelineno-14-66)<br> [67](#__codelineno-14-67)<br> [68](#__codelineno-14-68)<br> [69](#__codelineno-14-69)<br> [70](#__codelineno-14-70)<br> [71](#__codelineno-14-71)<br> [72](#__codelineno-14-72)<br> [73](#__codelineno-14-73)<br> [74](#__codelineno-14-74)<br> [75](#__codelineno-14-75)<br> [76](#__codelineno-14-76)<br> [77](#__codelineno-14-77)<br> [78](#__codelineno-14-78)<br> [79](#__codelineno-14-79)<br> [80](#__codelineno-14-80)<br> [81](#__codelineno-14-81)<br> [82](#__codelineno-14-82)<br> [83](#__codelineno-14-83)<br> [84](#__codelineno-14-84)<br> [85](#__codelineno-14-85)<br> [86](#__codelineno-14-86)<br> [87](#__codelineno-14-87)<br> [88](#__codelineno-14-88)<br> [89](#__codelineno-14-89)<br> [90](#__codelineno-14-90)<br> [91](#__codelineno-14-91)<br> [92](#__codelineno-14-92)<br> [93](#__codelineno-14-93)<br> [94](#__codelineno-14-94)<br> [95](#__codelineno-14-95)<br> [96](#__codelineno-14-96)<br> [97](#__codelineno-14-97)<br> [98](#__codelineno-14-98)<br> [99](#__codelineno-14-99)<br>[100](#__codelineno-14-100)<br>[101](#__codelineno-14-101)<br>[102](#__codelineno-14-102)<br>[103](#__codelineno-14-103)<br>[104](#__codelineno-14-104)<br>[105](#__codelineno-14-105)<br>[106](#__codelineno-14-106)<br>[107](#__codelineno-14-107)<br>[108](#__codelineno-14-108)<br>[109](#__codelineno-14-109)<br>[110](#__codelineno-14-110)<br>[111](#__codelineno-14-111)<br>[112](#__codelineno-14-112)<br>[113](#__codelineno-14-113)<br>[114](#__codelineno-14-114)<br>[115](#__codelineno-14-115)<br>[116](#__codelineno-14-116)<br>[117](#__codelineno-14-117)<br>[118](#__codelineno-14-118)<br>[119](#__codelineno-14-119)<br>[120](#__codelineno-14-120)<br>[121](#__codelineno-14-121)|`-- cte concatenant les données avec les clés et nettoyant les caractères spéciaux. WITH d AS (   SELECT CODGEO, regexp_replace('pop_p,' \| POP_P \| ',   pop_0_14_ans_p,' \| POP0014_P \| ',   pop_15_29_ans_p,' \| POP1529_P \| ',   pop_30_44_ans_p,' \| POP3044_P \| ',   pop_45_59_ans_p,' \| POP4559_P \| ',   pop_60_74_ans_p,' \| POP6074_P \| ',   pop_75_89_ans_p,' \| POP7589_P \| ',   pop_90_ans_plus_p,' \| POP90P_P \| ',   hommes_p,' \| POPH_P \| ',   hommes_0_14_ans_p,' \| H0014_P \| ',   hommes_15_29_ans_p,' \| H1529_P \| ',   hommes_30_44_ans_p,' \| H3044_P \| ',   hommes_45_59_ans_p,' \| H4559_P \| ',   hommes_60_74_ans_p,' \| H6074_P \| ',   hommes_75_89_ans_p,' \| H7589_P \| ',   hommes_90_ans_plus_p,' \| H90P_P \| ',   hommes_0_19_ans_p,' \| H0019_P \| ',   hommes_20_64_ans_p,' \| H2064_P \| ',   hommes_65_ans_plus_p,' \| H65P_P \| ',   femmes_p,' \| POPF_P \| ',   femmes_0_14_ans_p,' \| F0014_P \| ',   femmes_15_29_ans_p,' \| F1529_P \| ',   femmes_30_44_ans_p,' \| F3044_P \| ',   femmes_45_59_ans_p,' \| F4559_P \| ',   femmes_60_74_ans_p,' \| F6074_P \| ',   femmes_75_89_ans_p,' \| F7589_P \| ',   femmes_90_ans_plus_p,' \| F90P_P \| ',   femmes_0_19_ans_p,' \| F0019_P \| ',   femmes_20_64_ans_p,' \| F2064_P \| ',   femmes_65_ans_plus_p,' \| F65P_P \| ',   pop_1an_ou_plus_localisee_1an_auparavant_p,' \| POP01P_P \| ',   pop_1an_ou_plus_meme_logement_1an_auparavant_p,' \| POP01P_IRAN1_P \| ',   pop_1an_ou_plus_meme_commune_1an_auparavant_p,' \| POP01P_IRAN2_P \| ',   pop_1an_ou_plus_meme_departement_1an_auparavant_p,' \| POP01P_IRAN3_P \| ',   pop_1an_ou_plus_meme_region_1an_auparavant_p,' \| POP01P_IRAN4_P \| ',   pop_1an_ou_plus_autre_region_1an_auparavant_p,' \| POP01P_IRAN5_P \| ',   pop_1an_ou_plus_un_dom_1an_auparavant_p,' \| POP01P_IRAN6_P \| ',   pop_1an_ou_plus_hors_metropole_ou_dom_1an_auparavant_p,' \| POP01P_IRAN7_P \| ',   pop_1_14ans_autre_logement_1an_auparavant_p,' \| POP0114_IRAN2P_P \| ',   pop_1_14ans_meme_commune_1an_auparavant_p,' \| POP0114_IRAN2_P \| ',   pop_1_14ans_autre_commune_1an_auparavant_p,' \| POP0114_IRAN3P_P \| ',   pop_15_24ans_autre_logement_1an_auparavant_p,' \| POP1524_IRAN2P_P \| ',   pop_15_24ans_meme_commune_1an_auparavant_p,' \| POP1524_IRAN2_P \| ',   pop_15_24ans_autre_commune_1an_auparavant_p,' \| POP1524_IRAN3P_P \| ',   pop_25_54ans_autre_logement_1an_auparavant_p,' \| POP2554_IRAN2P_P \| ',   pop_25_54ans_meme_commune_1an_auparavant_p,' \| POP2554_IRAN2_P \| ',   pop_25_54ans_autre_commune_1an_auparavant_p,' \| POP2554_IRAN3P_P \| ',   pop_55_ou_plus_autre_logement_1an_auparavant_p,' \| POP55P_IRAN2P_P \| ',   pop_55_ou_plus_meme_commune_1an_auparavant_p,' \| POP55P_IRAN2_P \| ',   pop_55_ou_plus_autre_commune_1an_auparavant_p,' \| POP55P_IRAN3P_P \| ',   pop_15_ans_plus_c,' \| POP15P_C \| ',   agriculteurs_15_ans_plus_c,' \| POP15P_CS1_C \| ',   artisants_commercants_chefs_entreprise_15_ans_plus_c,' \| POP15P_CS2_C \| ',   cadres_prof_intel_sup_15_ans_plus_c,' \| POP15P_CS3_C \| ',   professions_intermediaires_15_ans_plus_c,' \| POP15P_CS4_C \| ',   employes_15_ans_plus_c,' \| POP15P_CS5_C \| ',   ouvriers_15_ans_plus_c,' \| POP15P_CS6_C \| ',   retraites_15_ans_plus_c,' \| POP15P_CS7_C \| ',   autres_15_ans_plus_c,' \| POP15P_CS8_C \| ',   hommes_15_ans_plus_c,' \| H15P_C \| ',   h_agriculteurs_15_ans_plus_c,' \| H15P_CS1_C \| ',   h_artisants_commercants_chefs_entreprise_15_ans_plus_c,' \| H15P_CS2_C \| ',   h_cadres_prof_intel_sup_15_ans_plus_c,' \| H15P_CS3_C \| ',   h_professions_intermediaires_15_ans_plus_c,' \| H15P_CS4_C \| ',   h_employes_15_ans_plus_c,' \| H15P_CS5_C \| ',   h_ouvriers_15_ans_plus_c,' \| H15P_CS6_C \| ',   h_retraites_15_ans_plus_c,' \| H15P_CS7_C \| ',   h_autres_15_ans_plus_c,' \| H15P_CS8_C \| ',   femmes_15_ans_plus_c,' \| F15P_C \| ',   f_agricultrices_15_ans_plus_c,' \| F15P_CS1_C \| ',   f_artisanes_commercantes_cheffes_entreprise_15_ans_plus_c,' \| F15P_CS2_C \| ',   f_cadres_prof_intel_sup_15_ans_plus_c,' \| F15P_CS3_C \| ',   f_professions_intermediaires_15_ans_plus_c,' \| F15P_CS4_C \| ',   f_employees_15_ans_plus_c,' \| F15P_CS5_C \| ',   f_ouvrieres_15_ans_plus_c,' \| F15P_CS6_C \| ',   f_retraitees_15_ans_plus_c,' \| F15P_CS7_C \| ',   f_autres_15_ans_plus_c,' \| F15P_CS8_C \| ',   population_15_24_ans_c,' \| POP1524_C \| ',   pop_15_24_ans_agriculteurs_c,' \| POP1524_CS1_C \| ',   pop_15_24_ans_artisants_commercants_chefs_entreprise_c,' \| POP1524_CS2_C \| ',   pop_15_24_ans_cadres_prof_intel_sup_c,' \| POP1524_CS3_C \| ',   pop_15_24_ans_professions_intermediaires_c,' \| POP1524_CS4_C \| ',   pop_15_24_ans_employes_c,' \| POP1524_CS5_C \| ',   pop_15_24_ans_ouvriers_c,' \| POP1524_CS6_C \| ',   pop_15_24_ans_retraites_c,' \| POP1524_CS7_C \| ',   pop_15_24_ans_autres_c,' \| POP1524_CS8_C \| ',   population_25_54_ans_c,' \| POP2554_C \| ',   pop_25_54_ans_agriculteurs_c,' \| POP2554_CS1_C \| ',   pop_25_54_ans_artisants_commercants_chefs_entreprise_c,' \| POP2554_CS2_C \| ',   pop_25_54_ans_cadres_prof_intel_sup_c,' \| POP2554_CS3_C \| ',   pop_25_54_ans_professions_intermediaires_c,' \| POP2554_CS4_C \| ',   pop_25_54_ans_employes_c,' \| POP2554_CS5_C \| ',   pop_25_54_ans_ouvriers_c,' \| POP2554_CS6_C \| ',   pop_25_54_ans_retraites_c,' \| POP2554_CS7_C \| ',   pop_25_54_ans_autres_c,' \| POP554_CS8_C \| ',   population_55_ans_et_plus_c,' \| POP55P_C \| ',   pop_55_ans_et_plus_ans_agriculteurs_c,' \| POP55P_CS1_C \| ',   pop_55_ans_et_plus_ans_artisants_commercants_chefs_entreprise_c,' \| POP55P_CS2_C \| ',   pop_55_ans_et_plus_ans_cadres_prof_intel_sup_c,' \| POP55P_CS3_C \| ',   pop_55_ans_et_plus_ans_professions_intermediaires_c,' \| POP55P_CS4_C \| ',   pop_55_ans_et_plus_ans_employes_c,' \| POP55P_CS5_C \| ',   pop_55_ans_et_plus_ans_ouvriers_c,' \| POP55P_CS6_C \| ',   pop_55_ans_et_plus_ans_retraites_c,' \| POP55P_CS7_C \| ',   pop_55_ans_et_plus_ans_autres_c,' \| POP55P_CS8_C,  E'[\t\n\r]','','g') AS data  FROM insee.rp_population_import ) -- Conversion des chaines de texte en json et insertion dans la table INSERT INTO insee.donnees_communes("code_commune","annee","fk_base","donnees") SELECT     CODGEO,    2021,    1,    jsonb_object(string_to_array(data::text,',')) FROM d ORDER BY "CODGEO";`|

Ouf.

![Aperçu de la table donnees_communes après insertion des données](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2024/postgresql_json/donnees_communes.png)

On veillera à bien grouper les inserts dans cet ordre annee/base/code_commune afin de faciliter la lecture des données par PostgreSQL.

Maintenant, imaginez que comme l'auteur de ces lignes, vos gros doigts boudinnés cafouillent et glissent sur les touches lors de la rédaction de cette requête, qu'une faute de frappe s'y glisse, et là c'est le drame. Comment modifier le nom d'une clé déjà encodée dans la table ? Avec cette astuce :

`[](#__codelineno-15-1)CREATE TABLE example(id int PRIMARY KEY, champ jsonb); [](#__codelineno-15-2)INSERT INTO example VALUES     [](#__codelineno-15-3)    (1, '{"nme": "test"}'),    [](#__codelineno-15-4)    (2, '{"nme": "second test"}'); [](#__codelineno-15-5)[](#__codelineno-15-6)UPDATE example [](#__codelineno-15-7)SET champ = champ - 'nme' || jsonb_build_object('name', champ -> 'nme') [](#__codelineno-15-8)WHERE champ ? 'nme' [](#__codelineno-15-9)returning *;`

`-` est un opérateur qui permet de supprimer une clé d'un objet json. Pour notre UPDATE on enlève donc notre faute de frappe de l'ensemble de l'objet. De plus, on concatène tout le reste avec la construction d'un nouvel objet où l'on corrige le nom de la clé. On assigne également la valeur de la clé en train d'être supprimée, elle est toujours utilisable au moment de l'UPDATE, sur les champs qui la contiennent à l'origine avec `?`.

## Et maintenant ? Qu'est-ce qu'on fait de ça ?[#](#et-maintenant-quest-ce-quon-fait-de-ca "Permanent link")

Jusqu'à maintenant, nous n'avons travaillé qu'avec le volet population pour son dernier millésime. Imaginons maintenant que nous répétions l'exercice pour les 6 volets et sur plusieurs millésimes, et ce, en sachant qu'au cours du temps, certains champs peuvent apparaitre ou disparaitre ; changement dans les niveaux de diplômes observés par exemple. Il serait intéressant de récupérer une table indiquant la première et la dernière année de présence de chaque clé. Mettons que lors de ces travaux, nous en avons profité pour mettre à jour une table "correspondance_clefs_champs" listant chaque clé présentes et son nom INSEE d'origine (tout du moins, celui que nous avions normalisé).

`[](#__codelineno-16-1)CREATE MATERIALIZED VIEW insee.presence_clefs_annees AS [](#__codelineno-16-2)SELECT  [](#__codelineno-16-3) c.pk_id AS pk_id, [](#__codelineno-16-4) c.clef_json AS clef_json, [](#__codelineno-16-5) c.fk_base AS fk_base, [](#__codelineno-16-6) a.premiere AS premiere_annee_presence, [](#__codelineno-16-7) a.derniere AS derniere_annee_presence [](#__codelineno-16-8)FROM insee.correspondance_clefs_champs AS c,  [](#__codelineno-16-9) LATERAL (SELECT     [](#__codelineno-16-10)     min(annee) AS premiere,     [](#__codelineno-16-11)     max(annee) AS derniere     [](#__codelineno-16-12)     FROM insee.donnees_communes WHERE fk_base = c.fk_base AND donnees ? c.clef_json) AS a [](#__codelineno-16-13)ORDER BY fk_base, clef_json;`

![Apercu de la vm avec la première et dernière année de présence des cles](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2024/postgresql_json/vm_presence_cles_annees.png)

La seule difficulté ici est la présence de [LATERAL](https://docs.postgresql.fr/17/queries-table-expressions.html#QUERIES-FROM). Ce « mot » permet d'utiliser un champ de la requête principale dans une sous-requête placée dans une clause FROM. Les éléments de la sous-requête seront ensuite évalués atomiquement avant d'être joints à la table d'origine. Oui, ce n'est pas très facile à expliquer. Ici, le WHERE de la sous-requête va interroger le champ `donnees` de la table "donnees_communes" pour voir s'il y voit la clef_json en train d'être évaluée dans la table "correspondance_clefs_champs" possédant l'alias "c". Si oui, alors on prend la valeur minimum/maximum du champ année et on le joint à cette ligne et uniquement à cette ligne. Puis évaluation de clef_json suivante ... (_évaluée atomiquement_)

Maintenant, on voudrait proposer un produit un peu plus simple d'utilisation avec tout ça. Excusez-moi d'avance, je copierai une requête de 50 lignes une seconde fois.  
La seule table utilisée ici que nous n'avons pas créée est une table `zonages_administratifs` comprenant les codes communes dans un champ `code_admin` et un champ `fk_type` contenant le type de zonage administratif (1 pour les communes).

|Création de la vue matérialisée agrémentée|   |
|---|---|
|[1](#__codelineno-17-1)<br> [2](#__codelineno-17-2)<br> [3](#__codelineno-17-3)<br> [4](#__codelineno-17-4)<br> [5](#__codelineno-17-5)<br> [6](#__codelineno-17-6)<br> [7](#__codelineno-17-7)<br> [8](#__codelineno-17-8)<br> [9](#__codelineno-17-9)<br>[10](#__codelineno-17-10)<br>[11](#__codelineno-17-11)<br>[12](#__codelineno-17-12)<br>[13](#__codelineno-17-13)<br>[14](#__codelineno-17-14)<br>[15](#__codelineno-17-15)<br>[16](#__codelineno-17-16)<br>[17](#__codelineno-17-17)<br>[18](#__codelineno-17-18)<br>[19](#__codelineno-17-19)<br>[20](#__codelineno-17-20)<br>[21](#__codelineno-17-21)<br>[22](#__codelineno-17-22)<br>[23](#__codelineno-17-23)<br>[24](#__codelineno-17-24)<br>[25](#__codelineno-17-25)<br>[26](#__codelineno-17-26)<br>[27](#__codelineno-17-27)<br>[28](#__codelineno-17-28)<br>[29](#__codelineno-17-29)<br>[30](#__codelineno-17-30)<br>[31](#__codelineno-17-31)<br>[32](#__codelineno-17-32)<br>[33](#__codelineno-17-33)<br>[34](#__codelineno-17-34)<br>[35](#__codelineno-17-35)<br>[36](#__codelineno-17-36)<br>[37](#__codelineno-17-37)<br>[38](#__codelineno-17-38)<br>[39](#__codelineno-17-39)<br>[40](#__codelineno-17-40)<br>[41](#__codelineno-17-41)<br>[42](#__codelineno-17-42)<br>[43](#__codelineno-17-43)<br>[44](#__codelineno-17-44)|`CREATE MATERIALIZED VIEW insee.donnees_communes_olap AS WITH -- Sélection des codes communes  codes_com AS (  SELECT   code_admin  FROM insee.zonages_administratifs  WHERE fk_type = 1 ), -- Sélections des clefs et unnest par année  clefs AS (  SELECT   pk_id,   generate_series(premiere_annee_presence, derniere_annee_presence,1) AS annee  FROM insee.presence_clefs_annees ), -- cross join des clefs + année unnestées et des codes communaux  tc AS ( SELECT  cc.code_admin AS code_com,  cl.annee AS annee,  cl.pk_id AS pk_id FROM codes_com AS cc CROSS JOIN clefs AS cl ), -- Selection finale avec récupération des données  final AS ( SELECT  tc.code_com,  tc.annee,  co.fk_base,  co.clef_json,  CASE   WHEN (d.donnees ->> clef_json) IN ('','null','s','nd') THEN NULL   ELSE ((d.donnees ->> clef_json)::real)  END AS valeur FROM tc JOIN  insee.donnees_communes AS d ON (tc.code_com = d.code_commune AND tc.annee = d.annee) LEFT JOIN  insee.presence_clefs_annees AS co ON tc.pk_id = co.pk_id ORDER BY tc.annee, co.fk_base, co.clef_json, tc.code_com ) SELECT * FROM final WHERE valeur IS NOT NULL;`|

![aperçu de la vm avec les données organisées façon "olap"](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2024/postgresql_json/vm_donnees_olap.png)

Attention, la création de cette vue matérialisée ou son rafraîchissement peut prendre un certain temps si vous avez stocké beaucoup de données (1 heure chez moi pour les 6 volets de 2015 à 2021).

Enfin, histoire de vivre avec son temps et non comme un vieil ours des cavernes, on va convertir cette vue matérialisée en fichier [parquet](https://parquet.apache.org/). Et, pour ça, on va utiliser GDAL qui est décidément incroyable.

Export vue au format parquet

`[](#__codelineno-18-1)ogr2ogr -of parquet donnees_insee.parquet PG:"dbname='insee' schema='insee' tables='donnees_communes_olap' user='nom_utilisateur' password='votre_mot_de_passe'"`

Et on peut ainsi mettre le fichier sur un espace cloud, comme [ici](https://donnees-insee.s3.fr-par.scw.cloud/donnees_insee_olap.parquet) ! Vous pouvez ensuite sortir votre plus beau générateur de publications Linkedin qui mettra plein d'emojis choupi et faire le cake sur les rezos (imaginez que 90% du contenu de Linkedin doit être fait avec ces trucs qui sont capable de vous générer des publications expliquant que l'un des rares avantages du shape sur le geopackage est d'être un format multi-fichiers, le tout sur un ton très assuré).

## Auteur·ice[#](#auteurice "Permanent link")

### [Thomas SZCZUREK-GAYANT](app://obsidian.md/team/thomas-szczurek-gayant/ "Voir la page complète de l'auteur·ice avec la liste de ses articles")[#](#thomas-szczurek-gayant "Permanent link")

![Portrait Thomas Szczurek-Gayant](https://cdn.geotribu.fr/img/articles-blog-rdp/articles/2024/gdal_qgis_blender/photo_id_tsg.png)

Géomaticien, on a installé un jour QGIS sur mon ordinateur alors que je ne l'avais pas ouvert depuis 2008 et j'ai compris que ma vision de l'open source était dépassée de plus de 10 ans. On peut me retrouver maintenant errant dans les rues en hurlant "passez sous Linux" aux inconnus. Sinon j'aime bien l'astronomie et les chats.

## Licence [#](#license "Permanent link")

Ce contenu est sous licence [Creative Commons International 4.0 BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.fr), avec attribution et partage dans les mêmes conditions, sauf dans le cadre d'une utilisation commerciale.  
Les médias d'illustration sont potentiellement soumis à d'autres conditions d'utilisation.

Réutiliser, citer l'article

Vous êtes autorisé(e) à :

- **Partager** : copier, distribuer et communiquer le matériel par tous moyens et sous tous formats
- **Adapter** : remixer, transformer et créer à partir du matériel pour toute utilisation, exceptée commerciale.

Citer cet article :

> "_Travailler avec du JSON et PostgreSQL_" publié par Thomas SZCZUREK-GAYANT sur **Geotribu** sous [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.fr) - Source : [https://geotribu.fr/articles/2025/2025-01-21_travailler-avec-JSON-et-PostgreSQL/](https://geotribu.fr/articles/2025/2025-01-21_travailler-avec-JSON-et-PostgreSQL/)

Afin de favoriser les échanges constructifs, merci de préférer le [**pseudonymat**](https://fr.wikipedia.org/wiki/Pseudonymat "Page Wikipédia détaillant le pseudonymat") à l'anonymat. Pour rappel, l'adresse mail n'est pas exposée publiquement et sert principalement aux notifications de réponse. Les commentaires sont automatiquement republiés sur nos réseaux sociaux pour favoriser la discussion. Consulter la [page sur la confidentialité et les données personnelles](https://geotribu.fr/about/confidentialite/#commentaires "Confidentialité - Geotribu").  
Une version minimale de la [syntaxe markdown](https://docs.framasoft.org/fr/grav/markdown.html) est acceptée pour la mise en forme des commentaires.  
Propulsé par [Isso](https://posativ.org/isso/).

---