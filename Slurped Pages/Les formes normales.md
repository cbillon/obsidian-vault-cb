---
link: https://www.loxodata.com/post/formes-normales/
excerpt: Un petit peu de théorie avec un rappel sur les formes normales.
slurped: 2025-02-22T09:08
title: Les formes normales
tags:
  - postgresql
  - forme-normale
---

## Formes normales

Les formes normales sont un ensemble de normes permettant de structurer une base de données relationnelles avec cohérence. Elles visent à réduire la redondance et à améliorer l’intégrité des données. Les formes normales ont été initiées par Edgar Frank Codd, le concepteur du modèle relationnel.

On peut voir les formes normales comme un ensemble de couches dans lequel chaque forme normale est un niveau. Chaque niveau est alors construit sur son précédent en y ajoutant des exigences supplémentaires. Si votre modèle de base de données correspond à la troisième forme normale, alors il correspond obligatoirement aux première et deuxième formes normales. L’application de ces normes facilite également la maintenance et l’évolution des bases de données.

## Avantages

La normalisation des bases de données est un processus essentiel dans la conception de systèmes de gestion de données. En éliminant les anomalies potentielles découlant d’une mauvaise modélisation de données, elle a plusieurs avantages :

- assurer l’intégrité et la fiabilité des données ;
- optimiser les performances de sa base de données ;
- réduire les risques de problèmes de mise à jour ;
- éviter les problèmes tels que les anomalies de lecture et les anomalies d’écriture avec une cohérence des données.

## Première forme normale (1NF)

- **Valeur atomique** : Une valeur indivisible.
- **Cellule** : L’intersection d’une ligne et d’une colonne dans une table de base de données, contenant une valeur de données unique.
- **Attribut** : Une colonne dans une table de base de données représentant une entité.

_Définition_ : garantit que chaque cellule contient une seule valeur atomique.

On souhaite stocker dans une table, les différentes informations d’une marque avec ses produits.

**Table Brand**

|Brand ID#|Brand Name|Products|
|---|---|---|
|1|Philips|TV, Microwave, Speaker|
|2|Sony|TV, Headphones|
|3|Dell|Laptop, Monitor|

La table **Brand** est identifiée par le **Brand ID** et est composée d’un attribut **Products**. Les valeurs de **Products** rassemblent l’ensemble des produits de la marque. Cette table n’est donc pas conforme à la première forme normale, car les cellules de **Products** ne contiennent pas une seule valeur atomique.

Avec cette conception, l’extraction de produits uniques implique d’extraire tous les produits de la marque. De plus, cette conception n’empêche pas la duplication d’un même produit pour une même marque.

Il serait préférable de décomposer les différents produits en différentes cellules. La table symboliserait alors les productions et aurait un identifiant composé de **Brand ID** et du produit :

**Table Production**

|Maker ID#|Maker Name|Product#|
|---|---|---|
|1|Philips|TV|
|1|Philips|Microwave|
|1|Philips|Speaker|
|2|Sony|TV|
|2|Sony|Headphones|
|3|Dell|Laptop|
|3|Dell|Monitor|

Cette forme normale permet d’éviter la présence de valeurs multiples ainsi que de faciliter l’extraction de données avec des structures de données moins complexes dans une même cellule.

## Deuxième forme normale (2NF)

_Définition_ : garantit que chaque attribut non-clé dépend de l’ensemble de la clé primaire, éliminant ainsi les dépendances partielles au sein de la table, en plus de satisfaire à la première forme normale.

**Table Product**

|Product ID#|Product|Brand Name|Reference|
|---|---|---|---|
|1|TV|Philips|35"RfC…|
|2|TV|Philips|40"RfC…|
|3|Microwave|Philips|ABC-123|
|4|Speaker|Philips|XYZ-456|
|5|TV|Sony|SON-40"D|
|6|Headphones|Sony|GHI-101|
|7|Laptop|Dell|JKL-202|
|8|Monitor|Dell|MNO-303|
|9|Tablet|Philips|PQR-404|
|10|Camera|Sony|STU-505|
|11|Printer|Dell|VWX-606|

