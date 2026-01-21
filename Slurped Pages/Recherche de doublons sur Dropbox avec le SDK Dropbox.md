---
link: https://blog.goovy.io/recherche-de-doublons-sur-dropbox-avec-le-sdk-dropbox/
byline: Denis Garcia & Gerald Colin
site: Goovy Lab
date: 2020-09-12T23:33
excerpt: Voici un exemple d'utilisation du SDK Dropbox pour la recherche de doublons.
slurped: 2026-01-13T09:49
title: Recherche de doublons sur Dropbox avec le SDK Dropbox
---

## Objectif

- Utiliser le SDK Dropbox
- Récupérer les méta données des fichiers et répertoires
- Utiliser le digest des fichiers (content_hash) pour identifier les doublons
- Générer une liste de doublons et calculer la taille qu'on pourrait gagner en supprimant les doublons

## Avant de commencer

- Avoir node and npm installés
- Avoir un compte Dropbox

## Préparation

- Création de l'espace de travail

```
$ mkdir espace_de_travail
$ cd espace_de_travail
$ npm init --yes
```

- installation des dépendances

```
$ npm install -s dropbox isomorphic-fetch lodash
```

- Génération du token Dropbox

Créer un compte sur [https://www.dropbox.com/developers/apps](https://www.dropbox.com/developers/apps?ref=blog.goovy.io)

Créer une application

Récupérer le 'access_token'

- Ajouter le token  dans le fichier token.js

```

exports.TOKEN ='LE_TOKEN_GENERE';
```

## Récupération des fichiers

Nous allons maintenant travailler dans le fichier _recuperation-meta-donnees.js_

- Imports

```
const fs = require('fs');
const fetch = require('isomorphic-fetch');
const Dropbox = require('dropbox').Dropbox;
```

- Activation du SDK Dropbox

```
const TOKEN = require('./token').TOKEN;
var dbx = new Dropbox({ fetch: fetch, accessToken: TOKEN });
```

- La fonction pour récupérer les fichiers

```
async function getFiles(path, process) {
  var response = await dbx.filesListFolder({
          path: path,
          recursive: true
      });
    
  processResponse(response);
  
  while(response.has_more) {
    try {
      response = await dbx.filesListFolderContinue({cursor: response.cursor});
      processResponse(response);
    }
    catch(e) {
      console.error('error', e);
      break;
    }
  }
}
```

- Préparation  de notre data store local

```
const entries = {};
// content_hash: Array<meta_donnees_fichier>
```

- Traitement des données

```
async function processResponse(response) {
    
  response.entries.forEach(processEntry);
  const currentSize = Object.keys(entries).length;
}

async function processEntry(entry) {
	entries[entry.id] = entry
}
```

- Action!

On lance maintenant le programme, on affiche le nombre de meta données et on sauvegarde le résultat dans un fichier json qu'on utilisera par la suite pour trouver les doublons.

```
const path = '/Caderias'; // Le chemin du répertoire Dropbox à scanner. Utiliser '' pour tout sinon '/le-chemin'
getFiles(path, processResponse)
  .then(() => {
    console.log('entries', Object.keys(entries).length);
    fs.writeFileSync('db.json', JSON.stringify(entries, null, 4));
  });
```

## Trouver les doublons

- Les imports

```
const _ = require('lodash'); 
const fs = require('fs'); 
```

- Chargement de la base de données des metadata des fichiers de Dropbox

```

const fileContent = fs.readFileSync('db.json', {
  		encoding:'utf8',
   		flag:'r'
	}
);
const db = fileContent.length > 0 ? JSON.parse(fileContent) : {};

const entries = Object.values(db);
console.log('entries', entries.length);
```

- Maintenant le travail commence, nous allons d'abord filtrer les meta données pour ne garder que les fichiers, puis les grouper sur le champ 'content_hash' qui  représente  le digeste des fichiers. Si deux fichiers ont le même 'content_hash' alors on peut considérer qu'ils sont identiques.

```

// Filtrer les fichiers seulement
const files = entries.filter( e => e['.tag'] === 'file');
console.log('files', files.length);

// Grouper les fichiers par 'content_hash'
const grouped = _.groupBy(files, 'content_hash');
console.log('grouped', Object.keys(grouped).length);
```

- Afficher les doublons

A ce stade nous avons un Map dont la clé est le digest des fichiers ('hash_content') et la valeur est une liste de méta données des fichiers sur Dropbox

Nous allons simplement filtrer et conserver uniquement les digests qui ont une liste avec strictement plus d'un élément, car il s'agit de doublons

```
// ne conserver que les digestes avec doublons
const duplicates = Object.keys(grouped)
    .filter( k => grouped[k].length > 1);

// On map maintenant le nom des fichiers
const duplicatesFiles = duplicates.map(l => 
	l.map(e => e.path_display) // remplacer les méta données du fichier par le nom du fichier dans la liste
);
console.log('duplicatesFiles', duplicatesFiles);
```

- Calculer la taille des fichiers doublons

```
const duplicatedSize = duplicates
    .map(doublons => doublons[0].size * (doublons.length - 1))
    .reduce((a,b) => a + b);
console.log('Manque à gagner', duplicatedSize / 1000 / 1000, 'mb');
```

Nous avons maintenant une liste de doublons (donc une liste de liste).

Nous parcourons la liste des doublons, et pour chaque liste de doublons nous calculons la taille que l'on peut gagner en supprimant tous les doublons (en ne conservant qu'une version).

