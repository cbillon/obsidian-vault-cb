---
tags:
  - linux
---
La gestion des utilisateurs est une compétence centrale pour tout administrateur système Linux. Ce guide complet couvre la création, modification, sécurisation des comptes utilisateurs et la gestion des permissions.

#### Comprendre les Utilisateurs Linux

##### Types d'Utilisateurs

Linux distingue trois types d'utilisateurs :

**1. Root (UID 0)**

```bash
# Superutilisateur avec tous les privilèges
root:x:0:0:root:/root:/bin/bash
```

**2. Utilisateurs système (UID 1-999)**

```bash
# Pour services et daemons
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/false
```

**3. Utilisateurs normaux (UID >= 1000)**

```bash
# Utilisateurs humains
john:x:1000:1000:John Doe:/home/john:/bin/bash
```

##### Fichiers Système Clés

```bash
/etc/passwd       # Informations utilisateurs
/etc/shadow       # Mots de passe chiffrés
/etc/group        # Groupes
/etc/gshadow      # Mots de passe groupes (rarement utilisé)
/etc/sudoers      # Configuration sudo
/home/username    # Répertoire personnel
```

##### Format /etc/passwd

```bash
# Format : login:x:UID:GID:commentaire:home:shell
john:x:1000:1000:John Doe,,,:/home/john:/bin/bash
│    │ │    │    │             │           └─ Shell par défaut
│    │ │    │    │             └─ Répertoire home
│    │ │    │    └─ Commentaire (nom complet)
│    │ │    └─ GID (groupe principal)
│    │ └─ UID (identifiant unique)
│    └─ x = mot de passe dans /etc/shadow
└─ Login
```

##### Format /etc/shadow

```bash
# Format : login:password:lastchange:min:max:warn:inactive:expire
john:$6$xyz...:19000:0:99999:7:::
│    │          │     │ │     │ │ └─ Date expiration compte
│    │          │     │ │     │ └─ Jours inactivité avant disable
│    │          │     │ │     └─ Jours d'avertissement avant expiration
│    │          │     │ └─ Jours max avant changement obligatoire
│    │          │     └─ Jours min entre deux changements
│    │          └─ Dernier changement (jours depuis 1/1/1970)
│    └─ Hash mot de passe ($6$ = SHA-512)
└─ Login
```

---

#### Créer des Utilisateurs

##### useradd vs adduser

**useradd** : Commande bas niveau (toutes distributions) **adduser** : Script wrapper interactif (Debian/Ubuntu)

##### Créer Utilisateur avec useradd

**Syntaxe basique :**

```bash
# Création minimale
sudo useradd john

# Création complète avec options
sudo useradd -m -s /bin/bash -c "John Doe" -G sudo,docker john
```

**Options importantes :**

|Option|Description|Exemple|
|---|---|---|
|`-m`|Créer home directory|`useradd -m john`|
|`-d`|Spécifier home|`useradd -d /custom/home john`|
|`-s`|Définir shell|`useradd -s /bin/zsh john`|
|`-c`|Commentaire|`useradd -c "John Doe" john`|
|`-G`|Groupes secondaires|`useradd -G sudo,docker john`|
|`-u`|UID spécifique|`useradd -u 1500 john`|
|`-e`|Date expiration|`useradd -e 2026-12-31 john`|

**Exemple complet étape par étape :**

```bash
# 1. Créer utilisateur avec home et bash
sudo useradd -m -s /bin/bash -c "John Doe" john

# 2. Définir mot de passe
sudo passwd john
# Entrer le mot de passe deux fois

# 3. Vérifier création
id john
# uid=1001(john) gid=1001(john) groups=1001(john)

grep john /etc/passwd
# john:x:1001:1001:John Doe:/home/john:/bin/bash

# 4. Tester connexion
su - john
whoami
# john
```

##### Créer Utilisateur avec adduser (Debian/Ubuntu)

```bash
# Interface interactive (plus simple)
sudo adduser john

# Réponses aux questions :
# - Mot de passe (2 fois)
# - Nom complet
# - Numéro de bureau
# - Téléphone professionnel
# - Téléphone personnel
# - Autre
# - Confirmation

# adduser fait automatiquement :
# - Crée home directory
# - Copie fichiers de /etc/skel
# - Configure shell par défaut
# - Crée groupe du même nom
```

##### Configuration par Défaut

