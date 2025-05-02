---
link: https://dev.to/sebos/streamlining-ssh-key-management-93b
byline: Richard Chamberlain
site: DEV Community
date: 2024-10-13T21:47
excerpt: This article explores how to enhance the security and organization of SSH authentication keys using a custom script. By automating key generation and managing SSH configurations with include files, this approach simplifies server connections while boosting security.
twitter: https://twitter.com/@SebosInfo
tags:
  - ssh
  - SSH-key
slurped: 2025-03-12T13:52
title: Streamlining SSH Key Management
---
## [](https://dev.to/sebos/streamlining-ssh-key-management-93b#automating-and-securing-ssh-configurations-with-a-custom-scrip)Automating and Securing SSH Configurations with a Custom Scrip

When managing multiple servers on a daily basis, SSH authentication keys are invaluable. In the past, I would typically create a single key pair and reuse it across all my servers. While convenient, this approach poses a significant security risk—if that key were to be compromised, it could expose all of my servers to potential threats. To mitigate this risk, generating unique key pairs for each server is the best practice, but managing all those keys can quickly clutter the `.ssh` directory. To address this, I developed a Bash script that not only generates key pairs but also keeps the directory organized and simplifies the process overall.

## [](https://dev.to/sebos/streamlining-ssh-key-management-93b#the-sshconfig)the .ssh/config

SSH relies on the `.ssh/config` file to store information about various SSH connections. In this file, you define several key parameters:

- **Host** – The alias used to reference the server.
- **HostName** – The server’s IP address or DNS name.
- **User** – The username SSH will use to log in.
- **IdentityFile** – The path to the SSH key pair for authentication.

By using include files within the SSH config, it becomes much easier to organize multiple server logins. The script I developed automates this process by creating individual config files for each server and linking them back to the main `.ssh/config` file. Here's how the script works:

## [](https://dev.to/sebos/streamlining-ssh-key-management-93b#how-the-script-works)How the Script Works

The script takes a hostname, IP address, and username as inputs. It then generates the necessary SSH keys, creates a config file for the server, and automatically links that file to the main `.ssh/config` file.

### [](https://dev.to/sebos/streamlining-ssh-key-management-93b#server-directory)Server Directory

Creates the directory to store the SSH key pair and configuration file  

```
# Define directories
config_directory=/home/${local_user}/.ssh/include.d/${host_name}
echo "${config_directory}"
# Check if the SSH config directory exists, create it if not
if [ ! -d "${config_directory}" ]; then
  echo "Creating SSH config directory at ${config_directory}..."
  mkdir -p ${config_directory}
  if [ $? -ne 0 ]; then
    echo "Error: Failed to create config directory at ${config_directory}"
    exit 1
  fi
fi
```

 

### [](https://dev.to/sebos/streamlining-ssh-key-management-93b#file-permissions)File Permissions

To secure the keys generated in the next steps, the default file permissions must be set to `600`. This ensures that only the file owner has read and write access. The following code sets the correct permissions on the parent directory to maintain this level of security.  

```
# Set default ACL permissions on the login directory
setfacl -d -m u::rw,g::-,o::- ${config_directory}
if [ $? -ne 0 ]; then
  echo "Error: Failed to set ACL on ${config_directory}"
  exit 1
fi
```

 

### [](https://dev.to/sebos/streamlining-ssh-key-management-93b#generates-the-keys)Generates the Keys

With a designated place to store the keys, the script proceeds to generate the SSH key pair and copy them to the server

> **Note:** The script does not prompt for a password when generating the key pair, but it will ask for the remote server’s login password during the connection process.  

```
# Generate a new SSH key pair using ed25519 algorithm, with no passphrase (-N "")
ssh-keygen -t ed25519 -f ${config_directory}/${host_name} -N ""  # Empty passphrase
if [ $? -ne 0 ]; then
  echo "Error: SSH key generation failed for ${host_name}"
  exit 1
fi

# Copy the SSH key to the remote host
echo "Copying SSH public key to ${user}@${ip_address}..."
ssh-copy-id -i ${config_directory}/${host_name}.pub ${user}@${ip_address}
if [ $? -ne 0 ]; then
  echo "Error: Failed to copy SSH public key to ${user}@${ip_address}"
  exit 1
fi

```

 

### [](https://dev.to/sebos/streamlining-ssh-key-management-93b#ssh-config-file-for-server).ssh config File for Server

The config file for the new login is created and stored in the same directory as the SSH keys.  

```
# Create the config file inside the config_directory and write the necessary SSH config lines
echo "Creating SSH config file at ${config_directory}/config..."
cat <<EOL > ${config_directory}/config
Host ${host_name}
     HostName ${ip_address}
     User ${user}
     IdentityFile ${config_directory}/${host_name}
EOL

```

 

### [](https://dev.to/sebos/streamlining-ssh-key-management-93b#making-server-callable)Making Server Callable

One of the key benefits of using a config file is that it simplifies connecting to servers. By linking the server’s address and username to a single alias, you can easily initiate a connection with a simple `ssh <alias>` command, eliminating the need to remember or type the full server details each time.  
The next step adds the server’s config file to the main `.ssh/config`, ensuring it's included for future SSH connections.  

```
# Append the Include line to ~/.ssh/config if it's not already present
ssh_config_file=/home/${local_user}/.ssh/config

# Ensure the ~/.ssh/config file exists
if [ ! -f "${ssh_config_file}" ]; then
  touch "${ssh_config_file}"
  chmod 600 "${ssh_config_file}"
fi

# Check if the Include line is already in the file
if ! grep -Fxq "Include ${config_directory}/config" "${ssh_config_file}"; then
  echo "Adding 'Include ${config_directory}/config' to ${ssh_config_file}..."
  #echo "Include ${config_directory}/config" >> "${ssh_config_file}"
  echo "Include ${config_directory}/config" | cat - "${ssh_config_file}" > temp_file && mv temp_file ${ssh_config_file}
  if [ $? -ne 0 ]; then
    echo "Error: Failed to append the Include line to ${ssh_config_file}"
    exit 1
  fi
fi


```

 

### [](https://dev.to/sebos/streamlining-ssh-key-management-93b#test-the-connection)Test the Connection

Finally, it test the login:  

```
# Try to SSH into the server using the newly created key
ssh ${host_name}
if [ $? -ne 0 ]; then
  echo "Error: Failed to connect to ${host_name}"
  exit 1
fi

echo "SSH key successfully created, user logged into the server, and config files updated."

```

 

Don't forget to logout the remote server at the end of the script.

Creating a new authentication key and login is simple:

**Usage:** `create_ssh_login.bash <host_name> <ip_address> <username>`

Once the script completes, you can easily connect to the server with:

`ssh <hostname>`

Running it for:

`create_ssh_login.bash proxmox 192.168.177.7 richard`  
will create the blow directory structure.  

```
~/.ssh
├── config
├── include.d/
│   └── proxmox/
│      ├── proxmox.pub
│      ├── proxmox
│      ├── config


```

 

What I appreciate about this script is that it enhances SSH key security while making it easier to generate keys and keep them well-organized.

For the full version of the script, visit [https://github.com/richard-sebos/Streamlining-SSH-Key](https://github.com/richard-sebos/Streamlining-SSH-Key).
