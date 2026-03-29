---
link: https://last9.io/blog/postgres-logs-101/
byline: Anjali Udasi
site: Last9
date: 2025-02-06T17:03
excerpt: Learn the essentials of PostgreSQL logs, including types, configuration
  tips, and troubleshooting strategies to optimize your database performance.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T18:55
title: "Postgres Logs 101: Types, Configuration, and Troubleshooting"
---

PostgreSQL, one of the most powerful open-source relational databases, provides extensive logging capabilities to help administrators monitor performance, debug issues, and ensure database security.

However, many developers and database administrators (DBAs) overlook the full potential of Postgres logs, leading to inefficient troubleshooting and missed optimization opportunities.

This guide covers everything you need to know about Postgres logs—from configuration to advanced logging techniques—helping you maximize their value and stay ahead of the competition.

## Why PostgreSQL Logging Matters

PostgreSQL logs serve as a vital resource for:

- **Debugging performance issues**: Identify slow queries, deadlocks, and locking issues.
- **Security auditing**: Monitor unauthorized access attempts and suspicious activity.
- **Query optimization**: Analyze execution times and optimize queries accordingly.
- **System monitoring**: Detect errors, crashes, or anomalies before they become major issues.

## Types of PostgreSQL Logs

PostgreSQL provides various types of logs to help monitor database activity, diagnose issues, and optimize performance.

These logs capture different aspects of database operations, including errors, slow queries, connections, and more. Below are the main types of PostgreSQL logs:

### 1. Error Logs

Error logs record messages related to database errors, warnings, and notices. These logs help administrators identify issues such as failed queries, permission errors, and syntax mistakes.

The logging level can be configured using parameters like `log_min_error_statement` to filter logs by severity (e.g., `ERROR`, `FATAL`, `PANIC`).

### 2. Query Logs

Query logs track SQL statements executed by users. Enabling `log_statement` allows the logging of `ALL`, `MOD`, `DDL`, or `NONE` queries. Additionally, `log_min_duration_statement` can be set to log queries that take longer than a specified duration, helping in performance tuning.

### 3. Connection and Authentication Logs

These logs capture details about user connections, disconnections, and authentication failures. Parameters like `log_connections`, `log_disconnections`, and `log_authentication` provide insights into user activity and potential security threats.

### 4. Slow Query Logs

Slow query logs record queries that exceed a specified execution time. This helps database administrators identify bottlenecks and optimize indexing and query execution plans. The `log_min_duration_statement` the parameter controls this logging.

### 5. Autovacuum Logs

PostgreSQL’s auto vacuum process helps maintain database health by reclaiming storage and updating statistics. Enabling `log_autovacuum_min_duration` logs auto vacuum activities that exceed a defined duration, aiding in performance monitoring.

💡

