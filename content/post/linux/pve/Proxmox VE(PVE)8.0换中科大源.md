---
title: Proxmox VE(PVE)8.0换中科大源
author: 要名俗气
type: post
date: 2024-03-26T23:45:43+00:00
url: /2024/pve-change-ustc-source
description: 前边介绍了[PVE的安装](https://www.iminling.com/2024/03/23/459.html "畅网J4125安装Proxmox VE(PVE)8.0")，Proxmox VE(PVE)8.0的底层使用的是debian12，默认的源在国外下载速度非常慢，甚至可能连接失败，所以需要更换一下默认源为[中科大的源](https://mirrors.ustc.edu.cn/)来加快下载以及更新软件的速度。
featured_image: /wp-content/uploads/2024/03/proxmox_logo.png
categories:
  - PVE
tags:
  - pve
---
![](https://www.iminling.com/wp-content/uploads/2024/03/B42FD7FF691A2DA49E72BE3C533CF000.webp)

前边介绍了[PVE的安装](https://www.iminling.com/2024/03/23/459.html "畅网J4125安装Proxmox VE(PVE)8.0")，Proxmox VE(PVE)8.0的底层使用的是debian12，默认的源在国外下载速度非常慢，甚至可能连接失败，所以需要更换一下默认源为[中科大的源](https://mirrors.ustc.edu.cn/)来加快下载以及更新软件的速度。

## APT源

需要修改三个文件

### sources.list

位置在/etc/apt/sources.list,这个文件里默认有三行：

```
root@pve:~# cat /etc/apt/sources.list
deb http://ftp.debian.org/debian bookworm main contrib

deb http://ftp.debian.org/debian bookworm-updates main contrib

# security updates
deb http://security.debian.org bookworm-security main contrib
```

替换为一下三条：

```
root@pve:~# cat /etc/apt/sources.list
#deb http://ftp.debian.org/debian bookworm main contrib
deb https://mirrors.ustc.edu.cn/debian bookworm main contrib

#deb http://ftp.debian.org/debian bookworm-updates main contrib
deb https://mirrors.ustc.edu.cn/debian bookworm-updates main contrib

# security updates
#deb http://security.debian.org bookworm-security main contrib
deb https://mirrors.ustc.edu.cn/debian-security bookworm-security main contrib
```

或使用一键命令进行替换：

```
sed -i 's|^deb http://ftp.debian.org|deb https://mirrors.ustc.edu.cn|g' /etc/apt/sources.list
sed -i 's|^deb http://security.debian.org|deb https://mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
```

最主要的就是 `http://ftp.debain.org`  替换为 `https://mirrors.ustc.edu.cn`  ,  `http://security.debian.org`  替换为 `https://mirrors.ustc.edu.cn/debian-security`  。

### ceph.list

位置：`/etc/apt/sources.list.d/ceph.list`,原始内容如下：

```
root@pve:~# cat /etc/apt/sources.list.d/ceph.list
deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
```

替换为：

```
root@pve:~# cat /etc/apt/sources.list.d/ceph.list
#deb https://enterprise.proxmox.com/debian/ceph-quincy bookworm enterprise
deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-quincy bookworm no-subscription
```

替换完成。

### pve-enterprise.list {#pve-enterprise-list}

位置是：`/etc/apt/sources.list.d/pve-enterprise.list`,这个是pve企业的源，没有订阅pve也就没什么用，直接把这个文件里的内容注释了。

```
root@pve:~# cat /etc/apt/sources.list.d/pve-enterprise.list
#deb https://enterprise.proxmox.com/debian/pve bookworm pve-enterprise
```

到这里apt的源就替换完成了，可以执行apt udpate来看一下是否更换成功。

```
root@pve:~# apt update
Hit:1 https://mirrors.ustc.edu.cn/debian bookworm InRelease
Get:2 https://mirrors.ustc.edu.cn/debian bookworm-updates InRelease [55.4 kB]
Get:3 https://mirrors.ustc.edu.cn/debian-security bookworm-security InRelease [48.0 kB]
Hit:4 https://mirrors.ustc.edu.cn/proxmox/debian/ceph-quincy bookworm InRelease
Get:5 https://mirrors.ustc.edu.cn/debian-security bookworm-security/main amd64 Packages [148 kB]
Fetched 251 kB in 1s (300 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
94 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

已经使用了中科大的源了，替换成功。

## CT模板源

文件位置：`/usr/share/perl5/PVE/APLInfo.pm`，需要替换 `http://download.proxmox.com`  为 `https://mirrors.ustc.edu.cn/proxmox`,可以执行命令：

```
sed -i 's|http://download.proxmox.com|https://mirrors.ustc.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
```

或者通过直接修改APLInfo.pm的内容进行修改，替换后的内容如下：

```
my $sources = [
        {
            host => "download.proxmox.com",
            url => "https://mirrors.ustc.edu.cn/proxmox/images",
            file => 'aplinfo-pve-8.dat',
        }
```

保存后进行重启pvedaemon：`systemctl restart pvedaemon.service` 。

整个pve的源就替换完成了。
