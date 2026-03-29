---
link: https://last9.io/blog/ubuntu-performance-monitoring/
byline: Faiz Shaikh
site: Last9
date: 2025-04-04T13:28
excerpt: A practical guide to monitoring performance on Ubuntu—tools, tips, and
  commands to keep your system running efficiently.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T18:33
title: The Ultimate Guide to Ubuntu Performance Monitoring
---

When your Ubuntu system slows down, finding the root cause quickly is essential. This comprehensive guide covers everything from basic monitoring tools to advanced techniques for both beginners and system administrators.

## What Makes Ubuntu Performance Monitoring Different?

Ubuntu's Linux foundation provides deeper visibility into system performance compared to other operating systems. The system exposes detailed metrics through the `/proc` filesystem and specialized tools that access kernel-level information.

The `/proc` virtual filesystem serves as a window into the kernel, with files like `/proc/cpuinfo` for CPU specs and `/proc/meminfo` for detailed memory allocation. This transparency enables tracking from individual thread performance to specific memory allocation types.

Ubuntu's modular monitoring approach allows you to combine different tools according to your needs rather than relying on a single monitoring application.

### top - The Basic Performance Tool

```
$ top
```

This command displays:

- CPU usage per core
- Memory consumption
- Running processes and their resource usage
- Load averages (1, 5, and 15-minute intervals)
- Uptime and user session count

The load average figures represent the average number of processes waiting for CPU time. On a quad-core system, a load of 4.0 indicates full CPU utilization. Anything higher suggests performance bottlenecks.

Useful keyboard commands:

- '1' - Display individual CPU core performance
- 'M' - Sort by memory usage
- 'c' - Show full command path
- 'k' - Kill a process (followed by PID)
- 'b' - Enable bold highlighting for running processes
- 'x' - Highlight the sorting column
- 'u [username]' - Filter processes by user
- 'W' - Save current configuration to ~/.toprc

### htop - An Enhanced Alternative

```
$ sudo apt install htop
$ htop
```

Key features:

- Color-coded process visualization
- Mouse support in terminal
- Vertical and horizontal scrolling
- Built-in process management (kill, nice)
- Visual CPU, memory, and swap meters
- Process tree view (F5)

The process tree view is particularly useful for identifying parent-child relationships between processes, helpful when tracking down resource-intensive subprocesses.

Configuration options:

- F2 opens setup menu with display customization
- Save custom layouts for different monitoring scenarios
- Add custom meters in the header (I/O wait time is informative)

### iotop - For Disk Activity Monitoring

```
$ sudo apt install iotop
$ sudo iotop
```

Displays:

- Total disk throughput
- Per-process I/O statistics
- I/O percentage relative to total
- Real-time disk read/write speeds

Keyboard shortcuts:

- 'o' - Show only processes doing active I/O
- 'a' - Accumulate I/O instead of showing current rates
- 'p' - Toggle between programs and threads
- 'i' - Hide idle processes

For capturing intermittent I/O issues:

```
$ sudo iotop -botd 5 > disk_activity.log
```

This records batch output (-b) of only processes doing I/O (-o), showing threads (-t), refreshing every 5 seconds (-d 5), and saving to a log file.

### vmstat - Memory and VM Statistics

```
$ vmstat 3
```

Provides system statistics every 3 seconds:

- Memory (swap, free, buffer, cache)
- CPU (user, system, idle, wait)
- I/O (blocks in/out)
- System (interrupts, context switches)

The first line shows averages since boot; subsequent lines show interval activity. The 'si' and 'so' columns indicate when the system starts swapping to disk, which significantly impacts performance.

### nmon - Comprehensive System Monitor

```
$ sudo apt install nmon
$ nmon
```

Provides a dashboard with toggleable views:

- 'c' for CPU
- 'm' for memory
- 'd' for disks
- 'n' for network
- 't' for top processes
- 'V' for virtual memory stats

