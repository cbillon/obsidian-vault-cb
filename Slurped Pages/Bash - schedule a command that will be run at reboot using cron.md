---
link: https://fabianlee.org/2023/11/30/bash-schedule-a-command-that-will-be-run-at-reboot-using-cron/
site: "Fabian Lee : Software Engineer"
excerpt: 'If there is a command that needs to run on startup/reboot, you can use
  the @reboot directive in cron to schedule it.  Here is a simple example of
  logging the date to a file at startup. (crontab -l 2>/dev/null; echo "@reboot
  date >> ~/startup.log") | sort -u | crontab - The sort command avoids
  duplicate ... Bash: schedule a command that will be run at reboot using cron'
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/crontab
  - slurp/editor
  - slurp/reboot
  - slurp/restart
  - slurp/script
  - slurp/stdin
  - slurp/ubuntu
  - slurp/without
slurped: 2025-03-02T09:58
title: "Bash: schedule a command that will be run at reboot using cron"
---

November 30, 2023

  

Categories: [Linux](https://fabianlee.org/category/linux/)  

![](https://fabianlee.org/wp-content/uploads/2018/10/gnu-logo.gif)If there is a command that needs to run on startup/reboot, you can use the @reboot directive in cron to schedule it.  Here is a simple example of logging the date to a file at startup.

(crontab -l 2>/dev/null; echo "@reboot date >> ~/startup.log") | sort -u | crontab -

The sort command avoids duplicate commands being written to the crontab.

The command runs early in the startup process (before multiuser mode is initialized), so as another example, if you wanted to ensure the ssh keys in the “/etc/ssh” directory were populated before the ssh daemon was started, the command below would work.

(crontab -l 2>/dev/null; echo "@reboot ssh-keygen -A") | sort -u | crontab -

Note that “ssh-keygen -A” command is idempotent and does not overwrite ssh keys in “/etc/ssh” that already exist.

### Running only once

If you wanted this entry to be only run on a single reboot, you could have cron invoke a Bash script that either:

- Removed its own cron entry
- OR Created a lock/file signaling it had already been run, and check for this in the script logic

REFERENCES

[Baeldung, create crontab script](https://www.baeldung.com/linux/create-crontab-script)

[Phoenixnap.com, reboot command showing date](https://phoenixnap.com/kb/crontab-reboot)