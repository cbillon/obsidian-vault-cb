---
tags:
  - mysql
  - tips
---

====== Calculer la taille d'une base MySQL ======

[[https://www.vincentliefooghe.net/index.php/content/calculer-la-taille-dune-base-mysql]]

Il est assez simple de récupérer des données sur la taille d'une base MySQL, via le schéma information_schema.

===== Données pour tous les schémas =====

Par exemple, pour avoir la taille totale de la base de données :

  SELECT table_schema "Nom de la DataBase", SUM( data_length + index_length) / 1024 / 1024 
      -> "Taille en Mo" FROM information_schema.TABLES GROUP BY table_schema ;

===== Données pour un schéma spécifique =====

Si on veut plus de détails sur un seul schéma, on peut utiliser une requête du type :

  mysql> SELECT TABLE_NAME, table_rows, data_length, index_length, 
      -> round(((data_length + index_length) / 1024 / 1024),2) "Taille en Mo"
      -> FROM information_schema.TABLES WHERE table_schema = "monschema";