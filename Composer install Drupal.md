---
tags:
  - composer
  - drupal
---
[Install Drupal with Composer](https://www.drupal.org/node/2718229)

[dockerize](https://git.drupalcode.org/project/dockerize)

Si composer n'est install on peut lancer un container docker
à partir de l'image de composer
lance la commande create-project avec les bons paramètres
- drupal/recommended-project (derbiere versin stable, sinon on indique la version)
- directory my_site_name
```
docker run --rm -i --tty -v $PWD:/app composer create-project drupal/recommended-project my_site_name --ignore-platform-reqs

```

[commande create-projet pour SPIP](https://discuter.spip.net/t/composer-la-commande-create-project/186621)

[The Seed to grow a new Moodle site](https://github.com/moodle/seed)