---
tags:
  - docker
  - nfs
---

## Mount NFS Shares Inside Docker Container 

[site](https://www.baeldung.com/linux/docker-mount-nfs-shares)

## 1. Overview 
Network File System (NFS) enables remote access to filesystems over a shared network. Hence, it’s suitable for scenarios wherein multiple Docker containers require access to common files, such as containerized microservices having identical configuration files.

This tutorial demonstrates different ways to mount NFS shares inside a Docker container. We’ll connect to a pre-configured NFS server through the Docker host, which acts as an NFS client.

The commands in this tutorial were run on CentOS 7 in Bash after installing nfs-utils, or nfs-common in some distros, and Docker Engine.

## 2. Docker Container Storage and NFS Shares

Docker container files don’t persist after removing the container. Nonetheless, we can mount external filesystems onto a container.
The image below highlights the mounting methods covered in upcoming sections:




A container supports three types of storage mounts: 
  - bind mounts, 
  - volumes, 
  - tmpfs. 
However, tmpfs is a volatile memory storage. Hence, it isn’t viable for persistent external filesystems like NFS. On the other hand, we can mount such filesystems via both bind mounts and volumes. Furthermore, we can directly mount a filesystem through a running container’s terminal under certain conditions.

### 2.1. Checking the NFS Connection

A container connects to the NFS server through its host machine’s IP address. Therefore, the host’s IP address must be included in the NFS exports. Also, the server firewall might require additional configuration to permit this NFS client.

The NFS server for this tutorial has the IP address 32.55.178.8. It’s important to replace this address with the actual server address when using this tutorial’s scripts.

The server’s /etc/exports file exposes the file share path /opt/nfsserver/shared to the host having the IP address 32.93.10.15:
/opt/nfsserver/shared 32.93.10.15(no_root_squash,rw,sync)
Copy
Let’s create a directory named /mnt/testnfsdir on the host machine. Then, we’ll check the NFS connection by mounting the file share using the mount command:

$ sudo mkdir /mnt/testnfsdir
$ sudo mount -t nfs -o nfsvers=4 32.55.178.8:/opt/nfsserver/shared /mnt/testnfsdir
Copy
Here, the absence of an error output indicates success. In the case of NFS version 3, we can change nfsvers=4 to nfsvers=3.

Let’s now create a file named b0.txt inside that directory using touch, and then run ls:

$ sudo touch /mnt/testnfsdir/b0.txt
$ ls /mnt/testnfsdir
b0.txt
Copy
We’ve now validated that the host can access the file share.

dans notre cas sur OVH

[site](https://dokuwiki.cbillon.ovh/doku.php?id=nfs)
### 3. Mounting Directly Inside the Container

Docker doesn’t support running mount within a container’s terminal unless we assign some additional privileges — for example, by supplying the –privileged flag.

Let’s start an Alpine Linux image container named priv with an interactive terminal by passing the options –interactive (-i) and –tty (-t) to the docker run command:

   sudo docker run -it --privileged --name priv alpine:latest

Now, we can create a directory named /mnt/shared and run the mount command:

   mkdir /mnt/shared
   mount -t nfs -o nfsvers=4 32.55.178.8:/opt/nfsserver/shared /mnt/shared


Next, we can test whether we’re able to read from and write to the NFS files. Let’s create a file named b1.txt inside /mnt/shared in the container:

  ls /mnt/shared

  touch /mnt/shared/b1.txt
  ls /mnt/shared

  
We successfully mounted the file share directly inside the container. However, this isn’t the best practice, as it requires running a privileged container and is less configurable. In fact, without the –privileged flag, the mount command prints an error:

  sudo docker run -it --name unpriv alpine:latest
  mkdir /mnt/shared
  mount -t nfs -o nfsvers=4 32.55.178.8:/opt/nfsserver/shared /mnt/shared
  mount: permission denied (are you root?)
  whoami
  root
  exit

Despite running the command as the root user, we got a permission-denied error.


## 4.1. Using docker run

docker run is convenient for running a single container with fewer configuration options. Let’s pass the source path as /mnt/testnfsdir. Then, we can provide the target path as /mnt/shared inside the container where the bind mount gets mapped.

Let’s run a container named bindmount1 and verify the setup:

  sudo docker run -it --name bindmount1 \
  --mount type=bind,source=/mnt/testnfsdir,target=/mnt/shared \
  alpine:latest
  touch /mnt/shared/b2.txt
  ls /mnt/shared
  exit

As shown here, we successfully created b2.txt in the file share.

## 4.2. Using docker compose

Let’s create a file named docker-compose.yml and specify the bind mount under volumes:


  services:
      bindmount2:
          image: alpine:latest
          container_name: bindmount2
          volumes:
              - "/mnt/testnfsdir:/mnt/shared:rw"
          tty: true

Let’s bring up the service using docker compose up with the –detach (-d) flag:

    sudo docker compose up -d

   ✔ Network baeldung_default    Created    0.2s
   ✔ Container bindmount2    Started    0.0s

Here, we can replace docker compose with docker-compose for devices using Docker Compose V1.

Let’s now open this container’s terminal by passing the -it parameters along with sh to docker exec. Then, let’s create a new file named b3.txt:

   sudo docker exec -it bindmount2 sh
   touch /mnt/shared/b3.txt
   ls /mnt/shared
   exit

Having successfully created the file, we can now bring down the service using docker compose down:


   sudo docker compose down

   ✔ Container bindmount2    Removed    10.2s
   ✔ Network baeldung_default    Removed    0.2s
   
Thus, we’ve shown that we can mount file shares as bind mounts. However, this creates a dependency on the host filesystem setup. The bind mount source directory should be connected to the file share before we create any  container

## 5. Mounting as a Volume


Docker manages volumes in isolation from other files on the host. Therefore, volumes are preferable over bind mounts. Docker volume support for NFS shares is available starting with docker v17.06.

First, let’s create a Docker volume by providing the NFS details to the docker volume create command. The volume connects to NFS only when mounted to a container. Let’s name the volume nfsvolume1:

    sudo docker volume create --driver local \
    -o type=nfs \
    -o o=addr="32.55.178.8,rw,nfsvers=4" \
    -o device=:/opt/nfsserver/shared \
    nfsvolume1
 


Copy
We can inspect the volume using docker volume inspect:

$ sudo docker volume inspect nfsvolume1
[
    {
        "CreatedAt": "2023-10-14T00:01:19Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/nfsvolume1/_data",
        "Name": "nfsvolume1",
        "Options": {
            "device": ":/opt/nfsserver/shared",
            "o": "addr=32.55.178.8,rw,nfsvers=4",
            "type": "nfs"
        },
        "Scope": "local"
    }
]
Copy
As shown above, the volume has the local driver and the NFS mount options.

### 5.1. Using docker run With an External Volume
Let’s spin up a new container named volumecont1 using docker run and specify the volume as a source to the –mount parameter:

$ sudo docker run -it --name volumecont1 \
  --mount type=volume,source=nfsvolume1,target=/mnt/shared \
  alpine:latest
/ # ls /mnt/shared
b0.txt b1.txt b2.txt b3.txt
/ # touch /mnt/shared/b4.txt
/ # ls /mnt/shared
b0.txt b1.txt b2.txt b3.txt b4.txt
/ # exit
Copy
The file creation operation was successful.

### 5.2. Using docker run to Create a Volume
We can let Docker create the volume automatically by providing a new name along with the necessary volume options as volume-opt to the –mount parameter:


