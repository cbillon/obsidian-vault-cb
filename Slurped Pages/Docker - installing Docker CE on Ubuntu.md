---
link: https://fabianlee.org/2023/09/14/docker-installing-docker-ce-on-ubuntu/
site: "Fabian Lee : Software Engineer"
excerpt: "Docker is a container platform that streamlines software delivery and
  provides isolation, scalability, and efficiency with less overhead than OS
  level virtualization. These instructions are taken from the official Docker
  for Ubuntu page, but I fine-tuned them per Ubuntu22+ standards. Uninstall
  older versions for pkg in docker.io docker-doc docker-compose podman-docker
  containerd runc; do sudo apt ... Docker: installing Docker CE on Ubuntu"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/community
  - slurp/docker
  - slurp/edition
  - slurp/install
  - slurp/jammy
  - slurp/signed-by
  - slurp/socket
  - slurp/tcp
  - slurp/ubuntu
  - slurp/unix
slurped: 2025-03-02T10:04
title: "Docker: installing Docker CE on Ubuntu"
---

![](https://fabianlee.org/wp-content/uploads/2017/03/docker-logo.png)[Docker](https://www.docker.com/) is a container platform that streamlines software delivery and provides isolation, scalability, and efficiency with less overhead than OS level virtualization.

These instructions are taken from the [official Docker for Ubuntu page](https://docs.docker.com/engine/installation/linux/ubuntu/), but I fine-tuned them per Ubuntu22+ standards.

### Uninstall older versions

for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt remove $pkg; done

### Setup Docker Repository

# additional packages
sudo apt install -y ca-certificates apt-transport-https ca-certificates curl gnupg software-properties-common

# add docker gpg key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker.gpg
sudo chmod 0644 /usr/share/keyrings/docker.gpg

# add to repository list
echo "deb [arch="$(dpkg --print-architecture)" signed-by=/usr/share/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -sc) stable" | sudo tee /etc/apt/sources.list.d/docker.list

### Install Docker Community Edition

# update apt and install
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin

This should have installed docker which is running under systemd. The docker engine logs should report “Started Docker Application Container Engine”.

sudo journalctl -u docker.service

systemd will already have the service configured to start on boot, but if you want to check:

# ensure start on boot 
sudo systemctl enable docker

# check status
sudo systemctl status docker.service

# check most recent logs
sudo journalctl -u docker.service -r --no-pager

If you see a lot of “Channel authority” and “SubChannel” messages from the logs instead of simple “Server created” and “Daemon has completed initialization”, then you need to configure the DNS used by the Docker daemon.

# create docker daemon config file
# set DNS to google public dns server
sudo mkdir -p /etc/docker
cat <<EOF  | sudo tee /etc/docker/daemon.json
{
"dns": ["8.8.8.8"]
}
EOF

# restart daemon, show logs again
sudo systemctl restart docker.service
sudo journalctl -u docker.service -r --no-pager

### Validate Install

# version installed
sudo docker version

# quick test of container
sudo docker run hello-world

The docker run command from above should return a message like:

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it to your terminal.

### Docker without sudo

You have have noticed that you had to run docker with sudo in the hello-world test above. That is because the Docker daemon binds to a unix socket which is owned by root. If you don’t want to force sudo access, then you can create a ‘docker’ group and add your user to it.

# group already created, but go ahead and make sure
sudo groupadd docker

# add self to docker group
sudo usermod -aG docker $USER

# reevaluate group memberships without exiting
exec su -l $USER

# should list 'docker' group now
id

Logout, then log back in to make sure the membership is reevaluated (su – $USER). Then test by running hello-world again without sudo.

docker run hello-world

### Docker via TCP

By default, the docker daemon listens on a local unix port owned by root (/run/docker.sock) which means your docker client will only work from the installed server and as sudo.

If you want to use a remote docker CLI to connect to this server, you need to consider the security implications and then you can create the [x509 certs to secure the communication as described in the official docs](https://docs.docker.com/engine/security/https/).

REFERENCES

[docker, Official install doc](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

NOTES

backing storage for docker

sudo docker info | grep -i storage -A2