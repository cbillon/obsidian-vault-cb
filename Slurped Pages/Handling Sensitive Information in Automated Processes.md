---
link: https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm
byline: Richard Chamberlain
site: DEV Community
date: 2024-11-17T21:51
excerpt: Learn secure password management techniques for automated processes. Explore methods using Ansible Vault, SSH keys, hashed passwords, and file encryption to protect sensitive data effectively.
twitter: https://twitter.com/@SebosInfo
tags:
  - secret
  - ssh
  - gpg
slurped: 2025-03-12T13:40
title: Handling Sensitive Information in Automated Processes
---

When running automated processes like **Ansible** playbooks via systemd timers or cron jobs, securely managing sensitive information, such as passwords, becomes crucial. This information must remain encrypted at rest and be securely decrypted during runtime. Here’s a guide to handling this challenge effectively.  
Here’s a Markdown table of contents for the blog post:

## [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#table-of-contents)Table of Contents

1. [Handling Sensitive Information in Automated Processes](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#handling-sensitive-information-in-automated-processes)
2. [The Challenge of Secure Password Management](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#the-challenge-of-secure-password-management)
3. [Using Password Files with Strict Permissions](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#using-password-files-with-strict-permissions)
4. [Generating Passwords with SSH Keys and Hashing Tools](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#generating-passwords-with-ssh-keys-and-hashing-tools)
5. [Using Ansible Vault with a Hashed Password](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#using-ansible-vault-with-a-hashed-password)
    - [Creating an Encrypted File](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#creating-an-encrypted-file)
    - [Using the Encrypted File in a Playbook](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#using-the-encrypted-file-in-a-playbook)
    - [Example Workflow](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#example-workflow)
6. [Is This a Serious Solution?](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#is-this-a-serious-solution)
    - [Pros](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#pros)
    - [Cons](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#cons)
7. [Encrypting Files at Rest](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#encrypting-files-at-rest)
8. [Final Thoughts](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#final-thoughts)

### [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#the-challenge-of-secure-password-management)The Challenge of Secure Password Management

The key question is: **How can passwords or sensitive information be stored and accessed securely during runtime?** While there’s no one-size-fits-all solution, the method you choose should balance security, simplicity, and practicality.

---

### [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#using-password-files-with-strict-permissions)Using Password Files with Strict Permissions

One common approach is to store the password in a file with strict permissions so that only the application requiring it can access it. For example:  

```
echo "securE-p@ssw0rd" > password_file.txt
chmod 600 password_file.txt
sudo chown app_owner:app_owner password_file.txt
```

 

In this setup:

- The file’s permissions (`chmod 600`) ensure only the owner can read or write to it.
- The ownership (`chown`) restricts access to the application user.

**Important Note:** The password file should not have an obvious name like `password_file.txt`. Use a non-descriptive name to avoid drawing attention to its purpose.

---

### [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#generating-passwords-with-ssh-keys-and-hashing-tools)Generating Passwords with SSH Keys and Hashing Tools

An interesting alternative is to derive a password from an **SSH key** using a hashing tool. This method creates a repeatable, secure password without storing it in plaintext:  

```
## Create a test SSH authentication key
ssh-keygen -t ed25519 -f innocent_file

## Use hashing tool to generate a password
echo $(cat innocent_file | sha256sum | awk '{print $1}')
```

 

For example, hashing the key file might generate a password like:  

```
604e8f705bdb411ff3813e4fd536a52e6e25545d3ff5ea6d038d460e84201c88
```

 

---

### [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#using-ansible-vault-with-a-hashed-password)Using Ansible Vault with a Hashed Password

Ansible provides a built-in solution for encrypting sensitive information: **Ansible Vault**. This approach allows you to securely store and use sensitive data in playbooks.

#### [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#creating-an-encrypted-file)Creating an Encrypted File

You can create an encrypted file using the hashed password:  

```
ansible-vault create encrypted_file --vault-password-file <(echo $(cat innocent_file | sha256sum | awk '{print $1}'))
```

 

#### [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#using-the-encrypted-file-in-a-playbook)Using the Encrypted File in a Playbook

When running a playbook, provide the vault password as a hashed value:  

```
ansible-playbook <some_file>.yml --vault-password-file <(echo $(cat innocent_file | sha256sum | awk '{print $1}'))
```

 

#### [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#example-workflow)Example Workflow

In one implementation, the `innocent_file` (SSH key) was generated on a separate server. It was securely transferred to the Ansible server when needed and deleted after use. This added an extra layer of security by ensuring the password was ephemeral.

---

### [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#is-this-a-serious-solution)Is This a Serious Solution?

The SSH key-based approach is not a universal solution but has its use cases:

- **Pros**:
    
    - Quick and easy for generating secure passwords.
    - No need to store plaintext passwords on disk.
    - Repeatable and deterministic.
- **Cons**:
    
    - Relies on securing the SSH key file (`innocent_file`).
    - Requires additional steps for transferring and managing the key file.
    - Might not meet compliance standards for environments requiring long-term key management.

For environments with stringent security requirements, more robust solutions like a **secrets manager** (e.g., HashiCorp Vault, AWS Secrets Manager) might be better suited.

---

### [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#encrypting-files-at-rest)Encrypting Files at Rest

Encrypting files at rest is a fundamental security practice. Ansible Vault is a powerful tool for this purpose, offering seamless integration with Ansible workflows. For non-Ansible processes, consider using tools like **GPG** or **OpenSSL** to encrypt sensitive files before storing them.

Example using GPG:  

```
gpg --output encrypted_file.gpg --symmetric --cipher-algo AES256 sensitive_file.txt
```

 

To decrypt at runtime:  

```
gpg --decrypt encrypted_file.gpg > sensitive_file.txt
```

 

---

### [](https://dev.to/sebos/handling-sensitive-information-in-automated-processes-3inm#final-thoughts)Final Thoughts

Securely managing sensitive information in automated workflows requires a thoughtful approach. Whether you choose password files, hashed keys, or secrets management solutions, always assess your security needs and operational constraints. The methods outlined here provide a good starting point, but remember: **there’s no substitute for a well-designed security strategy.**