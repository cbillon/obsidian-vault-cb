---
link: https://last9.io/blog/linux-log-file-locations/
byline: Anjali Udasi
site: Last9
date: 2025-06-09T16:08
excerpt: Learn where Linux stores logs, what each file does, and how to use them
  for debugging, monitoring, and keeping your systems in check.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T15:12
title: A Complete Guide to Linux Log File Locations and Their Usage
---

Linux log files are text-based records that capture system events, application activities, and user actions. They're stored primarily in the `/var/log` directory and provide essential information for debugging issues, monitoring system health, and maintaining security.

This guide covers the most important Linux log files and a few detailed techniques for reading and analyzing them.

## How Linux Logging Systems Work and Store Data

Linux systems usually rely on two logging mechanisms that work side by side: `rsyslog` and `journald`. Each handles logs differently, but together they cover most observability needs.

#### Traditional rsyslog (syslog)

This is the older, file-based logging system still used in most distributions.

- Logs are stored as plain text files in `/var/log/`.
- This makes it easy to read logs manually or process them using tools like `grep`, `awk`, or `sed`.

Each log entry follows a standard format:

```
timestamp hostname service[PID]: message
```

For example:  
`Dec 25 14:30:42 server01 sshd[1234]: Accepted password for user from 192.168.1.100 port 52890 ssh2`  
This tells you that at 14:30 on Dec 25, the SSH daemon (`sshd`) accepted a password-based login from a user on that IP address.

#### Journald (from systemd)

Systemd introduced its logging system — `journald` which works a bit differently.

- Stores logs in a structured binary format
- Supports rich metadata with each log entry
- Indexes logs for fast querying and filtering
- You interact with it using `journalctl`, a command-line tool with powerful options

Instead of scanning flat files, you can filter logs by time, unit, service name, or even error level.

#### How They Work Together

In most Linux setups:

- `journald` captures everything — system logs, service logs, kernel messages.
- It can forward logs to `rsyslog`, which then writes them to disk under `/var/log/`.

This hybrid setup gives you:

- Structured access via `journalctl` (great for scripting, dashboards)
- Traditional file-based logs that work well with legacy tools

💡

