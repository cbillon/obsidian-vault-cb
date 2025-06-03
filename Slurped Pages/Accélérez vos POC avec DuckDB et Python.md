---
link: https://blog.ippon.fr/2025/03/10/accelerez-vos-poc-avec-duckdb-et-python/
byline: Jérome Liermann
site: Blog Ippon - Ippon Technologies
date: 2025-03-10T10:09
excerpt: |-
  Depuis sa première release majeure en mai 2024, DuckDB fait beaucoup parler de lui, au moyen de nombreuses punchlines ésotériques telles que :

  DuckDB is the SQLite of OLAP1
  Ducks are amazing animals. They can fly, walk and swim.2

  Reconnaissons à leurs community manager un pragmatisme certain, les grandes lignes du projet étant clairement dessinées au travers de ces accroches. Pour étayer le propos, DuckDB c'est :

   * un projet créé en 2019 par deux chercheurs néerlandais Mark Raasveldt et Hann
twitter: https://twitter.com/@ippontech
tags:
  - duckdb
  - python
slurped: 2025-05-17T09:17
title: Accélérez vos POC avec DuckDB et Python
---

Depuis sa première release majeure en mai 2024, DuckDB fait beaucoup parler de lui, au moyen de nombreuses punchlines ésotériques telles que :

> _DuckDB is the SQLite of OLAP_[_1_](https://www.cloudraft.io/blog/clickhouse-vs-duckdb?ref=blog.ippon.fr)  
> _Ducks are amazing animals. They can fly, walk and swim._[_2_](https://duckdb.org/faq?ref=blog.ippon.fr)

Reconnaissons à leurs community manager un pragmatisme certain, les grandes lignes du projet étant clairement dessinées au travers de ces accroches. Pour étayer le propos, DuckDB c'est :

- un projet créé en 2019 par deux chercheurs néerlandais [Mark Raasveldt](https://mytherin.github.io/?ref=blog.ippon.fr) et [Hannes Mühleisen](https://hannes.muehleisen.org/?ref=blog.ippon.fr)
- entièrement gratuit et **open source**
- comme SQLite, une base de données serverless, avec une persistance en mode mono-fichier ou in-memory
- Contrairement à SQLite, une base **OLAP**, orientée colonne et optimisée pour l'analytique

Après ces nombreuses comparaisons avec SQLite, j'aurais pu me lancer dans un grand comparatif entre DuckDB et SQLite, benchmark à l'appui, pour vous expliquer dans quel cas utiliser l'un ou l'autre. En vérité, je ne pense pas que le débat se situe ici. Dans le monde des bases "légères" plus qu'ailleurs, la prise en main, la facilité d'implémentation est un atout majeur.

Laissez-moi vous relater mes récentes expériences avec DuckDB en tant que support de stockage pour le développement de POC en Python.

## L'API Python

### La connexion

DuckDB partage avec SQLite sa connexion à une base de données extrêmement simple. Il suffit de lui fournir le chemin du fichier où stocker ses données. Il se chargera de le créer s'il n'existe pas. Alternativement, et cette fois encore en reprenant la convention SQLite, créer une connexion avec le paramètre :memory: ou sans paramètre créera un espace de stockage temporaire en RAM. Ensuite, tout est prêt pour exécuter vos requêtes favorites.

```
import duckdb

conn = duckdb.connect("database_file.db") 
# duckdb.connect(":memory:") / duckdb.connect()
conn.sql("CREATE TABLE dummy AS (SELECT 42 AS 'mycol')")
conn.sql("SELECT * FROM dummy")
```

Malgré sa simplicité, DuckDB reste cependant un système de base de données complet. Un certain nombre de configurations supplémentaires permet par exemple une gestion :

- des droits d'accès : lecture seule, restriction, ...
- de ses comportements par défaut : timezone, valeurs nulles, ...
- de sa performance : threads, RAM min et max, ...

La documentation exhaustive des paramètres de configuration est disponible [ici](https://duckdb.org/docs/configuration/overview?ref=blog.ippon.fr).

La syntaxe SQL utilisée est décrite comme un surensemble de PostgreSQL, moyennant quelques exceptions[3](https://duckdb.org/docs/sql/introduction?ref=blog.ippon.fr). Si DuckDB tire sa force de sa compatibilité avec PostgreSQL, c'est au travers de ses extensions au langage SQL et de son intégration transparente avec les bibliothèques de traitement de données en Python qu'il brille réellement.

### Une API fluide et intégrée à vos outils de traitement de données

Revenons-en rapidement au POC dont j'ai promis de vous parler.Pour recontextualiser mes objectifs, il s'agissait essentiellement de valider la faisabilité de certaines transformations in-memory. Pour me rapprocher du cadre de production, sourcé et persisté sur [AWS Redshift](https://aws.amazon.com/fr/redshift/?ref=blog.ippon.fr), il me fallait mettre en place une base de données de démonstration. Puis construire deux interfaces de lecture et écriture, comme ceci :

![](http://blog.ippon.fr/content/images/2025/03/img1.png)

Deux possibilités se sont offertes à moi :

1. Monter un environnement Redshift sur un compte AWS de dev. Pour établir la connexion sans trop souffrir, cela implique au moins deux bibliothèques supplémentaires (sqlalchemy-redshift et psycopg2[4](https://github.com/sqlalchemy-redshift/sqlalchemy-redshift?ref=blog.ippon.fr)). Sans compter l'éventuelle gestion IAM sur AWS.
2. Établir une base de données mockée le temps du POC, grâce à DuckDB.

Pour des raisons narratives évidentes, j'ai fait le choix de mocker ma base de données. Pour faciliter son intégration, duckDB fournit d'une part des fonctions de conversions standard, depuis des requêtes SQL vers tous nos outils de traitement de données préférés[5](https://duckdb.org/docs/guides/python/install?ref=blog.ippon.fr). Parmi elles :

- polars : `.pl()`
- pandas : `.df()`
- numpy : `.fetchnumpy()`
- arrow : `.arrow()`

```
import duckdb

conn = duckdb.connect()
result = conn.sql("SELECT 1 AS 'a', 2 AS 'b', 3 AS 'c'").pl()

print(result)
```

![](http://blog.ippon.fr/content/images/2025/03/img2.png)

D'autre part, la bibliothèque permet aussi d'interroger directement les DataFrames en SQL, grâce à la réflexivité naturelle du langage. Tant qu'un DataFrame est défini dans le scope courant, il peut être traité comme une table SQL.

```
import duckdb
import polars as pl

conn = duckdb.connect()

my_df = pl.DataFrame({"a": 1, "b": 2, "c": 3})
print(conn.sql("SELECT * FROM my_df"))
```

![](http://blog.ippon.fr/content/images/2025/03/img3.png)

Ces deux fonctionnalités permettent une implémentation immédiate de fonctions d'interface, sur une seule ligne.

```
import polars as pl

def from_database(connection, tablename: str) -> pl.DataFrame:
    return connection.sql(f"SELECT * FROM {tablename}").pl()

def to_database(connection, my_data: pl.DataFrame, tablename: str) -> None:
    connection.sql(f"INSERT INTO {tablename} BY NAME (SELECT * FROM my_data);")
```

L'approche légère et simple de DuckDB en font un outil rapide à prendre en main.Non seulement sa syntaxe courte et ses nombreuses interfaces facilitent l'implémentation de fonctionnalités, mais elle rend aussi les refontes (**très** nombreuses en phase exploratoire) beaucoup moins lourdes à effectuer, limitant ainsi le _biais des coûts irrécupérables_.

**Pourquoi choisir DuckDB pour un POC ?**  
✅ Installation rapide  
✅ Prise en main immédiate  
✅ Intégration directe avec Pandas, Polars, NumPy ou Arrow

## DuckDB SQL

Jusqu'ici, on ne s'est penché que sur l'interface Python/DuckDB. Mais DuckDB, c'est aussi une syntaxe SQL enrichie, qui ne se limite pas à s'interfacer avec des DataFrames.

### Lecture / Ecriture

Si l'intégration avec Python permet l'usage de SQL ad-hoc sur des DataFrames, de manière générale, DuckDB s'interface simplement avec de nombreuses entrées/sorties différentes. On notera par exemple la possibilité d'aller interagir directement avec des fichiers sur le disque local, sous différents formats. Parmi les formats compatibles, on trouvera entre autres les habituels json, csv, et parquet.

```
CREATE TABLE src AS 
  (SELECT * FROM './file.parquet');
```

```
COPY 
  (SELECT * FROM src WHERE id='1234')
  TO './filtered_file.parquet'
  (FORMAT 'parquet');
```

### D'autres extensions

En fait, toutes ces fonctionnalités ne font pas partie intégrante d'une espèce de DuckDB-SQL à part entière, mais sont chacune des extensions indépendantes, activables ou non.On distingue deux types d'extensions :

1. Les extensions **core** : Ces extensions sont maintenues par DuckDB, et selon le client utilisé (Python, CLI, Java, etc), certaines sont activées par défaut. La liste complète des extensions core est disponible [ici](https://duckdb.org/docs/extensions/core_extensions?ref=blog.ippon.fr), mais on peut en citer quelques-unes de remarquables :
    - Toutes les extensions de formats de fichier : **json, csv, parquet, excel**
    - Les connexions avec **Azure** et **AWS**
    - Certaines fonctionnalités de traitement de données : recherche full-text, autocomplete, gestion des données spatiales
2. Les extensions **community** : Regroupées sur un [nexus commun](https://duckdb.org/community_extensions/?ref=blog.ippon.fr), il s'agit d'extensions développées et maintenues par des tiers. Pour en citer également quelques-unes :
    - La compatibilité **bigquery** manquante aux extensions core
    - Des fonctions de transformations : **fuzzy matching, hash cryptographiques**
    - Plus de formats de fichiers : **avro, archives zip**

Pour installer et utiliser ces extensions, rien de plus simple. Il suffit d'utiliser la syntaxe INSTALL/LOAD directement en SQL. Par exemple avec l'extension spatial :

```
INSTALL spatial FROM core; -- ou juste INSTALL spatial;
LOAD spatial;
```

Ou tout aussi simplement en python directement :

```
import duckdb

conn = duckdb.connect()
conn.install_extension("spatial", repository="core") 
# ou juste conn.install_extension("spatial") 
conn.load_extension("spatial")
```

Pour charger des extensions community, la méthode reste la même, il suffit de remplacer core par community (le paramètre devient obligatoire dans ce cas).

```
INSTALL bigquery FROM community;
LOAD bigquery;
```

**Les forces du SQL DuckDB**  
✅ Interface avec les formats de fichiers les plus courants  
✅ Fonctionnement modulaire et open-source

## Production-Ready

On a vu jusque-là que DuckDB était à la fois un outil à l'API simple et efficace, et puissant au travers de son aspect open source encourageant la modularité. Ce n'en fait pas pour autant un outil fourre-tout, lui-même à peine plus qu'un POC et voué à un usage purement théorique ou exploratoire.

### Stabilité

C'est même tout le contraire. Tiré directement de sa documentation :

> _While DuckDB was originally created by a research group, it was never intended to be a research prototype. Instead, it was intended to become a stable and mature database system_[_6_](https://duckdb.org/why_duckdb?ref=blog.ippon.fr)

La volonté de l'outil est de pouvoir être utilisé dans un cadre de production.Cette ambition est corroborée notamment par son cycle de développement (il a fallu près de 5 ans entre le début du projet en 2019 et sa release en 2024), confirmant une volonté de ses deux papas de s'assurer de la livraison d'un outil stable.Cependant la meilleure preuve de l'utilisabilité de l'outil est aujourd'hui l'existence de projets satellites entièrement basés sur DuckDB.On citera par exemple [Motherduck](https://motherduck.com/?ref=blog.ippon.fr), une initiative de datawarehouse hébergée dans le cloud entièrement construite sur DuckDB, et produite avec l'approbation et le financement des créateurs.

### Testing

Sortons du cadre de la base de données exploitée en production pour s'attarder sur un cas d'usage différent, mais tout aussi important dans le cadre d'un processus d'intégration continue. Souvenons-nous, lorsque j'ai présenté le résultat de ma propre expérience :

> _Pour des raisons narratives évidentes, j'ai fait le choix de mocker ma base de données_

DuckDB est en effet un excellent choix pour émuler une base de données. Je l'ai certes utilisé dans un contexte de POC, mais imaginons la même utilisation pour développer un test d'intégration end-to-end (lecture - transformation - écriture). A nouveau, deux solutions :

1. Emuler un environnement le plus semblable à la production, par exemple à base de Docker et [LocalStack](https://www.localstack.cloud/?ref=blog.ippon.fr)
2. duckdb.connect(":memory:") et éventuellement mocker également les fonctions de lecture et d'écriture comme pour le POC :

```
import polars as pl

def from_database(connection, tablename: str) -> pl.DataFrame:
   return connection.sql(f"SELECT * FROM {tablename}").pl()

def to_database(connection, my_data: pl.DataFrame, tablename: str) -> None:
   connection.sql(f"INSERT INTO {tablename} BY NAME (SELECT * FROM my_data);")
```

![](http://blog.ippon.fr/content/images/2025/03/img4.png)

Si l'on passe outre la rhétorique un peu simpliste sur la rapidité d'implémentation (je vous laisserai juger par vous même), l'usage de DuckDB présente tout de même deux avantages majeurs :

1. Son côté entièrement gratuit et open-source, un aspect souhaitable dans un cadre de test où les moyens financiers sont différents de la production
2. Sa légèreté. Instancier une connexion in-memory sera systématiquement plus rapide qu'un environnement dockerisé, ce qui réduit mécaniquement la durée des tests, et améliore la boucle de feedback, ce qui induit à terme un code de meilleure qualité[7](https://dl.acm.org/doi/abs/10.1145/3328433.3328456?ref=blog.ippon.fr)

**Un outil robuste**  
✅ Développé avec l'ambition d'être utilisable en production  
✅ A la fois stable et léger à implémenter, parfait pour les tests

## Conclusion

A titre personnel, DuckDB m'a permis de réaliser un POC Data en un temps record, m'absolvant des contraintes de mise place d'un système de gestion de base de données plus lourd. 

Son implémentation rapide et efficace est également un allié incomparable dans la mise en place de tests pertinents, souvent fastidieux à implémenter, maintenir et exécuter.

De manière générale, on a devant nous un outil simple, robuste et open-source, ayant la capacité de vivre avec un projet depuis ses premières phases de développement jusqu'en production.

## Ressources

[https://duckdb.org](https://duckdb.org/?ref=blog.ippon.fr)  
[https://motherduck.com](https://motherduck.com/?ref=blog.ippon.fr)  
[https://mytherin.github.io](https://mytherin.github.io/?ref=blog.ippon.fr)  
[https://hannes.muehleisen.org](https://hannes.muehleisen.org/?ref=blog.ippon.fr)  
[https://www.cloudraft.io/blog/clickhouse-vs-duckdb](https://www.cloudraft.io/blog/clickhouse-vs-duckdb?ref=blog.ippon.fr)  
[https://dl.acm.org/doi/abs/10.1145/3328433.3328456](https://dl.acm.org/doi/abs/10.1145/3328433.3328456?ref=blog.ippon.fr)  
[https://www.localstack.cloud](https://www.localstack.cloud/?ref=blog.ippon.fr)  
[https://aws.amazon.com/fr/redshift](https://aws.amazon.com/fr/redshift/?ref=blog.ippon.fr)