---
type: tips
link: 
tags:
  - docker
---

``` bash
#!/usr/bin/env bash

set -eu -o pipefail
docker_tags() {
item="$1"
case "$item" in
*/*) :;; # namespace/repository syntax, leave as is
*) item="library/$item";; # bare repository name (docker official image); must convert to namespace/repository syntax

esac
authUrl="https://auth.docker.io/token?service=registry.docker.io&scope=repository:$item:pull"
token="$(curl -fsSL "$authUrl" | jq --raw-output '.token')"
tagsUrl="https://registry-1.docker.io/v2/$item/tags/list"
curl -fsSL -H "Accept: application/json" -H "Authorization: Bearer $token" "$tagsUrl" | jq --raw-output '.tags[]'
}

docker_tags "$@"

```