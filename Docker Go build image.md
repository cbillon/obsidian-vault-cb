---
tags:
  - docker
  - image
  - golang
---
[Source](https://nesbitt.io/2026/02/15/separating-download-from-install-in-docker-builds.html)
[Voir aussi](https://packagemain.tech/p/docker-build-cache) 

Go modules shipped with Go 1.11 in August 2018, and the community figured out the Docker pattern within weeks. It’s now the canonical Go Dockerfile pattern, recommended by Docker’s own documentation:

```
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app .

```


go mod download reads go.mod and go.sum and fetches everything without doing any resolution or building, and the layer caches when those two files haven’t changed.


Before Go 1.11, GOPATH-based dependency management didn’t have a clean two-file manifest that could be separated from source code for layer caching, and the design of go.mod and go.sum as small standalone files made this Docker pattern fall out naturally once modules landed.

go build can still contact the checksum database (sum.golang.org) after go mod download to verify modules not yet in go.sum. Setting GOFLAGS=-mod=readonly after the download step prevents any network access during the build.