---
title: openwrt配置Cloudflare DDNS
author: 要名俗气
type: post
date: 2023-04-08T18:12:20+00:00
url: /2023/openwrt-configuration-cf-ddns
description: 想要在外边访问家里的网络，可以使用运营商分配的ip，但是ip可能会经常变动，所以需要进行ddns配置，配置DDNS的前提条件是家里的宽带需要有公网ip。下面我们开始使用cloudflare来进行DDNS的配置。
image: https://images.iminling.com/app/hide.php?key=N3NkMDdwS3dld0x2Q1I2a09HUW9SZU5Fb2g1SEV5Z1FBaU9QcjNwbzE3ZFZLNWlLdVBFZC9Uai9hS0VLdCtDMUFEcVVsZ1E9
categories:
  - router
tags:
  - cloudflare
  - ddns
  - openwrt
  - r2s
---
想要在外边访问家里的网络，可以使用运营商分配的ip，但是ip可能会经常变动，所以需要进行ddns配置，配置DDNS的前提条件是家里的宽带需要有公网ip。下面我们开始使用cloudflare来进行DDNS的配置。

## 添加ddns配置

选择openwrt里的动态DNS功能，然后在里边添加一个自己的配置，名称随便，如我添加的`MYDDNS`\`。

![](https://images.iminling.com/app/hide.php?key=SnR4NzYydnp1dWRnL2NQTXBPeDhaUk1BYWNoczg5OEdXejZEbjVmTFVlZEdzRFFwV20zY1BSNjNXQWcrZGhzb2c1bzU5Vm89)

添加后就跳转到了配置界面，首先我们要选择ddns的提供商，这里我们选择cloudflare:

![](https://images.iminling.com/app/hide.php?key=RVpXQnJCRHFsZVM0cXdtSjN3T1BYaWpDd1BPdjNBYktVdHM1R0w5RXdVaytpaXRNM3lOc3NlRHJsb2lueGFHV25hNkNLVWM9)

切换后如下图：

![](https://images.iminling.com/app/hide.php?key=VllGYlZxRmg1enZJWWNjVDJWQzRtOExhUHRYdjl0M0RQanZWbUJBdGsyNTNVc0FJdmEvZnhNaFBtWjNkeFZwOE41cVYvSmc9)

### 查询主机名

这个就是我们的域名，通过这个域名访问到我们的openwrt管理后台，配置一个自己的三级域名，比如购买的域名的`openwrt.com`\`。这里则填写`admin.openwrt.com`\`。

### 域名

域名这个选项其实就上将查询主机名修改为`admin@openwrt.com`。

### 用户名和密码

用户名是cloudflare的登录邮箱，密码是cloudflare的global api key。如下图，点击后边的view就可以查看自己的key了。

![](https://images.iminling.com/app/hide.php?key=M1EzaHhtZkNrUWNrUWJvdE9ZVzlSMHhtZmZOZWNPTTZqdVZ6MHdzbHpLYTN4UlJZNDFtOW1iandLc2t2aEkvamMxVHdraEk9)

截止上边openwrt里的配置就已经完成了。点击保存并应用就可以了。

## 域名解析

我们在上边使用到了`admin.openwrt.com`。然后我们需要在cloudflare的后台去配置这个域名的解析。添加一条A记录，记录值为\`admin\`,然后我们就可以通过admin.自己的二级域名来访问我们软路由系统了。IPv4地址可以先随便填一个，等待ddns程序自动进行更新。

![](https://images.iminling.com/app/hide.php?key=dnBCdUNXU1d5cXZldkhyVWpJU04xNll2SG1mWEpRUWlNV0lUNWRKTWh4YVQ2VU1BbU80MDhFS2puc3grbDJublNUMW93ajQ9)

## 开放openwrt端口

在openwrt后台系统中找到`网络-防火墙-端口转发`\`，进行端口转发配置

具体配置说明：

名称：随便填一个标识。

传输协议：选则TCP+UDP。

外部区域：选wan口

外部端口：ddns中的域名+外部端口访问到你这个机器。端口一般在1024~65535中选择。

内部区域：选lan口

内部ip地址：选openwrt的管理后台ip,例如我这里是192.168.6.1。

内部端口：openwrt的管理后台一般都是输入ip就可以访问，所以端口是80.

![](https://images.iminling.com/app/hide.php?key=cXR2WTJpMTY4WFpRY0g4aGsrcWthd2xMdmZzZzJsc0JNd05GQ3RISjBycHZuYWFQL2hteW1STzRBRzJ3M2pHSjY5d1EyYWM9)

添加，然后保存并应用，就可以通过ddns里的配置的`查询域名:外部端口`访问到管理后台，本文中的就是`admin.openwrt.com:8080`。8080端口是我随便写的，根据自己情况选择一个可以使用的端口，1024~65535之间的都可以。