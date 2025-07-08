---
title: linux双栈网络设置IPv4优先
author: 要名俗气
type: post
date: 2023-09-03T02:58:44+00:00
url: /2023/linux-use-ipv4-first
description: 双协议栈技术就是指在一台设备上同时启用 IPv4 协议栈和 IPv6 协议栈，这样就可以同时使用 IPv4 和 IPv6 的网络。但是在默认情况下都会以IPv6网络优先，只有 IPv6 无法访问的时候才会尝试访问 IPv4，某些特定的应用和场景下，我们并不想要 IPv6 优先，这时候就需要修改一些配置文件让 IPv4 优先。
image: https://images.iminling.com/app/hide.php?key=cU9DUVBIUnEzWktLQnpML0x0b090RkVtOGRNa0VnNXpMMFNSSFZKcTNsZkQ3eXdWNDlnZS9pNWZvNHFXdVNZV0JZVVpBREE9
categories:
  - linux
tags:
  - debian
  - IPv4
  - IPV6
  - linux
---
双协议栈技术就是指在一台设备上同时启用 IPv4 协议栈和 IPv6 协议栈，这样就可以同时使用 IPv4 和 IPv6 的网络。但是在默认情况下都会以IPv6网络优先，只有 IPv6 无法访问的时候才会尝试访问 IPv4，某些特定的应用和场景下，我们并不想要 IPv6 优先，这时候就需要修改一些配置文件让 IPv4 优先。

## 查看网络优先级

可以通过以下命令来查看当前的机器是以哪个网络优先的：

```bash
root@gc:~# curl ip.sb
2000::97e4
```

如上返回了IPv6的地址，所以该机器默认就是通过IPv6来访问网站的，通过配置curl强制使用IPv4协议来进行访问：

```bash
root@gc:~# curl ip.sb -4
192.168.0.235
```

接下来就来修改配置使IPv4成功默认的出栈协议。

## 修改/etc/gai.conf

在 Debian、Ubuntu 等 Linux 系统下，有一个 `/etc/gai.conf` 文件，用于系统的 `getaddrinfo` 调用，默认情况下，它会使用 IPv6 优先，编辑gai.conf文件，会看到以下配置：

```bash
# precedence  <mask>   <value>
#    Add another rule to the RFC 3484 precedence table.  See section 2.1
#    and 10.3 in RFC 3484.  The default is:
#
#precedence  ::1/128       50
#precedence  ::/0          40
#precedence  2002::/16     30
#precedence ::/96          20
#precedence ::ffff:0:0/96  10
#
#    For sites which prefer IPv4 connections change the last line to
#
#precedence ::ffff:0:0/96  100
```

里边有一行提示：`For sites which prefer IPv4 connections change the last line to`，想要IPv4出栈，可以设置：`precedence ::ffff:0:0/96  100`

，根据提示把该行前的注释去掉，如下：

```bash
# precedence  <mask>   <value>
#    Add another rule to the RFC 3484 precedence table.  See section 2.1
#    and 10.3 in RFC 3484.  The default is:
#
#precedence  ::1/128       50
#precedence  ::/0          40
#precedence  2002::/16     30
#precedence ::/96          20
#precedence ::ffff:0:0/96  10
#
#    For sites which prefer IPv4 connections change the last line to
#
precedence ::ffff:0:0/96  100
```

修改后保存，然后再使用curl访问网站，不指定使用的IPv4还是IPv6，默认情况下就走了IPv4，如下：

```bash
root@gc:~# curl ip.sb -4
192.168.0.235
```

修改gai.conf后就默认走了IPv4协议了。

## 禁用IPv6

有一些极端情况下，我们可能需要禁止系统的 IPv6 功能，这时候就需要修改 `/etc/sysctl.conf` 文件，首先找到你的网卡名称，默认是 `eth0` ，可以通过以下命令查看自己的网卡名：

```bash
root@gc:~# ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq state UP group default qlen 1000
    link/ether 00:16:3c:6b:b7:c3 brd ff:ff:ff:ff:ff:ff
    altname enp0s3
    altname ens3
    inet 192.168.0.235/24 brd 192.168.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 2000::97e4/112 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3cff:fe6b:b7c3/64 scope link
       valid_lft forever preferred_lft forever
```

我的网卡名就是eth0，然后修改`/etc/sysctl.conf`，加入一下内容(将网卡名替换为自己的)：

```bash
net.ipv6.conf.all.autoconf = 0
net.ipv6.conf.default.autoconf = 0
net.ipv6.conf.all.accept_ra = 0
net.ipv6.conf.default.accept_ra = 0
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
net.ipv6.conf.eth0.disable_ipv6 = 1
```

然后使用 `sysctl -p` 来重新加载配置文件，此时查看 `ip a` 就可以发现 IPv6 已经被禁止了。

以上就是双栈网络的机器设置IPv4网络优先的方法，自己测试了Debian和Ubuntu都是可以实现的，希望可以帮到大家。

