---
title: Proxmox VE(PVE)8.0使用CT模板创建LXC版docker服务
author: 要名俗气
type: post
date: 2024-03-27T15:28:11+00:00
url: /2024/pve-use-ct-template-install-lxc-docker
description: 前边已经分享了pve的安装，pve下爱快安装以及pve下openwrt的安装，还想要弄一个docker来跑一些其他的服务，docker的安装有两种方式，一种是装一个linux的虚拟机，然后在里边来跑docker,另一种就是今天要介绍的使用CT模板来创建LXC容器跑docker。
image: https://images.iminling.com/app/hide.php?key=aXJqeW9LcUVTSmRtV2N0TXhRc05EZEpDQXpLTU5oWEtLcW5CejJNa0R0SW0zalI2ZDN1MDVrcUp4Znd3RWtHK1V0em9KeFU9
categories:
  - PVE
tags:
  - lxc
  - pve
---
前边已经分享了[PVE的安装]({{< ref "/post/linux/pve/畅网J4125安装Proxmox VE(PVE)8.0.md" >}})，pve下[安装爱快(ikuai)]({{< ref "/post/linux/pve/Proxmox VE(PVE)8.0安装爱快ikuai虚拟机并直通网卡.md" >}})以及pve下[安装openwrt]({{< ref "/post/linux/pve/Proxmox VE(PVE)8.0安装Openwrt实现旁路由模式.md" >}})实现旁路由，还想要弄一个docker来跑一些其他的服务，docker的安装有两种方式，一种是装一个linux的虚拟机，然后在里边来跑docker,另一种就是今天要介绍的使用CT模板来创建LXC容器跑docker。

## CT模板下载

我这里使用的是debain12来当作模板，可以在pve里下载该模板。

