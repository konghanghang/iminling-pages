---
title: 私人影院之emby安装配置以及通过embyExternalUrl实现302播放
author: 要名俗气
type: post
date: 2025-01-04T09:48:58+00:00
url: /2025/emby-install-and-configuration
description: 前两篇文章先讲了如何安装alist并添加常见网盘，以及如何把alist通过rclone挂载在本地硬盘实现像本地文件一样进行浏览，本篇文章咱们继续，介绍如何安装emby并对媒体进行添加进emby. 
image: https://images.iminling.com/app/hide.php?key=QStIZ0I0Y3k4RmJmL1lZY3lLbjZDS0hqVkcxQjRVU21iZnlwaTJqRk83cm9VTE1EZzg2bmp6b2ErcmM4RkpsVUJDa3h4TEk9
categories:
  - nas
tags:
  - 115网盘
  - alist
  - emby
  - rclone
  - webdav
---
前两篇文章先讲了如何[安装alist并添加常见网盘]({{< ref "/post/nas/video/私人影院之搭建自己的alist服务端并添加常见网盘.md" >}})，以及如何[把alist通过rclone挂载在本地硬盘]({{< ref "/post/nas/video/私人影院之rclone安装以及通过WebDav配置alist.md" >}})实现像本地文件一样进行浏览，本篇文章咱们继续，介绍如何安装emby并对媒体进行添加进emby.

## emby安装

