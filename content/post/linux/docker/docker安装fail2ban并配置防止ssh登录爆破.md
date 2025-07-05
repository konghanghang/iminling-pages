---
title: docker安装fail2ban并配置防止ssh登录爆破
author: 要名俗气
type: post
date: 2024-02-03T14:52:53+00:00
url: /2024/docker-install-fail2ban-protect-ssh
description: /var/log/auth.log，一看整个人都呆了，登录错误的日志不停的刷，都不带停的。如果密码设置的简单点，估计早就被人给试出来了。在网上查询到可以使用fail2ban来进行防御，把经常访问出错的ip加入到iptables中，禁止掉它的访问，于是就开干了，开始摸索fail2ban如何安装使用。
featured_image: /wp-content/uploads/2024/02/fail2ban-logo.jpg
categories:
  - fail2ban
  - Linux
tags:
  - fail2ban
  - linux
  - ssh
---
<div>
  ![fail2ban install](https://www.iminling.com/wp-content/uploads/2024/02/Install-and-configure-Fail2ban-on-Debian-10-11-Linux-server-min.jpg)
</div>

<div>
  平时也没有怎么关注过ssh的日志，而且服务器的ssh登录端口也是默认的22，只是密码设置的相对来说复杂了点，后边听说有人会进行密码爆破，于是就查看了一下ssh的日志，日志路径是<code>/var/log/auth.log</code>，一看整个人都呆了，登录错误的日志不停的刷，都不带停的。如果密码设置的简单点，估计早就被人给试出来了。在网上查询到可以使用fail2ban来进行防御，把经常访问出错的ip加入到iptables中，禁止掉它的访问，于是就开干了，开始摸索fail2ban如何安装使用。
</div>

<div>
  ```
Feb  3 15:04:36 localhost sshd[3030796]: pam_unix(sshd:auth): check pass; user unknown
Feb  3 15:04:36 localhost sshd[3030796]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=170.64.155.113
Feb  3 15:04:37 localhost sshd[3030785]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=104.250.50.16  user=root
Feb  3 15:04:38 localhost sshd[3030796]: Failed password for invalid user es from 170.64.155.113 port 56728 ssh2
Feb  3 15:04:39 localhost sshd[3030796]: Connection closed by invalid user es 170.64.155.113 port 56728 [preauth]
Feb  3 15:04:39 localhost sshd[3030785]: Failed password for root from 104.250.50.16 port 57284 ssh2
```
</div>

## 安装fail2ban

<div>
  <div>
    我这里使用docker进行安装fail2ban,使用的docker镜像地址：<a href="https://github.com/crazy-max/docker-fail2ban">crazy-max/docker-fail2ban,</a>目录如下所示：
  </div>

  <div>
    ```
ubuntu@VM-20-3-ubuntu:~/images/fail2ban$ tree
.
├── data
│   └── jail.d
│       └── sshd.conf
├── docker-compose.yaml
├── .env
└── logs
```
</div>

<h3>
  docker-compose.yaml文件
</h3>

<div>
  <div>
    先看一下docker-compose.yaml的内容：
  </div>

  <div>
    ```
version: "3"
services:
  fail2ban:
    image: crazymax/fail2ban:latest
    container_name: fail2ban
    restart: unless-stopped
    network_mode: "host"
    cap_add:
      - NET_ADMIN
      - NET_RAW
    env_file:
      - .env
    volumes:
      - ./data:/data
      - ./logs/:/var/log/
      # sshd 日志映射
      - /var/log:/var/logs/sshd:ro
    logging:
      driver: "json-file"
      options:
        max-size: "5m"
        max-file: "10"
```

<div>
  <div>
    这里说明两点，首先是网络模式使用的是<code>host</code>模式，第二，因为我们是要检测暴力破解sshd登录的，所以我们需要读取ssh登录的日志，日志文件路径是<code>/var/log/auth.log</code>,所以在卷挂载那里就把ssh日志的目录进行挂载。
  </div>

  <h3>
    配置监狱拦截
  </h3>

  <div>
    <div>
      <code>fail2ban</code>中是需要定义监狱(jail)来对访问者进行过滤，每个监狱(jail)可以对应我们的一个服务或者服务的某一个维度，比如我们要对ssh的登录这个场景进行监控，那我们就可以定义一个名称为<code>sshd.conf</code>的监狱配置，这里也是参考github上的示例：<a href="https://github.com/crazy-max/docker-fail2ban/blob/master/examples/jails/sshd/jail.d/sshd.conf">sshd.conf</a>,内容如下：
    </div>

    <div>
      ```
[sshd]
enabled = true
chain = INPUT
# port = ssh
port = 22
filter = sshd[mode=aggressive]
logpath = /var/logs/sshd/auth.log
maxretry = 3
bantime = 86400 # 禁止一天，单位秒
```

<div>
  <div>
    <code>logpath</code>的配置，因为在docker-compose.yaml中对ssh日志的目录进行了挂载配置，这里根据自己的需要进行修改
  </div>

  <div>
    <code>maxretry</code>也修改为3次，严格一点
  </div>
</div>

<div>
</div>

<h2>
  启动拦截
</h2>

<div>
  <div>
    首先启动容器：
  </div>
</div>

```
ubuntu@VM-20-3-ubuntu:~/images/fail2ban$ docker compose up -d
[+] Running 4/4
 ⠿ fail2ban Pulled                  36.8s
   ⠿ 7264a8db6415 Pull complete     8.6s
   ⠿ 81330df9fd42 Pull complete     9.4s
   ⠿ 39280c79d189 Pull complete     9.5s
[+] Running 1/1
 ⠿ Container fail2ban  Started      0.6s
```

<div>
  <div>
    查看当前fail2ban的状态，如下，可以看到有一个监狱：<code>sshd</code>正在生效中:
  </div>

  <div>
    ```
ubuntu@VM-20-3-ubuntu:~/images/fail2ban$ docker exec fail2ban fail2ban-client status
Status
|- Number of jail:      1
`- Jail list:   sshd
```

<div>
  <div>
    然后再来查看一下sshd监狱的状态，可以看到总共出现了53次错误登录，当前已经拦截了6个ip，信息很详细：
  </div>

  <div>
    ```
ubuntu@VM-20-3-ubuntu:~/images/fail2ban$ docker exec fail2ban fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 6
|  |- Total failed:     53
|  `- File list:        /var/logs/sshd/auth.log
`- Actions
   |- Currently banned: 6
   |- Total banned:     6
   `- Banned IP list:   112.213.110.8 196.189.21.247 104.250.50.16 175.144.208.9 183.56.193.178 141.98.11.90
```

<div>
  <div>
    被ban的ip会添加到iptables中，并且fail2ban会主动为每个监狱创建一个Chain,Chain的名称为<code>f2b-监狱名</code>，可以通过命令来查看iptables中添加的规则:
  </div>
</div>
</div>
</div>

```
ubuntu@VM-20-3-ubuntu:~/images/fail2ban$ sudo iptables -nvL f2b-sshd
Chain f2b-sshd (1 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 REJECT     all  --  *      *       183.56.193.178       0.0.0.0/0            reject-with icmp-port-unreachable
    3   156 REJECT     all  --  *      *       175.144.208.9        0.0.0.0/0            reject-with icmp-port-unreachable
    1    60 REJECT     all  --  *      *       104.250.50.16        0.0.0.0/0            reject-with icmp-port-unreachable
    3   180 REJECT     all  --  *      *       196.189.21.247       0.0.0.0/0            reject-with icmp-port-unreachable
    0     0 REJECT     all  --  *      *       112.213.110.8        0.0.0.0/0            reject-with icmp-port-unreachable
  354 24037 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0
```

<div>
  <div>
    可以看到很多ip已经被ban了，这是我才刚启动1分钟不到，就这么多ip被ban了，可见坏人真多，老是想着爆破我的服务器。
  </div>

  <h2>
    fail2ban的使用
  </h2>

  <div>
    <div>
      使用<code>docker exec <容器名称> fail2ban-client</code>命令来操作fail2ban。
    </div>

    <h3>
      查看状态
    </h3>

    <div>
      <div>
        一个是查看当前总状态：启用了什么监狱，另一个是查看某个监狱的状态，上边已经展示过了，这里就不再详细说了：
      </div>

      <div>
        ```
# 查看状态
docker exec fail2ban fail2ban-client status

# 查看指定监狱状态
docker exec fail2ban fail2ban-client status sshd
```
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>
</div>

<h3>
  主动屏蔽ip
</h3>
</div>

<div>
  <div>
    我们也可以主动的屏蔽某个ip的访问，通过下边命令：
  </div>

  <div>
    ```
# 指定监狱屏蔽IP（<JAIL>为监狱名）
docker exec fail2ban fail2ban-client set <JAIL> banip <IP>
# 示例：docker exec fail2ban fail2ban-client set sshd banip 1.1.1.1
ubuntu@VM-20-3-ubuntu:~$ docker exec fail2ban fail2ban-client set sshd banip 183.56.193.179
1
```

<div>
  <div>
    然后再通过查看监狱状态命令查看`Banned Ip List`中是否已经有了这个ip来判断是否成功。
  </div>

  <h3>
    主动解除已屏蔽ip
  </h3>

  <div>
    <div>
      有主动的去屏蔽某个ip，如果设置错误就需要主动把设置错误的取消掉，可以通过下边的命令来进行取消：
    </div>

    <div>
      ```
# 指定监狱解封IP（<JAIL>为监狱名）
docker exec fail2ban fail2ban-client set <JAIL> upbanip <IP>
# 示例：docker exec fail2ban fail2ban-client set sshd upbanip 1.1.1.1
ubuntu@VM-20-3-ubuntu:~$ docker exec fail2ban fail2ban-client set sshd unbanip 183.56.193.179
1
```

<div>
  <div>
    然后再通过查看监狱状态的命令查看ip是否已经被取消掉。
  </div>

  <div>
  </div>
</div>
</div>

<div>
  <div>
    <div>
      以上就是通过安装fail2ban来对暴力破解ssh密码进行拦截，另外也可以通过修改ssh的端口来进行简单的拦截，通常默认的ssh端口是22，如果换了端口他们想要找到这个ssh端口也是要花费一些时间的，再就是可以通过禁用密码只使用key来登录，会更安全一些，这个可以参考我的另一篇文章：<a title="修改SSH端口使用公钥登录并禁用root密码登录" href="https://www.iminling.com/2024/modify-ssh-port-and-use-publick-key-login">修改SSH端口使用公钥登录并禁用root密码登录</a>。希望可以帮到大家。
    </div>
  </div>
</div>

<div>
</div>
</div>
</div>
</div>
</div>