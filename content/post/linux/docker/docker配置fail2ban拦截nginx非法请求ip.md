---
title: docker配置fail2ban拦截nginx非法请求ip
author: 要名俗气
type: post
date: 2024-02-05T13:12:33+00:00
url: /2024/docker-install-fail2ban-interception-nginx-ip
description: docker安装fail2ban并配置防止ssh登录爆破，fail2ban能做到的远不止这些，如果你有一个属于自己的网站，那么一定会遇到有无数不知明ip来访问你网站的敏感路径，总是想着要来获取你网站的一些有用信息。今天我们就来使用fail2ban去拦截那些非法访问的ip地址。 nginx-cc,filter.d和jail.d中的配置文件名称一致。
image: https://images.iminling.com/app/hide.php?key=S2lnNFVkM3BpS1h5b1RvRlF3V0tocVFuZ3YrTHQ3YWhJQXp6YnNJdUZlenRpSTZOdmF2b0M0MTJpR2VoaVU1eG1PNTZ6Z1E9
categories:
  - docker
tags:
  - fail2ban
  - nginx
---
前边写过一篇通过fail2ban来拦截ssh暴力破解：[docker安装fail2ban并配置防止ssh登录爆破]({{< ref "/post/linux/docker/docker安装fail2ban并配置防止ssh登录爆破.md" >}})，fail2ban能做到的远不止这些，如果你有一个属于自己的网站，那么一定会遇到有无数不知明ip来访问你网站的敏感路径，总是想着要来获取你网站的一些有用信息。今天我们就来使用fail2ban去拦截那些非法访问的ip地址。


## 环境与目录情况

fail2ban如上篇：中的安装方式，是使用docker来安装的
nginx也是使用docker来安装的，并使用的是桥接网络。
然后再来看一下配置的目录：
```
ubuntu@VM-20-3-ubuntu:~/images/fail2ban$ tree
.
├── data
│   ├── filter.d
│   │   └── nginx-cc.conf
│   └── jail.d
│       ├── nginx-cc.conf
│       ├── sshd.conf
├── docker-compose.yaml
├── .env
└── logs
```


如上，需要创建两个文件，一个是针对非法访问的日志的过滤规则，另一个是针对非法访问的ip的监狱规则。我这里暂定这个监狱的名称就叫做：`nginx-cc`,`filter.d`和`jail.d`中的配置文件名称一致。

## 过滤配置

先来看一下过滤规则的配置：

```
[Definition]
failregex = ^<HOST> .* "(GET|POST|HEAD).*HTTP.*" (404|503) .*$
ignoreregex =.*(robots.txt|favicon.ico|jpg|png)
```

`failregex`配置的就是针对错误访问日志进行过滤的正则表达试，以ip开头，并把ip使用`<HOST>`代替。这里是针对访问状态是404和503的请求。
`ignoreregex`配置了针对什么样的路径日志进行忽略。

以上是nginx默认日志的访问日志匹配的正则，如果你修改了自己的nginx的访问日志输出格式，则需要按自己的日志输出格式来编写失败请求的正则表达式。

## 监狱配置
直接先看一下监狱的具体配置：

```
[nginx-cc]
enabled = true
filter = nginx-cc
chain = DOCKER-USER
#action = iptables-multiport[name=nginx-cc, port="http,https", protocol=tcp]
port = http,https
logpath = /var/logs/nginx/access.log
maxretry = 3
bantime = 86400
findtime = 3600
ignoreip = 192.168.0.1/24
```

下边详细说明一下各参数：
`filter`对应的就是上边的过滤规则中的名称
`chain`为`DOCKER-USER`,因为nginx是使用docker安装的，所以需要把iptables的规则添加到`DOCKER-USER`中，`DOCKER-USER`中的规则在Docker自动创建的任何规则之前加载，拥有更高的优先级。
`port`满足添加是禁止的端口
`logpath`读取的日志路径
`findtime`日志文件中分析错误连接数的时间。以秒计算


## 启动配置
因为以前已经配置过sshd的监狱，并且fail2ban也是正常运行的，此时只需要对配置进行重新加载就可以了，可以使用以下命令：

