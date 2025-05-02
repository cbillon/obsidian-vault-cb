---
tags:
  - docker
  - volumes
---
# Mounting a Single File in a Volume Using Docker

## 1. Overview[](https://www.baeldung.com/ops/docker-mount-single-file-in-volume#overview)

As we know, we can containerize our applications using Docker, making software delivery easier. Besides embedding only our app, we can extend the target containers using various options, e.g., mounting external files.

In this tutorial, **we’ll focus on mounting a single file using Docker**, showing different ways to do that. Moreover, we’ll discuss common mistakes during the process and how to fix them.

## 2. Persisting Data in Docker[](https://www.baeldung.com/ops/docker-mount-single-file-in-volume#persisting-data-in-docker)

## [](https://www.baeldung.com/ops/docker-mount-single-file-in-volume#persisting-data-in-docker)

Before we move on, let’s make a quick recap of [how Docker manages application data](https://www.baeldung.com/ops/docker-volumes#what-is-a-volume).

We know that **Docker creates an isolated environment for every running container**. By default, files are stored on an internal container’s writable layer. This means that **any changes done inside the container will be lost when it no longer exists**.

To prevent data loss, Docker provides two main mechanisms for persisting our files to host machines: [volumes](https://docs.docker.com/storage/volumes/) and [bind mounts](https://docs.docker.com/storage/bind-mounts/).

Volumes are created and managed by Docker itself, and external processes shouldn’t modify them. On the other hand, bind mounts may be stored and managed by the host system without preventing non-Docker processes from editing the data.

To achieve our goal, we’ll focus on the binding mechanism.

## 3. Binding Files Using Docker CLI[](https://www.baeldung.com/ops/docker-mount-single-file-in-volume#binding-files-using-docker-cli)
The easiest way to interact with Docker is to use a dedicated Docker CLI that executes various commands with configuration flags. As we know, to create a single container, we can use the _docker run_ command with the desired image:

```bash
docker run alpine:latest
```

[According to the official reference](https://docs.docker.com/engine/reference/commandline/run/), the _run_ command supports many additional options that allow preconfiguring the container. Let’s look through the list of available options:

```bash
--mount		        Attach a filesystem mount to the container
--volume , -v		Bind mount a volume
```

**Both [work similarly, allowing us to persist the data locally](https://docs.docker.com/storage/bind-mounts/#choose-the--v-or---mount-flag)**. Now let’s look at each of them.


### 3.1. _–mount_ Option[](https://www.baeldung.com/ops/docker-mount-single-file-in-volume#1---mount-option)

### [](https://www.baeldung.com/ops/docker-mount-single-file-in-volume#1---mount-option)

The syntax of **the _–mount_ option consists of multiple key-value pairs**, using _<key>=<value>_ tuples. **The order of the keys is not significant**, separating multiple pairs by commas.

First, let’s create a dummy file in our working directory:

```bash
$ echo 'Hi Baeldung! >> file.txt
```

To mount a single local file to the container, we can extend the previous _run_ command:

```bash
$ docker run -d -it \
   --mount type=bind,source="$(pwd)"/file.txt,target=/file.txt,readonly \
   alpine:latest
```

We’ve just created and started a new container mounting our local file. Let’s now take a look at the configuration keys.

The _type_ specifies the mounting mechanism with available values: _bind_, _volume,_ or _tmpfs_. In our case, we should always set a value to _bind_.

The _source_ (alternatively – _src_) is the absolute path to the file or directory on the host that should be mounted. We can also use local shell commands to calculate the result.

The _target_ (alternatively – _destination, dst_) takes the absolute path where the file or directory is mounted inside the container.

Finally, there’s also a _readonly_ option which makes the bind mount read-only. This flag is optional.

In the end, let’s verify the mounting result:

```bash
$ docker exec ... cat /file.txt
Hi Baeldung!
```

We can also inspect container details, using the _docker inspect_ command to check all mounts:

```bash
"Mounts": [
    {
        "Type": "bind",
        "Source": ".../file.txt",
        "Destination": "/file.txt",
        "Mode": "",
        "RW": false,
        "Propagation": "rprivate"
    }
],
```

Now, we can look at common mistakes related to paths. **If we provide non-absolute paths, Docker CLI will return an error** that terminates the command execution:

```bash
docker: Error response from daemon: invalid mount config for type "bind": invalid mount path: 'file.txt+' mount path must be absolute.
```
Sometimes we provide an absolute source path to a file missing on the host machine. In this case, **the container will start mounting an empty directory in the target path**. Moreover, if we are working on Windows, we **[should take care of the path conversions](https://docs.docker.com/desktop/windows/troubleshoot/#path-conversion-on-windows)**.

### 3.2. _–volume_ Option

As we mentioned earlier, we can replace _–mount_ and_–volume_ (_–v)_ flags with the same functionality. We must also remember that the syntax is completely different.

**The _–volume_ syntax consists of three fields**, separated by colon characters. Moreover, **the order of the values is significant**.

Let’s transform the previous example by using the _–v_ option:

```bash
$ docker run -d -it
    -v "$(pwd)"/file.txt:/file.txt:ro \
    alpine:latest
```

The result is the same. We’ve just mounted our local file to the container.

As we can see, three values given by the _–v_ option are similar to the keys used with the _–mount_ flag.

The first value, like the _source_ key, specifies the path to a file or directory on the host machine.

The second field provides a path inside the container and the _target_ key.

Finally, we have an optional _ro_ option that specifies the _read-only_ attribute.

After we check the syntax, let’s see some common mistakes. Same as before, **we should remember about Windows path separators**. **Selecting a non-existent file will also result in creating an empty directory**.

But there is a slight difference with non-absolute paths. **If we provide an** **invalid path to the second value, same as previously, Docker CLI will return an error**. However, if we provide such a path as a source value, the Docker will create a named volume which is another mechanism of persisting files.

In summary, the most significant difference between the _–mount_ and _–volume_ flags is their syntax. We can use both of them interchangeably.

## 4. Binding Files Using Docker Compose

After we learned how to bind files using the Docker CLI, let’s now check if we can still get the same result with docker-compose files.

As we know, [docker-compose is a convenient way of creating containers by providing configuration files](https://www.baeldung.com/ops/docker-compose). For each service, we can declare **[a _volumes_ section to configure binding options](https://docs.docker.com/compose/compose-file/#volumes)**. The _volumes_ section can be specified using either the long syntax or the short one, which has lots in common with _–mount_ and _–volumes_ flags, respectively.

### 4.1. Long Syntax[](https://www.baeldung.com/ops/docker-mount-single-file-in-volume#1-long-syntax)

### [](https://www.baeldung.com/ops/docker-mount-single-file-in-volume#1-long-syntax)

**The long syntax allows us to configure each key separately** to specify the volume mount. With the same example, let’s prepare a docker-compose entry:

```yaml
services:
  alpine:
    image: alpine:latest
    tty: true
    volumes:
      - type: bind
        source: ./file.txt
        target: file.txt
        read_only: true
```

Same as with the Docker CLI, our container is now preconfigured to mount a local file in it. **We use _type_, _source_, _target,_ and the optional _read_only_ keys to determine the configuration**, just like we use the _–mount_ flag. Additionally, we can use a relative path as the _source_ value, calculated from the docker-compose file.

### 4.2. Short Syntax[](https://www.baeldung.com/ops/docker-mount-single-file-in-volume#2-short-syntax)

### [](https://www.baeldung.com/ops/docker-mount-single-file-in-volume#2-short-syntax)

**The short syntax uses a single string value** **separated by colons to specify a volume mount**:

```yaml
services:
  alpine:
    image: alpine:latest
    tty: true
    volumes:
      - ./file.txt:/file.txt:ro
```

The string is almost the same as the _–volume_ flag. **The first two values represent the source and target paths, respectively. The last part specifies additional flags**, where we can specify the read-only attribute. As with the long syntax, we can also use a relative source path.


We must remember that the long form allows us to configure additional fields that can’t be expressed in the short syntax. Moreover, we can mix both syntaxes in the single _volumes_ section.

## 5. Conclusion
In this article, we’ve just covered a part of data persistence in Docker. We tried to mount a single local file in a container using both the Docker CLI and docker-compose files.

**The Docker CLI provides the _–mount_ and _–volume_ options with a _run_ command** to bind a single file or directory. Both flags work similarly but have different syntaxes. As a result, we can use them interchangeably.

We can also reach the same result using docker-compose files. **Inside the _volumes_ section for each service, we can configure a volume mount using either a long or short syntax**. Same as before, these syntaxes interchangeably produce the same result.