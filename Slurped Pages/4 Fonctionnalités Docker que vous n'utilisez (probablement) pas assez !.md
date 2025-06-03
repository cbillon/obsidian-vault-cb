---
link: https://blog.ippon.fr/2025/04/14/docker-les-outils-pour-mieux-utiliser-les-conteneurs/
byline: Hugo Joubert
site: Blog Ippon - Ippon Technologies
date: 2025-04-14T10:05
excerpt: |-
  Docker est un outil incontournable pour les DevOps et les Platform Engineers. Cependant, maîtrisez-vous vraiment les subtilités qui apparaissent lors d'une conférence ou d'une recherche dans la documentation Docker ? Ces détails, qui peuvent sembler anodins ou inutiles à première vue, cachent en réalité des fonctionnalités puissantes. Dans cet article nous allons mettre en avant 4 fonctionnalités méconnues de Docker.


  1ere fonctionnalité : SBOM et l’analyse avec Trivy


  Qu'est-ce qu'un SBOM ?
twitter: https://twitter.com/@ippontech
tags:
  - docker
  - tips
slurped: 2025-05-17T09:13
title: 4 Fonctionnalités Docker que vous n'utilisez (probablement) pas assez !
---

Docker est un outil incontournable pour les DevOps et les Platform Engineers. Cependant, maîtrisez-vous vraiment les subtilités qui apparaissent lors d'une conférence ou d'une recherche dans la documentation Docker ? Ces détails, qui peuvent sembler anodins ou inutiles à première vue, cachent en réalité des fonctionnalités puissantes. Dans cet article nous allons mettre en avant 4 fonctionnalités méconnues de Docker.

---

## 1ere fonctionnalité : SBOM et l’analyse avec Trivy

### Qu'est-ce qu'un SBOM ?

Pour une rapide remise en contexte, SBOM est l'acronyme pour **Software Bills Of Materials** qui permet d'avoir un inventaire des composants et des logiciels de l'image du conteneur. On peut voir cela comme la liste des ingrédients trouvable au dos d'un produit. Il faut savoir que plusieurs formats de fichier existent pour les SBOM, les plus importants sont **CycloneDX (format préconisé par OWASP)** et **SPDX (format préconisé par Linux Foundation)**. Pour générer un fichier SBOM avec Docker :

```
docker scout sbom name-of-image
```

Avec cette commande, on va générer un SBOM en format JSON. Dans notre cas, on va vouloir générer en format SPDX, en passant en paramètre --format spdx.

