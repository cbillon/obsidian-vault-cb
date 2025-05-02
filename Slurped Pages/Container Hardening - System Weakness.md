---
link: https://systemweakness.com/container-hardening-999acb9d2692
byline: Iram Jack
site: System Weakness
date: 2024-10-12T08:51
excerpt: Securing Docker containers is crucial for maintaining a robust,
  isolated environment where applications can run without exposing
  vulnerabilities. By hardening containers, we mitigate risks such as…
twitter: https://twitter.com/@systemweakness
slurped: 2025-02-08T09:01
title: Container Hardening - System Weakness
---

## Securing Docker Containers

## Introduction

Securing Docker containers is crucial for maintaining a robust, isolated environment where applications can run without exposing vulnerabilities. By hardening containers, we mitigate risks such as unauthorized access, resource exhaustion, and privilege escalation.

Key practices include restricting container privileges, safeguarding the Docker daemon, controlling resource allocation, and leveraging security frameworks like Seccomp and AppArmor, these measures ensure that containers are not only functional but also resilient against potential threats.

1. **Preventing “Over-Privileged” Containers**

First, it’s important to understand what privileged containers are in this context. Privileged containers are those that have unrestricted access to the host system.

The key principle of containerization is to isolate containers from the host. Running Docker containers in “privileged” mode bypasses the normal security mechanisms that enforce this isolation. While privileged containers can have legitimate uses — such as running Docker-in-Docker (a container inside another container) or for debugging — they pose significant security risks.

When a Docker container runs in “privileged” mode, Docker grants it all possible capabilities, giving it full access to the host system, including filesystems.  
**Capabilities in Linux:**Capabilities are a security feature of Linux that control what processes can and cannot do at a granular level. Traditionally, processes could either have full root privileges or none at all, which could be dangerous if full privileges are unnecessary.

With capabilities, you can fine-tune the privileges a process has, below we have standard capabilities, descriptions, and potential use cases:

- **CAP_NET_BIND_SERVICE:** Allows services to bind to ports below 1024, which typically require root privileges. Letting a web server bind to port 80 without root access.
- **CAP_SYS_ADMIN:** Grants various administrative privileges, such as mounting/unmounting filesystems, changing network settings, and performing system reboots or shutdowns. Used by processes that automate administrative tasks, such as modifying users or managing services.
- **CAP_SYS_RESOURCE:** Allows a process to modify the maximum limit of resources, such as memory or bandwidth. Used for controlling the resources a process can consume, either increasing or decreasing limits.

Privileged containers are granted full root access, which can allow attackers to escape the container and access the host.

To minimize security risks, it is recommended to assign capabilities to containers individually, rather than using the — privileged flag, which grants all capabilities. For example, you can assign the NET_BIND_SERVICE capability to a container running a web server on port 80 with the following command:

    docker run -it --rm --cap-drop=ALL --cap-add=NET_BIND_SERVICE mywebserver

You can also use the command capsh -print to check the capabilities assigned to a process.

Regularly reviewing the capabilities assigned to containers is critical. When a container is privileged, it shares the same namespace as the host, allowing access to host resources and compromising the container’s “isolated” environment.

