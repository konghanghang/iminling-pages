---
title: linux服务器快速安装docker和docker-compose
author: 要名俗气
type: post
date: 2023-06-09T21:49:42+00:00
url: /2023/linux-install-docker
description: docker可以快速的构建我们的应用，并且应用之间的环境相互隔离，非常方便，本文就ubuntu和debian下安装docker和docker compose进行简单的记录。
image: https://images.iminling.com/app/hide.php?key=dnp6YVlCOXUzcXZ3NHRyMlZGVEZDYUY3c3h6eW9ET21TY3YrWWwrZTZBcllvQU1ZQkIxWXlaQnRpRVR5cml3VDVIRUJUTFk9
categories:
  - docker
tags:
  - debian
  - docker
  - linux
  - ubuntu

---
记录一下安装docker的过程，具体的安装步骤都是参考docker的官方网站进行的安装。

## ubuntu安装

### 卸载旧版

执行下边的命令

```bash
ubuntu@VM-20-3-ubuntu:~$ sudo apt-get remove docker docker-engine docker.io containerd runc
Reading package lists... Done
Building dependency tree
Reading state information... Done
E: Unable to locate package docker-engine
```


执行上边的命令清除以前的安装。

### 设置源

执行以下命令：

```bash
ubuntu@VM-20-3-ubuntu:~$ sudo apt-get update
ubuntu@VM-20-3-ubuntu:~$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
# 添加docker官方GPG key
ubuntu@VM-20-3-ubuntu:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```




# 执行
```bash
ubuntu@VM-20-3-ubuntu:~$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list &gt; /dev/null
```

设置安装源。



### 安装

先更新刚才安装的源，然后就可以执行安装了：

```bash
# 更新
sudo apt-get update
# 安装
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```




上边的安装是安装最新版本，如果想安装特定版本则需要先查看有哪些可以安装的版本：

```bash
ubuntu@VM-20-3-ubuntu:~$ apt-cache madison docker-ce
 docker-ce | 5:20.10.16~3-0~ubuntu-focal | https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
 docker-ce | 5:20.10.15~3-0~ubuntu-focal | https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
 docker-ce | 5:20.10.14~3-0~ubuntu-focal | https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
 docker-ce | 5:20.10.13~3-0~ubuntu-focal | https://download.docker.com/linux/ubuntu focal/stable amd64 Packages
```




找到版本后执行以下命令来进行安装：

```bash
# 版本是上个命令输出的第个列5:20.10.16~3-0~ubuntu-focal
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```




安装后就可以使用了。

&nbsp;

### 不使用sudo

ubuntu的系统如果使用的不是root安装，每次运行命令都是需要使用sudo,相对来说是比较麻烦的，如果不想使用sudo，可以使用下面的命令来解决：

```bash
ubuntu@VM-20-3-ubuntu:~$ sudo groupadd docker
[sudo] password for root:
groupadd: group 'docker' already exists
ubuntu@VM-20-3-ubuntu:~$ sudo gpasswd -a ${USER} docker
Adding user ubuntu to group docker
ubuntu@VM-20-3-ubuntu:~$ newgrp - docker
```




这样子使用起来就方便了很多。

&nbsp;

## debain安装

前两步和ubuntu基本一样，删除旧版，然后设置源。

### 删除旧版以及更新apt

```bash
# 删除旧版
root@s9707 ~ # apt-get remove docker docker-engine docker.io containerd runc
# 更新apt包索引
root@s9707 ~ # apt-get update
```



### 添加gpg

执行以下命令添加gpg

```bash
root@s9707 ~ # mkdir -p /etc/apt/keyrings 
root@s9707 ~ # curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
```




如果在添加gpg的时候报错：`gpg: command not found`

则可以先安装gpg：

```bash
root@s9707:~# apt-get install gnupg gnupg2
```

&nbsp;

### 设置仓库

通过以下命令添加仓库

```bash
root@s9707:~# echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list &gt; /dev/null
```




然后再次更新apt包索引：

```bash
root@s9707 ~ # apt-get update
```



### 安装docker

接下来就是安装：

```bash
apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```



整个安装过程也不是很麻烦，跟着官方的步骤走就可以了，这里记录一下自己的安装过程，方便以后快速安装。

&nbsp;

&nbsp;

&nbsp;