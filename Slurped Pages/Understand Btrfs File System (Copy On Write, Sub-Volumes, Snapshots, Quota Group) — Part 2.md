---
link: https://surajsinghbisht054.medium.com/understand-btrfs-file-system-copy-on-write-sub-volumes-snapshots-quota-group-part-2-2b60209c5296
byline: Suraj Singh Bisht
site: Medium
date: 2023-11-10T12:07
excerpt: Understand Btrfs File System (Copy On Write, Sub-Volumes, Snapshots, Quota Group) — Part 2 Btrfs is an advanced file system that uses copy-on-write techniques, sub-volumes, snapshots, compression …
twitter: https://twitter.com/@Medium
slurped: 2026-06-12T15:25
title: Understand Btrfs File System (Copy On Write, Sub-Volumes, Snapshots, Quota Group) — Part 2
tags:
  - btrfs
---

Btrfs is an advanced file system that uses copy-on-write techniques, sub-volumes, snapshots, compression, and quota groups to provides impressive capabilities and reliability to store and manage data.

Press enter or click to view image in full size

Photo by [Nick van der Ende](https://unsplash.com/@nkend?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

Hello There, I hope all you guys are doing well. This serves as the continuation of our series. Before delving into this part, please make sure you are already aware of all the concepts explained in first part. if not, You can read first part here. [https://medium.com/@surajsinghbisht054/understand-btrfs-file-system-copy-on-write-sub-volumes-snapshots-quota-group-part-1-c6305f87df9b](https://medium.com/@surajsinghbisht054/understand-btrfs-file-system-copy-on-write-sub-volumes-snapshots-quota-group-part-1-c6305f87df9b)

In our first part of this series, We covered.

- **What is a file system?**
- Btrfs File System?
- Copy-On-Write?
- Sub-Volumes?
- Snapshots?
- Quota Group?

In this part, we will cover

- Creation, management, and deletion of sub-volumes
- Snapshot implementation
- Copy-on-write (COW) management
- Utilization of Quota Group

I will highly recommend to try all these commands on a test environment if you are not entirely clear what we are doing here.

Few Useful Command

  
# Use the btrfs command to manage and display information about a   
# Btrfs file system. The command requires a subcommand.   
# Enter btrfs without any arguments to list the subcommands  
btrfs

# show file system  
btrfs filesystem show

# create new subvolume  
btrfs subvolume create SUBVOLUME_NAME

# list subvolumes  
btrfs subvolume list PATH

# show subvolumes details  
btrfs subvolume show PATH

# Take snapshot of subvolume  
btrfs subvolume snapshot SOURCE_PATH DEST_PATH

# Get the usages details  
compsize ./PATH

# delete subvolume  
btrfs subvolume delete PATH

# To get the error stats  
btrfs device stats /

# list file attributes  
lsattr

# disable COW  
chattr +C PATH

# btrfs disk balancing  
btrfs balance start --bg /  
btrfs balance status /

## Sub-Volume & Snapshots

Let us start by creating a simple sub-volume named **demo**. In the Btrfs filesystem, a sub-volume functions similarly to a directory, as most commands applicable to a standard directory are also valid for a sub-volume.

To create a sub-volume, create a folder.

> **_mkdir demo-snapshot_**

My Terminal (Fedora)

> **_cd demo-snapshot/_**
> 
> **_sudo btrfs subvolume create demo_**

Press enter or click to view image in full size

My Terminal (Fedora)

As we can see in above given screenshot of terminal. demo directory is the newly created sub-volume but as i already mentioned above, filesystem will handle it like a directory.

By default, newly generated sub-volumes are under the ownership of the root. Altering the ownership of this sub-volume is straightforward using the “chown” tool, similar to managing ownership for any other file or folder.

> **_chown -R user_name:group_name path_to_folder_**

Press enter or click to view image in full size

My Terminal (Fedora)

The demo sub-volume is ready to go. To grasp how snapshots work, add some files and folders to the demo sub-volume.

> cd
> 
> touch first
> 
> mkdir -p second/third
> 
> touch second/third/fourth.txt
> 
> tree

Press enter or click to view image in full size

My Terminal (Fedora)

I’ve created two txt files and two nested directories, as shown in the screenshot above. The idea is to make a snapshot of this current directory structure. Afterward, we’ll modify the structure to test if the snapshot retains the original state while the main sub-volume reflects the latest changes.

This concept is similar to a git-based filesystem. Snapshots capture the entire filesystem at a specific point, preserving files and their data. We can update the original files as usual, and the snapshot keeps track of the data at the time it was taken.

**How does this differ from the traditional copy & paste method?**

In Btrfs, it uses Copy-On-Write (COW). Every file update is stored in a separate block, and the original file pointer points to this updated data block. So, in a snapshot, there’s no actual copying of files or folders; instead, we freeze the state of the pointers pointing to the data blocks.

For example, if we have 10 GB of data in a sub-volume and create a snapshot, it might seem like a copy-paste, but the total size consumed remains 10 GB. Updating the original sub-volume only takes up space for the newly updated data, not duplicating the entire content.

Now, let’s create a snapshot of the “**demo**” directory and name it “**chkpoint-one.**”

This snapshot will capture the current directory structure and data of the demo sub-volume, preserving its state for future reference.

> sudo btrfs subvolume snapshot demo chkpoint-one

Press enter or click to view image in full size

My Terminal (Fedora)

Alright, looking at the terminal output, the snapshot has been successfully created and is accessible like a regular directory.

Now, let’s proceed with some updates to the “demo” directory. I’ll be adding two new files to this directory.

> touch demo/second/third/fifth.txt
> 
> touch demo/second/third/sixth.txt
> 
> tree

Press enter or click to view image in full size

My Terminal (Fedora)

Observing the terminal output, it’s clear that the “chkpoint-one” directory, being a snapshot of the “demo” directory, remains unchanged even after updating the original “demo” directory.

So, This is a easiest example to understand the snapshot concept. Now, As I already told you, It may seems like we have duplicated complete demo directory but under the hood that snapshot is not occupying any new space at this stage except the meta data of the new snapshot.

**Still in doubt?**

Let try this same routine with a video file to see, if we duplicates it. Will It occupy the size Or not.

Create a new sub-volume videos. change its ownership to regular user. go inside the videos directory. Copy any video file Or any other big size file. In my case, the size of my sample.mp4 file is 39MB.

Copied It inside videos directory

Press enter or click to view image in full size

My Terminal (Fedora)

Now create snapshot of videos directory and named it video-snapshot.

Press enter or click to view image in full size

My Terminal (Fedora)

Now look at the total size occupied for demo-snapshot.

> sudo compsize ./demo-snapshot/

Press enter or click to view image in full size

My Terminal (Fedora)

Did you noticed there are three columns for size

1. **Disk Usages** : Actual Size Occupied to store the data
2. **Uncompressed**: Uncompressed size (if compression applied during the sub-volume creation)
3. **Referenced :** Subvolumen + Snapshot size (39*2=78MB)

So, as we can see we created snapshot of video file that have 38.* MB of data but It occupied space for only one time even after creating the snapshot of it.

I hope now it’s clear how it works.

**To delete a snapshot, Or Sub-Volume**

> sudo btrfs subvolume delete chkpoint-one/

Press enter or click to view image in full size

My Terminal (Fedora)

To make a new snapshot readonly.

Press enter or click to view image in full size

My Terminal (Fedora)

I took a new snapshot of a directory called “videos” with the read-only flag enabled. After that, I attempted to create a new file within the newly created “video-snapshot-readonly” directory to check if an error would occur. Indeed, an error was thrown when attempting to modify the snapshot.

Press enter or click to view image in full size

My Terminal (Fedora)

And disk usages is still same.

Press enter or click to view image in full size

My Terminal (Fedora)

**Note :** It might be tempting to assume that by taking a sequence of Btrfs snapshots on your local PC, you’ve established a robust backup plan. However, this is not the case. If the foundational data, shared by Btrfs subvolumes, gets unintentionally corrupted (due to external factors like cosmic rays beyond Btrfs’ control), all subvolumes linked to this data will harbor the same error.

To transform these snapshots into genuine backups, store them on a separate Btrfs filesystem, like an external drive.

## Send/receive

Send and receive in Btrfs are complementary functions facilitating data transfer between filesystems in a streamable format.

Put simply, if two or more physically distinct disks (both utilizing the Btrfs system) are connected to a single system and share a snapshot in their space, when changes are made to the base image, only the updated blocks (due to Copy-on-Write) are transferred to the new disk to maintain directory synchronization. While this concept is beyond the scope of this article, if you're interested in more details, you can refer to the Btrfs documentation.

You might be wondering why I introduced Send/Receive if it’s beyond the scope of this article. The reason is, although we created a snapshot of the base sub-volume, we didn’t discuss what to do if we want to restore a sub-volume from a snapshot. To restore the sub-volume from a snapshot, we have two options:

1. Directly use the snapshot as the main directory.  
2. Use the send/receive utility to restore the base image into the snapshot state.

sudo btrfs send SOURCE_SNAPSHOT | sudo btrfs receive RECEIVING_SUBVOLUME

## Enable / Disable COW

The Copy-On-Write (CoW) option is a double-edged sword. While it can help mitigate data corruption risks, enabling it on files with frequent random writes can lead to heavy fragmentation, causing issues like trashing on HDDs or significant multi-second spikes in CPU load on systems with SSDs or large RAM. Over time, it can gradually slow down and fill up your Btrfs filesystem.

It’s worth noting that regular balancing of the Btrfs tree can address high fragmentation. Instead of disabling CoW for the entire filesystem, you can selectively disable it for specific directories, such as those used for database storage or VM storage.

You can read more about balancing the btrfs here.

**A crucial warning**: Disabling CoW in Btrfs also turns off checksums, rendering Btrfs unable to detect corruption in nodatacow files.

To check if any specific attribute flag is enabled or not?

Press enter or click to view image in full size

My Terminal (Fedora)

> lsattr

No flag is enable, hence we can conclude that COW is enable for all these directories.

## Disabling COW

To disable copy-on-write for newly created files in a mounted subvolume, use the `nodatacow` mount option. This will only affect newly created files. Copy-on-write will still happen for existing files. The `nodatacow` option also disables compression

To disable it on a new directory or file, use “chattr +C “.

Setting chattr +C on a directory disables COW on all NEWLY created child directories and files.

> chattr +C ./videos

Press enter or click to view image in full size

My Terminal (Fedora)

To enable COW

> Note from chattr man Page:
> 
> A file with the ‘C’ attribute set will not be subject to copy-on-write updates. This flag is only supported on file systems which perform copy-on-write.
> 
> (Note: For btrfs, the ‘C’ flag should be set on new or empty files. If it is set on a file which already has data blocks, it is undefined when the blocks assigned to the file will be fully stable. If the ‘C’ flag is set on a directory, it will have no effect on the direc‐ tory, but new files created in that directory will have the No_COW attribute set. If the ‘C’ flag is set, then the ‘c’ flag cannot be set.)

## Deduplication

With the use of copy-on-write, Btrfs can duplicate files or entire subvolumes without physically copying the data. However, when a file is modified, a new actual copy is generated. Deduplication extends this process by actively recognizing blocks of data that share common sequences and consolidating them into an extent with the same copy-on-write principles.

While this topic is beyond the scope of this article, you can find more detailed information here.

## Quota Group

Btrfs subvolume quota introduces the idea of quota groups (qgroups) to address these challenges. Qgroups are organized in a hierarchy and can be used to limit both total and exclusive referenced space for subvolumes. Referenced space is the data accessible from any subvolume within the qgroup, while exclusive space is the data accessible only within that qgroup.

To demonstrate the use of Quota, I will enable QGroup for the “video” directory. Btrfs utilizes QGroup to enforce quota restrictions. Initially, the “video” directory will contain only one video file of 38 MB when the maximum size limit of 50 MB is applied as a quota.

After setting the max size limit to 50 MB, I will intentionally move another video file to the “video” directory to exceed the quota limit. This action will cause the directory size to surpass the set limit. Subsequently, I will attempt to add a single txt file, and the system should throw an error indicating that the file size exceeds the limit and block the operation.

Note: By default, QGroup is disabled for all directories. To enable it, use the provided command:

> sudo btrfs quota enable ./video
> 
> sudo btrfs qgroup show ./ -r  
> sudo btrfs quota rescan ./videos/
> 
> sudo btrfs qgroup limit 50M ./videos/

Press enter or click to view image in full size

My Terminal (Fedora)

Error. Quota exceeds error.

You can read more about here

## Conclusion

In conclusion, this article delved into various aspects of Btrfs, covering topics such as snapshot creation, the impact of Copy-On-Write, the utility of send/receive functions, and the use of Quota and QGroup for enforcing size restrictions.

Btrfs offers powerful features like snapshots for data protection, though it’s essential to be mindful of potential challenges, such as fragmentation when using Copy-On-Write. The send/receive functionality facilitates efficient data transfer, and Quota/QGroup provides a means to control storage usage.

I encourage you to share your thoughts and experiences. If you have questions, suggestions, or feedback, feel free to leave a comment below.

Your insights are valuable in fostering a collaborative learning environment.