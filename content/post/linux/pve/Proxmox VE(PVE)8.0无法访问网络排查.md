---
title: Proxmox VE(PVE)8.0无法访问网络排查
author: 要名俗气
type: post
date: 2024-03-26T05:34:09+00:00
url: /2024/pve-network-troubleshooting
description: 前边介绍了[Proxmox VE(PVE)的安装](https://www.iminling.com/2024/03/23/459.html "畅网J4125安装Proxmox virtual environment(pve) 8.0")，一直也没有在pve中进行过联网动作，直到今天想要在pve中运行LXC容器，需要下载CT模板的时候才发现原来pve是无法联网的，于是又开始了折腾pve的网络了。 情况说明 在我下载CT模板的时候模板列表是能出来的，但是当点击下载后，等待一段时间后就提示无法下载，但是那个地址我直接在浏览器中访问是可以的。
featured_image: /wp-content/uploads/2024/03/proxmox_logo.png
categories:
  - PVE
tags:
  - pve
---
前边介绍了[Proxmox VE(PVE)的安装](https://www.iminling.com/2024/03/23/459.html "畅网J4125安装Proxmox virtual environment(pve) 8.0")，一直也没有在pve中进行过联网动作，直到今天想要在pve中运行LXC容器，需要下载CT模板的时候才发现原来pve是无法联网的，于是又开始了折腾pve的网络了。

## 情况说明

在我下载CT模板的时候模板列表是能出来的，但是当点击下载后，等待一段时间后就提示无法下载，但是那个地址我直接在浏览器中访问是可以的。在pve的shell中ping一些地址提示：Destination Host Unreachable，如下：

```
root@pve:~# ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
From 192.168.1.251 icmp_seq=1 Destination Host Unreachable
From 192.168.1.251 icmp_seq=2 Destination Host Unreachable
From 192.168.1.251 icmp_seq=3 Destination Host Unreachable
From 192.168.1.251 icmp_seq=4 Destination Host Unreachable
^C
--- 1.1.1.1 ping statistics ---
5 packets transmitted, 0 received, +4 errors, 100% packet loss, time 4065ms
pipe 4
```

但是ping内网的网段是完全ok的，可见是内网通，只是无法连接到互联网。

## 问题分析

在遇到上边问题后，我也上网找了一些解决办法，大部分都是说DNS服务器设置的不对，第一个为lan，第二个为阿里的dns服务器，第三个为爱快(ikuai)的地址，这样子设置后发现还是同样的问题。

![pve dns](https://www.iminling.com/wp-content/uploads/2024/03/EB7A04F12FB94485C93870ECAFD3A4C9.png)

后来在网络配置中发现了问题，这里的网关我配置的是192.168.1.1。

![pve network](https://www.iminling.com/wp-content/uploads/2024/03/B42FD7FF691A2DA49E72BE3C533CFE1E.png)

当时安装pve的时候在规划网络时填的是192.168.1.1，但是当安装完pve，进行ikuai配置的时候我改变了注意，把ikuai的地址分配为192.168.1.253，所以导致192.168.1.1其实并没有设备使用，导致pve无法访问网络。

## 问题解决

既然找到了问题，那就修改一下试一试，于是对pve的网关配置进行修改，把网关地址设置为爱快(ikuai)的地址：

![pve network gateway](https://www.iminling.com/wp-content/uploads/2024/03/AC7CCDE1F9D6DDE1321A91A1B013DC38.png)

再去pve中进行ping检测，发现已经可以通了。

```
root@pve:~# ping 1.1.1.1
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=152 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=164 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=54 time=166 ms
^C
--- 1.1.1.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 151.887/160.656/165.863/6.236 ms
root@pve:~#
```

成功的解决了pve无法联网的问题。
