---
title: debian系统使用docker快速安装promutheus
author: 要名俗气
type: post
date: 2023-11-26
url: /2023/debian-use-docker-install-promutheus
description: Prometheus是一套开源的监控 报警 时间序列数据库的组合,起始是由SoundCloud公司开发的。它的功能强大，被广泛使用。今天这边文章就简单介绍一下如何使用docker快速的搭建一个prometheus的服务，其他相关的知识在后续的文章里再详细分享。先把服务跑起来。如果还没有安装docker的同学可以参考的我另一篇文章来快速的安装docker环境。
image: https://images.iminling.com/app/hide.php?key=M3JOYmRIT1UyZHJjcTZJRXloNjR6aTUyME1KbkhNak8yVDlTVm9saGloSXUwWkNsN0VZSStxRGU1Q1ZTQS9xVzRuY0szTHM9
categories:
  - docker
tags:
  - docker
  - prometheus
---
[Prometheus](https://prometheus.io/)是一套开源的监控&报警&时间序列数据库的组合,起始是由SoundCloud公司开发的。它的功能强大，被广泛使用。今天这边文章就简单介绍一下如何使用docker快速的搭建一个prometheus的服务，其他相关的知识在后续的文章里再详细分享。先把服务跑起来。

如果还没有安装docker的同学可以参考的我另一篇文章来快速的安装docker环境：[docker安装]({{< ref "/post/linux/docker/linux服务器快速安装docker和docker-compose.md" >}})。

## docker-compose文件准备

我习惯直接使用docker-compose来安装所有服务，我的所有服务都使用同一个网络，实现服务的互通，下边贴出我的配置：

```yaml
networks:
  mynet:
    external: true
services:
  prometheus:
    image: prom/prometheus:v2.46.0
    restart: unless-stopped
    container_name: prometheus
    volumes:
      - ./config/prometheus.yml:/etc/prometheus/prometheus.yml
      - /data/prometheus/data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    networks:
      - mynet
    ports:
      - 9090:9090
    environment:
      - TZ=Asia/Shanghai
    user: "0"
```

以上是我的docker-compose文件，下边我们介绍一下个项的意义

### networks

网络，我指定了一个已经创建好的桥接网络：mynet，这个大家可以指定自己的网络，或者删除这个networks，让docker为该compose创建一个网络。

### services

服务，就是我们要创建的prometheus服务了，指定了镜像，重启策略，启动命令，网络，端口以及时区等信息。

## 目录结构

上边是docker-compose的配置信息，里边指定了prometheus启动的配置文件地址，我们准备一个最简单的配置文件，prometheus自己拉取自己的监控信息：

```yaml
global:
  scrape_interval: 1m

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 1m
    static_configs:
      - targets: ['localhost:9090']
```

具体的配置信息以后再讲，这里的global设置了整个拉取间隔为1分钟，并添加了一个拉取配置，任务名叫：prometheus，目标是localhost:9090，就是安装的prometheus自己。

整个的目录结构如下：

```bash
prometheus
├── README.md
├── config
│   └── prometheus.yml
└── docker-compose.yaml
```

## 启动服务

在根目录使用命令来启动prometheus：

```
docker compose up -d
```

启动后可以使用docker ps 来查看服务是否启动成功，如下启动成功：

```bash
root@WIKI-HK-A1:~# docker ps
CONTAINER ID   IMAGE                     COMMAND                  CREATED        STATUS          PORTS                                         NAMES
9b24b2c868ad   prom/prometheus:v2.46.0   "/bin/prometheus --c…"   3 hours ago    Up 34 minutes   0.0.0.0:9090->9090/tcp, :::9090->9090/tcp   prometheus
```

## 访问服务

prometheus启动占用的是9090端口，我们可以使用自己服务器的ip:9090来访问：

![](https://images.iminling.com/app/hide.php?key=clZkV3BZeklFSTNzRmJERkN1d0t2K0JSN0trQVRicUVlckpYVUlDYy9taC9TOVJHMmNtamxCNnBORUlwUTFFU0JFODFyMGc9)

访问后如上，我们可以在staus-targets中查看我们添加的那个任务：

![](https://images.iminling.com/app/hide.php?key=Q1AvdXgyWU5aZjRFY3dqeEt5OWlBUkhoclVPbWdQQm9URVAvSERHTGw1b1FkZ0ZWKzN2dnRPZ1cwK0lqTVpVTVlKWGk1b2c9)

以上就是简单的安装prometheus的过程，希望可以帮到大家。
