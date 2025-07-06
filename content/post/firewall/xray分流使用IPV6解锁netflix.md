---
title: xray分流使用IPV6解锁netflix
author: 要名俗气
type: post
date: 2023-08-13T15:13:49+00:00
url: /2023/xray-use-ipv6-unlock-netflix
description: netflix在中国大陆是无法直接观看的，当我们买了一个vps搭建节点代理后正常情况是可以观看的，但是奈飞因为版权原因，有些vps机器IPv4并不能完整的看奈飞的所有内容，本文讲述如何使用双栈机器的IPv6来解锁。
image: https://images.iminling.com/app/hide.php?key=a3J1MHlDOU5KWjlodHdVUnJyQm5EckxmcmwvMmVoVjRqdHNheGcveTM3ZUlqTWRjb0IyUCtZM0x6cUFzbFFQb2MvS1QrLzQ9
categories:
  - xray
tags:
  - IPV6
  - netflix
  - xray
---
![](https://www.iminling.com/wp-content/uploads/2023/08/e7653336f56a5b7486a4bfe7c4dbcfff.jpg)

netflix中文名又叫网飞或者奈飞，在中国大陆是无法直接观看的，当我们买了一个vps搭建节点代理后正常情况是可以观看的，但是奈飞因为版权原因，有些vps机器并不能完整的看奈飞的所有内容，这时就引出了另一个概念：奈飞解锁。一台vps观看奈飞可以分为三种情况：

  1. 无法观看，ip比较黑，直接连奈飞的官网都无法打开
  2. 只能观看奈飞的自制剧，因为是奈飞的自制剧，就没有所谓的严格版权，都可以看。大部分vps都是这种情况
  3. 可以观看奈飞非自制剧。可以观看奈飞购买的其他的版权剧。

上面我们了解到了奈飞的一些限制，所以网上就有很多解锁教程，通过warp，dns等来解锁。本文的前提是你的vps的ipv6是解锁奈飞的非自制剧的。

我手上的这台vps就是这种情况，ipv4只能观看自制剧，ipv6可以观看奈飞非自制剧，关于查看自己的vps是否解锁，可以通过以下脚本来验证：`bash <(curl -L -s check.unlock.media)`

我的机器解锁情况如下：
```bash
** 正在测试IPv4解锁情况
--------------------------------
============[ Multination ]============
 Dazn:                                  No
 HotStar:                               No
 Disney+:                               Yes (Region: GB)
 Netflix:                               Originals Only
 YouTube Premium:                       Yes
 Amazon Prime Video:                    Yes (Region: HK)
 TVBAnywhere+:                          No
 iQyi Oversea Region:                   HK
 Viu.com:                               Yes (Region: HK)
 YouTube CDN:                           Hong Kong
 Netflix Preferred CDN:                 Hong Kong
 Spotify Registration:                  No
 Steam Currency:                        HKD
 ChatGPT:                               Yes
=======================================

 ** 正在测试IPv6解锁情况
--------------------------------
============[ Multination ]============
 Dazn:                                  Failed (Network Connection)
 HotStar:                               No
 Disney+:                               No
 Netflix:                               Yes (Region: HK)
 YouTube Premium:                       No  (Region: CN)
 Amazon Prime Video:                    Unsupported
 TVBAnywhere+:                          Failed (Network Connection)
 iQyi Oversea Region:                   Failed
 Viu.com:                               Failed
 YouTube CDN:                           Hong Kong
 Netflix Preferred CDN:                 Hong Kong
 Spotify Registration:                  No
 Steam Currency:                        Failed (Network Connection)

 ChatGPT:                               Yes
=======================================
本次测试已结束，感谢使用此脚本
```

可以看到ipv4的结果是`Originals Only`就是所谓的只能观看自制剧，ipv6就是完整的解锁香港的奈飞的。

下面我们将使用xray来进行分流，使观看奈飞的时候走ipv6，观看奈飞的非自制剧。

## xray安装

xray可以使用官方的安装脚本，也可以参考我的xui安装教程：[x-ui安装配置使用](https://www.iminling.com/2023/03/26/54.html "x-ui安装配置使用"),具体的安装这里就不再详细讲了。

## 配置节点

先建一个入站节点，即我们需要连接到这台vps所使用的节点，可以通过xui来配置，节点类型根据自己的情况配置(vless,vmess,ss等都可以，比较稳的还是vless)，可以参考我的教程来搭建vless节点：[xray搭建reality和网站共用443端口启用网站SSL](https://www.iminling.com/2023/07/31/191.html "xray搭建reality和网站共用443端口启用网站SSL")，我这里为了简单直接搭建一个ss的节点，下边直接贴出xray的具体配置：

```json
{
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      {
        "type": "field",
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api"
      },
      {
        "type": "field",
        "outboundTag": "blocked",
        "protocol": [
          "bittorrent"
        ]
      }
    ]
  },
  "dns": null,
  "inbounds": [
    {
      "listen": null,
      "port": 24567,
      "protocol": "shadowsocks",
      "settings": {
        "method": "aes-256-gcm",
        "password": "WqOQzCxMwdfeegegOK36n",
        "network": "tcp,udp"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "none",
        "tcpSettings": {
          "header": {
            "type": "none"
          },
          "acceptProxyProtocol": false
        }
      },
      "tag": "inbound-13103",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls",
          "quic"
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "blocked",
      "protocol": "blackhole",
      "settings": {}
    }
  ],
  "stats": {},
  "reverse": null,
  "fakeDns": null
}
```



如上就是一个ss的完整配置，直接运行xray核心就可以访问了。上边的配置所有的网络都还是使用的ipv4来进行代理访问的。

## 配置奈飞ipv6出站

这里我们就需要使用到xray的分流策略，需要在routing配置中进行分流配置，然后在outbounds中配置具体的策略。关于分流参考官方的教程：[ruleObject](https://xtls.github.io/config/routing.html#ruleobject),ruleObject的规则如下：

```json
{
  "domainMatcher": "hybrid",
  "type": "field",
  "domain": ["baidu.com", "qq.com", "geosite:cn"],
  "ip": ["0.0.0.0/8", "10.0.0.0/8", "fc00::/7", "fe80::/10", "geoip:cn"],
  "port": "53,443,1000-2000",
  "sourcePort": "53,443,1000-2000",
  "network": "tcp",
  "source": ["10.0.0.1"],
  "user": ["love@xray.com"],
  "inboundTag": ["tag-vmess"],
  "protocol": ["http", "tls", "bittorrent"],
  "attrs": { ":method": "GET" },
  "outboundTag": "direct",
  "balancerTag": "balancer"
}
```




这里边的属性，如果配置多个则是要多个同时匹配中才能生效，下边我贴出配置奈飞的配置：

```json
{
   "type": "field",
   "outboundTag": "netflix-tag",
   "domain": [
     "geosite:netflix"
   ]
}
```




我这里使用domain来配置所有netflix的网址，然后使用outboundTag来指定如果匹配中netflix的网址，应该使用netflix-tag来进行出站访问，这个netflix-tag名称是可以自定义的，但是它需要和接下来讲的outbonds中的属性匹配，outbouds的配置可以参考xray官方文档：[outboudsObject](https://xtls.github.io/config/outbound.html#outboundobject),outboudsObject的规则如下：

```json
{
  "outbounds": [
    {
      "sendThrough": "0.0.0.0",
      "protocol": "协议名称",
      "settings": {},
      "tag": "标识",
      "streamSettings": {},
      "proxySettings": {
        "tag": "another-outbound-tag"
      },
      "mux": {}
    }
  ]
}
```




下边我贴出具体的配置：

```json
{
  "tag": "netflix-tag",
  "protocol": "freedom",
  "settings": {
    "domainStrategy": "UseIPv6"
  }
}
```




tag和routing下的ruleobject中的tag对应，经过上边的两个配置就可以使奈飞通过ipv6出站了。

## 完整配置

完整的xray配置config.json如下：

```json
{
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      {
        "type": "field",
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api"
      },
      {
        "type": "field",
        "outboundTag": "netflix-tag",
        "domain": [
          "geosite:netflix"
        ]
      },
      {
        "type": "field",
        "outboundTag": "blocked",
        "protocol": [
          "bittorrent"
        ]
      }
    ]
  },
  "dns": null,
  "inbounds": [
    {
      "listen": null,
      "port": 13103,
      "protocol": "shadowsocks",
      "settings": {
        "method": "aes-256-gcm",
        "password": "WqOQzCxMwdfeegegOK36n",
        "network": "tcp,udp"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "none",
        "tcpSettings": {
          "header": {
            "type": "none"
          },
          "acceptProxyProtocol": false
        }
      },
      "tag": "inbound-13103",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls",
          "quic"
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "netflix-tag",
      "protocol": "freedom",
      "settings": {
        "domainStrategy": "UseIPv6"
      }
    },
    {
      "tag": "blocked",
      "protocol": "blackhole",
      "settings": {}
    }
  ],
  "stats": {},
  "reverse": null,
  "fakeDns": null
}
```




下边也给出在x-ui中的配置，具体配置位置是登录x-ui后左侧菜单：面板设置，然后点击到xray配置，然后把下边的配置粘贴到配置模板中就可以了：

```json
{
  "api": {
    "services": [
      "HandlerService",
      "LoggerService",
      "StatsService"
    ],
    "tag": "api"
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 62789,
      "protocol": "dokodemo-door",
      "settings": {
        "address": "127.0.0.1"
      },
      "tag": "api"
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "netflix-tag",
      "protocol": "freedom",
      "settings": {
        "domainStrategy": "UseIPv6"
      }
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "policy": {
    "levels": {
      "0": {
        "handshake": 10,
        "connIdle": 100,
        "uplinkOnly": 2,
        "downlinkOnly": 3,
        "statsUserUplink": true,
        "statsUserDownlink": true,
        "bufferSize": 10240
      }
    },
    "system": {
      "statsInboundDownlink": true,
      "statsInboundUplink": true
    }
  },
  "routing": {
    "rules": [
      {
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api",
        "type": "field"
      },
      {
        "type": "field",
        "outboundTag": "netflix-tag",
        "domain": [
          "geosite:netflix"
        ]
      }
    ]
  },
  "stats": {}
}
```




经过上边的配置，正确的创建节点，就可以愉快的通过ipv4入站，然后通过ipv6出站去访问netflix网站了，享受解锁的快乐。当然上边的配置只是netflix才走ipv6出站，其他的网站还是通过ipv4来出站，如果想要让其他什么网站也通过ipv6出站，就需要在routing中配置ruleObject来达到目的。

## xui网页配置(另一种方法)

上边介绍的是纯手工的去对配置文件进行编辑的方式，最近xui更新后，这些配置可以在网页端进行图形化的操作，大大降低了对新手的难度，下边来看一下如何在xui网页端来对netflix由IPv6处站做配置。如果对xui还不知道如何使用的同学可以参考我的文章来安装配置xui：[x-ui安装配置使用](https://www.iminling.com/2023/03/26/54.html "x-ui安装配置使用").

### 出站配置

登录xui网站，菜单栏有一个xray设置一级菜单：

![xui-xray配置](https://images.iminling.com/app/hide.php?key=ck9VVTR3MU1XblNqWS9Vam1BSVdMZTNZcEFjdjVjNUoyTEtNSFMyMzFKZ1NONG1obTRKbFNDaGU5Q1BRZUxycERHdmlOWVE9)

点开xray设置后看到有四个tab页：基本模板、路由规则、出站和高级模板部件。在上边纯手工配置xray的时候也提到过，需要有一个outboud和一个routing，在xui的网页上其实就对应出站(outbond)以及路由规则(routing)。所以在网页端配置这两个就可以了，先来配置出站，添加一个freedom类型的节点：

协议：freedom

标签：全局唯一(英文)，等下要在路由规则那里进行选择

strategy：选择使用IPv6

然后点击保存就创建好了一个出站规则了，此时出站规则并没有任何路由规则引用到它，所以并不会有任何作用，接下来就需要在路由规则中进行引用，使特定网站使用IPv6进行出站访问。

![freedom out](https://images.iminling.com/app/hide.php?key=V3RQYk90ZTBwK1FmNVJ0RkUvVmxscW50MC9CRWQyQXRkWWRwRnZIbGp4Y3F5RVBaZFlzT0tzTzNQdWgrb2VZRE5YeTd0Wkk9)

### 路由规则配置

如下图，添加一个路由规则，Outbound Tag就是刚才添加的出站配置的标签，是唯一的，因为是要设置netflix使用IPv6，所以Domain那项配置就把netflix的域名规则写进去，这里的配置是 **且** 的关系，所以如果需要更灵活的配置可以自己添加。例如我只想某个用户在访问netflix的时候使用IPv6出站，那么就可以在User里把这个用户的信息配置进去，具体用户是什么，是我们在添加节点的时候每个节点都要求填一个邮箱，邮箱即用户。

![xui-routing](https://images.iminling.com/app/hide.php?key=QmxaTVd0eG5XWElDRVBqY1haUlR3MTNvNUNHS0pXUE1WRFdqcmg2REVNOE9Scm1oT1k3bmtpVlA0R0g4RlhsakdYZGNGb2M9)

配置好后就可以重启一下xray，规则就生效了。

好了本文的教程就到这里，祝大家都可以成功的搭建成功。