More details are demonstrated here:  
[https://medium.com/@iramjack8/container-vulnerabilities-part-1-fe6e56de36aa](https://medium.com/@iramjack8/container-vulnerabilities-part-1-fe6e56de36aa)

**2. Protecting the Docker Daemon**

We may recall from the previous [write-up](https://medium.com/@iramjack8/container-vulnerabilities-part-2-3e3ae8c07934) that the Docker daemon is responsible for processing requests such as managing containers and pulling or uploading images to a Docker registry. Docker can be managed remotely, which is often done in CI (**_Continuous Integration)_** and CD (**_Continuous Development_**) pipelines, such as when pushing and running new code in a container on another host to check for errors.

If an attacker can interact with the Docker daemon, they can manipulate containers and images. For instance, they could launch their own malicious containers or gain access to containers running applications with sensitive information, such as databases.

The Docker daemon is not exposed to the network by default and must be manually configured. However, exposing the Docker daemon is a common practice, especially in cloud environments such as CI/CD pipelines. Therefore, implementing secure communication and authentication methods, as listed below, is crucial in preventing unauthorized access to the Docker daemon.

**_SSH:_**

Developers can interact with other devices running Docker using SSH authentication. Docker uses contexts, which act like profiles, allowing developers to save and switch between configurations for different devices. For example, a developer may have one context for a development device and another for a production device running Docker.

**_Note:_** You must have SSH access to the remote device, and your user account must have permission to execute Docker commands.

docker context create

Once completed, you can switch to this context, where all Docker-related commands will now be executed on the remote host.

docker context use development-environment-host

ocker contexts allow secure, encrypted communication with the Docker daemon over SSH.

**_TLS Encryption:_**

The Docker daemon can also be interacted with using HTTP/S, which is useful when a web service or application needs to communicate with Docker on a remote device. To do this securely, you can use the TLS cryptographic protocol to encrypt data sent between devices.

When Docker is configured in TLS mode, it only accepts remote commands from devices signed against the desired remote host.

**_Note:_** Creating and managing TLS certificates is beyond the scope of this discussion, as you will need to consider factors like expiration dates and encryption strength. Once the certificates are created, Docker can be run in TLS mode with the generated certificate.

On the host (server) from which you’re issuing the commands:

dockerd - tlsverify - tlscacert=myca.pem - tlscert=myserver-cert.pem - tlskey=myserver-key.pem -H=0.0.0.0:2376

On the client that issues commands:

docker --tlsverify --tlscacert=myca.pem --tlscert=client-cert.pem --tlskey=client-key.pem -H=SERVERIP:2376 in

It’s essential to remember that this is not inherently secure. For example, anyone with a valid certificate and private key can be recognized as a “**trusted**” device. Below is an explanation of the arguments used when generating a TLS certificate and key:

**_--tlscacert:_** Specifies the certificate of the certificate authority (CA), a trusted entity that issues certificates for device identification.  
**_--tlscert:_** Specifies the certificate used to identify the device.  
**_--tlskey:_** Specifies the private key used to decrypt communication sent to the device.

More details are demonstrated here:  
[https://medium.com/@iramjack8/container-vulnerabilities-part-2-3e3ae8c07934](https://medium.com/@iramjack8/container-vulnerabilities-part-2-3e3ae8c07934)

**3. Implementing Control Groups**

Control Groups (also known as **cgroups**) are a Linux kernel feature that enables restricting and prioritizing the system resources a process can use.

For example, a process like an application can be limited to use only a specific amount of RAM or CPU power, or given priority over other processes. This improves system stability and helps administrators better track resource usage.

In the context of Docker, cgroups are crucial for ensuring isolation and stability, by using cgroups to limit or prioritize the resources a container consumes, you can prevent faulty or malicious containers from exhausting system resources. While preventing such issues is ideal, stopping a container from crashing the entire system is an important fallback.

This behavior is not enabled by default in Docker and must be specified per container at the time of starting the container. The following table shows the switches used to limit resources for a container:

CPU: --cpus (in core count)

docker run -it --cpus="1" mycontainer

Memory: --memory (in k, m, g)

docker run -it --memory="20m" mycontainer

You can also modify resource limits after a container is running by using the docker update command with the new memory value and the container name. For example:

docker update --memory="40m" mycontainer

To view a container’s resource limits, you can use the docker inspect container name command. If a resource limit is set to 0, it means no limits have been configured.

Docker also uses namespaces to create isolated environments. Namespaces allow different actions to occur without affecting other processes, similar to how separate rooms in an office operate independently. This isolation enhances security by preventing processes from interfering with one another.

More details are demonstrated here:  
[https://medium.com/@iramjack8/container-vulnerabilities-part-3-a77f40828a25](https://medium.com/@iramjack8/container-vulnerabilities-part-3-a77f40828a25)

**4. Seccomp & AppArmor 101**

Namespaces in Linux create isolated environments that limit what processes can see or access, they are essential for Docker, as they help maintain the separation between the host system and containers. However, mis-configuring or abusing namespaces can allow attackers to “escape” the container and gain access to the host.  
Docker uses several namespaces to isolate containers, such as: PID namespace, NET namespace, MNT namespace, USER namespace.

**Detecting and Mitigating Container Escape Attempts:**  
It is crucial to monitor containers for signs of potential namespace abuse. Tools like AppArmor or Seccomp can be used to confine container actions. AppArmor and Seccomp limit system calls and actions that a container can perform.

**_Seccomp | Restricting System Calls:_**

Seccomp is a Linux security feature that limits what system calls (syscalls) a program can make, acting like a security guard for applications. It restricts dangerous actions, such as blocking syscalls that could open network connections (e.g., reverse shells) while allowing necessary operations like reading files.

You can apply a custom Seccomp profile to a container at runtime:

docker run --rm -it --security-opt seccomp=/path/to/seccomp/profile.json mycontainer

Docker provides a default Seccomp profile, but customizing one helps tighten security further.

**_AppArmor | Enforcing Access Control:_**

AppArmor is a Linux Mandatory Access Control (MAC) system that limits what processes can access at the operating system level. It prevents unauthorized actions by enforcing a set of predefined rules.

To apply a custom AppArmor profile:

docker run --rm -it --security-opt apparmor=/path/to/apparmor/profile.json mycontainer

Like Seccomp, Docker applies a default AppArmor profile, but custom profiles allow for stronger, more tailored security.

More details are demonstrated here:  
[https://medium.com/@iramjack8/container-vulnerabilities-part-4-d3bd34af5d24](https://medium.com/@iramjack8/container-vulnerabilities-part-4-d3bd34af5d24)

## Conclusion

Effective container hardening enhances security while maintaining system performance. By preventing over-privileged containers, protecting the Docker daemon, enforcing resource controls, and using tools like Seccomp and AppArmor, you significantly reduce the attack surface, these strategies ensure that Docker containers operate securely in any environment, from development to production.