## Récapitulatif

fichier **get-files.js**

```
const fs = require('fs');
const fetch = require('isomorphic-fetch');
const Dropbox = require('dropbox').Dropbox;

const TOKEN = require('./token').TOKEN;
const dbx = new Dropbox({ fetch: fetch, accessToken: TOKEN });

async function getFiles(path, process) {
  var response = await dbx.filesListFolder({
          path: path,
          recursive: true
      });
    
  processResponse(response);
  
  while(response.has_more) {
    try {
      response = await dbx.filesListFolderContinue({cursor: response.cursor});
      processResponse(response);
    }
    catch(e) {
      console.error('error', e);
      break;
    }
  }
}

const entries = {};
// content_hash: Array<meta_donnees_fichier>

async function processResponse(response) {
    
  response.entries.forEach(processEntry);
  const currentSize = Object.keys(entries).length;
}

async function processEntry(entry) {
	entries[entry.id] = entry
}

function saveDB(entries) {
  fs.writeFileSync('db.json', JSON.stringify(entries, null, 4));
}

const path = '/Caderias'; // Le chemin du répertoire Dropbox à scanner. Utiliser '' pour tout sinon '/le-chemin'
getFiles(path, processResponse)
  .then(() => {
    saveDB(entries);
    console.log('entries', Object.keys(entries).length);
  });
```

fichier **analyse-files.js**

```
const _ = require('lodash'); 
const fs = require('fs'); 

const fileContent = fs.readFileSync('db.json', {encoding:'utf8', flag:'r'});
const db = fileContent.length > 0 ? JSON.parse(fileContent) : {};

const entries = Object.values(db);
console.log('entries', entries.length);

const files = entries.filter( e => e['.tag'] === 'file');
console.log('files', files.length);

const grouped = _.groupBy(files, 'content_hash');
console.log('grouped', Object.keys(grouped).length);

const duplicates = Object.values(grouped)
    .filter( l => l.length > 1);
console.log('duplicates', duplicates.length);

const duplicatesFiles = duplicates.map(l => l.map(e => e.path_display))
console.log('duplicatesFiles', duplicatesFiles);

const duplicatedSize = duplicates
    .map(l => l[0].size * (l.length - 1))
    .reduce((a,b) => a + b);
    // .reduce((a,b) => a.b);
console.log('duplicatedSize', duplicatedSize / 1000 / 1000, 'mb');

console.log('DONE');
```