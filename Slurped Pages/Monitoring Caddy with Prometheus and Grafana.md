---
link: https://thedevelopercafe.com/articles/monitoring-caddy-with-prometheus-and-grafana-2f1f869097d8
excerpt: In this article you are going to learn how to monitor Caddy by
  collecting metrics with Prometheus and visualizing with Grafana.
slurped: 2024-05-13T14:25:26.866Z
title: Monitoring Caddy with Prometheus and Grafana
---

In this article you are going to learn how to monitor [Caddy](https://caddyserver.com/) by collecting metrics with [Prometheus](https://prometheus.io/).

You are going to use [docker compose](https://docs.docker.com/compose/) to spin up all the required services.

![Caddy Prometheus Metrics Architecture Dialog](https://tdc-public.nyc3.digitaloceanspaces.com/photos/1c0ea101-de2b-4f15-b8b2-b82ce7198792-caddy-prometheus-metrics.png)

## Caddyfile 

Let's start with a basic `Caddyfile`.

```
:80 {
	handle / {
    	respond "Hello World"
    }
}
```

Write a `docker-compose.yaml` file to run the above `Caddyfile`.

```
version: '3.8'

services:
  caddy:
    image: caddy:latest
    ports:
      - 8080:80
      - 2019:2019
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
```

Now bring up the services using.

```
docker compose up
```

The above will run caddy and make it available it on `localhost:8080`.

> Make sure to expose the port `2019` as well, this is the admin api for Caddy which will be used to collect metrics.

## Enabling metrics 

Use the caddy's [global options](https://caddyserver.com/docs/caddyfile/options) to enable prometheus metrics.

```
{
	servers {
    	metrics
    }
    admin :2019
}

:80 {
	handle / {
    	respond "Hello World"
    }
}
```

The above additional options enables prometheus's metrics endpoint in Caddy.

Re-run the services using `docker compose up` and visit `localhost:2019/metrics`, you should see the metrics.

```
$ curl -X GET localhost:2019/metrics

# TYPE caddy_admin_http_requests_total counter
caddy_admin_http_requests_total{code="200",handler="metrics",method="GET",path="/metrics"} 1
# HELP go_build_info Build information about the main Go module.
# TYPE go_build_info gauge
go_build_info{checksum="",path="",version=""} 1
# HELP go_gc_duration_seconds A summary of the pause duration of garbage collection cycles.
# TYPE go_gc_duration_seconds summary
go_gc_duration_seconds{quantile="0"} 3.3406e-05
go_gc_duration_seconds{quantile="0.25"} 4.1655e-05
go_gc_duration_seconds{quantile="0.5"} 5.0581e-05
go_gc_duration_seconds{quantile="0.75"} 0.000366327
go_gc_duration_seconds{quantile="1"} 0.000366327
go_gc_duration_seconds_sum 0.000491969
go_gc_duration_seconds_count 4
# HELP go_goroutines Number of goroutines that currently exist.
# TYPE go_goroutines gauge
go_goroutines 16
# HELP go_info Information about the Go environment.
# TYPE go_info gauge
go_info{version="go1.20"} 1
# HELP go_memstats_alloc_bytes Number of bytes allocated and still in use.

... more metrics
```

## Setting up Prometheus 

Create the prometheus config.

```
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: caddy
    static_configs:
      - targets: ['caddy:2019']
```

The above adds a new scrape config with url`caddy:2019` (because in docker compose other services are reachable by name).

Add prometheus service in the `docker-compose.yaml` file.

```
version: '3.8'

services:
  caddy:
    image: caddy:latest
    ports:
      - 8080:80
      - 2019:2019
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile

  prometheus:
    image: prom/prometheus:latest
    ports:
      - 9090:9090
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
```

Re-run using `docker compose up`.

Prometheus should start collecting metrics. Open `localhost:9090` and type in a prometheus query.

```
rate(caddy_admin_http_requests_total[1m])
```

![Prometheus Caddy Metrics Query](https://tdc-public.nyc3.digitaloceanspaces.com/photos/804b9e0f-cf26-472c-b9c3-9e3a718e109c-prometheus-caddy-metrics-query.png)

Now you can go ham with the queries!

## Grafana Dashboard

Let's connect Prometheus with Grafana Dashboard for Visualization.

Add `grafana` to the services list in `docker-compose.yaml` file.

```
version: '3.8'

services:
  caddy:
    image: caddy:latest
    ports:
      - 8080:80
      - 2019:2019
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile

  prometheus:
    image: prom/prometheus:latest
    ports:
      - 9090:9090
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - 3000:3000
```

Re-run using `docker compose up`. Grafana dashboard should be running on `localhost:3000`.

## Prometheus Grafana Config [#](https://thedevelopercafe.com/articles/monitoring-caddy-with-prometheus-and-grafana-2f1f869097d8#prometheus-grafana-config)

1. Go to `http://localhost:3000/datasources/new` and select Prometheus.
2. Set the server url as `http://prometheus:9090`.
3. Click `Save and Test`.

![Caddy Prometheus Grafana Metrics Data Source](https://tdc-public.nyc3.digitaloceanspaces.com/photos/9e5b2f1f-4cf2-44b8-999b-689f99740f73-caddy-prometheus-grafana-metrics-data-source.png)

Go to `localhost:3000/explore` and type in the same query.

![Caddy metrics prometheus dashboard](https://tdc-public.nyc3.digitaloceanspaces.com/photos/907a2bd9-8b9d-4e77-8cf0-77cf647f66d5-caddy-metrics-prometheus-dashboard.png)