---
link: https://planetscale.com/blog/io-devices-and-latency
excerpt: Take an interactive journey through the history of IO devices, and
  learn how IO device latency affects performance.
slurped: 2026-04-12T08:42
title: IO devices and latency — PlanetScale
---

By Ben Dicken | March 13, 2025

Non-volatile storage is a cornerstone of modern computer systems. Every modern photo, email, bank balance, medical record, and other critical pieces of data are kept on digital storage devices, often replicated many times over for added durability.

**Non-volatile** storage, or colloquially just "**disk**", can store binary data even when the computer it is attached to is powered off. Computers have other forms of **volatile** storage such as CPU registers, CPU cache, and random-access memory, all of which are faster but require continuous power to function.

Here, we're going to cover the history, functionality, and performance of non-volatile storage devices over the history of computing, all using fun and interactive visual elements. This blog is written in celebration of our latest product release: [PlanetScale Metal](https://planetscale.com/blog/announcing-metal). Metal uses locally attached NVMe drives to run your cloud database, as opposed to the slower and less consistent network-attached storage used by most cloud database providers. This results in a blazing fast queries, low latency, and unlimited IOPS. Check out [the docs](https://planetscale.com/docs/metal) to learn more.

## [Tape Storage](https://planetscale.com/blog/io-devices-and-latency#tape-storage)

As early as the 1950s, computers were using **tape drives** for non-volatile digital storage. Tape storage systems have been produced in many form factors over the years, ranging from ones that take up an entire room to small drives that can fit in your pocket, such as the iconic Sony Walkman. A tape **Reader** is a box containing hardware specifically designed for reading **tape cartridges**. Tape cartridges are inserted and then unwound which causes the tape to move over the **IO Head**, which can read and write data.

Though tape started being used to store digital information over 70 years ago, it is still in use today for certain applications. A standard LTO tape cartridge has several-hundred meters of 0.5 inch wide tape. The tape has several tracks running along its length, each track being further divided up into many small cells. A single tape cartridge contains many trillions of cells.

Each cell can have its magnetic polarization set to up or down, corresponding to a binary 0 or 1. Technically, the magnetic field created by the _transition_ between two cells is what makes the 1 or zero. A long sequence of bits on a tape forms a page of data. In the visualization of the tape reader, we simplify this by showing the tape as a simple sequence of data pages, rather than showing individual bits.

When a tape needs to be read, it is loaded into a reader, sometimes by hand and sometimes by robot. The reader then spins the cartridge with its motor and uses the **reader head** to read off the binary values as the tape passes underneath.

Give this a try with the (greatly slowed down) interactive visualization below. You can control the speed of the tape if you'd like it faster or slower. You can also issue read requests and write requests and then monitor how long these take. You'll also be able to see the queue of pending IO operations pop up in the top-left corner. Try issuing a few requests to get a feel for how tape storage works:

If you spend enough time with this, you will notice that:

1. If you read/write to a cell "near" the read head, it's fast.
2. If you read/write to a cell "far" from the read head, it's slow.

Even with modern tape systems, reading data that is far away on a tape can take 10s of seconds, because it may need to spin the tape by hundreds of meters to reach the desired data. Let's compare two more specific, interactive examples to illustrate this further.

Say we need to read a total of 4 pages and write an additional 4 pages worth of data. In the first scenario, all 4 pages we need to read are in a neat sequence, and the 4 to write to are immediately after the reads. You can see the IO operations queued up in the white container on the top-left. Go ahead and click the Time IO button to see this in action, and observe the time it takes to complete.

As you can see, it takes somewhere around 3-4 seconds. On a real system, with an IO head that can operate much faster and motors that can drive the spools more quickly, it would be much faster.

Now consider another scenario where we need to read and write the same number of pages. However, these reads and writes are spread out throughout the tape. Go ahead and click the Time IO button again.

