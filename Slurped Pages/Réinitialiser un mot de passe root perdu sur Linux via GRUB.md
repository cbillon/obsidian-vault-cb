---
link: https://www.shpv.fr/blog/reset-password-linux-grub/
byline: SHPV
site: SHPV
date: 2025-06-20T12:00
excerpt: Un guide simple et rapide pour réinitialiser le mot de passe root d’un système Linux depuis le bootloader GRUB, sans live CD.
twitter: https://twitter.com/@SHPVFR
tags:
  - linux
  - grub
slurped: 2025-10-12T19:11
title: Réinitialiser un mot de passe root perdu sur Linux via GRUB
---

Il peut arriver que l’on perde l’accès à un compte root sur un système Linux. Heureusement, il est possible de **réinitialiser le mot de passe root directement depuis GRUB**, sans nécessiter de live CD. Voici comment procéder en toute sécurité.

---

## ⚠️ Attention

Cette méthode donne un accès **complet au système** : elle doit être utilisée uniquement sur un serveur que vous êtes autorisé à administrer.

---

## Étape 1 : Accéder au menu GRUB

Au démarrage de la machine, appuyez sur `Esc`, `Shift` ou `e` dès que GRUB s’affiche.

---

## Étape 2 : Modifier l’entrée de démarrage

Une fois dans le menu GRUB :

1. Sélectionnez la ligne de démarrage (`Ubuntu` ou `Debian GNU/Linux`, etc.)
2. Appuyez sur la touche `e` pour éditer les paramètres du noyau

Repérez la ligne commençant par `linux` ou `linux16`, par exemple :

```
linux /boot/vmlinuz-... root=UUID=xxxx ro quiet splash
```

Remplacez ro quiet splash par :

```
rw init=/bin/bash
```

Ainsi, la ligne devient :

```
linux /boot/vmlinuz-... root=UUID=xxxx rw init=/bin/bash
```

Appuyez sur Ctrl + X ou F10 pour démarrer avec ces paramètres.

## Étape 3 : Réinitialiser le mot de passe root

Le système va démarrer directement sur un shell root, sans mot de passe.

Remontez la partition root en écriture (au cas où) :

```
mount -o remount,rw /
```

Puis changez le mot de passe :

```
passwd
```

Saisissez un nouveau mot de passe root. Si vous avez aussi besoin de modifier un utilisateur standard :

```
passwd nom_utilisateur
```

## Étape 4 : Redémarrer le système

Une fois le mot de passe modifié, exécutez :

```
exec /sbin/init
```

Ou forcez un redémarrage :

```
reboot -f
```

Le système redémarrera normalement avec le nouveau mot de passe root.

### Sécuriser GRUB après la récupération (optionnel)

Pour éviter que n’importe qui puisse accéder au shell root via GRUB :

Protégez GRUB par un mot de passe :

```
grub-mkpasswd-pbkdf2
```

Ajoutez la sortie à /etc/grub.d/40_custom :

```
set superusers="admin"
password_pbkdf2 admin <hash>
```

Mettez à jour GRUB :

```
update-grub
```

---

## Conclusion

Cette méthode est extrêmement pratique pour restaurer l’accès root à une machine Linux sans support externe. En revanche, elle montre aussi qu’un accès physique = accès total. Protégez donc GRUB si la machine est exposée.

---

## Ressources complémentaires

- [Documentation Ubuntu GRUB](https://help.ubuntu.com/community/Grub2)
- [grub-mkpasswd-pbkdf2](https://www.gnu.org/software/grub/manual/grub/html_node/Authentication.html)
- [Restauration root Debian](https://wiki.debian.org/fr/RecoveringRootPassword)