Fichier `/etc/default/useradd` :

```bash
# Voir configuration
sudo cat /etc/default/useradd

# Modifier defaults
sudo useradd -D -s /bin/zsh     # Shell par défaut
sudo useradd -D -b /users       # Home base directory
```

Fichier `/etc/login.defs` :

```bash
# UID/GID ranges
UID_MIN                  1000
UID_MAX                 60000
GID_MIN                  1000
GID_MAX                 60000

# Password aging
PASS_MAX_DAYS           99999
PASS_MIN_DAYS           0
PASS_WARN_AGE           7

# Home directory
CREATE_HOME             yes
UMASK                   077
```

---

#### Modifier des Utilisateurs

##### Commande usermod

```bash
# Changer shell
sudo usermod -s /bin/zsh john

# Changer home directory (et déplacer fichiers)
sudo usermod -d /new/home -m john

# Changer UID
sudo usermod -u 1500 john

# Changer commentaire
sudo usermod -c "John Smith" john

# Ajouter à des groupes (sans supprimer existants)
sudo usermod -aG sudo,docker john

# Remplacer tous les groupes
sudo usermod -G developers,testers john

# Verrouiller compte
sudo usermod -L john

# Déverrouiller compte
sudo usermod -U john

# Définir date expiration
sudo usermod -e 2026-12-31 john

# Désactiver date expiration
sudo usermod -e "" john
```

##### Changer Mot de Passe

```bash
# Changer son propre mot de passe
passwd

# Changer mot de passe d'un utilisateur (root)
sudo passwd john

# Forcer changement au prochain login
sudo passwd -e john

# Définir politique expiration mot de passe
sudo chage -M 90 john        # Max 90 jours
sudo chage -m 7 john         # Min 7 jours entre changements
sudo chage -W 14 john        # Avertir 14 jours avant expiration
sudo chage -I 30 john        # Inactif après 30 jours

# Voir politique
sudo chage -l john
```

##### Renommer Utilisateur

```bash
# Renommer (nécessite déconnexion utilisateur)
sudo usermod -l newname oldname

# Renommer home directory et groupe
sudo usermod -d /home/newname -m newname
sudo groupmod -n newname oldname

# Vérifier
id newname
```

---

#### Supprimer des Utilisateurs

##### Commande userdel

```bash
# Supprimer utilisateur (garde home)
sudo userdel john

# Supprimer utilisateur ET home directory
sudo userdel -r john

# Supprimer même si connecté (force)
sudo userdel -f john
```

##### Supprimer Proprement (Best Practice)

```bash
# 1. Vérifier si utilisateur connecté
who | grep john
ps -u john

# 2. Terminer processus
sudo pkill -u john

# 3. Archiver home avant suppression
sudo tar -czf /backups/john-home-$(date +%Y%m%d).tar.gz /home/john

# 4. Trouver fichiers appartenant à l'utilisateur
sudo find / -user john 2>/dev/null

# 5. Supprimer utilisateur et home
sudo userdel -r john

# 6. Vérifier suppression
id john  # Devrait retourner "no such user"
```

---

#### Gestion des Groupes

##### Comprendre les Groupes

Chaque utilisateur a :

- **Un groupe principal** (GID dans /etc/passwd)
- **Des groupes secondaires** (listés dans /etc/group)

##### Fichier /etc/group

```bash
# Format : nom:x:GID:membres
sudo:x:27:john,alice
docker:x:999:john,bob
developers:x:1100:john,alice,charlie
```

##### Créer Groupe

```bash
# Créer groupe
sudo groupadd developers

# Créer avec GID spécifique
sudo groupadd -g 1500 testers

# Vérifier
grep developers /etc/group
```

##### Ajouter Utilisateur à Groupe

```bash
# Ajouter aux groupes secondaires (sans supprimer existants)
sudo usermod -aG sudo john
sudo usermod -aG docker,developers john

# Remplacer tous les groupes secondaires
sudo usermod -G developers,testers john

# Alternative : gpasswd
sudo gpasswd -a john sudo

# Vérifier appartenance
groups john
id john
```

##### Retirer Utilisateur d'un Groupe

```bash
# Avec gpasswd
sudo gpasswd -d john docker

# Avec usermod (liste tous les groupes sauf celui à supprimer)
sudo usermod -G sudo,developers john  # docker retiré
```

##### Modifier/Supprimer Groupe

