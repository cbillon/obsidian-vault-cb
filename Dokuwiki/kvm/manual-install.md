====== Use Ubuntu Cloud Image with KVM ======

[[https://medium.com/@art.vasilyev/use-ubuntu-cloud-image-with-kvm-1f28c19f82f8]]

In this tutorial, we are going to create a virtual machine based on the Ubuntu cloud image. We will go through such steps as preparing image, cloud-init configuration and attaching logical volumes.

==== Preparation ====

Install required system packages:

  sudo apt install qemu-kvm libvirt-bin qemu-utils genisoimage virtinst

Download Ubuntu cloud image:

 wget https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img

Verify that image format is QCOW2:

  qemu-img info xenial-server-cloudimg-amd64-disk1.img

  image: xenial-server-cloudimg-amd64-disk1.img
  file format: qcow2
  virtual size: 2.2G (2361393152 bytes)
  disk size: 284M
  cluster_size: 65536
  Format specific information:
    compat: 0.10
    refcount bits: 16

Create directory for base images:

   sudo mkdir /var/lib/libvirt/images/base

Move downloaded image into this folder:

  sudo mv xenial-server-cloudimg-amd64-disk1.img /var/lib/libvirt/images/base/ubuntu-16.04.qcow2

==== Virtual machine image ====

Create directory for our instance images:

  sudo mkdir /var/lib/libvirt/images/instance-1

Create a disk image based on the Ubuntu image:

  sudo qemu-img create -f qcow2 -F qcow2 -o backing_file=/var/lib/libvirt/images/base/ubuntu-16.04.qcow2 
  /var/lib/libvirt/images/instance-1/instance-1.qcow2

Let’s take a look at the image:

  sudo qemu-img info /var/lib/libvirt/images/instance-1/instance-1.qcow2

  image: /var/lib/libvirt/images/instance-1/instance-1.qcow2
  file format: qcow2
  virtual size: 2.2G (2361393152 bytes)
  disk size: 196K
  cluster_size: 65536
  backing file: /var/lib/libvirt/images/base/ubuntu-16.04.qcow2
  Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false

Current virtual size is 2.2 GB, let’s set it to 5 GB:

  sudo qemu-img resize /var/lib/libvirt/images/instance-1/instance-1.qcow2 5G

  Image resized.

  sudo qemu-img info /var/lib/libvirt/images/instance-1/instance-1.qcow2
  
  image: /var/lib/libvirt/images/instance-1/instance-1.qcow2
  file format: qcow2
  virtual size: 5.0G (5368709120 bytes)
  disk size: 200K
  cluster_size: 65536
  backing file: /var/lib/libvirt/images/base/ubuntu-16.04.qcow2
  Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false

Cloud-Init will resize partitions to fill the available disk space on first boot.

==== Cloud-Init Configuration ====

Now we are going to create configs to make Cloud-Init do following:

  * create new user
  * configure SSH access by a public key
  * Create meta-data:

  cat >meta-data <<EOF
  local-hostname: instance-1
  EOF

Read public key into environment variable:

  export PUB_KEY=$(cat ~/.ssh/id_rsa.pub)

Create user-data:

  cat >user-data <<EOF
  #cloud-config
  users:
    - name: ubuntu
      ssh-authorized-keys:
        - $PUB_KEY
      sudo: ['ALL=(ALL) NOPASSWD:ALL']
      groups: sudo
      shell: /bin/bash
  runcmd:
    - echo "AllowUsers ubuntu" >> /etc/ssh/sshd_config
    - restart ssh
  EOF

Create a disk to attach with Cloud-Init configuration:

  sudo genisoimage  -output /var/lib/libvirt/images/instance-1/instance-1-cidata.iso -volid cidata -joliet 
  -rock user-data meta-data

==== Launch virtual machine ====

Start the virtual machine with two disks attached: 
  * instance-1.qcow2 as root disk 
  * instance-1-cidata.iso as disk with Cloud-Init configuration.

   virt-install --connect qemu:///system --virt-type kvm --name instance-1 --ram 1024 --vcpus=1 --os-type 
   linux --os-variant ubuntu16.04 --disk path=/var/lib/libvirt/images/instance-1/instance- 
   1.qcow2,format=qcow2 --disk /var/lib/libvirt/images/instance-1/instance-1-cidata.iso,device=cdrom -- 
   import --network network=default --noautoconsole

  Starting install...
  Domain creation completed.
  
Make sure the virtual machine is running:

   sudo virsh list
   Id    Name                           State
   ----------------------------------------------------
   6     instance-1                     running

Get the IP address:

   sudo virsh domifaddr instance-1
   Name       MAC address          Protocol     Address
   ----------------------------------------------------------------
   vnet0      52:54:00:1b:3b:4f    ipv4         192.168.122.201/24

Connect to the instance by the public key:

   ssh ubuntu@192.168.122.201
  Warning: Permanently added '192.168.122.201' (ECDSA) to the list of known hosts.
  Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-150-generic x86_64)
  ubuntu@instance-1:~$

If the virtual machine is not accessible, you may mount instance image and inspect it locally:

   sudo guestmount -a /var/lib/libvirt/images/instance-1.img -m /dev/sda1 /mnt

To manage the virtual machine use virsh command:

  #shutdown
  sudo virsh shutdown instance-name
  #reboot
  sudo virsh reboot instance-name
  #start
  sudo virsh start instance-name
  #remove
  sudo virsh undefine instance-name
  
==== Attaching Logical Volumes ====

To test attachment let’s create fake 5 GB disk image:

   dd if=/dev/zero of=disk.img bs=1 count=1 seek=5G

Find free loop device:

   sudo losetup -f
  /dev/loop11

Attach the image as a disk device:

   sudo losetup /dev/loop11 disk.img
   Fake device is ready, next we are going to create PV (physical volume) and VG (volume group).

   sudo pvcreate /dev/loop11
   Physical volume "/dev/loop11" successfully created.
   sudo vgcreate cloud /dev/loop11
   Volume group "cloud" successfully created

Create logical volume:

   sudo lvcreate --name lv1 --size 1G cloud
   Logical volume "lv1" created.

Finally, attach new device:

  sudo virsh attach-disk instance-1 /dev/cloud/lv1 vdb --config

config flag is used to write attachment configuration into the instance configuration file, otherwise, disk will not be attached after next reboot.

Connect to the virtual machine and check that new device is available:

   ssh ubuntu@192.168.122.127 ls /dev/vdb
   /dev/vdb