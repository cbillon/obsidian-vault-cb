---
tags:
  - docker
  - cgroups
---
> If you haven't read the Part-1 it is highly recommended to read it [here](https://blog.iamvedant.in/containers-are-not-magic-namespaces-from-scratch) so that you can fully understand the context of this blog. It is the continuation of the previous blog where we dive into implementing namespace for our container.

---

In this part we will implement cgroups and chroot to finish what we started. By the end you will have a strong foundation of how containers actually work and why they were never a black box to begin with (like Neural Networks).

---

## Quick Recap

In Part 1 you gave your container its own hostname, its own process tree and its own view of mounted filesystems. It was starting to feel like a real container. But remember I said each namespace is like a hotel room. Same building, same infrastructure, but every guest gets their own private space. Well, we never added any security to that room. The hotel forgot to put a lock on the minibar and gave you access to every other room on the floor. The container was isolated but not limited and not truly private. Today you are going to fix both of those with cgroups and chroot.

---

## Adding the Door Locks and Level Restrictions

Before diving into our implementation, I'll briefly explain the concepts we will be using to add our door locks logic.

### cgroup

Think of cgroups like the hotel management system. When you check into a hotel, the front desk does not just give you a room key. They also set limits on what you can use. Your room plan says you get access to the gym but not the spa, and the minibar has a fixed budget. If you try to exceed that budget, the system cuts you off. cgroups work exactly the same way for your processes. You decide how much RAM, CPU or how many processes a container can use and the kernel enforces it. Try to go over, and the kernel cuts you off just like the hotel would cut your minibar access. Here is a video [resource](https://www.youtube.com/watch?v=z7mgaWqiV90) that I found helpful in understanding cgroups.

You might wonder that VMs also work the same way, so what is the difference? A VM allocates a fixed chunk of memory and CPU upfront. If you tell a VM it needs 4GB of RAM, the system reserves that 4GB immediately even if the VM is sitting idle using only 200MB. That reserved memory is gone for everyone else. cgroups work differently. They set a ceiling, not a reservation. Your container can use anywhere from 0 to 50MB, and the host only pays for what is actually used. No waste, no hogging, no unnecessary .

### chroot

Think of chroot like a film set. The audience watching the movie genuinely believes the actor is in a mediaeval castle or a space station. They have no idea there are cables, crew members and a parking lot right behind those walls. chroot does the same thing for your container. It builds a set, and the container is the audience. It genuinely believes that the root directory you told it is the root of the entire world. Everything outside does not exist as far as it is concerned. The kernel here only performs different functions, like camera, lighting, etc., that the audience never sees.

> We are using `chroot` in the blog for educational purposes only. `chroot` is more of an isolation feature rather than a security feature. A privileged process having `CAP_SYS_CHROOT` capability can escape the container by using `chdir("..")` directly. To avoid this docker uses `pivot_root` but for educational purposes I am limiting to implementing `chroot`. This is why production ready containers need more hardening.

### Let's start building it

First of all, let's do the honours: Here is the file to the whole code:

How-containers-work/container/container.go at main · Vedant-Gandhi/How-containers-work

This repository implements the internals of container in an emulation way to help understand how docker container actually works. - Vedant-Gandhi/How-...

![github.com](https://github.com/Vedant-Gandhi/How-containers-work/blob/main/container/container.go)

I have put this in a separate package. If you are wondering why it is not in the same file as Part 1, that is a story for another day. Just know it works and follow along.

You can clone this repo and run the program with following commands:

```shell
go build -o mycontainer .
sudo ./mycontainer --mode=container --name=<container_name(optional)>
```

I made a few changes to the container, such as adding named flags and adding your custom container name to add parity with how Docker handles the same. Here are details:

- `mode`: If it has a value of container, it runs the whole container implementation with namespace, cgroup and chroot, and if it is not mentioned, then it runs the namespace code only mentioned in the previous part.
    
- `name`: An optional name for the container that works only with the container mode. It is similar to `docker run --name`.
    

Now the functions `StartParentNameSpace` and `StartNameSpaceChild` are the same as in the previous blog but are copied here again for isolation with only 2 main changes:

- `StartParentNameSpace`: Instead of running the child directly using `cmd.Run` we return the command object because we need the child's PID to add it to the cgroup before it starts running.
    
- `StartNameSpaceChild`: Before mounting the proc, I have added the chroot mount to emulate the file system.
    

Just keep track of above changes. I will explain it in a neat way such that it does not cause confusion.

Our entry point in this code is the Run function that accepts the container name passed as flag. You can ignore that flag if it feels too much overload.

```go
func Run(name string) {
	args := os.Args

	if strings.EqualFold("", name) {
		name = strconv.Itoa(rand.Int())
	}

	// Confirm if the current process is for child if there is a child arg.
	for _, arg := range args {
		if strings.EqualFold(arg, "child") {
			StartNameSpaceChild(name)
			return
		}
	}

	cmd := StartParentNameSpace(name)
	err := cmd.Start()
	if err != nil {
		fmt.Printf("Failed to start the container: %v\n", err)
		os.Exit(1)
	}

	AddCgroup(cmd.Process.Pid, name)
	defer CleanupCgroup(name)

	err = cmd.Wait()

	if err != nil {
		fmt.Printf("Failed to wait for the container: %v\n", err)
		os.Exit(1)
	}

}
```

This function checks if the current invocation is by the child or the parent by looking for child in the arguments, just like we did in Part 1. If it finds child it hands off to StartNameSpaceChild and returns immediately.

If it is the parent, we call `StartParentNameSpace` which sets up the namespace configuration and returns the command object instead of running it directly. We then call `cmd.Start()` which starts the child process but does not block the current thread. This is important because we need the child's PID before it starts doing anything. `cmd.Start()` gives us that window.

Once we have the PID we call `AddCgroup` which creates a cgroup for our container and applies the resource limits. Then we call `cmd.Wait()` which now blocks until the container exits. The defer CleanupCgroup at the end ensures the cgroup directory is always cleaned up when the container exits, no matter what happens.

> **Fun Fact:** In C you would use `fork()` which gives you the child PID immediately since the child is a direct copy of the parent. You set up the cgroup right there and move on. In Go we cannot do that because of the multithreaded runtime as we discussed in Part 1. So `cmd.Start()` and `cmd.Wait()` is our workaround. Same result, different path to get there.
> 
> Multithreaded languages has their own headaches and workarounds.

> Another fun fact: There is a small window between `cmd.Start()` and `AddCgroup()` where our container runs without limits. `runc` solves this by embedding a small C program inside the Go binary specifically for this purpose, using `fork()` to set up cgroups before the container process runs its first instruction. Sometimes even Go needs to call in C for backup.

Now let's move on to look at the cgroup implementation.

```go
func AddCgroup(pid int, name string) {

	// Add permission to allow read and write.
	err := os.Mkdir(fmt.Sprintf("/sys/fs/cgroup/%s", name), 0755)
	if err != nil && !os.IsExist(err) {
		fmt.Printf("Failed to setup permissions for the container: %v\n", err)
		os.Exit(1)
	}

	// We set limit for now to 50MB for now.
	err = os.WriteFile(fmt.Sprintf("/sys/fs/cgroup/%s/memory.max", name), []byte(strconv.Itoa(50*1024*1024)), 0755)
	if err != nil {
		fmt.Printf("Failed to setup memory limit for the container: %v\n", err)
		os.Exit(1)
	}

	// Write the process id to our cgroup.
	err = os.WriteFile(fmt.Sprintf("/sys/fs/cgroup/%s/cgroup.procs", name), []byte(strconv.Itoa(pid)), 0755)
	if err != nil {
		fmt.Printf("Failed to setup the cgroup for container: %v\n", err)
		os.Exit(1)
	}

	// We set the swap to 10MB for now to ensure limts are working.
	err = os.WriteFile(fmt.Sprintf("/sys/fs/cgroup/%s/memory.swap.max", name), []byte(strconv.Itoa(10*1024*1024)), 0755)
	if err != nil {
		fmt.Printf("Failed to set the swap for container: %v\n", err)
		os.Exit(1)
	}

	// We set the max allowed processes that can be created by child to 1000.
	err = os.WriteFile(fmt.Sprintf("/sys/fs/cgroup/%s/pids.max", name), []byte("1000"), 0755)
	if err != nil {
		fmt.Printf("Failed to set th max process allowed for the container : %v\n", err)
		os.Exit(1)
	}

}
```

> **Note:** We are using cgroup v2. If you do not know what that means, ignore it and move on. If you do know, yes it is v2, you are welcome.

This is the hero function that sets our door locks so that the guests stay within their limits.

The first thing it does is create a directory under `/sys/fs/cgroup` with the container name. This is a special path the kernel watches. The moment you create a directory here, you are telling the kernel, **"I want a new cgroup with this name."** If you are on a Linux machine and have Docker running, try `ls /sys/fs/cgroup` and look for any `docker-*` directories. Docker creates one for every container you run, exactly like we are doing here. The kernel then automatically populates that directory with all the control files.

Think of it like checking a new guest into the hotel. The moment you create a room entry in the system, the hotel management software automatically sets up all the default rules for that room such as room service access, minibar budget, gym access. You just created the room, the system handled the rest.

Then we write four files to apply our limits:

- `memory.max` : sets the hard memory limit to 50MB. When the container hits this limit it tries to use swap if available. If not it gets killed. **Equivalent Docker flag:** `--memory=50m`
    
- `cgroup.procs` : this is where the magic happens. Writing the child's PID here tells the kernel to put that process under this cgroup. From this point every process the container spawns automatically inherits these limits. This is the golden file that tells the kernel where to enforce everything. Docker does this internally using `runc`.
    
- `memory.swap.max` : sets the swap limit to 10MB. Without this the kernel lets the container use unlimited swap to sneak around the memory limit. We give it a small buffer before the kill. **Equivalent Docker flag:** `--memory-swap=60m` **.**  
    **Note:** The `--memory-swap` in Docker is the total of RAM plus swap combined, not just the swap alone. So `--memory=50m` and `--memory-swap=60m` means 50MB of RAM and only 10MB of swap. That matches exactly what we set in `memory.swap.max`.
    
- `pids.max` : limits the total number of processes the container can create to 1000. This prevents fork bombs where a process keeps spawning children until the system collapses. **Equivalent Docker flag:** `--pids-limit=1000`
    

That is all it takes to limit what your container can consume. A directory and a few file writes. There are many more control files ([refer cgroup v2 docs](https://www.kernel.org/doc/html/v4.18/admin-guide/cgroup-v2.html#core-interface-files)) you can play with, like CPU limits, disk IO limits and more, but we are keeping it focused for now. If you are curious, go explore `/sys/fs/cgroup` on your machine, everything is right there waiting for you. So we have set up the door locks and minibar limits. Any mischievous guest will be caught red-handed.

Now we move to chroot, and I am going to switch the analogy on you because the hotel does not quite capture what chroot does. For this one, think film sets. Now let me show you how we actually implement this in code.

> Before running this, make sure you have run the [`setup.sh`](http://setup.sh/) script from the repository. It downloads and extracts Alpine Linux into the `rootfs` folder next to your binary. If you want to use a different base like Ubuntu or Debian that works too, just extract it into the same `rootfs` folder and you are good to go. The Alpine Linux here is the same that you use in Docker base build in `FROM alpine:latest.` It is exactly the same implementation only in Docker it sets up automatically using Docker hub and uses build layers from image files.

[

How-containers-work/setup.sh at main · Vedant-Gandhi/How-containers-work

This repository implements the internals of container in an emulation way to help understand how docker container actually works. - Vedant-Gandhi/How-...

![github.com](https://github.com/Vedant-Gandhi/How-containers-work/blob/main/setup.sh)

```go
func SetupChRoot() {
	ex, _ := os.Executable()

	// We get the path of executable and assume the rootfs is store there as well according to the script :).
	rootfs := filepath.Join(filepath.Dir(ex), "rootfs")

	err := syscall.Chroot(rootfs)
	if err != nil {
		fmt.Printf("Failed to setup environment for the container: %v\n", err)
		os.Exit(1)
	}

	// We need to reset the root dir to root else it points to the working directory before we set the new root for current process.
	err = os.Chdir("/")
	if err != nil {
		fmt.Printf("Failed to change directory to root: %v\n", err)
		os.Exit(1)
	}
}
```

This function is where the film set gets built. Let me walk you through it.

First we get the path of the currently running binary using `os.Executable()`. We then assume the `rootfs` directory sits right next to it. Think of it like an actor preparing for a role. Before they step on set, the costume, the props, everything that makes them their in-movie character is laid out in the dressing room right next to the studio. The `rootfs` is that dressing room. Everything the container needs to become its character is sitting right there next to the binary.

Then we call `syscall.Chroot(rootfs)` which tells the kernel "_**for this process, this**_ `rootfs` _**directory is now**_ `/`_**. Everything starts here and nothing exists above it.**_" This is the moment the actor steps on set and becomes the in-movie character. The cameras start rolling, and from this point the actor is no longer themselves. The container has no idea there is a host filesystem behind the walls just like the in-movie character has no idea there is a parking lot behind the castle walls because logically they are supposed to be in a castle. Since we are using Alpine as our rootfs, the container gets an Alpine environment. But this could be Ubuntu, Debian or anything you extract into that folder. The kernel does not care what is inside, it just enforces who the character is.

The last part is important and easy to miss. After `chroot` the process is inside the new root but its current working directory still points to the old path from before the chroot. Think of it like the actor who just stepped on set but is still mentally still thinking about the parking lot. They are wearing the costume but their head is still outside. `os.Chdir("/")` is the director shouting "**action**" and it is the moment the actor fully commits to the in-movie character and forgets who they were outside. This is not a Go specific thing by the way. Any language or program that calls `chroot` must follow it with `chdir("/")`. It is just standard practice across the board.

Now we have setup every form of isolation that a process needs to believe that it is in a unique environment and added every handle to ensure it is shown its place if it acts naughty. Let's test if the kernel really keeps its promise after letting us go through all the complexity hell.

We have already tested namespaces in previous blog so I won't be testing them again. Here I will only test if cgroup and chroot works or not.

---

## Validation and Testing

### Memory Limit

We have set the memory limit to 50MB. I am going to run a memory bomb inside the container that keeps allocating memory and we will see if the cgroup police shows up to handle it.

Here is the memory bomb:

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	var data [][]byte
	for i := range 1000 {
		// Allocate a chunk of 2 MB.
		chunk := make([]byte, 2048*1024)
		// actually touch every page because go does lazy memory allocation.
		for j := range chunk {
			chunk[j] = 1
		}
		data = append(data, chunk)
		fmt.Printf("Allocated %dMB\n", (i+1)*2)
		time.Sleep(100 * time.Millisecond)
	}
}
```

Build it using following command:

```shell
go build -o mem_bomb main.go
```

I have copied it inside the bin folder of my rootfs. If you are going to run this test ensure you do this before starting the container.

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/723d4119-fd21-4815-bb0c-244d88795164.png)

As you can see it allocated up to 56MB and then the cgroup police showed up. The kernel killed the process because it crossed the 60MB total limit we set which is 50MB of RAM plus 10MB of swap. No warnings, no second chances. The kernel just pulled the plug.

Notice that only the memory bomb was killed, not the entire container. The shell survived because it is PID 1 inside our container. If the memory bomb was our entrypoint and was PID 1, it would have taken the whole container down with it. That is exactly how Docker handles OOM kills too.

The killed process exits with code 137 which is the standard Linux exit code for a process killed by SIGKILL. When Docker sees exit code 137 it checks the [`memory.events`](http://memory.events/) file from the container's cgroup to confirm it was an OOM kill. Let's check ours from the host:

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/6c67c007-1ac5-4df4-ae31-de2bf5c9f935.png)

As you can see `oom` is 1 and `oom_kill` is 1 which means one process hit the memory limit and was killed. The `max` value of 76 means the process hit the memory limit 76 times. Each time the kernel tried to reclaim memory before giving up. After exhausting all options it finally invoked the OOM killer and sent the kill signal. And yes, if the memory bomb had continued without the cgroup limit it would have consumed almost 2GB before finishing.

The cgroup police did their job. It enforced the memory limits for the container. So we can conclude that the cgroup are working as intended which means our door locks are working and resource utilization is watched by the kernel.

> Here is a short challenge for you. We also set a process limit of 1000 in our cgroup which means the container can only have 1000 processes running at a time. Write a program that creates a process bomb and run it inside the container. See what happens and drop your findings in the discussion forum of the blog. Curious to see how many of you try it.

### FileSystem Isolation

Now we have also implemented chroot which means filesystem isolation or in our analogy: the film set. Let's test that:

Let's see what is the host OS inside the container:

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/7fbea835-648a-47ae-a63c-e22fa2982232.png)

As you can see the container genuinely believes it is running on Alpine Linux even though the host is Ubuntu. The actor(container) has fully committed to the role. It has no idea it is still on the same kernel, the same machine, just looking at a made up set.

Now let's see what does it show if we print the available files in root:

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/4dd1cdad-51e4-4352-86b8-c5b7e1f82510.png)

As you can see the root for the container is Alpine Linux. If you observe there is no `boot` folder. That is because Alpine does not need one since it is not booting anything. It is just a userspace filesystem sitting on top of your host kernel. The actor does not need to know how the camera works, they just perform within the set they have been given.

Let's assume the container has a mischevious process and is trying to escape **The Matrix**. Let's see what happens if it tries that:

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/56f2c7b8-313f-48ef-97de-dc9dce1263ee.png)

As you can see no matter how many times it tries to go up, it always ends up back at Alpine's root. The process is Neo before he took the red pill. It is running, it thinks it is free, but it is completely bound by the walls of `chroot` and can never escape **The Matrix**. Of course it has an exception we discussed previously : With right capabilities a privileged process can escape our matrix but they need that red pill.

---

## You've Built The Matrix

![](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExdWVqYTZ5cjNjaWNwaGozbW5yNzY3Mnh0YTNqaXh3NjVhdW9oNDlsMCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/zXmbOaTpbY6mA/giphy.gif)

Congratulations for making it to the end. Genuinely. In the age of 30 second videos and infinite scroll, reading a deep technical article takes a different kind of patience.

**A recap of what we did in both articles :**

Namespaces control _**what a process can see**_. We gave the container its own hostname so it has its own identity, its own process tree so it cannot see what else is running on the host, and its own mount namespace so our `/proc` mount stayed contained. But namespaces only isolate the kernel resources that live in memory. They say nothing about how much a process can consume or what files it can access on disk.

That is where `cgroups` and `chroot` come in. `cgroups` control _**what a process can use**_. Without them your container could eat all your RAM and the host would feel every bit of it. `cgroups` put a hard ceiling on memory, swap and process count and the kernel enforces it with no mercy.

`chroot` controls what a _**process can see on disk.**_ `Namespaces` gave the container its own view of kernel resources but the filesystem was still the host's filesystem. `chroot` changes the root directory so the container thinks Alpine is the entire world. It cannot see your home directory, your configs or anything outside that rootfs boundary.

You just built your own basic container runtime and that is exactly what `runc` does at the core of Docker. We did intentionally skip a few things to keep it focused such as user namespaces as our container process and child process all run as root, network isolation so the container shares the host network, and IPC isolation for inter-process communication. Docker handles all of these out of the box along with pulling images from registries, layered filesystems and a lot of production hardening. But the heart of it, `namespaces`, `cgroups` and `chroot`, is exactly what you implemented today. Next time someone says containers are magic, you know better.

Oh and before ending here is the equivalent DockerFile of what we did in both articles:

```dockerfile
FROM alpine:3.19

CMD ["/bin/sh"]
```

And to run it:

```shell
docker run -it \
  --name mycontainer \
  --memory=50m \
  --memory-swap=60m \
  --pids-limit=1000 \
  alpine:3.19
```

And we are done. It took us two blogs and a few hundred lines of Go, Docker does in one command. Now you know what is hiding behind that command. You are no longer just a Docker user, you are someone who understands what Docker actually is. The black box of abstraction is now open and it does not look as messy as we thought before opening it.