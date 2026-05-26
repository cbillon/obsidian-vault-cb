---
link: https://cylab.be/blog/507/build-a-custom-linux-container
excerpt: Modern container engines like Docker and Podman act as convenient black boxes, obscuring the Linux primitives running behind them. In this guide, we’ll tear down that box by building an entirely rootless, network-isolated, and cgroup-limited Linux container from scratch using only raw Linux commands. By manually orchestrating unshare for namespaces, systemd-run for resource limits, slirp4netns for user-space networking, and pivot_root for filesystem isolation, you’ll gain an understanding of host boundaries and some sysadmin skills required to debug containers directly at the kernel level.
slurped: 2026-05-19T08:13
title: Build a Custom Linux Container
tags:
  - docker
---

Modern container engines like Docker and Podman act as convenient black boxes, obscuring the Linux primitives running behind them. In this guide, we’ll tear down that box by building an entirely rootless, network-isolated, and cgroup-limited Linux container from scratch using only raw Linux commands. By manually orchestrating `unshare` for namespaces, `systemd-run` for resource limits, `slirp4netns` for user-space networking, and `pivot_root` for filesystem isolation, you’ll gain an understanding of host boundaries and some sysadmin skills required to debug containers directly at the kernel level.

You will need three terminal windows.

### Host Preparation (Terminal 1)

First, we install necessary tools and capture current kernel security settings so we can revert them later, adjust them to allow namespaces, and lay down the base operating system.

Open **Terminal 1** (your standard host prompt) and run:

```
# Install necessary tools
sudo apt update
sudo apt install debootstrap slirp4netns systemd -y

# Check and note your original kernel settings (so we can revert them later)
sysctl -n kernel.unprivileged_userns_clone
sysctl -n kernel.apparmor_restrict_unprivileged_userns
sysctl -n net.ipv4.ping_group_range

# Tell the kernel to allow unprivileged user namespaces
sudo sysctl -w kernel.unprivileged_userns_clone=1
sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=0

# Tell the kernel to allow regular users to send ICMP (ping) packets
sudo sysctl -w net.ipv4.ping_group_range="0 2147483647"

# Create the directory and download a minimal Debian system
mkdir -p ~/my-container/rootfs
sudo debootstrap stable ~/my-container/rootfs http://deb.debian.org/debian/
```

**A security note on setting `kernel.apparmor_restrict_unprivileged_userns` to `0`:** disabling AppArmor’s restrictions on unprivileged user namespaces allows the `unshare` program to create a user namespace in which root can be mapped back to the original namespace via `unshare -rU`, increasing the potential attack surface.

### Enter the Namespaces (Terminal 1)

Now we create the container boundary.

Still in **Terminal 1**:

```
systemd-run --user --scope -p MemoryMax=500M -p CPUQuota=50% \
  unshare --user --map-root-user --mount --pid --fork --uts --net \
  /bin/bash
```

Your prompt will change to `root@...`. **Immediately run this to prevent the filesystem changes from crashing the host:**

```
# Stop mounts from leaking back to the host filesystem
mount --make-rprivate /
```

Leave Terminal 1 open and sitting at this root prompt. Do not exit!

### Network Wiring (Terminal 2)

Before we attach the network, we must re-assert total ownership over the filesystem. Background host processes often stealthily reclaim ownership of symlinks (like `/etc/resolv.conf`) right after they are created.

Open a **NEW terminal window (Terminal 2)** on your host, and run:

```
# Take ownership
sudo chown -hR $USER:$USER ~/my-container

# Find the exact Host PID of the unshare command we ran in Terminal 1
CPID=$(pgrep -n unshare)

# Attach the user-space network
slirp4netns --configure --mtu=65520 $CPID tap0
```

Leave Terminal 2 running in the foreground.

### Isolate Filesystems and Pivot (Terminal 1)

Return to your container shell in **Terminal 1**. We will configure the safe `/dev` and execute the `pivot_root` maneuver.

We will stay in `~/my-container` (outside the `rootfs` folder) while executing the pivot. Otherwise, the kernel will refuse to swap the root filesystem if you are currently standing inside the directory you are trying to swap.

