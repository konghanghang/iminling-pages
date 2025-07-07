---
title: 自定义Nginx日志属性并设置日志以JSON输出
author: 要名俗气
type: post
date: 2023-10-04T15:04:07+00:00
url: /2023/custom-nginx-log-output-json
description: 日志在日常开发中是非常重要的东西，在出现错误后可以快速的进行排查，对Nginx而言同样也很重要，同Nginx的日志我们可以清晰的了解什么url被访问，是什么Ip访问过来等一些重要信息，所以无论是在开发中还是线上环境，都会开启日志。下边就来了解一下Nginx的日志。
image: https://images.iminling.com/app/hide.php?key=eEZmbmFBSW14MCtoRHpiR0RHeW52cHZqTmNJRU1Zblg0cCtiYWo1Z3VSbHNiNUY2cHF0OHJKZHlpbGVCQUpGRVRMWmRwYXM9
categories:
  - nginx
tags:
  - nginx
---
日志在日常开发中是非常重要的东西，在出现错误后可以快速的进行排查，对Nginx而言同样也很重要，同Nginx的日志我们可以清晰的了解什么url被访问，是什么Ip访问过来等一些重要信息，所以无论是在开发中还是线上环境，都会开启日志。下边就来了解一下Nginx的日志。

## 日志类型

在Nginx的配置文件中，日志相关的配置可以出现在_**http,server,location**_中，通常我们会设置以下格式：

```
server {
    listen                  443 ssl http2;
    server_name             _;

    index                   index.php index.html index.htm;
    root                    /var/www/html;
    server_tokens           off;
    client_max_body_size    75M;

    access_log              /var/log/access.log;
    error_log               /var/log/error.log;

}
```

设置了access和error两种类型的日志，日志的设置语法是：

```
access_log path [format [buffer=size] [gzip[=level]] [flush=time] [if=condition]];
access_log off;
```

可以设置开启日志，也可以设置关闭日志。access\_log中记录了所有的访问日志，error\_log中则记录访问中的一些异常请求，方便在出现问题的时候进行错误排查。

另外也可以定义缓存区以及刷新时间等属性。

如果使用 gzip 参数，则缓冲数据将在写入文件之前被压缩。压缩级别可以设置在 1（最快，压缩程度较低）和 9（最慢，压缩效果最佳）之间。默认情况下，缓冲区大小等于64K字节，压缩级别设置为1。由于数据被压缩在原子块中，因此日志文件可以随时由“zcat”解压或读取。使用gzip有一个要求，那就是在构建Nginx必须使用 zlib 库构建。

以上只是设置了访问日志和错误日志的记录位置或者不记录日志，那么日志记录的格式是怎样的呢？

## 日志格式

上边的配置没有指定具体的日志格式，那么Nginx有一个默认的日志记录格式：

```
log_format combined '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent"';
```

它的名称叫combined,具体log_format的语法如下：

```
log_format name [escape=default|json|none] string ...;
```

每一个日志格式都有一个名称，然后在配置access\_log的时候可以指定用哪种格式来进行记录日志。log\_format只能定义在_**http**_块中。查看在Nginx配置文件中的定义：

```
http {

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
}

server {
    access_log              /var/log/nginx/access.log main;
    error_log               /var/log/nginx/error.log;
}
```

如上在http块中定义了一个名称为main的日志格式，在server块中通过名称对该日志格式进行了引用，这样子日志的格式都会按找main中定义的一些属性来进行记录访问信息。

escape 参数（1.11.8）允许在变量中设置 json 或默认字符转义，默认情况下使用默认转义。 none 值 (1.13.10) 禁用转义。

对于默认转义，字符“"”、“\”以及其他值小于32（0.7.0）或大于126（1.1.6）的字符将被转义为“\xXX”。如果找不到变量值，将记录连字符（“-”）。

对于json转义，JSON字符串中不允许的所有字符都将被转义：字符“"”和“\”被转义为“\”和“\\”，值小于32的字符被转义为“\n”， “\r”、“\t”、“\b”、“\f”或“\u00XX”。

但是这些属性究竟什么意思？Nginx有哪些属性可以使用呢？

## 日志属性

日志属性可以包含公共变量以及仅在日志写入时存在的变量：

  * $bytes_sent：发送到客户端的字节数
  * $connection：连接序列号
  * $connection_requests：当前通过连接发出的请求数 (1.1.18)
  * $msec：日志写入时的时间（以秒为单位，精度为毫秒）
  * $pipe：如果请求是通过管道传输的，则为“p”，“.”否则
  * $request_length：请求长度（包括请求行、请求头和请求正文）
  * $request_time：请求处理时间以秒为单位，精度为毫秒；从客户端读取第一个字节与将最后一个字节发送到客户端后写入日志之间经过的时间
  * $status：响应状态
  * $time_iso8601：ISO 8601 标准格式的当地时间
  * $time_local：通用日志格式中的本地时间

