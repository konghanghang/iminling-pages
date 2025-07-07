---
title: docker搭建wordpress教程
author: 要名俗气
type: post
date: 2023-03-07T06:59:39+00:00
url: /2023/docker-install-wordpress
description: 搭建一个wordpress网站。本文主要介绍使用docker搭建一个完整的wordpress网站。 安装docker就不做多介绍了. 准备wordpress的docker-compose.yaml文件： 使用官方的6.1.0版本的镜像，把文件中的数据库的信息改成自己的就可以正常启动了。
image: https://images.iminling.com/app/hide.php?key=U3grOTJQSUZnQ1N0T2hlU0gydlRwbGdXalNiNmdMSGl4Rk1XNTF2dE0ydlpydmFkZXA0WFhsUEZ5d0o5MnZ4c2tHTkEzSm89
categories:
  - wordpress
tags:
  - wordpress
---
搭建一个wordpress网站。本文主要介绍使用docker搭建一个完整的wordpress网站。

安装docker就不做多介绍了.

准备wordpress的docker-compose.yaml文件：

```yaml
version: '3.9'
networks:
  mynet:
    external: true
services:
  wordpress:
    # default port 9000 (FastCGI)
    image: wordpress:6.1.0-fpm
    container_name: wordpress
    env_file:
      - .env
    restart: unless-stopped
    networks:
      - mynet
    volumes:
      - ${WORDPRESS_LOCAL_HOME}:/var/www/html
      - ${WORDPRESS_UPLOADS_CONFIG}:/usr/local/etc/php/conf.d/uploads.ini
      # - /path/to/repo/myTheme/:/var/www/html/wp-content/themes/myTheme
    environment:
      - WORDPRESS_DB_HOST=${WORDPRESS_DB_HOST}
      - WORDPRESS_DB_NAME=${WORDPRESS_DB_NAME}
      - WORDPRESS_DB_USER=${WORDPRESS_DB_USER}
      - WORDPRESS_DB_PASSWORD=${WORDPRESS_DB_PASSWORD}
  nginx:
    image: nginx:1.22.0
    container_name: nginx
    env_file:
      - .env
    restart: unless-stopped
    networks:
      - mynet
    depends_on:
      - wordpress
    ports:
      - "80:80"    # http
    volumes:
      - ${WORDPRESS_LOCAL_HOME}:/var/www/html
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
      - ./logs:/var/log/nginx
```

使用官方的6.1.0版本的镜像，把文件中的数据库的信息改成自己的就可以正常启动了。
端口这里没有暴露出来，因为我想使用nginx来处理所有请求，所以还需要配置一下nginx的代理规则：

```bash
server {
    listen 80;
    listen [::]:80;
    server_name ${HOST};
    index index.php index.html index.htm;
    root /var/www/html;
    server_tokens off;
    client_max_body_size 75M;
    # update ssl files as required by your deployment
    #ssl_certificate     /etc/ssl/fullchain.pem;
    #ssl_certificate_key /etc/ssl/privkey.pem;

    # logging
    access_log /var/log/nginx/wordpress.access.log;
    error_log  /var/log/nginx/wordpress.error.log;

    # some security headers ( optional )
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src * data: 'unsafe-eval' 'unsafe-inline'" always;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        try_files $uri = 404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~ /\.ht {
        deny all;
    }

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /favicon.svg {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        log_not_found off;
        access_log off;
        allow all;
    }

    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }
}
```

完成后就可以使用域名来访问了。

也可以参考我的github仓库中的docker-compose.yaml文件：[wordpress](https://github.com/konghanghang/images/tree/master/wordpress)