That took ~7x longer for the same total number of reads and writes! Imagine if this system was being used to load your social media feed or your email inbox. It might take 10s of seconds or even a full minute to display. This would be totally unacceptable.

Though the latency for random reads and writes is poor, tape systems operate quite well when reading or writing data in long sequences. In fact, tape storage still has many such use cases today in the modern tech world. Tape is particularly well-suited for situations where there is a need for _massive_ amounts of storage that does not need to be read frequently, but needs to be safely stored. This is because tape is both cheaper per-gigabyte and has a longer shelf-life than its competition: solid state drives and hard disk drives. For example, CERN has a tape storage data warehouse with [over 400 petabytes of data](https://home.cern/science/computing/data-preservation) under management. AWS also offers [tape archiving](https://aws.amazon.com/storagegateway/vtl/) as a service.

What tape is not well suited for is high-traffic transactional databases. For these and many other high-performance tasks, other storage mediums are needed.

## [Hard Disk Drives](https://planetscale.com/blog/io-devices-and-latency#hard-disk-drives)

The next major breakthrough in storage technology was the **hard disk drive**.

Instead of storing binary data on a tape, we store them on a small circular metal disk known as the **Platter**. This disk is placed inside of an **enclosure** with a special **read/write head**, and spins very fast (7200 RPM is common, for example). Like the tape, this disk is also divided into tracks. However, the tracks are _circular_, and a single disk will often have well over 100,000 tracks. Each track contains hundreds of thousands of pages, and each page containing 4k (or so) of data.

An HDD requires a mechanical spinning motion of both the reader and the platter to bring the data to the correct location for reading. One advantage of HDD over tape is that the entire surface area of the bits is available 100% of the time. It still takes time to move the needle + spin the disk to the correct location for a read or write, but it does not need to be "uncovered" like it needs to be for a tape. This combined with the fact that there are two different things that can spin, means data can be read and written with much lower latency. A typical random read can be performed in 1-3 milliseconds.

Below is an interactive hard drive. You can control the speed of the platter if you'd like it faster or slower. You can request that the hard drive read a page and write to a nearby available page. If you request a read or write before the previous one is complete, a queue will be built up, and the disk will process the requests in the order it receives them. As before, you'll also be able to see the queue of pending IO operations in the white IO queue box.

As with the tape, the speed of the platter spin has been slowed down by orders of magnitude to make it easier to see what's going on. In real disks, there would also be many more tracks and sectors, enough to store multiple terabytes of data in some cases.

Let's again consider a few specific scenarios to see how the order of reads and writes affects latency.

Say we need to write a total of three pages of data and then read 3 pages afterward. The three writes will happen on nearby available pages, and the reads will be from tracks 1, 4, and 3. Go ahead and click the Time IO button. You'll see the requests hit the queue, the reads and writes get fulfilled, and then the total time at the end.

Due to the sequential nature of most of these operations, all the tasks were able to complete quickly.

Now consider the same set of 6 reads and writes, but with them being interleaved in a different order. Go ahead and click the Time IO button again.

If you had the patience to wait until the end, you should notice how the same total number of reads and writes took much longer. A lot of time was spent waiting for the platter to spin into the correct place under the read head.

Magnetic disks have supported command queueing directly on the disks for a long time (80s with SCSI, 2000s with SATA). Because of this, the OS can issue multiple commands that run in parallel and potentially out-of-order, similar to SSDs. Magnetic disks also improve their performance if they can build up a queue of operations that the disk controller can then schedule reads and writes to optimize for the geometry of the disk.

Here's a visualization to help us see the difference between the latency of a random tape read compared to a random disk read. A random tape read will often take multiple seconds (I put 1 second here to be generous) and a disk head seek takes closer to 2 milliseconds (one thousandth of a second)

Even though HDDs are an improvement over tape, they are still "slow" in some scenarios, especially random reads and writes. The next big breakthrough, and currently the most common storage format for transactional databases, are SSDs.

## [Solid State Drives](https://planetscale.com/blog/io-devices-and-latency#solid-state-drives)

Solid State Storage, or "flash" storage, was invented in the 1980s. It was around even while tape and hard disk drives dominated the commercial and consumer storage spaces. It didn't become mainstream for consumer storage until the 2000s due to technological limitations and cost.

The advantage of SSDs over both tape and disk is that they do not rely on any _mechanical_ components to read data. All data is read, written, and erased electronically using a special type of non-volatile _transistor_ known as NAND flash. This means that each 1 or 0 can be read or written without the need to move any physical components, but 100% through electrical signaling.

SSDs are organized into one or more **targets**, each of which contains many **blocks** which each contain some number of **pages**. SSDs read and write data at the page level, meaning they can only read or write full pages at a time. In the SSD below, you can see reads and writes happening via the **lines** between the controller and targets (also called "traces").

The removal of mechanical components reduces the latency between when a request is made and when the drive can fulfill the request. There is no more waiting around for something to spin.

We're showing small examples in the visual to make it easier to follow along, but a single SSD is capable of storing multiple terabytes of data. For example, say each page holds 4096 bits of data (4k). Now, say each block stores 16k pages, each target stores 16k blocks, and our device has 8 targets. This comes out to `4k * 16k * 16k * 8 = 8,796,093,022,208` bits, or 8 terabytes. We could increase the capacity of this drive by adding more targets or packing more pages in per block.

Here's a visualization to help us see the difference between the latency of a random read on an HDD vs SSD. A random read on an SSD varies by model, but can execute as fast as 16μs (μs = microsecond, which is one millionth of a second).

It would be tempting to think that with the removal of mechanical parts, the organization of data on an SSD no longer matters. Since we don't have to wait for things to spin, we can access any data at any location with perfect speed, right?

Not quite.

There are other factors that impact the performance of IO operations on an SSD. We won't cover them all here, but two that we will discuss are **parallelism** and **garbage collection**.

### [SSD Parallelism](https://planetscale.com/blog/io-devices-and-latency#ssd-parallelism)

Typically, each **target** has a dedicated **line** going from the control unit to the target. This line is what processes reads and writes, and only one page can be communicated by each line at a time. Pages can be communicated on these lines _really fast_, but it still does take a small slice of time. The organization of data and sequence of reads and writes has a significant impact on how efficiently these lines can be used.

In the interactive SSD below, we have 4 targets and a set of 8 write operations queued up. You can click the Time IO button to see what happens when we can use the lines in parallel to get these pages written.

In this case, we wrote 8 pages spread across the 4 targets. Because they were spread out, we were able to leverage parallelism to write 4 at a time in two time slices.

Compare that with another sequence where the SSD writes all 8 pages to the same target. The SSD can only utilize a single data line for the writes. Again, hit the Time IO button to see the timing.

Notice how only one line was used and it needed to write sequentially. All the other lines sat dormant.

This demonstrates that the order in which we read and write data matters for performance. Many software engineers don't have to think about this on a day-to-day basis, but those designing software like MySQL need to pay careful attention to what [structures data is being stored in](https://planetscale.com/blog/btrees-and-database-indexes) and how data is laid out on disk.

### [SSD Garbage Collection](https://planetscale.com/blog/io-devices-and-latency#ssd-garbage-collection)

The minimum "chunk" of data that can be read from or written to an SSD is the size of a page. Even if you only need a subset of the data within, that is the unit that requests to the drive must be made in.

Data can be read from a page any number of times. However, writes are a bit different. After a page is written to, it cannot be overwritten with new data until the old data has been explicitly **erased**. The tricky part is, individual pages cannot be erased. When you need to erase data, the entire block must be erased, and afterwards all of the pages within it can be reused.

Each SSD needs to have an internal algorithm for managing which pages are empty, which are in use, and which are **dirty**. A **dirty** page is one that has been written to but the data is no longer needed and ready to be erased. Data also sometimes needs to be re-organized to allow for new write traffic. The algorithm that manages this is called the **garbage collector**.

Let's see how this can have an impact by looking at another visualization. In the below SSD, all four of the targets are storing data. Some of the data is dirty, indicated by red text. We want to write 5 pages worth of data to this SSD. If we time this sequence of writes, the SSD can happily write them to free pages with no need for extra garbage collection. There are sufficient unused pages in the first target.

Now say we have a drive with different data already on it, but we want to write those same 5 pages of data to it. In this drive, we only have 2 pages that are unused, but a number of dirty pages. In order to write 5 pages of data, the SSD will need to spend some time doing garbage collection to make room for the new data. When attempting to time another sequence of writes, some garbage collection will take place to make room for the data, slowing down the write.

In this case, the drive had to move the two non-dirty pages from the top-left target to new locations. By doing this, it was able to make all of the pages on the top-left target dirty, making it safe to erase that data. This made room for the 5 new pages of data to be written. These additional steps significantly slowed down the performance of the write.

This shows how the organization of data on the drive can have an impact on performance. When SSDs have a lot of reads, writes, and deletes, we can end up with SSDs that have degraded performance due to garbage collection. Though you may not be aware, busy SSDs do garbage collection tasks regularly, which can slow down other operations.

These are just two of many reasons why the arrangement of data on a SSD affects its performance.

## [Storage in the cloud](https://planetscale.com/blog/io-devices-and-latency#storage-in-the-cloud)

The shift from tape, to disk, to solid state has allowed durable IO performance to accelerate dramatically over the past several decades. However, there is another phenomenon that has caused an additional shift in IO performance: moving to the cloud.

Though there were companies offering cloud compute services before this, the mass move to cloud gained significant traction when Amazon AWS launched in 2006. Since that time, tens of thousands of companies have moved their app servers and database systems to their cloud and other similar services from Google, Microsoft, and others.

Though there are many upsides to this trend, there are several downsides. One of these is that servers tend to have less permanence. Users rent (virtualised) servers on arbitrary hardware within gigantic data centers. These servers can get shut down at any time for a variety of reasons - hardware failure, hardware replacement, network disconnects, etc. When building platforms on rented cloud infrastructure, computer systems need to be able to tolerate more frequent failures at any moment. This, along with many engineers' desire for dynamically-scaleable storage volumes has led to a new sub-phenomenon: Separation of **storage** and **compute**.

## [Separating storage from compute](https://planetscale.com/blog/io-devices-and-latency#separating-storage-from-compute)

Traditionally, most servers, desktops, laptops, phones and other computing devices have their non-volatile storage directly attached. These are attached with SATA cables, PCIe interfaces, or even built directly into the same SOC as the RAM, CPU, and other components. This is great for speed, but provides the following challenges:

1. If the server goes down, the data goes down with it.
2. The storage is of a fixed size.

For application servers, 1. and 2. are typically not a big deal since they work well in ephemeral environments by design. If one goes down, just spin up a new one. They also don't typically need much storage, as most of what they do happens in-memory.

Databases are a different story. If a server goes down, we don't want to lose our data, and data size grows quickly, meaning we may hit storage limits. _Partly_ due to this, many cloud providers allow you to spin up compute instances with a separately-configurable storage system attached over the network. In other words, using network-attached storage as the default.

When you create a new server in EC2, the default is typically to attach an EBS network storage volume. Many database services including Amazon RDS, Amazon Aurora, Google Cloud SQL, and PlanetScale rely on these types of storage systems that have compute separated from storage over the network. This provides a nice advantage in the that the storage volume can be dynamically resized as data grows and shrinks. It also means that if a server goes down, the data is still safe, and can be re-attached to a different server. This simplicity has come at a cost, however.

## [Local vs network storage](https://planetscale.com/blog/io-devices-and-latency#local-vs-network-storage)

Consider the following simple configuration. In it, we have a server with a CPU, RAM, and direct-attached NVMe SSD. NVMe SSDs are a type of solid state disk that use the non-volatile memory host controller interface specification for blazing-fast IO speed and great bandwidth. In such a setup, the round trip from CPU to memory (RAM) takes about 100 nanoseconds (a nanosecond is 1 billionth of a second). A round trip from the CPU to a locally-attached NVMe SSD takes about 50,000 nanoseconds (50 microseconds).

This makes it pretty clear that it's best to keep as much data in memory as possible for faster IO times. However, we still need disk because (A) memory is more expensive and (B) we need to store our data somewhere permanent. As slow as it may seem here, a locally-attached NVMe SSD is about as fast as it gets for modern storage.

Let's compare this to the speed of a network-attached storage volume, such as EBS. Read and write requires a short network round trip within a data center. The round trip time is significantly worse, taking about 250,000 nanoseconds (250 microseconds, or 0.25 milliseconds).

Using the same cutting-edge SSD now takes an _order of magnitude_ longer to fulfill individual read and write requests. When we have large amounts of sequential IO, the negative impact of this can be reduced, but not eliminated. We have introduced significant _latency_ deterioration for every time we need to hit our storage system.

Another issue with network-attached storage in the cloud comes in the form of limiting IOPS. Many cloud providers that use this model, including AWS and Google Cloud, limit the amount of IO operations you can send over the wire. By default, a GP3 EBS instance on Amazon allows you to send 3000 IOPS per-second. This can be configured higher, but comes at extra cost.

The older GP2 EBS volumes operate with a pool of IOPS that can be built up to allow for occasional bursts. The following visual shows how this works. Note that the burst balance size is smaller here than in reality to make it easier to see.

If instead you have your storage attached directly to your compute instance, there are no artificial limits placed on IO operations. You can read and write as fast as the hardware will allow for.

For as many steps as we've taken forward in IO performance over the years, this seems like a step in the wrong direction. This separation buys some nice conveniences, but at what cost to performance?

How do we overcome issue 1 (data durability) and 2 (drive scalability) while keeping good IOPS performance?

Issue 1 can be overcome with replication. Instead of relying on a single server to store all data, we can replicate it onto several computers. One common way of doing this is to have one server act as the primary, which will receive all write requests. Then 2 or more additional servers get all the data replicated to them. With the data in three places, the likelihood of losing data becomes very small.

Let's look at concrete numbers. As a made up value, say in a given month, there is a 1% chance of a server failing. With a single server, this means we have a 1% chance of losing our data each month. This is an unacceptable for any serious business purpose. However, with three servers, this goes down to 1% × 1% × 1% = 0.0001% chance (1 in one million). At PlanetScale the protection is actually far stronger than even this, as we automatically detect and replace failed nodes in your cluster. We take frequent and reliable backups of the data in your database for added protection.

Problem 2. can be solved, though it takes a bit more manual intervention when working with directly-attached SSDs. We need to ensure that we monitor and get alerted when our disk approaches capacity limits, and then have tools to easily increase capacity when needed. With such a feature, we can have data permanence, scalability, and blazing fast performance. This is exactly what PlanetScale has built with Metal.

Planetscale just announced **[Metal](https://planetscale.com/blog/announcing-metal)**, an industry-leading solution to this problem.

With Metal, you get a full-fledged database cluster set up (Vitess or Postgres), with each database instance running with a direct-attached NVMe SSD drive. Each Metal cluster comes with a primary and two replicas by default for extremely durable data. We allow you to resize your servers with larger drives with just a few clicks of a button when you run up against storage limits. Behind the scenes, we handle spinning up new nodes and migrating your data from your old instances to the new ones with zero downtime.

Perhaps most importantly, with a Metal database, there is no artificial cap on IOPS. You can perform IO operations with minimal latency, and hammer it as hard as you want without being throttled or paying for expensive IOPS classes on your favorite cloud provider.

If you want the ultimate in performance and scalability, **[try Metal today](https://planetscale.com/metal)**.