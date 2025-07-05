---
title: nginx配置隐藏版本号提升安全
author: 要名俗气
type: post
date: 2023-08-05T19:21:16+00:00
url: /2023/nginx-hides-version-number
description: servertokens off就可以了，servertokens可以到http,server和location块中，我们找一个合适的位置添加进去就可以了。
featured_image: /wp-content/uploads/2023/08/459feb46c0c414a826f2055d8d28cd27.png
categories:
  - nginx
tags:
  - nginx
---
> <div>
>
> </div>

<div>
  <p>
    ![nginx](https://www.iminling.com/wp-content/uploads/2023/08/459feb46c0c414a826f2055d8d28cd27.png)
  </p>
</div>

<div>
  Nginx (engine x) 是一个高性能的HTTP和反向代理web服务器  ，同时也提供了IMAP/POP3/SMTP服务，是个软件肯定都会多多少少有一些漏洞，所以我们需要隐藏nginx的版本号，让攻击者没那么容易很快的攻破我们的网站。
</div>

<div>
</div>

<div>
  nginx官方提供了关闭显示版本号的方法，其实也很简单，只需要添加<code>server_tokens off</code>就可以了，server_tokens可以到http,server和location块中，我们找一个合适的位置添加进去就可以了。
</div>

## 添加前

<div>
  没有添加之前请求网站信息如下：
</div>

<div>
  <pre class="core-next-code-pre"><code>$ curl -I https://www.iminling.com
HTTP/2 200
server: nginx/2.24.2
date: Sun, 06 Aug 2023 03:02:12 GMT
content-type: text/html; charset=UTF-8
x-powered-by: PHP
link: <https://www.iminling.com/wp-json/>; rel="https://api.w.org/"</code></pre>

<p>
  如上，server中会把nginx的完整版本号信息展示出来。
</p>
</div>

## 添加server_token

我这里把server_token添加到server块中，如下：

<pre class="core-next-code-pre"><code>server {

    listen                  8443 ssl http2;
    server_name www.iminling.com;

    index index.php index.html index.htm;
    root /var/www/html;
    server_tokens off; # 添加server_tokens屏蔽显示版本号

    include conf.d/ssl_params.conf;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }
}</code></pre>

如上，在server中添加server_tokens为off，然后我们再去验证一下。

## 添加后

再来看一下请求网站的信息：

<pre class="core-next-code-pre"><code>$ curl -I https://www.iminling.com
HTTP/2 200
server: nginx
date: Sun, 06 Aug 2023 03:02:31 GMT
content-type: text/html; charset=UTF-8
x-powered-by: PHP
link: <https://www.iminling.com/wp-json/>; rel="https://api.w.org/"</code></pre>

如上，server里只显示了nginx。

经过上边的配置就可以隐藏nginx的版本号了。