```
ubuntu@VM-20-3-ubuntu:~/images/fail2ban$ docker exec fail2ban fail2ban-client reload
OK
```

再来查看当前生效的监狱有哪些：

```
ubuntu@VM-20-3-ubuntu:~/images/fail2ban# docker exec fail2ban fail2ban-client status
Status
|- Number of jail:      2
`- Jail list:   nginx-cc, sshd
```
可以看到`nginx-cc`监狱已经生效了，再来看一下这个监狱的详细情况：

```
ubuntu@VM-20-3-ubuntu:~/images/fail2ban# docker exec fail2ban fail2ban-client status nginx-cc
Status for the jail: nginx-cc
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     134
|  `- File list:        /var/logs/nginx/access.log
`- Actions
   |- Currently banned: 0
   |- Total banned:     21
   `- Banned IP list:
```
可以看到鉴于的详细信息，此时不出意外就可以拦截到那些异常情况了。

## 遇到的问题

不出意外，肯定会出意外。我在debian11的机器上安装，请求404的url会被记录到`Total failed`里，并且请求失败的ip也会出现在`Banned Ip list`里，但是被ban后竟然还可以访问，此时通过查看`fail2ban`容器发现竟然添加iptables规则的时候出现了异常。

![fail2ban add fail](https://images.iminling.com/app/hide.php?key=Zk9mSUhyREttOTBtYU45dXlrNEZwclE1NW9NTVpkeGt4WG5tQU5mdndsQkxLYVhuRFNUbHBLR3ZjTGZmbWE4a3RmWTNzUk09)

如上图，出现`iptables: No chain/target/match by that name` 错误，导致iptables规则添加失败，通过`iptables -nvL`的确也看不添加的被禁的ip。在网上找了好久不知道是什么原因，后来发现官方的github仓库的说明里就有关于这个的说明[crazy-max/docker-fail2ban](https://github.com/crazy-max/docker-fail2ban?tab=readme-ov-file#use-iptables-tooling-without-nftables-backend)：

> If your system's `iptables` tooling uses the nftables backend, this will throw the error `stderr: 'iptables: No chain/target/match by that name.'`. You need to switch the `iptables` tooling to 'legacy' mode to avoid these problems. This is the case on at least Debian 10 (Buster), Ubuntu 19.04, Fedora 29 and newer releases of these distributions by default. RHEL 8 does not support switching to legacy mode, and is therefore currently incompatible with this image.

原来是因为iptable的模式问题，默认debian11使用的`nf_tables`模式,按照官方的提示需要切换为`legacy`模式，如何查看自己的iptables是哪种模式呢？可以使用`iptables --version`来查看，如下就使用的是`nf_tables`模式。

```
ubuntu@VM-20-3-ubuntu:~# iptables --version
iptables v1.8.7 (nf_tables)
```

那如何修改成`legacy`模式呢？官方也给了解决方案：

On Ubuntu or Debian：

```
$ update-alternatives --set iptables /usr/sbin/iptables-legacy
$ update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
$ update-alternatives --set arptables /usr/sbin/arptables-legacy
$ update-alternatives --set ebtables /usr/sbin/ebtables-legacy
```

On Fedora:

```
$ update-alternatives --set iptables /usr/sbin/iptables-legacy
```

我的系统是debian，所以按照上边的四条来执行，后边两条并没有执行成功，提示没有开启，那就不管了。执行完后可以再使用上边的version命令查看模式是否已经改过来了:

```
ubuntu@VM-20-3-ubuntu:~/images/fail2ban# iptables --version
iptables v1.8.7 (legacy)
ubuntu@VM-20-3-ubuntu:service networking restart
```

可以看到，已经切换成功了，并且重启了`networking`服务。此时再去看禁止ip发现还是不行，仍然可以访问，**官方建议重启**，我以为重启`networking`就可以了，但是并不行，最后只好**对服务器进行了重启**，最终终于生效，成功的对异常访问ip进行了禁止。

以上就是折腾过程，希望可以帮助到大家。
