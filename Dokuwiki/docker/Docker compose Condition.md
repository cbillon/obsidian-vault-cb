---
tags:
  - docker-compose
  - condition
---

====== Depends_on condition alternatives ======

[site](https://github.com/docker/compose/issues/9843)

tester CB

docker-compose.yaml

  services:
    db:
      image: redis
      healthcheck:
        test: "exit 0"
    test:
      image: nginx
      volumes_from:
        - db
      depends_on:
        db:
          condition: service_healthy




  docker compose down --remove-orphans && docker compose up -d && docker compose ps

  