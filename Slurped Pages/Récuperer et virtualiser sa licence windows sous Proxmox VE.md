---
link: https://www.emaxilde.net/posts/2022/01/28/recuperer-et-virtualiser-sa-licence-windows-sous-proxmox-ve.html
byline: Olivier Poncet
site: Olivier Poncet
date: 2022-01-28T16:30
excerpt: Si comme moi vous êtes sur PC et que vous n’êtes pas utilisateur/trice
  de Windows, il arrive parfois que vous ayez besoin d’utiliser un logiciel
  fonctionnement exclusivement sous le système d’exploitation de Microsoft. Cela
  peut être par exemple un logiciel d’accès VPN propriétaire ou bien des
  logiciels métiers très spécifiques. Plutôt que de racheter à vil prix une
  licence Windows, je vais vous montrer comment récupérer la licence inutilisée
  de votre machine et comment l’exploiter dans une machine virtuelle sous
  Proxmox VE, sachant que c’est aussi applicable à d’autres solutions de
  virtualisation.
slurped: 2024-06-06T16:50:44.205Z
title: Récuperer et virtualiser sa licence windows sous Proxmox VE
---

Si comme moi vous êtes sur PC et que vous n’êtes pas utilisateur/trice de Windows, il arrive parfois que vous ayez besoin d’utiliser un logiciel fonctionnement exclusivement sous le système d’exploitation de Microsoft. Cela peut être par exemple un logiciel d’accès VPN propriétaire ou bien des logiciels métiers très spécifiques. Plutôt que de racheter à vil prix une licence Windows, je vais vous montrer comment récupérer la licence inutilisée de votre machine et comment l’exploiter dans une machine virtuelle sous Proxmox VE, sachant que c’est aussi applicable à d’autres solutions de virtualisation.

### Disclaimer

Cet article va exposer une méthode de récupération et d’exploitation de votre licence Windows **inutilisée**. J’insiste lourdement sur le point « **non utilisée** ». Je suis contre le piratage de façon générale et ce HOWTO n’a pas vocation à encourager ce type d’activité, mais a vocation de vous retirer le caillou qui est dans votre chaussure et exploiter une licence que vous avez déjà payée et que vous n’exploitez pas.

