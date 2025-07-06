---
title: xray搭建reality和网站共用443端口启用网站SSL
author: 要名俗气
type: post
date: 2023-07-31T06:30:46+00:00
url: /2023/both-xray-reality-website-use-443
description: 当我们只有一个vps，又想部署网站同时又想搭建代理，当然两个程序可以分开占用端口，做到不冲突，但是感觉不怎么优雅，所以想要让两个程序都去共用一个443端口，这样子网站可以使用域名并开启ssl，代理也可以使用443(心理作用，感觉代理走443会更好一些，毕竟代理使用了ssl，走其他端口怪怪的)。下面就开启折腾之路。
image: https://images.iminling.com/app/hide.php?key=TUVLUjRhOHdSS21VVjh4dDdmbElGUVhHZHRFRlR3c3h1R2F2Qi8wOEg0V3V6cGZReGx5NUs1Z3FQYjRhVXdCdm14WWVma2c9
categories:
  - xray
tags:
  - certbot
  - docker
  - nginx
  - reality
  - ssl
  - xray
---
当我们只有一个vps，又想部署网站同时又想搭建代理，当然两个程序可以分开占用端口，做到不冲突，但是感觉不怎么优雅，所以想要让两个程序都去共用一个443端口，这样子网站可以使用域名并开启ssl，代理也可以使用443(心理作用，感觉代理走443会更好一些，毕竟代理使用了ssl，走其他端口怪怪的)。下面就开启折腾之路。

## 代理协议

目前比较稳定的协议要数reality了，所以自己搭建的代理也是使用reality，reality可以使用自己的证书，也可以使用其他域名的证书：就是宣传的“偷证书”。既然我的网站要使用443开启ssl，那我当时使用自己的证书最方便，而且延迟最小。

## 证书申请

安装reality可以使用xui来安装，具体的xui安装可以参考我的文章：[x-ui安装配置使用]({{< ref "/post/firewall/x-ui安装配置使用.md" >}})。xui本身可以帮助我们申请证书，如果不想使用xui的证书申请，还可以参考我的另一篇文章使用certbot来申请证书：[docker部署certbot申请Let’s Encrypt证书]({{< ref "/post/linux/docker/docker部署certbot申请Let’s Encrypt证书.md" >}})，我这里就使用certbot来申请证书。

## 站点部署

这里是为了演示，所以我这里就只启动nginx，然后配置ssl，对域名启用ssl。nginx我使用的是docker来启动的，具体的目录如下,所在的路径是：`/root/images/nginx`
```
├── README.md
├── conf
│   ├── conf.d
│   └── nginx.conf
├── docker-compose.yaml
├── html
│   └── index.html
└── logs
    ├── 1.txt
    ├── access.log
    ├── error.log</code></pre>
```
conf目录下是站点的配置文件，nginx.conf是默认的，它包含了conf.d目录为的所有`.conf`结尾的文件。

### nginx.conf

nginx.conf的配置如下：
```
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log  /var/log/nginx/access.log  main;
    
    sendfile        on;
    #tcp_nopush     on;
    
    keepalive_timeout  65;
    
    #gzip  on;
    
    include /etc/nginx/conf.d/*.conf;
}
```

很简单就是默认配置。

### web.conf

web.conf是在conf.d目录下的，就是我们部署的站点的配置文件，如下：
```
# 设置 HTTP 服务器，将所有 HTTP 请求都重定向到 HTTPS
server {
    listen 80;
    server_name test.xxx.me;

    # 重定向所有 HTTP 请求到 HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# 设置 HTTPS 服务器
server {
    listen 443 ssl http2;
    server_name test.xxx.me; # ======域名，需要改成自己的=========

    # SSL 证书和密钥文件的路径
    ssl_certificate         /etc/letsencrypt/ssl/xx/fullchain.pem; # ======自己的证书文件路径=========
    ssl_certificate_key     /etc/letsencrypt/ssl/xx/privkey.pem;# ====自己的证书文件路径=======
    
    # 可选：设置 SSL 会话缓存
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # 可选：设置 SSL 协议和加密算法
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    
    # 可选：开启 OCSP Stapling，提高 SSL 握手性能
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/ssl/xx/fullchain.pem; # ======自己的证书文件路径=======
    
    # 可选：设置 Content Security Policy（内容安全策略）头
    # add_header Content-Security-Policy "default-src 'self';" always;
    
    # 可选：设置访问日志
    access_log /var/log/nginx/test.access.log;
    error_log /var/log/nginx/test.error.log;
    
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
    }
}
```

