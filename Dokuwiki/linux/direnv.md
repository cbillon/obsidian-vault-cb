---
tags:
  - linux
  - environment
  - direnv
---

=== DIRENV ===

Les liens

[[https://blog.wescale.fr/optimiser-cli-avec-direnv| source de l'article]]

[[https://github.com/direnv/direnv/wiki|wiki du projet]]

== Comment fonctionne direnv ? ==

Simplissime : direnv est lié à notre configuration de shell (bash, zsh, tcsh, fish et elvish sont supportés) et vient rajouter le dynamisme qui change tout.

Avant chaque prompt, direnv vérifie l’existence d’un fichier .envrc dans le dossier courant et ses parents. Si le fichier existe et qu’il a été explicitement autorisé par une commande direnv allow, il est sourcé dans un sous-shell et toutes les variables d’environnement de ce sous-shell sont mises à disposition de votre shell. Si vous ressortez du répertoire, ces variables disparaissent et vous retrouvez votre environnement d’origine.
Ce mécanisme très simple peut apporter beaucoup quand on taquine un peu le scripting et la configuration par variables d’environnement. Voyons ce qu’on peut faire avec...

== Installation ==

Commencer par le début est souvent une bonne idée... Vous pouvez installer direnv sur tous les systèmes Linux par 2 options simples :

il est présent dans les packages système de votre distribution (la plupart du temps).
si ce n’est pas le cas, ou si la version disponible ne vous convient pas, vous pouvez toujours récupérer le build qui vous correspond directement sur Github. C’est un binaire Go, donc pas de souci de dépendances système.
Une fois le binaire disponible, il faut intégrer une ligne dans l’init de votre shell pour que direnv soit en capacité de manipuler l’environnement à chaque changement de répertoire.

# dans votre ~/.bashrc par exemple

  eval "$(direnv hook bash)"

Direnv a des intégrations pour Bash, Zsh, Fish, Tcsh et Elvish.

== 4 Exemples concrets ==

Voici quelques exemples d’usages personnels que je trouve bien pratiques. Vous pouvez intégrer directement ces snippets de code dans vos fichiers .envrc.

== 1. Vérifier la disponibilité de certaines commandes ==

À ajouter dans votre .envrc pour vous assurer de la présence de certaines commandes que vous pourriez utiliser plus loin dans votre .envrc :

  DIRENV_CMD_DEPENDENCIES="unzip tar mkdir curl chmod rm"
  for mandatory_cmd in ${DIRENV_CMD_DEPENDENCIES}; do
     if [ -z "$(which ${mandatory_cmd})" ]; then
          echo "===> Mandatory command not found: ${mandatory_cmd}"
          exit 1
     fi
  done
  
== 2. virtualenv Python ==

L’isolation des dépendances projets est un marronnier des projets en Python. direnv supporte plusieurs méthodes pour gérer les virtualenv. Je vous donne ici ma chouchoute, pour sa concision :

  # .envrc
  layout_python3
  
Au chargement, direnv va créer et activer un environnement Python dans un répertoire .direnv qu’il va créer dans le même répertoire que votre fichier .envrc. Simple et efficace.

== 3. Gérer des outils spécifiques au projet ==

Il arrive d’avoir besoin de binaire en version particulière pour builder nos projets. L’installation de Terraform au niveau système est souvent plus un problème qu’autre chose quand on doit maintenir des bases de code qui nécessitent différentes versions...

On va donc s’appuyer sur le répertoire de travail .direnv et y installer nos binaires, tout en manipulant la variable PATH pour les mettre au premier plan quand on se trouve dans le projet.

Important à savoir avant de se lancer dans le scripting : quand le fichier .envrc est interprété par direnv, la variable d’environnement PWD donne le chemin du répertoire où se trouve le fichier .envrc

Pour préparer un répertoire .direnv/bin prêt à recevoir nos binaires et les faire passer en priorité :

  # .envrc
  export DIRENV_TMP_DIR="${PWD}/.direnv"
  export DIRENV_BIN_DIR="${DIRENV_TMP_DIR}/bin"
  if [ ! -e "${DIRENV_BIN_DIR}" ]; then
     mkdir -p "${DIRENV_BIN_DIR}"
  fi
  export PATH="${DIRENV_BIN_DIR}:${PATH}"

Pour télécharger Terraform et le ranger dans nos outils projets :

  # .envrc
  #
  # Terraform CLI installation
  # ==========================
  #
  TF_VERSION="0.14.10"
  TF_ARCH="linux_amd64"
  TF_PKG_NAME="terraform_${TF_VERSION}_${TF_ARCH}.zip"
  TF_PKG_URL="https://releases.hashicorp.com/terraform/${TF_VERSION}/${TF_PKG_NAME}"
  TF_PKG_PATH="${DIRENV_TMP_DIR}/${TF_PKG_NAME}"
  if [ ! -e "${DIRENV_BIN_DIR}/terraform" ]; then
     echo "===> Getting terraform:${TF_VERSION}:${TF_ARCH} (can take a while to execute)"
     curl -s -L "${TF_PKG_URL}" -o "${TF_PKG_PATH}"
     unzip ${TF_PKG_PATH} -d ${DIRENV_BIN_DIR}
     chmod 700 ${DIRENV_BIN_DIR}/terraform
     rm -f ${TF_PKG_PATH}
  fi

Pour changer de version, il suffira de :

changer la valeur de TF_VERSION
supprimer le répertoire .direnv (ou juste le binaire dans .direnv/bin)
lancer direnv reload pour relancer la construction du .direnv (ou sortez du répertoire et revenez-y avec cd)

== 4. Configuration de Terraform ==

Si vous jouez avec Terraform, suivant les providers, on peut rapidement avoir besoin de variables d’environnement pour éviter de mettre ses accès dans le code. Prenons comme exemple le provider AWS. Pour éviter de mettre des valeurs spécifiques en dur dans le code Terraform, on peut s’appuyer sur des variables d’environnement. Mais forcément : on n’a pas plus envie de les insérer dans le .envrc que dans le code Terraform.

Au lieu de ça je vous propose d’ajouter au fichier .envrc le chargement conditionnel suivant :

  #
  # Environment configuration
  # =========================
  #
  # Customize your project environment by filling these files with variables exports
  # at the root of your project.  
  #
  ENV_ADDONS=".env.aws .env.drone"
  for addon in ${ENV_ADDONS}; do
     if [ -e "${PWD}/${addon}" ]; then
         source ${PWD}/${addon}
     fi
  done

Vous ajoutez dans votre .gitignore les fichiers listés dans ENV_ADDONS pour vous assurer que leur contenu ne sera pas poussé sur le dépôt central.

Vous pouvez maintenant remplir les variables AWS spécifiques à votre profil de connexion tranquillement :

  # .env.aws
  export AWS_PROFILE=...
  export AWS_DEFAULT_REGION=...

Et finir par un :

  $ direnv reload

...pour vous assurer du chargement.

Vous pouvez propager ce pattern à tout ce qui permet une configuration par variable d’environnement et qui nécessite des valeurs liées à l’utilisateur.

== Conclusion ==

Direnv a une belle base d’utilisateurs qui a contribué à un certain nombre de recettes pour se faciliter la vie sur l’intégration d‘outils tels que : Git, NodeJS, Emacs, Vim, Tmux, Go, GCloud, et d’autres. Vous trouverez tout ça sur le wiki du projet.

Le projet va fêter ses 11 ans en novembre prochain, est toujours très actif et a plus de 120 contributeurs. C’est donc selon moi une bonne mise pour un outil qui permet de tacler à la cheville les opérations récurrentes qui concernent nos workspaces locaux. L’essayer c’est l’adopter.