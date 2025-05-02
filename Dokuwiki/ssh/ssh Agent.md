---
tags:
  - ssh
---

====== SSH Agent ======

[ssh agaent](https://www.ssh.com/academy/ssh/agent)

[Passing key ssh Agent](http://www.whiteboardcoder.com/2018/06/passing-keys-ssh-agent.html)


Start the ssh-agent


   eval $(ssh-agent)


Now add your private keys  using the ssh-add tool


   ssh-add ~/.ssh/id_rsa


As a double check run this to list your keys


  ssh-add -l
  
  
ssh-add adds private key identities to the authentication agent, ssh-agent(1). When run without arguments, it adds the files ~/.ssh/id_rsa, ~/.ssh/id_dsa, ~/.ssh/id_ecdsa and ~/.ssh/identity.[...]

Identity files should not be readable by anyone but the user. Note that ssh-add ignores identity files if they are accessible by others.

So, because of The agent has no identities. error, you probably don't have those files or maybe those files are accessible by others. You can check these using the following command:

ls -l ~/.ssh
Also, after you run ssh-add command, run echo $? to see the error status of the previous command. If exit status returned 0, the command was executed successfully. If exit status returned a non-zero value, the command failed to execute.

Now ssh to a box and use the -A option to enable forwarding of the authentication agent connection to the machine you are ssh’n to.

Here is my example
 (Of course replace haproxy with your hostname J )


   ssh -A haproxy
  
As another double check you can run this command on the machine you just logged into via ssh to confirm the key was sent along


   ssh-add -l


Wahoo it worked… Now if I try to run a git clone via ssh…


  git clone git@github.com:EXAMPLE/some-private-repo.git



That fulfills my needs
Now there are lots of other useful things you can do with this such as use it to ssh to yet another box which contained your public key.