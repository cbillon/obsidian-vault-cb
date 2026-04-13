---
link: https://planetscale.com/blog/processes-and-threads
excerpt: Processes and threads are fundamental abstrations for operating systems. Learn how they work and how they impact database performance in this interactive article.
slurped: 2026-04-12T08:32
title: Processes and Threads — PlanetScale
tags:
  - linux
---

By Ben Dicken | September 24, 2025

What do Slack, Cursor, Ghostty, and Chrome have in common?

No, they aren't all written in TypeScript. These, along with every other piece of software running on your computer, executes within a Process.

The _Process_ is a fundamental abstraction provided by modern _operating systems_. An operating system's job is to provide abstractions on top of the underlying CPU, RAM, and disk that allow software to execute code and share resources. There are many important abstractions such as virtual memory and file systems, but the most fundamental is the _Process_.

This interactive article allows you to build an understanding of what Processes are, how they allow your computer to _multitask_, and how they differ from Threads.

## [CPU and RAM](https://planetscale.com/blog/processes-and-threads#cpu-and-ram)

The two most important components of a computer are the CPU and RAM.

The CPU is considered by many to be the brain of the computer. A CPU chip looks like this:

Beneath the shiny metal enclosure is a piece of silicon with billions of transistors, each built at nanometer-scale. The CPU is where the primary sequence of code is executed on the computer. These instructions do very simple things like adding two numbers, jumping to other instructions, and moving data from one location to another. It doesn't sound like much, but when you can execute billions of these instructions each second you can accomplish a lot.

RAM acts as the short-term memory of a computer. Though they come in different shapes and sizes, a typical DDR RAM chip looks like this:

The big black rectangles are the Dynamic Random Access Memory (DRAM) chips, where the data is physically stored. DRAM chips come in capacities of 8, 16, 32 or more gigabytes per chip, and many computers support multiple chips in combination.

RAM is used to store the numbers, text, and other data that the CPU operates on, all in binary format. The CPU makes requests for data stored in RAM and does computations with this data. The CPU and RAM chips are both connected to a motherboard, which bridges communication between the two.

The CPU isn't much use without RAM, and the RAM is of no use without a CPU. They go hand-in-hand, working together to execute code and manipulate data.

## [Instruction sets](https://planetscale.com/blog/processes-and-threads#instruction-sets)

An **instruction set** is a set of instructions a CPU can execute. These are defined by detailed specifications created by chip manufacturers. The most common ones used in modern CPUs are `x86-64` for ones developed by Intel and `ARM64` for chips based on the ARM architecture.

Both are quite sophisticated, each including hundreds of unique instructions. To give you an idea of how complex these can get, the [x86-64 reference manual](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html) is over 5,000 pages! Though complex, at their core they are still just doing basic math, conditional jumping, and memory management.

In this article we're going to use a simpler (made up!) instruction set. It captures much of the same behavior as "real" ones while being easier to follow in the limited space of a blog. Here is our simple instruction set:

We can implement real programs with this, and will visualize these executing on CPU, RAM, and display visuals interactively. We call a sequence of instructions that accomplish some useful task a Program. Here's a simple program that prints all even numbers between 0 and 10 inclusive. Hit the ▶ (play) button below to play and pause the execution. You can use the other two buttons to change the execution speed and reset the simulation.

