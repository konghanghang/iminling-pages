---
title: 私人影院之rclone安装以及通过WebDav配置alist
author: 要名俗气
type: post
date: 2024-10-13T02:04:10+00:00
url: /2024/install-rclone-and-configuration
description: 上一篇文章介绍了对[alist的安装](https://www.iminling.com/2024/build-alist-server "私人影院之搭建自己的alist服务端并添加常见网盘")，本篇文章则来介绍一下rclone把alist里挂载的网盘再套娃挂载到本地磁盘，为后续emby直接读取本地的挂载文件做准备。那么接下来就开始折腾rclone. 配置文件 本文使用docker compose来进行安装，可以[docker-hub](https://hub.docker.com/r/rclone/rclone)找到rclone的官方镜像地址，安装可以参考rclone的官方[安装教程](https://rclone.org/install/#docker)。
featured_image: /wp-content/uploads/2024/10/rclone_logo_icon_169806.webp
categories:
  - 影音
tags:
  - alist
  - emby
  - rclone
  - webdav
---
上一篇文章介绍了对[alist的安装](https://www.iminling.com/2024/build-alist-server "私人影院之搭建自己的alist服务端并添加常见网盘")，本篇文章则来介绍一下rclone把alist里挂载的网盘再套娃挂载到本地磁盘，为后续emby直接读取本地的挂载文件做准备。那么接下来就开始折腾rclone.

![rclone ](https://www.iminling.com/wp-content/uploads/2024/10/FE27085E1D15A96BA8B9CD0E182FA3C9.png)

## 配置文件

本文使用docker compose来进行安装，可以[docker-hub](https://hub.docker.com/r/rclone/rclone)找到rclone的官方镜像地址，安装可以参考rclone的官方[安装教程](https://rclone.org/install/#docker)。

先要把rclone的配置文件给配置好，我这里只配置alist，所以直接给出配置文件内容方便大家快速使用:

```
[alist]
type = webdav
url = http://alist:5244/dav
vendor = other
user = admin
pass = x9WN7skj6o7_fqjd4F
```

如上配置，

[]中的名称自己定义一个，

type固定为webdav，

url为alist的webdav的url，url地址在[alist的文档](https://alist.nn.ci/zh/guide/webdav.html)中也有说明，就是自己alist的ip+端口然后拼上/dav就可以了。

vendor值为other

user为自己的alist的用户名

pass为自己的alist的用户名密码，但是密码需要进行加密才行，所以可以使用下边的命令来生成自己密码的密文，

生成加密密码：

```
docker run --rm -it \
    rclone/rclone \
    obscure [替换为具体的密码]
```

执行命令后就会生成一个你密码的密文，然后替换在配置文件中就可以了。

经过上边处理配置文件就处理好了，可以命名为`rclone.conf`，rclone的默认配置文件名就是这个，如果名字改动了，则需要在下边docker-compose配置文件的挂载时特殊配置。方便下边docker-compose文件使用。

## docker compose

老规矩，所有的应用程序都使用docker compose安装，下边是docker-compose.yaml的完整配置：

```
networks:
  mynet:
    external: true
services:
  rclone:
    image: rclone/rclone:latest
    container_name: rclone
    restart: always
    cap_add:
      - SYS_ADMIN
    devices:
      - /dev/fuse
    security_opt:
      - apparmor:unconfined
    volumes:
      - ./config:/config/rclone
      - /data/movies:/data:shared
      - /tmp:/tmp
    command: [
      "mount alist:/ /data --cache-dir /tmp --allow-other --vfs-cache-mode writes --allow-non-empty"
    ]
    networks:
      - mynet
```

network网络配置我这里就不讲了，是我自己创建的一个桥接网络。

这里主要说一下`volumes`和`command,` 其中`cap_add`，`devices`和`security_opt`有兴趣的同学可以自己去研究他的含义，这里直接按照我给的配置文件就行了。

volumes中`/tmp`，临时文件存放在目录，在`command`中指出了`cache-dir`的位置，然后挂载到宿住机的/tmp.

rclone的配置在容器的`/config/rclone`目录下，我挂载到本地的config目录，把上边咱们生成的配置文件放在里边。

`/data/movies:/data`则是目录的配置，先是在`command`中把咱们的远程alist中的/目录`(alist:/)`和rclone容器中的`/data`目录关联在一起，然后再通过volumes把容器中的`/data`挂载在宿主机的`/data/movies`下。通过这个目录挂载，咱们就可以在宿主机下`/data/movies`中浏览alist中所有挂载的文件信息，为后边`emby`添加影视库做准备。

## 容器启动

上边的配置信息完成后，文件放到对应的目录下，我这边的目录结构如下：

```
root@docker:~/rclone# tree
.
├── config
│   └── rclone.conf
└── docker-compose.yaml
```

然后在rclone目录下执行`docker compose up -d`启动容器就可以了。

如果配置没问题应该就可以在宿主机`/data/movies`目录下看到alist中的网盘和所有文件了。

以上就是折腾rclone和alist的过程，把alist挂载到本地，想本地目录文件一样来进行浏览，方便后续emby进行影视库的添加。