La clé primaire de la table **Product** est composée de l’**ID** du produit. Le nom de la marque ne doit pas dépendre de l’identifiant du produit, mais de l’identifiant de la marque. On a donc ici un attribut **Brand Name** qui ne dépend pas de **Product ID**, clé primaire de la table.

Avec cette conception, nous ne pouvons pas obtenir l’ensemble de la marque depuis un produit, le nom de la marque ne permettant pas une identification certaine. Il serait préférable de remplacer le nom de la marque par son identifiant qui lui est unique comme ci-dessous :

**Table Product**

|Product ID#|Product|Brand Id|Reference|
|---|---|---|---|
|1|TV|1|35"RfC…|
|2|TV|1|40"RfC…|
|3|Microwave|1|ABC-123|
|4|Speaker|1|XYZ-456|
|5|TV|2|SON-40"D|
|6|Headphones|2|GHI-101|
|7|Laptop|3|JKL-202|
|8|Monitor|3|MNO-303|
|9|Tablet|1|PQR-404|
|10|Camera|2|STU-505|
|11|Printer|3|VWX-606|

**Table Brand**

|Brand Id#|Brand Name|
|---|---|
|1|Philips|
|2|Sony|
|3|Dell|

Cette forme normale permet d’éviter :

- les problèmes de performance (écriture) ;
- les problèmes de cohérence des données.

## Troisième forme normale (3NF)

_Définition_ : garantit que chaque attribut non-clé ne dépend pas d’un attribut ne participant pas à l’identification, en plus de satisfaire aux deux premières formes normales.

On souhaite stocker dans une table, les différentes informations d’une livraison. Une livraison est composée d’un produit en une certaine quantité.

**Table Delivery**

|Delivery Id#|Product Id|Product Category|Quantity|
|---|---|---|---|
|1|1|TV and song|90|
|2|3|Home_Appliance|60|
|3|6|TV and song|110|
|4|1|TV and song|95|

La catégorie du produit dépend normalement du produit et non de **Delivery Id**, l’identifiant de la livraison. On a donc un attribut **Product Category** qui ne dépend pas de l’attribut clé **Delivery Id**. Avec cette conception, la catégorie du produit ne dépend pas du bon élément, les données peuvent ne pas être cohérentes. De plus, la catégorie du produit sera répétée pour différentes livraisons du même produit.

Il serait préférable de définir la catégorie dans la table **Product**.

**Table Delivery**

|Delivery Id#|Product Id|Quantity|
|---|---|---|
|1|1|90|
|2|3|60|
|3|6|110|

**Table Product**

|Product ID#|Product|Brand Id|Reference|Category|
|---|---|---|---|---|
|1|TV|1|35"RfC…|TV and song|
|3|Microwave|1|ABC-123|Home_Appliance|
|6|Headphones|2|GHI-101|TV and song|

Cette forme normale assure une cohérence des données et évite la redondance des valeurs.

## Forme normale de Boyce-Codd (BCNF)

_Définition_ : garantit que seul un attribut clé permet de déterminer un attribut non-clé et non l’inverse. Une partie de l’identifiant ne dépend pas d’un attribut non identifiant.

Imaginons que notre livraison soit définie par le contenu de la livraison à une date. Il est donc impossible d’avoir deux livraisons du même contenu le même jour.

**Table Delivery**

|Delivery Date#|Product Category#|Product Id|Quantity|
|---|---|---|---|
|2023-12-22|TV and song|1|90|
|2023-12-22|Home_Appliance|3|60|
|2024-01-24|TV and song|6|110|

L’attribut **Product Category** dépend de l’attribut **Product Id**.

On a donc un attribut clé **Product Category**, qui dépend d’un attribut non-clé **Product Id**. Avec cette conception, le contenu de la livraison n’est pas suffisamment précis. De plus, une anomalie peut avoir lieu avec deux livraisons d’un même produit, mais de catégories différentes.

Il serait préférable de définir la catégorie dans la table **Product**, et de définir **Delivery** par sa date et le produit livré.

**Table Delivery**

|Delivery Date#|Product Id#|Quantity|
|---|---|---|
|2023-12-22|1|90|
|2023-12-22|3|60|
|2024-01-24|6|110|

**Table Product**

