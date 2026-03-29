---
link: https://last9.io/blog/monitoring-ubuntu-servers/
byline: Anjali Udasi
site: Last9
date: 2025-05-02T08:53
excerpt: Learn how to set up effective monitoring for your Ubuntu servers, from
  basic to advanced strategies, to keep your systems running smoothly.
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T15:23
title: A Practical Guide to Monitoring Ubuntu Servers
---

Running Ubuntu servers without proper monitoring can lead to unexpected issues. For DevOps engineers and SREs, effective tracking is crucial for maintaining system health and performance.

This guide covers everything you need to know about monitoring Ubuntu servers, from the basics to advanced strategies, helping you keep your systems running smoothly, whether you manage a single server or a large fleet.

## What Makes Ubuntu Server Monitoring Different?

Ubuntu servers have specific characteristics that influence how you approach monitoring. As a Debian-based distribution, Ubuntu has its own package management system, service management approach, and file organization that affect what and how you monitor.

The beauty of Ubuntu is its balance between stability and access to recent software versions. This makes it popular for production environments, but also means your monitoring setup needs to account for Ubuntu-specific considerations.

For instance, Ubuntu uses systemd, which provides its logging system (journald) alongside traditional syslog. Your monitoring strategy needs to handle both to get a complete picture of system health.

## Key Metrics You Should Track on Ubuntu Servers

Before jumping into tools and implementation, let's clarify what you need to monitor. Here are the essential metrics for any Ubuntu server:

### System Resources

- **CPU usage**: Track overall usage, user/system time split, and load averages
- **Memory utilization**: Physical memory usage, swap usage, and available memory
- **Disk usage**: Space utilization, inode usage, and read/write operations
- **Network traffic**: Bandwidth usage, connection counts, and packet loss

### System Health

- **Uptime**: How long your server has been running
- **Process counts**: Total processes and their states
- **Login sessions**: Who's logged in and from where
- **Cron jobs**: Are scheduled tasks running properly?

### Service-Specific Metrics

- **Web server metrics**: Request rates, response times, error rates
- **Database metrics**: Query performance, connection counts, buffer usage
- **Application metrics**: Custom metrics specific to your applications

### Log Monitoring

- **System logs**: Kernel messages, authentication attempts, system errors
- **Application logs**: Error logs, access logs, custom application logs

A good monitoring setup will cover all these areas, giving you both real-time visibility and historical data for troubleshooting and capacity planning.

Ubuntu comes with several built-in tools that are perfect for quick checks and basic monitoring:

### top and htop

These command-line tools give you a real-time view of system resources. `htop` offers a more user-friendly interface with color-coding and mouse support:

```
sudo apt install htop
htop
```

### iotop

For monitoring disk I/O by process:

```
sudo apt install iotop
sudo iotop
```

### netstat and ss

These tools help monitor network connections:

```
# List all listening TCP ports
netstat -tlpn
# Modern alternative with similar functionality
ss -tlpn
```

### df and du

For disk usage monitoring:

```
# View disk space usage
df -h
# Check directory sizes
du -sh /var/*
```

### journalctl

Access systemd journal logs:

```
# View all logs
journalctl
# Follow logs in real-time
journalctl -f
# View logs for a specific service
journalctl -u apache2.service
```

These tools are great for immediate troubleshooting but lack long-term data storage and alerting capabilities. That's where dedicated monitoring solutions like Last9 and more come in.

💡

If you're managing Ubuntu logs and want to understand the messages in the /var/log directory, check out our article on [Ubuntu /var/log messages](https://last9.io/blog/ubuntu-var-log-messages/) for practical insights.

## How to Set Up Prometheus for Ubuntu Server Monitoring

Prometheus has become a standard for monitoring infrastructure. Here's how to set it up on your Ubuntu server:

### Step 1: Install and Set Up the Prometheus Server

