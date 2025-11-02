---
tags:
  - go
  - bubble_tea
---
[Composable view](http://golangweekly.com/)

[Tips Bubble Tea](https://leg100.github.io/en/posts/building-bubbletea-programs/)
Le point n°6 - Construire des modèles d'arborescence - vaut la peine d'être considéré. Et comme je le dis à ce sujet :

"Cependant, Bubble Tea vous laisse les décisions architecturales et vous devrez prendre des décisions conscientes sur la façon de gérer la complexité qui survient inévitablement une fois que votre programme atteint une certaine taille."

Mes applications TUI ont généralement un modèle "racine". Celui-ci a à son tour des modèles enfants, dont un ou plusieurs peuvent être actuellement visibles, mais l'utilisateur peut les "échanger" contre d'autres modèles.

Chaque modèle est conservé dans un cache. Si l'utilisateur a déjà visité le modèle, le modèle est récupéré du cache, sinon le modèle est créé et ajouté au cache. Une clé de cache distingue les modèles les uns des autres.

Une pile de modèles suit "l'historique", pour permettre à l'utilisateur de revenir au modèle précédent.

J'ai constaté plus récemment qu'il est préférable de référencer ces modèles via des pointeurs, avec des fonctions Update(msg) qui se mettent à jour et ne renvoient qu'une commande.

Je suis d'accord avec un autre intervenant sur le fait qu'il ne faut pas mélanger la logique métier avec votre code bubbletea. Traitez le code bubbletea comme une simple couche de présentation, appelant des services qui exécutent la logique du domaine métier et effectuent des appels réseau, des requêtes de base de données, etc., tout le truc DDD standard.

Et je suis d'accord avec d'autres pour dire qu'un framework de style Elm n'est pas une bonne solution pour Go. En même temps, cela fait de vous un meilleur programmeur Go car vous devez vraiment être au top en ce qui concerne les récepteurs de pointeurs Vs valeurs (si vous ne l'étiez pas déjà).