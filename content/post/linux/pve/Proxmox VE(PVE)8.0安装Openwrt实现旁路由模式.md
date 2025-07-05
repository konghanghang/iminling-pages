---
title: Proxmox VE(PVE)8.0安装Openwrt实现旁路由模式
author: 要名俗气
type: post
date: 2024-03-24T05:43:31+00:00
url: /2024/pve-install-openwrt
description: 在前两篇文章中分别介绍了[pve的安装](https://www.iminling.com/2024/03/23/459.html "畅网J4125安装Proxmox virtual environment(pve) 8.0")以及pve安装ikuai做为主路由来拨号上网，此时的网络环境基本已经稳定，但出于特殊需求，我需要使用Openwrt来进行科学上网，所以今天就来折腾一下pve安装Openwrt并实现旁路由模式。 openwrt镜像 我使用的是[骷髅头的openwrt](https://github.com/DHDAXCW/OpenWRT_x86_x64/releases)镜像，下载的是beggar版本，可根据自己的需求找不同的openwrt版本。
featured_image: /wp-content/uploads/2024/03/openwrt-gradient.jpg
categories:
  - PVE
tags:
  - openwrt
  - pve
---
在前两篇文章中分别介绍了[pve的安装](https://www.iminling.com/2024/03/23/459.html "畅网J4125安装Proxmox virtual environment(pve) 8.0")以及pve安装ikuai做为主路由来拨号上网，此时的网络环境基本已经稳定，但出于特殊需求，我需要使用Openwrt来进行科学上网，所以今天就来折腾一下pve安装Openwrt并实现旁路由模式。

## openwrt镜像

我使用的是[骷髅头的openwrt](https://github.com/DHDAXCW/OpenWRT_x86_x64/releases)镜像，下载的是beggar版本，可根据自己的需求找不同的openwrt版本。下载完成后需要对文件进行解压，下载下来的是`.gz`结尾的文件，需要解压成`.img`结尾的文件，解压后上传到pve中

![upload iso](https://www.iminling.com/wp-content/uploads/2024/03/BC97614C54342FD608F4DC6371CAC5A2.png)

## 创建

### 常规

vm ID默认自增到101(ikuai是100)，名称自己填，我这里填openwrt

![pve openwrt common](https://www.iminling.com/wp-content/uploads/2024/03/530BCC51DD19D59BD1FE510D32076904.png)

### 操作系统

选择不使用任何介质

![pve openwrt os](https://www.iminling.com/wp-content/uploads/2024/03/C8D5A323386C9A1DE5B0BB9BE0386666.png)

### 系统

全部默认就可以了

![pve openwrt os](https://www.iminling.com/wp-content/uploads/2024/03/E821D7E134B98F154E16F7EAC211E8C8.png)

### 磁盘

这里保持默认，后边会进行删除。

![pve openwrt disk](https://www.iminling.com/wp-content/uploads/2024/03/A26DDB4E4114936B881254ED7A0DF2B1.png)

### CPU

同样把j4125的四个核心都给到它

![pve openwrt cpu](https://www.iminling.com/wp-content/uploads/2024/03/837D4B78C94107F0889EC9C53103D241.png)

### 内存

运行openwrt,1GB的RAM就够用了。

![pve openwrt ram](https://www.iminling.com/wp-content/uploads/2024/03/A7271E4BAEB3BEFE01324962F694F811.png)

### 网络

全部默认

![pve openwrt network](https://www.iminling.com/wp-content/uploads/2024/03/5A90B845671AC32312801582885D4C09.png)

### 确认

确认所有选项是否正确。

![pve openwrt summary](https://www.iminling.com/wp-content/uploads/2024/03/87FEC65DDEE9CF8BB85FFFFF5B6D8C10.png)

没有问题就点完成。

## 硬件配置

创建过程中选择的硬盘是需要删除的，所以需要对硬件进行一些配置。

### 移除CD

在硬件配置中将已存在的CD/DVD驱动器移除

![pve openwrt remove cd](https://www.iminling.com/wp-content/uploads/2024/03/81E209EBFF4348F0AE3B4E54454553D6.png)

### 分离并移除硬盘

选中硬盘，先进行分离，然后进行移除。

![pve openwrt remove disk](https://www.iminling.com/wp-content/uploads/2024/03/5894F46E2E267BC2B2E470D46D4C1CE0.png)

移除

![pve openwrt remove disk](https://www.iminling.com/wp-content/uploads/2024/03/F613F961817AF86518ABE38F0FC5C596.png)

## 添加磁盘

需要通过进入`pve的shell`执行命令来导入磁盘,执行命令`qm importdisk 虚拟主机编号 /var/lib/vz/template/iso/openwrt镜像名 local-lvm` ，虚拟主机编号是在经过上边创建虚拟机后pve生成的一个编号，例如我的是101，openwrt镜像名是在刚开始的时候上传到pve中的名称，我的名称是：openwrt.img. 替换后的命令如下：`qm importdisk 101 /var/lib/vz/template/iso/openwrt.img local-lvm`。

![pve openwrt importdisk](https://www.iminling.com/wp-content/uploads/2024/03/CC7BD8ADEB55C47A3D661F37295D1213.png)

执行成功后会在硬件中看到一个未使用的磁盘，进行添加。

![pve openwrt importdisk](https://www.iminling.com/wp-content/uploads/2024/03/3B19CC29E51F0933E2F10FCE711C62B4.png)

### 引导顺序

磁盘添加后需要修改引导为当前的磁盘

![pve openwrt boot sort](https://www.iminling.com/wp-content/uploads/2024/03/3B9F70D8EF2F373E80CB1162B86C7A3E.png)

经过上边的设置就已经完全ok了，可以来启动openwrt了。

## openwrt设置

启动openwrt虚拟机，如下会卡住，敲回车就可以进行下去

![pve openwrt boot](https://www.iminling.com/wp-content/uploads/2024/03/2651DCA4CAEAB2FC8C5B7552D7AFD17B.png)

回车后如下图，表示启动成功

![pve openwrt](https://www.iminling.com/wp-content/uploads/2024/03/3C4703FF7B2FFF4B2100411200448BCA.png)

### 设置ip

一般固件都有自己的管理ip，可能和我们自己的网络不是在同一个网段，比如我下载这个默认的管理网段是192.168.6.1，我需要改成和我的网络在同一个网段，192.168.1.xxx。

编辑 /etc/config/network，修改如下：

![pve openwrt ip](https://www.iminling.com/wp-content/uploads/2024/03/30305231BD4E418E530C4EFB1F2D2B2F.png)

我这里改成192.168.1.254，修改保存后重启openwrt,重启后就可以通过访问192.168.1.254在浏览器中进行相关配置了。

### LAN口设置

进入openwrt系统后，首先是对网络接口LAN口进行设置，设置网关为爱快(ikuai)的管理地址，并关闭LAN口的DHCP功能

![pve openwrt lan](https://www.iminling.com/wp-content/uploads/2024/03/7F71B9608FB1F3EC02023B3A60FDCD08.png)

修改网关

![pve openwrt gateway](https://www.iminling.com/wp-content/uploads/2024/03/0DC12C48DA49DFC3761270F7C1647457.png)

这里需要注意一点，我在刚开始设置的时候并没有设置`使用自定义DNS服务器`，后边发现openwrt无法上网，最终是填写了一个DNS服务器后才解决的，这里可以使用223.5.5.5或者其他的DNS服务器。

### 关闭防火墙

接下来关闭openwrt的防火墙

![pve openwrt firewall](https://www.iminling.com/wp-content/uploads/2024/03/FD8C04F7DB94CBB12DB8675733828FDA.png)

### UPNP

开启UPNP

![pve openwrt upnp](https://www.iminling.com/wp-content/uploads/2024/03/A69A4758D6648D6D3977B0F2A9552B54.png)

经过上边的设置，openwrt基本已设置完毕，可以开始使用了。

## 旁路由模式

openwrt做为旁路由，该怎么工作呢，在ikuai设置的时候我设置了31-250的一个DHCP网络段，他们只会经过ikuai来处理流量，那什么设备可以经过这个openwrt旁路由呢？这里就需要在ikuai中进行一些设置了，同样是DHCP服务,我设置了10-30这个ip段的ip的网关为`192.168.1.254`也就是openwrt的ip，那么这个ip段里的设置的网络就会经过openwrt来处理，满足特定需求了。

![pve openwrt ikuai](https://www.iminling.com/wp-content/uploads/2024/03/213157C522C8091C348D788D4777FDE0.png)

因为我是先设置的31-250这个ip段，所以连接上来的设置被优先往这个ip段分配，自己的其他设置如果想要使用openwrt来处理流量，则对特定设置固定ip就可以了，设置他们的ip在10-30之间。

![pve ikuai dhcp](https://www.iminling.com/wp-content/uploads/2024/03/5B2B6D7B677CDC4C07E01EC2ADB813F4.png)

大功告成，使用pve安装ikuai并使用openwrt做为旁路由。
