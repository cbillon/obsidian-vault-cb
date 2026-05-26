---
tags:
  - linux
  - grub
  - repair
---
Il peut arriver que l'on perde l'accès à un compte root sur un système Linux. Heureusement, il est possible de **réinitialiser le mot de passe root directement depuis GRUB**, sans nécessiter de live CD. Pour une réparation système plus complète, découvrez [comment réparer avec chroot](https://www.shpv.fr/blog/reparer-linux-avec-chroot/). Voici comment procéder en toute sécurité.

---

#### ⚠️ Attention

Cette méthode donne un accès **complet au système** : elle doit être utilisée uniquement sur un serveur que vous êtes autorisé à administrer.

---

#### Étape 1 : Accéder au menu GRUB

Au démarrage de la machine, appuyez sur `Esc`, `Shift` ou `e` dès que GRUB s'affiche.

---

#### Étape 2 : Modifier l'entrée de démarrage

Une fois dans le menu GRUB :

1. Sélectionnez la ligne de démarrage (`Ubuntu` ou `Debian GNU/Linux`, etc.)
2. Appuyez sur la touche `e` pour éditer les paramètres du noyau

Repérez la ligne commençant par `linux` ou `linux16`, par exemple :

```bash
linux /boot/vmlinuz-... root=UUID=xxxx ro quiet splash
```

Remplacez `ro quiet splash` par :

```bash
rw init=/bin/bash
```

Ainsi, la ligne devient :

```bash
linux /boot/vmlinuz-... root=UUID=xxxx rw init=/bin/bash
```

Appuyez sur `Ctrl + X` ou `F10` pour démarrer avec ces paramètres.

#### Étape 3 : Réinitialiser le mot de passe root

Le système va démarrer directement sur un shell root, sans mot de passe.

Remontez la partition root en écriture (au cas où) :

```bash
mount -o remount,rw /
```

Puis changez le mot de passe :

```bash
passwd
```

Saisissez un nouveau mot de passe root. Si vous avez aussi besoin de modifier un utilisateur standard :

```bash
passwd nom_utilisateur
```

#### Étape 4 : Redémarrer le système

Une fois le mot de passe modifié, exécutez :

```bash
exec /sbin/init
```

Ou forcez un redémarrage :

```bash
reboot -f
```

Le système redémarrera normalement avec le nouveau mot de passe root.

##### Sécuriser GRUB après la récupération (optionnel)

Pour éviter que n'importe qui puisse accéder au shell root via GRUB :

Protégez GRUB par un mot de passe :

```bash
grub-mkpasswd-pbkdf2
```

Ajoutez la sortie à `/etc/grub.d/40_custom` :

```bash
set superusers="admin"
password_pbkdf2 admin <hash>
```

Mettez à jour GRUB :

```bash
update-grub
```

---

#### Conclusion

Cette méthode est extrêmement pratique pour restaurer l'accès root à une machine Linux sans support externe. En revanche, elle montre aussi qu'un accès physique = accès total. Protégez donc GRUB si la machine est exposée.