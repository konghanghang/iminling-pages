---
title: 畅网J4125安装Proxmox VE(PVE)8.0
author: 要名俗气
type: post
date: 2024-03-23T09:15:47+00:00
url: /2024/j4125-install-pve
description: 年前综合考虑买了一台畅网的J4125，低功耗，也能满足一些搞机需求。由于一直忙于工作，一直没有时间研究它，最近终于抽空给机器装上了PVE8.0系统，记录一下折腾的过程。
image: https://images.iminling.com/app/hide.php?key=ZG5zSWhxS29UVDdDc25iUEVOTmRiREVtRjdYOHNNMmNnbHVyRXd2QUJZRnVacll6TW8vZ3EzQ0g2ZnFhYXN5TEozU3hQOUU9
categories:
  - PVE
tags:
  - Proxmox
  - pve
---
年前综合考虑买了一台畅网的J4125，低功耗，也能满足一些搞机需求。由于一直忙于工作，一直没有时间研究它，最近终于抽空给机器装上了PVE8.0系统，记录一下折腾的过程。

## 机器配置

畅网J4125
- 英特尔®第十代Gemini Lake低功耗四核处理器
- 4个Intel i226-V 2.5G网卡
- 2条SO-DIMM DDR4 2400MHz（兼容2666/3200MHz）
- 1个mSATA接口、1个SATA3.0硬盘接口
- 1个HDMI 1.4接口支持4K（4096x2160@30Hz）
- 1个DP 1.2接口支持4K（4096x2160@60Hz）
- 4个USB 3.0
- 全长miniPCle插槽支持USB信号的4G/WiFi通讯模块
- 直流DC-12V 55*25输入或凤凰端子供电
- 支持看门狗、GPIO、PXE无盘启动、网络唤醒

我这边是买了一个梵想的256G mSATA硬盘，另外买了两个三星8GB,3200HZ的内存条，整机的硬件算是完成。


## 准备工作

### U盘

