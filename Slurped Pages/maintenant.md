---
link: https://docs.maintenant.dev/
excerpt: The all-in-one monitoring dashboard your self-hosted stack deserves.
slurped: 2026-03-21T12:01
title: maintenant
---

**The all-in-one monitoring dashboard your self-hosted stack deserves.**

Drop a single container. Watch everything. Sleep at night.

---

## What is maintenant?[¶](https://docs.maintenant.dev/#what-is-maintenant "Permanent link")

Most self-hosters juggle 3-5 tools to monitor their stack: one for containers, one for uptime, one for certs, one for metrics, and yet another for a status page. maintenant replaces all of them with a single binary, zero external dependencies, and zero configuration to get started.

Deploy one container, and maintenant auto-discovers your entire stack. Docker or Kubernetes — it does not matter.

[Installation](https://docs.maintenant.dev/getting-started/installation/)
---

## Key Features[¶](https://docs.maintenant.dev/#key-features "Permanent link")

- **[Container Monitoring](https://docs.maintenant.dev/features/containers/)** — Zero-config auto-discovery for Docker and Kubernetes. State tracking, health checks, restart loop detection, log streaming.
- **[Endpoint Monitoring](https://docs.maintenant.dev/features/endpoints/)** — HTTP and TCP checks defined as Docker labels. Response times, uptime history, sparklines.
- **[Heartbeat & Cron Monitoring](https://docs.maintenant.dev/features/heartbeats/)** — Create a monitor, get a URL, curl from your cron job. Tracks durations, exit codes, missed deadlines.
- **[TLS Certificate Monitoring](https://docs.maintenant.dev/features/certificates/)** — Auto-detection from HTTPS endpoints. Alerts at 30, 14, 7, 3, and 1 day before expiry. Full chain validation.
- **[Resource Metrics](https://docs.maintenant.dev/features/resources/)** — CPU, memory, network I/O, disk I/O per container. Historical charts, alert thresholds, top consumers view.
- **[Update Intelligence](https://docs.maintenant.dev/features/updates/)** — OCI registry scanning, digest comparison. Compose-aware update commands. Know when your images have updates available.
- **[Network Security Insights](https://docs.maintenant.dev/features/security/)** — Automatic detection of exposed ports, dangerous network configurations, and privileged containers. CVE ecosystem mapping via OCI manifest inspection.
- **[Alert Engine](https://docs.maintenant.dev/features/alerts/)** — Unified alerts across all sources. Webhook and Discord channels. Silence rules, exponential backoff. Slack, Teams, and Email with Pro.
- **[Public Status Page](https://docs.maintenant.dev/features/status-page/)** — Component groups, live SSE updates. Incident management, maintenance windows, and subscriber notifications with Pro.
- **[MCP Server](https://docs.maintenant.dev/features/mcp/)** — Expose monitoring data to AI assistants (Claude Code, Cursor) via the Model Context Protocol. 18 tools, stdio and HTTP transports.

---

## Comparison[¶](https://docs.maintenant.dev/#comparison "Permanent link")

||maintenant|Uptime Kuma|Portainer|Dozzle|
|---|---|---|---|---|
|Container auto-discovery|**Yes**|No|Yes|Yes|
|HTTP/TCP endpoint checks|**Yes**|Yes|No|No|
|Cron/heartbeat monitoring|**Yes**|Yes|No|No|
|SSL certificate tracking|**Yes**|Yes|No|No|
|CPU/memory/network metrics|**Yes**|No|Limited|No|
|Image update detection|**Yes**|No|Yes|No|
|Network security insights|**Yes**|No|No|No|
|Public status page|**Yes**|Yes|No|No|
|Alerting (webhook, Discord)|**Yes**|Yes|Limited|No|
|Kubernetes native|**Yes**|No|Yes|No|
|MCP for AI assistants|**Yes**|No|No|No|
|Single binary, zero deps|**Yes**|Node.js|Docker API|Docker API|

---

## Quick Start[¶](https://docs.maintenant.dev/#quick-start "Permanent link")

Get maintenant running in 30 seconds:
```# docker-compose.yml
services:
  maintenant:
    image: ghcr.io/kolapsis/maintenant:latest
    ports:
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /proc:/host/proc:ro
      - maintenant-data:/data
    environment:
      MAINTENANT_ADDR: "0.0.0.0:8080"
      MAINTENANT_DB: "/data/maintenant.db"
    restart: unless-stopped

volumes:
  maintenant-data:

```


Open **http://localhost:8080** — your containers are already there. No configuration needed.

For detailed installation instructions, see [Installation](https://docs.maintenant.dev/getting-started/installation/).