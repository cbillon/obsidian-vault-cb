---
link: https://zonetuto.fr/linux/ubuntu-comment-desinstaller-completement-un-pilote-nvidia/
byline: MrTuto
site: ZoneTuto
date: 2024-03-03T15:16
excerpt: Le tutoriel pour supprimer totalement l'ensemble des drivers Nvidia sur Linux Ubuntu et réinstaller le bon pilote en ligne de commande
slurped: 2025-02-22T09:31
title: "Ubuntu : comment désinstaller complètement les pilotes Nvidia"
tags:
  - nvidia
  - desinstaller
  - pilotes
  - ubuntu
---

Je continue de documenter ici les solutions que je trouve pour résoudre mes problèmes et tenter d’aider le plus grand nombre qui tombe mes articles en cherchant sur un moteur de recherche. J’ai donc eu une nouvelle mésaventure avec mon Linux Ubuntu et c’est la première fois que ça m’arrivait. Voyons ensemble comment désinstaller totalement les drivers Nvidia sur un Linux Ubuntu.

J’ai donc un Ubuntu 22.04.4 LTS et après une mise à jour du kernel vers la version 6.5.0-21, j’ai eu une mauvaise surprise au redémarrage. La photo de la scène de crime :

![](https://zonetuto.fr/wp-content/uploads/2024/03/iwlwifi-wrt-invalid-buffer-destination-1024x319.jpg)

Oups, vraiment pas terrible cette erreur : **iwlwifi 0000:06:00.0: WRT: Invalid buffer destination** … En cherchant sur les forums, j’ai vu que c’était lié au pilote de Nvidia qui est incompatible avec la nouvelle version du kernel. Impossible de faire une [mise à jour du pilote Nvidia sur Linux](https://zonetuto.fr/hardware/nvidia-mettre-a-jour-le-driver-et-la-version-de-cuda-sur-ubuntu/) avec cette technique, aucune commande ne fonctionnait. La solution semble donc assez simple, il faut tout supprimer et les réinstaller proprement.

## Supprimer totalement l’ensemble des drivers Nvidia

Dans mon cas, pour récupérer la main sur un terminal, je suis obligé de faire la combinaison de touches suivante : **Ctrl+Alt+F3**. Dans le terminal, il vaut mieux commencer par regarder ce que vous avez avec la commande suivante :

```
dpkg -l | grep -i nvidia
```

Cette commande devrait normalement vous lister des choses. Voici la réponse que j’obtiens dans mon cas avec mon Linux malade :

![](https://zonetuto.fr/wp-content/uploads/2024/03/grep-nvidia-terminal-commande-1024x454.jpg)

Si vous voulez faire un maximum de ménage et supprimer l’ensemble des pilotes Nvidia, vous pouvez lancer la commande suivante :

```
sudo apt-get remove --purge '^nvidia-.*'
sudo apt autoremove
sudo apt autoclean
```

Si vous voulez faire les choses plus finement et que le remove n’a pas suffit à tout supprimer, vous pouvez cibler directement ce que vous voulez dans la liste au-dessus avec cette commande d’exemple :

```
sudo apt-get remove libnvidia-compute-535:i386
```

Maintenant, que tout semble bien supprimé, après un redémarrage, je n’ai plus cette erreur iwlwifi 0000:06:00.0: WRT: Invalid buffer destination. Par contre j’ai quand même besoin d’avoir des pilotes pour [utiliser ma RTX 4090](https://zonetuto.fr/hardware/test-jai-achete-une-carte-graphique-doccasion-sur-amazon/) et jouer avec des modèles d’intelligence artificielle.

## Réinstaller les pilotes Nvidia sur Linux Ubuntu

Au final, cette partie est assez rapide, car pour installer les drivers de Nvidia cela tient en une simple ligne de commande. Pour installer les pilotes Nvidia tapez la commande suivante sur Ubuntu :

```
sudo apt update
sudo ubuntu-drivers autoinstall
```

Pour vérifier que tout est bon, je relance un dpkg -l | grep -i nvidia pour voir ce qui est installé et cette fois j’ai la liste suivante :

```
ii  libnvidia-cfg1-550:amd64                      550.54.14-0ubuntu1                          amd64        NVIDIA binary OpenGL/GLX configuration library
ii  libnvidia-common-550                          550.54.14-0ubuntu1                          all          Shared files used by the NVIDIA libraries
rc  libnvidia-compute-525:amd64                   525.147.05-0ubuntu0.22.04.1                 amd64        NVIDIA libcompute package
rc  libnvidia-compute-535:amd64                   535.154.05-0ubuntu0.22.04.1                 amd64        NVIDIA libcompute package
ii  libnvidia-compute-550:amd64                   550.54.14-0ubuntu1                          amd64        NVIDIA libcompute package
ii  libnvidia-compute-550:i386                    550.54.14-0ubuntu1                          i386         NVIDIA libcompute package
ii  libnvidia-decode-550:amd64                    550.54.14-0ubuntu1                          amd64        NVIDIA Video Decoding runtime libraries
ii  libnvidia-decode-550:i386                     550.54.14-0ubuntu1                          i386         NVIDIA Video Decoding runtime libraries
ii  libnvidia-encode-550:amd64                    550.54.14-0ubuntu1                          amd64        NVENC Video Encoding runtime library
ii  libnvidia-encode-550:i386                     550.54.14-0ubuntu1                          i386         NVENC Video Encoding runtime library
ii  libnvidia-extra-550:amd64                     550.54.14-0ubuntu1                          amd64        Extra libraries for the NVIDIA driver
ii  libnvidia-fbc1-550:amd64                      550.54.14-0ubuntu1                          amd64        NVIDIA OpenGL-based Framebuffer Capture runtime library
ii  libnvidia-fbc1-550:i386                       550.54.14-0ubuntu1                          i386         NVIDIA OpenGL-based Framebuffer Capture runtime library
ii  libnvidia-gl-550:amd64                        550.54.14-0ubuntu1                          amd64        NVIDIA OpenGL/GLX/EGL/GLES GLVND libraries and Vulkan ICD
ii  libnvidia-gl-550:i386                         550.54.14-0ubuntu1                          i386         NVIDIA OpenGL/GLX/EGL/GLES GLVND libraries and Vulkan ICD
rc  linux-modules-nvidia-525-5.19.0-32-generic    5.19.0-32.33~22.04.1                        amd64        Linux kernel nvidia modules for version 5.19.0-32
rc  linux-modules-nvidia-525-6.2.0-35-generic     6.2.0-35.35~22.04.1                         amd64        Linux kernel nvidia modules for version 6.2.0-35
rc  linux-modules-nvidia-525-6.2.0-37-generic     6.2.0-37.38~22.04.1                         amd64        Linux kernel nvidia modules for version 6.2.0-37
rc  linux-modules-nvidia-525-6.2.0-39-generic     6.2.0-39.40~22.04.1                         amd64        Linux kernel nvidia modules for version 6.2.0-39
rc  linux-objects-nvidia-525-5.19.0-32-generic    5.19.0-32.33~22.04.1                        amd64        Linux kernel nvidia modules for version 5.19.0-32 (objects)
rc  linux-objects-nvidia-525-6.2.0-35-generic     6.2.0-35.35~22.04.1                         amd64        Linux kernel nvidia modules for version 6.2.0-35 (objects)
rc  linux-objects-nvidia-525-6.2.0-37-generic     6.2.0-37.38~22.04.1                         amd64        Linux kernel nvidia modules for version 6.2.0-37 (objects)
rc  linux-objects-nvidia-525-6.2.0-39-generic     6.2.0-39.40~22.04.1+1                       amd64        Linux kernel nvidia modules for version 6.2.0-39 (objects)
ii  nvidia-compute-utils-550                      550.54.14-0ubuntu1                          amd64        NVIDIA compute utilities
ii  nvidia-dkms-550                               550.54.14-0ubuntu1                          amd64        NVIDIA DKMS package
ii  nvidia-driver-550                             550.54.14-0ubuntu1                          amd64        NVIDIA driver metapackage
ii  nvidia-firmware-550-550.54.14                 550.54.14-0ubuntu1                          amd64        Firmware files used by the kernel module
ii  nvidia-kernel-common-550                      550.54.14-0ubuntu1                          amd64        Shared files used with the kernel module
ii  nvidia-kernel-source-550                      550.54.14-0ubuntu1                          amd64        NVIDIA kernel source package
ii  nvidia-prime                                  0.8.17.1                                    all          Tools to enable NVIDIA's Prime
ii  nvidia-settings                               550.54.14-0ubuntu1                          amd64        Tool for configuring the NVIDIA graphics driver
ii  nvidia-utils-550                              550.54.14-0ubuntu1                          amd64        NVIDIA driver support binaries
ii  screen-resolution-extra                       0.18.2                                      all          Extension for the nvidia-settings control panel
ii  xserver-xorg-video-nvidia-550                 550.54.14-0ubuntu1                          amd64        NVIDIA binary Xorg driver
```

J’ai bien les nouveaux pilotes compatibles avec cette version du kernel et tout est rentré dans l’ordre. Vous pouvez maintenant exploiter votre carte graphique au maximum de ses capacités pour jouer sur Linux ou faire du machine learning !