---
title: 搭建属于自己的影视小站：小雅Alist安装及使用
author: 要名俗气
type: post
date: 2024-04-20T06:39:56+00:00
url: /2024/how-to-build-xiaoya-alist-and-use
description: 之前一直在使用其他的影视站去看一些电影，这些影视站基本网速都比较慢，画质也都一般，观看体验也不怎么好，这里顺便也分享一下我一直使用的影视站：影视森林，该站整理了一些比较不错的影视站。偶然的机会下了解到了小雅(xiaoya)Alist，搭配阿里云盘简直不要太爽，作者也在一直更新资源，非常推荐自己搭建，下边就简单介绍一下我的搭建过程以及一些使用。
image: https://images.iminling.com/app/hide.php?key=VTJuMzZadnFMdGQzWkp6ZFlHWk44dm1scFJnV0xZVXp4aGVnbHBCNmszQ2tqSU9sRkxTSmJNZlljSWtSb3pkWVdMdXVkQlk9
categories:
  - nas
tags:
  - alist
  - kodi
  - tvbox
  - 阿里云盘
---
之前一直在使用其他的影视站去看一些电影，这些影视站基本网速都比较慢，画质也都一般，观看体验也不怎么好，这里顺便也分享一下我一直使用的影视站：[影视森林](https://www.549.tv/)，该站整理了一些比较不错的影视站。偶然的机会下了解到了小雅(xiaoya)Alist，搭配阿里云盘简直不要太爽，作者也在一直更新资源，非常推荐自己搭建，下边就简单介绍一下我的搭建过程以及一些使用。

## 准备工作

在进行安装前需要先获取三个东西：token,open token以及folder id,下边介绍一下如何获取这几个信息。另外需要在机器上安装docker，因为小雅Alist是使用docker镜像来安装的，如果本机还没有安装docker,可以参考我的文章：[docker安装](https://www.iminling.com/2023/06/10/164.html "docker安装") 来安装docker。

### token

打开网站：<https://aliyuntoken.vercel.app/>，会出现二维码，使用手机端的阿里云盘扫描就可以获取到token。先记录下。

### open token

打开网站：[Get Aliyundrive Refresh Token](https://alist.nn.ci/tool/aliyundrive/request.html)，首先点击`Scan QrCode`获取二维码，然后使用手机端阿里云盘扫描，扫描完成后再点击`I have scan` 来获取`refresh token`。同时记录下这个token。

![aliyun refresh token](https://images.iminling.com/app/hide.php?key=MU5XZFJXZ09VR2xVZkt3dS8vNEVzY2JvK3pWR0xLckI4M0c3M1JTdG5ER2piOERla09Obnd0NGJmd0thUTNseFNNc3JvdXM9)

### folder id

打开小雅的分享地址：<https://www.alipan.com/s/rP9gP3h9asE>，将里边的内容转存到自己的网盘下，选择资源库。

![aliyun share save](https://images.iminling.com/app/hide.php?key=d0I5Y0JBQUVXSVBRRlVLcEF4TGEwU3k5ekxnVS85cXJTVU1iRHFDUStYNlRkMDhCcjZPdzF3WkFGSTlMQ0g3eGRubExvQjg9)

然后进入到自己的[阿里云盘](https://www.alipan.com/)，打开刚才转存的目录，看地址栏，最后那一串字符串就是需要的folder id.记录改folder id.

![xiaoya folder id](https://images.iminling.com/app/hide.php?key=TDBpdktOVmhIYXJKbDgxMk1uOFpBd3p3Zk85Ynd2Mm04VDZpNzllR0xOek5lb2Q4dlNTK0xiYmJ5eU4wc1A1TzZyZGN0VTA9)

经过上边几步，已经准备好了所需要的三个必须信息，下边就开始安装小雅Alist。

## 小雅Alist安装

安装有几种方式，小雅也有提供一键安装脚本，比较方便，下边我介绍一下几种安装方式。

### 一键脚本

这种方式最简单，在获取了上边的信息后，就可以直接在vps或者自己的电脑上进行安装，打开命令窗口(终端)执行一键安装脚本：bash -c "$(curl http://docker.xiaoya.pro/update_new.sh)"，执行后会提示输入token（32位）、open Token也就是刚才的refresh Token(335位)以及folder id(40位)，输入完成后就开始了安装，等待结束就可以了。通过docker 命令`docker ps`来查看安装情况,看是否有一个容器名称为`xiaoya`,存在就说明安装成功。

一键脚本安装的小雅默认配置文件是在`/etc/xiaoya`目录下，后续的配置文件都需要往这个文件下配置。

### docker命令

上边介绍了一键脚本，本质上也是使用docker命令来安装的，如果我们想要自己控制配置放在什么地址，可以把脚本下载下来自行修改，或者来进行docker的配置安装。

比如我想把配置文件放在`/data/xiaoya`下，那么我需要在`data/xiaoya`目录下先新建三个文件:

  1. `mytoken.txt`：把信息准备中的token字符串放在这个文件里。
  2. `myopentoken.txt`：把信息准备中的refresh Token字符串放在这个文件里
  3. `temp_transfer_folder_id.txt`：把folder id字符串放在这个文件里。

此时准备工作就完成了，可以来使用命令来安装小雅了：
`docker run -d --network=host -v /data/xiaoya:/data --restart=always --name=xiaoya xiaoyaliu/alist:hostmode`,等待安装完成，同样通过`docker ps`来查看是否启动成功，后续所有的配置文件都放在`/data/xiaoya`目录下。

### docker compose安装

这种是我最常用的方式，容器的配置都放在compose文件中，方便管理。和docker安装差不多，也是先需要将三个文件创建出来，目录结构如下：

```bash
root@docker:~/xiaoya# tree -l
.
├── data
│   ├── docker_address.txt
│   ├── myopentoken.txt
│   ├── mytoken.txt
│   └── temp_transfer_folder_id.txt
└── docker-compose.yaml
```

docker-compose.yaml文件内容如下：
```yaml
root@docker:~/xiaoya# cat docker-compose.yaml
version: '3.9'
services:
  xiaoya:
    image: xiaoyaliu/alist:latest
    container_name: xiaoya
    env_file:
      - .env
    restart: unless-stopped
    ports:
      - 5678:80
      - 2345:2345
      - 2346:2346
    volumes:
      - ./data:/data
    environment:
      - TZ=Asia/Shanghai
```

然后在`~/xiaoya`目录下执行 `docker compose up -d`启动就可以了。


经过上边的安装已经可以通过你的`机器ip:5678`来访问了。

![alist index](https://images.iminling.com/app/hide.php?key=ektqWVRFVFhvR1dxYVh3L2tER0pPcW81enFVTTdLZDNqWGRsVUdWV25ZYW0xL0thaTJEdFQwa3cyM3ZwNUcrTEsxYVZSWHM9)

## 播放

经过上边安装默认情况下已经可以正常使用了，在网页端找到自己想观看的电影，直接在网页端观看，或者在视频播放器下边的按钮里选择自己本地已安装的程序打开进行播放,例如下边有iina,VLC等播放器，自己在本地安装就可以了。

![alist play](https://images.iminling.com/app/hide.php?key=QUVpM0YwUDcxRG5IZGlISXhyU0pJY21VTGFMUzg2bE1qbGFXY1FFOXNIb2d4ZUlnWkticUJ5MEM1eWk3YjFiUHhyUWxlekE9)

也可以在小雅Alist中找到对应的安装包，也提供了很多播放器的安装包：

![alist software](https://images.iminling.com/app/hide.php?key=YnZOV01KZXNKZnlXbkZXMytqUFN4Yzc4eUxvUjJkOGpzY2JiZ05ZalB1V05UNjZSam5QZDhyMWpFUUxMQnY3Z0U0NDNON0E9)


### 安卓手机

想要在安卓手机上使用alist,我是使用tvbox来观看的，在小雅Alist提供的安装包里也是有tvbox的，需要在配置文件目录新建一个`docker_address.txt`文件，里边填写http://xxxxx:5678，网址最后不需要/, xxx替换为自己的vps的ip，或者搭建机器的内网ip。

安卓手机正常安装后进行配置：http://xxxxx:5678/tvbox/my.json。

![tvbox setting](https://images.iminling.com/app/hide.php?key=aC9La1lEV1pSUHhsK1ZWcHZPN1hzOTU5SlljWnZEaFkweUt6NW1PNk5naHZqR0VrMUNWWWhLdWl1c3RjTVNTNXJEeXVQMTg9)

如果同时有内网以及外网地址，那么想要在公网访问，则需要再多建一个`docker_address_ext.txt`,里边需要填写公网的`ip:端口`,然后在tvbox中进行配置:http://xxxxx:5678/tvbox/my_ext.json就可以了。


### 电视端

其实和安卓端是一样的，安装tvbox(安装包也是在小雅Alist中获取)，按照安卓的配置方式，进行配置后就可以观看了。

## 配置
### 登录配置

如果你搭建的机器是公网的，又不想别人随便使用，可以通过配置，需要使用账号和密码来登录，需要在配置文件目录添加两个文件:`guestlogin.txt`,和`guestpass.txt`两个文件，`guestlogin.txt`控制Alist网页端需要登录，只是一个空文件就可以了，`guestpass.txt`里边内容是登录密码，这样子访问你的Alist就需要账号和密码了，账号是：dav，密码是`guestpass.txt`中的内容。


其他配置暂时也没有使用到，后续有使用到再进行更新，祝大家都能顺利安装成功。
