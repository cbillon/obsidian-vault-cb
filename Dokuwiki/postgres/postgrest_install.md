---
tags:
  - docker
  - postgresql
  - install
---

====== Installation ======
## Installer une image docker de postgres

  # Run the server
  
  docker run --rm --net=host \
  -e PGRST_DB_URI="postgres://app_user:password@localhost/postgres" \
  postgrest/postgrest


Télécharger le fichier

  wget https://github.com/PostgREST/postgrest/releases/download/v11.2.1/postgrest-v11.2.1-linux-static-x64.tar.xz

Prendre la version linux -x64 (l'autre version provoque des erreurs)

La décompresser :

  tar -Jxf postgrest-[version]-[platform].tar.xz

Créer un fichier conf

  nano tutorial.conf                               
  db-uri = "postgres://authenticator:sesame@localhost:5433/postgres"
  db-schemas = "api"
  db-anon-role ="web_anon"

Lancer

  ./postgrest  tutorial.conf

Ouvrir un autre terminal