```bash
# Renommer groupe
sudo groupmod -n newname oldname

# Changer GID
sudo groupmod -g 1600 developers

# Supprimer groupe
sudo groupdel developers
# Note : impossible si c'est le groupe principal d'un utilisateur
```

---

#### Permissions Sudo

##### Qu'est-ce que Sudo ?

**sudo** (Super User DO) permet d'exécuter des commandes avec privilèges root de façon contrôlée.

##### Fichier /etc/sudoers

```bash
# JAMAIS éditer directement !
# Toujours utiliser visudo (vérifie syntaxe)
sudo visudo
```

##### Format /etc/sudoers

```bash
# Qui    Où       =(AsQui)  Quoi
john    ALL      =(ALL)     ALL
│       │        │          └─ Commandes autorisées
│       │        └─ Utilisateur cible (root par défaut)
│       └─ Hôtes (ALL = tous serveurs)
└─ Utilisateur/Groupe
```

##### Donner Accès Sudo Complet

**Méthode 1 : Ajouter au groupe sudo**

```bash
# Debian/Ubuntu
sudo usermod -aG sudo john

# RedHat/CentOS
sudo usermod -aG wheel john
```

**Méthode 2 : Fichier /etc/sudoers.d/**

```bash
# Créer fichier spécifique
sudo visudo -f /etc/sudoers.d/john

# Contenu
john ALL=(ALL:ALL) ALL
```

##### Accès Sudo Limité

**Permettre seulement certaines commandes :**

```bash
# john peut redémarrer nginx
john ALL=(ALL) NOPASSWD: /usr/sbin/service nginx restart

# alice peut gérer utilisateurs
alice ALL=(ALL) /usr/sbin/useradd, /usr/sbin/userdel, /usr/sbin/usermod

# bob peut tout sauf commandes dangereuses
bob ALL=(ALL) ALL, !/bin/su, !/usr/bin/passwd root
```

**Groupe avec commandes spécifiques :**

```bash
# Groupe developers peut redémarrer services
%developers ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart *
```

##### Options Sudo Utiles

```bash
# Exécuter sans mot de passe
john ALL=(ALL) NOPASSWD: ALL

# Exiger mot de passe
john ALL=(ALL) PASSWD: ALL

# Timeout session (défaut 15min)
Defaults timestamp_timeout=30

# Logger toutes commandes sudo
Defaults log_output
Defaults!/usr/bin/sudoreplay !log_output

# Alias pour simplifier
# Alias commandes
Cmnd_Alias SERVICES = /usr/bin/systemctl restart *, /usr/bin/systemctl status *
# Alias utilisateurs
User_Alias ADMINS = john, alice, bob
# Utilisation
ADMINS ALL=(ALL) SERVICES
```

##### Vérifier Accès Sudo

```bash
# Liste des commandes sudo autorisées
sudo -l

# Tester commande (dry-run)
sudo -v

# Historique sudo
sudo journalctl _COMM=sudo
```

Pour une analyse complète des logs sudo, consultez notre [guide sur l'analyse des logs avec journalctl](https://www.shpv.fr/blog/logs-linux-2026/).

---

#### Quotas Disque

##### Pourquoi des Quotas ?

Empêcher un utilisateur de remplir tout le disque.

##### Activer Quotas

**1. Installer outils**

```bash
sudo apt install quota quotatool   # Debian/Ubuntu
sudo yum install quota              # RedHat/CentOS
```

**2. Modifier /etc/fstab**

```bash
sudo nano /etc/fstab

# Ajouter usrquota,grpquota
/dev/sda1  /home  ext4  defaults,usrquota,grpquota  0  2
```

**3. Remonter partition**

```bash
sudo mount -o remount /home
```

**4. Créer fichiers quota**

```bash
sudo quotacheck -cugm /home
sudo quotaon /home
```

##### Définir Quotas

```bash
# Éditer quota utilisateur
sudo edquota john

# Interface vi s'ouvre :
Disk quotas for user john (uid 1001):
  Filesystem  blocks  soft  hard  inodes  soft  hard
  /dev/sda1   10000   90000 100000  100    0     0

# soft = avertissement
# hard = limite stricte
# blocks = espace (Ko)
# inodes = nombre de fichiers
```

**Définir via commande :**

```bash
# 500MB soft, 1GB hard
sudo setquota -u john 512000 1024000 0 0 /home

# Copier quota d'un user vers autre
sudo edquota -p john -u alice
```

##### Vérifier Quotas

```bash
# Quota utilisateur
quota john

# Tous les quotas
sudo repquota -a

# Quota d'un filesystem
sudo repquota /home

# Quotas par groupe
sudo repquota -g /home
```

##### Quotas par Groupe

```bash
# Éditer quota groupe
sudo edquota -g developers

# Définir
sudo setquota -g developers 5120000 10240000 0 0 /home
```

---

#### Audit et Sécurité

##### Lister Tous les Utilisateurs

```bash
# Tous les utilisateurs
cut -d: -f1 /etc/passwd

# Utilisateurs normaux (UID >= 1000)
awk -F: '$3 >= 1000 {print $1}' /etc/passwd

# Avec détails
cat /etc/passwd | column -t -s :
```

##### Trouver Utilisateurs sans Mot de Passe

```bash
# Utilisateurs avec mot de passe vide (DANGEREUX)
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# Comptes verrouillés
sudo awk -F: '($2 ~ /^!/) {print $1}' /etc/shadow
```

Si vous avez perdu votre mot de passe root, consultez notre guide pour [réinitialiser un mot de passe root perdu via GRUB](https://www.shpv.fr/blog/reset-password-linux-grub/).

##### Voir Dernières Connexions

```bash
# Dernières connexions
last

# Dernières connexions d'un utilisateur
last john

# Tentatives échouées
sudo lastb

# Utilisateurs actuellement connectés
who
w
```

##### Surveiller l'Activité Utilisateur

```bash
# Processus d'un utilisateur
ps -u john

# Connexions SSH actives
sudo netstat -tnpa | grep 'ESTABLISHED.*sshd'

# Historique commandes (bash)
sudo cat /home/john/.bash_history
```

Pour surveiller les changements de fichiers en temps réel, découvrez comment utiliser [inotifywait pour la surveillance de fichiers](https://www.shpv.fr/blog/inotify-alert/).

##### Détecter Comptes Suspects

```bash
# Utilisateurs sans home directory
awk -F: '$6 !~ /^\/home/ {print $1,$6}' /etc/passwd

# Utilisateurs avec UID 0 (doit être seulement root!)
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Comptes sans mot de passe
sudo awk -F: 'length($2) == 0 {print $1}' /etc/shadow

# Trouver fichiers SUID appartenant aux users
sudo find / -perm -4000 -user john 2>/dev/null
```

---

#### Scripts d'Administration

Pour des scripts d'administration plus avancés, découvrez nos [guides bash scripting pour sysadmin](https://www.shpv.fr/blog/bash-scripting-sysadmin/).

##### Script Création Utilisateur en Masse

```bash
#!/bin/bash
# create-users.sh

# Fichier CSV : username,fullname,groups
# john,John Doe,developers
# alice,Alice Smith,developers,sudo

CSV_FILE="users.csv"

while IFS=',' read -r username fullname groups; do
    # Ignorer ligne d'en-tête
    [[ "$username" == "username" ]] && continue

    echo "Creating user: $username"

    # Créer utilisateur
    if sudo useradd -m -s /bin/bash -c "$fullname" "$username"; then
        echo "User $username created successfully"

        # Définir mot de passe temporaire
        echo "$username:ChangeMe123!" | sudo chpasswd

        # Forcer changement mot de passe
        sudo passwd -e "$username"

        # Ajouter aux groupes
        if [[ -n "$groups" ]]; then
            IFS=',' read -ra GROUP_ARRAY <<< "$groups"
            for group in "${GROUP_ARRAY[@]}"; do
                sudo usermod -aG "$group" "$username"
            done
        fi

        echo "User $username configured"
    else
        echo "ERROR: Failed to create $username"
    fi

done < "$CSV_FILE"

echo "User creation completed"
```

##### Script Audit Utilisateurs

```bash
#!/bin/bash
# user-audit.sh

echo "=== User Audit Report ==="
echo "Date: $(date)"
echo ""

# Total utilisateurs
TOTAL=$(wc -l < /etc/passwd)
echo "Total accounts: $TOTAL"

# Utilisateurs normaux
NORMAL=$(awk -F: '$3 >= 1000 && $3 != 65534 {print $1}' /etc/passwd | wc -l)
echo "Normal users: $NORMAL"

# Utilisateurs avec sudo
echo ""
echo "=== Users with sudo access ==="
grep -E '^sudo|^wheel' /etc/group | cut -d: -f4

# Comptes sans mot de passe
echo ""
echo "=== Accounts without password (CRITICAL) ==="
sudo awk -F: '($2 == "" || $2 == "*") {print $1}' /etc/shadow

# Comptes avec UID 0
echo ""
echo "=== Accounts with UID 0 (should be only root) ==="
awk -F: '$3 == 0 {print $1}' /etc/passwd

# Dernières connexions
echo ""
echo "=== Last 10 logins ==="
last -n 10

# Utilisateurs jamais connectés
echo ""
echo "=== Users never logged in ==="
for user in $(awk -F: '$3 >= 1000 && $3 != 65534 {print $1}' /etc/passwd); do
    if ! last "$user" | grep -q "$user"; then
        echo "  - $user"
    fi
done
```

---

#### Bonnes Pratiques

##### Sécurité

✓ **Utiliser sudo plutôt que root direct**

```bash
# Désactiver login root SSH
sudo nano /etc/ssh/sshd_config
PermitRootLogin no
```

✓ **Politique mots de passe forte**

```bash
# Installer libpam-pwquality
sudo apt install libpam-pwquality

# /etc/security/pwquality.conf
minlen = 12
dcredit = -1    # Au moins 1 chiffre
ucredit = -1    # Au moins 1 majuscule
lcredit = -1    # Au moins 1 minuscule
ocredit = -1    # Au moins 1 caractère spécial
```

✓ **Expiration comptes temporaires**

```bash
# Compte expirant dans 30 jours
sudo useradd -e $(date -d "+30 days" +%Y-%m-%d) temp_user
```

✓ **Surveiller fichiers sensibles**

```bash
# Alertes sur modifications /etc/passwd, /etc/shadow
sudo apt install inotify-tools

# Script surveillance
#!/bin/bash
inotifywait -m /etc/passwd /etc/shadow -e modify |
while read path action file; do
    echo "ALERT: $file was modified" | mail -s "Security Alert" admin@example.com
done
```

##### Organisation

✓ **Nommage cohérent**

```bash
# Convention : prenom.nom
john.doe
alice.smith
```

✓ **Groupes par fonction**

```bash
developers
sysadmins
database-admins
app-operators
```

✓ **Documentation**

```bash
# Utiliser champ commentaire
sudo usermod -c "John Doe - Backend Developer - Team Alpha" john
```

##### Automatisation

✓ **Scripts onboarding/offboarding** ✓ **Audit régulier (cron weekly)** ✓ **Synchronisation LDAP/AD si entreprise**

---

#### Troubleshooting

##### Utilisateur ne peut pas se connecter

```bash
# 1. Vérifier compte existe
id username

# 2. Vérifier mot de passe
sudo passwd username  # Reset

# 3. Vérifier shell valide
grep username /etc/passwd
# Shell doit être dans /etc/shells

# 4. Vérifier home existe
ls -ld /home/username

# 5. Vérifier permissions home
sudo chmod 755 /home/username
sudo chown username:username /home/username

# 6. Vérifier compte pas verrouillé
sudo passwd -S username
# Doit afficher PS (Password Set), pas L (Locked)

# 7. Vérifier logs
sudo grep username /var/log/auth.log
```

##### Sudo ne fonctionne pas

```bash
# 1. Vérifier appartenance groupe sudo
groups username

# 2. Vérifier /etc/sudoers
sudo visudo -c  # Check syntax

# 3. Vérifier permissions /etc/sudoers
ls -l /etc/sudoers
# Doit être 440 (r--r-----)

# 4. Tester
sudo -l -U username
```

---

#### Conclusion

La gestion des utilisateurs Linux est au cœur de l'administration système. Points clés :

**Création utilisateurs :**

```bash
sudo useradd -m -s /bin/bash username
sudo passwd username
```

**Groupes et sudo :**

```bash
sudo usermod -aG sudo username
sudo visudo  # Configuration avancée
```

**Quotas :**

```bash
sudo setquota -u username 500000 1000000 0 0 /home
```

**Audit régulier :**

```bash
# Vérifier comptes suspects
# Surveiller connexions
# Nettoyer comptes inutilisés
```

Avec ces connaissances, vous maîtrisez la gestion complète des utilisateurs sous Linux !