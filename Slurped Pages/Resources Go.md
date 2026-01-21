---
link: https://blog.goovy.io/resources-go/
byline: Gerald Colin & Denis Garcia
site: Goovy Lab
date: 2019-06-14T02:28
excerpt: Une liste de resources, articles, librairies pour le développement en Go.
slurped: 2026-01-13T09:51
title: Resources Go
---

Une liste de resources, articles, librairies pour le développement en Go.

- [](https://blog.goovy.io/author/gerald/)

Une liste de resources, articles, librairies pour le développement en [Go](https://golang.org/?ref=blog.goovy.io). Nous les avons testés et implémentés et avons validé leur intérêt. Ce post sera mis à jour au fur et à mesure.

[NDLR] Quand on aborde une problématique (parsing de json, sécurité...) il y a plusieurs approches possibles qui dépendent du contexte :

- vous codez directement la solution, si possible en s'appuyant sur des patterns de programmation : cela à l'avantage de garder votre code concis et minime en ne répondant qu'à votre problématique et vous facilite les optimisations éventuelles; à contrario vous réinventez la roue et pour des algos plus complexes peut être moins optimal qu'une lib développée par une équipe le ferait.
- vous utilisez une librairie externe : cela à l'avantage de traiter plus de cas d'usages en une fois (quand la lib a été éprouvée) mais à l'inconvénient d'ajouter des dépendances et augmente la taille de vos projets.

---

Lib pour gérer les paramètres de ligne de commande mais oblige à suivre leur paradigme. De nombreux gros projets l'utilisent (docker, kubernetes, istio...)  
- [https://github.com/spf13/cobra](https://github.com/spf13/cobra?ref=blog.goovy.io)

Lib pour charger facilement des paramètres d'un fichier de config  
- [https://github.com/joho/godotenv](https://github.com/joho/godotenv?ref=blog.goovy.io)

Lib pour traiter le JSon plus facilement et sans créer de struc:  
- [https://github.com/valyala/fastjson](https://github.com/valyala/fastjson?ref=blog.goovy.io)

Lib pour consommer des APIs REST, resty :  
- [https://github.com/go-resty/resty](https://github.com/go-resty/resty?ref=blog.goovy.io)

Lib qui implemente les principaux patterns de messaging (pub/sub, req/rep, push/pull...):  
- [https://github.com/nanomsg/mangos](https://github.com/nanomsg/mangos?ref=blog.goovy.io)

Lib pour implémenter un réseau P2P:  
- [https://github.com/libp2p/go-libp2p](https://github.com/libp2p/go-libp2p?ref=blog.goovy.io)

Lib pour construire des microservices (ou de jolis monoliths) :  
- [https://gokit.io/](https://gokit.io/?ref=blog.goovy.io)

Framework de site web en go :  
- [https://gobuffalo.io/fr](https://gobuffalo.io/fr?ref=blog.goovy.io)

Framework de tests de validation :  
- [https://agouti.org/](https://agouti.org/?ref=blog.goovy.io)