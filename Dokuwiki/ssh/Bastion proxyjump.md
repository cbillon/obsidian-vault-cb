---
tags:
  - bastion
  - ssh
---

====== Commande ProxyJump ======

I am comfortable with using the ProxyCommand feature of ssh and can use it to hop through mulitple bastion hosts to reach the final host efficiently. But I just can't seem to understand how it actually works in the backend.

For eg. I have the following config file.

  Host final
  Hostname final.com
  Port 22
  AgentForwarding yes
  User guestuser
  ProxyCommand "ssh user@bastion.com -W %h:%p"
  
  
  * 1) The user enters ssh final on localhost. This launches the parent ssh process
  * 2) The parent ssh creates a child ssh with I/O redirected to pipes
  * 3) The child ssh creates a connection to bastion.com.
  * 4) The sshd process on bastion.com creates a tcp connection to final.com:22
  * 5) An ssh channel is added to existing ssh connection between localhost and bastion.com
  * 6) Parent ssh writes the handshake data to the pipe, the child ssh reads it from the pipe, sends via the ssh channel to sshd on bastion.com;sshd reads it and writes it to the socket connected to final.com. Similarly, the data is transmitted from final.com to localhost


Long story short, ProxyCommand is just a command to build a pseudo-socket in replace of the real socket addr:port to the SSH server.
%h and %p is tokens for ProxyCommand to replace %h to target host and %p to target port. So ssh user@bastion.com -W %h:%p will be replaced to ssh user@bastion.com -W final.com:22.