```
# Create a system user for Prometheus
sudo useradd --no-create-home --shell /bin/false prometheus

# Create directories for Prometheus
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus

# Download and install Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.39.1/prometheus-2.39.1.linux-amd64.tar.gz
tar xvfz prometheus-2.39.1.linux-amd64.tar.gz
sudo cp prometheus-2.39.1.linux-amd64/prometheus /usr/local/bin/
sudo cp prometheus-2.39.1.linux-amd64/promtool /usr/local/bin/
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool

# Copy configuration files
sudo cp -r prometheus-2.39.1.linux-amd64/consoles /etc/prometheus
sudo cp -r prometheus-2.39.1.linux-amd64/console_libraries /etc/prometheus
sudo cp prometheus-2.39.1.linux-amd64/prometheus.yml /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus
```

### Step 2: Configure Prometheus to Monitor Your Ubuntu Server

Edit the Prometheus configuration file:

```
sudo nano /etc/prometheus/prometheus.yml
```

Add your Ubuntu server as a target:

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'ubuntu_node'
    static_configs:
      - targets: ['localhost:9100']
```

### Step 3: Create a Systemd Service for Automatic Prometheus Startup

```
sudo nano /etc/systemd/system/prometheus.service
```

Add the following content:

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

Start the service:

```
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

### Step 4: Install Node Exporter to Collect Ubuntu System Metrics

Node Exporter collects system metrics from your Ubuntu server:

```
# Create a system user for Node Exporter
sudo useradd --no-create-home --shell /bin/false node_exporter

# Download and install Node Exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.4.0.linux-amd64.tar.gz
sudo cp node_exporter-1.4.0.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

Create a systemd service for Node Exporter:

```
sudo nano /etc/systemd/system/node_exporter.service
```

Add the following content:

```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Start the service:

```
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

Now Prometheus will collect metrics from Node Exporter. You can access the Prometheus web interface at http://your-server-ip:9090.

💡

If you're using Prometheus and want to extend it for application monitoring, check out our article on [Prometheus as an APM](https://last9.io/blog/prometheus-apm/).

## Visualizing Ubuntu Server Metrics with Grafana

While Prometheus is great for collecting and storing metrics, Grafana excels at visualization. Here's how to set it up:

### Step 1: Install Grafana for Advanced Metric Visualization

```
# Add the GPG key
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# Add the repository
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"

# Update and install
sudo apt update
sudo apt install grafana

# Start and enable Grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### Step 2: Connect Grafana to Your Prometheus Data Source

Access Grafana at http://your-server-ip:3000 (default credentials: admin/admin).

1. Go to Configuration > Data Sources
2. Click "Add data source"
3. Select "Prometheus"
4. Set the URL to http://localhost:9090
5. Click "Save & Test"

### Step 3: Import Pre-Built Dashboards for Ubuntu Server Monitoring

Grafana has many pre-built dashboards for Ubuntu server monitoring:

1. Go to Dashboard > Import
2. Enter ID 1860 (Node Exporter Full dashboard)
3. Select your Prometheus data source
4. Click "Import"

You now have a comprehensive monitoring dashboard for your Ubuntu server!

### Advanced Monitoring with Last9

