---
link: https://austinsnerdythings.com/2021/08/30/how-to-create-a-proxmox-ubuntu-cloud-init-image/
byline: Austin
site: Austin's Nerdy Things
date: 2021-08-31T03:45
excerpt: Easily create a Proxmox Ubuntu cloud-init image for use with Terraform,
  Ansible, and other automation tools
slurped: 2025-02-10T19:07
title: How to create a Proxmox Ubuntu cloud-init image - Austin's Nerdy Things
---

## Background for why I wanted to make a Proxmox Ubuntu cloud-init image

I have recently ventured down the path of attempting to learn CI/CD concepts. I have tried docker multiple times and haven’t really enjoyed the nuances any of the times. To me, LXC/LXD containers are far easier to understand than Docker when coming from a ‘one VM per service’ background. LXC/LXD containers can be assigned IP addresses (or get them from DHCP) and otherwise behave basically exactly like a VM from a networking perspective. Docker’s networking model is quite a bit more nuanced. Lots of people say it’s easier, but having everything run on ‘localhost:[high number port]’ doesn’t work well when you’ve got lots of services, unless you do some reverse proxying, like with Traefik or similar. Which is another configuration step.

It is so much easier to just have a LXC get an IP via DHCP and then it’s accessible from hostname right off the bat (I use pfSense for DHCP/DNS – all DHCP leases are entered right into DNS). Regardless, I know Kubernetes is the new hotness so I figured I need to learn it. Every tutorial says you need a master and at least two worker nodes. No sense making three separate virtual machines – let’s use the magic of virtualization and clone some images! I plan on using Terraform to deploy the virtual machines for my Kubernetes cluster (as in, I’ve already used this Proxmox Ubuntu cloud-init image to make my own Kubernetes nodes but haven’t documented it yet).

## Overview

The quick summary for this tutorial is:

1. Download a base Ubuntu cloud image
2. Install some packages into the image
3. Create a Proxmox VM using the image
4. Convert it to a template
5. Clone the template into a full VM and set some parameters
6. Automate it so it runs on a regular basis (extra credit)?
7. ???
8. Profit!

## Youtube Video Link