安装PVE需要从U盘进行启动，这里使用[ventory](https://www.ventoy.net/cn/download.html)来制作启动盘。电脑上插上需要制作启动盘的U盘，然后下载ventory解压后运行`ventory2Disk.exe`，设备选择自己的U盘，安装就可以了。

![ventory](https://images.iminling.com/app/hide.php?key=R1ZvOW8rYzdBYlhXYmdFZGl3L0ordU11cjdNRUFlMDBhVW10cUNxSEt4MWxvMFA3R1YxR3FMaDUraG1CNVphN0N0ZDlQS3c9)

### PVE下载

直接[PVE官网](https://pve.proxmox.com/wiki/Downloads)进行下载，根据自己的情况下载，我这里下载的是8.0的版本。

![pve download](https://images.iminling.com/app/hide.php?key=Q0FpelpEUnUxeStGdHFxRjNuZUpabFlGS3hvZGVlSTZoMGtuc3ZjMnJCYlFSbnUvQnhYQlduYWFjWS9ROVpkcXBmaFRZaTg9)

把下载好的pve镜像放到上边制作好的U盘中。

准备工作已完成。

## 安装PVE

**安装的所有截图都来源于网络，因为安装的过程没办法截图，具体的ip地址请以文字中说明的为准。**

**网段是：192.168.1.xxx**

**网关：192.168.1.253**

**pve管理ip：192.168.1.251**

### 网口规划

机器有4个网口，ETH0、ETH1，ETH2，ETH3，我打算是让ETH3来作为管理端口，所以在后边安装的时候选择最后一个网口。

![j4125](https://images.iminling.com/app/hide.php?key=S3cwNHpESWhZUndqVGtCL04xUFAwdlYzeC91ZHhENExXQklDekpwemtzYWovU2pyc1FXSm52MTFZWUJJZHcwUUlUYlF5eDA9)

### 安装

下边就可以开始安装pve了，机器通过HDMI连接到显示器，并连接键盘和鼠标。然后进行开机，我这台机器是按`delete`键进入bios设置,在`Boot`选项中选择`Boot option #1`中选择上边制作的启动盘，然后F10保存重启。

![bios](https://www.iminling.com/wp-content/uploads/2024/03/24959220BB5C4D43BDFF77B6BC32C837.png)

#### 模式选择

开机后就进入了安装，选择第一个模式进行安装：
![pve mode](https://images.iminling.com/app/hide.php?key=YXZYa20rRGJIRCtoYzA1a0VGaC9QN0dPQTdxOHdWTng2dVNKeUx3RUhOVmVZTldFOWY4MDBYeEtvYm9Xa2s1QXEwS1c2NEE9)

#### 协议

接下来就是同意协议：

![agree](https://images.iminling.com/app/hide.php?key=YXZYa20rRGJIRCtoYzA1a0VGaC9Qenp5Q0NZVVN5c1EyV3VacWlQWGhKL092Mm00OXRPVGowTWtVWnZxRGFlSzB5RXJZc0U9)

#### 硬盘选择

选择硬盘，选择自己的J4125机器中安装的硬盘，不要选择成了U盘。
![pve disk](https://images.iminling.com/app/hide.php?key=MTVic251b3hPNkxPcWk4L2RXZDN3c2tqQ2Z0V2RGLy82WXp2R2xSZFVZUEhpSW8rVVFlTmZqaWh1Y24vakNyTmtueXdiNm89)

#### 地区设置

地区和时区以及键盘设置：
![pve location](https://images.iminling.com/app/hide.php?key=RE00UnlGYTVZRnBydVd5bjdCeG5pWXBlODFVWTNuTUthQnFWS0d0WmtMaXhTYmRBYjdaamU1UTdNWCtIbXJzQWFFYmErNW89)

#### 密码设置

管理密码和邮箱设置,**密码在后边登录pve的时候要用到，请牢记。**
![pve password](https://images.iminling.com/app/hide.php?key=Q01pd2g3dFo0SmVOdWdvUUNjQjY2RHNrTlpSTFc4Yzg4eDNFLzZ1MUVLeWpFK1JpdnhET1d1Sm0vUVVmNk1uanI0eUo0OVk9)


#### 网络设置

接下来的就是重点了，需要指定pve的管理网口以及ip的一些信息
![pve network](https://images.iminling.com/app/hide.php?key=SjhTMHBELzNqUjFYZ01xbEtZSWY0Sjh5d2VTQ3BwOEdteTdpbzN3aUovY3o3eGkxTG9LOGZkWWRIUDRqNkIvcG10SVdXZWc9)


假如我想让我家的所有设置的ip是`192.168.1.xxx`开头,其中第三位的1可以替换为自己想要的网段，比如：`192.168.31.xxx`(小米路由器就是用这个网段)

详细说明一下这5个选项：

管理接口：这里在上边也说了，我打算把管理接口放在ETH3，所以我这里选了最后一个：enp4s0.

主机名称：这里可以不用处理，默认就行了。

ip地址：这里的ip是等下在网页登录的地址，根据自己需求来指定，我自己的设置为`192.168.1.251`,

网关：这里是主路由的地址，比如我在安装pve后想要再装ikuai来拨号上网，并且ikuai的管理ip为：192.168.1.253，那么我这里的网关地址就填这个了。如果你后边的ikuai是其他的ip，替换为自己的。

DNS服务器：填写和网关一样的地址就可以了。

这里的网关如果填错了也没关系，后边在pve里也可以进行修改，如果填错最直接的表现就是pve没办法上网。这个我在折腾的过程中也遇到了。

剩下的就是核对信息，然后进行安装了。等安装完成后会提示让重启。进行重启就可以了，**在重启的时候拔掉U盘**，防止又从启动盘中进行了启动。

![pve installed](https://images.iminling.com/app/hide.php?key=ZXRPUS82d3lMbUNqVWZWTXg0cVVHb0xaZDc4SGFYNWN2MzdFZm5ZU09KTXltZGdBcFBIak05aXhzaDN0akt6YU84NUZENWs9)

## PVE设置

上边安装pve后会进行重启，重启后会进入一个命令界面，需要输入账号和密码才能进入

![pve login](https://images.iminling.com/app/hide.php?key=RVpXQnJCRHFsZVM0cXdtSjN3T1BYblpQYjNOSUJJeEtiYlRQczgwQTlKSk4yTm8xd2NJbGN4Nk94RmR3QW9kbGJJSG5XeDA9)

这时候可以把机器连接到电脑，使用电脑通过网页端来设置pve了。正确的连接方式是：在安装过程中的管理端口连上网线，网线的另一端插入到电脑的网线口，并设置电脑端和pve在同一个网段（这个在上边安装的时候已经说明过），比如我的安装网段是192.168.1.xxx，pve管理ip是：192.168.1.251，那么设置电脑的网络为192.168.1.200就可以了。

![ip](https://images.iminling.com/app/hide.php?key=dmlMc2RYVnRoTHNrNDV3UDlaNi80cG9XZ0VqU2hycTNNUUZXYS84OS9wcEtoa0Q1aS8yLzJEN0JnUkI2VTYzNktpNGYvTWM9)

设置好后就可以通过网页端访问ip：192.168.1.251:8006来访问了,如果提示不安全则点击高级继续访问：

![access pve](https://images.iminling.com/app/hide.php?key=aC9La1lEV1pSUHhsK1ZWcHZPN1hzMXgxVS9aRmZDSzc4N256UmJuOC93Mys3czZib3RJLzVMMmpXOER0aFRSUUMrd1owSFU9)

然后输入用户名和密码进行登录，用户名：root，密码是在安装的时候输入的密码。就进入了pve的管理界面：

![pve manage](https://images.iminling.com/app/hide.php?key=bStPc2JCOEkvU0YvWXF5YWNTUlZOKzZCWTJZRW5wbEFxSW1Fb3hjYmxEUU5LY1ZkemVUTmpnN0ZOVzU3anpCYjlvODdCL2M9)

### 网口直通

进入后首先需要设置的就是网口直通，进入pve的shell界面，修改/etc/default/grub文件，修改GRUB\_CMDLINE\_LINUX\_DEFAULT="quiet"为GRUB\_CMDLINE\_LINUX\_DEFAULT="quiet intel_iommu=on"

![pve grub](https://images.iminling.com/app/hide.php?key=RVpXQnJCRHFsZVM0cXdtSjN3T1BYaU5LbWpBa3czN2xFTkR1WVNJOGVFNTFJV2VxUXlzVDMyMmF1UjFoOERsSGVUQWdxMUU9)

然后执行`update-grub`更新并重启：

```bash
root@pve:~# vim /etc/default/grub
root@pve:~# update-grub
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-6.2.16-3-pve
Found initrd image: /boot/initrd.img-6.2.16-3-pve
Found memtest86+ 64bit EFI image: /boot/memtest86+x64.efi
Adding boot menu entry for UEFI Firmware Settings ...
done
root@pve:~# reboot
```

进行到这里pve安装基本已经完成了，截止到现在pve还不可以上网，这个需要在进行爱快(ikuai)安装后，进行拨号上网让整个环境都有网络。后续在进行介绍。
