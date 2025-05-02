---
tags:
  - docker
---

## Installation de hadolint

[hadolint](https://github.com/hadolint/hadolint)

A smarter Dockerfile linter that helps you build best practice Docker images. The linter is parsing the Dockerfile into an AST and performs rules on top of the AST. It additionally is using the famous Shellcheck to lint the Bash code inside RUN instructions.


Hadolint peut être installé avec asdf


  asdf plugin add hadolint
  asdf install hadolint latest
  asdf global hadolint latest
 
Vérification de l'installation 

  hadolint -v
  
Pour vérifier un dockerfile : hadolint <nom du docker file>

[site](https://blog.stephane-robert.info/docs/conteneurs/images-conteneurs/ecrire-dockerfile/|conseils pour ecrire un docker file)

