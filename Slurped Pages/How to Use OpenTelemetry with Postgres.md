---
link: https://last9.io/blog/how-to-use-opentelemetry-with-postgres/
byline: Prathamesh Sonpatki
site: Last9
date: 2025-04-11T15:30
excerpt: Learn how to set up OpenTelemetry with Postgres to trace queries,
  monitor performance, and get better visibility into your database activity.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T18:26
title: How to Use OpenTelemetry with Postgres
---

Effective database monitoring is essential for maintaining application performance and reliability. This guide explores how to implement OpenTelemetry with PostgreSQL, providing DevOps engineers and Site Reliability Engineers with practical steps for comprehensive database observability.

## Benefits of PostgreSQL Monitoring

OpenTelemetry (OTel) is an open-source observability framework that collects metrics, logs, and traces from applications and infrastructure. When used with PostgreSQL, it offers several advantages:

- Vendor-neutral data collection that works with multiple monitoring backends
- Unified collection of metrics, logs, and traces
- Standardized approach to monitoring across your entire infrastructure
- Detailed visibility into database performance without heavy instrumentation

💡

To make sense of the data OpenTelemetry collects, you'll need a backend—this guide on [Otel backends](https://last9.io/blog/opentelemetry-backends/) explains your options.

## Installing and Configuring OpenTelemetry for PostgreSQL

Setting up OpenTelemetry with PostgreSQL involves several straightforward steps:

### 1. Installing the OpenTelemetry Collector on Linux Systems

```
# Download the OpenTelemetry Collector
curl -L -o otelcol.tar.gz https://github.com/open-telemetry/opentelemetry-collector-releases/releases/download/v0.83.0/otelcol_0.83.0_linux_amd64.tar.gz

# Extract it
tar xzf otelcol.tar.gz

# Move to a suitable location
sudo mv otelcol /usr/local/bin/
```

This code downloads the OpenTelemetry Collector binary for Linux, extracts it from the compressed archive, and moves it to a system directory where executables are typically stored. This makes the collector available system-wide for any user to run.

### 2. Creating a PostgreSQL-Specific Collector Configuration File

Create a configuration file at `/etc/otel-collector-config.yaml`:

```
receivers:
  postgresql:
    endpoint: localhost:5432
    transport: tcp
    username: postgres_exporter
    password: your_password
    databases:
      - your_database_name
    collection_interval: 10s

processors:
  batch:
    timeout: 10s

exporters:
  prometheus:
    endpoint: 0.0.0.0:8889
  
  otlp:
    endpoint: your-backend:4317
    tls:
      insecure: true

service:
  pipelines:
    metrics:
      receivers: [postgresql]
      processors: [batch]
      exporters: [prometheus, otlp]
```

This configuration file tells the OpenTelemetry Collector how to collect metrics from your Postgres database and where to send them. It defines:

- A PostgreSQL receiver that connects to your database every 10 seconds
- A batch processor that groups metrics for efficient transmission
- Two exporters: one for Prometheus (for local viewing) and one for OTLP (to send to an observability platform)
- A pipeline that ties these components together

💡

If you're wondering how OpenTelemetry fits in with APM tools, this post on [Otel and APM](https://last9.io/blog/opentelemetry-and-apm/) breaks it down clearly.

### 3. Preparing PostgreSQL Database for Secure Monitoring Access

Create a dedicated read-only user for monitoring purposes:

```
CREATE USER postgres_exporter WITH PASSWORD 'your_password';
GRANT pg_monitor TO postgres_exporter;
```

This SQL creates a dedicated database user called 'postgres_exporter' with the password you specify. The second line grants this user the 'pg_monitor' role, which provides read-only access to PostgreSQL's monitoring views and functions without giving broader permissions that could affect database operation or security.

### 4. Running the OpenTelemetry Collector as a Persistent Service

Run the OpenTelemetry Collector:

```
otelcol --config=/etc/otel-collector-config.yaml
```

This command starts the OpenTelemetry Collector using the configuration file we created earlier. The collector will run in the foreground, so you'll see logs directly in your terminal. In a production environment, you'd typically run this as a system service to ensure it starts automatically and runs in the background.

For production environments, create a systemd service file at `/etc/systemd/system/otel-collector.service`:

```
[Unit]
Description=OpenTelemetry Collector
After=network.target

[Service]
ExecStart=/usr/local/bin/otelcol --config=/etc/otel-collector-config.yaml
Restart=always
User=otel
Group=otel

[Install]
WantedBy=multi-user.target
```

Then enable and start the service:

```
sudo systemctl enable otel-collector
sudo systemctl start otel-collector
```

💡