```
# Ensure we are standing OUTSIDE the target rootfs
cd ~/my-container
hostname namespaced-debian

# Mount an empty tmpfs over dev to hide host hardware
mount -t tmpfs tmpfs rootfs/dev

# Safely bind-mount only the essential character devices
touch rootfs/dev/null rootfs/dev/zero rootfs/dev/urandom
mount --bind /dev/null rootfs/dev/null
mount --bind /dev/zero rootfs/dev/zero
mount --bind /dev/urandom rootfs/dev/urandom

# Prepare and execute the pivot root
mount --bind rootfs rootfs
mkdir -p rootfs/put_old
pivot_root rootfs rootfs/put_old

# Enter the new root and mount system processes
cd /
mount -t proc proc /proc

# Configure DNS (IP defined in DNS output of slirp4netns in Terminal 2)
echo "nameserver 10.0.2.3" > /etc/resolv.conf

# Tell apt to stay as root to give it necessary rights to run
echo 'APT::Sandbox::User "root";' > /etc/apt/apt.conf.d/99-rootless

# Clean up the old host filesystem traces
umount -l /put_old
rmdir /put_old
```

### Test and Verify

You are inside a fully isolated, cgroup-limited, network-connected rootless container.

Test the following in Terminal 1:

```
ls -l /dev           # You can't see host disks
ping -c 3 google.com # Internet access and ping work
apt update           # The package manager works
ps aux               # Process isolation works
```

## How to visualize the new namespace and cgroups from the host? (Terminal 3)[](https://cylab.be/blog/507/build-a-custom-linux-container#how-to-visualize-the-new-namespace-and-cgroups-from-the-host-terminal-3)

Because you built this using native Linux primitives (`unshare` and `systemd-run`), you don’t need Docker or Podman commands to see what’s going on. You can use standard Linux inspection tools.

Here is how you visualize and inspect your container’s namespaces and cgroups directly from the host. Make sure your container is currently running in Terminal 1 and the networking is managed by `slirp4netns` in Terminal 2 so you have something to look at!

### Find your Container’s Anchor

Everything on the host revolves around the Process ID (PID). Let’s grab the PID of the `unshare` process that is holding your container open.

Open a **NEW terminal window (Terminal 3)** on your host, and run:

```
CPID=$(pgrep -n unshare)
echo "Container Host PID: $CPID"
```

### Visualizing the Namespaces

The Linux kernel provides a built-in tool for listing namespaces: `lsns`.

#### View the isolated boundaries

```
lsns -p $CPID
```

This outputs a table showing every namespace attached to that process. You will see different rows for `mnt` (mount), `uts` (hostname), `ipc` (inter-process communication), `pid` (process tree), `net` (network), and `user` (user mapping). Notice how the `USER` column for that process might say your regular username, but the namespace itself is completely distinct from the host’s root namespace.

#### The Raw Kernel View

If you want to see exactly how the kernel tracks these under the hood, look at the process directory:

```
ls -l /proc/$CPID/ns
```

These are the actual file descriptors for your namespaces. When container runtimes like Docker “attach” a network to a container, they are doing the same, binding for example: `net -> net:[number]`

### Visualizing the Cgroups (Resource Limits)

Because we launched the container using `systemd-run`, systemd is strictly managing the CPU and memory limits we passed (`MemoryMax=500M -p CPUQuota=50%`).

#### Find the systemd Scope

First, we need to see exactly where systemd filed the container. Look at the cgroup your process belongs to:

```
cat /proc/$CPID/cgroup
```

You will see a long path ending in something like `run-r587bed7aab09479...scope`. That `run-...scope` is your container!

#### Inspect the Hard Limits

Want to verify that the 500MB memory limit is actually being enforced? Let’s ask systemd for the exact properties of your container’s scope. Replace the `run-...scope` below with the scope you previously found:

```
systemctl --user show run-YOUR-SCOPE-NAME.scope -p MemoryMax -p CPUQuotaPerSecUSec
```

