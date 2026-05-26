---
tags:
  - ssh
  - security
---
La sécurité des accès SSH conditionne la résistance aux tentatives d'intrusion. En complément des [clés SSH et restrictions](https://www.shpv.fr/blog/ssh-securisation/), du filtrage IP et de la limitation de connexions, il est possible d'utiliser une technique appelée **Port Knocking**. Cette méthode consiste à garder le port SSH fermé et à ne l'ouvrir qu'après qu'un client a frappé une séquence spécifique de ports.

#### Plan de l'article

- Principe du port knocking
- Installation des outils nécessaires
- Configuration avec `iptables`
- Automatisation du client
- Bonnes pratiques de sécurité
- Conclusion

---

#### Principe du port knocking

Le serveur garde le port SSH (par ex. 22) **fermé par défaut**.  
Pour y accéder, le client doit envoyer une série de paquets sur des ports définis (ex. 7000, 8000, 9000).  
Lorsque la séquence est correcte, une règle du pare-feu ouvre temporairement le port SSH.

---

#### Installation des outils nécessaires

Sous Debian/Ubuntu :

```bash
sudo apt install knockd
```

Sous CentOS/RHEL :

```bash
sudo yum install knock-server
```

---

#### Configuration avec iptables

Exemple `/etc/knockd.conf` :

```conf
[options]
    logfile = /var/log/knockd.log

[openSSH]
    sequence    = 7000,8000,9000
    seq_timeout = 5
    command     = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn

[closeSSH]
    sequence    = 9000,8000,7000
    seq_timeout = 5
    command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
    tcpflags    = syn
```

Sur Debian/Ubuntu, activez knockd dans `/etc/default/knockd` :

```bash
START_KNOCKD=1
KNOCKD_OPTS="-i eth0"
```

Puis démarrez le service :

```bash
sudo systemctl enable knockd
sudo systemctl start knockd
```

---

#### Automatisation du client

Pour envoyer la séquence depuis un poste client :

```bash
knock serveur.exemple.com 7000 8000 9000
ssh utilisateur@serveur.exemple.com
```

---

#### Bonnes pratiques de sécurité

- Combiner le port knocking avec des **clés SSH**.
- Utiliser des séquences longues et non triviales.
- Surveiller les logs du serveur knockd.
- Coupler avec [Fail2ban](https://www.shpv.fr/blog/fail2ban/) pour plus de sécurité.

---

#### Conclusion

Le **Port Knocking** renforce la sécurité SSH en masquant le port d'accès et en réduisant drastiquement les tentatives de brute-force.  
C'est une méthode simple à mettre en place, qui complète efficacement les autres bonnes pratiques de sécurisation SSH.