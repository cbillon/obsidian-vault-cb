---
tags:
  - docker
  - cgroups
---
[](https://blog.iamvedant.in/containers-are-not-magic-namespaces-from-scratch)
## Prerequisite (Recommended)

> To understand this article you must know the following:
> 
> 1. Basic familiarity with Docker and you've run a container before.
>     
> 2. Basic Go knowledge (It's okay if you can read Go code even if you don't use it regularly).
>     

---

The prerequisite is there so that you can have a clear gap. I will try to explain and recap about what a container is but if you already know it you can skip to the `Enough Theory, Let's Build` section.

## What is Docker?

So what exactly is Docker?

Think of it like this.

Imagine you built an app on your laptop. It works perfectly. Then you send it to your friend and it breaks immediately because they have a different Node version, different libraries, different everything.

Docker solves this by saying:  
“Don’t just send the code. Send the entire environment.”

It packs your app along with everything it needs into a single unit called a container. Same app, same dependencies, same behavior, no surprises.

That is why people love saying _“it works on my machine”_ Docker basically turns that into _“it works on every machine”_

Docker uses containers to implement this system. A container is basically a running program that already has everything baked in to run your program. You can have multiple container running on a single machine and they will not bother each other because the apps inside that container will never know what else is running. It only knows what you've told it about the outside world. Like a frog in a well that doesn't know the ocean exists. Isolation is the most remarkable feature of containerization as it prevents resource conflicts, makes your application traceable, and keeps failures contained to one place instead of causing a domino effect across your system.

![](https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExejIxNGhoMDNud2hhaTB6N3ZudmZhb2RhYmtiYWx0YXZrZGJzZmZjMiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/zFb4l4CvD4MOQ/giphy.gif)

## Why not use VM directly

Virtual Machines solve the same problem but in a heavy way. A VM emulates an entire computer with its own OS, kernel and virtual hardware. This means booting a full operating system just to run your application. Running five VMs means running five operating systems at the same time. That is a lot of overhead.

Containers are much lighter. Instead of emulating hardware, containers share the host kernel directly. The isolation comes from features already built into the Linux kernel called namespaces and cgroups. This makes containers fast to start and cheap to run. You can run dozens of containers where you could barely fit five VMs.

> The only disadvantage of container is if the host kernel has some security vulnerability then the container also has it whereas in VM it does not because each VM machine is its own OS. So its a security isolation tradeoff for multiple advantages.

## Enough Theory, Let's Build

As the title says, let's start building our own container and see what is actually happening under the hood. By the end of this you will know exactly what a container looks like from the inside.

First let's get you familiar with namespace and cgroup which are the heart and the soul of our precious containers.Trust me, once you understand these two, containers will never feel like a black box again(like Neural Networks).

> Our goal in this tutorial is to run bash in a container. Nothing fancy just our plain simple bash.

### Namespace

A namespace is a Linux kernel feature that wraps a global resource and gives your process its own private view of it. Think of it like your ENV_TYPE variable. You set it to production, preprod or development and based on that your application behaves completely differently. Same codebase, different view of the world. Namespaces work the same way. Same kernel, but every process gets its own private view of resources like hostname, process tree and network. You can also think of each namespace as a hotel room. Same building, same infrastructure, but every guest gets their own private space and whatever they do in their room stays in their room. Read more about it here: [Linux Namespace Manual](https://man7.org/linux/man-pages/man7/namespaces.7.html).

### cgroup

cgroups or control groups is a Linux kernel feature that limits how much of a resource a process can use. Think of it like shared hosting. You and 50 other people are on the same server but your provider makes sure one person cannot eat up all the RAM and starve everyone else. Each account gets a slice and stays within it. cgroups give you that same control but for any process running on your machine. You decide how much CPU or memory a process gets and the kernel enforces it.

### Let's get to work

I will only work on namespaces now and add cgroup in the next one to keep you focued and not get bored.

Here is the link to the whole file:

[

How-containers-work/namespace.go at main · Vedant-Gandhi/How-containers-work

This repository implements the internals of container in an emulation way to help understand how docker container actually works. - Vedant-Gandhi/How-...

github.com





](https://github.com/Vedant-Gandhi/How-containers-work/blob/main/namespace.go)

You can clone this repository and run it with the following commands:

```shell
go build -o mycontainer main.go namespace.go
sudo ./mycontainer # Run as root since namespace creation needs root permission.
```

Right now it might seem too overwhelming but trust me and stay here, I will explain it to you in a very simple and streamlined way.

So our entry point is this function `RunNameSpace` .

```go
func RunNameSpace() {
	args := os.Args

	// The current process is for the child.
	if len(args) > 1 && strings.EqualFold(args[1], "child") {
		StartNameSpaceChild()
		return
	}

	StartParentNameSpace()

}
```

This function checks if the binary has any argument named child to determine if it is a parent process running on the host or a child process running inside the container. If you run the command after cloning the repository it will start the parent flow.

#### The absurd way of creating child process in Linux

In Windows you can specify exactly what process you want to start as a child. Linux does not work like that. In Linux you use `fork()` which copies the entire current process as it is and runs the same code from that point. Both parent and child are running the same code, the only difference is the return value of fork().

Now this works fine for single threaded programs but Go is always multi threaded. When `fork()` is called only the thread that invoked it survives in the child. Every other thread is gone, including ones that were holding locks or managing memory. This leads to deadlocks and crashes.

Go solves this by not using `fork()` at all. Instead of forking, Go starts a completely fresh new process using exec. This new process initializes its own Go runtime from scratch which means no inherited threads, no dangling locks and no risk of corruption. Since the process is spawned using `exec.Command` the parent child relationship is still there at the OS level just like `fork()`. It will get more clear once you get familiar with the `StartParentNameSpace` function.

#### Running the Parent

```go
func StartParentNameSpace() {

	// In linux /proc/self/exe points to current running binary which prevents spoofing via CLI.
	// Here we are mentioning child as arg so as to allow us to recognize child process. It is purely syntactical and you can replace it with anything to detect if process is child.
	cmd := exec.Command("/proc/self/exe", "child", "/bin/sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
		Pdeathsig:  syscall.SIGTERM,
		Setsid:     true,
	}

	// We map the host pipelines to allow us to see output.
	cmd.Stderr = os.Stderr
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout

	err := cmd.Run()

	if err != nil {
		fmt.Printf("Failed to clone the child process: %v\n", err)
		os.Exit(1)
	}
}
```

This is where the actual magic happens. Let me walk you through it line by line.

> When I say parent I just mean the process that starts when you run the program from your terminal. That is it. Nothing fancy.

First we create a command using `/proc/self/exe`. This is a special Linux path that always points to the currently running binary. Think of it like a self referencing letter that says "run me again but this time act differently." We pass `child` as the first argument so our program knows it is now inside the container. This also prevents any spoofing or argument injection where a different binary sneaks in as the child. We also pass `/bin/sh` which is the shell we want running inside our container.

Now this is the interesting part. We attach a `SysProcAttr` to our command before running it. Think of this like a configuration sheet you hand to the kernel before it starts the process. Like when you are setting up a new employee and you hand them an access card that defines exactly what rooms they can enter and what they cannot touch. In that sheet we are specifying three namespace flags:

- `CLONE_NEWUTS` — gives the child its own hostname so its identity is completely isolated from your machine.
    
- `CLONE_NEWPID` — gives it its own process tree so it cannot see anything running on your host.
    
- `CLONE_NEWNS` — gives it its own mount namespace so our /proc mount stays inside the container and never leaks back to your host. See what [/proc](https://www.geeksforgeeks.org/linux-unix/proc-file-system-linux/) is used for
    

We also set `Setsid` to true which gives the child its own terminal session. Without this the child inherits your terminal session and the shell gets confused about who is in control, like two people trying to drive the same car at the same time. `Pdeathsig` tells the kernel to send a `SIGTERM` to the child if the parent dies so we do not leave orphan processes running in the background. Think of it as a safety net that says if the parent is gone, the child should not keep running alone.

The last part is the `stdin stdout stderr` wiring. By default a child process is completely blind and mute. It has no connection to your terminal. It is like putting someone in a room with no windows, no door and no phone. By mapping it to the parent's stdin stdout and stderr you open that connection so you can actually see and interact with the shell running inside your container. Without this you would start the container and see absolutely nothing. This is exactly what the command `docker run` does when you start a container with the `-it` flag.

> **Fun Fact:** Ever wondered how `docker exec -it` works? When you run it, your request first goes to the Docker daemon which then spawns a brand new process and slides it into the same namespaces of the running container using a syscall called `setns()`. Then it sets up a pseudo terminal (PTY) which is basically a fake terminal. The daemon sits in the middle proxying everything between your terminal and the process inside the container. You think you are sitting inside the container but you are actually talking to the daemon which is passing notes back and forth. The container never knew you knocked. Sneaky right?

Finally we call [`cmd.Run`](http://cmd.run/)`()` which actually starts the child process and blocks until it exits. So when you are inside your container shell doing your thing, your parent process is just sitting here waiting patiently like a driver waiting outside while you run your errands. The moment you type exit in your shell, [`cmd.Run`](http://cmd.run/)`()` returns and the parent cleans up and exits too.

> **TLDR:** We create a new process pointing to our own binary, hand the kernel a configuration sheet with three namespace flags, wire up the terminal so you can interact with it and then wait for you to exit.

Now you know how the parent starts the child process in isolation and hands over execution to it. The parent is just setting up the template, defining what the container should look like before it starts. This is exactly what the Docker daemon does when you run `docker run`. It sets up all the namespace configuration and then hands control over to your container. We are doing the same thing, just without the extra layers of modularity, flexibility and complexity.

So now let's see what our child will do. It has to take control now and start doing its own thing i.e set up everything to run our bash in isolation.

```go
func StartNameSpaceChild() {
	args := os.Args

	err := syscall.Sethostname([]byte("custom-host"))
	if err != nil {
		fmt.Println("Failed to change the host name of child namespace")
		os.Exit(1)
	}
	hname, err := os.Hostname()
	if err != nil {
		fmt.Printf("Failed to get the host name of child namespace: %v", err)

	} else {
		fmt.Printf("Hostname changed. New host name is: %s\n", hname)
	}

	// We prevent any event propoagation to the host.
	err = syscall.Mount("", "/", "", syscall.MS_PRIVATE|syscall.MS_REC, "")
	if err != nil {
		fmt.Printf("Failed to make the root mount private : %v", err)
		os.Exit(1)
	}

	// We add hardening just like Docker do.
	err = syscall.Mount("proc", "/proc", "proc", syscall.MS_NOSUID|syscall.MS_NODEV|syscall.MS_NOEXEC, "")
	if err != nil {
		fmt.Printf("Failed to mount the /proc: %v", err)
		os.Exit(1)
	}

	if len(args) > 2 && len(args[2]) > 0 {
		err := syscall.Exec(args[2], args[2:], os.Environ())
		if err != nil {
			fmt.Printf("Failed to run the binary in child namespace: %v\n", err)
			os.Exit(1)
		}
		return
	}

}
```

The first thing our child does is set its own hostname using syscall.Sethostname. Remember the hotel room analogy? This is the moment our room gets its own number. We hardcode it to custom-host here but in a real container runtime like Docker this would be a randomly generated name or whatever you pass with --name. After setting it we read it back with os.Hostname just to confirm it worked. Trust but verify.

Now here is where it gets interesting. Remember we said when you create a new mount namespace it starts as a copy of the host's mounts? Think of it like a hotel room that was set up from a master template. Every room looks the same when you check in. But before you start rearranging the furniture you need to tell the hotel this is your room now and your changes should not affect the master template or any other room. MS_PRIVATE|MS_REC is us detaching our copy so whatever we do inside our container never bleeds back to the host.

Then we mount a fresh /proc filesystem. Remember /proc is not a real filesystem on disk, it is a virtual one the kernel generates in memory to expose process information. Without remounting it our container would still see the host's process tree and ps would show everything running on your machine. Not very isolated is it? We also add three hardening flags:

- `MS_NOSUID` — ignores any setuid bits on executables inside this mount. This prevents privilege escalation attacks where a process tries to run as root on host.
    
- `MS_NODEV` — blocks access to device files inside this mount. No sneaking into `/dev/sda` from inside the container.
    
- `MS_NOEXEC` — prevents executing binaries directly from this mount. An extra layer so nothing suspicious runs straight out of `/proc`.
    

Finally we call `syscall.Exec` which replaces our Go process entirely with /bin/sh. The Go runtime is gone, the shell takes over. This is your container. Everything you do from this point is inside the isolated environment we just built.

> **TLDR:** The child sets its own hostname, detaches from the host's mount template, mounts a fresh `/proc` so it can only see its own processes, and then hands over control to your shell. From this point you are inside the container.

Once you run the program you will see as follows on your terminal:

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/7bf7fc70-36de-4c2e-a4b4-f054c2f08fc5.png)

This means your program ran effectively. You can safely ignore the warning because I couldn't find how to turn it off **😅.**

## Process Isolation Test

If we type `ps` in the child bash you see this:

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/0eb34ac8-6370-4f84-b562-4940727cc936.png)

Notice the PID of sh is 1 which means the container thinks the bash is the first process started on the system. Remember PID 1 is the first process that starts after 0. On your actual machine PID 1 is systemd or init, the process that bootstraps everything. But inside our container your shell has stolen that crown. It has no idea there are thousands of processes running outside.

Now let's see what the actual PID of the container is according to the host system:

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/dcfeddb0-a913-4a5b-8974-6736376a4a4f.png)

Notice the PID according to the host is 124553 but any process running inside our namespace custom-host will see the PID of bash as 1. The host knows the truth, the container lives in its own reality. This means we have successfully isolated the process tree.

## Validation and Testing

Let us put our work to the test now. Here is proof that each isolation is actually working. This is where you can confirm that the isolation at namespace level we have implemented is there and works perfect as intended just like Docker.

## Hostname Isolation Test

If we type `hostname` in the child bash you see this:

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/7edc3c9b-a7f3-4cd1-b104-18197f26a07f.png)

Notice the hostname is custom-host, exactly what we set. The container has its own identity now.

Now let's see what the parent `hostname` is. Run the same command in your host system :

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/de6838cb-72e6-4ef0-9b79-d3f97594c09e.png)

