---
link: https://colinfo.fr/installation-dansible-et-deploiement-de-docker-compose/
byline: Thomas COLIN
date: 2024-03-29T19:10
excerpt: Découvrez comment installer Ansible et déployer Docker Compose de manière dynamique sur vos hôtes distants pour une automatisation efficace des tâches informatiques. Grâce à Ansible, un moteur d'automatisation open source, simplifiez la gestion, la configuration et le déploiement d'applications, tout en bénéficiant d'avantages tels que la simplicité, la flexibilité et l'absence d'agents. Automatisez vos processus avec facilité et fiabilité.
slurped: 2025-02-28T18:38
title: Installation d'ansible et déploiement de docker compose -
tags:
  - docker
  - ansible
  - install
---

![](https://colinfo.fr/wp-content/uploads/2024/03/deploiement-de-docker-compose-avec-ansible-2.png)

- [Introduction](https://colinfo.fr/installation-dansible-et-deploiement-de-docker-compose/#Introduction "Introduction")
- [Installation de ansible](https://colinfo.fr/installation-dansible-et-deploiement-de-docker-compose/#Installation_de_ansible "Installation de ansible")
- [Installation de docker compose avec ansible](https://colinfo.fr/installation-dansible-et-deploiement-de-docker-compose/#Installation_de_docker_compose_avec_ansible "Installation de docker compose avec ansible")
- [Conclusion](https://colinfo.fr/installation-dansible-et-deploiement-de-docker-compose/#Conclusion "Conclusion")

## Introduction

Dans cet article nous allons voir comment installer ansible et déployer dynamiquement docker compose sur nos hôtes distants.

Ansible est un moteur d’automatisation informatique open source, qui permet d’éliminer les tâches répétitives de votre vie professionnelle et améliorer également l’évolutivité, la cohérence et la fiabilité de votre environnement informatique.

Ansible permet d’automatiser trois types de tâches :  
• Provisioning : configurer les différents serveurs dont vous avez besoin dans votre infrastructure.  
• Configuration : modifier la configuration d’une application, d’un système d’exploitation ou d’un périphérique, démarrer et arrêter les services, installer ou mettre à jour des applications, mettre en œuvre une politique de sécurité, ou effectuer une grande variété d’autres tâches de configuration.  
• Déploiement d’applications : adopter une démarche DevOps en automatisant le déploiement d’applications développées en interne sur vos environnements de production.

Voici une liste des avantages d’Ansible :  
• Gratuit : Ansible est un outil open source.  
• Simple : Ansible utilise une syntaxe simple écrite en YAML. Aucune compétence en programmation particulière n’est nécessaire pour créer les playbooks d’Ansible est également simple à installer.  
• Flexible : il existe des centaines de modules prêts à l’emploi pour gérer vos tâches, quel que soit l’endroit où ils sont déployés. Vous pouvez réutiliser le même playbook sur un parc de machines Red Hat, Ubuntu ou autres.  
• Agentless : Pas besoin d’installer d’autres logiciels ou d’ouvrir des ports de parefeu supplémentaires sur les systèmes clients que vous souhaitez automatiser. Ansible utilise la connexion SSH.

Ansible est inclus par défaut dans les repositories de Debian. Nous allons installer le binaire ansible.

```
sudo apt update

sudo apt install ansible
```

On peut vérifier l’installation en effectuant la commande ansible –version

![Une image contenant texte, capture d’écran, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-43.png)

## Installation de docker compose avec ansible

Je vais créer un un répertoire ansible qui contiendra tous mes fichiers de configuration.

```
mkdir ansible
```

Je vais créer un fichier d’inventaire nommé inventory.ini au format IN.  
Il contiendra 2 groupes, [LAN] pour les serveurs dans le LAN et [DMZ] pour l’unique machine dans la DMZ.  

```
[lan] 
docker-app 
docker-sec 
[dmz] 
docker-rp
```

On peut tester notre première commande sur ce fichier d’inventaire :

```
ansible -i inventory.ini -m ping all
```

Les hôtes répondent correctement à la demande de ping depuis ansible

![Une image contenant texte, capture d’écran, Police
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-police-44.png)

Je crée ensuite un nouveau dossier docker qui contiendra mon playbook

```
mkdir docker
cd docker
```

  
Nous allons créer un playbook pour installer docker compose sur tous nos hôtes

```
nano docker.yml
```

```
---

- hosts: all

become: true

tasks:

## installation de docker

- name: mettre a jour la machine distante

apt:

update_cache: true

- name: installation des pre-requis

ansible.builtin.package:

name:

- apt-transport-https

- ca-certificates

- curl

- gnupg2

- software-properties-common

state: present

- name: recuperer la cle gpg pour de le depot de docker

apt_key:

url: https://download.docker.com/linux/debian/gpg

state: present

- name: installation du repository de docker

apt_repository:

repo: "deb https://download.docker.com/linux/debian {{ ansible_distribution_release }} stable"

update_cache: true

state: present

- name: installation de docker

ansible.builtin.package:

name:

- docker-ce

- docker-ce-cli

- containerd.io

state: present

- name: activer le service docker au demarrage

ansible.builtin.service:

name: docker

state: started

enabled: yes

## installation de docker compose

- name: installation de docker compose

ansible.builtin.package:

name:

- docker-compose-plugin
```

  
Voici la commande pour lancer le playbook depuis le dossier ansible :

```
ansible-playbook -i inventory.ini docker/docker.yml
```

Nous pouvons rapidement visualiser et constater que l’installation de docker et docker compose s’est correctement réalisé

![Une image contenant texte, capture d’écran, ligne
Description générée automatiquement](https://colinfo.fr/wp-content/uploads/2024/03/une-image-contenant-texte-capture-decran-ligne.png)

## Conclusion

L’automatisation des tâches informatiques est devenue un pilier essentiel pour les entreprises cherchant à optimiser leur infrastructure. Nous avons vu à travers l’automatisation de l’installation de docker compose avec Ansible que cette solution se positionne comme un moteur d’automatisation puissant et flexible.