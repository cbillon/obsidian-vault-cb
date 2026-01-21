---
link: https://www.vincentschmalbach.com/speeding-up-laravel-tests-with-tmpfs-mysql-in-ram/?utm_source=laravelnews&utm_medium=link&utm_campaign=laravelnews
byline: by Vincent Schmalbach
site: Vincent Schmalbach
date: 2026-01-02T15:59
excerpt: If your Laravel test suite is painfully slow and does a lot of database work (multi-tenancy, migrations, seeding), the bottleneck is almost certainly disk I/O. Not CPU, not PHP, not Docker overhead.
slurped: 2026-01-21T17:58
title: Speeding Up Laravel Tests with tmpfs (MySQL in RAM)
tags:
  - mysql
  - tmpfs
  - docker
---

If your Laravel test suite is painfully slow and does a lot of database work (multi-tenancy, migrations, seeding), the bottleneck is almost certainly disk I/O. Not CPU, not PHP, not Docker overhead. Disk.

The fix: run MySQL on tmpfs.

## My setup

I use Laravel Sail, which runs PHP, MySQL, Redis etc. in Docker containers via a `docker-compose.yml` in your project root. The approach below adds a second MySQL container specifically for tests, running entirely in RAM. Your normal development database stays untouched.

## What does not help

I tried all the usual recommendations you find online:

- **OPcache for CLI**: no improvement
- **Disabling Xdebug**: no improvement
- **Excluding vendor/storage from Docker bind mounts**: no improvement
- **Moving the repo to ext4**: no improvement
- **Using Docker named volumes instead of bind mounts**: no improvement

None of these made a measurable difference for database-heavy tests.

## What helps: tmpfs

tmpfs is a filesystem that lives entirely in RAM. Reading and writing to RAM is orders of magnitude faster than disk I/O. By running MySQL with its data directory on tmpfs, all database operations happen in memory.

This gave me roughly 18x faster test execution. A test suite that previously couldn't finish after 2 hours now completes in under 9 minutes.

## Docker Compose setup

Add the following service to your `docker-compose.yml` (or create a separate override file). This creates a second MySQL container that runs entirely in RAM:

```
services:
  mysql-test:
    image: 'mysql/mysql-server:8.0'
    environment:
      MYSQL_ROOT_PASSWORD: 'password'
      MYSQL_ROOT_HOST: '%'
      MYSQL_DATABASE: 'testing'
      MYSQL_USER: 'sail'
      MYSQL_PASSWORD: 'password'
      MYSQL_ALLOW_EMPTY_PASSWORD: 1
    tmpfs:
      - /var/lib/mysql:size=16G
    command: >
      --default-authentication-plugin=mysql_native_password
      --innodb_flush_log_at_trx_commit=0
      --sync_binlog=0
      --innodb_doublewrite=0
      --skip-log-bin
    networks:
      - sail
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-ppassword"]
      retries: 3
      timeout: 5s
```

Key settings:

- `tmpfs: /var/lib/mysql:size=16G` puts the MySQL data directory in RAM (adjust size based on your available memory)
- `innodb_flush_log_at_trx_commit=0` disables flushing the log on every commit
- `sync_binlog=0` disables syncing the binary log
- `innodb_doublewrite=0` disables the doublewrite buffer
- `skip-log-bin` disables binary logging entirely

These settings are unsafe for production but perfect for tests where data loss doesn't matter.

## Point your tests at the tmpfs container

Create a file called `.env.testing` in your project root (next to your regular `.env`):

```
DB_HOST=mysql-test
DB_DATABASE=testing
```

Laravel automatically loads `.env.testing` when running `artisan test`, so your tests will use the fast tmpfs MySQL while your normal development environment stays on the regular MySQL container.

## Usage

```
./vendor/bin/sail up -d
./vendor/bin/sail artisan test
```

## Notes on Mac vs Linux

I also tested on an M4 Mac mini (see [how I set that up](https://www.vincentschmalbach.com/rent-a-mac-m4-mini-and-access-it-via-ssh-from-linux/)). The Mac was already quite fast without any tmpfs optimizations. The same test suite that took 2+ hours on my Linux machine completed in about 4 minutes on the M4 running natively (no Docker).

So if you're on an M4 Mac, you might not even need these optimizations. If you're on Linux or older hardware, tmpfs makes a huge difference.

Also worth noting: Docker on Mac runs in a VM, which adds significant overhead. On Linux, Docker has almost no overhead compared to native. So if you're on Mac, running PHP and MySQL natively (via Homebrew) will be faster than Docker. On Linux, Docker with tmpfs performs nearly identically to a native tmpfs setup, so you can stick with Sail and still get good performance.

You can set up tmpfs for MySQL natively without Docker as well, but that requires more manual configuration and maintenance. The Docker approach described here is easier to manage and portable across machines.

## Requirements

You need enough RAM to hold your test database in memory. 16GB tmpfs worked fine for my ~1000 test suite. If you have less RAM available, reduce the tmpfs size accordingly, but make sure it's enough for your largest test database state.

## What's next

tmpfs addresses the I/O bottleneck, but there's more you can do at the test suite level. Using database transactions (begin before test, rollback after) instead of recreating the database fresh each time would be even faster. But that requires changes to your test setup and is a separate topic.

## Subscribe to my Newsletter

Get the latest updates delivered straight to your inbox

I respect your privacy. Unsubscribe at any time.