For data collection and later analysis:

```
$ nmon -f -s 30 -c 120
```

This captures system stats every 30 seconds for 1 hour (120 samples) as a CSV file, which can be analyzed with nmon_analyzer or imported into spreadsheet software.

### Glances - Modern Terminal Monitor

```
$ sudo apt install glances
$ glances
```

Notable features:

- Web interface (`glances -w`)
- REST API for custom applications
- Cross-platform compatibility
- Built-in alert system
- History mode
- Export capabilities to databases

For remote monitoring:

```
$ glances -w --username glances --password monitoring
```

This starts a web server on port 61208 with authentication, accessible from any device on your network.

For long-term monitoring with visualization:

```
$ glances --export influxdb
```

This sends data to InfluxDB, which can be connected to Grafana for dashboard visualization.

### netstat and ss - Network Connection Monitoring

```
$ netstat -tuln
$ ss -tuln
```

These commands show:

- Open ports
- Active connections
- Waiting connections
- Network statistics
- Connection states

The ss command is newer and more efficient than netstat. For tracking established connections:

```
$ ss -tnp state established
```

To monitor listening applications:

```
$ sudo ss -ltpn
```

For continuous network monitoring:

```
$ watch -n 1 'ss -t -o state established'
```

This refreshes every second to display active TCP connections with timing information.

## Using /proc for Custom Monitoring

The /proc filesystem contains valuable data for custom monitoring:

### /proc/loadavg - System Load

```
$ cat /proc/loadavg
0.45 0.17 0.11 1/292 16870
```

Shows 1, 5, and 15-minute load averages, running/total processes, and the last process ID.

### /proc/meminfo - Memory Details

```
$ cat /proc/meminfo
```

Displays over 40 memory metrics, including total RAM, free memory, available memory, and buffer/cache statistics.

### /proc/diskstats - Disk Activity

```
$ cat /proc/diskstats
```

Contains raw I/O statistics for each device, including sectors read/written and I/O time.

### /proc/stat - CPU and System Statistics

```
$ cat /proc/stat
```

Shows CPU time allocation (user, nice, system, idle, iowait) and other system counters.

## GUI Monitoring Options You Need to Know

### System Monitor - Built-in Solution

Ubuntu includes GNOME System Monitor:

- Real-time resource graphs
- Process management
- Resource usage history
- File systems tab for disk usage

Launch with:

```
$ gnome-system-monitor
```

Features include:

- Process sorting by any resource metric
- Process priority adjustment
- Application termination
- Process search
- Open Files tab (right-click a process → Open Files)
- Memory Maps view (right-click a process → Memory Maps)

### Stacer - All-in-One System Utility

```
$ sudo apt install stacer
$ stacer
```

Provides:

- Dashboard with resource graphs
- System cleaner
- Startup application manager
- Service manager
- Process monitoring
- Resource usage history
- APT package management interface

The System Cleaner can remove:

- Package caches
- Crash reports
- Application logs
- Application caches
- Trash contents

### Conky - Desktop Widget Monitor

```
$ sudo apt install conky-all
$ conky
```

Displays system statistics directly on the desktop. Create a `~/.conkyrc` file to customize:

```
# Basic conky configuration
background yes
use_xft yes
xftalpha 0.8
update_interval 1.0
total_run_times 0
own_window yes
own_window_transparent yes
own_window_hints undecorated,below,sticky,skip_taskbar,skip_pager
double_buffer yes
minimum_size 250 5
maximum_width 400
draw_shades no
draw_outline no
draw_borders no
draw_graph_borders yes
default_color white
default_shade_color black
default_outline_color green
alignment top_right
gap_x 10
gap_y 40
no_buffers yes
uppercase no
cpu_avg_samples 2
override_utf8_locale no

TEXT
${font sans-serif:bold:size=10}SYSTEM ${hr 2}
${font sans-serif:normal:size=8}Uptime: $uptime
Kernel: $kernel

${font sans-serif:bold:size=10}CPU ${hr 2}
${font sans-serif:normal:size=8}CPU Frequency: $freq_g GHz
CPU Usage: $cpu% ${cpubar 4}
${cpugraph 20,400 000000 ffffff}

${font sans-serif:bold:size=10}MEMORY ${hr 2}
${font sans-serif:normal:size=8}RAM Used: $mem / $memmax
RAM Usage: $memperc% ${membar 4}
Swap Used: $swap / $swapmax
Swap Usage: $swapperc% ${swapbar 4}

${font sans-serif:bold:size=10}DISK ${hr 2}
${font sans-serif:normal:size=8}Root: ${fs_used /} / ${fs_size /}
Usage: ${fs_used_perc /}% ${fs_bar 4 /}
I/O Read: ${diskio_read}
I/O Write: ${diskio_write}
${diskiograph 20,400 000000 ffffff}

${font sans-serif:bold:size=10}NETWORK ${hr 2}
${font sans-serif:normal:size=8}Local IP: ${addr wlp2s0}
Down: ${downspeed wlp2s0} kb/s
Up: ${upspeed wlp2s0} kb/s
${downspeedgraph wlp2s0 20,400 000000 ffffff}
${upspeedgraph wlp2s0 20,400 000000 ffffff}

${font sans-serif:bold:size=10}TOP PROCESSES ${hr 2}
${font sans-serif:normal:size=8}Name $alignr PID   CPU%   MEM%
${top name 1} $alignr ${top pid 1} ${top cpu 1} ${top mem 1}
${top name 2} $alignr ${top pid 2} ${top cpu 2} ${top mem 2}
${top name 3} $alignr ${top pid 3} ${top cpu 3} ${top mem 3}
${top name 4} $alignr ${top pid 4} ${top cpu 4} ${top mem 4}
${top name 5} $alignr ${top pid 5} ${top cpu 5} ${top mem 5}
```

💡

