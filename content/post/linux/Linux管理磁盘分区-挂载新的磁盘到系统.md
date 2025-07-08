---
title: Linux管理磁盘分区-挂载新的磁盘到系统
author: 要名俗气
type: post
date: 2024-12-04T15:04:19+00:00
url: /2024/linux-mount-new-disk
description: 最近购买了两个服务器，其中一个带了一个1T的磁盘，另一个带了一个3T的磁盘，默认这两个磁盘都是没有挂载状态的，需要自己手动进行挂载，于是就开始了研究如何在linux中进行磁盘分区以及磁盘挂载，接下来记录一下折腾过程。 
image: https://images.iminling.com/app/hide.php?key=STRHaU84cVVtSm1LSEZrWHVLdDhqbzZPUURSR0EzWDB5a3RJL2RIdWgwZUVHWTFNQjV6WEtndkhQQUcxb3VlQjNGNzg5eWc9
categories:
  - Linux
tags:
  - fdisk
  - linux
  - parted
---
最近购买了两个服务器，其中一个带了一个1T的磁盘，另一个带了一个3T的磁盘，默认这两个磁盘都是没有挂载状态的，需要自己手动进行挂载，于是就开始了研究如何在linux中进行磁盘分区以及磁盘挂载，接下来记录一下折腾过程。

## 3T盘挂载

### 磁盘查看

先使用fdisk查看磁盘

```bash
root@host:~# fdisk -l
Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 0DA7595A-DE2A-1042-8C4F-3C883C4D7BA6

Device      Start      End  Sectors  Size Type
/dev/sda1  262144 41943006 41680863 19.9G Linux filesystem
/dev/sda14   2048     8191     6144    3M BIOS boot
/dev/sda15   8192   262143   253952  124M EFI System

Partition table entries are not in disk order.

Disk /dev/sdb: 2.93 TiB, 3221225472000 bytes, 6291456000 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

可以看到`/dev/sdb`有一个2.93TB的硬盘，下边来对这个磁盘进行分区并添加。

### 磁盘分区

传统的MBR（主引导记录）分区表只能支持最多2TB的磁盘，而我这个盘是3T的，需要使用GPT，这个格式支持超过2TB的磁盘，Linux支持GPT（GUID分区表）格式。对于大于2T的硬盘使用`parted`命令来进行处理。

首先要确认是否已安装`parted`,如果没有安装使用一下命令进行安装：

```bash
root@host:~# apt install parted
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
```

然后执行`parted /dev/sdb`对`/dev/sdb`这个盘进行处理，输入命令后会进入命令模式，需要经过几个步骤

  1. `mklabel gpt` ，<span class="hljs-comment">创建GPT分区表</span>
  2. `mkpart primary ext4 0% 100%` <span class="hljs-comment">，创建一个新分区，使用100%的磁盘空间</span>
  3. `print` ，显示设置的分区(可选操作)
  4. `quit` ，退出

完整的操作命令如下：

```bash
root@host:~# parted /dev/sdb
GNU Parted 3.5
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt
(parted) mkpart primary ext4 0% 100%
(parted) print
Model: QEMU QEMU HARDDISK (scsi)
Disk /dev/sdb: 3221GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  3221GB  3221GB  ext4         primary

