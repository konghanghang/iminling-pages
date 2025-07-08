---
title: Proxmox VE(PVE)8.0安装爱快ikuai虚拟机并直通网卡
author: 要名俗气
type: post
date: 2024-03-24T04:02:02+00:00
url: /2024/pve-install-ikuai
description: 上一篇介绍了在J4125机器上安装pve环境，安装好pve后整个机器还是无法上网的状态，所以接下来就来介绍一下在pve中安装爱快，使用爱快(ikuai)来进行拨号上网，当然也可以直接安装openwrt,出于综合考虑我这里还是安装爱快(ikuai)，会稳定一些，后续再折腾其他的也不影响家人上网。
image: https://images.iminling.com/app/hide.php?key=MGt0VDNLdUNhUDI2MXVlRkUvSXZvVS9KSE14UzlOazF4Z3JqRjhmVnhFbmV0UzdXNE1NUXUyNUxXNkVQNjN6VlVtL1JpN009
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

![upload iso](https://images.iminling.com/app/hide.php?key=YmVpMHVUeVovMHZzMWd5bklhdERmR2kwTlBnWUx1U0xlc2M4Y3Z0OE5wdXNraTNKVGZnZW1XaVJMSFhWWDdUNEZpVkcvKzA9)

## 创建ikuai虚拟机

鼠标右键点击pve选择创建虚拟机：

![create vm](https://images.iminling.com/app/hide.php?key=TzFEUWFRdXdmV3NWcURMSUxWRUdNMG5HcDZQZ04yZzBWaitZVDgxdHFqeXBXeDliQXZocytmMmpGVHhFZjk3MGVlQ1Nyckk9)

### 常规

名称命名为ikuai，其他默认

![ikuai common](https://images.iminling.com/app/hide.php?key=QStIZ0I0Y3k4RmJmL1lZY3lLbjZDQzkvNXh2RkRNNXp2QmpoRGZVNVhiWWlJdlA1V0hJdDNoRlVOZWk1VHBDR2lBQVRHNWs9)

### 操作系统

ISO镜像选择上传的爱快镜像：

![ikuai os](https://images.iminling.com/app/hide.php?key=R0dOdys1WjBPVHJ2bjQzT083aE5TM1ZKZ0I4NWcrbldxeFZldnh0ZEZTcVEwaGx4T3JDcS9PUFc3VllYdjZ5bWRpSVpxdFE9)

### 系统

保持默认继续下一步

![ikuai os](https://images.iminling.com/app/hide.php?key=dTRuQi9oWFFCZ2JmUDF2bDRUUFVEQk9Kd3BpamVVU2QzWDdTUWdHWkVZMCtaclRHKzBXNlQ0bDlhdmRjZWRoYlNSa2l1QmM9)

### 磁盘

ikuai我不做其他用处，所以这里给5GB硬盘

![ikuai disk](https://images.iminling.com/app/hide.php?key=eTdOSzFldUthWHMzcGRNTUxHeHVnL2VvQ2pMUEl6OWVKZGsrbk1VU014c1FXWEFzYTF2eitOWnI1allHV0xPZ3VUVDF5bUU9)

### CPU

cpu把j4125的四个核心都给它

![ikuai core](https://images.iminling.com/app/hide.php?key=aFlZM3FOVzV1WWoxT0x0TVZyZHQvdGFFMmRvNzBVckJDYnljWk9MTGRoM1ZOc2hvb3NOVllFRUJYbFhDWlhMcXF5TjVBbk09)

### 内存

我这里给2GB的内存

![ikuai ram](https://images.iminling.com/app/hide.php?key=N1ZWR3orL1JUZUJ0bmE0NzI0TmFMZm9IVmRFUngvTE56cit6M3JUOG1IRzV5SmJ0SHhTcWhlVFBwdVgvU2k1ajVONWFXQ2M9)

### 网络

默认

![ikuai network](https://images.iminling.com/app/hide.php?key=VTJuMzZadnFMdGQzWkp6ZFlHWk44dllQZjRCRjZhYlRSS0ROaVVZOFpmVXFZYlRYYmJCS2o4WDZvNkh0NmFzY2lPR3FyZW89)

### 确认

显示ikuai虚拟机的一些信息，确认和我们选择的是否一致

![ikuai summary](https://images.iminling.com/app/hide.php?key=UFdSZWV6ZHVBWGV2aVduRVRpTnFtaGlnS1h4RGdqVm5SalVHVWFXOHJTNVNmaGZJK1FHSE04SlE4UXNZekFhYkx3WG9JcVU9)

一些基本信息就已经完成了。

## 添加直通网口

在[畅网J4125安装pve 8.0]({{< ref "/post/linux/pve/畅网J4125安装Proxmox VE(PVE)8.0.md" >}})一 篇中已经规划了ETH3归为pve的管理口，那么剩下的三个网口都直通给ikuai,让ikuai来管理,选择硬件，然后点击添加，PCI设备，就会弹出下图：

![ikuai interface](https://images.iminling.com/app/hide.php?key=b0xvQzNlRWRxYUFBcGVXOEpZaXRScnFKYUFFbkhsSE5UV01RaFpaUTdvWG4wTGpjVnlhLzlEMEVuNEljblBWRXZQQ1JLVGc9)

可以看到总共有4个I226-V的网口，04已经被pve管理口使用了，那么可以把01，02，03分别添加到硬件中。

## 设置引导项

默认优先是从添加的硬盘启动，因为是第一次启动硬盘里还没有数据，所以需要先从上传的ISO文件引导启动，修改启动项为ISO文件

![ikuai引导](https://images.iminling.com/app/hide.php?key=SmNJeGlsSVc0SXBEVWVrYWJsdTNyeVpPYml0YVJVeGc0Z0hVT1N6WlM5TGppL1JrVHNOY05DeW00TVgycWhOVk55YUE0TU09)

## 安装

经过上边的设置，下边就可以来真正的安装，点击启动

![ikuai install](https://images.iminling.com/app/hide.php?key=L0s5cGd2ZHhMNUl5NTRKaHAvTjVWVzV3aTZmVFRhRHJvcE1iUFZneFoxSzNIWmFYeUhadFJCN1dsOGJYd01qblNLTHoxanc9)

选择1安装到硬盘，安装成功后就会自动重启。重启后先关闭虚拟机，因为启动项问题 还是从镜像启动的，所以先关机 然后去调整启动项为硬盘

![ikuai boot](https://images.iminling.com/app/hide.php?key=bjhCenFXRWRTcG1sd281Ym5XZThJRmlOdUoyaGtUMjlOTmlzVHZ5SEFQQWtqN1BXTzAwUEpjbmRyNHhMQVhlWWt0dFI0dlk9)

调整完成后就可以再次启动ikuai虚拟机了。

### 设置LAN

看到下图说明ikuai已经安装并启动成功了

![ikuai](https://images.iminling.com/app/hide.php?key=b0w0Mnh3eThuS0JueUNaajQrNU1LUmpkc0dnRTJITFUvWTNCaDA0b2MzbU1Mb0FQWFZCdGdDTm5zU0xTSC9wL3VFbW9Ka289)

可以看到上图里有四个网口，eth0-eth3,其中eth0是pve一个虚拟网口，是ikuai的管理网口，其他三个网口是直通的三个PIC网口设备，后边进入ikuai设置界面后会把eth1设置为wan口，用来连接光猫拨号使用。

选择2设置LAN,我规划的ikuai的管理地址是192.168.1.253

![ikuai lan](https://images.iminling.com/app/hide.php?key=TlUwdWJRaGpRZ2llajJNcFpuYU1oYmFoNEpDUStQY084VlNpV2NLVVgzd0MzZVdOQjREcGpPN0JneEtmQXJBVDQ1NURvc2M9)

设置成功后就可以不用再理pve了，此时可以通过网页进入ikuai的管理页面，浏览器直接访问192.168.1.253(这个根据自己设置的lan地址来进入)，密码都是admin/admin。初次登录会让改密码，修改为自己常用的密码。

### 配置WAN

进入管理页面后会看到WAN口有异常，先配置WAN口

![ikuai index](https://images.iminling.com/app/hide.php?key=OWlLNVVaL0JXY242anVuNHJDZjdNR3pnRUNKT2ltQUFGN3BhQlJDTUxmVDhBR3htK3hybnZTTXZuQ1IrYUVnVW0zNXM3K2c9)

绑定wan并设置pppoe拨号上网：

![ikuai wan](https://images.iminling.com/app/hide.php?key=ejE5UmtyOWZkSlgwMlJDSGU4dENjZmlsbExoYnkxRzRHQ0NJdmZnZ0xmdmFMR0F2KzlERDhyVFgvT1lxa1VQQU9PME5aK3c9)

设置好WAN后，此时还没有把WAN口和光猫连在一起，网络还是不行，我们先完成一些其他的配置项。

### 配置LAN

将直通给爱快的其他两个口分配给lan

![ikuai lan](https://images.iminling.com/app/hide.php?key=YWxVV205YnhXcWRyc1VYTkcwcmEvbkNGbVQ3c3VBZ09UOWJacGdnakV0WWViclFiT0gyeTc5c1M2VTFDdm1uNzZ0ZEEvMzg9)

### DHCP配置

对连接上来的设备进行ip地址自动分配

![ikuai dhcp](https://images.iminling.com/app/hide.php?key=RjFDWXBkVGJlbVdYdkpqZTM0QmxzZU1uc3dyajNHZk90N05iSkNpSlR1UUJ2L3VXMEx4czBTUE5GYisyamdXTDlwejZIc289)

添加一条新的dhcp配置,我这里预留一些ip给自己使用，所以分配的ip是从31到250。

网关：设置为ikuai自己的ip

DNS：设置了一个阿里的和一个114的。

其他默认就好。

![ikuai dhcp add](https://images.iminling.com/app/hide.php?key=cFB4czUxV3pvSVh3cm8xd2xxYlB3Y1dEY09uMHVFZS8yNHBJUTk5M3dxMWo0ZW56eVhtY0crakdvdGdlek4zQi94Ti9BRmc9)

然后重启dhcp服务。

### UPNP

开启ikuai的UPNP服务

![ikuai upnp](https://images.iminling.com/app/hide.php?key=VW54V3VOcDcvYVBFRnFQQjlSb1NIdWVOMkpwTEFNdjU1dklnaDRvN2pEZmpxS2paTGE3Y3JBRUpLM2RRU2ZqbjBGUFRDT0k9)

ikuai的设置基本已经设置完毕了。剩下的就是要接入互联网了。

把光猫连在j4125机器的eth1上，路由器接到机器的eth3上来开启wifi，整个家庭的网络就已经通了。