If you prefer video versions to follow along, please head on over to [https://youtu.be/1sPG3mFVafE](https://youtu.be/1sPG3mFVafE) for a live action video of me creating the Proxmox Ubuntu cloud-init image and why we’re running each command.

### #1 – Downloading the base Ubuntu image

Luckily, Ubuntu (my preferred distro, guessing others do the same) provides base images that are updated on a regular basis – [https://cloud-images.ubuntu.com/](https://cloud-images.ubuntu.com/). We are interested in the “current” release of Ubuntu 20.04 Focal, which is the current Long Term Support version. Further, since Proxmox uses KVM, we will be pulling that image:

wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img

### #2 – Install packages

It took me quite a while into my Terraform debugging process to determine that qemu-guest-agent wasn’t included in the cloud-init image. Why it isn’t, I have no idea. Luckily there is a very cool tool that I just learned about that enables installing packages directly into a image. The tool is called virt-customize and it comes in the libguestfs-tools package (“libguestfs is a set of tools for accessing and modifying virtual machine (VM) disk images” – [https://www.libguestfs.org/](https://www.libguestfs.org/)).

Install the tools:

sudo apt update -y && sudo apt install libguestfs-tools -y

Then install qemu-guest-agent into the newly downloaded image:

sudo virt-customize -a focal-server-cloudimg-amd64.img --install qemu-guest-agent

At this point you can presumably install whatever else you want into the image but I haven’t tested installing other packages. qemu-guest-agent was what I needed to get the VM recognized by Terraform and accessible.

Update 2021-12-30 – it is possible to inject the SSH keys into the cloud image itself before turning it into a template and VM. You need to create a user first and the necessary folders:

# not quite working yet. skip this and continue
#sudo virt-customize -a focal-server-cloudimg-amd64.img --run-command 'useradd austin'
#sudo virt-customize -a focal-server-cloudimg-amd64.img --run-command 'mkdir -p /home/austin/.ssh'
#sudo virt-customize -a focal-server-cloudimg-amd64.img --ssh-inject austin:file:/home/austin/.ssh/id_rsa.pub
#sudo virt-customize -a focal-server-cloudimg-amd64.img --run-command 'chown -R austin:austin /home/austin'

### #3 – Create a Proxmox virtual machine using the newly modified image

The commands here should be relatively self explanatory but in general we are creating a VM (VMID=9000, basically every other resource I saw used this ID so we will too) with basic resources (2 cores, 2048MB), assigning networking to a virtio adapter on vmbr0, importing the image to storage (your storage here will be different if you’re not using ZFS, probably either ‘local’ or ‘local-lvm’), setting disk 0 to use the image, setting boot drive to disk, setting the cloud init stuff to ide2 (which is apparently appears as a CD-ROM to the VM, at least upon inital boot), and adding a virtual serial port. I had only used qm to force stop VMs before this but it’s pretty useful.

sudo qm create 9000 --name "ubuntu-2004-cloudinit-template" --memory 2048 --cores 2 --net0 virtio,bridge=vmbr0
sudo qm importdisk 9000 focal-server-cloudimg-amd64.img local-zfs
sudo qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-zfs:vm-9000-disk-0
sudo qm set 9000 --boot c --bootdisk scsi0
sudo qm set 9000 --ide2 local-zfs:cloudinit
sudo qm set 9000 --serial0 socket --vga serial0
sudo qm set 9000 --agent enabled=1

You can start the VM up at this point if you’d like and make any other changes you want because the next step is converting it to a template. If you do boot it, I will be completely honest I have no idea how to log into it. I actually just googled this because I don’t want to leave you without an answer – looks like you can use the same virt-customize we used before to set a root password according to stackoverflow ([https://stackoverflow.com/questions/29137679/login-credentials-of-ubuntu-cloud-server-image](https://stackoverflow.com/questions/29137679/login-credentials-of-ubuntu-cloud-server-image)). Not going to put that into a command window here because cloud-init is really meant for public/private key authentication (see post [here](https://austinsnerdythings.com/2021/04/02/ssh-key-tutorial/) for a quick SSH tutorial).

### #4 – Convert VM to a template

Ok if you made any changes, shut down the VM. If you didn’t boot the VM, that’s perfectly fine also. We need to convert it to a template:

sudo qm template 9000

And now we have a functioning template!

[![screenshot of proxmox ui showing ubuntu 20.04 cloud-init template](https://austinsnerdythings.com/wp-content/uploads/2021/08/ubuntu-cloud-init-template.png)](https://austinsnerdythings.com/wp-content/uploads/2021/08/ubuntu-cloud-init-template.png)

### #5 – Clone the template into a full VM and set some parameters

From this point you can clone the template as much as you want. But, each time you do so it makes sense to set some parameters, namely the SSH keys present in the VM as well as the IP address for the main interface. You could also add the SSH keys with virt-customize but I like doing it here.

First, clone the VM (here we are cloning the template with ID 9000 to a new VM with ID 999):

sudo qm clone 9000 999 --name test-clone-cloud-init

Next, set the SSH keys and IP address:

sudo qm set 999 --sshkey ~/.ssh/id_rsa.pub
sudo qm set 999 --ipconfig0 ip=10.98.1.96/24,gw=10.98.1.1

It’s now ready to start up!

sudo qm start 999

You should be able to log in without any problems (after trusting the SSH fingerprint). **Note that the username is ‘ubuntu’, not whatever the username is for the key you provided.**

ssh [[email protected]](https://austinsnerdythings.com/cdn-cgi/l/email-protection)

Once you’re happy with how things worked, you can stop the VM and clean up the resources:

sudo qm stop 999 && sudo qm destroy 999
rm focal-server-cloudimg-amd64.img

### #6 – automating the process

I have not done so yet, but if you create VMs on a somewhat regular basis, it wouldn’t be hard to stick all of the above into a simple shell script (update 2022-04-19: simple shell script below) and run it via cron on a weekly basis or whatever frequency you prefer. I can’t tell you how many times I make a new VM from whatever .iso I downloaded and the first task is _apt upgrade_ taking forever to run (‘sudo apt update’ –> “176 packages can be upgraded”). Having a nice template always ready to go would solve that issue and would frankly save me a ton of time.

### #6.5 – Shell script to create template

# installing libguestfs-tools only required once, prior to first run
sudo apt update -y
sudo apt install libguestfs-tools -y

# remove existing image in case last execution did not complete successfully
rm focal-server-cloudimg-amd64.img
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
sudo virt-customize -a focal-server-cloudimg-amd64.img --install qemu-guest-agent
sudo qm create 9000 --name "ubuntu-2004-cloudinit-template" --memory 2048 --cores 2 --net0 virtio,bridge=vmbr0
sudo qm importdisk 9000 focal-server-cloudimg-amd64.img local-zfs
sudo qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-zfs:vm-9000-disk-0
sudo qm set 9000 --boot c --bootdisk scsi0
sudo qm set 9000 --ide2 local-zfs:cloudinit
sudo qm set 9000 --serial0 socket --vga serial0
sudo qm set 9000 --agent enabled=1
sudo qm template 9000
rm focal-server-cloudimg-amd64.img
echo "next up, clone VM, then expand the disk"
echo "you also still need to copy ssh keys to the newly cloned VM"

### #7-8 – Using this template with Terraform to automate VM creation

Next post – [How to deploy VMs in Proxmox with Terraform](https://austinsnerdythings.com/2021/09/01/how-to-deploy-vms-in-proxmox-with-terraform/)

### References

[https://matthewkalnins.com/posts/home-lab-setup-part-1-proxmox-cloud-init/](https://matthewkalnins.com/posts/home-lab-setup-part-1-proxmox-cloud-init/)  
[https://registry.terraform.io/modules/sdhibit/cloud-init-vm/proxmox/latest/examples/ubuntu_single_vm](https://registry.terraform.io/modules/sdhibit/cloud-init-vm/proxmox/latest/examples/ubuntu_single_vm)

#### My original notes

https://matthewkalnins.com/posts/home-lab-setup-part-1-proxmox-cloud-init/
https://registry.terraform.io/modules/sdhibit/cloud-init-vm/proxmox/latest/examples/ubuntu_single_vm

# create cloud image VM
wget https://cloud-images.ubuntu.com/focal/20210824/focal-server-cloudimg-amd64.img
sudo qm create 9000 --name "ubuntu-2004-cloudinit-template" --memory 2048 --cores 2 --net0 virtio,bridge=vmbr0

# to install qemu-guest-agent or whatever into the guest image
#sudo apt-get install libguestfs-tools
#virt-customize -a focal-server-cloudimg-amd64.img --install qemu-guest-agent
sudo qm importdisk 9000 focal-server-cloudimg-amd64.img local-zfs
sudo qm set 9000 --scsihw virtio-scsi-pci --scsi0 local-zfs:vm-9000-disk-0
sudo qm set 9000 --boot c --bootdisk scsi0
sudo qm set 9000 --ide2 local-zfs:cloudinit
sudo qm set 9000 --serial0 socket --vga serial0
sudo qm template 9000

# clone cloud image to new VM
sudo qm clone 9000 999 --name test-clone-cloud-init
sudo qm set 999 --sshkey ~/.ssh/id_rsa.pub
sudo qm set 999 --ipconfig0 ip=10.98.1.96/24,gw=10.98.1.1
sudo qm start 999
  
# remove known host because SSH key changed
ssh-keygen -f "/home/austin/.ssh/known_hosts" -R "10.98.1.96"

# ssh in
ssh -i ~/.ssh/id_rsa [[email protected]](https://austinsnerdythings.com/cdn-cgi/l/email-protection)

# stop and destroy VM
sudo qm stop 999 && sudo qm destroy 999

Post Views: 66,513