Sequences of instructions can execute on the CPU, make modifications to the data on RAM, and print messages to the terminal. (We're fans of Ghostty here at PlanetScale, can you tell?)

## [Running multiple programs](https://planetscale.com/blog/processes-and-threads#running-multiple-programs)

Computers are not limited to running one program at a time. For decades, CPUs have been capable of running many simultaneously. How do they accomplish this?

One thing you might be thinking is: multi-core CPUs! Most CPUs today have multiple execution cores, each of which can run separate programs simultaneously. Nearly all CPUs in modern laptops, desktops, and phones have 2 or more cores. While it is true that this facilitates a computer's ability to multitask (running many programs at once), computers have been able to do this long before multi-core. In fact, we're going to assume you only have one core in your CPU for the rest of this article.

In the early days of computing, computers were fundamentally designed to run only one program at a time from start to finish. A single program was loaded and executed to completion, then the next program could begin.

This limitation was problematic:

1. Sometimes CPUs need to "wait around" for things like reading data from a hard drive. While it waits, the CPU sits idle. Can we use this idle time to run other programs?
2. Computers used to be prohibitively expensive and physically huge, taking up entire rooms. It would be costly to give each employee a dedicated machine, so can we share a CPU?

These and other factors drove an innovation: CPU multitasking via processes.

A process is an _instance_ of a _program_ being executed by a computer. The job of an _operating system_ is to manage many processes at once and allow the CPU to switch between them.

Below is an example of a computer with two processes. One of them is responsible for executing a program that prints all positive even numbers up to 10, the other prints all positive odd numbers up to 10. Hit the ▶ (play) button to begin execution. This visual now has a fourth button, which you can use to swap from the current process to another one.

What's happening here?

1. The first process begins execution to print out even numbers.
2. After a few seconds it pauses, saves the state of the RAM, and changes processes.
3. The odd-number process begins executing. After a few seconds it also pauses, saves the RAM, and switches.

This cycle continues until both programs complete. The CPU divides its time up into short time-slices, giving each process a burst of time to make progress.

In the examples here we give each process a few seconds of time to execute, so that our feeble human brains can follow. On real CPUs, the time slices are measured in milliseconds (one millisecond = 1/1000 of a full second).

Modern CPUs are capable of executing well over 1 billion instructions per second. Even if our time slice is 1 millisecond, this means we can execute 1,000,000,000 ÷ 1,000 = 1 million+ instructions per time slice. By quickly switching between processes, our computers are able to give the illusion of multiple programs running at the same time. CPUs switch so fast humans can't even notice!

## [Context Switching](https://planetscale.com/blog/processes-and-threads#context-switching)

When a computer swaps execution of one process for another, this is called a **context switch**. Each context switch requires quite a bit of "work" including switching to kernel mode, saving register state, and virtual memory management (beyond the scope of here!).

The full time of a context switch takes ~5 microseconds on modern CPUs (1 microsecond = 1 millionth of a second). Though this sounds fast (and it is!) it requires executing tens of thousands of instructions, and this happens hundreds of times per second. A CPU typically executes several billion instructions in a second, but managing and switching between processes can consume tens of millions of these. In other words, the convenience of multi-processing comes with a small performance penalty. We can see this visualized below:

The penalty is almost universally considered "worth it" as it's such a convenient abstraction.

## [Process States](https://planetscale.com/blog/processes-and-threads#process-states)

During the lifetime of a process, it can transition between a number of **process states**. These states are assigned by the Operating System.

While a process is executing on the CPU, it is considered running. Processes can get kicked-off the CPU for one of two reasons: Its time slice is up, or it needs to wait for a disk or network request to continue. In the former case, it moves to the ready state. In the latter case, it moves to the waiting state.

When the process is complete, it moves to the killed state. Hover over or tap on the nodes and arrows in the state diagram below to see how processes flow from one state to the next.

## [Creating new Processes](https://planetscale.com/blog/processes-and-threads#creating-new-processes)

There are many pieces of software that are designed to use multiple processes running together, all coordinating to accomplish a single task.

The Postgres database is a perfect example. Postgres is implemented with a process-per-connection architecture. Each time a client makes a connection, a new Postgres process is created on the server's operating system. There is a single "main" process (PostMaster) that manages Postgres operations, and all new connections create a new Process that coordinates with PostMaster.

Note

PlanetScale Postgres is now generally available and it's the fastest way to run Postgres in the cloud. Check out [our benchmarks](https://planetscale.com/blog/benchmarking-postgres) to see for yourself.

Programs create new processes via two main system calls: `fork()` and `execve()`.

Calling `fork()` makes a process create an exact clone of itself as a new process. The cloned process gets immediately placed into the ready state, while the current one continues to execute (but this varies). We'll introduce a new instruction, `FORK`, to do this for our little CPU simulation:

Let's try this out in our simulator. Hit the play button to begin execution.

We call the process that initiated the `fork` the parent process and the new process the child. When a computer boots up, a single process is initiated and all others are descendants of this one.

However, we don't want all of our processes executing identical code. So, there must be more to this than just `fork`, right?

Right.

A process can call `execve()` to replace its currently executing program with a new one. The programs we may want to execute are stored on our computer's hard drive. `execve()` is given the name of the file to load the program from, and it handles exchanging the instructions and executing the new program. We use the name `EXEC` for this system call on our simplified CPU:

Let's look at this in action. Again, press the play button.

The program begins execution, forks a new process, then both the parent and child load up a new program that prints out even number strings. These two system calls are how we can spawn new processes that do the wide variety of tasks that run on your computer!

Note

Check out `man fork` and `man execve` for more juicy details.

## [Postgres and MySQL](https://planetscale.com/blog/processes-and-threads#postgres-and-mysql)

[Postgres](https://www.postgresql.org/) is one of the most popular pieces of database software in the world, perhaps only second to [MySQL](https://www.mysql.com/). Postgres is designed to handle many thousands of queries per second and tens or even one hundred+ simultaneous connections.

Postgres uses a model of connection-per-process to handle this concurrency. Every time an application server connects to it, a new process is `FORK`ed to handle the queries that it sends. Connections can last for as short as a second to as long as weeks. Either way, it's not uncommon for a single Postgres server to have many simultaneous connections.

Though processes are a convenient way of handling this, Postgres has received some criticism for this architecture. Processes are heavy: there is memory overhead and a time overhead for managing them.

Consider the following program that is running three processes. Each computes a running sum of the values in a sequence in memory - akin to a SQL query doing an aggregation.

How many instructions could have been executed in all the time the CPU spent switching from one process to the next?

## [Threads](https://planetscale.com/blog/processes-and-threads#threads)

MySQL is a great contrast, designed to run as a single process (`mysqld`). However, it is also capable of handling thousands of queries per-second, hundreds of connections, and utilizing multi-core CPUs. It achieves this via threads.

A thread is an additional mechanism for achieving multitasking on a CPU, all within one process. Threads share all the process memory and code (other than their program stacks), but each can be executing at different program locations. They can be switched between, much like process context switches.

Switching between threads is around 5x faster, taking closer to 1 microsecond to complete. If we can architect our applications to handle concurrency with threads, we can achieve better overall performance.

Let's compare the same task being completed by a program that uses multi-processing vs multi-threading. Here we complete the same task as the last example, but with a single process that switches between threads. Notice in the visualization below we never do a full process context switch. Rather, we can switch threads (when the instructions slide to the side) and all of these sequences of execution share the same RAM.

These are rudimentary programs, but these same principles apply to the way that Postgres and MySQL work. Postgres does process-per-connection, MySQL does thread-per-connection. This gives MySQL some advantages in terms of performance in some scenarios.

## [POSIX threads](https://planetscale.com/blog/processes-and-threads#posix-threads)

On modern Unix systems, new threads are typically created via the `pthread_create` POSIX library call. Some lower-level programs call this directly, while others build abstractions atop it.

Both `fork()` and `pthread_create()` are ultimately wrappers around another system call: `clone()`.

There are a number of flags that you can pass to `clone()` it to adjust its behavior for spawning either a process or a thread. For example, the `CLONE_VM` flag causes it to share the virtual memory between the caller and new thread and `CLONE_FILES` causes it to share file descriptors. We wont get into all these details here, but run `man clone` on a linux machine for the details. We add a `PTCREATE` instruction for our CPU to execute:

Multithreaded programs are particularly useful when you have data in RAM that you want to compute multiple things with or subdivide into smaller chunks. Here's an example that calls `PTCREATE` at the beginning with two sequences of execution: One to find the min value and another to find the max value within the first 7 memory slots of the shared RAM.

This would be more efficient to compute in a single loop, but this shows how several threads can work together.

## [Connection Pooling](https://planetscale.com/blog/processes-and-threads#connection-pooling)

In the database world, thread-per-connection is generally preferable to process-per-connection. However, both MySQL and Postgres suffer from performance issues when the connection counts get too high. Even with threads, each connection requires dedicated memory resources to manage connection state.

MySQL, Postgres, and many other databases use a technique known as **connection pooling** to help.

Connection poolers sit between clients and the database. All connections from the client are made to the pooler, which is designed to be able to handle thousands at a time. It maintains its own pool of direct connections to the database, typically between 5 and 50. This is a small enough number that the database server is not negatively impacted by too many connections.

The pooler then intelligently distributes incoming queries/transactions across the fixed set of connections. It acts as a funnel: pushing the queries from thousands of connections into tens of connections.

## [Virtual memory](https://planetscale.com/blog/processes-and-threads#virtual-memory)

In the visuals above, we simplified the process of context switching. This is especially true of the data in RAM. The visuals made it appear that all of the RAM data was copied and restored when context switching. This would be incredibly slow, so what actually happens is that OSs use **virtual memory**.

This is a subject for another day, but in the meantime you can [read more about it online](https://en.wikipedia.org/wiki/Virtual_memory) or purchase [the definitive OS book](https://en.wikipedia.org/wiki/Modern_Operating_Systems).

## [Conclusion](https://planetscale.com/blog/processes-and-threads#conclusion)

Processes and threads are two foundational abstractions. Every program that runs on your computer or phone runs in a Process, and many use multiple Processes, Threads, or a mix of both! Now you know the basics of how these work and what the tradeoffs are when designing software with them.