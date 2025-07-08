---
title: 私人影院之搭建自己的alist服务端并添加常见网盘
author: 要名俗气
type: post
date: 2024-09-21T09:56:03+00:00
url: /2024/build-alist-server
description: 前段时间写了一篇如何搭建小雅alist的文章，使用小雅的媒体库来进行观影，但是这种有一个问题就是小雅的库更新的不是很快，而且如果我想在电视上观看比较麻烦，就算搭配emby,也无法避免资源更新慢的问题。所以就想着使用自己的网盘配合自己搭建的alist服务端，来进行资源快速入库，方便自己快速观看。那么接下来就详细说明一下自己的搭建过程。
image: https://images.iminling.com/app/hide.php?key=ZVpuQ3crbnpMRHc2N2lQQktsMm93TkVobGtsOXZrV2ZsUUYyajMxd1FuSlFXSW5iK0wxRUxPcEYvcEhDU0NkSnJJQzdnT289
categories:
  - nas
tags:
  - 115网盘
  - 189
  - alist
  - 天翼网盘
---
前段时间写了一篇如何[搭建小雅alist]({{< ref "/post/nas/video/搭建属于自己的影视小站：小雅Alist安装及使用.md" >}})的文章，使用小雅的媒体库来进行观影，但是这种有一个问题就是小雅的库更新的不是很快，而且如果我想在电视上观看比较麻烦，就算搭配emby,也无法避免资源更新慢的问题。所以就想着使用自己的网盘配合自己搭建的alist服务端，来进行资源快速入库，方便自己快速观看。那么接下来就详细说明一下自己的搭建过程。

## alist服务端

