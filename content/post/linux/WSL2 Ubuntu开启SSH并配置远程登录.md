---
title: WSL2 Ubuntu开启SSH并配置远程登录
author: 要名俗气
type: post
date: 2024-08-17T07:12:52+00:00
url: /2024/wsl2-ubuntu-open-ssh-remote-login
description: 在公司有两台电脑，其中一台正常使用的工作电脑(A)，另一台(B)想要部署一些服务，当做一台服务器来使用，于是就是在B机器上启用了WSL服务，并安装了Ubuntu 22。但是这里遇到一个问题，我总不能一直远程桌面从A中来通过SSH操作B电脑，于是就开始折腾在B电脑开启Ubuntu的远程访问，下边记录一下折腾过程。
image: https://images.iminling.com/app/hide.php?key=ektqWVRFVFhvR1dxYVh3L2tER0pPa0dsR00zZUxYZGJHbnNaY1lZankzNEpyMzgyT0ZoclN3bG5CQkVhajB6cjh0bVl3R0U9
categories:
  - WSL
tags:
  - ssh
  - ubuntu
  - wsl
---
在公司有两台电脑，其中一台正常使用的工作电脑(A)，另一台(B)想要部署一些服务，当做一台服务器来使用，于是就是在B机器上启用了WSL服务，并安装了Ubuntu 22。但是这里遇到一个问题，我总不能一直远程桌面从A中来通过SSH操作B电脑，于是就开始折腾在B电脑开启Ubuntu的远程访问，下边记录一下折腾过程。

## 准备

A电脑需要有SSH远程访问工具，使用自带的powershell或者其他第三方的工具都是可以的。

B电脑需要开启WSL并安装好Ubuntu子程序。

## 开启远程访问

首先需要在B电脑安装ssh-server服务:

```bash
ai@DESKTOP-KKMQ55S:/etc/apt$ sudo apt install openssh-server -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
```

等待安装成功后就可以对ssh服务进行启用：

```bash
ai@DESKTOP-KKMQ55S:~$ sudo service ssh restart
 * Restarting OpenBSD Secure Shell server sshd                                                       [ OK ]
ai@DESKTOP-KKMQ55S:~$ service ssh status
 * sshd is running
```

SSH服务启动后还需要获取当前ubuntu的ip才能进行访问，WSL安装的ubuntu默认无法使用ifconfig命令，需要安装net-tools才可以使用，下边进行安装net-tools以及查看ip地址：

```bash
ai@DESKTOP-KKMQ55S:~$ sudo apt install net-tools
Reading package lists... Done
Setting up net-tools (1.60+git20181103.0eebece-1ubuntu5) ...
Processing triggers for man-db (2.10.2-1) ...
ai@DESKTOP-KKMQ55S:~$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.33.211  netmask 255.255.240.0  broadcast 172.17.47.255
```

如上，通过ifconfig命令查看到ubuntu机器的ip是172.17.33.211，那么可以在B机器中进行远程ubuntu机器，看一下是否可以正常远程：

```bash
C:\Users\ai001>ssh ai@172.17.33.211
ai@172.17.33.211's password:
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.10.16.3-microsoft-standard-WSL2 x86_64)

Last login: Fri Aug 16 16:15:43 2024 from 127.0.0.1
ai@DESKTOP-KKMQ55S:~$
```

可以看到才已经正常开启了ssh并可以连接。

## 其他机器访问

上边已经对WSL的Ubuntu开启了SSH访问，接下来还需要通过其他的机器来连接这个机器，连接示意图如下：

![wsl ubuntu](https://images.iminling.com/app/hide.php?key=Ni9teHZsaWUwOGliZUkyc05OdTFxQmJzOUlwU0pFZkZwNnR0d2d5U2ZSMkgvL2M5Y1ZvL1dLQ2l1bThGYkRDUndyUlloVjQ9)

需要从电脑A访问电脑B，然后电脑B再访问WSL子系统的ubuntu.使用电脑B的ip是无法直接访问到Ubuntu系统的，

这时候就需要在电脑B中进行代理设置，需要以管理员运行Powershell工具，然后执行以下命令，listenport和listenaddress是指电脑B监听的地址和端口，connectport和connectaddress是指需要代理到的地址和端口，我这里是从电脑B的22端口到Ubuntu的22端口，所以端口都是22，ubuntu的ip这里写成了localhost，在上边咱们获取到的ip是172.17.33.211，写成这localhost和172开头的ip应该都是可以的，我这里只是测试了localhost,是可以正常联通的，有兴趣的同学可以自行测试一下。

添加protproxy的语法可以参考微软官方文档：[Netsh interface portproxy commands](https://learn.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-interface-portproxy)

```bash
PS C:\Users\ai001> netsh interface portproxy add v4tov4 listenport=22 listenaddress=0.0.0.0 connectport=22 connectaddress=localhost

PS C:\Users\ai001> netsh interface portproxy show v4tov4

侦听 ipv4:                 连接到 ipv4:

地址            端口        地址            端口
--------------- ----------  --------------- ----------
0.0.0.0         22          localhost       22
```

经过上边的步骤后，我们还需要对电脑B进行防火墙放行22端口的入站流量，命令如下，name可以自定义，我这里是对ssh放行的，所以取名字就叫ssh，[官方文档](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/netsh-advfirewall-firewall-control-firewall-behavior)

```bash
PS C:\Users\ai001> netsh advfirewall firewall add rule name=ssh dir=in action=allow protocol=TCP localport=22
```

然后就可以在电脑A通过电脑B的22端口访问到电脑B的WSL Ubuntu子系统了。

```bash
k@admin:~$ ssh ai@192.168.6.39
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.10.16.3-microsoft-standard-WSL2 x86_64)
Last login: Sat Aug 17 14:36:06 2024 from 172.17.32.1
ai@DESKTOP-KKMQ55S:~$
```

以上就可以完成直接从A电脑访问B电脑的linux服务器，可以直接在上边愉快的部署服务了。

## 其他服务访问

在WSL的Ubuntu中部署其他的服务，然后根据服务占用的端口，需要再去配置电脑B的端口转发，转发到Ubuntu的对应端口去，这里我尝试了部署一个Java的tomcat服务，占用了18080端口，我通过上边的配置代理命令，connectaddress还是localhost,此时是无法访问的，然后对代理进行删除，改成了具体的172.17.33.211后，再通过192.168.6.39:18080就可以正常访问了。

```bash
PS C:\Users\ai001> netsh interface portproxy add v4tov4 listenport=18080 listenaddress=0.0.0.0 connectport=18080 connectaddress=172.17.33.211

PS C:\Users\ai001> netsh interface portproxy show all

侦听 ipv4:                 连接到 ipv4:

地址            端口        地址            端口
--------------- ----------  --------------- ----------
0.0.0.0         22          localhost       22
0.0.0.0         18080       172.17.33.211   18080
```

如果对代理规则或者防火墙配置错误可以通过以下命令来进行删除，代理规则根据监听的端口和ip来进行删除，防火墙则通过name来进行删除。

```bash
PS C:\Users\ai001> netsh interface portproxy delete v4tov4 listenport=18080 listenaddress=0.0.0.0
PS C:\Users\ai001> netsh advfirewall firewall delete rule name=tomcat

已删除 1 规则。
确定。
```

以上就是WSL配置SSH远程访问的过程，并对特定服务的特定端口进行代理转发，实现访问。
