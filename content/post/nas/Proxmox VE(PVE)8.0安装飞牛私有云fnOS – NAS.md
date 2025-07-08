---
title: Proxmox VE(PVE)8.0安装飞牛私有云fnOS – NAS
author: 要名俗气
type: post
date: 2025-02-09T03:15:47+00:00
url: /2025/install-fnos-nas
description: 飞牛nas系统出来也有段时间了，一个很不错的nas系统，在以前的文章中也介绍过群晖nas系统的安装，今天就来一次尝鲜，安装一下fnOS，看一下群晖的系统和飞牛的系统有什么区别。接下来就开始安装折腾之路 - 飞牛私有云fnOS nas安装。
image: https://images.iminling.com/i/2025/01/08/d3c94bd7539929c05abffe55a8295ea2.webp
categories:
  - nas
tags:
  - FnOS
  - nas
  - pve
---
[飞牛nas](https://www.fnnas.com)系统出来也有段时间了，一个很不错的nas系统，在以前的文章中也介绍过[群晖nas系统的安装]({{< ref "/post/linux/pve/Proxmox VE(PVE)8.0安装黑群晖NAS并直通硬盘.md" >}})，今天就来一次尝鲜，安装一下fnOS，看一下群晖的系统和飞牛的系统有什么区别。接下来就开始安装折腾之路 - 飞牛私有云fnOS nas安装。

## PVE配置

首先要先去[飞牛官网](https://www.fnnas.com/download)把官方镜像ISO文件给下载下来，上传到pve中，这个还有不清楚的可以参考[Proxmox VE(PVE)8.0安装爱快ikuai虚拟机并直通网卡]({{< ref "/post/linux/pve/Proxmox VE(PVE)8.0安装爱快ikuai虚拟机并直通网卡.md" >}})中对爱快ISO文件上传的步骤。

### 常规

修改为自己想要的vmid 以及 名称，我这里名称填了103，可以修改自己喜欢的，比如`飞牛NAS`等。其他默认。

[![fnos common](https://images.iminling.com/i/2025/01/08/37837647596f9bb193eb5478ce1cde0f.webp)][5]

### 操作系统

操作系统选择在飞牛官网下载的iso文件。其他的默认

[![fnos os](https://images.iminling.com/i/2025/01/08/e4de0f82014a774996efa5f6568c0acf.webp)][6]

### 系统

机型这里选择q35，其他不变。

[![fnos system](https://images.iminling.com/i/2025/01/08/ffa5be3d705f4d6fae78b707bf0cc037.webp)][7]

### 磁盘

这里没有截图到，实际是和安装群群一样，把磁盘删除了，什么都不留。如果只是想体验则可以根据实际情况来分配硬盘大小。

我这里是要把一个硬盘直通给飞牛的，所以我这里就把硬盘都给删除了 。

如果只是用来体验，则可以给一个30G的磁盘。

### cpu

cpu核心根据自己的情况来，我的J4125只有4个核心，所以全给了。类别选host。

[![fnos cpu](https://images.iminling.com/i/2025/01/08/e0272ce01b4a02382119f02e9df93e3e.webp)][8]

### 内存配置

内存根据自己的需求来给，建议2G以上，我这里给4G.

[![fnos ram](https://images.iminling.com/i/2025/01/08/c2ec1daba8630f17c9a006afa1a9d040.webp)][9]

### 网络

网络我这里直接默认，使用pve的网桥`vmbr0`就可以了。

[![fnos network](https://images.iminling.com/i/2025/01/08/42a35339b69c2328ebeff937053706d2.webp)][10]

### 确认

都设置完成后对已设置的信息进行确认，基本不会有什么问题，完成。

[![fnos confirm](https://images.iminling.com/i/2025/01/08/c2f60e6ebc7d2de5314c6312dd7b1f52.webp)][11]

### 硬盘直通

经过上边设置后，需要再给nas系统一个磁盘，可以参考我在[Proxmox VE(PVE)8.0安装黑群晖NAS并直通硬盘](https://www.iminling.com/2024/pve-install-nas "Proxmox VE(PVE)8.0安装黑群晖NAS并直通硬盘")时的直通操作，基本是一样的。

硬盘直通后，要看一下启动顺序，需要先从我们上传的iso镜像启动，如果没有问题就可以启动了。

## FnOS安装

经过上边的步骤，已经创建好了一个虚拟机。接下来就是对系统进行配置启动了。点击控制台查看启动。默认图形安装。

![fnos boot](https://images.iminling.com/i/2025/01/08/366271dd2ffa0ef1821b8e3b86440a6c.webp)

选择安装位置，本文这里是选择直通的硬盘：

![fnos disk select](https://images.iminling.com/i/2025/01/08/e5ee329267cbe5bd4aae8fe6a258267a.webp)

下一步选择系统分区大小,官方默认给的是64G，这里直接按官方建议来：

![fnos system disk](https://images.iminling.com/i/2025/01/08/226fe70c13e5ab86147589ed6746b766.webp)

接下来就是安装了，等待安装成功就可以了。

![fnos setup complete](https://images.iminling.com/i/2025/01/08/cf81cf188faa745ddfe2c241b150d05e.webp)

下一步就会出现重启了，重启后控制台会显示一下内容：

![fnos login](https://images.iminling.com/i/2025/01/08/a09fdf42f173d00e8a376c07e7bb4305.webp)

会告诉登录的ip和端口，这里是`http://192.168.1.47:5666`,等下来浏览器直接输入这个地址来进行访问。

## 初始化FnOS

### 系统设置

接下来对fnos进行初始化，访问上边的地址：

![fnos setup](https://images.iminling.com/i/2025/01/08/d3c94bd7539929c05abffe55a8295ea2.webp)

初次进入后需要设置设备名称以及管理员信息。

![fnos setting](https://images.iminling.com/i/2025/01/08/e6061080b1049a0751792f173238d904.webp)

### 存储空间创建

设置后会进入系统，因为是第一次进入系统，系统会提示创建存储空间：

![fnos disk](https://images.iminling.com/i/2025/01/08/b312decdfb16d2338ac395e0f5485d23.webp)

点击立即创建，选择磁盘以及数据保护默认，我这里直接使用默认basic方式。

![fnos disk select ](https://images.iminling.com/i/2025/01/08/63391e0b3418e5fbf3be4f6819c88b68.webp)

下一步设置存储空间可以使用的用户以及可以使用的大小，我这里只有自己使用，容量不限制：

![fnos create disk](https://images.iminling.com/i/2025/01/08/e10b83abed66f0285f19eb6dfb5f1a25.webp)

下一步创建确认：

![fnos create confirm](https://images.iminling.com/i/2025/01/08/b02e6bce0be8396b9d261a020abc8d5d.webp)

对磁盘进行格式化：

![fnos format disk](https://images.iminling.com/i/2025/01/08/d75627947bc95dadc91e27fe57914980.webp)

格式化完就创建成功了：

![fnos disk created](https://images.iminling.com/i/2025/01/08/4488f9a7b2b8f817f1f48ef411064ae2.webp)

查看创建的存储空间：

![fnos disk](https://images.iminling.com/i/2025/01/08/cd7395e330ccc57ac0fe7fc80a122fe6.webp)

### FnOS文件系统

上边就已经完全安装好了飞牛NSA，飞牛的磁盘存储位置和群晖还是有一点不一样的，我们可以通过ssh登录进系统进行查看他的磁盘信息。

ssh登录信息为访问网页版的用户名和密码。使用访问网页的ip和22端口+用户名和密码来进行登录。

```bash
aa@FnOS:/$ df -h
Filesystem                                               Size  Used Avail Use% Mounted on
udev                                                     1.9G     0  1.9G   0% /dev
tmpfs                                                    394M  7.7M  386M   2% /run
/dev/sda2                                                 63G  9.1G   51G  16% /
tmpfs                                                    2.0G  1.4M  2.0G   1% /dev/shm
tmpfs                                                    5.0M     0  5.0M   0% /run/lock
trimafs                                                  1.8T  1.7T   58G  97% /fs
/dev/mapper/trim_f9cc7969_89e3_4c0b_9857_bcec3a1144d7-0  1.8T  1.7T   58G  97% /vol1
tmpfs                                                    394M     0  394M   0% /run/user/1000
```

可以看到飞牛的系统是放在`/`目录下的，63G, 而我直通的硬盘有两个目录`/fs`和`/vol1`。

<div data-page-id="SPIRdFXeFoo8mBxUJMgu3LHGsIe" data-lark-html-role="root" data-docx-has-block-data="false">
  <div class="ace-line ace-line old-record-id-YajHdl9ZxoJsJIxpfL0uECmrsrb">
    在我的文件自己创建的目录在/fs目录下：
  </div>
```bash
aa@FnOS:/fs/1000/ftp$ ls
docker  download
```




<p>
  直接使用应用商店安装的qb下载目录可以在qb的设置里看到，实际是在<code>/vol1</code>下。</div>
    aa@FnOS:/var/apps/qBittorrent/shares$ ls -alh
    total 8.0K
    drwxr-xr-x 2 root root 4.0K Jan  7 23:05 .
    drwxr-xr-x 6 root root 4.0K Jan  7 23:05 ..
    lrwxrwxrwx 1 root root   27 Jan  7 23:05 qBittorrent -> /vol1/@appshare/qBittorrent
其他的目录信息大家根据情况再去研究。



### qb下载

默认qb只能访问安装后设置里配置的目录，如果想要改变qb下载的目录，则需要去<code>系统设置-应用</code>找到qb，添加一下允许它访问的文件目录。

<p>
  下边是完整后的桌面预览图
</p>
![fnos complete](https://images.iminling.com/i/2025/01/08/3b89be1091025e546acd4224064abb0a.webp)

