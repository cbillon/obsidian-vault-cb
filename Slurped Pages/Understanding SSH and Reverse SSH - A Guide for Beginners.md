---
link: https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch
byline: Richard Chamberlain
site: DEV Community
date: 2024-12-08T17:34
excerpt: Learn the fundamentals of Reverse SSH with this beginner-friendly guide. Discover how to set up Reverse SSH to navigate network restrictions, automate connections, and securely manage remote devices. Explore practical use cases, step-by-step setup instructions, and essential tips for secure configurations.
twitter: https://twitter.com/@SebosInfo
tags:
  - ssh
  - remote
slurped: 2025-03-12T13:17
title: "Understanding SSH and Reverse SSH: A Guide for Beginners"
---

When I first started using SSH, I saw it as a straightforward way to access a remote server from my local desktop. In the simplest terms, SSH (Secure Shell) was my go-to tool for connecting to a remote server to execute commands. The typical usage was clear: initiate a connection from my local machine, and access a remote resource.

However, as I delved deeper into SSH concepts like jump and tunnel connections, I began to see the power of creating secure, encrypted tunnels. These tunnels allow traffic to pass securely between machines. Even then, the model was still straightforward—I’d start locally and connect to remote resources.

But what happens when your local machine doesn’t have direct access to the remote resource? This is where **Reverse SSH** becomes a valuable tool.

---

Here's a brief table of contents for your article:

---

## [](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#table-of-contents)Table of Contents

1. [Introduction](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#introduction)
2. [What Is Reverse SSH?](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#what-is-reverse-ssh)
3. [Setting Up Reverse SSH](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#setting-up-reverse-ssh)
    - [Restricting Access](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#restricting-access)
    - [Starting the Reverse SSH Connection](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#starting-the-reverse-ssh-connection)
4. [Why Use Reverse SSH?](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#why-use-reverse-ssh)
5. [Conclusion](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#conclusion)

---

You can now link this table to the corresponding sections in the article for easy navigation.

### [](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#what-is-reverse-ssh)What Is Reverse SSH?

Reverse SSH leverages a common network rule: **traffic initiated from a device is generally allowed to return to that device**. This principle underpins much of how web browsing works. Normally, SSH involves a local device initiating a connection to a remote server. But in cases where firewall rules or network configurations block this access, Reverse SSH offers a solution.

With Reverse SSH, the remote device initiates the SSH connection, creating a secure tunnel back to the local device. Through this tunnel, the local device can communicate with the remote server as if it had direct access.

---

### [](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#setting-up-reverse-ssh)Setting Up Reverse SSH

To illustrate how Reverse SSH works, I tested it using three devices:

1. **Remote device (192.168.178.17):** Initiates the Reverse SSH connection.
2. **Local device (192.168.178.19):** Utilizes the Reverse SSH connection to access the remote server.
3. **Testing device (192.168.178.10):** Always maintains direct SSH access to the remote device.

Initially, both the local and testing devices had SSH access to the remote device.

#### [](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#restricting-access)Restricting Access

To simulate a real-world scenario, I modified the firewall on the remote device to allow SSH access only from the testing device:  

```
sudo ufw allow from 192.168.178.10 to any port 22
sudo ufw deny 22
sudo ufw enable
```

 

After applying these rules, the local device could no longer access the remote device directly.

#### [](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#starting-the-reverse-ssh-connection)Starting the Reverse SSH Connection

Next, I initiated a Reverse SSH connection from the remote device to the local device:  

```
ssh -R 7000:localhost:22 richard@192.168.178.19
```

 

This command forwards SSH traffic from the remote device’s port 22 through a tunnel to port 7000 on the local device. With this tunnel in place, the local device can connect back to the remote device:  

```
ssh richard@localhost -p 7000
```

 

This command uses the tunnel to access the remote device, granting terminal access as if the local device had a direct connection.

To run the Reverse SSH connection in the background, you can use the following command:  

```
ssh -f -N -R 7000:localhost:22 richard@192.168.178.19
```

 

- `-f`: Runs the SSH session in the background.

## [](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#-raw-n-endraw-useful-for-port-forwarding-without-executing-remote-commands)- `-N`: Useful for port forwarding without executing remote commands.

## [](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#but-why-would-you-need-reverse-ssh-if-you-already-have-access-to-the-remote-device)But why would you need Reverse SSH if you already have access to the remote device?

### [](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#why-use-reverse-ssh)Why Use Reverse SSH?

You might wonder: if the connection starts with the remote server, doesn’t that imply access is already available? The answer lies in automation. Reverse SSH is often used for scenarios where remote devices—like IoT devices—need to initiate a connection or, in a worst-case scenario, allow unauthorized control of a remote server.

In my case, I discovered that adding a simple line to my SSH configuration effectively blocked Reverse SSH, even when the firewall rules didn’t:  

```
AllowUsers richard@192.168.178.10  # Restrict to Richard on the testing device
```

 

This highlights the importance of carefully restricting which users and devices can access your SSH servers.

---

### [](https://dev.to/sebos/understanding-ssh-and-reverse-ssh-a-guide-for-beginners-18ch#conclusion)Conclusion

Reverse SSH is a powerful tool for navigating network restrictions, enabling secure communication when direct connections aren’t possible. It’s especially useful for automation or managing devices in restrictive environments.

So, how would you use Reverse SSH in your setup? Whether for troubleshooting, automation, or security testing, it’s a technique worth mastering.