D’un point de vue légal, l’immense majorité des licences Windows préinstallées dans les PC lors de l’achat sont des licences [OEM](https://fr.wikipedia.org/wiki/Fabricant_d%27%C3%A9quipement_d%27origine), négociées en volume entre les constructeurs et Microsoft. Elles vous octroient un droit d’utilisation du système d’exploitation sur votre machine et d’après le CLUF (Contrat de Licence Utilisateur Final), cette licence n’a normalement pas vocation à être utilisée sur une autre machine que celle sur laquelle elle a été installée.

Cependant vous pouvez constater que depuis un moment, de nombreux sites/revendeurs proposent des licences Windows à prix cassés voire ultra-cassés. Sachez que la plupart du temps, ces licences sont aussi des licences OEM récupérées depuis des machines destinées au recyclage. Microsoft n’ayant pas de position très claire par rapport à cette pratique, je considère donc (peut-être à tort) que je peux m’octroyer le droit de récupérer **la licence OEM inutilisée de mon PC** et de l’exploiter comme bon me semble.

Petit ajout à la publication de mon article : l’excellent [Wassim Chegham](https://twitter.com/manekinekko) m’a pointé une mention liée à la virtualisation d’une licence OEM dans le [texte de la licence](https://www.microsoft.com/en-us/Useterms/OEM/Windows/10/UseTerms_OEM_Windows_10_FRENCH.htm) :

> 2. Droits d’installation et d’utilisation.
> 
> […]
> 
> d. Utilisation par plusieurs utilisateurs.
> 
> […]
> 
> (iv) **Utilisation dans un environnement virtualisé**. La présente licence vous autorise à installer **une seule instance** du logiciel sur un dispositif, qu’il soit physique ou virtuel. Pour pouvoir utiliser le logiciel sur plusieurs dispositifs virtuels, vous devez disposer d’une licence distincte pour chaque instance.
> 
> […]

Voilà ! Ceci étant clairement dit, nous pouvons désormais passer aux choses sérieuses.

### Un peu de contexte

Si vous me connaissez déjà ou bien que vous me suivez sur les réseaux sociaux, je suis un utilisateur et défenseur des logiciels libres. L’intégralité de mes machines, serveurs, ordinateurs professionnels ou personnels fonctionnent quasi-exclusivent sous Linux et un petit peu de FreeBSD, aucun ordinateur fonctionnant sous Windows ni MacOS-X. Mais ne vous méprenez pas, je ne suis pas un Ayatollah du libre, chacun fait comme il veut.

Il m’arrive parfois, de façon très ponctuelle, de me retrouver dans une situation où je dois composer avec un logiciel fonctionnant exclusivement sous Windows et n’ayant aucune alternative en termes d’interopérabilité. J’ai eu le cas avec des logiciels de VPN propriétaires par exemple, ou bien des logiciels opérationnels très spécifiques, et ne fonctionnant pas non plus sous [Wine](https://www.winehq.org/). Bref, on pourrait débattre pendant des heures voire des années sur ces sujets d’intéropérabilité, mais ce n’est pas l’objet du présent billet.

Lorsque j’ai acquis mon laptop, j’ai naturellement écrasé le Windows pré-installé par une distribution [Linux Mint](https://www.linuxmint.com/) mais je n’ai malheureusement pas réussi à obtenir le remboursement de cette licence Windows par le constructeur, et ce malgré mes demandes … Bref, j’ai toujours envie de crier « [halte aux racketiciels](https://non.aux.racketiciels.info/) » !!

Bref, j’ai donc techniquement une licence Windows dormante que je n’utilise pas et qui peut me servir dans ces très rares cas où je suis dans l’impossibilité de contourner facilement une contrainte technique. Mais il reste cette question à 100 balles : comment récupérer ce numéro de licence qui m’a été facturé et dont je pourrais me servir, sachant que j’ai nettoyé toutes traces de présence du Windows ?

Toutes traces ? Vraiment ? Peut-être pas …

### Fasten your seatbelt

Vous l’ignorez peut-être ou bien peut-être pas, mais lorsqu’une machine est livrée avec une version OEM de Windows, le numéro de licence se trouve inscrit dans les entrailles du système d’exploitation, mais pas que ! Ce numéro est aussi inscrit dans les tréfonds de votre matériel et réside dans une table spécifique du sous-système [ACPI](https://fr.wikipedia.org/wiki/Advanced_Configuration_and_Power_Interface) de l’ordinateur, acronyme de « Advanced Configuration and Power Interface ».

Dans ce sous-système, il existe plusieurs « tables » de configuration de votre matériel, dont deux qui nous intéressent tout particulièrement :

- Une table nommée `SLIC`, pour **S**oftware **LIC**encing.
- Une table nommée `MSDM`, pour **M**icro**S**oft **D**ata **M**anagement.

La table `SLIC` définit les informations nécessaires à la mise en place de l’activation OEM générique. Elle contient les informations relatives au licensing global du constructeur.

```
+------------------------------+--------+--------+---------------------------------------------------------------------------------------------------------------+
| Field                        | Length | Offset | Description                                                                                                   |
+------------------------------+--------+--------+---------------------------------------------------------------------------------------------------------------+
| Signature                    | 4      | 0      | SLIC                                                                                                          |
| Length                       | 4      | 4      | Length, in bytes, of the entire table. The length implies the number of entry fields at the end of the table. |
| Revision                     | 1      | 8      | 0x01                                                                                                          |
| Checksum                     | 1      | 9      | Checksum of the entire table.                                                                                 |
| OEMID                        | 6      | 10     | An OEM-supplied string that identifies the OEM; must match OEMID in the ACPI RSDT and XSDT tables.            |
| OEM Table ID                 | 8      | 16     | Optional motherboard/BIOS logical identifier; must match OEM Table ID in the ACPI RSDT and XSDT tables.       |
| OEM Revision                 | 4      | 24     | OEM revision number of the table for the supplied OEM Table ID.                                               |
| Creator ID                   | 4      | 28     | Vendor ID of the utility that created the table.                                                              |
| Creator Revision             | 4      | 32     | Revision of the utility that created the table.                                                               |
| Software Licensing Structure | Var.   | 36     | Proprietary data structure that contains all the licensing data necessary to enable Windows activation.       |
+------------------------------+--------+--------+---------------------------------------------------------------------------------------------------------------+
```

La table `MSDM` définit les informations nécessaires à l’activation OEM individualisée. Elle contient les informations relatives au licensing spécifique de la machine, dont le numéro de licence qui nous intéresse.

```
+------------------------------+--------+--------+---------------------------------------------------------------------------------------------------------------+
| Field                        | Length | Offset | Description                                                                                                   |
+------------------------------+--------+--------+---------------------------------------------------------------------------------------------------------------+
| Signature                    | 4      | 0      | MSDM                                                                                                          |
| Length                       | 4      | 4      | Length, in bytes, of the entire table.                                                                        |
| Revision                     | 1      | 8      | 0x01                                                                                                          |
| Checksum                     | 1      | 9      | Checksum of the entire table.                                                                                 |
| OEMID                        | 6      | 10     | An OEM-supplied string that identifies the OEM.                                                               |
| OEM Table ID                 | 8      | 16     | Optional motherboard/BIOS logical identifier.                                                                 |
| OEM Revision                 | 4      | 24     | OEM revision number of the table for the supplied OEM Table ID.                                               |
| Creator ID                   | 4      | 28     | Vendor ID of the utility that created the table.                                                              |
| Creator Revision             | 4      | 32     | Revision of the utility that created the table.                                                               |
| Software Licensing Structure | Var.   | 36     | Proprietary data structure that contains all the licensing data necessary to enable Windows activation.       |
+------------------------------+--------+--------+---------------------------------------------------------------------------------------------------------------+
```

Pour notre manipulation, nous aurons seulement besoin de la seconde table, la `MSDM`. Cependant, si ce que nous nous apprêtons à faire devait échouer au niveau de l’activation de Windows, nous pourrions aussi tenter avec les deux tables en même temps, `SLIC` et `MSDM`.

### Allons à la pêche aux données

Nous aurons besoin des données uniques de la machine. Nous allons donc les extraire avec la commande `dmidecode` qui permet d’obtenir toutes les données `DMI` (norme [Desktop Management Interface](https://fr.wikipedia.org/wiki/Desktop_management_interface)) de l’ordinateur. Cette commande a besoin des droits superutilisateur, donc soit vous êtes déjà `root`, soit vous utilisez `sudo`.

```
user@host:~$ sudo dmidecode | grep -A8 "System Information" | tee dmidecode.txt
```

Vous devriez obtenir quelque-chose ressemblant à ceci à l’écran ainsi que dans le fichier `dmidecode.txt` :

```
System Information
	Manufacturer: <fabricant du PC>
	Product Name: <modèle de PC>
	Version: <version du modèle, si applicable>
	Serial Number: <numéro de série du PC>
	UUID: <numéro unique de la machine>
	Wake-up Type: Power Switch
	SKU Number: <unité de gestion de stock>
	Family: <famille de l'odinateur>
```

Gardez précieusement ces données quelque-part, nous en aurons besoin un peu plus tard lors de la création de la machine virtuelle.

Nous allons maintenant extraire la table `SLIC`. A priori elle ne devrait pas nous servir (je n’en n’ai jamais eu besoin en tout cas), mais autant l’avoir sous la main au cas où.

```
user@host:~$ sudo cat /sys/firmware/acpi/tables/SLIC > SLIC.bin
```

Pour terminer, nous allons extraire la table `MSDM`. C’est elle qui nous permettra d’installer Windows sans qu’il ne nous demande un quelconque numéro de licence.

```
user@host:~$ sudo cat /sys/firmware/acpi/tables/MSDM > MSDM.bin
```

#### Transfert des tables sur le nœud Proxmox VE

Une fois les tables extraites, vous devez maintenant les transférer sur votre nœud Proxmox VE.

Je vous conseille de les placer à un endroit logique et facile d’accès. Pour ma part, j’ai décidé de créer un sous-répertoire `acpi` dans le répertoire `/var/lib/vz` qui est traditionnellement l’endroit où sont stockées les données des machines virtuelles dans l’hyperviseur.

Connectez-vous par `sftp` à votre nœud Proxmox VE :

```
user@host:~$ sftp root@{nom-ou-adresse-ip-de-la-machine}
```

Puis passez les commandes suivantes :

```
cd /var/lib/vz
mkdir acpi
cd acpi
put SLIC.bin
put MSDM.bin
exit
```

### Préparation de la machine virtuelle

Les étapes précédentes étant réalisées, nous allons maintenant créer la machine virtuelle. Notez que je vais prendre le parti d’utiliser des périphériques `VirtIO` pour des raisons assez évidentes de performances.

- Téléchargez l’image ISO correspondant au CD d’installation de Windows depuis [le site internet de Microsoft](https://www.microsoft.com/fr-fr/software-download/windows10ISO).
- Téléchargez l’image ISO correspondant aux pilotes `VirtIO` pour Windows depuis [le site internet de Fedora](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/?C=M;O=D).

Une fois les images ISO téléchargées, cliquez sur le bouton `Créer VM` et laissez vous guider …

#### Étape 1

Renseignez le champ `VM ID` ainsi que le champ `Nom`.

![Étape 1](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-01.png#thumbnail)

#### Étape 2

Renseignez le champ `Image ISO` en sélectionnant l’image ISO d’installation de Windows. Configurez ensuite le type d’OS invité en positionnant le champ `Type` à `Microsoft Windows` puis le champ `Version` à `10/2016/2019`.

![Étape 2](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-02.png#thumbnail)

#### Étape 3

Renseignez le champ `Carte graphique` ; pour ma part j’utilise ici `SPICE` car cela permet de rediriger l’affichage, les ports USB, l’audio, … et donc d’avoir un véritable desktop distant de façon transparente ; cela implique évidemment que vous ayez à disposition un client spice installé sur votre machine. Si ce n’est pas le cas, choisissez autre chose. Puis séléctionnez le contrôleur SCSI en le positionnant sur `VirtIO SCSI`. Enfin cochez la case `Agent Qemu`.

![Étape 3](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-03.png#thumbnail)

#### Étape 4

Configurez le disque virtuel en positionnant `Bus/Device` sur `SCSI`, puis sélectionnez la destination du disque virtuel (ici `local-zfs`), la taille (je vous recommande au moins 64Gio), enfin cochez les cases `Discard` et `Émulation de SSD` (si vous le souhaitez et si applicable pour votre hyperviseur).

![Étape 4](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-04.png#thumbnail)

#### Étape 5

Configurez la partie CPU. Ici, pas trop de mystères, mettez ce que vous voulez ou bien ce que vous pouvez. Je dirais qu’une configuration basico-basique de type `1 socket` / `4 cœurs` fait largement le boulot, mais encore une fois, cela dépend du hardware de votre hyperviseur et je vous conseillerais de ne pas trop vous écarter des spécifications de la machine d’origine, la validation des licences Windows par les serveurs de Microsoft reposant à priori en partie dessus.

![Étape 5](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-05.png#thumbnail)

#### Étape 6

Configurez ici la mémoire de votre machine virtuelle en renseignant le champ `Mémoire` exprimé en Mio. Un bon 4Gio des familles ira très bien même si la machine d’origine sortie d’usine possèdait 8Gio. La quantité de mémoire n’influe à priori pas beaucoup sur la validation ou l’invalidation de la licence Windows. Activez la case à cocher `Périphérique de Balloning` ce qui permettra de gérer dynamiquement les besoins en mémoire de la machine virtuelle.

![Étape 6](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-06.png#thumbnail)

#### Étape 7

Configurez votre périphérique réseau. Sélectionnez le `bridge` correspondant à votre réseau interne (ici `vmbr1` sur mon hyperviseur), puis sélectionnez le modèle de carte réseau en le positionnant sur `VirtIO`. Enfin, cochez la case `Déconnecter`, c’est important pour l’installation. Optionnellement vous pouvez cocher ou décocher la case `Parefeu` en fonction de vos besoins ou exigences.

![Étape 7](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-07.png#thumbnail)

#### Étape 8

Nous voici sur la page récapitulant les caractéristiques de la machine virtuelle. Vous n’avez donc qu’à tout vérifier et si tout est bon pour vous, cliquez sur `Terminer` afin de créer effectivement la machine virtuelle.

![Étape 8](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-08.png#thumbnail)

#### Étape 9

La machine virtuelle créée, rendez-vous sur l’onglet `Matériel`. Ajoutez un second lecteur CD/DVD sur le port `ide3` et chargez-y l’image `virtio-win-x.x.xxx.iso` précédemment téléchargée. Si vous utilisez un client `SPICE` comme moi, vous pouvez aussi ajouter un périphérique audio ainsi qu’un périphérique USB en fonction de vos usages futurs.

![Étape 9](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-09.png#thumbnail)

#### Étape 10

Rendez-vous ensuite sur l’onglet `Options`. Vérifiez et ajustez les paramètres de la machine virtuelle si nécessaire, par exemple `Ordre de boot`, `Protection`, etc …

![Étape 10](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-10.png#thumbnail)

#### Étape 11

Toujours sur l’onglet `Options`, double-cliquez sur `Paramètres SMBIOS (type 1)`. Une boite de dialogue va s’ouvrir. Renseignez ici les informations avec celles recueillies précédemment dans le fichier `dmidecode.txt` puis cliquez sur `OK`.

![Étape 11](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-11.png#thumbnail)

#### Étape 12

Nous en avons quasiment terminé avec la préparation de la machine virtuelle. Il ne nous reste plus qu’à déclarer la table `ACPI` `MSDM`. Pour ce faire, ouvrez la console de votre nœud Proxmox VE, puis ouvrez le fichier de configuration correspondant à votre machine virtuelle présent dans le répertoire `/etc/pve/qemu-server`.

Avec `vi` :

```
user@host:~$ vi /etc/pve/qemu-server/{vmid}.conf
```

ou bien `nano` :

```
user@host:~$ nano /etc/pve/qemu-server/{vmid}.conf
```

![Étape 12](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-12.png#thumbnail)

#### Étape 13

Ajoutez une entrée `args` avec une option `-acpitable` comme décrit ci-dessous. Cette option sera passée à `qemu` lors de l’exécution de la machine virtuelle.

```
args: -acpitable file=/var/lib/vz/acpi/MSDM.bin
```

Selon votre configuration réseau, peut-être avez-vous besoin que la machine virtuelle fonctionne en mode `NAT`. Cette possibilité a été retirée de l’interface utilisateur de Proxmox il y a un moment, mais elle reste toujours possible. Il suffit pour cela de retirer le bridge de l’interface réseau virtuelle. Si vous ne savez pas à quoi ça sert ou ce que cela implique, ne touchez à rien. Vérifiez juste que vous avez bien la mention `link_down=1`.

Une fois tout saisi et vérifié **deux fois**, enregistrez le fichier de configuration puis quittez votre éditeur de texte.

![Étape 13](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-13.png#thumbnail)

#### Étape 14

Votre machine virtuelle est désormais fin prête à l’installation de Windows et accompagnée de votre clé de licence originale.

![Étape 14](https://www.emaxilde.net/posts/2022/01/28/vm-create/vm-create-14.png#thumbnail)

### Installation de la machine virtuelle

Démarrez la machine virtuelle et connectez-vous à sa console.

![Étape 0](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-00.png#medium-thumbnail)

Vous arrivez sur la mire d’installation, cliquez sur `Suivant`.

![Étape 1](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-01.png#medium-thumbnail)

Vous êtes prêt ? Alors cliquez sur `Installer maintenant`.

![Étape 2](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-02.png#medium-thumbnail)

Le programme d’installation démarre.

![Étape 3](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-03.png#medium-thumbnail)

Si vous arrivez directement sur cette page sans que l’installeur ne vous demande de rentre un numéro de licence, alors c’est qu’il a bien détecté notre fichier de licence dans les informations ACPI. Cochez la case `J'accepte les termes du contrat de licence` puis cliquez sur `Suivant`.

![Étape 4](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-04.png#medium-thumbnail)

Selectionnez le type d’installation `Personnalisé : installer uniquement Windows (avancé)`.

![Étape 5](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-05.png#medium-thumbnail)

Vous arrivez sur le choix du disque de destination. Comme nous avons choisi un périphérique `VirtIO` comme disque dur, nous devons charger les pilotes nécessaires pour que l’installeur puisse le détecter. Cliquez alors sur `Charger un pilote`.

![Étape 6](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-06.png#medium-thumbnail)

Une fenêtre popup apparaît, cliquez sur le bouton `Parcourir`.

![Étape 7](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-07.png#medium-thumbnail)

Une boite de dialogue de sélection de fichier s’ouvre, naviguez vers le second lecteur CD/DVD (disque E:), dépliez le répertoire `amd64`, sélectionnez le répertoire `w10`, puis cliquez sur `OK`.

![Étape 8](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-08.png#medium-thumbnail)

L’installeur trouve le pilote `VirtIO SCSI`, cliquez sur `Suivant` et laissez l’installeur opérer le chargement du pilote.

![Étape 9](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-09.png#medium-thumbnail)

Vous voici maintenant devant le choix du disque de destination, votre disque virtuel doit maintenant apparaître. Sélectionnez-le puis cliquez sur `Suivant`.

![Étape 10](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-10.png#medium-thumbnail)

L’installation démarre …

![Étape 11](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-11.png#medium-thumbnail)

![Étape 12](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-12.png#medium-thumbnail)

![Étape 13](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-13.png#medium-thumbnail)

![Étape 14](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-14.png#medium-thumbnail)

Une fois les fichiers copiés, l’installeur va redémarrer une première fois et procéder à l’installation à proprement dite puis redémarrer une seconde fois.

![Étape 15](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-15.png#medium-thumbnail)

![Étape 16](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-16.png#medium-thumbnail)

![Étape 17](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-17.png#medium-thumbnail)

![Étape 18](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-18.png#medium-thumbnail)

![Étape 19](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-19.png#medium-thumbnail)

![Étape 20](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-20.png#medium-thumbnail)

Une fois l’installation effectuée et le redémarrage opéré, vous arrivez sur la finalisation de votre installation.

![Étape 21](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-21.png#medium-thumbnail)

![Étape 22](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-22.png#medium-thumbnail)

![Étape 23](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-23.png#medium-thumbnail)

![Étape 24](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-24.png#medium-thumbnail)

![Étape 25](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-25.png#medium-thumbnail)

![Étape 26](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-26.png#medium-thumbnail)

![Étape 27](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-27.png#medium-thumbnail)

![Étape 28](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-28.png#medium-thumbnail)

![Étape 29](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-29.png#medium-thumbnail)

![Étape 30](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-30.png#medium-thumbnail)

![Étape 31](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-31.png#medium-thumbnail)

![Étape 32](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-32.png#medium-thumbnail)

![Étape 33](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-33.png#medium-thumbnail)

![Étape 34](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-34.png#medium-thumbnail)

![Étape 35](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-35.png#medium-thumbnail)

![Étape 36](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-36.png#medium-thumbnail)

Une fois toutes les questions répondues, nous y sommes presque !

![Étape 37](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-37.png#medium-thumbnail)

![Étape 38](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-38.png#medium-thumbnail)

Et bim ! Voici un Windows tout neuf. Mais il nous reste encore à installer tous les pilotes `VirtIO` et les `outils invité`.

![Étape 39](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-39.png#medium-thumbnail)

Ouvrez le second lecteur CD/DVD virtuel.

![Étape 40](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-40.png#medium-thumbnail)

Lancez l’installeur `virtio-win-gt-64.exe` et suivez les étapes …

![Étape 41](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-41.png#medium-thumbnail)

![Étape 42](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-42.png#medium-thumbnail)

![Étape 43](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-43.png#medium-thumbnail)

![Étape 44](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-44.png#medium-thumbnail)

![Étape 45](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-45.png#medium-thumbnail)

![Étape 46](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-46.png#medium-thumbnail)

![Étape 47](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-47.png#medium-thumbnail)

Lancez ensuite l’installeur `virtio-win-guest-tools.exe` et suivez les étapes …

![Étape 48](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-48.png#medium-thumbnail)

![Étape 49](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-49.png#medium-thumbnail)

![Étape 51](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-51.png#medium-thumbnail)

Une fois que tout est installé, arrêtez votre machine virtuelle via le `menu démarrer` de Windows. Une fois la machine arrêtée, éjectez les images ISO des lecteurs CD/DVD virtuels, reconnectez le réseau de la carte réseau vrituelle dans l’onglet `Matériel` puis redémarrez la machine virtuelle, logguez-vous et rendez-vous dans la section `Activation` de Windows. Ce dernier doit vous notifier que cette instance de Windows est activée à l’aide d’une licence numérique.

![Étape 52](https://www.emaxilde.net/posts/2022/01/28/vm-install/vm-install-52.png#medium-thumbnail)

### Pour finir

Et voilà ! Vous n’avez plus qu’à passer les mises à jour du système d’exploitation et installer les logiciels dont vous avez besoin.

Si jamais quelque-chose échoue au niveau de l’activation, retentez en ajoutant la table `ACPI` `SLIC`. Sinon revérifiez tout, peut-être que le problème vient d’ailleurs ou bien que votre licence a été blacklistée par les serveurs de Microsoft.

### What else ?

Ce petit trick est bien utile pour récupérer une licence **acquise légalement** avec un ordinateur et pour pouvoir la réutiliser. Evidemment, ce que j’ai fait dans cet article sous Proxmox VE est transposable plus ou moins facilement vers un autre système de virtualisation, que ce soit VirtualBox, virt-manager, etc … Vous avez la recette de base.

Encore une fois je ne pousse pas au piratage de licences, chose contre laquelle je suis totalement opposé. Si vous avez besoin de licences logicielles, achetez-les ! Si vous ne souhaitez pas payer, tournez-vous vers les logiciels libres s’il existe des alternatives à vos besoins.