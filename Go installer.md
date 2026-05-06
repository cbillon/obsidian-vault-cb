---
tags:
  - go
  - install
---
[](https://github.com/grahamhelton/go-installer)
# Go Installer 🐹[![](https://camo.githubusercontent.com/456a29e93a0db42aaf852db6a3b4bab0b0d4add658c5b64905b1f562bc7390a2/68747470733a2f2f6b6f756e7465722e6b65726f6c6c6f7a2e6465762f62616467652f6b65726f6c6c6f7a2e676f2d696e7374616c6c65723f7374796c653d666f722d7468652d626164676526636f6c6f723d363964376534266c6162656c3d5669657773266c6162656c436f6c6f723d363964376534)](https://kounter.kerolloz.dev/)

[](https://github.com/grahamhelton/go-installer#go-installer---------)

[!/github.com/kerolloz/go-installer/actions/workflows/test.yml)

## How to use it 🤔

[](https://github.com/grahamhelton/go-installer#how-to-use-it-)

### Installing (or even _updating_) Go ⬇️

[](https://github.com/grahamhelton/go-installer#installing-or-even-updating-go-%EF%B8%8F)

You can _clone_ the repository and then run `bash go.sh`.

Or by simply running whatever suits you from the following commands (`wget`[1](https://github.com/grahamhelton/go-installer#user-content-fn-1-75dc157af1a58c923eba3801324d6034) or `curl`):

```shell
# downloads then runs the script
wget https://git.io/go-installer.sh && bash go-installer.sh
```

```shell
# doesn't download the script ~ runs the script directly 
bash <(curl -sL https://git.io/go-installer)
```

Now, you can go grab a cup of coffee ☕, sit back 😌 and relax while the magic happens! 🔮

> **Note**  
> By default the script will create `.go` and `go` folders on your _HOME_ directory & add the needed variables to your _PATH_ variable.

`$HOME/.go` is where Go will be installed. `$HOME/go` is the default workspace.

In order to install Go to another location or set custom workspace. You can set environment variables GOROOT or GOPATH before installing (or uninstalling) Go.

For example:

```shell
export GOROOT=/opt/go            # where Go is installed
export GOPATH=$HOME/projects/go  # your workspace
```

Read more about [workspaces](https://go.dev/doc/code.html#Workspaces) in Go.

### Specifying a version to install 🧐

[](https://github.com/grahamhelton/go-installer#specifying-a-version-to-install-)

By default, the script installs the latest version available.  
You can choose what version to install by adding the `--version` flag, followed by the version you want to install.

```
bash go.sh --version 1.19.4
```

### Show Help Message 🍁

[](https://github.com/grahamhelton/go-installer#show-help-message-)

To show the following help message use `bash go.sh help`.

[![](https://user-images.githubusercontent.com/36763164/207301551-c686e069-df78-4d28-af78-bedd02b36354.gif)](https://user-images.githubusercontent.com/36763164/207301551-c686e069-df78-4d28-af78-bedd02b36354.gif)

### Uninstalling Go ❌

[](https://github.com/grahamhelton/go-installer#uninstalling-go-)

```shell
bash go.sh remove
```

## How it works ⚙️

[](https://github.com/grahamhelton/go-installer#how-it-works-%EF%B8%8F)

The script does the following steps:

- Checks if Go is already installed.
- Detects the installed operating system (Linux or Mac).
- Detects system architecture (armv6, armv8, amd64, i386).
- Parses the [https://go.dev/dl](https://go.dev/dl) download page to find the latest version of Go that is available for your platform and architecture.
- Exits if you have the latest version of Go already installed.
- Downloads the latest version of Go.
- Creates the needed directories for workspace and Go binaries.
- Extracts the files of the downloaded package.
- Adds the binaries to PATH environment variable.
- Removes the downloaded installation file.

 Demo.mp4