![pve ct debain](https://images.iminling.com/app/hide.php?key=TzJJVm5XRjNrN1JRMGFZYTlnY3RTSWxwNVB2YXI5V2N0b2ZVYVpSdmFaaW5FTzdPYWFQQk14SzZZQ3RQUTl2cDZxeDY1Z1E9)

## 创建CT

### 常规

点击`创建CT`,这里的主机名随便给一个，因为我这里是要专门来运行docker,所以我这里填：docker, 密码是这个容器的登录密码，一定要牢记。**另外要把无特权的容器取消勾选**。

![pve ct docker common](https://images.iminling.com/app/hide.php?key=YmVpMHVUeVovMHZzMWd5bklhdERmTWU4TmhXNTdxRzlsZlBPOFliczMvdVpJaTA2MnlhSEtIemJuTVRLSUV6UkRxVW83Szg9)

### 模板

模板选择下载的debian模板

![pve ct docker template](https://images.iminling.com/app/hide.php?key=MU5XZFJXZ09VR2xVZkt3dS8vNEVzYmRqV1JUYWdaZCtDWjlmTG9aWnpvWHJ2K0JqWmZSRHVxMVRTQ2lrRmUzWmV6OUNLcnc9)

### 磁盘

磁盘大小根据自己的实际需要给

![pve ct docker disk](https://images.iminling.com/app/hide.php?key=ZXZqajRmS2xTN3dreUhxWXM4ZG9wdElSVmNWMlB1L3BYUFpONzJudGpnMjUwaENkU0llWWZHRkplUzVZWk5CTlNtKytpdEU9)

### CPU

cpu把j4125的4核都给过去

![pve ct docker cpu](https://images.iminling.com/app/hide.php?key=RWxKcnBENTFsa3RROE1uS1dFZkM2ZnAxVWJUdFVDUWVsZk1FZWZ4S3pLYk9ZdWVadzNHQ0xOKzczMjdjOEwxMUJuWURzdEU9)

### 内存

我这里先给2GB，后续根据需求再添加

![pve ct docker ram](https://images.iminling.com/app/hide.php?key=eFlxVkQrWjJYSGJhVDF1eWhmUy9LMXFlZUtzQ28wQXFIZ1JzUU84b0VIQllSS0Z3eUs3SHZYdTdKN1dBSUUyWDNBdWEvTlk9)

### 网络

我这里手动指定了ip，可以使用dhcp自动分配ip

![pve ct docker network](https://images.iminling.com/app/hide.php?key=Y2FYY1hHdHZaYjB0OFdmdklWNEF0TlpoOU53c1pQeVBYdHlLd3c4SkRuUWozZ0FyT1N0cUYrdEFwT05zZW1VdjR6d1Z0aEE9)

### DNS

dns默认

![pve ct docker dns](https://images.iminling.com/app/hide.php?key=V0hrWXB3bkFYQXY4bjArODQ5NW9KaVVYbDhKaEZzVWpESE85N1Y5RXJ4ZEdHTDZNTGpkeFJ2ejBjNEtza1VabzB5cytOb3c9)

### 确认

刚才设置的一些信息的预览

![pve ct docker confirm](https://images.iminling.com/app/hide.php?key=UHVPZkRvYkllMlN2SzRMa09qTXcxTTFkRHFLRFF4V0lFd1l6bGRMUGt6K0FyYkN1R3l2VDlQRWVBa0MydHUwV2kzWG9yYnM9)

创建完成后先不要开机，还需要一些其他的一些配置。

### 功能

在功能里勾选NFS等选项

![pve ct docker function](https://www.iminling.com/wp-content/uploads/2024/03/AAE848B3187FBB8E0F1A8F2DF763C112.png)

### LXC配置文件

还需要进入pve的shell,对刚创建的LXC容器的配置文件进行修改，位置：/etc/pve/lxc,此时里边应该只有1个配置文件，文件名对应创建的lxc容器在pve里的id。我的是102.conf

需要在后边再添加几行：

```bash
lxc.apparmor.profile: unconfined # 表示容器内的进程将不受任何 AppArmor 限制
lxc.mount.auto: cgroup:rw
lxc.mount.auto: proc:rw
lxc.mount.auto: sys:rw
lxc.cap.drop:  # 用于指定容器内进程的能力限制，允许进程执行一些特定的操作，例如修改系统时间、挂载文件系统等
lxc.cgroup.devices.allow: a
```

完整的配置如下：

```bash
root@pve:/etc/pve/lxc# cat 102.conf
arch: amd64
cores: 4
features: fuse=1,mount=nfs;cifs,nesting=1
hostname: docker
memory: 4096
net0: name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.253,hwaddr=BA:64:EF:29:04:A6,ip=192.168.1.22/24,type=veth
ostype: debian
rootfs: local-lvm:vm-102-disk-0,size=30G
swap: 512
lxc.apparmor.profile: unconfined
lxc.mount.auto: cgroup:rw
lxc.mount.auto: proc:rw
lxc.mount.auto: sys:rw
lxc.cap.drop:
lxc.cgroup.devices.allow: a
```

配置完成后就可以正常启动该容器了。

## 更换源

源文件路径：`/etc/apt/sources.list`,替换为下边的内容。

```bash
root@docker:/etc/apt# cat sources.list
deb https://mirrors.ustc.edu.cn/debian bookworm main contrib
deb https://mirrors.ustc.edu.cn/debian bookworm-updates main contrib
deb https://mirrors.ustc.edu.cn/debian-security bookworm-security main contrib
```

查看是否替换成功，执行命令apt update:

```bash
root@docker:/etc/apt# apt update
Hit:1 https://mirrors.ustc.edu.cn/debian bookworm InRelease
Get:2 https://mirrors.ustc.edu.cn/debian bookworm-updates InRelease [55.4 kB]
Get:3 https://mirrors.ustc.edu.cn/debian-security bookworm-security InRelease [48.0 kB]
Get:4 https://mirrors.ustc.edu.cn/debian-security bookworm-security/main amd64 Packages [148 kB]
Hit:5 https://download.docker.com/linux/debian bookworm InRelease
Fetched 251 kB in 1s (228 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
51 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

可见已经替换成功。

## SSH登录

默认情况下只能通过pve的控制台进行登录，无法在其他地方进行登录。

修改sshd的配置文件，文件路径：/etc/ssh/sshd_config,添加下边的内容：允许root登录，开启key登录：

```bash
PermitRootLogin yes
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
```

如果只想使用key登录，禁止密码登录可以再添加一行：`PasswordAuthentication no` 。根据自己需求添加。

## 时区设置

默认情况下是0时区：

```bash
root@docker:~# date
Sun Mar 24 07:04:09 UTC 2024
root@docker:~# date -R
Sun, 24 Mar 2024 07:04:11 +0000
```

修改为北京时间：

```bash
root@docker:~# timedatectl set-timezone Asia/Shanghai
root@docker:~# timedatectl
               Local time: Sun 2024-03-24 15:06:22 CST
           Universal time: Sun 2024-03-24 07:06:22 UTC
                 RTC time: n/a
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: inactive
          RTC in local TZ: no
```

可以看到已成功设置为北京时间了。

整个LXC容器已设置完成。