While Prometheus and Grafana offer strong monitoring capabilities, managing complex environments requires a more streamlined approach. Our platform, [Last9](https://last9.io/), provides a comprehensive monitoring solution that integrates seamlessly with Ubuntu servers.

We combine metrics, logs, and traces into one platform, making it ideal for high-cardinality monitoring at scale. Our platform integrates with OpenTelemetry and Prometheus, offering unified visibility across your entire infrastructure.

We’ve handled some of the largest live-streaming events in history, proving our reliability under extreme conditions. If you're looking for enterprise-grade monitoring without the enterprise-grade price tag, our platform is worth checking out.

## How to Monitor Ubuntu Server Logs

Metrics tell you what's happening, but logs tell you why. Here's how to set up effective log monitoring:

### Set Up Filebeat and ELK Stack for Comprehensive Log Analysis

The ELK Stack (Elasticsearch, Logstash, Kibana) is perfect for log analysis. Filebeat helps collect and ship logs:

```
# Install Filebeat
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt-get update
sudo apt-get install filebeat

# Configure Filebeat
sudo nano /etc/filebeat/filebeat.yml
```

Basic Filebeat configuration:

```
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log
    - /var/log/syslog
    - /var/log/auth.log

output.elasticsearch:
  hosts: ["your-elasticsearch-server:9200"]
```

Start and enable Filebeat:

```
sudo systemctl start filebeat
sudo systemctl enable filebeat
```

### Integrate Loki with Grafana for Lightweight Log Management

If you're already using Grafana, Loki provides a lightweight log aggregation system:

```
# Install Promtail (log collector for Loki)
wget https://github.com/grafana/loki/releases/download/v2.7.0/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
sudo mv promtail-linux-amd64 /usr/local/bin/promtail

# Configure Promtail
sudo mkdir -p /etc/promtail
sudo nano /etc/promtail/config.yml
```

Basic Promtail configuration:

```
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://your-loki-server:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
    - targets:
        - localhost
      labels:
        job: varlogs
        __path__: /var/log/*log
```

Create a systemd service for Promtail:

```
sudo nano /etc/systemd/system/promtail.service
```

Add the following content:

```
[Unit]
Description=Promtail
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/promtail -config.file /etc/promtail/config.yml

[Install]
WantedBy=multi-user.target
```

Start and enable Promtail:

```
sudo systemctl daemon-reload
sudo systemctl start promtail
sudo systemctl enable promtail
```

💡

If you're troubleshooting system crashes on Ubuntu, our guide on [Ubuntu crash logs](https://last9.io/blog/ubuntu-crash-logs/) can help you figure out what went wrong.

## Step-by-Step Process to Set Up Alerts for Ubuntu Server Monitoring

Monitoring is useless if you don't get notified when things go wrong. Here's how to set up alerts:

### Configure Prometheus Alerting Rules for Critical System Events

Create alert rules in Prometheus:

```
sudo nano /etc/prometheus/alert.rules.yml
```

Add basic alert rules:

```
groups:
- name: ubuntu_alerts
  rules:
  - alert: HighCPULoad
    expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU load (instance {{ $labels.instance }})"
      description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
  
  - alert: DiskSpaceFilling
    expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 10
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Disk space filling up (instance {{ $labels.instance }})"
      description: "Disk is almost full (< 10% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

Update your Prometheus configuration to include these rules:

```
# In prometheus.yml
rule_files:
  - "alert.rules.yml"
```

### Set Up Alertmanager for Notification Delivery and Alert Management

Alertmanager handles notifications:

```
# Download and install Alertmanager
wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
tar xvfz alertmanager-0.24.0.linux-amd64.tar.gz
sudo cp alertmanager-0.24.0.linux-amd64/alertmanager /usr/local/bin/
sudo mkdir -p /etc/alertmanager
sudo cp alertmanager-0.24.0.linux-amd64/alertmanager.yml /etc/alertmanager/
```

Configure Alertmanager:

```
sudo nano /etc/alertmanager/alertmanager.yml
```

Add a basic configuration:

```
global:
  smtp_smarthost: 'smtp.example.org:587'
  smtp_from: 'alertmanager@example.org'
  smtp_auth_username: 'alertmanager'
  smtp_auth_password: 'password'

route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'email'

receivers:
- name: 'email'
  email_configs:
  - to: 'alerts@example.org'
```

Create a systemd service for Alertmanager:

```
sudo nano /etc/systemd/system/alertmanager.service
```

Add the following content:

```
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/alertmanager \
    --config.file=/etc/alertmanager/alertmanager.yml \
    --storage.path=/var/lib/alertmanager

[Install]
WantedBy=multi-user.target
```

Start and enable Alertmanager:

```
sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```

Update Prometheus to use Alertmanager:

```
# In prometheus.yml
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - localhost:9093
```

Restart Prometheus:

```
sudo systemctl restart prometheus
```

💡

Last9's Alert Studio is built to handle high-cardinality use cases, helping reduce alert fatigue and improve Mean Time to Detect. Check it out [here](https://last9.io/alerting/).

## Common Ubuntu Server Monitoring Issues and Solutions

Even the best monitoring setups can run into problems. Here are some common issues and how to fix them:

### High CPU Usage from Monitoring Agents

**Problem**: Your monitoring agents (like Node Exporter) are consuming too much CPU.

**Solution**: Adjust the scrape interval in Prometheus to reduce the frequency of metric collection:

```
global:
  scrape_interval: 30s  # Increase from 15s to 30s
```

### Storage Space Running Out

**Problem**: Prometheus is filling up your disk with time-series data.

**Solution**: Configure data retention and storage parameters:

```
# In prometheus.yml
storage:
  tsdb:
    retention.time: 15d
    retention.size: 5GB
```

### Missing Data Points

**Problem**: You notice gaps in your monitoring graphs.

**Solution**: This could be due to network issues or service restarts. Check the status of your monitoring services:

```
sudo systemctl status prometheus
sudo systemctl status node_exporter
```

### Too Many Alerts

**Problem**: You're getting flooded with alerts.

**Solution**: Refine your alert thresholds and grouping in Alertmanager:

```
route:
  group_by: ['alertname', 'instance']
  group_wait: 1m
  group_interval: 10m
  repeat_interval: 3h
```

💡

Now, fix production monitoring issues instantly—right from your IDE, with AI and [Last9 MCP](https://last9.io/mcp/). Bring real-time production context—logs, metrics, and traces—into your local environment to auto-fix code faster.

## Monitoring Ubuntu in Container Environments

Containerized Ubuntu environments need special consideration for effective monitoring:

### Monitor Docker Containers on Ubuntu with cAdvisor

Install cAdvisor to monitor container metrics:

```
sudo docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  gcr.io/cadvisor/cadvisor:v0.45.0
```

Then add cAdvisor as a target in Prometheus:

```
- job_name: 'cadvisor'
  static_configs:
    - targets: ['localhost:8080']
```

### Monitor Kubernetes on Ubuntu with Prometheus Operator

For Kubernetes environments, use the Prometheus Operator:

```
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/main/manifests/setup/0setupNamespace.yaml
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/main/manifests/setup/
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/main/manifests/
```

This deploys a complete monitoring stack including Prometheus, Alertmanager, and Grafana, optimized for Kubernetes.

Here's a quick comparison of popular monitoring tools for Ubuntu servers:

|Tool|Strengths|Weaknesses|Best For|
|---|---|---|---|
|Prometheus|Open-source, powerful query language, great for metrics|Steep learning curve, limited log support|Metric collection and alerting|
|Grafana|Beautiful visualizations, plugin ecosystem|Requires data sources, no built-in alerting|Dashboards and visualization|
|Last9|Unified metrics, logs, and traces; high-cardinality support|Paid service|Enterprise environments, Teams needing high-cardinality observability without the hefty price tag|
|Zabbix|All-in-one solution, agent-based|Complex setup, resource-heavy|Traditional infrastructure|
|Nagios|Mature, extensive plugin ecosystem|Outdated UI, limited scalability|Basic monitoring needs|

## Conclusion

Effective Ubuntu server monitoring is crucial for keeping systems reliable. By combining native tools with solutions like Prometheus, Grafana, and Last9, you can create a comprehensive monitoring strategy that provides both real-time insights and historical context.

💡

And if you'd like to discuss further, our [Discord community](https://discord.com/invite/W8gMppQC4b) is open. We have a dedicated channel where you can share your use case and connect with other developers.

## FAQs

### How often should I check my Ubuntu server metrics?

For production systems, you should set up continuous monitoring with alerts for critical issues. For less critical systems, daily checks might be sufficient.

### What's the minimum monitoring setup I need for a small Ubuntu server?

At minimum, monitor CPU, memory, disk usage, and essential services. Tools like `htop`, `df`, and simple cron job checks can provide basic monitoring without much overhead.

### How do I monitor multiple Ubuntu servers efficiently?

Use a centralized monitoring solution like Prometheus with Node Exporter on each server. Grafana can then visualize metrics from all servers in a single dashboard.

### Can I monitor Ubuntu servers without installing agents?

Yes, but with limitations. You can use SNMP or agentless monitoring solutions, but you'll get less detailed information compared to agent-based monitoring.

### How much disk space should I allocate for monitoring data?

For a single server with default Prometheus settings, allocate at least 10-20GB for a few weeks of metrics. Adjust based on the number of metrics and retention period.

### How do I monitor GPU usage on Ubuntu servers?

For NVIDIA GPUs, install the NVIDIA GPU Exporter and add it as a target in Prometheus:

```
docker run -d --gpus all --restart unless-stopped -p 9835:9835 --name nvidia_exporter nvcr.io/nvidia/k8s/dcgm-exporter:2.4.6-2.6.7-ubuntu20.04
```