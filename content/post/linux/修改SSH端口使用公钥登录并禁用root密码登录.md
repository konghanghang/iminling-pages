---
title: 修改SSH端口使用公钥登录并禁用root密码登录
author: 要名俗气
type: post
date: 2024-01-05T15:14:15+00:00
url: /2024/modify-ssh-port-and-use-publick-key-login
description: 不得不说这个世界上坏人还是比较多的，以为购买一台服务器后部署自己的服务后就可以高枕无忧了吗？NO，总有刁民想还朕，默认的SSH端口是22，这个是SSH协议的规范，那么大部分人购买了服务器后也不会想着去更改这个端口，那么就会被一些不法分子利用，服务器用户名默认root，端口默认22，那么他们就会通过这个来扫你的机器，不停的尝试登录，如果我们设置的密码强度不够，...
featured_image: /wp-content/uploads/2024/01/ssh-thumbnail.png
categories:
  - Linux
---
![SSH连接](https://www.iminling.com/wp-content/uploads/2024/01/SSH_Key_-_Authentication_Using_SSH_Keys-2.png)

不得不说这个世界上坏人还是比较多的，以为购买一台服务器后部署自己的服务后就可以高枕无忧了吗？NO，总有刁民想还朕，默认的SSH端口是22，这个是SSH协议的规范，那么大部分人购买了服务器后也不会想着去更改这个端口，那么就会被一些不法分子利用，服务器用户名默认root，端口默认22，那么他们就会通过这个来扫你的机器，不停的尝试登录，如果我们设置的密码强度不够，那么很快就会被穷举出来，然后你的服务器就被别人控制。今天我们就来做三步操作增强我们服务器的安全性。

## 配置公钥

### 生成密钥对

我们首先要做的就是生成一对密钥，可以使用ssh-keygen来生成一对，生成语法如下：

```
ssh-keygen -t rsa -f ~/.ssh/my-ssh-key -C [USERNAME]
```

其中

-t 指定算法，一般使用rsa非对称加密算法

-f 指定生成的密钥对存放的路径，可以不指定。

-C 为该密钥对分配一个用户名，只是为了分组使用，可以随意命名。

当我们执行生成命令时，如果不指定文件路径，默认会放在当前用户的.ssh目录下，并会为改密钥设置密码，默认不设置，直接回车跳过，经过这个命令就生成了一对密钥，公钥名称：id\_rsa.pub，私钥名称：id\_rsa

```
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/aaa/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/aaa/.ssh/id_rsa
Your public key has been saved in /Users/aaa/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:03Gf81aw0hniwCGSylV1VbOntYOrgsw1RicsUxWwKs aaa
The key's randomart image is:
+---[RSA 3072]----+
|      .oo.o ooB=o|
|      o. o o B.oo|
|   . o    + O.*.o|
|    o    . = XoOo|
|        S . +oOo.|
|         . Eo.ooo|
|           + . .o|
|            o  . |
|                 |
+----[SHA256]-----+

$ ls .ssh
config          id_rsa.pub
id_rsa          known_hosts
```

###  上传公钥

生成密钥后，我们需要做的是把公钥上传的需要登录的服务器上，这样子我们才能在自己本地或者其他服务器上使用私钥去登录到这台服务器。上传公钥有两种方式，一种是我们直接拷贝公钥里的内容到需要登录服务器的

```
~/.ssh/authorized_keys
```

文件中，如果文件不存在直接新建一个。另一种是直接使用ssh-copy-id命令来完成。

#### 直接上传文件内容

我这里使用scp命令把文件传到服务器上，然后在服务器上把内容写入到authorized_keys中。上传命令中端口是非必填的默认是22端口，如果没有修改端口可以添加此参数，后边的.pub文件就是你要上传的文件，后边紧跟着你服务器的用户名@ip:具体服务器路径

```
# scp [-P 端口] ./my-ssh-key.pub [USERNAME]@[EXTERNAL_IP_ADDRESS]:/root/.ssh
scp ./my-ssh-key.pub root@192.168.31.1:/root/.ssh

# 上传成功后，ssh登录到服务器上，把上传的pub文件追加到.ssh目录下的authorized_keys中。
$ ssh root@192.168.31.1
$ cd .ssh
$ cat my-ssh-key.pub >> ~/.ssh/authorized_keys
```

经过上边步骤就上传好了pub公钥了，就可以使用key来登录了。下边介绍一下使用ssh-copy-id来上传。

#### ssh-copy-id上传

这个上传就很简单了，直接使用命令：

```
# ssh-copy-id -i ./my-ssh-key.pub [USERNAME]@[EXTERNAL_IP_ADDRESS] [-p 端口]
$ ssh-copy-id -i ./my-ssh-key.pub root@192.168.31.1
```

-i 指定要上传的文件路径，

-p 指定端口，默认22，如果没修改过ssh端口就直接不需要该参数

#### 公钥登录

在登录前要先确认要登录的服务器是否开启了支持公钥登录，其实现在大部分服务器都是默认支持的，如果遇到登录不了的情况可以检查一下是否开启，具体的文件路径是/etc/ssh/sshd_conf,设置以下参数为yes：

```
PasswordAuthentication yes　　　　　　# 口令登录
RSAAuthentication yes　　　　　　　　　# RSA认证
PubkeyAuthentication yes　　　　　　　# 公钥登录
```

如果有修改需要重启一下服务器的sshd服务：

```
service sshd restart
```

然后就可以在本地或者其他服务器使用ssh并指定要使用的私钥文件目录来连接这个远程的服务器了：

```
# ssh -i .ssh/my-ssh-key [USERNAME]@[EXTERNAL_IP_ADDRESS] [-p 端口]
ssh -i .ssh/my-ssh-key root@192.168.31.1
```

经过上边步骤就可以使用公钥连接到服务器了。

## 修改SSH端口

默认的ssh端口是22，是在/etc/ssh/sshd_conf中配置的，在这个配置文件中我们先找到原来的配置22端口，然后复制一行出来，改成另外一个端口，比如我改成1024端口，那么此时22端口和1024端口应该都是可以通过SSH来访问的，这里一定要小心，不能一上来就把22端口干掉，万一配置错误就凉凉了。所以配置两个端口，保证22端口是肯定可以使用的，然后再添加一个新的端口，配置后重启sshd服务。然后看新配置的端口是否可以访问SSH，如果可以那说明配置没有问题，就可以干掉原来的22端口了。

```
Port 22
Port 1024
```

配置的属性就叫Port。

## 禁用密码登录

上边已经配置了使用公钥登录，并且也修改了SSH的端口，但是root用户的密码还在，那么一旦一些人扫到了服务器的1024端口是ssh，那么他们还是会不停的尝试密码，暴力破解密码，所以需要把密码登录关掉，只用公钥来登录。同样也是修改/etc/ssh/sshd_conf，修改PasswordAuthentication为no就可以了：

```
PasswordAuthentication no 　　　　　　# 口令登录
RSAAuthentication yes　　　　　　　　　# RSA认证
PubkeyAuthentication yes　　　　　　　# 公钥登录
```

经过上边的配置就可以禁用掉root用户的密码登录功能，并修改了SSH登录的端口，这样子服务器的安全性就提高了一些。

## 遇到的问题

在第一步上传公钥到服务器的时候，上传成功了，但是始终无法通过公钥去登录到服务器，研究了好久发现是服务器上的目录权限不正确，正确的权限应该是下边这样子的:

```
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys
```