|Product ID#|Product|Category|
|---|---|---|
|1|TV|TV and song|
|3|Microwave|Home_Appliance|
|6|Headphones|TV and song|

Cette forme normale élimine les anomalies potentielles en assurant une cohérence des données. Elle ajoute aussi de la précision.

## Quatrième forme normale (4NF)

**Entité** : Un objet ou concept représenté par une table et ayant des attributs spécifiques.

_Définition_ : garantit qu’une identification multivaluée doit être cohérente. Une identification multivaluée ne peut être composée d’éléments non dépendants entre eux. Il ne doit pas être possible de créer une relation avec moins d’attributs que l’identification.

Imaginons qu’une marque fabrique différents produits et qu’une marque possède différents sites permettant de créer l’ensemble de ses produits.

**Table Production**

|Brand Id#|Product Id#|Site#|
|---|---|---|
|1|1|Dijon|
|1|2|Nantes|
|2|2|Fontainebleau|
|2|4|Fontainebleau|
|3|5|Orléans|
|3|5|Toulouse|

La table **Production** est identifiée par les attributs **Brand Id**, **Product Id** et **Site**. Le site de la marque et ses produits font partie d’un identifiant commun. Cependant, un site fabrique l’ensemble des produits d’une marque et l’ensemble de ses produits sont fabriqués dans chaque site.

Le produit et le site font partie d’une même identification multivaluée sans avoir de lien entre l’un et l’autre. Avec cette conception, une redondance de données peut avoir lieu, par exemple avec les cellules “Fontainebleau”. Il serait préférable de créer une table de relation entre l’entité **Brand** et l’entité **Products** et une deuxième table de relation entre l’entité **Brand** et l’entité **Site**.

**Table Production**

|Brand Id#|Product Id#|
|---|---|
|1|1|
|1|2|
|2|2|
|2|4|
|3|5|

**Table Site**

|Brand Id#|Site#|
|---|---|
|1|Dijon|
|1|Nantes|
|2|Fontainebleau|
|3|Orléans|
|3|Toulouse|

Cette forme normale permet d’éviter la redondance de données.

## Cinquième Forme normale (5NF)

L’application de la 4NF s’applique lorsqu’une relation a une identification multivaluée non obligatoire. _Définition_ : La cinquième forme normale interdit de décomposer une relation si elle implique une perte d’information.

Imaginons qu’une marque fabrique différents produits. Et qu’une marque possède différents sites, chaque site fabriquant certains produits de la marque.

**Table Production**

|Brand Id#|Product Id#|
|---|---|
|1|1|
|1|2|
|2|2|
|2|4|
|3|5|

**Table Site**

|Brand Id#|Site#|
|---|---|
|1|Dijon|
|1|Nantes|
|2|Fontainebleau|
|3|Orléans|
|3|Toulouse|

La table **Production** forme la dépendance entre l’entité **Brand** et l’entité **Product**. La table **Site** forme la dépendance entre **Brand** et **Site**.

Chaque site fabrique certains produits.

La table **Site** possède une dépendance avec **Product Id**. Avec cette conception, la relation entre l’entité **Site** et l’entité **Product** disparait.

Il serait préférable de créer une même table identifiée par la marque, le produit et le site.

**Table Production**

|Brand Id#|Product Id#|Site#|
|---|---|---|
|1|1|Dijon|
|1|2|Nantes|
|2|2|Fontainebleau|
|2|4|Fontainebleau|
|3|5|Orléans|
|3|5|Toulouse|

Cette forme normale permet d’éviter une perte de données.

## Avantages et inconvénients de la normalisation

En règle générale, la normalisation est effectuée jusqu’à la troisième forme normale, les autres formes normales étant appliquées dans des cas particuliers.

La mise en place de la normalisation permet de réduire la redondance des données et d’assurer leur intégrité. La normalisation peut également améliorer les performances des requêtes en réduisant la taille des tables et en facilitant l’indexation.

Cependant l’application de ces règles peut impliquer d’ajouter des tables et des clés étrangères, et donc de devoir gérer des jointures sur des requêtes plus ou moins complexes. Il est important de trouver un équilibre sur le respect de ces règles, en fonction des besoins spécifiques de l’application et des performances requises.