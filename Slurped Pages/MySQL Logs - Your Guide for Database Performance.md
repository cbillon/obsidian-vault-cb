---
link: https://last9.io/blog/mysql-logs/
byline: Faiz Shaikh
site: Last9
date: 2025-03-28T08:58
excerpt: Struggling with slow queries? MySQL logs hold the answers! Learn how to
  read them, fix issues, and boost your database performance.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T18:37
title: "MySQL Logs: Your Guide for Database Performance"
---

MySQL logs are basically your database's diary – they record everything happening behind the scenes. Think of them as the black box of your database operations.

You've got error logs showing you when things go sideways, query logs documenting every question asked of your database, and binary logs tracking changes like they're gossip in a small town.

Here's why you should give a damn: When your app slows to a crawl and your boss is breathing down your neck, these logs are often the difference between being the office hero or the person everyone side-eyes at the next meeting.

But more than just crisis management, MySQL logs give you:

- Proactive insights to prevent problems before they start
- Data for capacity planning and resource allocation
- Evidence for security audits and compliance requirements
- Historical patterns to inform database design decisions

💡

If you're working with MySQL logs, you might also find Linux event logs useful for troubleshooting system issues—[here’s a guide to help](https://last9.io/blog/linux-event-logs-your-troubleshooting-guide/).

## The MySQL Log Types You Need to Know

Let's break down the major players in the MySQL logs game:

### Error Log

This is your first stop when things get weird. The error log catches:

- Startup and shutdown messages (including crash information)
- Critical errors that make your database throw a tantrum
- Warnings about things that seem fishy but won't crash the system
- Authentication failures and access-denied errors
- Plugin and storage engine-specific messages

**Where to find it:** By default, it's named `hostname.err` and lives in your data directory, but your config might have it elsewhere.

The error log format varies slightly between MySQL versions. In MySQL 8.0+, the default format includes timestamps and error codes that look like this:

```
2023-04-15T14:23:45.342345Z 0 [Note] [MY-010311] [Server] Server socket created on IP: '::'.
2023-04-15T14:23:45.400394Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.32) initializing of server in progress as process 1234
2023-04-15T14:23:47.711522Z 0 [Warning] [MY-013242] [Server] --character-set-server: 'utf8' is currently an alias for the character set UTF8MB3, but will be an alias for UTF8MB4 in a future release. Please consider using UTF8MB4 in order to be unambiguous.
```

You can customize the verbosity of this log with the `--log-error-verbosity` option, of accepting values from 1 (errors only) to 3 (errors, warnings, and notes).

### General Query Log

Consider this as your database's journal where it writes down every single request that comes its way. It logs:

- All queries received by the server (including those with syntax errors)
- Client connection and disconnection events (with user and hostname information)
- When each statement started executing
- Server-side prepared statements and their execution
- Commands issued through stored procedures

While super helpful for debugging, this log can grow faster than your AWS bill, so use it wisely and temporarily.

A typical general query log entry looks like this:

```
2023-04-15T15:04:32.512413Z    42 Connect   app_user@app_server on my_database using TCP/IP
2023-04-15T15:04:32.512730Z    42 Query     SELECT user_id, username FROM users WHERE active = 1 LIMIT 50
2023-04-15T15:04:33.631024Z    42 Query     BEGIN
2023-04-15T15:04:33.631553Z    42 Query     UPDATE users SET last_login = NOW() WHERE user_id = 12345
2023-04-15T15:04:33.632210Z    42 Query     COMMIT
2023-04-15T15:04:38.102913Z    42 Quit
```

The number after the timestamp (42 in this example) is the connection ID, which helps you track queries from the same client session.

### Slow Query Log

My personal favorite – this one flags queries that are taking too long to execute. It's like having a timer on your database operations.

```
-- Set the slow query time threshold (in seconds)
SET GLOBAL long_query_time = 2;

-- Enable the slow query log
SET GLOBAL slow_query_log = 'ON';

-- Set the slow query log file
SET GLOBAL slow_query_log_file = '/var/log/mysql/mysql-slow.log';

-- Log queries that don't use indexes (optional but recommended)
SET GLOBAL log_queries_not_using_indexes = 'ON';

-- Set a minimum number of rows a query must examine to be logged when not using indexes
SET GLOBAL min_examined_row_limit = 1000;
```

This log is pure gold for performance tuning. It shows you exactly where to focus your optimization efforts.

A slow query log entry contains detailed execution metrics:

```
# Time: 2023-04-15T16:12:45.471232Z
# User@Host: app_user[app_user] @ app_server [192.168.1.100]  Id: 43
# Query_time: 5.234091  Lock_time: 0.000021  Rows_sent: 1452  Rows_examined: 3894371
SET timestamp=1681570365;
SELECT customer_id, SUM(total_amount) as revenue 
FROM orders 
WHERE order_date BETWEEN '2022-01-01' AND '2022-12-31' 
GROUP BY customer_id 
ORDER BY revenue DESC;
```

The key metrics here are:

- `Query_time`: Total execution time in seconds
- `Lock_time`: How long the query wait for table locks
- `Rows_sent`: Number of rows returned to the client
- `Rows_examined`: Number of rows the server had to scan

That last one is crucial – a high ratio between rows examined and rows sent indicates an inefficient query that could benefit from better indexing.

### Binary Log

This heavy hitter records all changes to your data. It's essential for:

- Replication between database servers
- Point-in-time recovery operations
- Auditing data changes
- Cross-datacenter synchronization
- Blue/green deployments with minimal downtime

It's stored in a binary format, but you can view it with the mysqlbinlog utility:

```
mysqlbinlog /var/lib/mysql/mysql-bin.000001
```

Binary logs use a different format than other logs – they contain "events" that represent changes to the database structure or content. Common event types include:

- `QUERY_EVENT`: DDL statements like CREATE TABLE or ALTER TABLE
- `ROWS_EVENT`: DML operations like INSERT, UPDATE, or DELETE
- `XID_EVENT`: Transaction commit markers
- `ROTATE_EVENT`: Indicates when MySQL switches to a new binary log file

The binary log has three different formats:

- `STATEMENT`: Logs the actual SQL statements (smaller logs, but less reliable for replication)
- `ROW`: Logs the actual row changes (larger logs, but more reliable replication)
- `MIXED`: Automatically chooses between STATEMENT and ROW based on the query type

For most modern setups, the ROW format is recommended:

```
SET GLOBAL binlog_format = 'ROW';
```

### Relay Log

If you're using replication, your replica servers maintain relay logs. These are similar to binary logs but are created on replica servers as they receive events from the source server.

The replica I/O thread reads events from the source's binary log and writes them to the relay log. Then, the replica SQL thread reads events from the relay log and applies them to the local database.

Monitoring relay logs helps you track replication lag and troubleshoot replication issues.

## Setting Up Logging Like You Know What You're Doing

Getting your logs configured right is half the battle. Here's how to set them up without breaking a sweat:

### Through the MySQL Configuration File

The most common way is through your `my.cnf` or `my.ini` file:

```
[mysqld]
# Error Log
log_error = /var/log/mysql/error.log
log_error_verbosity = 3

# General Query Log
general_log = 1
general_log_file = /var/log/mysql/query.log

# Slow Query Log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
log_queries_not_using_indexes = 1
min_examined_row_limit = 1000
log_slow_admin_statements = 1
log_slow_slave_statements = 1

# Binary Log
log_bin = /var/log/mysql/mysql-bin.log
binlog_format = ROW
binlog_row_image = FULL
expire_logs_days = 14
max_binlog_size = 100M
binlog_cache_size = 4M
sync_binlog = 1

# Relay Log (for replicas)
relay_log = /var/log/mysql/mysql-relay-bin
relay_log_purge = 1
```

Let's break down some of the less obvious settings:

- `log_slow_admin_statements`: Also logs slow administrative commands like ALTER TABLE
- `log_slow_slave_statements`: Logs slow statements executed by the SQL thread on replicas
- `binlog_row_image`: Controls how much information is logged for row-based logging
- `sync_binlog`: Controls how often binary log is synced to disk (1 = every transaction, more durable but slower)
- `binlog_cache_size`: Memory buffer for binary log during transactions

### Dynamic Configuration

Need to turn logs on or off without restarting? I've got you:

```
-- Turn on the general query log
SET GLOBAL general_log = 'ON';

-- Turn off when you're done (your disk space will thank you)
SET GLOBAL general_log = 'OFF';

-- Check current log settings
SHOW VARIABLES LIKE '%log%';

-- See log file locations
SELECT @@global.general_log_file, @@global.slow_query_log_file, @@global.log_error;

-- Check if binary logging is enabled
SHOW VARIABLES LIKE 'log_bin';

-- See binary log information
SHOW BINARY LOGS;

-- View status of binary logging
SHOW MASTER STATUS;

-- For replicas, view relay log status
SHOW SLAVE STATUS\G
```