This post on [OpenTelemetry agents](https://last9.io/blog/opentelemetry-agents/) explains how they collect telemetry data and where they fit in your setup.

## PostgreSQL Performance Metrics for Effective Monitoring

When monitoring PostgreSQL with OpenTelemetry, focus on these essential metrics:

|Metric Category|Key Metrics|Purpose|
|---|---|---|
|Connection Stats|active_connections, max_connections, wait_event_type|Identify connection pool issues and saturation|
|Query Performance|slow_queries, query_time, rows_fetched|Find performance bottlenecks and inefficient queries|
|Cache Efficiency|cache_hit_ratio, buffer_usage, shared_buffers_used|Assess memory usage efficiency|
|WAL/Replication|replication_lag, WAL_generation_rate|Monitor data durability and replication health|
|Resource Usage|disk_usage, index_size, table_size|Track storage growth and capacity planning|

These metrics provide a comprehensive view of your database's health and performance, allowing you to identify issues before they impact users.

## Resolving Common OpenTelemetry PostgreSQL Issues

### Diagnosing and Fixing PostgreSQL Connection Failures

If the collector fails to connect to PostgreSQL with errors like:

```
Error: could not connect to server: Connection refused
```

Verify the following:

1. PostgreSQL service status: `systemctl status postgresql`
2. Network connectivity: `telnet localhost 5432`
3. Credentials in the collector configuration
4. PostgreSQL's `pg_hba.conf` configuration to ensure it allows connections from the collector host
5. Firewall rules if the collector and database are on different hosts

### Resolving Missing or Incomplete PostgreSQL Metrics Collection

If certain metrics are absent or incomplete:

1. Verify the monitoring user has sufficient permissions
2. Check for collection interval issues if metrics appear intermittently

Increase the collector's logging level for more details:

```
otelcol --config=/etc/otel-collector-config.yaml --log-level=debug
```

Test direct access by running queries as the monitoring user:

```
psql -U postgres_exporter -h localhost -d your_database -c "SELECT * FROM pg_stat_database LIMIT 1;"
```

💡

Fix production Postgres observability issues instantly—right from your IDE, with AI and [Last9 MCP](https://last9.io/mcp/).

### Minimizing Monitoring Overhead on High-Traffic PostgreSQL Databases

To minimize the monitoring impact on busy production databases:

1. Increase the collection interval (e.g., from 10s to 30s or 60s)
2. Use connection pooling for the monitoring user
3. Schedule intensive metric collection during off-peak hours
4. Monitor the collector's resource usage

## How to Create PostgreSQL Dashboards with Collected Metrics

Collected metrics are most useful when properly visualized. Here's how to set up visualization:

### Building PostgreSQL Monitoring Dashboards with Grafana

1. Add your data source:
    - For Prometheus: Configure a Prometheus data source pointing to the collector's Prometheus endpoint
    - For other backends: Configure the appropriate data source for your observability platform
2. Import a PostgreSQL dashboard:
    - Grafana dashboard ID 9628 provides a good starting point
    - Customize as needed for your specific environment
3. Create alerts for critical metrics:
    - Connection saturation (>80% of max_connections)
    - Replication lag beyond acceptable thresholds
    - Low cache hit ratios
    - Unusual query patterns

Install Grafana if not already present:

```
sudo apt-get install -y grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

## Advanced OpenTelemetry Techniques for PostgreSQL Insights

### Custom Business Metrics with PostgreSQL Functions

Extend standard PostgreSQL metrics with business-specific metrics:

```
CREATE OR REPLACE FUNCTION get_custom_metrics() RETURNS TABLE(metric_name text, metric_value numeric) AS $$
BEGIN
  RETURN QUERY SELECT 
    'active_premium_users' as metric_name, 
    COUNT(*) as metric_value 
  FROM users 
  WHERE subscription_type = 'premium' AND last_active > NOW() - INTERVAL '24 hours';
END;
$$ LANGUAGE plpgsql;

GRANT EXECUTE ON FUNCTION get_custom_metrics() TO postgres_exporter;
```

This code creates a custom SQL function that calculates business metrics from your database - in this example, counting active premium users in the last 24 hours.

The function returns results in a format that OpenTelemetry can collect as metrics. The GRANT statement gives our monitoring user permission to run this function. You can create multiple such functions for different business metrics you want to track alongside technical database metrics.

To collect these custom metrics, add the following to your collector configuration:

```
receivers:
  postgresql:
    # ... existing config ...
    queries:
      - sql: "SELECT * FROM get_custom_metrics()"
        metrics:
          - metric_name: business_metrics
            value_column: metric_value
            attribute_columns: [metric_name]
```

### Connecting Application Requests to Database Performance

To gain deeper insights, correlate database performance with application behavior:

1. Propagate trace context to identify which application requests cause database load
2. Add database-specific attributes to spans for a more detailed analysis

Instrument your application with OpenTelemetry:

```
// Example in Java with JDBC
try (Connection connection = dataSource.getConnection()) {
  Span span = tracer.spanBuilder("DB Query").startSpan();
  try (Scope scope = span.makeCurrent()) {
    // Execute your query
    PreparedStatement stmt = connection.prepareStatement("SELECT * FROM users WHERE id = ?");
    stmt.setInt(1, userId);
    ResultSet rs = stmt.executeQuery();
    // Process results
  } finally {
    span.end();
  }
}
```

This correlation helps identify whether performance issues originate in your application code, database queries, or database configuration.

## Conclusion

Implementing OpenTelemetry with PostgreSQL provides comprehensive visibility into your database's performance and health.

The combination of standardized metrics collection, customizable dashboards, and correlation between application and database performance makes OpenTelemetry an excellent choice for modern PostgreSQL monitoring in DevOps and SRE environments.

💡

For further discussion on database observability strategies or to share your experiences with OpenTelemetry and PostgreSQL, join our [Discord community](https://discord.com/invite/W8gMppQC4b).

## FAQs

### What advantages does OpenTelemetry offer over native PostgreSQL monitoring tools?

OpenTelemetry provides several advantages over native monitoring solutions:

1. **Vendor neutrality:** Collect data once and send it to multiple backends (Prometheus, Last9, etc.)
2. **Unified observability:** Collect metrics, traces, and logs through a single framework
3. **Cross-service correlation:** Connect database performance to application behavior
4. **Standardization:** Use consistent monitoring approaches across your entire infrastructure
5. **Extensibility:** Add custom metrics and tracking without proprietary extensions

While tools like pg_stat_statements provide valuable insights, OpenTelemetry offers a more comprehensive monitoring solution that integrates with your broader observability strategy.

### Will OpenTelemetry monitoring impact my PostgreSQL performance?

When properly configured, OpenTelemetry should have minimal performance impact on your PostgreSQL databases. To ensure low overhead:

1. Use appropriate collection intervals (30-60 seconds for most metrics)
2. Limit collection of expensive metrics to off-peak hours
3. Implement connection pooling for monitoring connections
4. Avoid frequent collection of metrics requiring heavy table scans
5. Use the pg_monitor role rather than superuser privileges

Our testing shows that properly configured OpenTelemetry collection typically adds less than 1% CPU overhead to a moderately busy PostgreSQL database.

### How does OpenTelemetry handle PostgreSQL major version upgrades?

OpenTelemetry adapts well to PostgreSQL version changes for several reasons:

1. It relies on PostgreSQL's statistics views, which maintain reasonable backward compatibility
2. The collector can be updated independently of the database
3. The modular architecture allows for version-specific configurations

When upgrading PostgreSQL versions:

- Update your OpenTelemetry Collector to the latest version
- Review metric availability in the new PostgreSQL version
- Test the collection with the new version before production deployment
- Update dashboards to include any new metrics available in the newer version

### Can OpenTelemetry monitor PostgreSQL replication lag and failovers?

Yes, OpenTelemetry effectively monitors replication and failover scenarios:

1. **Replication lag:** The collector can track `pg_stat_replication.replay_lag` to monitor how far replicas lag behind the primary
2. **Promotion events:** Monitoring cluster state changes through pg_is_in_recovery()
3. **WAL generation rate:** Track write-ahead log generation to predict potential replication issues

For comprehensive replication monitoring, configure collectors on both primary and replica servers, and set up alerts for replication lag exceeding your recovery point objective (RPO).

### How do I correlate PostgreSQL metrics with application performance problems?

Effective correlation requires implementing both database and application instrumentation:

1. **Implement application tracing:** Add OpenTelemetry instrumentation to your application code
2. **Add database attributes:** Include query identifiers and database operations in application spans
3. **Use trace context propagation:** Pass trace IDs through your application to database calls
4. **Create correlation dashboards:** Build dashboards showing both application and database metrics aligned by time

This approach allows you to identify whether a performance issue originates in your application code, database queries, or database configuration.

### What are the most critical PostgreSQL metrics to alert on?

While monitoring needs vary by workload, these metrics generally warrant alerting:

1. **Connection saturation:** Alert when connections exceed 80% of max_connections
2. **Replication lag:** Alert when lag exceeds your recovery point objective
3. **Transaction wraparound:** Alert when approaching transaction ID wraparound
4. **Disk space:** Alert at 80% and 90% disk usage thresholds
5. **Cache hit ratio:** Alert on sudden drops in cache efficiency
6. **Long-running queries:** Alert on queries exceeding expected durations
7. **Error rates:** Alert on unusual increases in database errors or deadlocks

Fine-tune these thresholds based on your specific workload patterns and performance requirements.

### Can OpenTelemetry replace database query monitoring tools like pgBadger?

OpenTelemetry complements rather than replaces specialized query analysis tools:

- **OpenTelemetry:** Best for ongoing metrics collection, alerting, and correlation with other services
- **pgBadger/pg_stat_statements:** Better for detailed query analysis and optimization

For comprehensive monitoring:

1. Use OpenTelemetry for continuous metric collection and alerting
2. Use pgBadger for periodic in-depth query analysis
3. Use pg_stat_statements for identifying high-impact queries

This combined approach provides both real-time monitoring and detailed query optimization capabilities.