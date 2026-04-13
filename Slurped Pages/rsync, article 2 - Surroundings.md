---
link: https://michael.stapelberg.ch/posts/2022-07-02-rsync-surroundings/
byline: Michael Stapelberg
site: Michael Stapelberg
excerpt: >-
  This post is the second article in a series of blog posts about rsync, see the
  Series Overview.

  Now that we know what to use rsync for, how can we best integrate rsync into
  monitoring and alerting, and on which operating systems does it work?
slurped: 2026-04-08T14:39
title: "rsync, article 2: Surroundings"
---

This post is the second article in a series of blog posts about rsync, [see the Series Overview](https://michael.stapelberg.ch/posts/2022-06-18-rsync-overview/).

Now that we know what to use rsync for, how can we best integrate rsync into monitoring and alerting, and on which operating systems does it work?

## Monitoring and alerting for rsync jobs using Prometheus

Once you have one or two important `rsync` jobs, it might make sense to alert when your job has not completed as expected.

I’m using [Prometheus](https://prometheus.io/) for all my monitoring and alerting.

Because Prometheus _pulls_ metrics from its (typically always-running) targets, we need an extra component: the [Prometheus Pushgateway](https://prometheus.io/docs/practices/pushing/). The Pushgateway stores metrics pushed by short-lived jobs like `rsync` transfers and makes them available to subsequent Prometheus pulls.

To integrate `rsync` with the Prometheus Pushgateway, I wrote [`rsyncprom`](https://github.com/stapelberg/rsyncprom), a small tool that wraps `rsync`, or parses rsync output supplied by you. Once `rsync` completes, `rsyncprom` pushes the rsync exit code and parsed statistics about the transfer to your Pushgateway.

### Prometheus server-side setup

First, I set up the Prometheus Pushgateway (via Docker and systemd) on my server.

Then, in my `prometheus.conf` file, I instruct Prometheus to pull data from my Pushgateway:

```
# prometheus.conf

rule_files:
- backups.rules.yml

scrape_configs:
# […]
- job_name: pushgateway
  honor_labels: true
  static_configs:
  - targets: ['pushgateway:9091']
```

Finally, in `backups.rules.yml`, I configure an alert on the time series `rsync_exit_code`:

```
# backups.rules.yml

groups:
- name: backups.rules
  rules:
  - alert: RsyncFailing
    expr: rsync_exit_code{job="rsync"} > 0
    for: 1m
    labels:
      job: rsync
    annotations:
      description: rsync {{ $labels.instance }} is failing
      summary: rsync {{ $labels.instance }} is failing
```

This alert will fire any time an rsync job monitored via `rsyncprom` exits with a non-zero exit code.

### rsync client-side setup

On each machine that runs `rsync` jobs I want to monitor, I first install `rsyncprom`:

```
go install github.com/stapelberg/rsyncprom/cmd/rsync-prom@latest
```

Then, I just wrap `rsync` transfers where it’s most convenient, for example in my [`crontab(5)`](https://manpages.debian.org/crontab.5) :

```
# crontab -e
9 9 * * * /home/michael/go/bin/rsync-prom --job="cron" --instance="gphotos-sync@midna" -- /home/michael/gphotos-sync/sync.sh
```

The same wrapper technique works in [shell scripts or systemd service files](https://github.com/stapelberg/rsyncprom#setup-example-systemd).

You can also provide `rsync` output [from Go code](https://github.com/stapelberg/zkj-nas-tools/blob/02d46d718df60c413844d9218f6dd702ad94e5f1/dornroeschen/sshutil.go#L134-L139) (this example runs `rsync` via SSH).

### Monitoring architecture

Here’s how the whole setup looks like architecturally:

![rsync-prom architecture](https://michael.stapelberg.ch/posts/2022-07-02-rsync-surroundings/2022-05-29-rsync-prom-architecture.svg)

The rsync scheduler runs on a Raspberry Pi running [gokrazy](https://gokrazy.org/). The scheduler invokes the `rsync` job to back up _websrv.zekjur.net_ via SSH and sends the output to Prometheus, which is running on a (different) server at an ISP.

### Monitoring dashboard

The grafana dashboard looks like this in action:

[![rsync grafana dashboard](https://michael.stapelberg.ch/posts/2022-07-02-rsync-surroundings/2022-06-12-rsync-grafana-featured_hu_f9be59dd23c35f90.jpg "rsync grafana dashboard")](https://michael.stapelberg.ch/posts/2022-07-02-rsync-surroundings/2022-06-12-rsync-grafana-featured.jpg)

- The top left table shows the most recent rsync exit code, green means 0 (success).
- The top right graph shows rsync runtime (wall-clock time) over time. Long runtime can have any number of bottlenecks as the reason: network connections, storage devices, slow CPUs.
- The bottom left graph shows rsync dataset size over time. This allows you to quickly pinpoint transfers that are filling your disk up.
- The bottom right graph shows transferred bytes per rsync over time. The higher the value, the higher the amount of change in your data set between synchronization runs.

## rsync operating system availability

Now that we have learnt about a couple of typical use-cases, where can you use `rsync` to implement these use-cases? The answer is: in most environments, as `rsync` is widely available on different Linux and BSD versions.

Macs come with `rsync` available by default (but it’s an old, patched version), and OpenBSD comes with a BSD-licensed implementation called [openrsync](https://www.openrsync.org/) by default.

On Windows, you can use the [Windows Subsystem for Linux](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux).

| Operating System     | Implementation | Version                                                                                         |
| -------------------- | -------------- | ----------------------------------------------------------------------------------------------- |
| FreeBSD 13.1 (ports) | tridge         | [3.2.3](https://www.freebsd.org/cgi/man.cgi?query=rsync&manpath=FreeBSD+13.1-RELEASE+and+Ports) |
| OpenBSD 7.1          | openrsync      | (7.1)                                                                                           |
| OpenBSD 7.1 (ports)  | tridge         | [3.2.4](https://openports.se/net/rsync)                                                         |
| NetBSD 9.2 (pkgsrc)  | tridge         | [3.2.4](https://ftp.netbsd.org/pub/pkgsrc/current/pkgsrc/net/rsync/index.html)                  |
| Linux                | tridge         | [repology](https://repology.org/project/rsync/versions)                                         |
| macOS                | tridge         | [2.6.9](https://www.unix.com/man-page/osx/1/rsync/)                                             |

## Next Up

The third article in this series is [rsync, article 3: How does rsync work?](https://michael.stapelberg.ch/posts/2022-07-02-rsync-how-does-it-work/). With rsync up and running, it’s time to take a peek under the hood of rsync to better understand how it works.

Did you like this post? [Subscribe to this blog’s RSS feed](https://michael.stapelberg.ch/feed.xml) to not miss any new posts!

I run a blog since 2005, spreading knowledge and experience for over 20 years! :)

If you want to support my work, you can [buy me a coffee](https://www.buymeacoffee.com/stapelberg).

Thank you for your support! ❤️