As you can see the parent hostname is different than the child which means we have sucessfully isolated the hostname.

## Mount Isolation Test

`cat /proc/mounts` shows all the filesystems currently mounted on the system. It reads from `/proc` which as we know is a virtual filesystem the kernel generates in memory. Instead of showing the full list which could expose sensitive system details, let's just check the count.

If we type `cat /proc/mounts | wc -l` in the child bash you see this:

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/ddbb9ddf-d930-4b74-9435-59edd4e332d6.png)

Notice the count in child is 42.

Now let's see what the parent filesystem mount count is :

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/26aa1ac0-c1bc-4852-9395-efec336109ed.png)

As you can see the host has one less mount than the container. That extra mount in the child is exactly the `/proc` we mounted ourselves inside the namespace. The host never saw it, never knew about it. That is mount isolation working exactly as intended.

## Filesystem Isolation Test

`ls /proc` shows all the directories currently mounted in the proc.

If we type `ls /proc` in the child bash you see this:

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/55f47f5b-fd52-4ae8-ae16-49ce1907d624.png)

Now let's see what the parent /proc shows us :

![](https://cdn.hashnode.com/uploads/covers/63a479263916c9c9a801da99/7b893b67-efed-466f-af7e-5d19b6e7caea.png)

As you can see both show completely different directories. Each has its own `/proc` virtual filesystem, completely unaware of the other. Filesystem isolation confirmed.

## We are not done yet
Woah!!

You just built your own container using around 100 lines of Go.

No Docker. No magic. Just Linux doing what it has always been capable of.

The same primitives you used here are exactly what Docker uses under the hood.

The only difference is Docker adds polish, networking, volumes, and a lot of convenience on top.

Right now your container still has two problems:

1. It has no resource limits. One process can eat all your RAM
    
2. It can still see your host filesystem
    

We fix both in the next part using:

- cgroups
    
- chroot
    

Think of this part as building the walls.  
Next part, we add the roof and lock the doors.

If you made it this far, you are exactly the kind of person I am writing this for.

See you in Part 2. 🚀