This queries the DBus and returns the exact bytes and microseconds of CPU time your container is mathematically restricted to.

### Bonus: The “God Mode” Command (`nsenter`)

What if you want to run a command _inside_ the container, but you want to do it from the host, without installing SSH?

You can use `nsenter` (Namespace Enter) to inject a host process directly into the container’s namespaces. If you target the wrong process or forget to explicitly declare your user identity, the kernel will block you with `Permission denied`. Here is the way to inject a command (like `ps aux`) into your rootless container.

#### Target the Child Process

When we ran `unshare --pid`, the `unshare` command created the new PID namespace but physically stayed in the host’s PID namespace, pushing its child (`bash`) inside. We have to target that child.

Run this on your host to find the true container PID:

```
# Get the PID of the bash process that unshare spawned inside the container
BASH_PID=$(pgrep -P $CPID)
```

#### Execute the God Mode Command

Now we inject our host command (`ps aux`) into the container, forcing it to adopt the container’s root identity when it crosses the boundary.

```
sudo nsenter -a -t $BASH_PID -S 0 -G 0 ps aux
```

- **`-a`**: enter all namespaces at once (Mount, PID, Network, UTS, IPC, and User).
- **`-t $BASH_PID`**: target the specific process actually living inside the PID namespace.
- **`-S 0 -G 0`**: explicitly set our UID and GID to `0` (root) upon entering. Without this, `nsenter` keeps your host `sudo` identity, which doesn’t exist in the container’s user map.

This forces the host’s `ps aux` command to execute through the exact lens of the container’s isolation walls. It will return only the processes running inside the container!

## Cleanup[](https://cylab.be/blog/507/build-a-custom-linux-container#cleanup)

#### Kill the container (Terminal 1)

In your container shell, type:

```
exit
```

#### Revert the Host Kernel & Delete Files (Terminal 2)

Go to your normal host prompt in Terminal 2, put an end to the `slirp4netns` command with `CTRL-C` and revert the settings using the values you noted in the beginning (replace the numbers below with your original values):

```
# Revert user namespaces (Usually 0 or 1)
sudo sysctl -w kernel.unprivileged_userns_clone=1

# Revert AppArmor restrictions (Usually 1 or 0)
sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=1

# Revert ping privileges (The default disabled state is "1 0")
sudo sysctl -w net.ipv4.ping_group_range="1 0"

# Delete the container filesystem entirely
sudo rm -rf ~/my-container
```

## Conclusion[](https://cylab.be/blog/507/build-a-custom-linux-container#conclusion)

Congratulations on navigating the intricacies of mount propagation, user namespaces, and rootless networking to build a fully functional container from the ground up! By manually wiring these native Linux primitives together, you’ve proven that containers aren’t magic or virtual machines. They’re simply standard processes wrapped in orchestrated isolation and resource limits.

## References[](https://cylab.be/blog/507/build-a-custom-linux-container#references)

- [https://man7.org/linux/man-pages/man1/unshare.1.html](https://man7.org/linux/man-pages/man1/unshare.1.html)
- [https://man7.org/linux/man-pages/man2/pivot_root.2.html](https://man7.org/linux/man-pages/man2/pivot_root.2.html)
- [https://man7.org/linux/man-pages/man1/nsenter.1.html](https://man7.org/linux/man-pages/man1/nsenter.1.html)
- [https://wiki.debian.org/Debootstrap](https://wiki.debian.org/Debootstrap)
- [https://www.freedesktop.org/software/systemd/man/systemd-run.html](https://www.freedesktop.org/software/systemd/man/systemd-run.html)
- [https://github.com/rootless-containers/slirp4netns](https://github.com/rootless-containers/slirp4netns)
- [https://systemd.io/CGROUP_DELEGATION/](https://systemd.io/CGROUP_DELEGATION/)
- [https://blog.cloudflare.com/live-patch-security-vulnerabilities-with-ebpf-lsm/](https://blog.cloudflare.com/live-patch-security-vulnerabilities-with-ebpf-lsm/)