For insights on using Pino Pretty for log formatting, check out our guide on [Pino Pretty](https://last9.io/blog/pino-pretty/).

### 6. Checkpoint Logs

Checkpoint logs provide information on database checkpoints, which ensure data consistency. These logs help in understanding database write activity and tuning checkpoint-related settings like `log_checkpoints`.

### 7. Lock and Deadlock Logs

PostgreSQL can log lock waits and deadlocks, helping to diagnose transaction contention issues. Enabling `log_lock_waits` records queries waiting on locks, while deadlocks are automatically logged when detected.

### 8. Replication Logs

Replication logs capture details related to streaming replication, WAL (Write-Ahead Logging), and standby server synchronization. Parameters like `log_replication_commands` help monitor replication-related activities.

### 9. Temporary File Logs

PostgreSQL logs temporary file usage when queries exceed available memory and spill to disk. Setting `log_temp_files` allows monitoring of temporary file creation, which helps in optimizing memory settings.

Each of these log types serves a specific purpose, providing insights into different aspects of database performance, security, and maintenance.

## Logging Levels and Severity in PostgreSQL

PostgreSQL provides multiple logging levels that control the verbosity and detail of log outputs.

These levels help administrators filter logs based on the severity of messages, making it easier to diagnose issues, monitor activity, and optimize performance.

The logging level is primarily controlled by the `log_min_messages` and `log_min_error_statement` parameters.

### Logging Levels in PostgreSQL

PostgreSQL categorizes log messages into the following severity levels, listed from highest to lowest importance:

#### **PANIC**

The database system encountered a critical error and must shut down immediately.

#### **FATAL**

A serious issue occurred that prevents the session from continuing, such as an authentication failure or data corruption.

#### **ERROR**

A query or command failed due to a syntax issue, constraint violation, or other operational error.

#### **WARNING**

A potential issue was detected, but the query or operation was still completed. Examples include deprecated functions or near-full disk space.

#### **NOTICE**

Informational messages about database operations, such as implicit index creation or table modifications.

#### **INFO**

General system information messages are often useful for monitoring but not indicating issues.

#### **DEBUG**

Detailed internal messages used for debugging, available on multiple levels (`DEBUG1` to `DEBUG5`), with increasing verbosity.

💡

Learn more about syslog levels and their importance in log management by reading our guide on [what are syslog levels](https://last9.io/blog/what-are-syslog-levels/).

### Configuring Logging Levels

- **log_min_messages**: Defines the minimum severity level for messages to be logged. For example, the setting `log_min_messages = WARNING` will log `WARNING`, `ERROR`, `FATAL`, and `PANIC` messages but ignore `NOTICE`, `INFO`, and `DEBUG`.
- **log_min_error_statement**: Specifies the minimum severity level at which the full SQL statement causing the error is logged.

## Log Formatting and Structure in PostgreSQL

Proper log formatting and structure can make it easier to parse logs, integrate with external monitoring tools, and analyze specific database activities.

This section covers how to customize PostgreSQL log formats, including options for plain text, JSON, and CSV formatting.

### Basic Log Format Configuration

PostgreSQL logs are formatted using a combination of configuration parameters that define the output structure. These parameters are set in the `postgresql.conf` file, allowing fine-tuned control over log formatting.

#### **log_line_prefix**

This parameter allows you to define a custom prefix for each log line. The prefix can include placeholders that represent dynamic values, such as the timestamp, session ID, database name, or query text. Some commonly used placeholders are:

- **%t**: Timestamp of the log entry
- **%d**: Database name
- **%u**: User name
- **%r**: Remote host name or IP address
- **%p**: Process ID
- **%c**: Session ID
- **%x**: Transaction ID

**Example configuration:**

```
log_line_prefix = '%t [%p]: [%l-1] user=%u, db=%d, remote=%r '
```

This would produce log entries like:

```
2025-02-06 10:10:10.123 [12345]: [1-1] user=postgres, db=mydb, remote=192.168.1.100
```

#### **log_statement**

This parameter controls the level of SQL query logging. It can log `NONE`, `DDL` (Data Definition Language), `MOD` (Data Modification), or `ALL` (all SQL statements). Combined with `log_min_duration_statement`, it allows you to capture long-running queries as well.

### Structured Log Formats: JSON and CSV

PostgreSQL supports structured logging in both JSON and CSV formats. These formats are useful for integrating PostgreSQL logs with external log management systems, monitoring platforms, or data analysis tools.

#### **JSON Format**

The JSON log format provides a structured, machine-readable output that can be easily processed by tools like ELK Stack (Elasticsearch, Logstash, Kibana) or other log aggregators. To enable JSON logging, you can set the following parameters:

- **log_destination**: Set this to `json` to log in JSON format.
- **log_line_prefix**: Customizes the structure of each log entry.

**Example configuration for JSON:**

```
log_destination = 'json'
log_line_prefix = '{"time": "%t", "user": "%u", "db": "%d", "query": "%q", "duration": "%d"}'
```

**Sample output:**

```
{"time": "2025-02-06T10:10:10.123Z", "user": "postgres", "db": "mydb", "query": "SELECT * FROM users", "duration": 123}
```

This format is ideal for automated systems that need to parse logs efficiently and can help streamline the analysis process by providing structured data.

#### **CSV Format**

PostgreSQL also supports CSV logging for simple, tabular output. This format is useful for exporting log data to spreadsheets or other tools that require CSV input. To enable CSV logging, set the following parameters:

- **log_destination**: Set this to `csvlog` for CSV-formatted logs.

**Example configuration for CSV:**

```
log_destination = 'csvlog'
log_line_prefix = 'time,user,db,query,duration'
```

**Sample output:**

```
"2025-02-06T10:10:10.123Z", "postgres", "mydb", "SELECT * FROM users", 123
```

### Customizing Log Output

In addition to predefined formats, PostgreSQL allows fine-grained control over log content through additional parameters:

- **log_error_verbosity**: Controls the amount of detail provided in error messages. You can choose from `TERSE`, `DEFAULT`, or `VERBOSE` levels.
- **log_duration**: Logs the duration of each query execution. This can be helpful for performance analysis.
- **log_temp_files**: Logs the creation of temporary files during query execution. This helps identify queries that may require additional memory resources.

### Log Rotation and File Naming

Proper log rotation is essential for maintaining log files. PostgreSQL can automatically rotate logs based on size or time intervals. The following parameters control log rotation:

- **log_rotation_age**: Defines the maximum age of a log file before it is rotated.
- **log_rotation_size**: Specifies the maximum size of a log file before it is rotated.

## PostgreSQL Log Configuration

PostgreSQL provides several parameters that allow administrators to customize logging behavior, define log file locations, and control the level of detail captured in logs.

This section will cover key configuration settings for PostgreSQL logs, including. `log_directory`, `log_destination`, and enabling the `logging_collector`.

### log_directory

The `log_directory` parameter specifies the directory where PostgreSQL will store its log files.

By default, PostgreSQL stores logs in the `pg_log` subdirectory within the data directory. However, you can configure it to store logs in any location that has the necessary permissions.

**Configuration Example:**

```
log_directory = '/var/log/postgresql'
```

Make sure the directory exists and that PostgreSQL has appropriate permissions to write logs. Use an absolute path for the `log_directory` to avoid confusion and potential issues with relative paths.

### log_destination

The `log_destination` parameter determines where the log messages will be sent. PostgreSQL supports multiple log destinations, and you can specify one or more of them. The most commonly used destinations are `stderr`, `syslog`, `csvlog`, and `jsonlog`. You can even combine multiple destinations for more flexibility.

**Configuration Options:**

- **stderr**: Logs are written to the standard error output. This is the default setting for most PostgreSQL installations.
- **syslog**: Logs are sent to the system’s syslog service. This is useful for centralized logging.
- **csvlog**: Logs are written in CSV format, which can be easily processed by other tools.
- **jsonlog**: Logs are written in JSON format for structured data processing.

**Configuration Example:**

```
log_destination = 'stderr, csvlog'
```

This configuration writes logs both to the standard error output and in CSV format.

### logging_collector

The `logging_collector` parameter controls whether PostgreSQL collects log entries into log files or simply writes them to the standard output (`stderr`).

If enabled, PostgreSQL will capture all log messages and write them to files in the location defined by the `log_directory` parameter.

By default, the `logging_collector` is set to `off`, meaning PostgreSQL logs messages to `stderr`. However, enabling the collector is crucial for capturing and managing log files.

**Configuration Example:**

```
logging_collector = on
```

When enabled, you can specify additional parameters such as the log file name pattern and log file rotation behavior (e.g., based on size or time).

### log_filename

The `log_filename` parameter defines the name pattern for the log files. You can use placeholders to create dynamic log file names based on the date, time, or process ID.

**Common Placeholders:**

- **%Y**: Four-digit year (e.g., 2025)
- **%m**: Two-digit month (e.g., 02)
- **%d**: Two-digit day of the month (e.g., 06)
- **%H**: Two-digit hour (e.g., 10)
- **%M**: Two-digit minute (e.g., 30)
- **%P**: Process ID (e.g., 12345)

**Configuration Example:**

```
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
```

This will create log files with names like `postgresql-2025-02-06_103000.log`, making it easy to track logs by date and time.

### log_rotation_age and log_rotation_size

To prevent log files from growing too large, you can configure log rotation. The `log_rotation_age` and `log_rotation_size` parameters determine when log files are rotated.

- **log_rotation_age**: Specifies the maximum time interval before a new log file is created. It can be set in seconds, minutes, hours, or days.
- **log_rotation_size**: Defines the maximum size of a log file before rotation occurs. You can set this in kilobytes (KB), megabytes (MB), or gigabytes (GB).

**Configuration Examples:**

```
log_rotation_age = '1d'    # Rotate logs daily
log_rotation_size = '10MB' # Rotate logs when they reach 10MB
```

### log_min_messages

The `log_min_messages` parameter determines the minimum severity level for messages to be logged. This controls the verbosity of the logs. You can set this to various severity levels, such as `DEBUG`, `INFO`, `WARNING`, `ERROR`, and `PANIC`. The default is usually `WARNING`, but you can adjust this based on your needs.

**Configuration Example:**

```
log_min_messages = 'ERROR'
```

This configuration logs only errors, warnings, and critical messages, ignoring lower severity messages like notices and debug information.

### log_min_error_statement

The `log_min_error_statement` parameter specifies the minimum severity level of error messages for which the SQL statement will be logged. This is particularly useful when you want to log the actual SQL query that led to an error.

**Configuration Example:**

```
log_min_error_statement = 'ERROR'
```

This will log the SQL query whenever an error of severity `ERROR` or higher occurs.

### log_statement

The `log_statement` parameter controls the logging of SQL queries. You can set it to log `ALL` queries, `MOD` queries (modifications), `DDL` queries (data definition), or `NONE` (no queries). This is useful for tracking all database activities or narrowing down the logs to specific operations.

**Configuration Example:**

```
log_statement = 'MOD'   # Log all data modification queries (INSERT, UPDATE, DELETE)
```

### Summary of Key Log Configuration Parameters

|Parameter|Description|Example Value|
|---|---|---|
|**log_directory**|Defines where log files are stored.|`/var/log/postgresql`|
|**log_destination**|Specifies where logs are sent (stderr, syslog, etc.).|`stderr, csvlog`|
|**logging_collector**|Enables or disables log file collection.|`on`|
|**log_filename**|Defines the naming pattern for log files.|`postgresql-%Y-%m-%d.log`|
|**log_rotation_age**|Sets the rotation interval for logs.|`'1d'` (daily rotation)|
|**log_rotation_size**|Specifies the maximum size before log rotation.|`10MB`|
|**log_min_messages**|Sets the minimum message severity to log.|`'ERROR'`|
|**log_min_error_statement**|Controls the logging of SQL queries for error messages.|`'ERROR'`|
|**log_statement**|Defines the types of SQL queries to log.|`'MOD'` (modifications only)|

## How to Enable PostgreSQL Logging

To enable logging, modify your `postgresql.conf` file. The key settings include:

```
log_destination = 'stderr'  # Options: stderr, csvlog, syslog, eventlog (Windows)
logging_collector = on  # Required for stderr logs
log_directory = 'pg_log'  # Directory where logs will be stored
log_filename = 'postgresql-%Y-%m-%d.log'  # Naming pattern for log files
log_rotation_age = 1d  # Rotate logs daily
log_rotation_size = 10MB  # Rotate logs when they reach 10MB
```

Restart PostgreSQL to apply these changes:

```
sudo systemctl restart postgresql
```

### How to Choose the Right Log Destination

PostgreSQL supports multiple logging destinations. The most commonly used ones are:

- **stderr** (default): Suitable for most setups.
- **csvlog**: Useful for structured logs, making analysis easier.
- **syslog**: Ideal for integration with centralized logging systems.
- **eventlog** (Windows): Logs to the Windows Event Log.

## What are the Essential Logging Parameters

### 1. **Statement and Query Logging**

Enable logging of all executed queries to track database activity:

```
log_statement = 'all'  # Options: none, ddl, mod, all
log_min_duration_statement = 500  # Logs queries taking longer than 500ms
```

### 2. **Error Logging**

To capture critical errors, use:

```
log_min_messages = warning  # Options: debug5 - fatal
log_error_verbosity = verbose  # More details on errors
```

### 3. **Lock and Deadlock Logging**

For detecting locking issues:

```
deadlock_timeout = '2s'  # Logs when locks exceed 2 seconds
log_lock_waits = on  # Logs transactions waiting on locks
```

### 4. **Connection and Authentication Logging**

Monitor connection activity:

```
log_connections = on  # Logs each connection
log_disconnections = on  # Logs when users disconnect
log_hostname = on  # Logs client hostnames
```

💡

For a deeper dive into using Zap Logger for structured logging, check out our article on [Zap Logger](https://last9.io/blog/zap-logger/).

## Advanced Logging Techniques

### Log Parsing and Analysis

Using pgBadger, a powerful log analyzer, you can visualize logs with:

```
pgbadger -p "%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h" /var/log/postgresql/postgresql.log -o report.html
```

### JSON Logging for Structured Analysis

For better integration with logging systems like **Last9**:

```
log_destination = 'jsonlog'
```

This enables structured JSON logs, making analysis easier.

### Using Log Triggers for Real-Time Monitoring

You can set up triggers that log specific database actions:

```
CREATE OR REPLACE FUNCTION log_table_changes() RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, action, changed_by, changed_at)
    VALUES (TG_TABLE_NAME, TG_OP, current_user, NOW());
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
```

Attach this to a table:

```
CREATE TRIGGER users_log_trigger
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW EXECUTE FUNCTION log_table_changes();
```

## How to Integrate PostgreSQL Logs with External Tools

### 1. Centralized Logging

Forward PostgreSQL logs to Last9 for centralized log collection and analysis. You can use syslog or directly forward logs to Last9:

```
echo "local0.* @last9-server:514" >> /etc/rsyslog.conf
sudo systemctl restart rsyslog
```

Logs will be available in Last9 for search, alerting, and analysis.

### 2. Alerting with Last9

Last9 automatically collects PostgreSQL metrics and logs for alerting. Set up custom alerts for critical events like slow queries, errors, and performance degradation. Get notified via your preferred channels in real time.

### 3. Log Rotation & Retention Policies

Set up automatic log deletion for PostgreSQL logs older than 30 days to manage storage:

```
find /var/log/postgresql -name "*.log" -mtime +30 -delete
```

Use cron to schedule regular log cleanup.

### 4. Monitoring with Last9

Integrate PostgreSQL logs with Last9 for comprehensive, real-time monitoring and performance insights. Last9 offers advanced logging, query analysis, and system health monitoring, all in one platform.

[![Last9’s Single Pane for High Cardinality Observability](https://last9.ghost.io/content/images/2025/02/last9-telemetry-correlated.webp)](https://last9.io/blog/single-pane-observability-logs-traces-metrics/)

Last9’s Single Pane for High Cardinality Observability

## 3 Common PostgreSQL Logging Issues & Solutions

### 1. **Log Files Growing Too Large**

**Solution:** Adjust `log_rotation_size` and `log_rotation_age`.

### 2. **Too Much Noise in Logs**

**Solution:** Use `log_min_duration_statement` to filter slow queries only.

### 3. **Logs Not Capturing Enough Details**

**Solution:** Set `log_error_verbosity = verbose` for detailed logs.

## Best Practices for PostgreSQL Logging

1. **Use Log-Parsing Tools**  
    Use tools like pgBadger or Last9 for log parsing and performance insights.
2. **Enable Slow Query Logging**  
    Set `log_min_duration_statement` for slow query logging. Last9 automatically tracks and visualizes slow queries.
3. **Rotate Logs Regularly**  
    Configure log rotation with `log_rotation_age` and `log_rotation_size`. Last9 handles log retention and cleanup.
4. **Integrate with Monitoring Systems**  
    Integrate PostgreSQL logs with Last9 for centralized monitoring, performance insights, and alerting.
5. **Secure Log Files**  
    Restrict access to logs with proper permissions. Last9 secures and centralizes log data for monitoring.

## Conclusion

PostgreSQL logging is a crucial aspect of database administration that can significantly improve performance monitoring, troubleshooting, and security. Implement the techniques discussed in this guide to maximize the benefits of PostgreSQL logs and maintain a robust database environment.

💡

And if you'd like to continue the conversation, [our community on Discord is always open](https://discord.com/invite/W8gMppQC4b). We have a dedicated channel where you can connect with other developers and discuss your specific use case.