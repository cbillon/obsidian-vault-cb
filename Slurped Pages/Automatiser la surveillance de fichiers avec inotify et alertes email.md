---
link: https://www.shpv.fr/blog/inotify-alert/
byline: SHPV
site: SHPV
date: 2025-07-05T12:00
excerpt: Guide complet pour configurer inotify-tools sur Linux afin de détecter en temps réel les changements de fichiers et envoyer des notifications par email.
twitter: https://twitter.com/@SHPVFR
tags:
  - linux
  - file
  - automatisation
slurped: 2025-10-12T19:00
title: Automatiser la surveillance de fichiers avec inotify et alertes email
---

Publié le 5 juillet 2025

Administration

Surveillance

Automatisation

Surveiller automatiquement les changements dans des répertoires critiques est essentiel pour la sécurité et la maintenance. Avec **inotify-tools**, vous pouvez détecter en temps réel la création, suppression ou modification de fichiers et déclencher des **alertes email**.

## Prérequis

- Serveur Linux (Debian, Ubuntu, RHEL, CentOS)
- inotify-tools installé
- Un agent mail configuré (postfix, msmtp, etc.)

## Installation d’inotify-tools

```
sudo apt update
sudo apt install inotify-tools -y
```

## Script de surveillance

Créez un script bash `watch_and_alert.sh` :

```
#!/bin/bash

# Répertoire à surveiller
WATCH_DIR="/path/to/directory"

# Email de notification
ALERT_EMAIL="admin@example.com"

inotifywait -m -r -e create,modify,delete "$WATCH_DIR" --format '%T %w %f %e' --timefmt '%F %T' | while read TIMESTAMP DIR FILE EVENT; do
    echo -e "Sujet: Alerte inotify\n\nFichier: $DIR$FILE\nÉvénement: $EVENT\nDate: $TIMESTAMP" |     mail -s "Alerte inotify: $FILE $EVENT" "$ALERT_EMAIL"
done
```

Rendez-le exécutable :

```
chmod +x watch_and_alert.sh
```

## Exécution en arrière-plan

Utilisez `screen`, `tmux` ou un service systemd :

### Exemple service systemd :

Créez `/etc/systemd/system/inotify-alert.service` :

```
[Unit]
Description=Surveillance inotify avec alertes email

[Service]
ExecStart=/usr/local/bin/watch_and_alert.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

Activez et démarrez :

```
sudo systemctl daemon-reload
sudo systemctl enable --now inotify-alert.service
```

## Tests

Ajoutez, modifiez ou supprimez un fichier dans le répertoire surveillé et vérifiez la réception de l’email.

## Conclusion

Grâce à **inotify-tools** et un simple script bash, vous pouvez automatiser la surveillance de fichiers et recevoir des alertes instantanées, améliorant la réactivité et la sécurité de votre infrastructure.

###### Besoin d'aide sur ce sujet ?

Notre équipe d'experts est là pour vous accompagner dans vos projets.

[Contactez-nous](https://www.shpv.fr/demander-un-devis)

---

##### Articles similaires qui pourraient vous intéresser