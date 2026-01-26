---
link: https://joshtronic.com/2025/08/10/how-to-rotate-log-files-with-logrotate/
byline: Josh Sherman
excerpt: |-
  Posted 10-Aug-2025
              
              in Servers / Serverless
              
                tagged
                
                  DevOps
slurped: 2026-01-22T10:13
title: How to rotate log files with logrotate
tags:
  - log
---

Posted 10-Aug-2025 in [Servers / Serverless](https://joshtronic.com/category/servers-serverless) tagged [DevOps](https://joshtronic.com/tag/devops)

Part of the fun of [self-hosting](https://joshtronic.com/2025/07/06/technical-independence/) is making sure you follow best practices and don't shoot yourself in the foot. One of those tasks is making sure things are backed up, ideally to an off-site location (I use S3), and are encrypted.

I try to keep things simple. My go-to backup power move is to schedule a [Bash script](https://git.sherver.org/joshtronic/btfu) with cron. Because cron is a scheduler and not a logging system, I direct any output to a file in `/var/log` in case the need to debug arises.

One may think that logging to `/var/log` gets you some sort of log rotation for free. That's definitely not the case, as each file you see being rotated is accounted for in the `/etc/logrotate.d`, most likely with a file provided by the package maintainer.

For one-off scripts of your own design, you will need to provide some direction.

## Create a configuration file

Within the `/etc/logrotate.d` directory you will find a series of files. Each file is a configuration that maps out which log files to target, and how to approach rotating them.

The following configuration will:

- Look for files in `/var/log`
- That match the pattern `backup-*.log`
- Will rotate them daily
- Even if they don't exist
- Retaining 2 weeks worth
- Compressing the files
- But not the most recent ones
- Or if the file is empty
- With the specified permissions

Or more simply:

```
/var/log/backup-*.log {
        daily
        missingok
        rotate 14
        compress
        delaycompress
        notifempty
        create 0640 root adm
 }
```

You can tweak to your liking, but this tends to be pretty sufficient for most scenarios.

## Test out your new configuration

With a new configuration file created, you can now test things out. To perform a dry-run, better known as debug mode in `logrotate` speak, run:

```
logrotate -d /etc/logrotate.d/yourfile
```

This will spit out a bunch of debugging information, finishing with presenting you with your rules in plain text.

To actually test things out, flip the `-d` to `-f` to force execution:

```
logrotate -f /etc/logrotate.d/yourfile
```

Depending on what state your log file was currently in, you will most likely see some rotated files in `/var/log`. Check back in a few days to make sure things are to your liking and call it a day!