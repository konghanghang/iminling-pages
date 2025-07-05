---
title: docker部署certbot申请Let’s Encrypt证书
author: 要名俗气
type: post
date: 2023-07-30T00:21:32+00:00
url: /2023/docker-use-certbot-apply-lets-encrypt
description: 最近想给域名申请一个免费的证书，经过查询找到了Let's Encrypt，并且可以使用certbot来申请，因为一直在使用docker,所以就想着看是否可以使用docker来进行证书的申请，于是就开启了此次的折腾之路：使用docker部署certbot申请Let's Encrypt证书。
featured_image: /wp-content/uploads/2023/07/c7affd9a44cc655928e103db5974fdf9.png
categories:
  - docker
tags:
  - certbot
  - cloudflare
  - docker
  - letsencrypt
---
最近想给域名申请一个免费的证书，经过查询找到了Let's Encrypt，并且可以使用certbot来申请，因为一直在使用docker,所以就想着看是否可以使用docker来进行证书的申请，于是就开启了此次的折腾之路：使用docker部署certbot申请Let's Encrypt证书。

## 安装certbot

certbot的安装方式有很多种，官方也有一些系统的安装文档，可以根据官方的安装文档来直接安装到对应的操作系统上：[installation](https://certbot.eff.org/instructions)，非常详细。官方推荐的方式是通过`snap`的形式来安装，另外还有`docker`、`pip`、`第三放发行版本`以及`certbot-auto(官方已经标示为过期)`。

## 申请形式

申请证书的形式也有很多:

  * [apache](https://eff-certbot.readthedocs.io/en/stable/using.html#apache){#id9.reference.internal}
  * [Webroot](https://eff-certbot.readthedocs.io/en/stable/using.html#webroot){#id10.reference.internal}
  * [Nginx](https://eff-certbot.readthedocs.io/en/stable/using.html#nginx){#id11.reference.internal}
  * [Standalone](https://eff-certbot.readthedocs.io/en/stable/using.html#standalone){#id12.reference.internal}
  * [DNS Plugins](https://eff-certbot.readthedocs.io/en/stable/using.html#dns-plugins){#id13.reference.internal}
  * [Manual](https://eff-certbot.readthedocs.io/en/stable/using.html#manual){#id14.reference.internal}
  * [Combining plugins](https://eff-certbot.readthedocs.io/en/stable/using.html#combining-plugins){#id15.reference.internal}
  * [Third-party plugins](https://eff-certbot.readthedocs.io/en/stable/using.html#third-party-plugins){#id16.reference.internal}

其中一些方式要配合开启80和443端口来实现，还需要进行域名解析，我这里就选择使用`DNS Plugins`的形式来进行申请，因为域名在cloudflare，所以就使用`certbot-dns-cloudflare`插件来进行申请，文档地址：[Welcome to certbot-dns-cloudflare’s documentation!](https://certbot-dns-cloudflare.readthedocs.io/en/stable/)。

## 申请准备

docker文件的目录如下：

<pre class="core-next-code-pre"><code>├── certs //存放证书
├── conf
│   ├── cloudflare.ini //配置文件
├── logs //日志
└── docker-compose.yml //docker-compse文件
└── .env //环境变量信息</code></pre>

### certs

目录是映射到docker容器中的，存放证书文件。

### cloudflare.ini

该要存放cloudflare的api token,这里使用的是自己创建的token，配置文件内容如下：

<pre class="core-next-code-pre"><code>dns_cloudflare_api_token = your_cloudflare_api_key</code></pre>

具体的生成方式看下图：

![](https://www.iminling.com/wp-content/uploads/2023/07/2b10ab6732a555c77c7dbbef500b7157.png)

选择创建令牌：

![](https://www.iminling.com/wp-content/uploads/2023/07/b18db6009f4218e5be527153bc39a15d.png)

区域中选择DNS给一个编辑权限，然后就是区域字段选择自己的域名，我这里是给每个域名单独的token，所以需要选择一个域名，可以根据自己需求选择。

### logs

logs中是存在certbot执行日志的地方。

### docker-compose.yaml

docker-compose.yaml文件内容如下：

<pre class="core-next-code-pre"><code>version: "3"
services:
  certbot:
    image: certbot/dns-cloudflare:v2.4.0
    container_name: certbot
    env_file:
      - .env
    volumes:
      - ./certs:/etc/letsencrypt
      - ./logs:/var/log/letsencrypt
      - ./conf/cloudflare.ini:/secrets/cloudflare.ini
    # dry run
    # command: certonly --dns-cloudflare --agree-tos --non-interactive --dns-cloudflare-credentials /secrets/cloudflare.ini --email ${CERTBOT_EMAIL} --dns-cloudflare-propagation-seconds 20 -d ${CERTBOT_DOMAIN} --dry-run
    # issue --force-renewal
    command: certonly --dns-cloudflare --agree-tos --non-interactive --dns-cloudflare-credentials /secrets/cloudflare.ini --email ${CERTBOT_EMAIL} --dns-cloudflare-propagation-seconds 20 -d ${CERTBOT_DOMAIN}
    # renew
    # command: renew --dns-cloudflare --no-self-upgrade --agree-tos --non-interactive --dns-cloudflare-credentials /secrets/cloudflare.ini --dns-cloudflare-propagation-seconds 20</code></pre>

使用.env配置文件，然后把certbot中的/etc/letsencrypt目录映射到容器外部的certs目录，另外就是日志以及cloudflare的配置文件。

执行的命令有三个，dry-run是测试申请功能是否正常，如果正常则使用issue的命令进行正常申请，因为申请的证书只有三个月的有效期，所以需要进行续期，续期使用renew的命令。

--agree-tos 同意tos(term of service)

--non-interactive 非交互式的

--dns-cloudflare-credentials 指定cloudflare的token文件位置

--eamil 邮箱

--dns-cloudfalre-propagation-seconds 等待时间

-d 指定需要申请证书的域名，如果有多个域名则指定多个-d参数。如果需要申请泛域名证书，则使用*.xxx.com，生成的证书会在xxx.com目录下。

### .env

env文件是一些配置信息，如上边的docker-compose.yaml文件中有两个动态的参数，一个是邮箱，另一个是申请证书的域名，我们就是在.env文件中指定的，内容如下：

<pre class="core-next-code-pre"><code>CERTBOT_DOMAIN=your_domain_name
CERTBOT_EMAIL=your_email</code></pre>

指定邮箱和域名就可以了。

##  申请

经过上边的文件准备，接下来就可以开始申请了，使用docker compose up 来进行申请。申请后的证书就在certs目录下。

<pre class="core-next-code-pre"><code>├── 1.txt
├── accounts
├── archive
├── live
├── renewal
├── renewal-hooks
└── ssl</code></pre>

证书文件在live下。然后我们就可以在nginx中使用我们申请的证书文件了。

## 续期

证书三个月的有效期，在快要过期的时候需要进行续期，续期则使用docker-compose.yaml文件中的续期命令就可以了。大家也可以做一个corn定时任务来进行定期续期。

最后祝大家申请顺利。
