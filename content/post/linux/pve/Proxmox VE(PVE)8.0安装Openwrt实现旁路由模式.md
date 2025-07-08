---
title: Proxmox VE(PVE)8.0安装Openwrt实现旁路由模式
author: 要名俗气
type: post
date: 2024-03-24T05:43:31+00:00
url: /2024/pve-install-openwrt
description: 在前两篇文章中分别介绍了pve的安装以及pve安装ikuai做为主路由来拨号上网，此时的网络环境基本已经稳定，但出于特殊需求，我需要使用Openwrt来进行科学上网，所以本文就来折腾一下pve安装Openwrt并实现旁路由模式。
image: https://images.iminling.com/app/hide.php?key=UHVPZkRvYkllMlN2SzRMa09qTXcxSDNHaVd1czRGUGtjbmtNM2RoWnpCWEtWb0FYUmF5OHRtZWhxT3JQb2RUd2RDZkQvS3M9
categories:
  - PVE
tags:
  - openwrt
  - pve
---
在前两篇文章中分别介绍了[pve的安装]({{< ref "/post/linux/pve/畅网J4125安装Proxmox VE(PVE)8.0.md" >}})以及pve安装ikuai做为主路由来拨号上网，此时的网络环境基本已经稳定，但出于特殊需求，我需要使用Openwrt来进行科学上网，所以今天就来折腾一下pve安装Openwrt并实现旁路由模式。

## openwrt镜像

