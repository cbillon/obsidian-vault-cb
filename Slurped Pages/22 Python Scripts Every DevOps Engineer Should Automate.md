---
link: https://blog.devops.dev/22-python-scripts-every-devops-engineer-should-automate-0df7177675cd
byline: Obafemi
site: DevOps.dev
date: 2025-03-05T17:56
excerpt: let’s get into it. “22 Python Scripts Every DevOps Engineer Should Automate” is published by Obafemi in DevOps.dev.
twitter: https://twitter.com/@devops_blog
slurped: 2025-03-08T07:45
title: 22 Python Scripts Every DevOps Engineer Should Automate
tags:
  - python
  - script
---

## 1. System Resource Monitoring

This script checks CPU and memory usage, sending alerts if any threshold is exceeded.

import psutil  
def check_system_resources():  
 cpu_usage = psutil.cpu_percent(interval=1)  
 memory_usage = psutil.virtual_memory().percent

  if cpu_usage > 80:  
 print(f"High CPU usage: {cpu_usage}%")  
 if memory_usage > 80:  
 print(f"High Memory usage: {memory_usage}%")  
check_system_resources()

## 2. Disk Usage Monitoring and Alerting

Monitor disk space and trigger an alert if usage exceeds 80%.

import shutil  
def check_disk_usage(path="/"):  
 total, used, free = shutil.disk_usage(path)  
 usage_percentage = (used / total) * 100

  if usage_percentage > 80:  
 print(f"Warning! Disk usage is {usage_percentage:.2f}%")  
check_disk_usage()

## 3. Automated Backup Script

This script compresses and backs up a directory to a specified location.

import shutil  
import time  
def backup_directory(source, destination):  
 timestamp = time.strftime("%Y%m%d%H%M%S")  
 backup_file = f"{destination}/backup_{timestamp}.tar.gz"  
 shutil.make_archive(backup_file.replace(".tar.gz", ""), 'gztar', source)  
 print(f"Backup completed: {backup_file}")  
backup_directory("/path/to/important/data", "/path/to/backup/location")