Remember that dynamic settings don't persist through server restarts unless you also update your configuration file.

## Log Analysis That Solves Problems

Let's get to the good stuff – using these logs to fix real problems.

### Finding Performance Bottlenecks

The slow query log is your MVP here. Once you've collected some data, run:

```
# Basic analysis with mysqldumpslow
mysqldumpslow -s t /var/log/mysql/slow.log | head -20

# Group similar queries and sort by total time
mysqldumpslow -s t -a /var/log/mysql/slow.log | head -20

# Find queries that scan the most rows
mysqldumpslow -s r /var/log/mysql/slow.log | head -20

# Find queries with the worst ratio of rows examined to rows sent
pt-query-digest --filter '$event->{Rows_examined} / ($event->{Rows_sent} || 1) > 100' /var/log/mysql/slow.log
```

This shows your top 20 slowest queries sorted by time. Look for patterns like:

- Queries without proper indexes (high Rows_examined count)
- JOINs that are bringing your server to its knees
- Queries returning massive result sets
- Queries with high Lock_time (indicating contention)
- Repetitive queries that could be cached

For deeper analysis, Percona Toolkit's pt-query-digest is worth its weight in gold:

```
pt-query-digest /var/log/mysql/slow.log > query_report.txt
```

This produces a comprehensive report with:

- Overall stats on query performance
- Top Queries by different metrics (time, count, rows)
- Query fingerprints to identify patterns
- Detailed analysis of each problematic query

Once you've identified problematic queries, you can address them with:

```
-- Adding an index to speed up filtering
CREATE INDEX idx_order_date ON orders (order_date);

-- Rewriting a query to use a more selective index
-- Before: SELECT * FROM users WHERE LOWER(email) = 'user@example.com'
-- After: SELECT * FROM users WHERE email = 'user@example.com'

-- Using EXPLAIN to verify your optimizations
EXPLAIN SELECT customer_id, SUM(total_amount) as revenue 
FROM orders 
WHERE order_date BETWEEN '2022-01-01' AND '2022-12-31' 
GROUP BY customer_id 
ORDER BY revenue DESC;
```

### Troubleshooting Connection Issues to Solve

When clients can't connect, the error log becomes your best friend:

```
# Look for recent connection errors
grep "connection" /var/log/mysql/error.log | tail -50

# Check for "too many connections" errors
grep "connections" /var/log/mysql/error.log | grep "too many"

# Look for access denied errors
grep "Access denied" /var/log/mysql/error.log | tail -20

# Check for aborted connections
grep "Aborted connection" /var/log/mysql/error.log | tail -50
```

This might reveal:

- max_connections limits being hit (increase in my.cnf if needed)
- Authentication problems (check user privileges with `SHOW GRANTS FOR 'user'@'host'`)
- Network constraints (check firewall settings and MySQL's bind-address)
- Timeout issues (adjust wait_timeout and interactive_timeout)
- Client-side disconnections without proper cleanup

A common issue is running out of connections. Track current usage with:

```
-- Check current connection count
SHOW STATUS LIKE 'Threads_connected';

-- See connection limit
SHOW VARIABLES LIKE 'max_connections';

-- View active connections
SELECT id, user, host, db, command, time, state, info 
FROM information_schema.processlist 
ORDER BY time DESC;

-- Kill long-running queries (if necessary)
KILL QUERY 12345; -- connection ID
```

### Tracking Down Mysterious Crashes

When MySQL decides to take an unscheduled nap:

```
# Look at the last events before shutdown
grep -A 10 "Shutdown" /var/log/mysql/error.log

# Check for crash-related messages
grep -i "crash" /var/log/mysql/error.log
grep -i "exception" /var/log/mysql/error.log
grep -i "signal" /var/log/mysql/error.log

# Look for out of memory errors
grep -i "memory" /var/log/mysql/error.log
dmesg | grep -i "out of memory" | grep mysql

# Check InnoDB-specific errors
grep -i "innodb" /var/log/mysql/error.log | grep -i "error"
```

This often reveals:

- The last few errors before everything went south
- Memory allocation failures (adjust innodb_buffer_pool_size)
- Table corruption issues (run CHECK TABLE)
- Storage engine problems
- File system errors (check disk space and inode usage)

For persistent crashes, enable the core file:

```
[mysqld]
core-file
```

And analyze it with:

```
gdb /usr/sbin/mysqld /path/to/core
```

💡

MySQL logs are just one piece of the puzzle. To understand logs better, [this guide on log data](https://last9.io/blog/what-is-log-data/) breaks it down.

### Diagnosing Replication Issues

Binary and relay logs are key for troubleshooting replication:

```
# Check replication status
mysql -e "SHOW SLAVE STATUS\G"

# Look for replication errors in the error log
grep -i "replication" /var/log/mysql/error.log

# Examine the relay log position
mysqlbinlog --start-position=12345 /var/log/mysql/mysql-relay-bin.000123

# Check binary log events around a problematic position
mysqlbinlog --start-position=12340 --stop-position=12350 /var/log/mysql/mysql-bin.000456
```

Common replication issues and fixes:

- **Duplicate key errors**: Skip the problematic statement with `SET GLOBAL sql_slave_skip_counter = 1; START SLAVE;`
- **Temporary network issues**: Increase `MASTER_CONNECT_RETRY` in `CHANGE MASTER TO`
- **Binary log corruption**: Restart replication from a new position or restore from backup
- **Schema differences**: Ensure tables are identical between source and replica

## Advanced Log Management That Will Impress Your Boss

As your databases grow, managing logs becomes its own challenge. Here's how to level up:

### Log Rotation

Nobody wants a 50GB log file. Set up logrotate to keep things tidy:

```
/var/log/mysql/*.log {
        daily
        rotate 7
        missingok
        create 640 mysql adm
        compress
        delaycompress
        sharedscripts
        postrotate
                if [ -x /usr/bin/mysqladmin ] && [ -f /etc/mysql/debian.cnf ]; then
                        /usr/bin/mysqladmin --defaults-file=/etc/mysql/debian.cnf flush-logs
                fi
        endscript
}
```

Let's break down this config:

- `daily`: Rotate logs once per day
- `rotate 7`: Keep 7 days of logs before deleting
- `missingok`: Don't error if the log file is missing
- `create 640 mysql adm`: Create new log files with these permissions
- `compress`: Compress rotated logs to save space
- `delaycompress`: Wait until the next rotation cycle to compress
- `sharedscripts`: Run postrotate script once for all logs
- `postrotate`: Tell MySQL to flush its logs after rotation

For binary logs, use MySQL's built-in expiration:

```
-- Set binary log expiration to 7 days
SET GLOBAL expire_logs_days = 7;

-- In MySQL 8.0+, use the more precise setting
SET GLOBAL binlog_expire_logs_seconds = 604800;

-- Manually purge logs older than a specific date
PURGE BINARY LOGS BEFORE '2023-04-10 00:00:00';

-- Manually purge logs before a specific file
PURGE BINARY LOGS TO 'mysql-bin.000123';
```

### Centralized Logging

For multi-server setups, shipping logs to a central location is a game-changer. Consider using:

- ELK Stack (Elasticsearch, Logstash, Kibana)
- Graylog
- Splunk (if you've got cash to burn)
- Prometheus with Loki and Grafana (trending combo for cloud-native environments)

This gives you a unified view across your database fleet.

Here's a sample Filebeat configuration to ship MySQL logs to Elasticsearch:

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/mysql/error.log
  fields:
    log_type: mysql_error
    server: db-prod-1

- type: log
  enabled: true
  paths:
    - /var/log/mysql/slow.log
  fields:
    log_type: mysql_slow
    server: db-prod-1

output.elasticsearch:
  hosts: ["elasticsearch:9200"]
  index: "mysql-logs-%{+yyyy.MM.dd}"
```

For MySQL metrics, Prometheus with mysqld_exporter gives you real-time monitoring of your database health alongside logs.

### Real-time Log Analysis

Set up real-time monitoring to catch issues as they happen:

```
# Watch for new errors in real-time
tail -f /var/log/mysql/error.log | grep -i "error"

# Monitor the slow query log for new entries
tail -f /var/log/mysql/slow.log | grep -i "Query_time"

# Create a simple dashboard of current MySQL status
watch -n 1 'mysql -e "SHOW GLOBAL STATUS LIKE '\''Queries%'\''" -e "SHOW GLOBAL STATUS LIKE '\''Slow_queries%'\''" -e "SHOW GLOBAL STATUS LIKE '\''Threads_connected%'\''"'
```

For production environments, consider tools like:

- Percona Monitoring and Management (PMM)
- VividCortex (now SolarWinds Database Performance Monitor)
- Datadog Database Monitoring

These tools combine log analysis with performance metrics for a complete view of your database health.

## When to Watch MySQL Logs Like a Hawk

Some situations demand closer log monitoring:

|Scenario|Logs to Watch|What to Look For|Recommended Actions|
|---|---|---|---|
|After Deployment|General Query, Slow Query|New queries behaving badly, unexpected query patterns|Compare query patterns before/after, fine-tune new queries|
|During Peak Traffic|Error, Slow Query|Resource constraints, lock contention, connection spikes|Identify bottlenecks, implement query caching, connection pooling|
|Following Config Changes|Error|Startup problems, warnings about parameters, performance changes|Validate parameter values, check for deprecated options|
|Before Maintenance|Binary|Ensure replication is caught up, transaction completion|Confirm zero replication lag, wait for long-running transactions|
|After Failover|Error, Binary|Replication issues, clients reconnecting, data inconsistencies|Verify replica consistency, update connection strings if needed|
|During Data Migration|Slow Query, Error|Schema conversion issues, data type mismatches, constraint violations|Monitor progress, check for errors in data transformation|
|Security Audits|General Query, Error|Suspicious access patterns, privilege escalation attempts|Look for unauthorized access, unusual permission changes|

## MySQL Log Gotchas That Might Bite You

Watch out for these common pitfalls:

### Performance Impact

Aggressive logging (especially the general query log) can hurt performance. Be selective about what you enable in production.

Each log write requires disk I/O, which can be especially problematic on busy systems. Consider these alternatives:

- Enable logging only during troubleshooting sessions
- Use sampling (log only a percentage of queries)
- Offload logs to a separate disk or SSD to minimize I/O contention
- In high-performance environments, consider async_binlog for the binary log (though this risks data loss during crashes)

```
-- Set binary log to asynchronous mode (faster but less safe)
SET GLOBAL sync_binlog = 0;

-- Enable sampling for the slow query log (log ~10% of slow queries)
SET GLOBAL log_slow_rate_limit = 10;
SET GLOBAL log_slow_rate_type = 'query';
```

### Storage Space

Logs can grow fast. I once saw a busy server generate 30GB of binary logs in a day. Set up monitoring to alert you before disks fill up.

```
# Set up a simple disk space check
df -h | grep '/var/log' | awk '{ print $5 }' | sed 's/%//' | { read used; if [ $used -gt 80 ]; then echo "Disk space critical: $used%"; fi }

# Check total size of MySQL logs
du -sh /var/log/mysql/

# Find the largest log files
find /var/log/mysql/ -type f -name "*.log*" -exec ls -lh {} \; | sort -k5nr | head -10
```

To avoid surprises, implement log size monitoring in your favorite monitoring system with alerts at 75% and 90% capacity.

### Security Considerations

Query logs might contain sensitive data. Make sure your log files have proper permissions:

```
# Set correct permissions
chmod 640 /var/log/mysql/*.log
chown mysql:adm /var/log/mysql/*.log

# Check for world-readable logs (a security risk)
find /var/log/mysql/ -type f -perm -004 -exec ls -l {} \;

# Verify no unauthorized users can access logs
ls -la /var/log/mysql/
```

For systems handling sensitive data:

- Implement log masking for personally identifiable information (PII)
- Set up log encryption at rest
- Establish log access auditing
- Create a log retention policy that aligns with your compliance requirements

Also be aware that binary logs may contain sensitive data even if you're filtering it from query logs. Consider using the following to exclude specific databases from binary logging:

```
[mysqld]
binlog-ignore-db=sensitive_data_db
```

### Timezone Confusion

MySQL logs can use different timezones, leading to confusion during debugging:

```
-- Check MySQL's timezone setting
SELECT @@global.time_zone;

-- Set to your local timezone for easier correlation with application logs
SET GLOBAL time_zone = 'America/New_York';

-- Or use UTC for consistency across distributed systems
SET GLOBAL time_zone = '+00:00';
```

Always confirm timezone settings before correlating events between different logs.

## Taking Your MySQL Log Skills to the Next Level

Ready to become a true MySQL logs wizard? Try these next steps:

### Automated Analysis

Write scripts to parse and alert on log patterns. For example, this bash one-liner finds queries that scan more than 1000 rows:

```
grep "rows examined" /var/log/mysql/slow.log | awk '$NF > 1000 {print}' | mail -s "High scan queries detected" you@example.com
```

For more advanced automation:

```
#!/usr/bin/env python3
import re
import sys
from datetime import datetime

# Simple slow query log parser to find particularly bad queries
with open('/var/log/mysql/slow.log', 'r') as f:
    content = f.read()

# Find query blocks
query_blocks = content.split('# Time:')[1:]  # Skip header

bad_queries = []
for block in query_blocks:
    # Extract metrics
    query_time_match = re.search(r'Query_time: (\d+\.\d+)', block)
    rows_examined_match = re.search(r'Rows_examined: (\d+)', block)
    rows_sent_match = re.search(r'Rows_sent: (\d+)', block)
    
    if not all([query_time_match, rows_examined_match, rows_sent_match]):
        continue
        
    query_time = float(query_time_match.group(1))
    rows_examined = int(rows_examined_match.group(1))
    rows_sent = int(rows_sent_match.group(1))
    
    # Calculate efficiency ratio
    if rows_sent > 0:
        efficiency = rows_examined / rows_sent
    else:
        efficiency = rows_examined
    
    # Find the actual query
    query_lines = []
    capture = False
    for line in block.split('\n'):
        if line.startswith('SET timestamp='):
            capture = True
            continue
        if capture and line.strip():
            query_lines.append(line)
    
    query = '\n'.join(query_lines)
    
    # Flag particularly inefficient queries
    if efficiency > 1000 and query_time > 1:
        bad_queries.append({
            'query': query,
            'time': query_time,
            'efficiency': efficiency,
            'rows_examined': rows_examined,
            'rows_sent': rows_sent
        })

# Report the worst offenders
if bad_queries:
    print(f"Found {len(bad_queries)} inefficient queries")
    for i, q in enumerate(sorted(bad_queries, key=lambda x: x['efficiency'], reverse=True)[:5]):
        print(f"\n--- Bad Query #{i+1} ---")
        print(f"Efficiency ratio: {q['efficiency']:.2f} rows scanned per row returned")
        print(f"Query time: {q['time']} seconds")
        print(f"Rows examined: {q['rows_examined']}, Rows sent: {q['rows_sent']}")
        print("Query:")
        print(q['query'])
```

### Visualization

Turn log data into actionable insights with visualization tools. Grafana with the MySQL data source can create dashboards showing:

- Query performance trends
- Error frequency over time
- Connection patterns
- Replication lag
- Index usage statistics

A sample Grafana dashboard might include panels for:

1. **Query Overview**
    - Total queries per second
    - Slow queries per minute
    - Average query execution time
2. **Error Tracking**
    - Error count by type
    - Authentication failures
    - Deadlock incidents
3. **Resource Usage**
    - Connection count over time
    - Table lock wait time
    - Temporary tables created on the disk
4. **Replication Health**
    - Source/replica lag
    - Replication errors
    - Binary log size growth

### Advanced Debugging Techniques

When standard log analysis isn't enough, try these advanced techniques:

**Core Dump Analysis**For crash debugging:

```
# Generate a stack trace from a core dump
gdb -ex "thread apply all bt" -ex "quit" /usr/sbin/mysqld /var/lib/mysql/core.1234
```

**Performance Schema Metrics**MySQL's Performance Schema provides deeper insights than logs alone:

```
-- Find queries with the highest latency
SELECT digest_text, count_star, avg_timer_wait/1000000000 as avg_latency_ms
FROM performance_schema.events_statements_summary_by_digest
ORDER BY avg_timer_wait DESC
LIMIT 10;

-- Identify tables with the most I/O
SELECT object_schema, object_name, count_read, count_write, count_fetch
FROM performance_schema.table_io_waits_summary_by_table
ORDER BY count_read + count_write DESC
LIMIT 10;
```

**Process List Sampling**Capture the process list at regular intervals to identify patterns:

```
while true; do mysql -e "SHOW FULL PROCESSLIST" >> processlist.log; sleep 5; done
```

**Trace Thread Execution**MySQL allows you to trace specific connections:

```
-- Start the debug trace for connection 123
SET SESSION debug = '+d,info,query,general,error';
```

## Wrap-Up

MySQL logs might seem like a boring technical detail, but they're often the first place smart DBAs and DevOps pros look when trouble strikes.

Remember these key takeaways:

- Error logs tell you what broke
- Slow query logs show you what to optimize
- General query logs reveal what your application is doing
- Binary logs protect your data and enable replication
- Good log management is as important as logging itself

💡

What's your experience with MySQL logs? Jump into our [Discord community](https://discord.com/invite/W8gMppQC4b) and share your war stories – we're all in this together.