Fix Ubuntu monitoring issues—right from your IDE, with AI and Last9 MCP. [Setup [Last9 MCP](https://last9.io/mcp/)] ->[[View demo](https://youtu.be/AQH5xq6qzjI?si=YsgoCc-zY3vL9rnV)]

## Troubleshooting Common Performance Issues

### High CPU Usage

When CPU usage spikes:

1. Identify the process: `top -c` or `htop`
2. Lower process priority: `renice +10 -p [PID]`
3. Terminate if necessary: `kill -15 [PID]` or `kill -9 [PID]` as last resort

Advanced troubleshooting:

- Check for I/O wait in top ('wa' percentage)
- For Java applications: `jstack [PID] > thread_dump.txt`
- For web servers: Check access logs for traffic spikes
- Analyze system calls: `strace -p [PID]`

For kernel processes (names in brackets like `[kworker/0:0]`), check kernel logs:

```
$ dmesg | tail
```

### Memory Issues

When memory runs low:

1. Identify memory-intensive processes: `ps aux --sort=-%mem | head -10`
2. Clear the page cache if needed: `sudo sync && sudo echo 3 > /proc/sys/vm/drop_caches`
3. Adjust swappiness: `sudo sysctl vm.swappiness=10`

Install earlyoom to prevent system lockups:

```
$ sudo apt install earlyoom
$ sudo systemctl enable earlyoom
$ sudo systemctl start earlyoom
```

For memory leak detection:

```
$ sudo apt install valgrind
$ valgrind --tool=memcheck --leak-check=yes ./your_program
```

For detailed memory analysis:

```
$ sudo apt install smem
$ smem -tk
```

### Disk I/O Problems

When disk operations cause slowdowns:

1. Identify I/O-intensive processes: `sudo iotop`
2. Check disk health: `sudo smartctl -a /dev/sda`
3. Prioritize important processes: `ionice -c2 -n0 -p [PID]`

Optimize I/O scheduler for your storage type:

```
$ cat /sys/block/sda/queue/scheduler
```

For SSDs:

```
$ echo deadline | sudo tee /sys/block/sda/queue/scheduler
```

For permanent configuration, add `elevator=deadline` to GRUB_CMDLINE_LINUX in `/etc/default/grub`, then run `sudo update-grub`.

Install application-level disk cache:

```
$ sudo apt install preload
```

### Network Performance

For network issues:

Verify DNS resolution:

```
$ dig google.com
```

Check for packet loss:

```
$ ping -c 20 8.8.8.8
```

Test connection speed:

```
$ curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python -
```

Identify bandwidth-intensive applications:

```
$ sudo apt install nethogs iftop
$ sudo nethogs
```

Improve DNS resolution by editing `/etc/systemd/resolved.conf`:

```
[Resolve]
DNS=1.1.1.1 8.8.8.8
```

Then restart the service:

```
$ sudo systemctl restart systemd-resolved
```

## Enterprise Monitoring Solutions

### Prometheus and Grafana Setup

1. Import the "Node Exporter Full" dashboard (ID: 1860) in Grafana

Configure alerts in Prometheus:

```
groups:
- name: example
  rules:
  - alert: HighCPULoad
    expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU load (instance {{ $labels.instance }})"
      description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

Install and configure Grafana:

```
$ sudo apt install grafana
$ sudo systemctl enable grafana-server
$ sudo systemctl start grafana-server
```

Configure Prometheus in `/etc/prometheus/prometheus.yml`:

```
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100', 'server2:9100', 'server3:9100']
```

Install node_exporter on each server:

```
$ sudo apt install prometheus-node-exporter
```

Install Prometheus:

```
$ sudo apt install prometheus
```

Advanced monitoring components:

- Blackbox exporter for endpoint monitoring
- Alertmanager for alert routing
- cAdvisor for container metrics

### Last9 for High-Cardinality Observability at Scale

If you’re after a managed observability solution that won’t wreck your budget _or_ cut corners on performance, give Last9 a look. Our pricing is based on events ingested—simple, predictable, and no surprises.

[Last9](https://last9.io/) powers high-cardinality observability at scale for teams at Disney+ Hotstar, CleverTap, and Replit.

As a telemetry data platform, we’ve handled monitoring for 11 of the 20 largest live-streaming events ever. With native support for OpenTelemetry and Prometheus, Last9 brings together metrics, logs, and traces—so you get better performance, real-time insights, and smarter alerts without the usual chaos.

💡

Don’t forget to check out our [docs](https://last9.io/docs/) for step-by-step instructions on setting up log analysis and making the most of Last9.

### ELK Stack for Log Analysis

Create a Logstash pipeline for syslog in `/etc/logstash/conf.d/syslog.conf`:

```
input {
  beats {
    port => 5044
  }
}

filter {
  if [fileset][module] == "system" {
    if [fileset][name] == "auth" {
      grok {
        match => { "message" => "%{SYSLOGTIMESTAMP:[system][auth][timestamp]} %{SYSLOGHOST:[system][auth][hostname]} %{DATA:[system][auth][program]}(?:\[%{POSINT:[system][auth][pid]}\])?: %{GREEDYMULTILINE:[system][auth][message]}" }
        pattern_definitions => { "GREEDYMULTILINE" => "(.|\n)*" }
        remove_field => "message"
      }
      date {
        match => [ "[system][auth][timestamp]", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    manage_template => false
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

Install Kibana:

```
$ sudo apt install kibana
$ sudo systemctl enable kibana && sudo systemctl start kibana
```

Install Filebeat:

```
$ sudo apt install filebeat
$ sudo nano /etc/filebeat/filebeat.yml
```

Install Logstash:

```
$ sudo apt install logstash
$ sudo systemctl enable logstash && sudo systemctl start logstash
```

Install Elasticsearch:

```
$ wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
$ echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list
$ sudo apt update && sudo apt install elasticsearch
$ sudo systemctl enable elasticsearch && sudo systemctl start elasticsearch
```

Watch for concerning log patterns:

- Authentication failures
- Out of memory errors
- Storage errors
- Service restarts
- Slow database queries

💡

For a closer look at how the Linux OOM Killer decides what to terminate, [this guide](https://last9.io/blog/understanding-the-linux-oom-killer/) walks through the logic behind it.

## Automated Monitoring Scripts to Help You

### Multi-Resource Alert Script

```
#!/bin/bash
# system_monitor.sh - Resource threshold monitor

# Configuration
EMAIL="your@email.com"
CPU_THRESHOLD=90
MEMORY_THRESHOLD=90
DISK_THRESHOLD=90
SWAP_THRESHOLD=50
LOAD_FACTOR=0.8  # Multiple of CPU cores

# Initialize alert flag
ALERT=0
ALERT_MESSAGE="SYSTEM ALERT on $(hostname) at $(date):\n\n"

# Check CPU usage
CPU_IDLE=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/")
CPU_USAGE=$(echo "100 - $CPU_IDLE" | bc)

if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
    ALERT=1
    ALERT_MESSAGE+="HIGH CPU USAGE: ${CPU_USAGE}%\n"
    ALERT_MESSAGE+="Top CPU processes:\n"
    ALERT_MESSAGE+="$(ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%cpu | head -n 6)\n\n"
fi

# Check memory usage
MEM_TOTAL=$(free | grep Mem | awk '{print $2}')
MEM_AVAIL=$(free | grep Mem | awk '{print $7}')
MEM_USAGE_PCT=$(echo "scale=2; (($MEM_TOTAL - $MEM_AVAIL) / $MEM_TOTAL) * 100" | bc)

if (( $(echo "$MEM_USAGE_PCT > $MEMORY_THRESHOLD" | bc -l) )); then
    ALERT=1
    ALERT_MESSAGE+="HIGH MEMORY USAGE: ${MEM_USAGE_PCT}%\n"
    ALERT_MESSAGE+="Top memory processes:\n"
    ALERT_MESSAGE+="$(ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%mem | head -n 6)\n\n"
fi

# Check disk usage
DISK_USAGE=$(df -h | grep -vE "tmpfs|udev|loop" | awk '{print $5}' | sed 's/%//g' | sort -nr | head -n1)

if [ "$DISK_USAGE" -gt "$DISK_THRESHOLD" ]; then
    ALERT=1
    ALERT_MESSAGE+="HIGH DISK USAGE: ${DISK_USAGE}%\n"
    ALERT_MESSAGE+="Disk usage details:\n"
    ALERT_MESSAGE+="$(df -h | grep -vE 'tmpfs|udev|loop')\n\n"
fi

# Check swap usage
if [ -n "$(free | grep Swap)" ]; then
    SWAP_TOTAL=$(free | grep Swap | awk '{print $2}')
    if [ "$SWAP_TOTAL" -ne 0 ]; then
        SWAP_USED=$(free | grep Swap | awk '{print $3}')
        SWAP_PCT=$(echo "scale=2; ($SWAP_USED / $SWAP_TOTAL) * 100" | bc)
        
        if (( $(echo "$SWAP_PCT > $SWAP_THRESHOLD" | bc -l) )); then
            ALERT=1
            ALERT_MESSAGE+="HIGH SWAP USAGE: ${SWAP_PCT}%\n\n"
        fi
    fi
fi

# Check system load
CPU_CORES=$(grep -c "processor" /proc/cpuinfo)
MAX_LOAD=$(echo "$CPU_CORES * $LOAD_FACTOR" | bc)
CURRENT_LOAD=$(cut -d " " -f 1 /proc/loadavg)

if (( $(echo "$CURRENT_LOAD > $MAX_LOAD" | bc -l) )); then
    ALERT=1
    ALERT_MESSAGE+="HIGH SYSTEM LOAD: ${CURRENT_LOAD} (threshold: ${MAX_LOAD})\n"
    ALERT_MESSAGE+="Current processes:\n"
    ALERT_MESSAGE+="$(ps -eo pid,ppid,user,%cpu,%mem,cmd --sort=-%cpu | head -n 6)\n\n"
fi

# Send alert if thresholds exceeded
if [ "$ALERT" -eq 1 ]; then
    echo -e "$ALERT_MESSAGE" | mail -s "SYSTEM ALERT: $(hostname)" "$EMAIL"
    logger "System alert triggered: Resource thresholds exceeded"
fi
```

### Scheduled Monitoring with cron

Set up regular checks:

```
# Run system monitor every 15 minutes
*/15 * * * * /path/to/system_monitor.sh

# Daily disk space report at 7 AM
0 7 * * * df -h | mail -s "Daily Disk Report - $(hostname)" your@email.com

# Check service status hourly
0 * * * * systemctl is-active nginx mysql postgresql | grep -v "active" && echo "Service check failed on $(hostname)" | mail -s "SERVICE ALERT" your@email.com
```

### Simple Process Watchdog

```
#!/bin/bash
# process_watchdog.sh - Ensure critical processes are running

# Configuration
PROCESS_LIST=("nginx" "mysql" "ssh" "cron")
EMAIL="your@email.com"

for PROCESS in "${PROCESS_LIST[@]}"; do
    if ! pgrep -x "$PROCESS" > /dev/null; then
        echo "Process $PROCESS is not running! Attempting to restart..." | tee -a /var/log/watchdog.log
        
        # Try to restart the service
        systemctl restart "$PROCESS"
        
        # Check if restart was successful
        sleep 5
        if ! pgrep -x "$PROCESS" > /dev/null; then
            echo "Failed to restart $PROCESS service!" | tee -a /var/log/watchdog.log
            echo "Process $PROCESS failed to restart on $(hostname) at $(date)" | mail -s "PROCESS ALERT: $PROCESS down" "$EMAIL"
        else
            echo "Successfully restarted $PROCESS service." | tee -a /var/log/watchdog.log
        fi
    fi
done
```

## Desktop vs Server Monitoring Differences

Server monitoring priorities:

- Uptime and availability
- Service health metrics
- Network throughput
- Remote accessibility
- Security events
- Backup status

Desktop monitoring priorities:

- Application responsiveness
- GUI performance
- Battery/power management
- User experience metrics
- Peripheral connectivity
- Storage space

Tailor your monitoring approach based on the system's primary function.

## Best Practices for Ongoing Monitoring

1. Establish performance baselines
    - Document normal resource usage patterns
    - Record peak and average metrics
    - Note seasonal or time-based patterns
2. Set appropriate alert thresholds
    - Avoid alert fatigue with reasonable thresholds
    - Consider time-based thresholds (different for business hours)
    - Layer alerts by severity
3. Implement log rotation
    - Prevent storage consumption by monitoring logs
    - Configure logrotate for all application logs
    - Archive historical data efficiently
4. Document your monitoring setup
    - Keep records of all monitoring tools and configurations
    - Document alert response procedures
    - Maintain recovery playbooks
5. Test monitoring regularly
    - Verify alerts are working as expected
    - Simulate failure scenarios
    - Review monitoring coverage quarterly

## Conclusion

Effective Ubuntu performance monitoring requires understanding your system's baseline performance, selecting appropriate tools, and implementing proper alert thresholds.

Start with basic tools like top and htop for immediate insights, then progress to more specialized tools for specific issues. For enterprise environments, consider [Last9](https://last9.io/), or implementing Prometheus with Grafana or the ELK stack for comprehensive monitoring.

💡

Which monitoring techniques have you found most effective for your Ubuntu systems? Share your experiences with our [Discord community](https://discord.com/invite/W8gMppQC4b).