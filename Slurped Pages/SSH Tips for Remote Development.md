---
link: https://mutagen.io/blog/ssh-tips-for-remote-development/
byline: |-
  Jacob Howard
  in General8 January 2020
excerpt: |-
  SSH is undoubtedly one of the most powerful tools for enabling remote
  development. Over a history spanning decades, SSH has proven itself to be
  reliable, security-focused, and flexible. Since SSH is one of Mutagen’s core
  transports, I felt that it would be useful to share a handful of simple but
  powerful features that I use on a daily basis to drastically improve my SSH
  experience and remote development workflows.
slurped: 2026-02-17T08:48
title: SSH Tips for Remote Development
tags:
  - ssh
  - tips
---

SSH is undoubtedly one of the most powerful tools for enabling remote development. Over a history spanning decades, SSH has proven itself to be reliable, security-focused, and flexible. Since SSH is one of Mutagen’s core transports, I felt that it would be useful to share a handful of simple but powerful features that I use on a daily basis to drastically improve my SSH experience and remote development workflows.

None of these features will be new or surprising for SSH aficionados, but new or infrequent SSH users will hopefully find them valuable. I’ve intentionally kept this list short and restricted to features that are simple and broadly useful. I’ve also restricted it to OpenSSH features since OpenSSH is by far the most common SSH implementation, but most of these features have analogs in other SSH clients. For a full list of SSH’s superpowers, I’d recommend you grab a cup of tea and drop into the [`ssh` man page](https://linux.die.net/man/1/ssh)—it’s well worth the time investment.

## Aliases and configuration

Perhaps the most powerful SSH feature that users tend to avoid is its configuration system. The SSH client configuration file (stored in `~/.ssh/config` or specified via the `-F` flag) allows users to define hosts, aliases for those hosts, and host-specific configuration. This allows for less typing and significantly reduces the risk of command line typos.

A minimal configuration block in `~/.ssh/config` might look like the following:

```
Host example-server
HostName host.example.org
User me
```

This configuration would cause the command `ssh example-server` to behave like `ssh me@host.example.org`. This may not seem like a huge gain, but it already avoids the need to specify the username when invoking SSH (how many times have you had to `Ctrl-C` SSH after seeing `<local-username>@host.example.org`?). It also creates an alias for the SSH target (`example-server`) and adds it to SSH’s autocomplete functionality, meaning that if you type `ssh ex` and press `Tab`, you’ll see `example-server` in SSH’s list of suggestions (assuming you have autocomplete support enabled for your shell and SSH).

A more complex configuration block might look like the following:

```
Host example-server
HostName host.example.org
Port 24
User me
IdentityFile ~/.ssh/id_rsa_example_org
TCPKeepAlive yes
ServerAliveInterval 60
```

In this example, a custom port has been specified, a non-default SSH private key has been provided, and other connection parameters have been set. Having to type each of these configuration parameters as separate flags when invoking SSH would be both tedious and error-prone.

The best part of all of this is that programs that use SSH as a transport (usually) automatically inherit this functionality, meaning that you can insert the host alias into the location where you’d typically insert a more explicit specification, for example:

```
scp <local-file> example-server:~/<remote-file>
```

or

```
mutagen sync create <local-path> example-server:~/<remote-path>
```

These examples only scratch the surface of what is possible. For more information, including information on wildcard host matching, check out the [`ssh_config` man page](https://linux.die.net/man/5/ssh_config).

Factoring out SSH configuration from the command line to a configuration file has a huge impact on most SSH workflows, making it easier to manage large sets of servers and deal with complex server setups. The time invested in setting up this configuration is usually recouped within hours.

## Proxying connections

One of the coolest features of SSH is its `ProxyCommand` option. This option allows SSH to connect to a remote server via execution of an intermediate command, which is particularly useful in cases where an intermediate bastion server is used to access another internal SSH server.

The SSH configuration for a bastion setup might look something like:

```
Host bastion-server
HostName bastion.example.org
User bastionuser
IdentityFile ~/.ssh/id_rsa_bastion

Host internal-server
HostName 10.0.1.25
User internaluser
IdentityFile ~/.ssh/id_rsa_internal
ProxyCommand ssh bastion-server -W %h:%p
```

In this setup, a bastion server (`bastion.example.org`) is being used to access a private internal server (with the IP address `10.0.1.25`). The `ProxyCommand` option is used to specify a command that will open a TCP connection to the target SSH server and connect it to the standard input and output of the proxy command process. SSH will then use this proxied stream to communicate with the target SSH server. In this case, we’re using `ssh` itself as the proxy command, specifying the handy `-W` flag to open a connection to the internal SSH server via the bastion server, but you can also use tools like `nc` as the proxy command.

The `ProxyCommand` option provides support for templated placeholders like `%h` and `%p` that are used to pass information about the target SSH server to the proxy command. In this case, the hostname passed to the proxy command (filling in the `%h` placeholder) will be `10.0.1.25` and the port value (filling in `%p`) will default to 22. Generally speaking, you’ll want the `HostName` value specified for the target SSH server to be relative to the intermediate server, so it’s quite possible that you might have an internal IP or hostname specified for `HostName`, as is the case here.

What do we get for all this trouble? Well, now we can do:

and be directly connected to the internal server. No more intermediate SSH sessions (at least none that are visible to the user). Moreover, because other tools inherit SSH aliases and their behavior, we can do things like:

```
scp <local-file> internal-server:~/<remote-file>
```

or

```
mutagen forward create tcp:localhost:8080 internal-server:tcp:localhost:8080
```

The flexibility of `ProxyCommand` (and its relative `ProxyJump`) cannot be overstated. While it’s typically used for connecting via bastion servers, it can be used for other types of proxies as well (or to chain multiple proxies together). The only requirement is that the proxy command somehow create a TCP connection to the target SSH server and connect it to the process’ standard input/output. In theory you could write a proxy command that sends SSH data via a web socket, connects via a SOCKS proxy, or even bounces data off the moon using lasers (though the latency probably wouldn’t be ideal).

## Loading private keys

When using public key authentication, you ideally want to password-protect your private keys, though many people don’t. Those that do set a password when creating a private key are rewarded by needing to endlessly type in the password for that private key every time they connect to a server where the key is being used for authentication (which is doubly fun when using `ProxyCommand`).

Fortunately there’s a way to have your cake and secure it too. The `ssh-agent` command is a daemon that lives on the local system and stores your unlocked private keys in-memory, allowing SSH to grab them when necessary and only requiring you to type in the password once. You don’t actually start `ssh-agent` directly, but instead use the `ssh-add` command to unlock a private key and automatically start the `ssh-agent` daemon if it’s not already running. This basically looks like:

```
# Unlock a private key and add it to ssh-agent.
ssh-add ~/.ssh/id_rsa
Enter passphrase:
```

All you have to do is specify the private key that you want to unlock. Once added to `ssh-agent`, the private key will be available to SSH (and any other programs using SSH) without you needing to enter the password for the key.

There’s a lot more that one can do with `ssh-agent` than what’s outlined here, and there are also some security implementations worth reading about (especially on multi-user systems), but for single-user systems with secure SSH keys, `ssh-agent` can be a game changer. For more information, check out the [`ssh-agent` man page](https://linux.die.net/man/1/ssh-agent).

## Port and socket forwarding

Besides creating interactive terminal sessions and forwarding input and output to and from remote commands, SSH’s third major utility is forwarding network traffic. This forwarding is controlled by the `ssh` command’s `-L` and `-R` flags and the exact format for these flags is detailed extensively in the [`ssh` man page](https://linux.die.net/man/1/ssh), so I won’t regurgitate that here. Instead, I just want to touch on a few related topics. Of course I’ll also point out that you can perform the exact same forwarding with Mutagen, and it behaves better in some ways (for example, cleaning up sockets), but sometimes SSH is the right tool for the job (especially if it’s the only one available).

The first thing to mention is the `-N` flag, which essentially tells the `ssh` command that it’s only being used for forwarding and that it shouldn’t start an interactive session or execute any command. Using this flag, a typical TCP forwarding setup might look something like this:

```
# Start an SSH session that performs port forwarding and does nothing else.
ssh -N -L 'localhost:8080:localhost:8080' user@example.org
```

Nothing will be printed when you run this command (unless SSH needs a password or some other input), but the TCP forwarding will be established and continue to function until the command is terminated.

Second, while we’re on the topic of TCP forwarding, it’s really important to mention that the default bind if no hostname is specified is _**all interfaces**_, not `localhost`. This means that you don’t want to do something like:

```
# WARNING: This binds to port 5432 on *ALL* local interfaces!
ssh -N -L '5432:localhost:5432' user@sensitive-postgres-server.example.org
```

Doing so while sitting on a public network and without any local firewall would mean that your secure server was now exposed to anyone on that network who might be scanning for open ports. This same caveat also applies to Mutagen’s network forwarding.

Third, just in case it wasn’t clear, you can forward multiple ports with the same `ssh` command invocation (using repeated `-L` and/or `-R` specifications) and you can mix the `-L` and `-R` flags. Combining multiple forwarding specifications can be a simple and secure mechanism for tying together different application components (e.g. a local web application and a remote database).

Fourth, you can mix TCP and Unix domain socket endpoints when forwarding (for example, forwarding connections from a remote Unix domain socket to a local TCP server). This doesn’t have a lot of practical applications, but it is cool.

Fifth, you may want to remove stale Unix domain sockets when starting forwarding. SSH doesn’t clean up the Unix domain sockets that it creates, so starting a new forwarding command can require tedious removal of the previously created socket. The next best thing is using the `StreamLocalBindUnlink` option, which will remove any conflicting Unix domain sockets when starting forwarding, for example:

```
# Tell SSH to remove any conflicting Unix domain sockets when forwarding.
ssh -N -o 'StreamLocalBindUnlink=yes' -L '/home/me/local.sock:/var/remote.sock' example.org
```

Note that `StreamLocalBindUnlink` doesn’t determine whether or not the conflicting socket is in use—it simply removes any conflicting socket.

Sixth, you may want to set permissions on the socket. This is particularly important when forwarding sockets to a remote daemon that needs to remain secure. This can be done using the `StreamLocalBindMask` option, which overrides the default [`umask`](https://en.wikipedia.org/wiki/Umask) when creating sockets. The default mask is `0177`, which results in sockets which are only accessible by the current user (which is the safest default). Here’s an example of what explicitly specifying such a mask would look like:

```
# Tell SSH to use a umask of 0177 for setting Unix domain socket permissions.
ssh -N -o 'StreamLocalBindMask=0177' -L '/home/me/local.sock:/var/remote.sock' example.org
```

It’s worth noting that not all platforms use socket permissions to restrict access (though most modern systems do), so you should consult the relevant documentation for your platform.

Finally, it’s worth mentioning one important application of this forwarding: accessing a remote Docker® daemon. For example, if you had a Docker daemon installation accessible via SSH, this forwarding might look like:

```
# Forward a local socket to a remote Docker daemon.
ssh -N -o 'StreamLocalBindUnlink=yes' -o 'StreamLocalBindMask=0177' -L '/Users/me/docker.sock:/var/run/docker.sock' dockeruser@dockerhost.example.org
```

You can then tell your Docker client (and any software that runs on top of the client, e.g. Docker Compose or Mutagen) to connect to the remote daemon via this forwarded socket using the `DOCKER_HOST` environment variable, e.g.

```
# Tell the Docker client to use a non-standard socket to connect to the daemon.
export DOCKER_HOST=unix:///Users/me/docker.sock
```

This is a fantastic way to set up and access a Docker daemon without having to virtualize it on your laptop. All you need in this case is the Docker client, which is significantly easier to install. You can also use Mutagen to set up this forwarding, in which case the Mutagen daemon will keep this forwarding alive in the background and allow you to just add the `DOCKER_HOST` setting to your shell initialization file (e.g. `.profile` or `.bashrc`) and access it all the time.

## Multiplexing

All SSH connections are inherently multiplexed, meaning that the single TCP connection made from the `ssh` command to the SSH server handles multiple data streams. For example, one data stream might be used to create an interactive terminal session and another data stream might be used to forward a network connection (depending on what the `ssh` command has been told to do). SSH allows us to exploit this multiplexing even further by using a single multiplexed TCP connection to serve multiple `ssh` command invocations. Instead of opening a new TCP connection every time `ssh` is invoked, SSH can cache and store the TCP connection created for an `ssh` command and re-use it for subsequent `ssh` invocations. Of course, this also applies to commands that use the `ssh` command internally, such as `scp`, `rsync`, or `git`. This can drastically improve the performance of commands that regularly interface with the same SSH host (such as `git push` and `git pull` commands), and makes establishing new interactive SSH sessions lightning fast.

The way this multiplexing is established is via the `ControlMaster` directive. A typical configuration might look like:

```
Host git.example.org
ControlMaster auto
ControlPath ~/.ssh/connections/%r_%h_%p
ControlPersist 1h
```

The way that `ControlMaster` connections work is by creating a Unix domain socket (set via the `ControlPath` parameter) that will be used by `ssh` commands as the default mechanism for connecting to the remote host. By specifying `auto` for `ControlMaster`, we’re telling SSH that it should first try to connect via this Unix domain socket, but fall back to creating a standard TCP connection if the socket doesn’t exist. If an `ssh` command does create a TCP connection, it will then make that connection available via a Unix domain socket at the target path. Thus, you need to specfy a path where those connections should be created and make sure the parent directory (`~/.ssh/connections` in this example) is secure (usually with `0700` permissions). The `ControlPath` option also needs to uniquely correspond to the target connection (so that, for example, `ssh user1@server1.example.org` and `ssh user2@server2.example.org` don’t use the same Unix domain socket). Fortunately, the `ControlPath` option also allows for placeholders to be used in the path, and in the example above we use the `%r`, `%h`, and `%p` placeholders (username, hostname, and port, respectively) to uniquely identify connection parameters.

It sounds complicated, but basically the three lines starting with `Control` in the example above are all you need to add. You can even use them as-is, though you’ll need to create and secure `~/.ssh/connections`.

While `ControlMaster` is a game-changer for many SSH performance woes, it can actually hurt certain applications, because all traffic to the remote SSH server is going over the same TCP connection. This can lead to contention and (in my experience) head-of-line blocking in certain use cases. For things like `git` operations and interactive sessions, it works exceptionally well. For tools like Mutagen and rsync, I’ve found it’s actually best to avoid using `ControlMaster`. Fortunately, SSH’s flexible configuration makes it easy to enable `ControlMaster` only where it’s helpful.

## Conclusion

So that’s my list. There’s still much more that SSH can do, and by combining SSH’s behavioral primitives via other mechanisms, the setups that you can create are effectively limitless. In any case, I hope that these features are helpful in sparking workflow ideas and making your complex remote development experience just a little bit easier.