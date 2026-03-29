---
link: https://last9.io/blog/monitoring-disk-io-on-linux/
byline: Anjali Udasi
site: Last9
date: 2025-04-21T09:01
excerpt: Learn how to monitor and optimize disk I/O performance on Linux with
  this comprehensive guide to better manage system resources.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T15:32
title: A Comprehensive Guide to Monitoring Disk I/O on Linux
---

In a Linux environment, understanding how your storage devices perform can mean the difference between a system that flies and one that crawls. Whether you're troubleshooting performance issues or fine-tuning your server setup, getting familiar with Linux disk I/O statistics is an essential skill for any tech professional.

This guide breaks down everything you need to know about Linux disk I/O stats - from basic concepts to practical monitoring techniques that you can implement today.

## What Are Linux Disk I/O Stats?

Linux disk I/O (input/output) stats are metrics that show how your storage devices interact with your system. They measure read and write operations, throughput, wait times, and other key performance indicators.

These stats help you understand:

- How busy your disks are
- Whether your storage is becoming a bottleneck
- Which processes are most disk-intensive
- If your current storage configuration meets your needs

For developers and system maintainers, these metrics are like a health dashboard for your disk subsystem.

💡

To better understand where system logs fit into the picture while tracking disk I/O, it helps to know [what’s inside `/var/log`](https://last9.io/blog/what-is-var-log/).

## Why Monitor Disk I/O in Linux?

Keeping an eye on disk I/O statistics isn't just for when things go wrong. Proactive monitoring helps you:

- Spot potential issues before they cause downtime
- Make data-driven decisions about hardware upgrades
- Optimize application performance
- Balance workloads effectively
- Plan capacity for future needs

In environments where performance matters (and where doesn't it?), tracking these metrics gives you a competitive edge.

## 4 Core Linux Disk I/O Metrics You Should Know

Let's break down the fundamental metrics that matter most:

### IOPS (Input/Output Operations Per Second)

IOPS measures how many read and write operations your disk completes each second. Higher numbers generally mean better performance, but context matters.

Different workloads have different ideal IOPS profiles:

- Database servers typically need high IOPS
- File servers might prioritize throughput over IOPS
- Development environments may have varying requirements

### Throughput

Throughput measures the amount of data transferred to or from your disk over time, usually in megabytes per second (MB/s).

This metric matters most when handling large files or when multiple processes access the disk simultaneously.

### Latency

Latency represents the time it takes for a disk operation to complete. Lower latency means your system feels more responsive.

Key latency metrics include:

- Read latency: Time to retrieve data
- Write latency: Time to save data
- Queue time: Time operations wait before processing

### Utilization

Utilization shows the percentage of time the disk is busy processing requests. Consistently high utilization (>80-90%) often signals a bottleneck.

💡

If you're monitoring disk I/O to troubleshoot performance issues, it’s also worth checking out [this guide to Linux event logs](https://last9.io/blog/linux-event-logs-your-troubleshooting-guide/) for more context on what the system is trying to tell you.

Linux provides several built-in tools to check disk performance. Here are the most useful ones:

### iostat

The `iostat` command delivers comprehensive disk activity statistics. It's part of the `sysstat` package and provides both current and cumulative stats.

Basic usage:

```
iostat -xdz 1
```

This displays extended disk statistics (`-x`), for disks only (`-d`), without showing metrics since boot (`-z`), refreshing every second.

Key metrics from `iostat` output:

- `r/s` and `w/s`: Reads and writes per second
- `rkB/s` and `wkB/s`: Kilobytes read/written per second
- `await`: Average time (in milliseconds) for I/O requests
- `%util`: Percentage of CPU time during which I/O requests were issued

### vmstat

While primarily a memory statistics tool, `vmstat` also provides useful disk I/O information.

Basic usage:

```
vmstat 1
```

This shows system statistics refreshed every second. Look for:

- `bi`: Blocks received from a block device (blocks/s)
- `bo`: Blocks sent to a block device (blocks/s)

### iotop

Think of `iotop` as "top" but for disk operations. It shows which processes are using the most disk resources.

Basic usage:

```
iotop
```

You'll need to install it first on most distributions:

```
# On Debian/Ubuntu
sudo apt install iotop

# On RHEL/CentOS
sudo yum install iotop
```

### /proc/diskstats

For the curious, raw disk stats are available directly from the `/proc/diskstats` file:

```
cat /proc/diskstats
```

This provides low-level statistics for all block devices.

## Making Sense of Linux Disk I/O Data

Having data is one thing; understanding what it means is another. Here's how to interpret common patterns:

### High Read/Write Operations with Low Throughput

**What it means:** Your system is handling many small I/O operations.

**Common causes:** Database operations, log writes, and small file access.

**Potential fixes:** Consider using an SSD for better IOPS, implement caching, or optimize your database queries.

### Low Operations Count with High Throughput

**What it means:** Your system is processing fewer but larger operations.

**Common causes:** Large file transfers, backups, and media processing.

**Potential fixes:** If performance is lagging, consider RAID configurations or NVMe drives for higher sustained throughput.

### High Latency (await Times)

**What it means:** Operations are taking longer than they should.

**Common causes:** Disk saturation, hardware issues, resource contention.

**Potential fixes:** Distribute workload across multiple disks, check for hardware issues, or investigate competing processes.

## Practical Monitoring Strategies You Should Know

Now that you understand the metrics and tools, let's look at how to implement effective monitoring:

### Real-Time Monitoring Command

For a quick overview of current disk performance, this one-liner combines `iostat` with `watch`:

```
watch -n 1 'iostat -xdz 1 1'
```

### Scheduled Checks with Cron

To keep historical data, set up regular checks with cron:

```
# Add to crontab
*/5 * * * * iostat -xdz 1 1 >> /var/log/disk_stats.log
```

This logs disk stats every 5 minutes.

### Setting Up Alerts

For proactive management, set up alerts when disk metrics cross thresholds:

```
# Example bash script for basic alerting
if [ $(iostat -xd 1 2 | awk '/sda/ {print $14}' | tail -1) -gt 90 ]; then
  echo "Disk utilization above 90%" | mail -s "Disk Alert" admin@example.com
fi
```

## How to Optimize Disk I/O Performance

After monitoring and identifying issues, try these optimization techniques:

### File System Tuning

Different file systems have different performance characteristics:

- **ext4**: Good general-purpose file system
- **XFS**: Excellent for large files and high-performance requirements
- **Btrfs**: Great for snapshots and modern features

### Adjusting Kernel Parameters

Fine-tune how Linux handles I/O with sysctl parameters:

```
# Increase dirty buffer thresholds for better throughput
echo 30 > /proc/sys/vm/dirty_ratio
echo 10 > /proc/sys/vm/dirty_background_ratio

# Make permanent in /etc/sysctl.conf
vm.dirty_ratio = 30
vm.dirty_background_ratio = 10
```

### I/O Schedulers

Linux offers different I/O schedulers, each with pros and cons:

- **CFQ (Completely Fair Queuing)**: Balanced for mixed workloads
- **Deadline**: Better for latency-sensitive operations
- **NOOP**: Minimal overhead, good for SSDs

Check your current scheduler:

```
cat /sys/block/sda/queue/scheduler
```

Change it temporarily:

```
echo deadline > /sys/block/sda/queue/scheduler
```

## Advanced Monitoring with Last9

While built-in tools are powerful, modern observability platforms like [Last9](https://last9.io/) offer more comprehensive monitoring capabilities.

### [Last9](https://last9.io/): Simplified Observability

If you're looking for an observability solution that’s easy on your budget without sacrificing key capabilities, Last9 is a standout choice.

By integrating metrics, logs, and traces through OpenTelemetry and Prometheus, we provide a unified approach to monitoring. This allows you to correlate performance issues across your entire tech stack, not just disk I/O.

Trusted by high-scale operations like Probo and Replit, Last9 ensures critical infrastructure is always under control. We’ve managed observability for 11 of the 20 largest live-streaming events in history.

[![Replit's review on Last9](https://last9.ghost.io/content/images/2025/04/Replit.png)](https://last9.io/customers/)

Replit's review on Last9

### Using collectd for Continuous Monitoring

The collectd daemon provides continuous monitoring with minimal overhead:

```
# Install collectd
sudo apt install collectd

# Configure disk plugin in /etc/collectd/collectd.conf
LoadPlugin disk

<Plugin disk>
  Disk "sda"
  IgnoreSelected false
</Plugin>
```

This collects detailed disk metrics that can be stored in various backends for visualization.

## How to Resolve Common Disk I/O Performance Issues

Let's address some frequent disk I/O problems:

### Random vs. Sequential Access Patterns

**Issue:** Random access is much slower than sequential, especially on HDDs.

**Solution:** For databases with random access patterns, SSDs provide dramatically better performance.

### Write Amplification

**Issue:** Some operations cause more physical writes than logical writes.

**Solution:** Use file systems with good write combining, consider SSDs with good controllers, and monitor TRIM support.

### RAID Considerations

**Issue:** Different RAID levels have different I/O characteristics.

**Solution:** Choose wisely:

- RAID 0: Highest performance but no redundancy
- RAID 1: Good read performance, write performance equals single disk
- RAID 5/6: Good read performance, write penalty for parity
- RAID 10: Best all-around performance with redundancy

## How Disk I/O Stats Impact Containers in Linux

With containerization becoming standard, monitoring disk I/O in containerized environments requires special attention:

### Docker Storage Drivers

Different storage drivers have different I/O profiles:

- overlay2: Good general-purpose driver with reasonable performance
- devicemapper: Better isolation but potential performance overhead
- direct-lvm: Better performance than loop-lvm

Check your current driver:

```
docker info | grep "Storage Driver"
```

### Limiting Container I/O

To prevent one container from monopolizing disk resources:

```
# Limit writes to 10MB/s and reads to 20MB/s
docker run --device-write-bps /dev/sda:10mb --device-read-bps /dev/sda:20mb nginx
```

## Conclusion

Mastering Linux disk I/O statistics gives you the insights needed to optimize storage performance, troubleshoot bottlenecks, and make informed decisions about your infrastructure.

## FAQs

### How often should I monitor disk I/O stats?

For most systems, checking every 5-15 minutes is sufficient for baseline monitoring. During troubleshooting or performance tuning, real-time monitoring (every 1-5 seconds) provides more actionable data.

### What's a healthy disk utilization percentage?

Generally, sustained utilization below 70% is considered healthy. Between 70-85% indicates you're approaching capacity limits, while consistent usage above 85% typically signals a bottleneck that needs addressing.

### How do I identify which process is causing high disk I/O?

The `iotop` command shows processes sorted by disk usage. Alternative methods include using `ps` with the `io` option or checking `/proc/<pid>/io` for specific process stats.

### Do SSDs need different monitoring approaches than HDDs?

Yes. With SSDs, focus more on IOPS and less on seek times. Also monitor TRIM operations and wear leveling. Tools like `smartctl` provide SSD-specific health metrics.

### How can I simulate disk I/O load for testing?

Tools like `fio` (Flexible I/O Tester) and `dd` can generate controlled disk loads:

```
# Example fio command for random reads
fio --name=random-read --ioengine=libaio --direct=1 --bs=4k --size=4G --numjobs=1 --rw=randread
```

### What's the difference between buffered and direct I/O?

Buffered I/O uses the kernel's page cache as an intermediary, which can improve performance through caching but may hide actual disk performance. Direct I/O bypasses this cache, showing true disk performance but potentially reducing throughput.