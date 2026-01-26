---
link: https://joshtronic.com/2018/11/05/using-alternate-ssh-ports-with-rsync/
byline: Josh Sherman
excerpt: |-
  Posted 05-Nov-2018
              
              in Command-line Interface
slurped: 2026-01-22T09:59
title: Using alternate SSH ports with rsync
tags:
  - ssh
  - port
---

Posted 05-Nov-2018 in [Command-line Interface](https://joshtronic.com/category/command-line-interface)

I live on the command-line.

I also use the hell out of `rsync` to move files around.

And because I'm paranoid / security conscious, I run `sshd` on an alternate port to provide an additional layer of obscurity to things.

While providing me with some piece of mind, using an alternate port with SSH does come with some issues, specifically around needing to specify which port to use when using other apps that leverage `ssh`.

This is actually something I [documented before](https://joshtronic.com/2017/09/27/using-ssh-copy-id-with-an-alternate-ssh-port/) in regard to using `ssh-copy-id`. Similarly, getting `rsync` to play nice with an alternate SSH port just requires some additional command-line arguments.

To get things working, you will need to pass `-e` to `rsync`. The `-e` argument is short for `--rsh` which is an option that allows you to set your "remote shell".

In this instance, our remote shell is SSH with our alternate port:

```
rsync -avz -e "ssh -p 6789" some-file.txt user@server:~/some-path
```

This tells `rsync` "hey, instead of just using `ssh`, let's use this other string in it's place.

Pretty powerful stuff as it's not limited to just specifying a port, you can feed it any additional `ssh` arguments you'd like!

And for the record, I don't use port 6789 for `sshd` ;)