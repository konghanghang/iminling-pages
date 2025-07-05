---
title: Proxmox VE(PVE)8.0安装爱快ikuai虚拟机并直通网卡
author: 要名俗气
type: post
date: 2024-03-24T04:02:02+00:00
url: /2024/pve-install-ikuai
description: 上一篇介绍了在J4125机器上安装pve环境，安装好pve后整个机器还是无法上网的状态，所以接下来就来介绍一下在pve中安装爱快，使用爱快(ikuai)来进行拨号上网，当然也可以直接安装openwrt,出于综合考虑我这里还是安装爱快(ikuai)，会稳定一些，后续再折腾其他的也不影响家人上网。
featured_image: /wp-content/uploads/2024/03/FE710C520EE784CE8077C6F36D03B8ED.png
categories:
  - PVE
tags:
  - ikuai
  - pve
  - 爱快
---
上一篇介绍了在J4125机器上安装pve环境，安装好pve后整个机器还是无法上网的状态，所以接下来就来介绍一下在pve中安装爱快，使用爱快(ikuai)来进行拨号上网，当然也可以直接安装openwrt,出于综合考虑我这里还是安装爱快(ikuai)，会稳定一些，后续再折腾其他的也不影响家人上网。

## 爱快(ikuai)固件

爱快(ikuai)的固件也不大，直接[官网](https://www.ikuai8.com/component/download)下载就可以了，下载ISO 64位的镜像。把下载的镜像上传到pve中

![upload iso](https://www.iminling.com/wp-content/uploads/2024/03/BC97614C54342FD608F4DC6371CAC5A2.png)

## 创建ikuai虚拟机

鼠标右键点击pve选择创建虚拟机：

![create vm](https://www.iminling.com/wp-content/uploads/2024/03/AB41ED7FE57F77EBE2A7CC399EBDAC49.png)

### 常规

名称命名为ikuai，其他默认

![ikuai common](https://www.iminling.com/wp-content/uploads/2024/03/37C50635EADEB66E0F50391CFFF4E607.png)

### 操作系统

ISO镜像选择上传的爱快镜像：

![ikuai os](https://www.iminling.com/wp-content/uploads/2024/03/2000A752149251377A7E236B43A53685.png)

### 系统

保持默认继续下一步

![ikuai os](https://www.iminling.com/wp-content/uploads/2024/03/A902C31B27FC063D4011BC60D9D61B9D.png)

### 磁盘

ikuai我不做其他用处，所以这里给5GB硬盘

![ikuai disk](https://www.iminling.com/wp-content/uploads/2024/03/9B5DEEC9E17C047DDD48F1EB5D333094.png)

### CPU

cpu把j4125的四个核心都给它

![ikuai core](https://www.iminling.com/wp-content/uploads/2024/03/EE54A081D9E2211E67F697AA788E00D2.png)

### 内存

我这里给2GB的内存

![ikuai ram](https://www.iminling.com/wp-content/uploads/2024/03/844F8B73FDB622462F1AE20D96CC584A.png)

### 网络

默认

![ikuai network](https://www.iminling.com/wp-content/uploads/2024/03/62A58B9B7D3AFB4EAC410515596F963E.png)

### 确认

显示ikuai虚拟机的一些信息，确认和我们选择的是否一致

![ikuai summary](https://www.iminling.com/wp-content/uploads/2024/03/E2EF4664FA052E1AFCCD8A8B56D4BDAB.png)

一些基本信息就已经完成了。

## 添加直通网口

在[畅网J4125安装pve 8.0](https://www.iminling.com/2024/03/23/459.html "畅网J4125安装Proxmox8.0")一 篇中已经规划了ETH3归为pve的管理口，那么剩下的三个网口都直通给ikuai,让ikuai来管理,选择硬件，然后点击添加，PCI设备，就会弹出下图：

![ikuai interface](https://www.iminling.com/wp-content/uploads/2024/03/5FDA72A183F9D81807117C2A397A2B86.png)

可以看到总共有4个I226-V的网口，04已经被pve管理口使用了，那么可以把01，02，03分别添加到硬件中。

## 设置引导项

默认优先是从添加的硬盘启动，因为是第一次启动硬盘里还没有数据，所以需要先从上传的ISO文件引导启动，修改启动项为ISO文件

![ikuai引导](https://www.iminling.com/wp-content/uploads/2024/03/A0313AD16FD54168A7C89025AA6AA4BF.png)

## 安装

经过上边的设置，下边就可以来真正的安装，点击启动

![ikuai install](https://www.iminling.com/wp-content/uploads/2024/03/A78BA1A00FD69091B9137CA1B519E240.png)

选择1安装到硬盘，安装成功后就会自动重启。重启后先关闭虚拟机，因为启动项问题 还是从镜像启动的，所以先关机 然后去调整启动项为硬盘

![ikuai boot](https://www.iminling.com/wp-content/uploads/2024/03/B2891BA68D272DC4F94F7F975F545C1E.png)

调整完成后就可以再次启动ikuai虚拟机了。

### 设置LAN

看到下图说明ikuai已经安装并启动成功了

![ikuai](https://www.iminling.com/wp-content/uploads/2024/03/96E78C61B9C7CAA22C471057C8020643.png)

可以看到上图里有四个网口，eth0-eth3,其中eth0是pve一个虚拟网口，是ikuai的管理网口，其他三个网口是直通的三个PIC网口设备，后边进入ikuai设置界面后会把eth1设置为wan口，用来连接光猫拨号使用。

选择2设置LAN,我规划的ikuai的管理地址是192.168.1.253

![ikuai lan](https://www.iminling.com/wp-content/uploads/2024/03/DE6C391418B396B32F9CCF2605DEF430.png)

设置成功后就可以不用再理pve了，此时可以通过网页进入ikuai的管理页面，浏览器直接访问192.168.1.253(这个根据自己设置的lan地址来进入)，密码都是admin/admin。初次登录会让改密码，修改为自己常用的密码。

### 配置WAN

进入管理页面后会看到WAN口有异常，先配置WAN口

![ikuai index](https://www.iminling.com/wp-content/uploads/2024/03/4B47574A0DB86053168EBC355DBAD3E9.png)

绑定wan并设置pppoe拨号上网：

![ikuai wan](https://www.iminling.com/wp-content/uploads/2024/03/5AF13981CE3B311EC64240ABD76943F6.png)

设置好WAN后，此时还没有把WAN口和光猫连在一起，网络还是不行，我们先完成一些其他的配置项。

### 配置LAN

将直通给爱快的其他两个口分配给lan

![ikuai lan](https://www.iminling.com/wp-content/uploads/2024/03/5DEDA518AF9ED1A4806E37EC47BD7CC5.png)

### DHCP配置

对连接上来的设备进行ip地址自动分配

![ikuai dhcp](https://www.iminling.com/wp-content/uploads/2024/03/A081CFBD137F8CB96187484360AF97E7.png)

添加一条新的dhcp配置,我这里预留一些ip给自己使用，所以分配的ip是从31到250。

网关：设置为ikuai自己的ip

DNS：设置了一个阿里的和一个114的。

其他默认就好。

![ikuai dhcp add](https://www.iminling.com/wp-content/uploads/2024/03/98C1F01FB70729ADC48497586250F8CD.png)

然后重启dhcp服务。

### UPNP

开启ikuai的UPNP服务

![ikuai upnp](https://www.iminling.com/wp-content/uploads/2024/03/884FC4595B931D9989209265CF9BA440.png)

ikuai的设置基本已经设置完毕了。剩下的就是要接入互联网了。

把光猫连在j4125机器的eth1上，路由器接到机器的eth3上来开启wifi，整个家庭的网络就已经通了。
