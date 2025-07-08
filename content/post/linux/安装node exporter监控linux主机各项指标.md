---
title: 安装node exporter监控linux主机各项指标
author: 要名俗气
type: post
date: 2023-11-26T06:42:47+00:00
url: /2023/install-node-exporter
description: 在上一篇文章中讲解了如何在docker环境下快速的安装prometheus：[debian系统使用docker快速安装promutheus](https://www.iminling.com/2023/11/26/302.html "debian系统使用docker快速安装promutheus")，这篇文章咱们来讲一下如何在linux主机中安装node exporter来采集linux系统的各项指标。
image: https://images.iminling.com/app/hide.php?key=M3JOYmRIT1UyZHJjcTZJRXloNjR6aTUyME1KbkhNak8yVDlTVm9saGloSXUwWkNsN0VZSStxRGU1Q1ZTQS9xVzRuY0szTHM9
categories:
  - prometheus
tags:
  - prometheus
---
在上一篇文章中讲解了如何在docker环境下快速的安装prometheus：[debian系统使用docker快速安装promutheus]({{< ref "/post/linux/docker/debian系统使用docker快速安装promutheus.md" >}})，这篇文章咱们来讲一下如何在linux主机中安装node exporter来采集linux系统的各项指标。

## 安装node exporter

node exporter的安装非常简单，可以去prometheus的官网下载：[node exporter - prometheus](https://prometheus.io/download/ "node exporter")，或者去github进行下载：[node_exporter - github](https://github.com/prometheus/node_exporter "node_exporter - github")，我这里直接使用wget命令从github上下载最新的安装包`wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz`：

```bash
root@admin:~# wget https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
--2023-11-25 22:35:44--  https://github.com/prometheus/node_exporter/releases/download/v1.2.2/node_exporter-1.2.2.linux-amd64.tar.gz
Resolving github.com (github.com)... 20.205.243.166
Connecting to github.com (github.com)|20.205.243.166|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://objects.githubusercontent.com/github-production-release-asset-2e65be/9524057/28598a7c-d8ad-483d-85ba-8b2c9c08cf57?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231126%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231126T033542Z&X-Amz-Expires=300&X-Amz-Signature=cf09a0c524756802e669d7ccb2b60b9dcc5f3cfd0855cd9abad7d31947596c2f&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=9524057&response-content-disposition=attachment%3B%20filename%3Dnode_exporter-1.2.2.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [following]
--2023-11-25 22:35:45--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/9524057/28598a7c-d8ad-483d-85ba-8b2c9c08cf57?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20231126%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20231126T033542Z&X-Amz-Expires=300&X-Amz-Signature=cf09a0c524756802e669d7ccb2b60b9dcc5f3cfd0855cd9abad7d31947596c2f&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=9524057&response-content-disposition=attachment%3B%20filename%3Dnode_exporter-1.2.2.linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
Resolving objects.githubusercontent.com (objects.githubusercontent.com)... 185.199.108.133, 185.199.109.133, 185.199.111.133, ...
Connecting to objects.githubusercontent.com (objects.githubusercontent.com)|185.199.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8898481 (8.5M) [application/octet-stream]
Saving to: ‘node_exporter-1.2.2.linux-amd64.tar.gz’

node_exporter-1.2.2.linux-amd64.tar.gz   100%[===============================================================================>]   8.49M  20.2MB/s    in 0.4s

2023-11-25 22:35:46 (20.2 MB/s) - ‘node_exporter-1.2.2.linux-amd64.tar.gz’ saved [8898481/8898481]
```

下载到本地主机后解压刚下载的文件，里边有三个文件，node_exporter就是我们需要的可执行文件：

```bash
root@admin:~# tar -zxf node_exporter-1.2.2.linux-amd64.tar.gz
root@admin:~# ls
images  node_exporter-1.2.2.linux-amd64  node_exporter-1.2.2.linux-amd64.tar.gz
root@admin:~# cd node_exporter-1.2.2.linux-amd64
root@admin:~/node_exporter-1.2.2.linux-amd64# ls
LICENSE  node_exporter NOTICE
```

## 运行node exporter

可以进入到`node_exporter-1.2.2.linux-amd64`

目录，执行`./node_export`来执行项目，如下：

```bash
root@admin:~/node_exporter-1.2.2.linux-amd64# ./node_exporter
level=info ts=2023-11-26T06:17:11.718Z caller=node_exporter.go:182 msg="Starting node_exporter" version="(version=1.2.2, branch=HEAD, revision=26645363b486e12be40af7ce4fc91e731a33104e)"
.......
level=info ts=2023-11-26T06:17:11.720Z caller=node_exporter.go:199 msg="Listening on" address=:9100
level=info ts=2023-11-26T06:17:11.720Z caller=tls_config.go:191 msg="TLS is disabled." http2=false
```

从上边启动的日志可以看出采集的具体指标，倒数第二行日志可以看出node_exporter占用的是9100端口，如果我们想控制采集的指标或者更改占用端口，可以使用-h命令查看启动参数说明：

```bash
root@admin:~/node_exporter-1.2.2.linux-amd64# ./node_exporter -h
    --web.listen-address=":9100"  # 监听的端口，默认是9100
    --web.telemetry-path="/metrics"  # metrics的路径，默认为/metrics
    --web.disable-exporter-metrics  # 是否禁用go、prome默认的metrics
    --web.max-requests=40     # 最大并行请求数，默认40，设置为0时不限制
    --log.level="info"        # 日志等级: [debug, info, warn, error, fatal]
    --log.format=logfmt     # 置日志打印target和格式: [logfmt, json]
    --version                 # 版本号
    --collector.{metric-name} # 各个metric对应的参数
    ......
```

具体的采集参数后续在使用的时候再详细研究，这里就先全部让它采集。以上边的方式启动是以前台运行的，我们关闭窗口程序就会自动退出，下边来配置systemd管理node_exporter.

## 配置node_exporter.services

首先把把解压后的node\_exporter移动到/usr/local/bin目录下，然后新建一个node\_exporter.service，需要建到/etc/systemd/system目录下，具体内容如下：

```bash
Unit]
Description=node exporter service
Documentation=https://prometheus.io
After=network.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/usr/local/bin/node_exporter --web.listen-address=":9100"
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

如上所示，启动命令指定了启动的时候占用的端口，这个根据自己的实际情况进行更改，创建完毕后需要重启systemd服务，并使用systemd启动服务：

```bash
root@admin:/usr/local/bin# systemctl daemon-reload
root@admin:/usr/local/bin# systemctl start node_exporter
root@admin:/usr/local/bin# systemctl status node_exporter
● node_exporter.service - node exporter service
     Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-11-26 01:28:25 EST; 6s ago
       Docs: https://prometheus.io
   Main PID: 2794065 (node_exporter)
      Tasks: 4 (limit: 2358)
     Memory: 4.7M
        CPU: 15ms
     CGroup: /system.slice/node_exporter.service
             └─2794065 /usr/local/bin/node_exporter --web.listen-address=:9100

Nov 26 01:28:25 WIKI-HK-A1 node_exporter[2794065]: level=info ts=2023-11-26T06:28:25.705Z caller=node_exporter.go:115 collector=xfs
Nov 26 01:28:25 WIKI-HK-A1 node_exporter[2794065]: level=info ts=2023-11-26T06:28:25.705Z caller=node_exporter.go:115 collector=zfs
Nov 26 01:28:25 WIKI-HK-A1 node_exporter[2794065]: level=info ts=2023-11-26T06:28:25.706Z caller=node_exporter.go:199 msg="Listening on" address=:9100
Nov 26 01:28:25 WIKI-HK-A1 node_exporter[2794065]: level=info ts=2023-11-26T06:28:25.706Z caller=tls_config.go:191 msg="TLS is disabled." http2=false
```

经过上述步骤就在需要被监控的linux主机上安装好了node exporter，接下来需要在prometheus中配置对该主机的指标进行抓取。

## prometheus指标获取

在文章：[debian系统使用docker快速安装promutheus]({{< ref "/post/linux/docker/debian系统使用docker快速安装promutheus.md" >}})中只配置了对promutheus本身指标的采集，下边我们修改prometheus的配置文件，对刚才配置的linux主机进行指标采集，修改prometheus的配置文件如下：

```yaml
global:
  scrape_interval: 1m

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 1m
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'linux-node'
    static_configs:
      - targets: ['127.0.0.1:9100']
```

如上，配置了一个任务名称为linux-node的任务，它去获取127.0.0.1机器的指标，我们可以在prometheus的后台查看到对应的targets:

![](https://images.iminling.com/app/hide.php?key=TzFEUWFRdXdmV3NWcURMSUxWRUdNN1pXSTRveHIrL0tueFpCVmNncnBHZ21GaTJlakJYQ2RuV3ZZd3FQeXFKb3hSd3Y2Q289)

到这里就配置好了prometheus采集linux主机指标的配置，可以在prometheus的后台对信息进行查询，后续也可以配置grafana的图形界面来对node exporter采集的数据进行可视化展示。