要安装`emby`首先要面临的就是`emby`版本选择问题，`emby`本身是一个收费软件，但是也有大佬们的破解版本，我这里选择`amilys/embyserver` 这个版本，可以去[docker hub官方地址](https://hub.docker.com/r/amilys/embyserver)查看。确认镜像后，下边给出我的`docker-compose.yaml`配置文件:

```yaml
networks:
  mynet:
    external: true
services:
  emby:
    image: amilys/embyserver
    container_name: emby
    restart: unless-stopped
    volumes:
      - ./config:/config
      - /data/movies:/data/movies:rslave
    ports:
      - 8096:8096
    extra_hosts:
      - host.docker.internal:host-gateway
    networks:
      - mynet
```

network不再多说了，自己创建的一个桥接网络。

端口暴露`8096`出去，

`emby`的配置文件在`/config`目录，把它挂载到本地方便后续进行修改。

另外一个挂载路径就是本地的`/data/movies`挂载到容器的`/data/movies`目录。本地的`/data/movies`其实就是[上篇文章通过rclone处理的alist中所有的文件]({{< ref "/post/nas/video/私人影院之rclone安装以及通过WebDav配置alist.md" >}})。这里建议设置为一样的路径，方便后续在302播放的时候替换，不会遇到那么多麻烦。

整体目录如下：

```bash
root@docker:~/emby# tree -L 2
.
├── config
└── docker-compose.yaml
```

然后就是启动容器`docker compose up -d`进行`emby`的配置了。

## emby初始化

启动容器后，可以通过`ip:8096`来访问刚启动的容器。下边来进行`emby`的一些初始化配置，首先是进行语言配置，选择`chinese simplified`(简体中文)，下一步

![welcome to emby](https://images.iminling.com/app/hide.php?key=bjNUT2U1c1QvOHpVWDc5ZEJMQ3JiZ2U1VkxReVdyQ3BMazhUMndTV3ZqTVo2a1BXRlMwQ2s0M2JseGVRaUg2Ym1zZ0ZCd2M9)

设置完语言后就会提示创建用户,根据自己的需求创建用户以及设置用户名和密码。然后继续下一步

![emby create first user](https://images.iminling.com/app/hide.php?key=ZUs0RHZNQVJ3SEE2RDlYcFZ3eTZ4ZHBoMzhzUVNPTWJPQnA5VG81MjRPbmxKbWU3YVlvU1JDRjhMREg0MmJjTnMra2dTa289)

之后就会让添加一个媒体库，这里可以先不添加，直接下一步，等完全配置好了之后再去配置媒体库相关的东西

![emby setup media libraries](https://images.iminling.com/app/hide.php?key=NjF0QnBxRmdkbUZmdlBtcG9tekZiRUJxTWZaMVVQS2YzbEJWa3Z6L0FyY1lzY1BoQ3Nlek42Y3NrdXFuZFVtSFpuZUk1WWs9)

接下来是设置媒体库的偏好语言和地区，根据自己需求英文或者中文

![emby preferred metadata language](https://images.iminling.com/app/hide.php?key=S0E0b2ZQVnJLWWFPL0lQRmZxd05LUXE1S3VSUWRTajRCczdJLzl6YWZXWnVycDNwV0FIK3hkOGd6V050Z3M1VkJoYVdVYWs9)

下边是配置远程访问，开启就好了

![emby configure remote access](https://images.iminling.com/app/hide.php?key=QUVpM0YwUDcxRG5IZGlISXhyU0pJZUFVNS9iaGJVZ2hXZWluQXRxcjBvWjUxdHRadjNDbHovWWRqSHhxMUtuRHl3UitoNzg9)

下边是使用条例，接收继续下一步

![emby terms of use](https://images.iminling.com/app/hide.php?key=QlNvbkV2U2kyTnhrcnFzZk5xZnNRUUw1TENlMUt4dlFNTVQ3bExQVXZUelYzUWJqVElSWDVNZGtsemE0ZTVMbW42V01mTFE9)

终于完成了,点击finish跳转。

![emby set done](https://images.iminling.com/app/hide.php?key=aWtSeWdQRnQxazhMN0RNdGFCdytHbDFVaW1zOW9yV1dzZkNDYnYzMTBMbDFsbGREWVJqNXF2NUNHNjh4N202UEhORFV3MFU9)

经过上边设置，emby就算是初始化好了，接下来就可以进行媒体库添加了。

## 媒体库

进入到emby后台，找到媒体库菜单，就可以在这里进行添加媒体库了。

![emby 媒体库](https://images.iminling.com/app/hide.php?key=ZjdRV3Z6RCtTOFJuZy8yb01pNE1sWDF2Q3BySXAyNDl4cUxxQlFzbGt0YkNtT2dvMXR1R3JBeVdjNnNIZWlOWXZMNU9NU0k9)

添加媒体库时需要整理好分类,是电影、电视节目还是其他，方便emby正确刮削。

![emby add lib](https://images.iminling.com/app/hide.php?key=ajVxcTA0S1NrWVl3S1Y3UTQrNWJ4R0Rkd051YXFQckxRd1FDRC9KU21FVDBMS2EyQitOU1ZVVXhSZG1iUmpUZTh1YWdNaHc9)

根据情况配置上边的选项，配置好后emby就会开始进行刮削，完整后就可以在emby首页看到添加的目录下的所有电影了。

截止到现在emby已经可以正确的播放视频了，但是此时播放网盘的内容是会经过搭建alist那台机器，等于是那台机器把媒体内容下载下来，然后送给我们的播放器进行播放，如果alist部署在本地播放倒是什么没什么，反正都要下载下来。如果alist部署在其他vps上，流量消耗是一个比较大的问题。如果是部署在家里但是需要在家庭以外的地方播放，这时候流量也会经过家里然后才到你在外边的设备，播放速度就取决于你的家庭宽带的上行了，一般上行所以都不会太大，所以我们需要配置302重定向，让播放设备直接和网盘进行交互，而alist服务器只是负责给你列出目录树，供我们浏览使用。下边就来讲一下如何进行302播放。

## embyExternalUrl实现302播放

本文主要介绍使用[embyExternalUrl](https://github.com/bpking1/embyExternalUrl)工具进行302播放。有大佬做了整合，弄了一个docker镜像：[MediaLinker](https://github.com/thsrite/MediaLinker)，降低了操作的复杂度，并且支持ssl证书，可以根据需求进行配置。docker compose文件内容如下：

```bash
services:
    medialinker:
        container_name: medialinker
        image: thsrite/medialinker:latest
        restart: unless-stopped
        volumes:
            - ./data/:/opt/
        environment:
            - AUTO_UPDATE=true
            - SERVER=emby
            - NGINX_PORT=8091
        network_mode: host
```

具体环境信息可以看大佬的介绍，我这里使用emby且也不需要SSL，如上是最简单的配置了。启动后，在data目录下会有一个constant.js文件，我们需要改变其中emby、alist的一些配置，需要配置的地方如下图：

![embyExternalUrl setting](https://images.iminling.com/app/hide.php?key=VTJuMzZadnFMdGQzWkp6ZFlHWk44aEVrb2trOHlIU0xzQlRjK3RTTi9TVFVGN21lSVJ6dzRpa2h6YUNtTEd2RHhrbXpqaFU9)

根据上图可以看到有7项信息需要填写，主要分为两个大项，一个是emby的配置，一个是alist的配置。下边来详细说一下每个参数的配置。

### emby配置

emby需要三个参数，embyhost这个简单，就是我们打开emby的地址，安装的机器ip和docker-compose文件中的端口就可以了。

embyApiKey需要在emby的后台去获取，路径如下，新建一个api密钥，然后填进去就可以了。

![emby api key](https://images.iminling.com/app/hide.php?key=aWRtQlVGcExkd3dWR1RvTWdBN3VvaGFpSmc3ZnE0b3ZaS21CcXhzdU5naHdqNGlwL1hnU2lqMFd3ZHo1bk15ZlpKSyt1OHc9)

mediaMountPath文件中介绍的也比较清楚：

> // 挂载工具 rclone/CD2 多出来的挂载目录, 例如将 od,gd 挂载到 /mnt 目录下: /mnt/onedrive /mnt/gd ,那么这里就填写 /mnt
> // 通常配置一个远程挂载根路径就够了,默认非此路径开头文件将转给原始 emby 处理
> // 如果没有挂载,全部使用 strm 文件,此项填[""],必须要是数组

我们在上边安装alis和emby的时候，两个内部路径都是`/data/movies`，那么这个数组中就填`/data/movies`.

emby的配置就完成了。

### alist配置

host也比较简单，我们怎么访问alist的就怎么写，一般是ip:5244.

alist-token的获取路径如下：

![alist api key](https://images.iminling.com/app/hide.php?key=VnhkSm81T0cxajAydnFhZnRrbFRFcXFzVVhPeEhqSW1DcDJEVHFiVUtpNkhjaDE1VDExb0dMUTZZZnhVTlRaUDF6RHFsR3M9)

然后是否启用了sign，这个在往alist配置每个网盘的时候，我这边都是有开启的，所以这个参数我需要设置为true.

签名有效期，按照他提供的路径也很容易找到，默认是0，我这里也把他改成12，两边同步。

然后保存`constant.js`文件，此时需要再对`medialinker`容器进行一次重启，如果不重启，mediaMountPath不生效。

如何判断是否成功了呢？可以查看medialinker的日志，`docker compose logs -tf` 来查看。如下我在第一次没有重启的情况去播放视频，看到有以下日志，找的还是默认的/mnt路径，并且可以看到我的服务器流量在疯狂的跑下行流量，说明是通过服务器下载后转给我的。

![emby 302 error](https://images.iminling.com/app/hide.php?key=NTVJOUE4WnZBV1A4WlFRbVY2dnVocElWRXQzamJIS2VCVDlnTHB0alNrZmQ1aUJTb2NTTGZvTlBNZUtWSnBzNmRGYm40ekk9)

经过对容器进行重启后，再次去播放视频，可以看到以下日志，读取到了我配置的/data/movies.

![emby 302 success](https://images.iminling.com/app/hide.php?key=OEhWalpia1RGai94MjdiZWlNSURKMFhDVXNDNTA3SU9RODFNR0NydC8rdVM2Vk5oc3h0QlplTDFVejZUVFpXdUJBY0E1TEk9)

经过以上就可以愉快的302直接播放云盘中的视频了。希望可以帮到大家。