访问80端口强制转到443端口，证书使用的是docker中的使用certbot申请的证书。证书的路径是和下边的docker-compose.yaml中挂载的目录相关的，xx只是为了在nginx容器中找到正确的目录。

### docker-compose.yaml

具体内容如下：
```
version: '3'
networks:
  mynet:
    external: true
services:
  nginx:
    image: nginx:1.22.0
    restart: unless-stopped
    container_name: nginx
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./conf/conf.d:/etc/nginx/conf.d
      - ./html:/usr/share/nginx/html
      - ./logs:/var/log/nginx
      - /root/images/certbot/certs:/etc/letsencrypt
      - /root/images/certbot/certs/live/test.ml520.me:/etc/letsencrypt/ssl/xx
    networks:
      - mynet
    ports:
      - 80:80
      - 443:443
```

这里映射了letsencrypt的两个目录，配合nginx，找到对应的证书路径，如果不是在docker中就按找实际的证书路径处理就行了。

### index.html

index.html就是一个非常简单的网页了，里边的内容很随意,就是一个欢迎语。

这时候我们把nginx启动起来，通过域名访问我们的网站看是否已经可以正常使用https。

## 代理配置

代理的配置这里就简单带过，通过x-ui或者直接安装xray-core来运行。也可以通过我的镜像：[xray](https://github.com/konghanghang/images/tree/master/xray)，直接使用docker安装，参考文章：[xray配置使用reality]({{< ref "/post/firewall/xray配置使用reality.md" >}})

xray的配置文件如下：
```json
{
  "log": {
    "loglevel": "warning",
    "access": "/var/log/xray/access.log",
    "error": "/var/log/xray/error.log"
  },
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
      "listen": null,
      "port": 8443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "0c41318a-32b6-4017-9938-4033bf261234",
            "flow": "xtls-rprx-vision",
            "email": "reality@xray.com"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "show": false,
          "dest": "www.microsoft.com:443",
          "xver": 1,
          "serverNames": [
            "www.microsoft.com"
          ],
          "privateKey": "uPPcl9806d81DMdlojRB_ps78jrxNpT0wcG0ueOmD69",
          "shortIds": [
            "6ba85179e30d4fc2"
          ]
        }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      }
    }
  ],
  "outbounds": [
    {
      "tag": "common",
      "protocol": "freedom"
    }
  ],
  "routing": {
    "rules": [
      {
        "type": "field",
        "outboundTag": "common",
        "network": "udp,tcp"
      }
    ]
  },
  "policy": {
    "statsInboundUplink": true,
    "statsInboundDownlink": true,
    "statsOutboundUplink": true,
    "statsOutboundDownlink": true
  },
  "dns": null,
  "transport": null,
  "stats": null,
  "reverse": null,
  "fakeDns": null
}
```

这里先使用偷微软的证书进行验证。不出意外就已经ok了，此时xray用的8443，nginx使用的443，两个单独占用端口，是完全可以正常访问的。

## 共用端口

经过上边的准备，现在我们就可以开始来合并端口了，首先是修改nginx的配置，原来的nginx容器是暴露了80和443端口，现在我们需要把443端口让给xray，所以需要修改docker-compose.yaml文件，改变端口映射，让它的https使用18443，并且需要修改nginx的web.conf配置文件。

### docker-compose.yaml

内容如下：
```yaml
version: '3'
networks:
  mynet:
    external: true
services:
  nginx:
    image: nginx:1.22.0
    restart: unless-stopped
    container_name: nginx
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./conf/conf.d:/etc/nginx/conf.d
      - ./html:/usr/share/nginx/html
      - ./logs:/var/log/nginx
      - ./../naiveproxy/conf/cert:/ssl
      - ./../wordpress/html:/wordpress
      - /root/images/certbot/certs:/etc/letsencrypt
      - /root/images/certbot/certs/live/test.ml520.me:/etc/letsencrypt/ssl/xx
    networks:
      - mynet
    ports:
      - 80:80
      - 18443:18443 # 主要是修改这里，改变端口映射
```

主要是修改了端口，由原来的443改成18443，把443端口让给xray使用。

改了开放端口后，还需要修改web.conf，把监听的443变成18443以及一些其他配置。

### web.conf

先放出配置：
```
# 设置 HTTP 服务器，将所有 HTTP 请求都重定向到 HTTPS
server {
    listen 80;
    server_name test.xxx.me;

    # 重定向所有 HTTP 请求到 HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# 设置 HTTPS 服务器
server {
    listen                  18443 ssl http2 proxy_protocol; # =======修改了这里======
    server_name             test.xxx.me;
    set_real_ip_from        127.0.0.1; # ======添加了这里=======
    real_ip_header          proxy_protocol; # ======添加了这里======

    # SSL 证书和密钥文件的路径
    ssl_certificate         /etc/letsencrypt/ssl/xx/fullchain.pem; # 证书文件，通常不区分扩展名，证书文件需要使用fullchain（全SSL证书链）
    ssl_certificate_key     /etc/letsencrypt/ssl/xx/privkey.pem; # 私钥文件，通常不区分扩展名
    
    # 可选：设置 SSL 会话缓存
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # 可选：设置 SSL 协议和加密算法
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    
    # 可选：开启 OCSP Stapling，提高 SSL 握手性能
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_trusted_certificate /etc/letsencrypt/ssl/xx/fullchain.pem;
    
    # 可选：设置访问日志
    access_log /var/log/nginx/test.access.log;
    error_log /var/log/nginx/test.error.log;
    
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
    }
}
```

主要是修改了listen的配置，添加了proxy_protocol,另外添加了`set_real_ip_from`和`real_ip_header`。

接下来就是修改xray的配置文件了。

### xray配置

主要是修改config.json文件，从原来的8443换成可，以及域名配置，详细改动如下：
```json
{
  "log": {
    "loglevel": "warning",
    "access": "/var/log/xray/access.log",
    "error": "/var/log/xray/error.log"
  },
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
      "listen": null,
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "0c41234a-32b6-4017-9938-4033bf26b948",
            "flow": "xtls-rprx-vision",
            "email": "reality@xray.com"
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "show": false,
          "dest": "nginx:18443", # 这里目标网站要换成我们的nginx里的地址，这里使用了容器名加ssl监听端口
          "xver": 1, # 该参数我也不知道是什么意思
          "serverNames": [
            "test.xxx.me" # 域名就换成自己的域名
          ],
          "privateKey": "uPPcl3v45d81DMdlojRB_ps78jrxNpT0wcG0ueOmD12",
          "shortIds": [
            "6ba85179e30d4fc2"
          ]
        }
      },
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      }
    }
  ],
  "outbounds": [
    {
      "tag": "common",
      "protocol": "freedom"
    }
  ],
  "routing": {
    "rules": [
      {
        "type": "field",
        "outboundTag": "common",
        "network": "udp,tcp"
      }
    ]
  },
  "policy": {
    "statsInboundUplink": true,
    "statsInboundDownlink": true,
    "statsOutboundUplink": true,
    "statsOutboundDownlink": true
  },
  "dns": null,
  "transport": null,
  "stats": null,
  "reverse": null,
  "fakeDns": null
}
```

如上，改动不大，主要是dest、xver和server_name。另外需要修改xray的docker-compose.yaml,把原来的8443端口改成443端口，并重新build启动。

## 验证

经过上边的改变后，通过域名就可以访问到项目，并且reality也同时在监听443，代理工具的配置改成443端口，域名也改成自己的就可以正常使用了。我这里就不再贴图了。完美的实现网站和xray共存，xray也成功的偷了自己的证书。

最后祝大家都能成功配置。

参考：

> [steal_yourself](https://github.com/chika0801/Xray-examples/tree/main/VLESS-XTLS-uTLS-REALITY/steal_yourself)