(parted) quit
Information: You may need to update /etc/fstab.
```

分区完成。

### 磁盘格式化

创建分区后，需要格式化为文件系统。我这里使用`ext4`格式，因为我只分了一个盘，所以这里是sdb1,如果有多个盘，需要分别对每个进行格式化：

```bash
root@host:~# mkfs.ext4 /dev/sdb1
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done
Creating filesystem with 786431488 4k blocks and 196608000 inodes
Filesystem UUID: f2275364-6fa6-4ae0-a214-5bd98552f5b1
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
        102400000, 214990848, 512000000, 550731776, 644972544

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
```

经过上边就已经对刚分的那个盘进行了格式化。

### 磁盘挂载

接下来就是对这个盘进行挂载，我这里挂载到/data目录下，需要先创建/data这个目录，然后通过`mount /dev/sdb1 /data`命令，把/dev/sdb1挂载到/data目录：

```bash
root@host:~# mkdir /data
root@host:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            979M     0  979M   0% /dev
tmpfs           198M  732K  198M   1% /run
/dev/sda1        20G  2.1G   17G  12% /
tmpfs           990M     0  990M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/sda15      124M   12M  113M  10% /boot/efi
tmpfs           198M     0  198M   0% /run/user/0
root@host:~# mount /dev/sdb1 /data
root@host:~# df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            979M     0  979M   0% /dev
tmpfs           198M  732K  198M   1% /run
/dev/sda1        20G  2.1G   17G  12% /
tmpfs           990M     0  990M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/sda15      124M   12M  113M  10% /boot/efi
tmpfs           198M     0  198M   0% /run/user/0
/dev/sdb1       2.9T   28K  2.8T   1% /data
```

可以通过命令df -h查看磁盘挂载情况，如上是已经成功挂载了。

### 磁盘开机自动挂载

经过上面的挂载后，如果系统重启，那么挂载就会失效，所以如果你希望磁盘在系统重启时自动挂载，可以编辑`/etc/fstab`文件：

```bash
root@host-c:~# vim /etc/fstab
/dev/sdb1   /data   ext4   defaults   0   2
```

如上添加一行`/dev/sdb1   /data   ext4   defaults   0   2`就可以重启后自动实现挂载。

## 1T盘挂载

在3t盘挂载的教程中可以看出整个挂载过程分为：`磁盘查看-磁盘分区-磁盘格式化-磁盘挂载-磁盘开机自动挂载`共5个步骤，处理1T盘过程基本一致，只是在`磁盘分区`的时候有两种选择，第一种就还是使用到parted命令，另一种是使用fdisk命令。

先来查看磁盘的分布

```bash
root@host:~# fdisk -l
Disk /dev/sdb: 1000 GiB, 1073741824000 bytes, 2097152000 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
Disk model: QEMU HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 0DA7595A-DE2A-1042-8C4F-3C883C4D7BA6

Device      Start      End  Sectors  Size Type
/dev/sda1  262144 41943006 41680863 19.9G Linux filesystem
/dev/sda14   2048     8191     6144    3M BIOS boot
/dev/sda15   8192   262143   253952  124M EFI System

Partition table entries are not in disk order.
```

可以看到有一个`/dev/sdb: 1000 GiB`的磁盘。

### fdisk命令

先来介绍一下fdisk命令，输入/dev/sdb开始进行分区，默认是创建MBR分区，下边每个`Command (m for help)`都是需要进行输入的地方。首先是输入`o`,创建一个MBR分区，然后是输入`n`来创建分区，`partition type`选择主分区`p`，分区号码我这里只需要一个分区，所以输入1，然后就是`first sector`和`last sector`默认直接回车就是整个盘。最后输入`w`来写入修改，就完成了分区。

```bash
root@host:~# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.38.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS (MBR) disklabel with disk identifier 0x17afed0e.

Command (m for help): o
Created a new DOS (MBR) disklabel with disk identifier 0xa7272020.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-2097151999, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2097151999, default 2097151999):

Created a new partition 1 of type 'Linux' and of size 1000 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

然后就是对新建的分区进行格式化和挂载了。和3T盘操作一样。

### parted命令

parted不仅可以处理大于2T的，小于2T的也是可以处理的，下边就来看看具体怎么处理。

```bash
root@host:~# parted /dev/sdb
GNU Parted 3.5
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel msdos
(parted) mkpart primary ext4 0% 100%
(parted) quit
Information: You may need to update /etc/fstab.
```

唯一的区别就是mklable命令，原来是gpt（GPT分区），现在改成msdos（MBR分区），其他命令都一样。分区完成后进行格式化，挂载等一系列操作。

以上就是针对小于2T以及大于2T的盘的挂载教程，希望可以帮到大家。