---
title: docker网络模式简介
author: 要名俗气
type: post
date: 2023-08-07T21:29:16+00:00
url: /2023/introduction-to-docker-network-mode
description: 日常开发中都有使用到docker，一直对docker的网络不是很清楚，所以花了点时间了解一下docker的网络，这里对了解的知识进行一下总结。
image: https://images.iminling.com/app/hide.php?key=alYxQXJQb1dGYk9kMFhXVUtHR2NKNElMWkFmL2JGekd5a09YNmNzbWJBSERqeWR2OUFTTXo1a25MZGRPS2JlKytwUFgvQWs9
categories:
  - docker
tags:
  - docker
---
日常开发中都有使用到docker，一直对docker的网络不是很清楚，所以花了点时间了解一下docker的网络，这里对了解的知识进行一下总结。

当我们安装好docker后，docker默认给我们创建了3个网络，如下：

```bash
ubuntu@VM-20-3-ubuntu:~$ docker network ls
NETWORK ID     NAME            DRIVER    SCOPE
cde83331f977   bridge          bridge    local
0e504852d7be   host            host      local
cdc83d5d9388   none            null      local
```




分别是bridge,host和none。下边我们就来看一下这三种网络的区别。

## bridge网络

桥接网络也是docker的默认网络，当我们安装好docker后，docker会在宿主机创建一个网卡`docker0`：

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 52:54:00:ee:63:60 brd ff:ff:ff:ff:ff:ff
    inet 10.0.20.3/22 brd 10.0.23.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff::6360/64 scope link
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:0c:be:59:f7 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:cff:febe:59f7/64 scope link
       valid_lft forever preferred_lft forever
```




如上，docker0网络就是docker默认的网络，他的网关是`172.17.0.1`,我们可以先查看一下这个bridge网络的情况(已去除多余属性)：

```bash
ubuntu@VM-20-3-ubuntu:~$ docker inspect bridge
[
    {
        "Name": "bridge",
        "Id": "cde83331f977b86b4eaa67fe05ea18e493691fd0d3fc5edc51dec4e62a512d37",
        "Created": "2022-08-25T20:37:27.17350164+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Containers": {}
    }
]
```




subnet和docker0是一样的，containers那里暂时也是没有容器，当我们启动一个容器来使用默认的bridge网络的时候，这里边就会有对应的容器了。

我们来启动一个nginx容器，然后再来查看一下这个网络：

```bash
ubuntu@VM-20-3-ubuntu:~$ docker run -d --name nginx-test nginx:1.22.0
4ee2915a41fa9ed6e2591f852789f5762343e3de5bdabe718896778305df3569
ubuntu@VM-20-3-ubuntu:~$ docker inspect bridge
[
    {
        "Name": "bridge",
        "Id": "cde83331f977b86b4eaa67fe05ea18e493691fd0d3fc5edc51dec4e62a512d37",
        "Created": "2022-08-25T20:37:27.17350164+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Containers": {
            "4ee2915a41fa9ed6e2591f852789f5762343e3de5bdabe718896778305df3569": {
                "Name": "nginx-test",
                "EndpointID": "f518fa3072bbabc3de35577e5dc99570e8e3604e23a19911473b83fe90a85d73",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        }
    }
]
```




containers中已经有刚才创建的nginx的容器了。如果该容器想要和外界进行交互就需要暴露端口出来，然后外界才能和他进行网络通信，否则就只能从容器内部访问到外部，而无法从外部访问到容器内部。

```bash
ubuntu@VM-20-3-ubuntu:~$ docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED         STATUS         PORTS        NAMES
4ee2915a41fa   nginx:1.22.0                    "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   80/tcp      nginx-test
```




我们刚才的容器，内部会暴露80端口，如果我们需要和外部的端口映射，需要在启动容器的时候加上-v参数来映射端口，这里就不再演示了。

在默认bridge网络下的容器是无法相互通过容器名称进行通信的，因为没有进行name的主机解析，如果需要让多个容器通过name进行访问，就需要我们创建自己的bridge网络了。创建命令如下：

```
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
```



上边命令创建了一个mynet的桥接网络，然后在创建容器的时候使用这个网络，不同的容器间就可以使用容器名进行网络访问，创建容器命令：

```
ubuntu@VM-20-3-ubuntu:~$ docker run -d --network mynet --name nginx-test nginx:1.22.0
```



通过--network指定容器工作的网络。

## host网络

host顾名思义就是和宿主机共用同一个网络，容器没有和宿主机很好的隔离，容器占用的端口就是宿主机的端口，被容器占用的端口宿主机里的程序就没办法使用，容器里可以直接访问宿主机的其他应用程序服务，无需经过其他特殊的设置。

```bash
ubuntu@VM-20-3-ubuntu:~$ docker inspect host
[
    {
        "Name": "host",
        "Id": "0e504852d7be83b42232d744a36504266c52cca3f1647eaeaa9d090117826924",
        "Created": "2022-05-20T13:41:40.594740985+08:00",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
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
]
```




我们使用host模式创建的容器也不会显示在containers里边。

## none网络

在这种模式下，docker容器没有任何网络设置，无法访问宿主机网络，宿主机也没办法访问容器的网络，做到了很好的隔离，开发中也很少用到。

```bash
ubuntu@VM-20-3-ubuntu:~$ docker inspect none
[
    {
        "Name": "none",
        "Id": "cdc83d5d9388583dcee94758821adcd70f57825cbb05f2bcab726d484e9e9fdb",
        "Created": "2022-05-20T13:41:40.580477496+08:00",
        "Scope": "local",
        "Driver": "null",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
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
]
```




我们正常使用的话，还是bridge网络用的多一些，因为各服务都是需要相互调用的，要做到网络互通。以上就是对docker三种模式的一些总结。