我这里使用`docker compose`来搭建`alist server`,alist提供了两个版本的镜像，一个是普通的版本，一个是带aria2下载的版本，根据自己的需求进行安装，安装文档参考[官方文档](https://alist.nn.ci/zh/guide/install/docker.html)。

我的docker-compose.yaml如下：

```yaml
networks:
  mynet:
    external: true
services:
  alist:
    image: xhofe/alist-aria2:latest
    container_name: alist
    restart: unless-stopped
    volumes:
      - ./data:/opt/alist/data
    networks:
      - mynet
    ports:
      - 15244:5244
    extra_hosts:
      - host.docker.internal:host-gateway
    environment:
      - PUID=0
      - PGID=0
      - UMASK=022
      - TZ=Asia/Shanghai
```

我的目录分配如下：

```bash
.
├── data
└── docker-compose.yaml
```

整体配置是比较简单的，我使用了自己的网络`mynet`，而不是让docker自建一个网络，方便后续容器之间的项目访问。

按照上边配置好后直接`docker compose up -d`运行起来容器。之后通过访问ip:15244(我这里映射到主机的15244端口了)，默认admin用户和密码，进行登录并修改密码。容器启动成功，alist的服务端搭建就算完成了。

## 网盘添加

我这里主要添加三种类型，天翼云盘，115网盘，本地smb，下边就看一下每种网盘是需要怎么来添加。

### 天翼云盘

我家里使用的是电信的宽带，送了天翼网盘的会员，个人和家庭分别是4TB，整体也不算少。

进入到alist的管理界面

![alist disk](https://images.iminling.com/app/hide.php?key=VzFnSjkyMkZvU2JNMFZtT0pDeFhJTEFCM1NtK2ZwTGdmN2d5L251ZTVlMk80TzBFVWJqa1hhR2k2ZUR1V001amJPWVhhdXM9)

点击存储-添加,然后选择`天翼云盘客户端`(官方建议的类型)。然后就可以看到很多选项需要填写，具体的填写操作步骤参考[官方文档](https://alist.nn.ci/zh/guide/drivers/189.html)。

![alist add 189](https://images.iminling.com/app/hide.php?key=Y2FYY1hHdHZaYjB0OFdmdklWNEF0TXA0b0d3YmFIc2lzcklvajY5ZWc1bDJqRmFSRHM1TEFxYVpyWXJLaFlQdVZxMWdEL2c9)

挂载路径是最终显示在alist主页的名称，需要唯一。

缓存过期时间根据自己的需求来定义，比如我添加了一个文件想要快一点能刷新出来，那么我就定义短一点(比如1分钟)，这个根据自己需求来定。

WebDAV策略建议都使用302重定向，这样子在通过WebDAV访问alist服务端的内容的时候，实际上会重定向到网盘内真实的路径，等于直接访问网盘。

![alist add 189](https://images.iminling.com/app/hide.php?key=RnZOZEdPMDR4UTNuVzNnOXdnQm5rNDB2N05KTnpDcXVKL0FUeEZLMzlkRTgzME03NUxTQkp5TjhGUjNrdlZpTDRaMDJVRFE9)

启用签名选项如果后续是需要配合emby实现302播放，**一定要打开，一定要打开，一定要打开**。

用户名就是天翼云盘登录的手机号。

密码是天翼云盘登录的密码。

根文件的id根据自己需求，如果只是想把网盘内的某一个目录添加到alist中，那就按照[官方文档](https://alist.nn.ci/zh/guide/drivers/189.html)里的方法找到对应的文件夹id。

类型有`家庭云`和`个人云`，`family id`在是选择`家庭云`的时候需要填写，官网里都有介绍。我这里就是`家庭云`。

基本上述搞定就可以添加成功了。

### 115网盘

基本和天翼云盘添加流程差不多，驱动类型选择`115网盘`,挂载路径、缓存过期时间、WebDAV策略、启用签名四个选项和天翼云盘意向逻辑，进行处理就行了。有不明白的可以参考[官方文档](https://alist.nn.ci/zh/guide/drivers/115.html)。

![alist add 115](https://images.iminling.com/app/hide.php?key=NFZFUVpyNmtTaUY3TnhFd2o1MURwaGRTcWpZZHB5TWFLS0RXbzVpSXBLK2cyRlo3aWxiTmRVYzluenJIcFZUejcwU0hNNTQ9)

cookie和Qrcode二选一，[官方文档](https://alist.nn.ci/zh/guide/drivers/115.html#_1-qrcode-%E6%89%AB%E7%A0%81%E6%96%B9%E5%BC%8F%E7%99%BB%E5%BD%95)也有详细的介绍，包括根文件夹ID，都有详细的介绍，参考文档就可以了。感觉Qrcode会简单一点。我是直接使用的抓包软件抓的应用的cookie。这里有一个事项需要注意一下，如果使用了手机端软件，后续在网页登录的时候最好选择用手机扫码登录，要不然可能会把手机应用端挤掉导致cookie失效。

### SMB添加

有些文件没有放在网盘，而是放在自己本地或者放在本地nas中，比如我上篇文档也介绍了[Proxmox VE(PVE)8.0安装黑群晖NAS并直通硬盘]({{< ref "/post/linux/pve/Proxmox VE(PVE)8.0安装黑群晖NAS并直通硬盘.md" >}})，这样子想要在alist中把nas中的资源添加进去，就需要使用到SMB一些了，nas中是有开启这个功能的。

![alist add smb nas](https://images.iminling.com/app/hide.php?key=d2U1N1FpRE1RSnBmcEhLUFRibUxKV1lmM3ZpSGxzek5xYXZKcjJuZVVlQk1QcDVlMHB5bVNidE9pUDNJbXF0N3FZOVlaMzg9)

添加驱动类型选择`SMB`

![alist add smb nas](https://images.iminling.com/app/hide.php?key=MFVsanZ1SUh1bTgrYldBSjhGYThsdVlwelpSTHNSV3dKcm8zKy9kZ1FGOERCQktXRWNzRTIxYmpvZ2NSQ1crbWJVeHdkUzQ9)

smb是本地文件，启用签名就可以不用开了。地址、用户名、密码和分享名称(smb中的目录)根据自己的需求填写就可以了。

## 总结

我这里介绍了三种网盘的添加方式，其他的网盘也都大同小异，比如添加阿里云盘等。那么搭建alist服务端并添加网盘，就介绍到这里。希望可以帮到大家。

接下来还会介绍利用rclone挂载alist，然后让emby来使用并实现302播放。