For more on monitoring system security events through logs, check out our detailed [guide on auditd logs.](https://last9.io/blog/auditd-logs/)

## Critical System Log Files Every Administrator Should Monitor

Linux systems generate logs for almost everything, from hardware events to user activity. These are the ones you should keep a watch on regularly.

### General System Logs: `/var/log/syslog` (Debian/Ubuntu) or `/var/log/messages` (Red Hat/CentOS)

This is the go-to log file for most administrators. It captures messages from:

- The kernel
- System services
- Applications
- Hardware drivers

If something goes wrong and you're not sure where to look, start here. It gives you a bird’s-eye view of system activity, service startups, hardware detection, network changes, and more.

**Useful commands:**

```
# Monitor system messages in real-time
tail -f /var/log/syslog

# Search for recent error messages
grep -i "error" /var/log/syslog | tail -20

# View messages from the last hour
grep "$(date --date='1 hour ago' '+%b %d %H'):" /var/log/syslog
```

### Kernel Events and Hardware Logs: `/var/log/kern.log`

This log is all about the Linux kernel, which means:

- Hardware events
- Driver info
- Kernel module loads/unloads
- Memory issues

If your system freezes, reboots unexpectedly, or throws hardware-related errors, this file often holds the clue.

**Look for signs of trouble:**

```
# Check for hardware errors
grep -i "error\|fail" /var/log/kern.log

# View kernel messages about USB devices
grep -i "usb" /var/log/kern.log

# Check for memory issues or OOM events
grep -i "memory\|oom" /var/log/kern.log
```

💡

For insights into monitoring disk I/O performance on Linux systems, check out [this detailed blog](https://last9.io/blog/monitoring-disk-io-on-linux/).

## How to Monitor Authentication and Security Logs on Linux

If you're concerned about who’s accessing your systems, Linux gives you plenty of options.

### `/var/log/auth.log` (Ubuntu/Debian) or `/var/log/secure` (Red Hat/CentOS)

This log tracks all authentication-related activity, including:

- SSH logins (successful and failed)
- `sudo` usage and privilege escalations
- User account changes (e.g., password resets, account creation)
- System login attempts (local and remote)

It’s your first line of defense for spotting brute-force attacks, misused privileges, or suspicious login patterns.

**Useful commands:**

```
# Check for failed login attempts
grep "Failed password" /var/log/auth.log

# Monitor sudo usage with recent entries
grep "sudo:" /var/log/auth.log | tail -10

# Check for successful SSH logins
grep "Accepted password" /var/log/auth.log

# Find login attempts from a specific IP
grep "192.168.1.100" /var/log/auth.log

# Look for user account changes
grep -E "(useradd|usermod|userdel)" /var/log/auth.log
```

### `/var/log/btmp` and `/var/log/wtmp`: Binary Login Logs

These two logs store login records in a binary format:

- `btmp` → failed login attempts
- `wtmp` → successful logins, logouts, reboots

They can’t be read with `cat` or `less`, but they’re useful for long-term auditing and spotting login anomalies.

**Commands to explore these logs:**

```
# View failed login attempts
lastb | head -20

# View successful login history
last | head -20

# Login activity for a specific user
last username

# Logins from a specific IP
last | grep 192.168.1.100

# System reboot history
last reboot
```

💡

To better understand how Linux security logs help track and analyze potential threats, see [our in-depth overview](https://last9.io/blog/linux-security-logs/) here!

## Use Boot and Hardware Logs for Troubleshooting

If your system doesn’t boot cleanly or certain services fail to start, these logs help pinpoint what went wrong early in the startup sequence.

### `/var/log/boot.log`: Track System Startup and Service Initialization

This log captures the full boot sequence from kernel initialization to service startups. It’s helpful when your system starts, but some services don’t run as expected.

**Useful commands:**

```
# View the full boot log
less /var/log/boot.log

# Find any errors or failed services
grep -i "error\|fail" /var/log/boot.log

# Check which services started (or didn’t)
grep -i "service\|start" /var/log/boot.log
```

## Application-Specific Log Files and How They Help

Here are key application log locations and what each one reveals to help with faster, more targeted troubleshooting:

### **Apache Web Server Logs**

Location: `/var/log/apache2/` or `/var/log/httpd/`

Apache maintains separate logs for access and errors.

- The access log captures every incoming HTTP request client IP, URLs, status codes, and user agents.
- The error log records issues during request processing, including configuration problems and runtime errors.

These logs are useful for analyzing traffic patterns, identifying broken links (404s), and detecting suspicious activity like bot scans.

**Common checks:**

```
# Watch live requests
tail -f /var/log/apache2/access.log

# Search for server-side issues
grep -i "error" /var/log/apache2/error.log

# Find broken link activity
grep " 404 " /var/log/apache2/access.log

# Look for signs of scan attempts
grep -E "(admin|wp-|config)" /var/log/apache2/access.log

# Identify your most visited pages
awk '{print $7}' /var/log/apache2/access.log | sort | uniq -c | sort -nr | head -10
```

### **MySQL Database Logs**

Location: `/var/log/mysql/`

MySQL maintains multiple logs:

- The error log tracks startup/shutdown events, crashes, and serious errors.
- The slow query log (if enabled) flags queries taking longer than expected — helpful for performance tuning.

These logs are essential for spotting misconfigurations, sluggish queries, or failed access attempts.

**Useful commands:**

```
# Follow MySQL error log
tail -f /var/log/mysql/error.log

# Monitor slow queries (if enabled)
tail -f /var/log/mysql/slow.log

# Spot connectivity issues
grep -i "connection\|connect" /var/log/mysql/error.log

# Catch failed logins
grep "Access denied" /var/log/mysql/error.log
```

### **Cron Job Logs**

Location: `/var/log/cron`

Every scheduled task on your system, whether system-level maintenance or user-defined jobs, is recorded here.

Each entry shows when a job was supposed to run and whether it actually did. Cron logs are useful for debugging missed backups, script failures, or verifying scheduled tasks are firing on time.

**Quick checks:**

```
# View today’s cron activity
grep "$(date '+%b %d')" /var/log/cron

# Look for failed jobs
grep -i "error\|fail" /var/log/cron

# Check jobs run by a specific user
grep "username" /var/log/cron

# Review execution lines
grep "CMD" /var/log/cron | tail -20
```

### `cat` – View the Entire Log File at Once

- Best for small files or when you need to send the whole log to another command.
- Avoid using it on large logs—it'll dump everything into your terminal, fast.

**Examples:**

```
# Read the full boot log
cat /var/log/boot.log

# Count failed SSH login attempts
cat /var/log/auth.log | grep "Failed password" | wc -l
```

### `less` – Navigate Logs Interactively

- Perfect for skimming through large logs at your own pace.
- Supports forward/backward scrolling and keyword searches.
- Doesn’t load the whole file at once, so it's memory-efficient.

**Examples:**

```
# Open a log with scroll and search
less /var/log/syslog

# In 'less', use '/' to search, 'n' for next match, 'q' to quit
```

### `tail` – View the Most Recent Entries

- Shows the latest lines of a file—exactly what you need during active debugging.
- With `-f`, it follows the file in real time, like a live stream of events.

**Examples:**

```
# Last 20 lines of the system log
tail -n 20 /var/log/syslog

# Monitor logs in real-time
tail -f /var/log/syslog

# Watch multiple files together
tail -f /var/log/syslog /var/log/auth.log

# Show last 50 lines, then continue following
tail -n 50 -f /var/log/apache2/access.log
```

### `head` – Peek at the Beginning

- Useful for understanding the structure or metadata at the start of a file.
- Handy for checking when a file was rotated or last started.

**Examples:**

```
# Inspect log format or headers
head -n 10 /var/log/syslog

# Check when a new access log began
head -n 5 /var/log/apache2/access.log
```

## Advanced Log Filtering and Search Techniques

### Comprehensive grep Usage for Log Analysis

The `grep` command is the most powerful tool for searching through log files. Understanding its various options allows you to quickly find specific information in large log files and create complex search patterns.

```
# Basic case-insensitive search
grep -i "error" /var/log/syslog

# Search with line numbers
grep -n "Failed password" /var/log/auth.log

# Show context around matches (5 lines before and after)
grep -A 5 -B 5 "kernel panic" /var/log/kern.log

# Search multiple files recursively
grep -r "connection refused" /var/log/

# Count occurrences
grep -c "404" /var/log/apache2/access.log

# Show only filenames with matches
grep -l "error" /var/log/*.log

# Invert match (show lines that don't contain pattern)
grep -v "DEBUG" /var/log/application.log
```

### Regular Expression Patterns for Complex Log Searches

Regular expressions allow you to create sophisticated search patterns that match specific log entry formats, extract data, and identify complex patterns that simple text searches can't handle.

```
# Find IP addresses
grep -E "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" /var/log/apache2/access.log

# Find email addresses in mail logs
grep -E "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b" /var/log/mail.log

# Find timestamps in ISO format
grep -E "\b[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}\b" /var/log/syslog

# Find multiple patterns
grep -E "(error|fail|warn|critical)" /var/log/syslog

# Find SSH login attempts with specific pattern
grep -E "sshd\[[0-9]+\]: (Accepted|Failed)" /var/log/auth.log
```

### Time-Based Log Filtering Techniques

Filtering logs by periods is crucial for troubleshooting issues that occurred at specific times or analyzing system behavior during particular periods.

```
# Find entries from specific hour today
grep "$(date '+%b %d %H'):" /var/log/syslog

# Find entries from yesterday
grep "$(date --date='yesterday' '+%b %d')" /var/log/syslog

# Find entries from specific date
grep "Dec 25" /var/log/syslog

# Find entries from last 2 hours
grep -E "$(date '+%b %d') ($(date '+%H')|$(date --date='1 hour ago' '+%H')):" /var/log/syslog
```

## Systemd Journal Logs Using journalctl

The [`journalctl` command](https://last9.io/blog/journalctl-commands-cheatsheet/) provides powerful querying capabilities for systemd logs, offering filtering options that aren't available with traditional text-based log files.

**Basic journalctl Operations**

```
# View all journal entries
journalctl

# View entries in reverse chronological order (newest first)
journalctl -r

# Follow journal in real-time
journalctl -f

# Show kernel messages only
journalctl -k

# Show entries from current boot
journalctl -b

# Show entries from previous boot
journalctl -b -1
```

**Service-Specific and Advanced Filtering**

```
# Show logs for specific service
journalctl -u apache2

# Show logs for specific service with follow
journalctl -u mysql -f

# Filter by priority (emerg, alert, crit, err, warning, notice, info, debug)
journalctl -p err

# Filter by time range
journalctl --since "2024-01-01 00:00:00" --until "2024-01-01 23:59:59"

# Show logs since 1 hour ago
journalctl --since "1 hour ago"

# Show last 50 entries
journalctl -n 50

# Show logs for specific process ID
journalctl _PID=1234

# Show logs for specific user
journalctl _UID=1000
```

**Output Formatting and Advanced journalctl Features**

```
# Output in JSON format for processing
journalctl -o json

# Output in short format (one line per entry)
journalctl -o short

# Show disk usage of journal files
journalctl --disk-usage

# Verify journal file integrity
journalctl --verify

# Show available boots
journalctl --list-boots
```

## Log Rotation Configuration and Management

[Log rotation](https://last9.io/blog/log-rotation-in-linux/) prevents log files from consuming excessive disk space while maintaining historical data for analysis and compliance requirements. The `logrotate` utility handles this automatically based on configuration files.

**Understanding logrotate Configuration**

The main configuration file `/etc/logrotate.conf` contains global settings, while individual application configurations are typically stored in `/etc/logrotate.d/`.

```
# View current logrotate configuration
cat /etc/logrotate.conf

# List all logrotate configurations
ls -la /etc/logrotate.d/

# Test logrotate configuration without actually rotating
sudo logrotate -d /etc/logrotate.conf

# Force immediate rotation (useful for testing)
sudo logrotate -f /etc/logrotate.conf
```

**Creating Custom Log Rotation Rules**

```
# Example configuration for custom application logs
# File: /etc/logrotate.d/myapp
/var/log/myapp/*.log {
    daily                    # Rotate daily
    rotate 30               # Keep 30 days of logs
    compress                # Compress rotated logs
    delaycompress          # Don't compress the most recent rotated file
    missingok              # Don't error if log file is missing
    notifempty             # Don't rotate empty files
    create 644 root root   # Create new log file with these permissions
    postrotate
        systemctl reload myapp
    endscript
}
```

**Log Rotation Monitoring and Troubleshooting**

```
# Check logrotate status
cat /var/lib/logrotate/status

# View logrotate execution logs
grep logrotate /var/log/syslog

# Test specific configuration
sudo logrotate -d /etc/logrotate.d/apache2
```

💡

Now, fix production Linux log issues instantlyright from your IDE, with AI and [Last9 MCP](https://last9.io/mcp/). Bring real-time production context — logs, metrics, and traces into your local environment to auto-fix code faster.

## Troubleshooting Common System Issues Using Log File Analysis

When a service fails to start, crashes mid-run, or misbehaves silently in production, logs offer the clearest window into what went wrong. The goal isn’t to guess—it’s to build a sequence of clues that help you isolate and resolve the issue efficiently.

Start with the basics. Use `systemctl status` to check the current state of the service. It usually gives you a quick summary of the last few log entries and exit codes.

```
systemctl status apache2
```

For a deeper look, especially when the issue happened in the recent past, tap into `journalctl`. You can scope it to just the service and a time range.

```
journalctl -u apache2 --since "10 minutes ago"
```

Once logs confirm that the service is failing during startup, the next step is to check for obvious misconfigurations. Many services, like Apache or Nginx, include built-in config validators:

```
apache2ctl configtest
```

Networking issues are another common culprit. A port might already be bound by a different process, or the interface might not be up yet.

```
ss -tuln | grep :80
```

Don’t overlook the environment either—resource exhaustion often blocks services from launching cleanly. Use these to check system memory and disk usage:

```
free -h
df -h
```

Finally, inspect file and directory permissions. If the service can't read its config or write to its log files, it'll silently fail or exit with vague errors.

```
ls -la /var/log/apache2/
ls -la /etc/apache2/
```

For databases like MySQL, the steps are similar, but the symptoms might show up as slow queries, failed connections, or corrupted tables. Start by validating the service status and scanning the error log:

```
systemctl status mysql
journalctl -u mysql --since "1 hour ago"
tail -50 /var/log/mysql/error.log
```

If you’re seeing connection errors or max connection warnings, the fix might be tuning pool limits or increasing resource allocation:

```
grep -i "too many connections" /var/log/mysql/error.log
```

Disk space issues are common in database failures, especially if binary logs or temporary tables fill up the partition:

```
df -h /var/lib/mysql
```

And if you're still unsure about config issues, MySQL lets you review its active defaults:

```
mysql --help --verbose | grep "Default options"
```

Security incidents—whether it's brute force attempts, privilege escalation, or tampered user accounts—also surface in logs, though the clues are more scattered. `/var/log/auth.log` is a good starting point.

To identify brute-force login attempts, count failed SSH logins by IP:

```
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr | head -20
```

If you suspect an IP is suspicious, dig deeper into its login history:

```
SUSPICIOUS_IP="192.168.1.100"
grep "$SUSPICIOUS_IP" /var/log/auth.log | sort
grep "$SUSPICIOUS_IP" /var/log/auth.log | grep "Accepted"
```

If an attacker did get in, trace what happened next:

```
grep "$SUSPICIOUS_IP" /var/log/auth.log | grep -A 5 "Accepted"
grep "$SUSPICIOUS_IP" /var/log/syslog
```

Privilege escalation usually leaves a different footprint. Look at recent `sudo` activity, user creation, and permission changes:

```
grep "sudo:" /var/log/auth.log | tail -50
grep "su:" /var/log/auth.log
grep -E "(useradd|usermod)" /var/log/auth.log
grep -i "chmod\|chown" /var/log/syslog
```

If someone set up persistent access, modified cron jobs might reveal that:

```
grep -i "crontab" /var/log/syslog
```

Then there are the performance problems—the sluggish server, the periodic spikes, the memory ballooning. Logs won’t always give you one clear error message, but they’ll leave behind patterns.

For memory pressure, search for signs of OOM kills:

```
grep -i "out of memory\|oom-kill\|killed process" /var/log/syslog
```

You can dig into which processes were killed:

```
grep "Killed process" /var/log/syslog | awk '{print $10, $11, $12}' | sort | uniq -c
```

Swap usage and memory allocation failures will also appear in `dmesg`:

```
dmesg | grep -i "memory\|malloc"
```

Disk-related problems? Look for space exhaustion or I/O errors:

```
grep -i "no space left\|disk full" /var/log/syslog
grep -i "i/o error\|input/output error" /var/log/syslog
```

You can even trace hardware-level issues by checking `dmesg` for ATA or SCSI disk errors:

```
dmesg | grep -i "disk\|ata\|scsi" | grep -i error
```

Finally, when it comes to network hiccups — unreachable services, DNS delays, or random timeouts, logs are often your best bet before reaching for packet captures:

```
grep -i "link\|network\|eth0" /var/log/syslog
grep -i "dns\|resolve" /var/log/syslog
grep -i "connection refused\|connection timeout" /var/log/syslog
```

And for systems using UFW or similar firewalls, blocked packets show up here:

```
grep -i "blocked\|denied" /var/log/ufw.log
```

## A Quick Reference Guide to Linux Log File Locations

|Service/Application|Primary Log Location|Secondary Logs|Description|
|---|---|---|---|
|System Messages|`/var/log/syslog` (Debian/Ubuntu)<br>`/var/log/messages` (RHEL/CentOS)|`/var/log/kern.log`|General system events, kernel messages|
|Authentication|`/var/log/auth.log` (Debian/Ubuntu)<br>`/var/log/secure` (RHEL/CentOS)|`/var/log/btmp`, `/var/log/wtmp`|Login attempts, sudo usage, security events|
|Apache Web Server|`/var/log/apache2/access.log`<br>`/var/log/apache2/error.log`|`/var/log/apache2/other_vhosts_access.log`|HTTP requests, server errors|
|Nginx Web Server|`/var/log/nginx/access.log`<br>`/var/log/nginx/error.log`|Site-specific logs in `/var/log/nginx/`|HTTP requests, server errors|
|MySQL Database|`/var/log/mysql/error.log`|`/var/log/mysql/slow.log`<br>`/var/log/mysql/mysql.log`|Database errors, slow queries|
|PostgreSQL|`/var/log/postgresql/`|Cluster-specific subdirectories|Database operations, errors|
|SSH Daemon|Logged to `/var/log/auth.log` or `/var/log/secure`|`/var/log/sshd.log` (some systems)|SSH connections, authentication|
|Cron Jobs|`/var/log/cron`|User-specific cron logs|Scheduled task execution|
|Mail Server|`/var/log/mail.log` (Debian/Ubuntu)<br>`/var/log/maillog` (RHEL/CentOS)|`/var/log/mail.err`, `/var/log/mail.warn`|Email server operations|
|FTP Server|`/var/log/vsftpd.log` or `/var/log/proftpd/`|Service-specific locations|FTP connections, transfers|
|Firewall (UFW)|`/var/log/ufw.log`|Included in `/var/log/syslog`|Firewall rule matches, blocked connections|
|DNS (BIND)|`/var/log/named/` or `/var/log/bind/`|`/var/log/syslog` for basic messages|DNS queries, zone transfers|
|DHCP Server|`/var/log/dhcp.log` or `/var/log/syslog`|Service-specific configuration|DHCP lease assignments|

💡

And if there's something we missed or you'd like to explore further, hop into our [Discord community](https://discord.com/invite/W8gMppQC4b). There's a dedicated channel where you can chat with other developers and discuss your specific use case.

## FAQs

**Q: Where are most Linux log files stored, and how are they organized?** A: Most Linux log files are stored in `/var/log/` and their subdirectories. System-wide logs like `/var/log/syslog` contain general messages, while application-specific logs are typically in subdirectories like `/var/log/apache2/` or `/var/log/mysql/`. The organization follows a hierarchical structure based on the type of service or system component generating the logs.

**Q: What's the difference between using tail -f and journalctl -f for monitoring logs?** A: `tail -f` monitors plain text log files and shows new lines as they're appended to the file. `journalctl -f` monitors systemd journal logs stored in binary format and provides more sophisticated filtering options. Use `tail -f` for traditional syslog files and `journalctl -f` for systemd service logs, which offer better structure and metadata.

**Q: How can I effectively search for specific periods in log files?** A: Use `grep` with date patterns for syslog files: `grep "Dec 25 14:" /var/log/syslog` for specific hours. For systemd logs, use `journalctl --since "2024-01-01 00:00:00" --until "2024-01-01 23:59:59"`. You can also use relative times, like `journalctl --since "1 hour ago"` for recent entries.

**Q: What are the most important log files for security monitoring?** A: Focus on `/var/log/auth.log` (or `/var/log/secure` on RHEL) for authentication events, `/var/log/syslog` for general security messages, `/var/log/btmp` for failed login attempts, and application-specific logs like web server access logs. These files capture login attempts, privilege escalation, and access patterns that indicate security incidents.

**Q: How do I set up log rotation to prevent log files from consuming too much disk space?** A: Use `logrotate` with configuration files in `/etc/logrotate.d/`. Create rules specifying rotation frequency (daily, weekly, monthly), number of files to keep, compression options, and post-rotation actions. For example, `daily rotate 30 compress delaycompress` keeps 30 days of compressed logs.

**Q: What commands should I use to analyze high-traffic web server logs efficiently?** A: Use `awk` for statistical analysis: `awk '{print $1}' access.log | sort | uniq -c | sort -nr` for top IP addresses, `awk '{print $9}' access.log | sort | uniq -c` for HTTP status codes, and `grep` with patterns for specific errors or attacks. Combine with `tail` or time-based filtering for recent activity analysis.