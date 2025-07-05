---
title: docker容器配置ipv6
author: 要名俗气
type: post
date: 2023-07-15T21:34:56+00:00
url: /2023/docker-container-configuration-ipv6
description: 在默认情况下，docker默认只是配置了ipv4地址，并没有ipv6地址，需要我们手动的去进行ipv6地址配置。下面就介绍一下容器支持ipv6的一种方式。 docker启用ipv6 具体的方式我们先参考docker的官方文档：[Enable IPv6 support](https://docs.docker.com/config/daemon/ipv6/),启用docker对ipv6的支持。
featured_image: /wp-content/uploads/2023/06/docker-480x300.webp
categories:
  - docker
tags:
  - docker
---
在默认情况下，docker默认只是配置了ipv4地址，并没有ipv6地址，需要我们手动的去进行ipv6地址配置。下面就介绍一下容器支持ipv6的一种方式。

## docker启用ipv6

具体的方式我们先参考docker的官方文档：[<span class="md-plain">Enable IPv6 support</span>](https://docs.docker.com/config/daemon/ipv6/),启用docker对ipv6的支持。需要修改daemon.json文件，具体位置为/etc/docker/目录下，如果没有则自己创建daemon.json,添加以下内容：

<pre class="core-next-code-pre"><code>{
  "experimental": true,
  "ip6tables": true
}</code></pre>

然后对docker进行重启。`systemctl restart docker`

## 创建网络

我安装docker后都会自己创建一个桥接网络已方便启动的各个容器之间的网络通信。创建网络的命令如下：

<pre class="core-next-code-pre"><code>docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet</code></pre>

如上创建了一个名字为`mynet`的网络，可以使用docker inspect来查看这个网络：

<pre class="core-next-code-pre"><code>root@us-la:~# docker inspect mynet
[
    {
        "Name": "mynet",
        "Id": "914c2a3a0a6ae0bdc0b8d51ba4782b6d232e6c978f2b0596fc4b7938d6c72ddd",
        "Created": "2022-08-21T09:19:48.858428501Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "edfd85b3884679256388f3c13afcd1774ac282a34de2438191bdad824a68f765": {
                "Name": "nginx",
                "EndpointID": "eafe61ed19613cfd2a7f56ee1737987d907a26210f0de557499fe4e6c1171e9c",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]</code></pre>

这个网络只支持ipv4，我们可以ping一下v6看看：

<pre class="core-next-code-pre"><code>root@us-la:~# docker run --rm -it --net mynet busybox ping -6 -c4 ipv6-test.com
PING ipv6-test.com (2001:41d0:701:1100::29c8): 56 data bytes
ping: sendto: Cannot assign requested address</code></pre>

如果我们需要支持ipv6，需要创建一个ipv6网络。

### 创建ipv6网络

用以下命令来创建一个ipv6网络：

<pre class="core-next-code-pre"><code>docker network create --ipv6 --subnet="2001:db8:1::/64" --subnet="192.168.0.0/24" mynet
9db7d457fd11f8c7c4256a1f4b2e00085d537f34e15dd0b0636bc00d4aa6d234</code></pre>

再次检查 网络：

<pre class="core-next-code-pre"><code>root@WIKIHOST-221108EU1RTF:~/images# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
9a75cccf2d9a   bridge    bridge    local
5677c5bdadce   host      host      local
9db7d457fd11   mynet     bridge    local
86ce569ad194   none      null      local
root@1RTF:~/images# docker inspect mynet
[
    {
        "Name": "mynet",
        "Id": "9db7d457fd11f8c7c4256a1f4b2e00085d537f34e15dd0b0636bc00d4aa6d234",
        "Created": "2023-07-12T09:48:08.273299482-04:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": true,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/24"
                },
                {
                    "Subnet": "2001:db8:1::/64"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]</code></pre>

多了一个ipv6的子网。

## 容器支持ipv6

### docker run

使用docker run命令运行容器的时候只需要加上--net 就可以使用我们创建的网络了：

<pre class="core-next-code-pre"><code>root@U1RTF:~# docker run --rm -it --net mynet busybox ping -6 -c4 ipv6-test.com
PING ipv6-test.com (2001:41d0:701:1100::29c8): 56 data bytes
64 bytes from 2001:41d0:701:1100::29c8: seq=0 ttl=45 time=249.606 ms
64 bytes from 2001:41d0:701:1100::29c8: seq=1 ttl=45 time=249.622 ms
64 bytes from 2001:41d0:701:1100::29c8: seq=2 ttl=45 time=249.691 ms</code></pre>

通过上边命令可以发现已经支持了ipv6了。

### docker compose

编写docker-compose.yaml文件 ，使用刚才创建的网络。

<pre class="core-next-code-pre"><code>version: '3'
networks:
  mynet:
    external: true
services:
  xray:
    image: teddysun/xray
    restart: unless-stopped
    container_name: xray
    environment:
      # 自定义host
      - XRAY_HOST=${XRAY_HOST}
    ports:
      - 13100:13100
    networks:
      - mynet
    volumes:
      - ./log:/var/log/xray
      - ./conf/config.json:/etc/xray/config.json</code></pre>

运行上边的xray后再执行ping v6，也是可以正常ping通的。

至次，我们已经可以给docker容器分配ipv6地址了，以上就是折腾过程，记录一下。
