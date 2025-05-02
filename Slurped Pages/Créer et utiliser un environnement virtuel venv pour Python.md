---
link: https://zonetuto.fr/python/creer-et-utiliser-un-environnement-virtuel-venv-pour-python/
byline: MrTuto
site: ZoneTuto
date: 2025-01-08T16:21
excerpt: Pourquoi créer un environnement virtuel pour utiliser Python sans se soucier de la version des dépendances installées avec pip
slurped: 2025-02-22T09:20
title: Créer et utiliser un environnement virtuel venv pour Python
tags:
  - env
  - virtual
  - python
  - venv
---

On peut facilement remarquer que le langage de programmation Python s’est aujourd’hui largement imposé. Ce n’est d’ailleurs pas du tout mon langage de programmation préféré, mais force est de constater que pour faire beaucoup de choses facilement et surtout rapidement, Python est très un très bon choix. Il évolue rapidement et il y a surtout énormément de projets modernes qui l’utilisent aujourd’hui. Notamment dans le domaine de l’intelligence artificielle que j’aime bien traiter ici.

Comme beaucoup de développeurs, j’ai donc commencé à utiliser de manière intensive le Python pour tout un tas de raisons. La principale raison pour moi, c’est qu’il est facile à apprendre et surtout, il y a un très grand nombre de librairies qui sont maintenant en Python. Pour aller vite, je n’ai donc pas le choix et je m’adapte. Allez, c’est fini de parler de moi et de mes choix, on va maintenant évoquer ce qui m’amène à écrire cet article sur les environnements virtuels en Python et pourquoi, c’est très pratique d’utiliser venv.

## L’erreur courante This environment is externally managed

Sous Linux et notamment Ubuntu pour moi, car c’est celui que j’utilise, mais c’est le cas avec d’autres distributions, si vous essayez d’installer un projet avec pip et Python, vous pouvez rapidement rencontrer cette erreur très fréquente. Quand je viens de télécharger un projet et que je veux l’installer avec la commande suivante :

```
pip install -r requirements.txt
```

Je vous mets ici le message qui vous a surement poussé à faire des recherches et à tomber sur cet article :

```
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.
   
    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.
   
    If you wish to install a non-Debian packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.
   
    See /usr/share/doc/python3.12/README.venv for more information.

note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
hint: See PEP 668 for the detailed specification.
```

L’erreur externally-managed-environment survient lorsque vous essayez d’installer des paquets Python dans un environnement géré par le système, comme une installation globale de Python sur votre machine. Cela se produit principalement sur les distributions Linux modernes, notamment Ubuntu, où le gestionnaire de paquets de l’OS (comme `apt`) prend en charge les installations globales de Python et de ses bibliothèques.

Pour palier à ce problème et éviter de tout casser sur votre distribution Linux, la meilleure solution est de créer un environnement virtuel pour le projet que vous souhaitez utiliser. Ainsi, ce que vous installez dans le cadre de votre projet ne va pas interférer avec le système d’exploitation et inversement. C’est très pratique et pour cela, nous allons utiliser venv.

Avant de créer un environnement virtuel, assurez-vous que Python et pip sont installés sur votre système :

```
sudo apt update 
sudo apt upgrade
sudo apt install python3 python3-venv python3-pip
```

Pour vérifier que Python 3.x et pip sont bien installés :

```
python3 --version
pip --version
```

Vous pouvez ensuite aller dans le dossier votre projet qui nécessite de créer un environnement virtuel venv :

```
cd mon_projet_zoneutot
```

Utilisez la commande python3 -m venv pour créer un environnement virtuel dans ce répertoire.

```
python3 -m venv venv
```

Cela créera un sous-répertoire `venv` dans votre répertoire de projet. C’est ici que seront installées toutes les dépendances de votre projet. Une fois l’environnement créé, vous devez l’activer pour qu’il soit utilisé. Vous pouvez le faire en restant dans le dossier de votre projet avec la commande :

```
source venv/bin/activate
```

Vous devriez voir le nom de votre environnement virtuel (`venv`) apparaître dans votre invite de commande :

```
(venv) user@hostname:~/mon_projet_zoneutot$
```

Maintenant que votre environnement virtuel venv est bien activé et fonctionnel, vous pouvez à nouveau essayer d’installer les dépendances avec pip. Cette fois l’erreur externally-managed-environment ne devrait pas se produire et vous allez pouvoir utiliser votre projet.

Lorsque vous avez terminé de travailler dans l’environnement virtuel, vous pouvez aussi le désactiver :

```
deactivate
```

Cela vous ramène à l’environnement Python global du système. Si vous n’avez plus besoin de l’environnement virtuel, vous pouvez simplement supprimer le répertoire `venv` :

```
rm -rf venv
```

Cela supprimera l’environnement virtuel ainsi que tous les paquets installés dans cet environnement.