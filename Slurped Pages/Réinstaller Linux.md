---
link: https://www.glukhov.org/fr/developer-tools/terminals-shell/reinstall-linux/
byline: À propos de Rost Glukhov
site: Rost Glukhov | Site personnel et blog technique
excerpt: Comment réinstaller Linux avec les applications les plus utiles
slurped: 2026-03-22T11:00
title: Réinstaller Linux
---

Il faut avoir une séquence standard de tâches post-installation pour la référence, donc je la note ici.
## Où

Généralement, j’utilise des distributions basées sur Ubuntu. L’installation la plus récente est Mint 21.3 (basée sur Ubuntu 22.04).

### quelques outils agréables

```
sudo apt-get install git git-lfs gimp mc flameshot htop nvtop chkservice

# si le travail graphique est à l'horizon
sudo apt-get install imagemagick
git lfs install

# si une manipulation de PDF est nécessaire
sudo apt-get install poppler-utils
```

### manipulation JSON

Voir les exemples dans [Bash Cheat Sheet](https://www.glukhov.org/fr/developer-tools/terminals-shell/bash-cheat-sheet/)

```
sudo apt-get install jq jo
```

### Installer les pilotes NVidia

#### Méthode 1

Supprimer les pilotes NVidia locaux

```
sudo apt-get purge 'nvidia*'
sudo apt-get autoremove
sudo apt-get autoclean
```

Ajouter le PPA et mettre à jour les références de paquets locaux

```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
```

Vérifier quelle version de pilote NVidia recommande pour votre appareil

Installer

```
sudo apt-get install nvidia-driver-535
sudo reboot
```

Vérifier que vous pouvez voir votre GPU et voir quelle version est installée

#### Méthode 2

Voir ici pour votre version d’OS : [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads)

Pour installer la version du module noyau ouvert :

```
sudo apt-get install -y nvidia-driver-555-open
sudo apt-get install -y cuda-drivers-555
```

### Installer CUDA

Le même site officiel NVidia : [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads)

```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-ubuntu2204.pin
sudo mv cuda-ubuntu2204.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/12.9.0/local_installers/cuda-repo-ubuntu2204-12-9-local_12.9.0-575.51.03-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2204-12-9-local_12.9.0-575.51.03-1_amd64.deb
sudo cp /var/cuda-repo-ubuntu2204-12-9-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-9
```

### flatpacks

vlc, obsidian, nextcloud desktop, foliate, dbeaver

### ungoogled-chromium

[https://github.com/ungoogled-software/ungoogled-chromium](https://github.com/ungoogled-software/ungoogled-chromium)

```
echo 'deb http://download.opensuse.org/repositories/home:/ungoogled_chromium/Ubuntu_Jammy/ /' | sudo tee /etc/apt/sources.list.d/home:ungoogled_chromium.list
curl -fsSL https://download.opensuse.org/repositories/home:ungoogled_chromium/Ubuntu_Jammy/Release.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/home_ungoogled_chromium.gpg > /dev/null
sudo apt update
sudo apt install ungoogled-chromium
```

### golang

Cette version installe une version assez ancienne, actuellement 1.18

```
sudo apt-get install golang-go
```

donc, allez sur [https://go.dev/dl/](https://go.dev/dl/) et choisissez votre dernière version, puis,

et

```
wget https://go.dev/dl/go1.24.3.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.24.3.linux-amd64.tar.gz
```

ensuite, ajoutez à $HOME/.profile

```
export PATH=$PATH:/usr/local/go/bin
```

puis

```
source $HOME/.profile
go version
```

### vs code

[https://code.visualstudio.com/docs/setup/linux](https://code.visualstudio.com/docs/setup/linux)

Installer le paquet deb (il faut le télécharger d’abord). L’installation du paquet .deb installe automatiquement le dépôt apt et la clé de signature pour permettre la mise à jour automatique via le gestionnaire de paquets du système.

Ou faites la même chose manuellement ci-dessous :

```
sudo apt-get install wget gpg
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
rm -f packages.microsoft.gpg
```

Puis mettez à jour le cache des paquets et installez le paquet en utilisant :

```
sudo apt install apt-transport-https
sudo apt update
sudo apt install code # ou code-insiders
```

Installer les extensions de vs code :

Python, C#, Go, Hugohelper, Front Matter CMS, React*, [Flutter](https://www.glukhov.org/fr/post/2022/flutter-dart-cheatsheet/ "Flutter (Dart) Cheatsheet avec exemples etc")

#### Le vs code standard a quelques éléments de télémétrie - partiellement désactivés dans les paramètres: user:application:telemetry=>off … mais pas tout à fait …

La version flatpack de VSCodium est une version sans télémétrie, pas très en retard.

### Python et Anaconda

Installer pip

```
sudo apt install python3-pip
```

[https://www.anaconda.com/download/success](https://www.anaconda.com/download/success)

télécharger la version linux comme

```
wget https://repo.anaconda.com/archive/Anaconda3-2024.06-1-Linux-x86_64.sh
```

et l’exécuter

```
bash Anaconda3-2024.06-1-Linux-x86_64.sh
```

faites attention à la fin :

```
Si vous préférez que l'environnement base de conda ne soit pas activé au démarrage,
   exécutez la commande suivante lorsque conda est activé :

conda config --set auto_activate_base false

Vous pouvez annuler cela en exécutant `conda init --reverse $SHELL`? [yes|no]
```

### hugo

[https://gohugo.io/installation/linux/](https://gohugo.io/installation/linux/)

par exemple, ceci : [https://github.com/gohugoio/hugo/releases/tag/v0.124.1](https://github.com/gohugoio/hugo/releases/tag/v0.124.1)

télécharger et installer [hugo_extended_0.124.1_linux-amd64.deb](https://github.com/gohugoio/hugo/releases/download/v0.124.1/hugo_extended_0.124.1_linux-amd64.deb)

### kubectl

[https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management)

```
sudo apt-get update
# apt-transport-https peut être un paquet fictif ; si c'est le cas, vous pouvez le passer
sudo apt-get install -y apt-transport-https ca-certificates curl
```

```
# Si le dossier /etc/apt/keyrings n'existe pas, il devrait être créé avant la commande curl, lisez la note ci-dessous.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # permet aux programmes APT non privilégiés de lire cette cléring
```

```
# Cela remplace toute configuration existante dans /etc/apt/sources.list.d/kubernetes.list
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # aide les outils tels que command-not-found à fonctionner correctement
```

```
sudo apt-get update
sudo apt-get install -y kubectl
```

### ssh keys

copier les clés ssh vers ~/.ssh

puis copier l’id vers tous les ipaddrs de votre labo

```
ssh-copy-id username@ipaddr
```

### docker

désinstaller l’existant

```
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

Installer à l’aide du dépôt apt

```
# Ajouter la clé officielle de Docker :
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Ajouter le dépôt aux sources de Apt :
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

si vous utilisez des dérivés d’Ubuntu, vous devez utiliser UBUNTU_CODENAME au lieu de VERSION_CODENAME, comme

```
# Ajouter la clé officielle de Docker :
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Ajouter le dépôt aux sources de Apt :
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$UBUNTU_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Installer la dernière version :

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Vérifier

```
sudo docker run hello-world
```

Optionnellement, ajouter l’utilisateur actuel au groupe docker

### portainer

- [https://www.portainer.io/install](https://www.portainer.io/install)
- [https://docs.portainer.io/start/install-ce](https://docs.portainer.io/start/install-ce)
- [https://docs.portainer.io/start/install-ce/server](https://docs.portainer.io/start/install-ce/server)
- [https://docs.portainer.io/start/install-ce/server/docker/linux](https://docs.portainer.io/start/install-ce/server/docker/linux)

Pour commencer, vous aurez besoin de :

- La dernière version de Docker installée et fonctionnelle
- Accès sudo sur la machine qui hébergera votre instance Portainer Server
- Par défaut, Portainer Server exposera l’interface utilisateur sur le port 9443 et exposera un serveur de tunnel TCP sur le port 8000. Le dernier est optionnel et n’est requis que si vous prévoyez d’utiliser les fonctionnalités Edge avec des agents Edge.

Requis :

- SELinux est désactivé sur la machine exécutant Docker. Si vous avez besoin de SELinux, vous devrez passer le drapeau –privileged à Docker lors du déploiement de Portainer.

D’abord, créez le volume que Portainer Server utilisera pour stocker sa base de données :

```
sudo docker volume create portainer_data
```

Ensuite, téléchargez et installez le conteneur Portainer Server :

```
sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

vérifier

Accédez à : https://localhost:9443

Le nom d’utilisateur est admin, définissez le mot de passe dans l’interface utilisateur

### Agent Portainer Kubernetes

Créer l’environnement Kubernetes, l’agent

```
kubectl apply -f https://downloads.portainer.io/ce2-19/portainer-agent-k8s-lb.yaml
kubectl get services --all-namespaces
```

ensuite, copiez-collez l’adresse IP externe du service Portainer dans l’interface utilisateur, n’oubliez pas le port 9001

### .netcore sdk

[https://learn.microsoft.com/en-gb/dotnet/core/install/linux-ubuntu-install?tabs=dotnet8&pivots=os-linux-ubuntu-2204](https://learn.microsoft.com/en-gb/dotnet/core/install/linux-ubuntu-install?tabs=dotnet8&pivots=os-linux-ubuntu-2204)

```
sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-8.0 aspnetcore-runtime-8.0
```

ou si aspnet n’est pas attendu

```
sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-8.0 dotnet-runtime-8.0
```

Dépendances

Lorsque vous installez avec un gestionnaire de paquets, ces bibliothèques sont installées pour vous. Mais, si vous installez manuellement .NET ou si vous publiez une application auto-contenue, vous devrez vous assurer que ces bibliothèques sont installées :

- libc6
- libgcc-s1
- libgssapi-krb5-2
- libicu70
- liblttng-ust1
- libssl3
- libstdc++6
- libunwind8
- zlib1g

Les dépendances peuvent être installées avec la commande apt install. L’extrait suivant montre l’installation de la bibliothèque zlib1g :

### awscli

Pour installer awscli depuis le dépôt Ubuntu (en juillet 2024, cela vous donne v1.22.34-1) :

Pour vérifier la version d’awscli installée sur votre ordinateur :

Ou pour obtenir la dernière version fraîchement préparée par Amazon (2.0) : [https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

actuellement :

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
```

### Terraform

Terraform peut être installé de nombreuses façons, voir ici : [https://developer.hashicorp.com/terraform/install](https://developer.hashicorp.com/terraform/install)

J’installe depuis le dépôt de Hashicorp pour Ubuntu

```
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

Et pour Linux Mint, faites attention, l’installeur de Terraform ne fonctionne pas correctement. Vérifiez les sorties de ces commandes

```
lsb_release -cs

cat /etc/upstream-release/lsb-release 
```

Maintenant allez dans les Sources de logiciels et remplacez dans Hashicorp ‘virginia’ par ‘Jammy’ et exécutez à nouveau

```
sudo apt update && sudo apt install terraform
```

Pour vérifier que Terraform est installé correctement, exécutez

## Contrôle des services systemd

On peut utiliser

```
systemctl status
systemctl stop some-service
systemctl disable some-service 
```

ou utiliser chkservice

```
sudo apt-get install chkservice

sudo chkservice
```

## Liens utiles

- [Bash Cheat Sheet](https://www.glukhov.org/fr/developer-tools/terminals-shell/bash-cheat-sheet/)
- [Minio comme alternative à Aws S3. Aperçu et installation de Minio](https://www.glukhov.org/fr/data-infrastructure/object-storage/minio-vs-aws-s3/ "Minio comme alternative à Aws S3. Aperçu et installation de Minio")
- [Fiche de paramètres de ligne de commande MinIO](https://www.glukhov.org/fr/data-infrastructure/object-storage/minio-cheatsheet/ "Fiche de paramètres de ligne de commande MinIO")
- [Synchronisation des signets avec Floccus](https://www.glukhov.org/fr/post/2024/08/sync-bookmarks-floccus/)
- [Fiche de raccourcis Docker](https://www.glukhov.org/fr/developer-tools/containers/docker-cheatsheet/ "liste et description des commandes Docker les plus fréquentes et utiles - fiche de raccourcis Docker")
- [Fiche de raccourcis Ollama](https://www.glukhov.org/fr/llm-hosting/ollama/ollama-cheatsheet/ "Fiche de raccourcis Ollama")
- [Fiche de raccourcis Markdown](https://www.glukhov.org/fr/documentation-tools/markdown/markdown-cheatsheet/ "Fiche de raccourcis Markdown complète")
- [Outils de manipulation de PDF sous Ubuntu - Poppler](https://www.glukhov.org/fr/documentation-tools/pdf/ubuntu-poppler/ "Outils de manipulation de PDF sous Ubuntu - Poppler")
- [Installer Portainer sur Linux](https://www.glukhov.org/fr/developer-tools/containers/install-portainer-on-linux/ "Comment installer, se connecter et supprimer Portainer sur Linux")
- [Installer DBeaver sur Linux](https://www.glukhov.org/fr/developer-tools/database-tools/install-dbeaver-on-linux/ "Comment installer, se connecter et supprimer DBeaver sur Linux")
- [Comment démarrer des fenêtres de terminal en tuiles sous Linux Mint Ubuntu](https://www.glukhov.org/fr/developer-tools/terminals-shell/howto-start-terminal-windows-tiled-linux-mint-ubuntu/ "Comment démarrer des fenêtres de terminal en tuiles sous Linux Mint Ubuntu")
- [Déplacer les modèles Ollama vers un autre disque ou dossier](https://www.glukhov.org/fr/llm-hosting/ollama/move-ollama-models/ "Stocker les fichiers de modèles Ollama sur un autre disque")