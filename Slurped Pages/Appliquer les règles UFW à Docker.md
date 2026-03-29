---
link: https://antoinebonin.fr/Ecriture/Appliquer-les-r%C3%A8gles-UFW-%C3%A0-Docker
site: Antoine Bonin
excerpt: Vous utilisez Uncomplicated Firewall (UFW) et vous avez réalisé que vos conteneurs dockers sont quand même accessibles sur internet.
slurped: 2026-03-28T16:58
title: Appliquer les règles UFW à Docker
tags:
  - docker
  - ufw
---

Vous utilisez Uncomplicated Firewall (UFW) et vous avez réalisé que vos conteneurs dockers sont quand même accessibles sur internet. Au début vous pensez à une mauvaise configuration ou UFW qui n’est pas activé. Puis vous vous rendez compte que si.

Ici je vais vous expliquer pourquoi et surtout comment remédier a ce problème.

## Pourquoi UFW ne fonctionne pas avec Docker ?

Pour faire simple, votre serveur possède un logiciel [iptables](https://fr.wikipedia.org/wiki/Iptables) qui permet de configurer les règles de pare-feu du serveur directement dans le kernel. Pour les utilisateurs avancés, ils peuvent directement faire les modifications dedans, mais pour nous autres des outils comme UFW nous permettent de définir des règles plus simplement.

**Mais où est le problème ?**

Et bien, comme UFW, Docker utilise iptables pour configurer les ports des conteneurs que l’on spécifie (Ex: `-p 80:80`). Les règles que Docker appliquent peuvent passer à travers celle de UFW.

Il y a plusieurs manières de corriger ce comportement. Nous allons voir les 3 solutions, l’avantage et les inconvénients.

### Forcer Docker à ne plus modifier iptables

La première solution est assez simple a mettre en place et à comprendre, mais la plus compliquée à maintenir.

Dans un premier temps, on va configurer Docker pour ne pas modifier les iptables:

```
echo "{\"iptables\": false}" > /etc/docker/daemon.json
```

On redémarre Docker pour appliquer la configuration.

```
sudo service docker restart
```

En suite, on configure UFW pour qu’il transfère les connexions externes vers l’interface interne au lieu de juste les couper. Et on recharge la configuration d’UFW pour appliquer les changements:

```
sudo sed -i -e 's/DEFAULT_FORWARD_POLICY="DROP"/DEFAULT_FORWARD_POLICY="ACCEPT"/g' /etc/default/ufw
sudo ufw reload
```

Enfin, on doit ajouter une règle à l’iptables pour transférer les connexions vers le réseau de Docker.

```
iptables -t nat -A POSTROUTING ! -o docker0 -s 172.17.0.0/16 -j MASQUERADE
```

Pour résumer, toutes les connexions externes vont passer par UFW avant d’être transférées sur le réseau de Docker. Faites attention à bien mettre la bonne adresse de Docker, dans mon cas: `172.17.0.0/16`. Si celle-ci change, vous devez refaire la dernière étape.

### La solution la plus simple: ufw-docker

Comme il s’agit d’un problème courant et connu, Chai Feng a développé un [outil](https://github.com/chaifeng/ufw-docker) pour le résoudre. Le processus d’installation et son utilisation sont bien documentés dans le repository GitHub, mais pour les non anglophones, je vais quand même montrer l’installation.

> Avant d’installer ufw-docker, il est nécessaire de vérifier que la fonctionnalité `iptables` de Docker est activée. Si vous l’avez désactivée en ajoutant `--iptables=false` au fichier `/etc/docker/daemon.json`, supprimez cette ligne du fichier et redémarrez le service Docker.

Dans un premier temps, copiez le script `ufw-docker` dans le répertoire `/usr/local/bin/ufw-docker`. Vous pouvez le trouver en ligne à l’adresse : [https://github.com/chaifeng/ufw-docker/raw/master/ufw-docker](https://github.com/chaifeng/ufw-docker/raw/master/ufw-docker). Assurez-vous qu’il est exécutable, et voilà !

```
sudo wget -O /usr/local/bin/ufw-docker https://github.com/chaifeng/ufw-docker/raw/master/ufw-docker
chmod +x /usr/local/bin/ufw-docker
```

Il nous reste plus qu’à exécuter le script et redémarrer UFW:

```
ufw-docker install
sudo ufw reload
```

Vous pouvez maintenant ajouter ou supprimer des règles avec ufw-docker:

```
ufw-docker allow <container_name> <port>
ufw-docker delete allow <container_name> <port>
```

### Utiliser un pare-feu externe

**Il s’agit d’un conseil plutôt que d’une solution.**

Cela dit, je recommande d’utiliser un pare-feu externe à l’hôte exécutant Docker. Les fournisseurs de cloud fournissent généralement des pare-feu avec les offres de serveur. Cela à l’avantage de réduire le temps et la complexité de configuration du serveur exécutant Docker et éliminera les possibles erreurs sur le chemin.

## Sources

- Jarrousse - [How to use UFW firewall with Docker](https://blog.jarrousse.org/2023/03/18/how-to-use-ufw-firewall-with-docker-containers/)
- Chai Feng - [ufw-docker](https://github.com/chaifeng/ufw-docker)