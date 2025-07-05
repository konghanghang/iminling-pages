---
title: 多网卡环境下Spring Cloud微服务注册特定网卡Ip到注册中心(Nacos)
author: 要名俗气
type: post
date: 2023-12-24T09:02:19+00:00
url: /2023/spring-cloud-use-true-ip-register-nacos
description: 在本地开发或者在线上环境的时候，本地开发机器和线上都有可能有多张网卡：物理网卡或者虚拟网卡，如果只有一张网卡，那么 Spring Cloud微服务也不会有注册错ip的情况，一旦有多张网卡，问题就会暴露出来，不同的网卡网络段肯定是不同的，那么ip之间也不可能互通，就会出现虽然服务注册到了注册中心，但是却无法访问到这个已注册的服务。
featured_image: /wp-content/uploads/2023/12/B8809638E59FD785DA709434B4E96FA4.png
categories:
  - Spring
---
在本地开发或者在线上环境的时候，本地开发机器和线上都有可能有多张网卡：物理网卡或者虚拟网卡，如果只有一张网卡，那么 Spring Cloud微服务也不会有注册错ip的情况，一旦有多张网卡，问题就会暴露出来，不同的网卡网络段肯定是不同的，那么ip之间也不可能互通，就会出现虽然服务注册到了注册中心，但是却无法访问到这个已注册的服务。

## 遇到问题过程

我遇到这个问题主要是在自己的本地电脑上，我电脑上有两个网卡的地址不是同一个段上的，如下：

![](https://www.iminling.com/wp-content/uploads/2023/12/AA1F9BA18B8976A4F4B176CE3342224F.png)

有一个192.168.6.6和另一个192.168.64.1的，在我启动我的本地的Cloud服务的时候，Nacos中默认注册的是192.168.6.6的地址：

![注册详情](https://www.iminling.com/wp-content/uploads/2023/12/E306EB09E6747063B453D5B9D29D64BB.png)

经过查询后在boostrap.yaml配置文件中进行修改：

```
spring:
  cloud:
    inetutils:
      preferredNetworks:
        - 192.168.64
```

经过配置后，再次启动项目，再看注册中心中注册的地址如下：

![注册详情](https://www.iminling.com/wp-content/uploads/2023/12/8AFB40E82AF9E7E13E3E88492599AC49.png)

已经是想要的结果了。

## 问题深入

在spring官方文档可以查询到，有几种方式可以配置，地址：[common-abstractions](https://docs.spring.io/spring-cloud-commons/reference/spring-cloud-commons/common-abstractions.html "common-abstractions")。

可以使用忽略特定的网络接口：

```
spring:
  cloud:
    inetutils:
      ignoredInterfaces:
        - docker0
        - veth.*
```

如上忽略了docker的网络接口以及veth相关的网络接口。

另一种就是我上边提到的`preferredNetworks`

配置首选网络，偏向与使用哪个ip地址。

官网还提到可以强制只使用站点本地地址，如下例所示：

```
spring:
  cloud:
    inetutils:
      useOnlySiteLocalInterfaces: true
```

具体详见官网的文档。
