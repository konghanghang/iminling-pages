---
title: iKuai开启IPv6配置
author: 要名俗气
type: post
date: 2025-07-04
draft: true
url: /?p=775
description: 外网ipv6配置 点击添加，进行ipv6配置 配置保存后就可以看到获取到实际的ipv6地址了。 然后进行内网设置，绑定线路为宽带的端口，一般为wan1，DHCPv6模式配置为有状态。 这时候看看DHCPv6终端就可以看到有不少设备已经获取到了IPv6地址了 但是不想再设备都获取ipv6，只想特殊的设备获取，这时候就开启ipv6白名单 通过添加按钮，指定设备的mac地址就可以了。
categories:
  - 随记
---
外网ipv6配置

![ikuai ipv6](https://images.iminling.com/i/2025/02/16/b0c072f2f690627a072d4f4e8f3e5852.webp)

点击添加，进行ipv6配置

![iKuai ipv6配置](https://images.iminling.com/i/2025/02/16/94feaf1505cb3305feadeaa40ef3c20a.webp)

配置保存后就可以看到获取到实际的ipv6地址了。

![iKuai ipv6 get](https://images.iminling.com/i/2025/02/16/973b5e3f41fdf1374bcce54aafe82dd9.webp)

然后进行内网设置，绑定线路为宽带的端口，一般为wan1，DHCPv6模式配置为有状态。

![iKuai ipv6 内网](https://images.iminling.com/i/2025/02/16/f4b39cf3c003cb9d5e2f07a0bdc5d5ee.webp)

这时候看看DHCPv6终端就可以看到有不少设备已经获取到了IPv6地址了

![ipv6 terminal](https://images.iminling.com/i/2025/02/16/4b1d46f690bfe00dfa173446c95695c2.webp)

但是不想再设备都获取ipv6，只想特殊的设备获取，这时候就开启ipv6白名单

![ipv6 whitelist](https://images.iminling.com/i/2025/02/16/6a82129fac74a0625badb3c5ed17b24a.webp)

通过添加按钮，指定设备的mac地址就可以了。我这里对NAS进行白名单添加，