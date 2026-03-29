---
link: https://www.metal3d.org/blog/2019/httpie/
site: Metal3d
date: 2019-05-05T14:49
excerpt: Nous sommes dans l’ère des API REST, le Web se tourne vers ce mode depuis des années, et ce n’est pas pour nous déplaire - car effectivement, détacher la data du “front” est une bonne idée. Et clairement cela permet aussi de devenir “client” de l’API depuis n’importe quel technologie capable de faire des appels HTTP.
tags:
  - api
  - rest
  - jq
  - bash
slurped: 2026-03-28T10:29
title: Requête d'API REST en bash avec http et jq
---

Nous sommes dans l’ère des API REST, le Web se tourne vers ce mode depuis des années, et ce n’est pas pour nous déplaire - car effectivement, détacher la data du “front” est une bonne idée. Et clairement cela permet aussi de devenir “client” de l’API depuis n’importe quel technologie capable de faire des appels HTTP.

J’adore travailler avec un terminal, je pense que ceux qui me cotoient le savent. Et j’adore automatiser des tâches, telles que récupérer des outils non packagés sans avoir à aller sur une page web, télécharger, déplacer le binaire, etc.

Ici, je vais vous montrer comment je me débrouille pour rendre automatique une récupération de nouvelle version d’un outil que j’utilise avec [Scaleway](https://scaleway.com/), à savoir l’outil “scw” qui me permet de manipuler les serveurs à distance (un peu comme l’outil aws-cli pour Amazon).

Je vais utiliser httpie, jq, et wget en utilisant l’API github. Vous allez vite comprendre l’intérêt.

## Avant automatisation

D’abord, il faut comprendre comment ça se passe. Pour récupérer l’outil “scw”, je dois aller sur la page [https://github.com/scaleway/scaleway-cli/releases](https://github.com/scaleway/scaleway-cli/releases), chercher la dernière (taguée “latest”), puis trouver la version “linux-amd64” qui correspond à mon environnement.

Clique droit sur l’url… et je finis par coller l’url dans un terminal pour faire:

```
wget <url collée> -O ~/.local/bin/scw
chmod +x ~/.local/bin/scw
```

Ça parait assez simple, mais ça me demande pas mal de manipulations. Or, github propose une API ouverte. Alors ne nous en privons pas.

## Travailer avec l’API

Un petit tour sur la [documentation de github](https://developer.github.com/v3/projects/) pour me rendre compte que je peux avoir la liste des “releases” de scw rapidement, et sans effort avoir la “dernière release” - comprennez bien “latest”.

L’URL est donc, dans mon cas: [https://api.github.com/repos/scaleway/scaleway-cli/releases/latest](https://api.github.com/repos/scaleway/scaleway-cli/releases/latest)

Le retourn est un JSON qui me présente des “assets”, quelques infos utiles, et des URL de téléchargement. Bref, un “asset” est un objet que je peux télécharger, ici ce sont les binaires pour toutes les plateformes. Or, moi, je veux la version Linux amd64.

Il est temps de parler de httpie et jq.

## HTTPIe ?

J’utilise souvent “curl”, il est pratique et complet, mais depuis quelques années j’aime beaucoup “httpie” qui est selon moi plus adapté pour jouer avec des APIs. En soit, ça ne change pas grand chose, mais dans les faits il propose une “sortie” propre, colorée, et qui met en forme le JSON, le XML etc.

Vous pouvez bien entendu ne pas utiliser httpie, mais sa facilité pourrait vous convaincre.

Notez bien que “httpie” est le nom du logiciel, mais la commande à utiliser est “`http`”.

## jq ?

JQ, pour Json Query, est un outil en ligne de commande qui permet de traiter (depuis STDIN, donc via des “pipes”) du contenu JSON, de faire des sélections, de la manipulation…

JQ peut paraitre un peu complexe dans sa syntaxe, mais il évite de passer par des commandes grep/awk qui engendrent souvent des erreurs de sélection. JQ est bien plus fiable.

Allons-y…

## Traitement du JSON

Pour aller plus vite, je vais juste enregistrer l’url dans une variable pour éviter de la répéter dans les exemples ci-dessous.

```
$ export URL="https://api.github.com/repos/scaleway/scaleway-cli/releases/latest"
```

En utilisant l’adresse [https://api.github.com/repos/scaleway/scaleway-cli/releases/latest](https://api.github.com/repos/scaleway/scaleway-cli/releases/latest), on remarque une entrée “racine” nommé “assets”:

```
 $ http $URL
 ...
 ...
 {
    "assets": [
    ...
```

Donc, clairement, nous avons là un taleau d’assets. Nous devons donc descendre dans cette entrée pour avoir les assests.

JQ vous propose la syntaxe “`.assets[]`” qui va sortir tous les assets, un par un, mais “hors tableau”, donc en supprimant les crochets. Le résultat n’est plus un JSON valide, mais une série de JSON. En soit ça change presque rien, mais pour le traitement final, ça aura son effet. Mais allons-y pas à pas.

Testons:

```
http $URL | jq '.assets[]'
```

Très bien… Nous avons les assets. On peut récupérer les noms, par exemple:

```
$ http $URL | jq '.assets[].name'
"scw-darwin-amd64"
"scw-darwin-i386"
"scw-freebsd-amd64"
"scw-freebsd-arm"
"scw-freebsd-i386"
"scw-linux-amd64"
"scw-linux-arm"
"scw-linux-arm64"
"scw-linux-i386"
"scw-netbsd-amd64"
"scw-netbsd-arm"
"scw-netbsd-i386"
"scw-windows-amd64.exe"
"scw-windows-i386.exe"
"scw_1.19_amd64.deb"
"scw_1.19_arm.deb"
"scw_1.19_arm64.deb"
"scw_1.19_i386.deb"
```

Bien, et pour les url:

```
$ http $URL | jq '.assets[].browser_download_url'
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-darwin-amd64"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-darwin-i386"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-freebsd-amd64"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-freebsd-arm"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-freebsd-i386"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-linux-amd64"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-linux-arm"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-linux-arm64"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-linux-i386"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-netbsd-amd64"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-netbsd-arm"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-netbsd-i386"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-windows-amd64.exe"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-windows-i386.exe"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw_1.19_amd64.deb"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw_1.19_arm.deb"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw_1.19_arm64.deb"
"https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw_1.19_i386.deb"
```

Il faut maintenant ne récupérer que l’adresse de téléchargment de la version “scw-linux-amd64”, et c’est là que ça se complique un peu. Rien d’insurmontable, mais il faut connaitre ce que propose JQ en terme de filtrage.

Ici, nous voulons utiliser l’asset dont le nom (name) est “scw-linux-amd64”. Pour cela, on va utiliser un “pipe jq” avec la fonction “select” (Voir [la documentation](https://stedolan.github.io/jq/manual/#select\(boolean_expression\)))

```
$ http $URL | jq '.assets[] | select(.name=="scw-linux-amd64")'
{
  "url": "https://api.github.com/repos/scaleway/scaleway-cli/releases/assets/11793235",
  "id": 11793235,
  "node_id": "MDEyOlJlbGVhc2VBc3NldDExNzkzMjM1",
  "name": "scw-linux-amd64",
  "label": null,
  "uploader": {
    "login": "jerome-quere",
    "id": 526046,
    "node_id": "MDQ6VXNlcjUyNjA0Ng==",
    "avatar_url": "https://avatars0.githubusercontent.com/u/526046?v=4",
    "gravatar_id": "",
    "url": "https://api.github.com/users/jerome-quere",
    "html_url": "https://github.com/jerome-quere",
    "followers_url": "https://api.github.com/users/jerome-quere/followers",
    "following_url": "https://api.github.com/users/jerome-quere/following{/other_user}",
    "gists_url": "https://api.github.com/users/jerome-quere/gists{/gist_id}",
    "starred_url": "https://api.github.com/users/jerome-quere/starred{/owner}{/repo}",
    "subscriptions_url": "https://api.github.com/users/jerome-quere/subscriptions",
    "organizations_url": "https://api.github.com/users/jerome-quere/orgs",
    "repos_url": "https://api.github.com/users/jerome-quere/repos",
    "events_url": "https://api.github.com/users/jerome-quere/events{/privacy}",
    "received_events_url": "https://api.github.com/users/jerome-quere/received_events",
    "type": "User",
    "site_admin": false
  },
  "content_type": "application/octet-stream",
  "state": "uploaded",
  "size": 11409167,
  "download_count": 80,
  "created_at": "2019-03-29T13:24:02Z",
  "updated_at": "2019-03-29T13:24:06Z",
  "browser_download_url": "https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-linux-amd64"
}
```

Il ne reste plus qu’à récupérer la propriété “`browser_download_url`” qui est l’url de téléchargement. Un dernier point, nous utilisons l’option “`-r`” pour avoir une sortie brute (raw) afin d’éviter que JQ ne mette en forme l’url avec des guillemets.

```
$ http $URL | jq -r '.assets[] | select(.name=="scw-linux-amd64").browser_download_url'
https://github.com/scaleway/scaleway-cli/releases/download/v1.19/scw-linux-amd64
```

Voilà, nous avons l’url de téléchargement de la dernière version de `scw`, reste à faire un petit script de mise à jour (version simplifiée):

```
# simple function to get latest release from a repository
get_release() {
	REPO=$1
	NAME=$2
	URL="https://api.github.com/repos/${REPO}/releases/latest"
	http $URL | jq -r '.assets[] | select(.name=="'$NAME'").browser_download_url'
}

# update scaleway cli
update_scw() {
	REPO="/scaleway/scaleway-cli"
	NAME=scw-linux-amd64

	URL=$(get_release "$REPO" "$NAME")
	wget "$URL" -O ~/.local/bin/scw
}


case $1 in
    scw) update_scw ;
    *) echo "Unknown $1 command to update";;
esac
```

Et je peux ensuite créer des fonctions pour chaque clients que je récupère sur github, en ne spécifiant le dépôt, et le nom du binaire, en paramètre de “`get_release`”

Et voilà, de temps en temps je tape la commande: `update-cli scw` (ou autre, j’en ai quelques-uns dans ma liste), et plus besoin d’aller sur la page github, de faire des manipulations, etc.

## Conclusion

La ligne de commande n’est pas privée de l’intérêt des API REST. Avec httpie ou curl, jq, et l’ensemble des outils ancêstraux comme awk, grep, sed… quelques lignes de commande suffisent à avoir un client bash capable d’utiliser les API.

Il est évident que pour des API plus complexes, un langage de programmation plus adapté (Python, Go, NodeJS…) va proposer des options plus avancées, mais `jq` est assez complet pour faire des manipulations json remarquables.