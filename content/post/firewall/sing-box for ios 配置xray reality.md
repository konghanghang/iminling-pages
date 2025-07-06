---
title: sing-box for ios 配置xray reality
author: 要名俗气
type: post
date: 2023-05-18T16:52:19+00:00
url: /2023/sing-box-for-ios-configuration-xray-reality
description: 在ios端虽然有很多可以用来代理的软件，但是大多知名的软件都没法完整的支持reality, 小火箭(shadowrocket)是支持最好的一个软件了，支持到了xray1.7.5的版本，可以来配置rprx-vision,截止写这篇文章的时候还没有支持reality。
image: https://images.iminling.com/app/hide.php?key=MWZYT1hrOWxrOW0xcmZhcEF6WUNnUGo2ZWJEb0E0dFlvZnJtYU9ydnZZRERsRkZGcElubXpMaVUwZ0ZWVDVqdldVL1J4KzQ9
categories:
  - xray
tags:
  - sing-box
---
在ios端虽然有很多可以用来代理的软件，但是大多知名的软件都没法完整的支持reality, 小火箭(shadowrocket)是支持最好的一个软件了，支持到了xray1.7.5的版本，可以来配置rprx-vision,截止写这篇文章的时候还没有支持reality。其他的圈X(Quantumult X)、Loon以及Stash都还没有完整的支持vision,所以在ios上使用代理软件还是有一些尴尬的。好在还有两个软件是支持reality的。而且还都是免费软件，一个是sing-box for ios(SFI)，另一个就mate for iso(MFI)。现在都还在内测的阶段，今天我们就主要介绍SFI在ios上的安装和使用。

## 安装

先安装`TestFlight`才可以安装使用。直接在苹果应用商店搜索搜索就可以了。安装完`TestFlight`后就可以使用官方的邀请连接加入内测了：[SFI官方安装地址](https://sing-box.sagernet.org/installation/clients/sfi/)。

`TestFlight`安装后的界面：

![](https://images.iminling.com/app/hide.php?key=UkZtVXMvTUxMSzZCNURpVHdqUlozWDJFbjFuUUhFa0w5dllmMHBVTnNwVlFJeHpiUDdBVDRnWDQzb2RyQXVwWUdvMUYrQTQ9)

软件：

![](https://images.iminling.com/app/hide.php?key=NFRJU0prRWxiVXQ0eTgxNmFrS3dlZ2lDWXFWZkFEb3pPci9jWXo1ZnNaQ2ZWbjUwNUJva0lrZjFXZzJiNmo4U01IZUNJanc9)

## 使用

### 界面

下面我们先看一下软件打开后的界面：

![](https://images.iminling.com/app/hide.php?key=Q0ExYW9jN0hlN1FhZU1WQnFobCt3aHQ4YTFVS2RsRUxEUnBQR3o2S1dQQTlTSHJqYVdaa3hpQjQ3U1hHNkNvSEI4c25OV289)

初次打开的时候，Dashboard界面会有所不同，会提示安装一个配置文件，根据提示安装就可以了。

### 新建配置

配置文件是在`Profiles`菜单下新建的，点击`New Profile`就可以了，有三个参数需要填写：Name,Type和(File,Path或URL)。

Name随便起一个名字就可以了，Type可以选择local(本地),icloud以及Remote。

我们可以先在本地创建一个新的配置文件。

![](https://images.iminling.com/app/hide.php?key=bUl6UGdiL01WRmxOZ1FEcFh6SWlDTW9pdTMvb3ZQd0I3MGNvMGhYVEtLRGFXWDYvU0themswMEVtNzNWU3ptRWxQN3RIajQ9)

具体的配置文件如下：

```
{
    "log": {
      "level": "info",
      "timestamp": true
    },
    "dns": {
      "servers": [
        {
          "tag": "cloudflare",
          "address": "https://1.1.1.1/dns-query"
        },
        {
          "tag": "dnspod",
          "address": "https://1.12.12.12/dns-query",
          "detour": "direct"
        },
        {
          "tag": "block",
          "address": "rcode://success"
        }
      ],
      "rules": [
        {
          "geosite": "cn",
          "server": "dnspod"
        },
        {
          "geosite": "category-ads-all",
          "server": "block",
          "disable_cache": true
        }
      ]
    },
    "inbounds": [
      {
        "type": "tun",
        "tag": "tun-in",
        "interface_name": "utun",
        "inet4_address": "172.19.0.1/30",
        "auto_route": true,
        "strict_route": true,
        "stack": "gvisor",
        "sniff": true
      }
    ],
    "outbounds": [
      {
        "tag": "bwg", # tag和下边route中的要一致
        "type": "vless",
        "server": "自己的IP",
        "server_port": 自己的端口,
        "uuid": "自己的UUID",
        "flow": "xtls-rprx-vision",
        "network": "tcp",
        "packet_encoding": "xudp",
        "tls": {
          "enabled": true,
          "server_name": "www.microsoft.com", # 根据需要改成自己的
          "utls": {
            "enabled": true,
            "fingerprint": "chrome"
          },
          "reality": {
            "enabled": true,
            "public_key": "自己的Public_key",
            "short_id": "自己的short_id"
          }
        }
      },
      {
        "type": "direct",
        "tag": "direct"
      },
      {
        "type": "block",
        "tag": "block"
      },
      {
        "type": "dns",
        "tag": "dns"
      }
    ],
    "route": {
      "rules": [
        {
          "protocol": "dns",
          "outbound": "dns"
        },
        {
          "geosite": "cn",
          "geoip": [
            "cn",
            "private"
          ],
          "outbound": "direct"
        },
        {
          "geosite": "category-ads-all",
          "outbound": "block"
        }
      ],
      "auto_detect_interface": true,
      "final": "bwg"
    }
  }
```

将上边的相关配置改成自己的添加成功就可以了。

## 使用

通过上边的准备，现在已经可以开始使用了，在DashBoard界面，启动就可以了。也可以在Logs查看当前的连接情况，以及分流情况。

![](https://images.iminling.com/app/hide.php?key=V0lZV1htWGtXZGtldnE4eW03b0pSVi9LNExEUVh5WlozM2tLUzlOMEFOekhab0JEb1gwb1UxZC9iYkJjd1R5K1lMbzlPb1E9)

## 总结

以上就是reality的简单配置，sing-box支持多种协议，具体的可以参考：[官方文档](https://sing-box.sagernet.org/)。
