---
link: https://www.alexedwards.net/blog/a-time-saving-makefile-for-your-go-projects
byline: Alex Edwards
excerpt: My new book guides you through the start-to-finish build of a real world web application in Go — covering topics like how to structure your code, manage dependencies, create dynamic database-driven pages, and how to authenticate and authorize users securely.
twitter: https://twitter.com/@ajmedwards
slurped: 2026-04-19T15:09
title: A Time-Saving Makefile for Your Go Projects - Alex Edwards
tags:
  - golang
  - makefile
---

### Not sure how to structure your Go web application?

My new book guides you through the start-to-finish build of a real world web application in Go — covering topics like how to structure your code, manage dependencies, create dynamic database-driven pages, and how to authenticate and authorize users securely.



Whenever I start a new Go project, one of the first things I do is create a [Makefile](https://www.gnu.org/software/make/) in the root of my project directory.

This Makefile serves two purposes. The first is to automate common admin tasks (like running tests, checking for vulnerabilities, pushing changes to a remote repository, and deploying to production), and the second is to provide short aliases for Go commands that are long or difficult to remember. I find that it's a simple way to save time and mental overhead, as well as helping me to catch potential problems early and keep my codebases in good shape.

While the exact contents of the Makefile changes from project to project, in this post, I want to share the boilerplate that I'm currently using as a starting point. It's generic enough that you should be able to use it as-is for almost all projects.

---
```
File: Makefile
   # Change these variables as necessary.
    main_package_path = ./cmd/example 
    binary_name = example  
    #==================================================================================== #
# HELPERS 
#==================================================================================== #  
## help: print this help message 
.PHONY: help help:     @echo 'Usage:'     @sed -n 's/^##//p' ${MAKEFILE_LIST} | column -t -s ':' |  sed -e 's/^/ /'  .PHONY: confirm confirm:     @echo -n 'Are you sure? [y/N] ' && read ans && [ $${ans:-N} = y ]  .PHONY: no-dirty no-dirty:     @test -z "$(shell git status --porcelain)"  # ==================================================================================== # # QUALITY CONTROL # ==================================================================================== #  ## audit: run quality control checks .PHONY: audit audit: test     go mod tidy -diff     go mod verify     test -z "$(shell gofmt -l .)"      go vet ./...     go run honnef.co/go/tools/cmd/staticcheck@latest -checks=all,-ST1000,-U1000 ./...     go run golang.org/x/vuln/cmd/govulncheck@latest ./...  ## test: run all tests .PHONY: test test:     go test -v -race -buildvcs ./...  ## test/cover: run all tests and display coverage .PHONY: test/cover test/cover:     go test -v -race -buildvcs -coverprofile=/tmp/coverage.out ./...     go tool cover -html=/tmp/coverage.out  ## upgradeable: list direct dependencies that have upgrades available .PHONY: upgradeable upgradeable:     go run github.com/oligot/go-mod-upgrade@latest  # ==================================================================================== # # DEVELOPMENT # ==================================================================================== #  ## tidy: tidy modfiles and modernize and format .go files .PHONY: tidy tidy:     go mod tidy -v     go fix ./...     go fmt ./...  ## build: build the application .PHONY: build build:     # Include additional build steps, like TypeScript, SCSS or Tailwind compilation here...     go build -o=/tmp/bin/${binary_name} ${main_package_path}  ## run: run the  application .PHONY: run run: build     /tmp/bin/${binary_name}  ## run/live: run the application with reloading on file changes .PHONY: run/live run/live:     go run github.com/cosmtrek/air@v1.43.0 \         --build.cmd "make build" --build.bin "/tmp/bin/${binary_name}" --build.delay "100" \         --build.exclude_dir "" \         --build.include_ext "go, tpl, tmpl, html, css, scss, js, ts, sql, jpeg, jpg, gif, png, bmp, svg, webp, ico" \         --misc.clean_on_exit "true"  # ==================================================================================== # # OPERATIONS # ==================================================================================== #  ## push: push changes to the remote Git repository .PHONY: push push: confirm audit no-dirty     git push  ## production/deploy: deploy the application to production .PHONY: production/deploy production/deploy: confirm audit no-dirty     GOOS=linux GOARCH=amd64 go build -ldflags='-s' -o=/tmp/bin/linux_amd64/${binary_name} ${main_package_path}     upx -5 /tmp/bin/linux_amd64/${binary_name}     # Include additional deployment steps here...   `
```


File: Makefile
'# Change these variables as necessary. main_package_path = ./cmd/example binary_name = example  # ==================================================================================== # # HELPERS # ==================================================================================== #  ## help: print this help message .PHONY: help help:     @echo 'Usage:'     @sed -n 's/^##//p' ${MAKEFILE_LIST} | column -t -s ':' |  sed -e 's/^/ /'  .PHONY: confirm confirm:     @echo -n 'Are you sure? [y/N] ' && read ans && [ $${ans:-N} = y ]  .PHONY: no-dirty no-dirty:     @test -z "$(shell git status --porcelain)"  # ==================================================================================== # # QUALITY CONTROL # ==================================================================================== #  ## audit: run quality control checks .PHONY: audit audit: test     go mod tidy -diff     go mod verify     test -z "$(shell gofmt -l .)"      go vet ./...     go run honnef.co/go/tools/cmd/staticcheck@latest -checks=all,-ST1000,-U1000 ./...     go run golang.org/x/vuln/cmd/govulncheck@latest ./...  ## test: run all tests .PHONY: test test:     go test -v -race -buildvcs ./...  ## test/cover: run all tests and display coverage .PHONY: test/cover test/cover:     go test -v -race -buildvcs -coverprofile=/tmp/coverage.out ./...     go tool cover -html=/tmp/coverage.out  ## upgradeable: list direct dependencies that have upgrades available .PHONY: upgradeable upgradeable:     go run github.com/oligot/go-mod-upgrade@latest  # ==================================================================================== # # DEVELOPMENT # ==================================================================================== #  ## tidy: tidy modfiles and modernize and format .go files .PHONY: tidy tidy:     go mod tidy -v     go fix ./...     go fmt ./...  ## build: build the application .PHONY: build build:     # Include additional build steps, like TypeScript, SCSS or Tailwind compilation here...     go build -o=/tmp/bin/${binary_name} ${main_package_path}  ## run: run the  application .PHONY: run run: build     /tmp/bin/${binary_name}  ## run/live: run the application with reloading on file changes .PHONY: run/live run/live:     go run github.com/cosmtrek/[[email protected]](https://www.alexedwards.net/cdn-cgi/l/email-protection) \         --build.cmd "make build" --build.bin "/tmp/bin/${binary_name}" --build.delay "100" \         --build.exclude_dir "" \         --build.include_ext "go, tpl, tmpl, html, css, scss, js, ts, sql, jpeg, jpg, gif, png, bmp, svg, webp, ico" \         --misc.clean_on_exit "true"  # ==================================================================================== # # OPERATIONS # ==================================================================================== #  ## push: push changes to the remote Git repository .PHONY: push push: confirm audit no-dirty     git push  ## production/deploy: deploy the application to production .PHONY: production/deploy production/deploy: confirm audit no-dirty     GOOS=linux GOARCH=amd64 go build -ldflags='-s' -o=/tmp/bin/linux_amd64/${binary_name} ${main_package_path}     upx -5 /tmp/bin/linux_amd64/${binary_name}     # Include additional deployment steps here...`

The `Makefile` is organized into several sections, each with its own set of _targets_:

### Helpers

- `help`: Prints a help message for the Makefile, including a list of available targets and their descriptions.
- `confirm`: Prompts the user to confirm an action with a "y/N" prompt.
- `no-dirty`: Checks that there there are no untracked files or uncommitted changes to the tracked files in the current git repository.

### Quality control

- `audit`: Runs quality control checks on the codebase, including using `go mod tidy -diff` to check that the `go.mod` and `go.sum` files are up-to-date and correctly formatted, verifying the dependencies with `go mod verify`, running `test -z "$(shell gofmt -l .)"` to check that all `.go` files are correctly formatted, running static analysis with `go vet` and `staticcheck`, checking for vulnerabilities using `govulncheck`, and running all tests. Note that it uses `go run` to execute the latest versions of the remote `staticcheck` and `govulncheck` packages, meaning that you don't need to install these tools first. I've written more about this pattern in [a previous post](https://www.alexedwards.net/blog/using-go-run-to-manage-tool-dependencies).
- `test`: Runs all tests. Note that we enable the race detector and embed [build info](https://pkg.go.dev/runtime/debug#BuildInfo) in the test binary.
- `test/cover`: Runs all tests and outputs a coverage report in HTML format.
- `upgradeable`: List all direct module dependencies that have a newer version available, using the [`oligot/go-mod-upgrade`](https://github.com/oligot/go-mod-upgrade) tool.

### Development

- `tidy`: Updates the dependencies and formats the `go.mod` and `go.sum` using `go mod tidy`, updates your code to use modern Go features with `go fix`, and formats all `.go` files using `go fmt`.
- `build`: Builds the package at `main_package_path` and outputs a binary at `/tmp/bin/{binary_name}`.
- `run`: Calls the `build` target and then runs the binary. Note that my main reason for _not_ using `go run` here is that `go run` doesn't embed build info in the binary.
- `run/live`: Use the `air` tool to run the application with live reloading enabled. When changes are made to any files with the specified extensions, the application is rebuilt and the binary is re-run.
- **Depending on the project** I often add more to this section, such as targets for connecting to a development database instance and managing SQL migrations. Here's [an example](https://gist.github.com/alexedwards/8730fee140cf7c1018f5a6015a517458).

### Operations

- `push`: Push changes to the remote Git repository. This asks for y/N confirmation first, and automatically runs the `audit` and `no-dirty` targets to make sure that all audit checks are passing and there are no uncommitted changes in the repository before the push is executed.
- `production/deploy`: Builds the a binary for linux/amd64 architecture, compress it using `upx`, and then run any deployment steps. Note that this target asks for y/N confirmation before anything is executed, and also runs the `audit` and `no-dirty` checks too.
- **Depending on the project** I often add more to this section too. For example, a `staging/deploy` rule for deploying to a staging server, `production/connect` for SSHing into a production server, `production/log` for viewing production logs, `production/db` for connecting to the production database, and `production/upgrade` for updating and upgrading software on a production server.

## Usage

Each of these targets can be executed by running `make` followed by the target name in your terminal. For example:

`$ make tidy go mod tidy -v go fix ./... go fmt ./...`

If you run `make help` (or the naked `make` command without specifiying a target) then you'll get a description of the available targets.

`$ make help Usage:   help                print this help message   tidy                tidy modfiles and format .go files   audit               run quality control checks   test                run all tests   test/cover          run all tests and display coverage   build               build the application   run                 run the  application   run/live            run the application with reloading on file changes   push                push changes to the remote Git repository   production/deploy   deploy the application to production`