具体的参考：[Module ngx\_http\_log_module](http://nginx.org/en/docs/http/ngx_http_log_module.html#log_format)

## 设置JSON格式

通过上边对日志格式和日志文件的一些了解，我们就可以非常容易的把日志的格式设置成JSON了，只需要定义一个log_format，把它的格式定义成一个json字符串，然后在打印日志的时候引用这个格式就可以实现了，下边是一个完成的定义文件:

```
http {

    log_format  json escape=json '{'
                            '"msec": "$msec", ' # request unixtime in seconds with a milliseconds resolution
                            '"connection": "$connection", ' # connection serial number
                            '"connection_requests": "$connection_requests", ' # number of requests made in connection
                    '"pid": "$pid", ' # process pid
                    '"request_id": "$request_id", ' # the unique request id
                    '"request_length": "$request_length", ' # request length (including headers and body)
                    '"remote_addr": "$remote_addr", ' # client IP
                    '"remote_user": "$remote_user", ' # client HTTP username
                    '"remote_port": "$remote_port", ' # client port
                    '"time_local": "$time_local", '
                    '"time_iso8601": "$time_iso8601", ' # local time in the ISO 8601 standard format
                    '"request": "$request", ' # full path no arguments if the request
                    '"request_uri": "$request_uri", ' # full path and arguments if the request
                    '"args": "$args", ' # args
                    '"status": "$status", ' # response status code
                    '"body_bytes_sent": "$body_bytes_sent", ' # the number of body bytes exclude headers sent to a client
                    '"bytes_sent": "$bytes_sent", ' # the number of bytes sent to a client
                    '"http_referer": "$http_referer", ' # HTTP referer
                    '"http_user_agent": "$http_user_agent", ' # user agent
                    '"http_x_forwarded_for": "$http_x_forwarded_for", ' # http_x_forwarded_for
                    '"http_host": "$http_host", ' # the request Host: header
                    '"server_name": "$server_name", ' # the name of the vhost serving the request
                    '"request_time": "$request_time", ' # request processing time in seconds with msec resolution
                    '"upstream": "$upstream_addr", ' # upstream backend server for proxied requests
                    '"upstream_connect_time": "$upstream_connect_time", ' # upstream handshake time incl. TLS
                    '"upstream_header_time": "$upstream_header_time", ' # time spent receiving upstream headers
                    '"upstream_response_time": "$upstream_response_time", ' # time spend receiving upstream body
                    '"upstream_response_length": "$upstream_response_length", ' # upstream response length
                    '"upstream_cache_status": "$upstream_cache_status", ' # cache HIT/MISS where applicable
                    '"ssl_protocol": "$ssl_protocol", ' # TLS protocol
                    '"ssl_cipher": "$ssl_cipher", ' # TLS cipher
                    '"scheme": "$scheme", ' # http or https
                    '"request_method": "$request_method", ' # request method
                    '"server_protocol": "$server_protocol", ' # request protocol, like HTTP/1.1 or HTTP/2.0
                    '"pipe": "$pipe", ' # "p" if request was pipelined, "." otherwise
                    '"gzip_ratio": "$gzip_ratio", '
                    '"http_cf_ray": "$http_cf_ray"'
                    '}';
}

server {
    listen                  443 ssl http2;
    server_name             _;
    access_log              /var/log/nginx/access.log json;

}
```

如上定义了一个完成的json格式的输出形式，实际的请求输出是这样子的：

```
{
    "msec": "1696428300.136",
    "connection": "849615",
    "connection_requests": "1",
    "pid": "89",
    "request_id": "f17e4e5dcda30462ac83b120f05fedab",
    "request_length": "619",
    "remote_addr": "65.108.78.33",
    "remote_user": "",
    "remote_port": "",
    "time_local": "04/Oct/2023:22:05:00 +0800",
    "time_iso8601": "2023-10-04T22:05:00+08:00",
    "request": "GET /category/%E4%BF%A1%E7%94%A8%E5%8D%A1 HTTP/1.1",
    "request_uri": "/category/%E4%BF%A1%E7%94%A8%E5%8D%A1",
    "args": "",
    "status": "200",
    "body_bytes_sent": "12354",
    "bytes_sent": "12987",
    "http_referer": "",
    "http_user_agent": "Mozilla/5.0 (compatible; MJ12bot/v1.4.8; http://mj12bot.com/)",
    "http_x_forwarded_for": "65.108.78.33",
    "http_host": "www.iminling.com",
    "server_name": "www.iminling.com,iminling.com",
    "request_time": "0.093",
    "upstream": "192.168.0.3:9000",
    "upstream_connect_time": "0.000",
    "upstream_header_time": "0.092",
    "upstream_response_time": "0.093",
    "upstream_response_length": "49160",
    "upstream_cache_status": "",
    "ssl_protocol": "",
    "ssl_cipher": "",
    "scheme": "http",
    "request_method": "GET",
    "server_protocol": "HTTP/1.1",
    "pipe": ".",
    "gzip_ratio": "3.98",
    "http_cf_ray": "810dfa2978c4376f-HEL",
    "geoip_country_code": "FI",
    "geoip_city": "Helsinki" #这里的国家和城市需要开启新的功能，后续再详细说。
}
```

以上就是对日志格式的说明以及如何实现以json格式输出日志。