我使用的是[骷髅头的openwrt](https://github.com/DHDAXCW/OpenWRT_x86_x64/releases)镜像，下载的是beggar版本，可根据自己的需求找不同的openwrt版本。下载完成后需要对文件进行解压，下载下来的是`.gz`结尾的文件，需要解压成`.img`结尾的文件，解压后上传到pve中

![upload iso](https://images.iminling.com/app/hide.php?key=Z3djd056bzRnNlRrM05Ob29LazBoRFJpbkRmaTQ4c0QxVVZwbWUzbTRVaGtQUDlpR3dGY1dZMHZ4ak9Wc1hLeDc1TXNSeE09)

## 创建

### 常规

vm ID默认自增到101(ikuai是100)，名称自己填，我这里填openwrt

![pve openwrt common](https://images.iminling.com/app/hide.php?key=VTJuMzZadnFMdGQzWkp6ZFlHWk44Z0kxQVpqemxEQ2hVOFFoT3hpUkRqNDlqZGQzQjJaN3I5YThVUldMeklPd1lOTzVFV2c9)

### 操作系统

选择不使用任何介质

![pve openwrt os](https://images.iminling.com/app/hide.php?key=TzJJVm5XRjNrN1JRMGFZYTlnY3RTR0VPVHd2a1k0L3JKa0FHMmQ1RklOeFhKa0l3M3FMbEE1a0x2UlJaMzAwYkgvSjQzY1k9)

### 系统

全部默认就可以了

![pve openwrt os](https://images.iminling.com/app/hide.php?key=NnVNTjhMWXJ5N2tkOGpzYWFnVWJuWmxhMFEwb2NNdVlqNHFWTmQ1ZTFEcENNd3hKWWpMTlExNlVOb1BMTTltM1RoUjlUMm89)

### 磁盘

这里保持默认，后边会进行删除。

![pve openwrt disk](https://images.iminling.com/app/hide.php?key=czYyZnd1N2I0WnZNcHpiRDU5TWx4TkJucmYzRUpsUFMxc2VDYkl5aEhYanlJckUwcTFtenZ5T3dVdlYwM1U2eVdaa0lJL0E9)

### CPU

同样把j4125的四个核心都给到它

![pve openwrt cpu](https://images.iminling.com/app/hide.php?key=M1EzaHhtZkNrUWNrUWJvdE9ZVzlSdzlnVEJCTUp0YWNjcEJPQyswOFExYjU5aWFNencxM2dpVUhPSmlZc09VUWpnVFc0WjA9)

### 内存

运行openwrt,1GB的RAM就够用了。

![pve openwrt ram](https://images.iminling.com/app/hide.php?key=VE50Vk81QjMzTVYxSi9NZmNIQ2tEWG9KVHIzcGhIQ3RJMi9hVVBGT25wRnlMWEFHM0IwMlJqRzlTTlVOZ2pPZUFZOHp1OHM9)

### 网络

全部默认

![pve openwrt network](https://images.iminling.com/app/hide.php?key=U1V5NUZ5WlYrTGt1dElzaVc2VkgreGVETHpnQldXUE1Bb01INWZmVjBBcW50ak1veC9vcmEvRVliVW5LVmtLdzBWSVZ6SEk9)

### 确认

确认所有选项是否正确。

![pve openwrt summary](https://images.iminling.com/app/hide.php?key=THJaK0tLOGhrR1QxRjdXczNDcVIrVTZuMlBabEJTWmp6ZzRwZlQ0VlZSQWl6ZmtvckV3WTBwbHFyR3lKWHEwZVhObFN1clU9)

没有问题就点完成。

## 硬件配置

创建过程中选择的硬盘是需要删除的，所以需要对硬件进行一些配置。

### 移除CD

在硬件配置中将已存在的CD/DVD驱动器移除

![pve openwrt remove cd](https://images.iminling.com/app/hide.php?key=NnVNTjhMWXJ5N2tkOGpzYWFnVWJuVUZWWEJiVVFMeTdnVDlBMmV4enpzUnJwd3l2ZGlkL0NmY0l1VjRnKytUdngyWTJCWHc9)

### 分离并移除硬盘

选中硬盘，先进行分离，然后进行移除。

![pve openwrt remove disk](https://images.iminling.com/app/hide.php?key=d2YrUER3VXFyRFFmZXF5Z29UL1FHQVpReDFQRy9lNXFqNFZFdVVwZEpkemRSV25iWFd6VzU3WXJ5a1JuWGJ4TVdadUIreWc9)

移除

![pve openwrt remove disk](https://images.iminling.com/app/hide.php?key=d2U1N1FpRE1RSnBmcEhLUFRibUxKZTZUSTJpWGozaFFSdFNPNnFnbWVNZ1lFZW51dmN5dFpYSmdyVkxvSDI0MDBMRnpBOGM9)

## 添加磁盘

需要通过进入`pve的shell`执行命令来导入磁盘,执行命令`qm importdisk 虚拟主机编号 /var/lib/vz/template/iso/openwrt镜像名 local-lvm` ，虚拟主机编号是在经过上边创建虚拟机后pve生成的一个编号，例如我的是101，openwrt镜像名是在刚开始的时候上传到pve中的名称，我的名称是：openwrt.img. 替换后的命令如下：`qm importdisk 101 /var/lib/vz/template/iso/openwrt.img local-lvm`。

![pve openwrt importdisk](https://images.iminling.com/app/hide.php?key=RlhmT0s5OW9TYkg3NjBjNk1YcGNqZ0s1N2pUSDcxVGp5bU9TZk1CMUVrRDNSWlZ3NnVBMnhqblBDeEs2N21KTmY1dXJsM2M9)

执行成功后会在硬件中看到一个未使用的磁盘，进行添加。

![pve openwrt importdisk](https://images.iminling.com/app/hide.php?key=aC9YZlFkSjUxQ2E5aUl4dmVnaGtqdGVwMSs2NG0xemF0aTZ1MGZ0SDVkUitUT01Wd3ViZlgrVkZMckpyQzI3cFd5cGxaVlE9)

### 引导顺序

磁盘添加后需要修改引导为当前的磁盘

![pve openwrt boot sort](https://images.iminling.com/app/hide.php?key=N1ZWR3orL1JUZUJ0bmE0NzI0TmFMUThUTXBTeHBKL3BaSXM3ajdWMGVpakdZUC9pc0hsS2hlZ2pVRlRMOE42TzBzMm5zS2M9)

经过上边的设置就已经完全ok了，可以来启动openwrt了。

## openwrt设置

启动openwrt虚拟机，如下会卡住，敲回车就可以进行下去

![pve openwrt boot](https://images.iminling.com/app/hide.php?key=RWxKcnBENTFsa3RROE1uS1dFZkM2UVBJdDFXV0ZnV29TYmRtdHMyZXdyZXlFYUt2MWxjQWwzaU01WHNKQXB3d3JjR0FucDg9)

回车后如下图，表示启动成功

![pve openwrt](https://www.iminling.com/wp-content/uploads/2024/03/3C4703FF7B2FFF4B2100411200448BCA.png)

### 设置ip

一般固件都有自己的管理ip，可能和我们自己的网络不是在同一个网段，比如我下载这个默认的管理网段是192.168.6.1，我需要改成和我的网络在同一个网段，192.168.1.xxx。

编辑 /etc/config/network，修改如下：

![pve openwrt ip](https://images.iminling.com/app/hide.php?key=M01hVDFwY0tTQXByMFRweVNiNWZWeUxmSWM5SEI4MjRQOVZaRGFNRUlNbXRNSkJTM0tHNUU0TEg2Z3hrajhWL3ZhYS9zNjA9)

我这里改成192.168.1.254，修改保存后重启openwrt,重启后就可以通过访问192.168.1.254在浏览器中进行相关配置了。

### LAN口设置

进入openwrt系统后，首先是对网络接口LAN口进行设置，设置网关为爱快(ikuai)的管理地址，并关闭LAN口的DHCP功能

![pve openwrt lan](https://images.iminling.com/app/hide.php?key=a1g0YVRCMWkvWHh0RUF0dWw3dVlHWTFKejdyditBdHlsM2RKZ282aFBaODAxQzZzR2NObWZBRXZ4SW9jU3N2RDRNK0wvQk09)

修改网关

![pve openwrt gateway](https://images.iminling.com/app/hide.php?key=dExZaDkwMkc3ZitQdkFtdmFabVZYaldDWWdjS0F1WHdwNGVYMjd5eHJFOWl3QXQxMVgxdTEvNjJKTklYUmdsRFV2U2kwTWs9)

这里需要注意一点，我在刚开始设置的时候并没有设置`使用自定义DNS服务器`，后边发现openwrt无法上网，最终是填写了一个DNS服务器后才解决的，这里可以使用223.5.5.5或者其他的DNS服务器。

### 关闭防火墙

接下来关闭openwrt的防火墙

![pve openwrt firewall](https://images.iminling.com/app/hide.php?key=cTBqQlJhWTNGRFZCNVN3N29PYm5wWjFySGN5T21Sc3U5MU1DdXcxclVOd0ZGTUpFYUt1aWVWcFN6Z0NQcGxubHgvZG1hV2s9)

### UPNP

开启UPNP

![pve openwrt upnp](https://images.iminling.com/app/hide.php?key=YzV0N3RvUUcvM3dCR1V0VFIzaUFMeWhGWDZzN0F1aHdHYXRSUXZCVnlMRVNNMmIrWFk2NVpnZjRKZzNzN3FGREpPUlpJMm89)

经过上边的设置，openwrt基本已设置完毕，可以开始使用了。

## 旁路由模式

openwrt做为旁路由，该怎么工作呢，在ikuai设置的时候我设置了31-250的一个DHCP网络段，他们只会经过ikuai来处理流量，那什么设备可以经过这个openwrt旁路由呢？这里就需要在ikuai中进行一些设置了，同样是DHCP服务,我设置了10-30这个ip段的ip的网关为`192.168.1.254`也就是openwrt的ip，那么这个ip段里的设置的网络就会经过openwrt来处理，满足特定需求了。

![pve openwrt ikuai](https://images.iminling.com/app/hide.php?key=QW5DMC95SVl5Q0pwSEpTSDV1aEw4cGpwTVlGNGo5NnpxZkowMHdQVUVTSHhtdUNBM2hOV0NqWXZzOG9QWVFxTnp2aHZsbzg9)

因为我是先设置的31-250这个ip段，所以连接上来的设置被优先往这个ip段分配，自己的其他设置如果想要使用openwrt来处理流量，则对特定设置固定ip就可以了，设置他们的ip在10-30之间。

![pve ikuai dhcp](https://images.iminling.com/app/hide.php?key=Ni9teHZsaWUwOGliZUkyc05OdTFxR2MyOXhJdlZBUXY0VlVORklOZnIrR1o3ZytHTDNpVTRYcjRYUnMzYnlodTNEZjVuenc9)

大功告成，使用pve安装ikuai并使用openwrt做为旁路由。