Les SBOMs sont des types de fichiers indigestes pour un humain et on peut donc se demander à quoi peut servir la génération d'un tel fichier. On peut trouver une utilité à la génération de ce type de fichier : pouvoir le coupler à d'autres outils qui se basent sur les versions des logiciels pour faire des analyses. On peut utiliser ce format avec [**Renovate**](https://docs.renovatebot.com/?ref=blog.ippon.fr) ou pour notre exemple, [**Trivy**](https://github.com/aquasecurity/trivy?ref=blog.ippon.fr).

### Qu'est-ce que Trivy ?

**Trivy** est un outil et un projet CNCF pour le scan de sécurité qui peut prendre différentes cibles comme par exemple des éléments d'un cluster Kubernetes, des images de conteneurs, des repositories, des fichiers et dans notre cas, des SBOM. Pour analyser des SBOM avec **Trivy**, on peut se référer à la documentation :

```
trivy sbom file.sbom
```

Ci-dessous, on peut voir le résultat de l'analyse d'un fichier SBOM, tiré de l'image disponible sur **DockerHub** ([rappel de changement des quotas pour le 1er avril](https://www.linkedin.com/posts/yoan-simiand-cossin_cest-demain-le-1-avril-2025-docker-hub-activity-7312368423550787584-CONT?utm_source=share&utm_medium=member_desktop&rcm=ACoAADK-7m0BrmRNaAZIe0bLJskDlMwz1i_fPZA)) :

```
trivy sbom test.sbom

2025-04-07T11:28:30+02:00       INFO    [vulndb] Need to update DB
2025-04-07T11:28:30+02:00       INFO    [vulndb] Downloading vulnerability DB...
2025-04-07T11:28:30+02:00       INFO    [vulndb] Downloading artifact...        ...

Report Summary

┌────────┬──────────┬─────────────────┐
│ Target │   Type   │ Vulnerabilities │
├────────┼──────────┼─────────────────┤
│        │ gobinary │       58        │
└────────┴──────────┴─────────────────┘
Legend:
- '-': Not scanned
- '0': Clean (no security findings detected)


 (gobinary)

Total: 58 (UNKNOWN: 0, LOW: 1, MEDIUM: 23, HIGH: 31, CRITICAL: 3)

┌─────────┬────────────────┬──────────┬────────┬───────────────────┬──────────────────────────────────┬──────────────────────────────────────────────────────────────┐
│ Library │ Vulnerability  │ Severity │ Status │ Installed Version │          Fixed Version           │                            Title                             │
├─────────┼────────────────┼──────────┼────────┼───────────────────┼──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│ stdlib  │ CVE-2023-24538 │ CRITICAL │ fixed  │ 1.18.2            │ 1.19.8, 1.20.3                   │ golang: html/template: backticks not treated as string       │
│         │                │          │        │                   │                                  │ delimiters                                                   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-24538                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2023-24540 │          │        │                   │ 1.19.9, 1.20.4                   │ golang: html/template: improper handling of JavaScript       │
│         │                │          │        │                   │                                  │ whitespace                                                   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2023-24540                   │
│         ├────────────────┤          │        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
│         │ CVE-2024-24790 │          │        │                   │ 1.21.11, 1.22.4                  │ golang: net/netip: Unexpected behavior from Is methods for   │
│         │                │          │        │                   │                                  │ IPv4-mapped IPv6 addresses                                   │
│         │                │          │        │                   │                                  │ https://avd.aquasec.com/nvd/cve-2024-24790                   │
│         ├────────────────┼──────────┤        │                   ├──────────────────────────────────┼──────────────────────────────────────────────────────────────┤
...

```

L'analyse nous fournit plusieurs éléments intéressants, comme le nombre de vulnérabilités présentes dans l'image sur un binaire en particulier. Un tableau avec un aperçu plus détaillé pour le binaire avec la librairie compromise, la CVE associée et son titre, la sévérité, le statut, la version installée et la version qui corrige la CVE.

Avec cet outil, on peut imaginer une intégration dans une CI/CD pour scanner une image ou autre format (Trivy prenant en charge plusieurs types de cibles) avant upload pour traiter rapidement des potentiels problèmes de sécurité.

---

## 2eme fonctionnalité : Faire du multi-architecture avec Docker et buildx

### "Build once, deploy everywhere"

Docker veut mettre en avant la phrase : "**Build once, Deploy everywhere**". C'est vrai, sauf quand l'architecture du processeur entre l'hôte et le conteneur diffère. Le problème de l'architecture intervient lorsque l'on essaie de réduire les coûts et d'améliorer les performances. On aurait besoin de build pour une architecture différente à cause des images Docker qui fonctionnent avec des **manifests**.

Pour récupérer un manifest, on peut utiliser :

```
docker manifest inspect --verbose rustlang/rust:nightly-slim
```

```
{  
  "Ref": "docker.io/amd64/rust:1.42-slim-buster",  
  "Descriptor": {  
    "mediaType": "application/vnd.docker.distribution.manifest.v2+json",  
    "digest": "sha256:1bf29985958d1436197c3b507e697fbf1ae99489ea69e59972a30654cdce70cb",  
    "size": 742,  
    "platform": {  
      "architecture": "amd64",  
      "os": "linux"  
    }  
  },  
  "SchemaV2Manifest": {  
    "schemaVersion": 2,  
    "mediaType": "application/vnd.docker.distribution.manifest.v2+json",  
    "config": {  
      "mediaType": "application/vnd.docker.container.image.v1+json",  
      "size": 4830,  
      "digest": "sha256:dbeae51214f7ff96fb23481776002739cf29b47bce62ca8ebc5191d9ddcd85ae"  
    },  
    "layers": [  
      {  
      "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",  
      "size": 27091862,  
      "digest": "sha256:c499e6d256d6d4a546f1c141e04b5b4951983ba7581e39deaf5cc595289ee70f"  
      },  
      {  
        "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",  
        "size": 175987238,  
        "digest": "sha256:e2f298701fbeb02568c3dcb9822f8488e24ef12f5430bc2e8562016ba8670f0d"  
      }  
    ]  
  }  
}
```

C'est un fichier JSON qui contient les informations de l'image Docker : le hash, les informations des layers, la taille ainsi que la plateforme sur laquelle l'image est censée fonctionner (OS + Architecture).

```
docker manifest inspect ‐‐verbose rust:1.42-slim-buster
```

```
[  
  {  
    "Ref": "docker.io/library/rust:1.42-slim-buster@sha256:1bf29985958d1436197c3b507e697fbf1ae99489ea69e59972a30654cdce70cb",  
    "Descriptor": {  
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",  
      "digest": "sha256:1bf29985958d1436197c3b507e697fbf1ae99489ea69e59972a30654cdce70cb",  
      "size": 742,  
      "platform": {  
        "architecture": "amd64",  
        "os": "linux"  
      }  
    },  
    "SchemaV2Manifest": { ... }  
  },  
  {  
    "Ref": "docker.io/library/rust:1.42-slim-buster@sha256:116d243c6346c44f3d458e650e8cc4e0b66ae0bcd37897e77f06054a5691c570",  
    "Descriptor": {  
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",  
      "digest": "sha256:116d243c6346c44f3d458e650e8cc4e0b66ae0bcd37897e77f06054a5691c570",  
      "size": 742,  
      "platform": {  
        "architecture": "arm",  
        "os": "linux",  
        "variant": "v7"  
      }  
    },  
    "SchemaV2Manifest": { ... }  
...]
```

Dans ce manifest, on a fait du multi-build en mettant une liste de manifests, au lieu d'un seul et unique manifest.

Pour prendre un exemple, nous avons des Runners GitLab qui sont en marche lorsqu'on les appelle avec une CI. Nous voudrions avoir une seule image pour gérer le fait d’avoir des runners qui pourrait tourner sur sur machine ARM comme sur machine AMD en fonction des besoins niveau coût/performance (avec autoscaling). Dans ce cas, nous avons plusieurs façons de procéder. Une façon legacy, longue et fastidieuse avec docker manifest. Une autre façon plus rapide et moderne avec docker buildx qui permet d'effectuer ce travail en une seule commande :

```
docker buildx build \--push \--platform linux/arm/v7,linux/arm64/v8,linux/amd64 \--tag your-username/multiarch-example:buildx-latest .
```

Avec cela, nous avons notre image qui peut être exécutée sur plusieurs plateformes, "**build once, deploy everywhere**" !

---

## 3eme fonctionnalité : Signer ses images Docker

Avec Docker, nous n'avons pas de moyen de signer et vérifier nativement les images. Ce qui peut poser problème si, au sein de votre organisation, vous avez des contraintes de sécurité très importantes au niveau de la provenance des images ou que vous voulez faire attention aux images que vous uploadez sur votre registry. C'est là que **Sigstore** entre en jeu.

### Qu'est-ce que Sigstore ?

**Sigstore** est un projet open source financé par l'Open Source Security Foundation (supportée par la Linux Foundation), qui a pour objectif d'améliorer la sécurité de la chaîne d'approvisionnement de déploiement de logiciels. Il embarque un outil, **Cosign**, qui permet de signer et de vérifier des images de conteneurs, des SBOM, des binaires et plus encore. **Cosign** permet de gérer la signature des images de plusieurs façons différentes : sans clé ou avec clé.

Si on signe avec une clé, ce sera très simple puisqu'il faudra juste fournir la clé pour chiffrer et envoyer l'image sur le registry.

Si on ne fournit pas de clé, on peut quand même signer avec une clé temporaire avec un fournisseur d'identité OIDC. Les IDPs par défaut sont **Google**, **GitHub** et **Microsoft**, on peut en ajouter d'autres. Dans notre cas, nous allons signer avec une clé temporaire :

```
cosign sign registry/name-of-image 
```

Après cette commande, étant donné que l'on n'a pas donné de clé de chiffrement, on doit se connecter sur un des fournisseurs d'identité par défaut cités plus haut. Après cela, l'image est renvoyée au registry en étant signée.

![Capture d'écran du registry Harbor pour montrer que l'image a bien été signée](https://lh7-qw.googleusercontent.com/docsz/AD_4nXftMcKectgvKqefgnN3uITm0kUVESTiSR8nvKFh7Av5SLhP7WWG0sIzMY8QnnKaTTPZjIu_4u0gD9L1CgCgBur6LS07IQr6_HJNbtbPhZpt_mkhjE-3wMt3eA9KE4HUORU-U0Y-?key=Tz34isJ7bXpHeVQZKvJBjT-f)

Nous avons donc signé notre image et nous pouvons retracer la personne qui a fait l'envoi de cette image sur le registry.

**Cosign** s'accompagne d'autres fonctionnalités lors de la signature. On peut signer avec une clé présente sur différents cloud providers comme **Azure** ou **AWS** :

```
cosign sign --key awskms://[ENDPOINT]/[ID/ALIAS/ARN] registry/name-of-image
```

Une autre fonctionnalité qui peut s'avérer utile pour mieux gérer et trier ses images en fonction de la signature est l'annotation. Lors de la signature d'une image, on peut ajouter le flag -a pour ajouter une annotation à l'image qui se retrouvera dans le payload de signature dans la partie Optional.

```
cosign sign --key awskms://[ENDPOINT]/[ID/ALIAS/ARN] registry/name-of-image
```

```
"Optional":{"foo":"bar"}
```

---

## 4eme fonctionnalité : Les OCI Registry ne sert pas qu'à stocker des images (avec ORAS)


Les OCI registry sont des serveurs distants qui permettent de stocker et de récupérer facilement des images de conteneurs ou des artifacts liés aux conteneurs. Docker a déployé un OCI registry pour la première fois et en a payé les pots cassés. En effet, quand Docker a mis en place son registry, les utilisateurs essayaient de mettre tout et n'importe quoi dans les layers des images Docker, notamment des vidéos et des images, ce qui ne correspond pas à l'usage attendu. Le projet **OCI Artifacts** tente de résoudre ce problème en définissant des standards d'artifacts pour vérifier ce qui est poussé sur un registry. En standardisant les artifacts, on peut le combiner avec deux éléments vus en début d'article, **SBOM** et **Trivy**.

Un exemple vaudra mieux que 1000 mots, nous allons présenter comment pousser un fichier texte sur un registry avec [**ORAS**](https://oras.land/docs/?ref=blog.ippon.fr).

**ORAS** est un outil incubé par la CNCF, il permet de travailler avec les artéfacts OCI de la même manière que Docker travaille avec les images (push, pull, login...).

L'outil permet de pousser et récupérer des artifacts OCI depuis des registry OCI. **ORAS** peut s'utiliser comme ceci pour envoyer un fichier .txt en tant que artéfact :

💡

On peut push des artifacts sur AWS ECR

```
oras push $REPOSITORY_URL:text --artifact-type application/vnd.unknown.config.v1+json ./test.txt:application/vnd.unknown.layer.v1+txt
```

Avec cette commande, on a bien poussé l'artifact et on peut le récupérer avec :

```
oras pull $REPOSITORY_URL:text
```

Il est important de noter qu'**ECR** ne permet pas une gestion poussée des artifacts OCI. **Harbor** par exemple permet de récupérer les **SBOM** de l'artifact pour en effectuer des analyses avec **Trivy**.

La question qui se pose après une exposition des fonctionnalités de pull and push sur une registry est : Est-ce vraiment utile ?

### L'utilité des artifacts OCI

Nous pensons qu'il faut répondre à cette question en prévoyant les futurs types d'artifacts qu'il sera possible de pousser sur ces serveurs distants.

Nous allons porter notre dévolu sur les providers et les modules **OpenTofu/Terraform**. En effet, la question d'avoir accès aux modules et aux providers dans un registry privé se pose.

Pour récupérer des modules ou des providers publics nous avons 2 moyens simples, soit depuis le registry [Terraform](https://registry.terraform.io/?ref=blog.ippon.fr) (ou [OpenTofu](https://search.opentofu.org/?ref=blog.ippon.fr)) soit le récupérer depuis un repository de [versioning de code](https://developer.hashicorp.com/terraform/language/modules/sources?ref=blog.ippon.fr#generic-git-repository) (Git, BitBucket...).

Dans certaines organisations, nous avons des politiques de sécurité assez stricts, notamment en ce qui concerne les accès externe vers Internet. A l'heure actuelle, si on veut récupérer un module ou un provider privé, on doit créer un serveur HTTP interne pour exposer l'archive ou bien créer un repository privé sur, par exemple, Github et configurer Git sur la machine local pour avoir les identifiants qui permettront de cloner le projet. Nous admettrons que ces deux façons de faire ne sont pas les plus sécurisés et les plus simples à mettre en place. Alors que des solutions clés en main pour créer des registry privés gérant les artifacts OCI existent avec des solutions comme [Harbor](https://goharbor.io/?ref=blog.ippon.fr) en open source ou AWS ECR sur un cloud public.

Donc les Artifacts OCI dans les registry pourraient répondre à cette problématique de sécurisation d'archives. De plus, si on pousse des providers/modules sur des registry OCI, on peut appliquer toutes les fonctionnalités présentées ci-dessus :

- La signature
- La génération de SBOM
- Le scan de sécurité avec **Trivy** (qui stocke sa base de données de CVE sous forme d'artifacts OCI)

Nous pouvons donc voir que la gestion des artéfacts dans des registry OCI permettrait de gérer beaucoup plus d'archives, de formats de fichiers et également de gérer les artifacts sur des repository privées dans certains cas où l'accès direct à Internet est non-autorisé.

---

## Conclusion

À travers cet article, nous avons exploré plusieurs fonctionnalités avancées de Docker qui sont souvent méconnues mais qui peuvent considérablement améliorer votre workflow DevOps et renforcer la sécurité de vos déploiements.

Ces fonctionnalités représentent l'évolution de l'écosystème Docker vers une plateforme plus complète, sécurisée et flexible, capable de répondre aux besoins croissants des environnements DevOps modernes. En intégrant ces outils et pratiques dans vos workflows, vous pourrez non seulement améliorer la sécurité de vos déploiements mais aussi optimiser vos processus de développement et de déploiement.

---

## Sources

[https://dev.to/aurelievache/understanding-docker-part-39-sbom-3346](https://dev.to/aurelievache/understanding-docker-part-39-sbom-3346?ref=blog.ippon.fr)

[https://trivy.dev/v0.43/docs/target/sbom/](https://trivy.dev/v0.43/docs/target/sbom/?ref=blog.ippon.fr)

[https://medium.com/@krishnaduttpanchagnula/vulnerability-identification-of-images-and-files-using-sbom-with-trivy-23e1a4a5eea4](https://medium.com/@krishnaduttpanchagnula/vulnerability-identification-of-images-and-files-using-sbom-with-trivy-23e1a4a5eea4?ref=blog.ippon.fr)

[https://oras.land/docs/quickstart](https://oras.land/docs/quickstart?ref=blog.ippon.fr)

[https://github.com/opentofu/opentofu/blob/b47237d410bfeaa86aca84a79bde973e3d8a724e/rfc/20241206-oci-registries/4-providers.md](https://github.com/opentofu/opentofu/blob/b47237d410bfeaa86aca84a79bde973e3d8a724e/rfc/20241206-oci-registries/4-providers.md?ref=blog.ippon.fr)

[https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/?ref=blog.ippon.fr)

[https://docs.sigstore.dev/cosign/signing/signing_with_containers/](https://docs.sigstore.dev/cosign/signing/signing_with_containers/?ref=blog.ippon.fr)

