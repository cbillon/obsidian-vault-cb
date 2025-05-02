---
tags:
  - bitbucket
---


====== Change key ======


[[https://bitbucket.org/blog/ssh-host-key-changes]]

===== IDENTIFY IF YOUR CLIENT IS IMPACTED =====

To verify which host key your SSH client is using, you can run the following command:

  ssh git@bitbucket.org host_key_info
  

==== CONFIGURE YOUR CLIENT TO TRUST THE NEW HOST KEYS ====

If neither new fingerprints appear in the output of your OpenSSH (or compatible) client, you can configure the new trusted host keys in the known_hosts file with these commands:

   ssh-keygen -R bitbucket.org && curl https://bitbucket.org/site/ssh >> ~/.ssh/known_hosts
