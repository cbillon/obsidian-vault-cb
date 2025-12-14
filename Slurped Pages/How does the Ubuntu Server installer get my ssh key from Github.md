---
link: https://askubuntu.com/questions/1199553/how-does-the-ubuntu-server-installer-get-my-ssh-key-from-github
byline: |-
  u15p7fgy863eiq5
          
              33411 gold badge22 silver badges88 bronze badges
site: Ask Ubuntu
excerpt: |-
  I couldn't find a way to get my or other people's public key from Github, but the Ubuntu Server installer can.

  How can it and how could I get an ssh public key via GitHub username in a script myself?
slurped: 2025-12-14T09:45
title: How does the Ubuntu Server installer get my ssh key from Github?
tags:
  - ssh
  - SSH-key
---

I couldn't find a way to get my or other people's public key from Github, but the Ubuntu Server installer can.

How can it and how could I get an ssh public key via GitHub username in a script myself?

```
curl -O https://github.com/<username>.keys 
curl -O https://gitlab.com/<username>.keys
```

will retrieve the public keys uploaded for the given username. Both github and gitlab work.

In this particular instance [ssh-import-id](http://apt.ubuntu.com/p/ssh-import-id) retrieves an SSH key from GitHub for you. The tool can import from both GitHub and Launchpad. Normally the tool takes the retrieved key and adds it to your authorized keys file which is what Ubuntu Server would have been doing. The manual page [outlines the mechanics of the specific API calls it makes to retrieve the key](http://manpages.ubuntu.com/manpages/bionic/en/man1/ssh-import-id.1.html).

In addition to [Rinzwind's answer](https://askubuntu.com/a/1199559/653515), [Github has an API for this](https://developer.github.com/v3/users/keys/).

The exact usage is `ssh-import-id gh:github_username`

See the [manpages](http://manpages.ubuntu.com/manpages/xenial/man1/ssh-import-id.1.html)
