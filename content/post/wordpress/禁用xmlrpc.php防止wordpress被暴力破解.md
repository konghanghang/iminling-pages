---
title: 禁用xmlrpc.php防止wordpress被暴力破解
author: 要名俗气
type: post
date: 2023-09-03T09:07:36+00:00
url: /2023/disable-xmlrpc-php
description: 很多刚接触wordpress的朋友估计和我一样，对于xmlrpc.php的了解几乎为0，这玩意儿是干嘛的？为什么它会导致wordpress被暴力破解？本文我们就先来认识一下xmlrpc.php是个什么东西。 
image: https://images.iminling.com/app/hide.php?key=NnVNTjhMWXJ5N2tkOGpzYWFnVWJuYnhhOGFtQnZmYzc2S3BzdDJhY29HUEUvUHU5TkZVRk90VkgyTzlHUzMyVDAvMDlXcDg9
categories:
  - wordpress
tags:
  - wordpress
  - xmlrpc
---
很多刚接触wordpress的朋友估计和我一样，对于xmlrpc.php的了解几乎为0，这玩意儿是干嘛的？为什么它会导致wordpress被暴力破解？下面我们就先来认识一下xmlrpc.php是个什么东西。

## 什么是xmlrpc.php

XML-RPC是支持WordPress与其他系统之间通信的规范。它通过使用HTTP作为传输机制和XML作为编码机制来标准化这些通信来实现此目的。例如，假设您想通过移动设备在您的网站上发帖，因为您的计算机不在附近。 您可以使用 xmlrpc.php 启用的远程访问功能来做到这一点。

在WordPress的早期版本中，默认情况下已关闭XML-RPC。但是自v3.5版本开始，默认情况下又启用它。这样做的主要原因是允许WordPress移动应用程序与WordPress安装进行对话通讯。

但是由于REST API已集成到WordPress核心中，因此xmlrpc.php文件不再用于此通信。相反，REST API用于与WordPress移动应用程序，桌面客户端，其他博客平台，WordPress.com（用于Jetpack插件）以及其他系统和服务进行通信。REST API-可与之交互的系统范围比xmlrpc.php所允许的大得多。此外，拥有更强的灵活性。

## 经历

为什么我会知道这个玩意儿呢？因为在某一天的早上，我发现我的网站突然就多了两篇文章，并不是我写的，莫名其妙的就多了两篇，于是马上修改了管理用户的登录密码，防止别人再登录。之后便开始了此次被攻击的思考，我的密码设置的也并不简单，他们是怎么知道我的密码的呢？于是我就开始了查看请求日志，在nginx的日志中果然发现了一些端倪，有大量的请求在请求xmlrpc.php这个文件：

![](https://images.iminling.com/app/hide.php?key=RGdFaVV3bXgyNmpWcSt0aXIxeVRDTUVVY2tFaStNSkxibDUzK0U0L3NrczJKUEpvVmZJbysxWGd4cnROditaUVNxd00zNWs9)

于是我就去搜索了xmlrpc.php这个文件，不搜索不知道，一搜索吓一跳，md，肯定是这玩意儿导致我的网站多了两篇文章出来。于是开始了探索如何禁用之路。

## 禁用xmlrpc.php

既然wordpress出了REST API取代XML-RPC，为了安全我们应该在站点上禁用xmlrpc.php。接下来介绍三种禁用的方案。

### 通过nginx过滤请求

这种是我在使用的方案，通过nginx的location来定位这个请求，然后相应403，禁止请求访问。关于location的用法，参考我的另一篇文章：[nginx中location使用方法总结](https://www.iminling.com/2023/08/30/224.html "nginx中location使用方法总结")。在处理wordpress站点的server代码块中添加以下代码来阻止其他人访问xmlrpc.php:

```
location ~* ^/xmlrpc\.php$ {
    return 403;
}
```

添加以后，再有人访问xmlrpc.php文件的时候就会返回403.

### 通过functions.php禁用

刚开始我使用的就是这种方式来禁用的，但是在更新了主题文件后，设置丢失了，所以就换成了nginx来进行拦截，这里也顺带说一下这种方式。在后台中，找到外观-主题文件编辑器菜单，然后点击右侧的functions.php,在编辑区将以下方法添加进去保存文件就可以了。

```
add_filter('xmlrpc_enabled', '__return_false');
```

保存后，就没办法再访问xmlrpc.php文件了。

### 通过.htaccess 文件禁用

.htaccess文件在/var/www/html目录下，是一个隐藏文件，我们编辑这个文件，在文件中添加以下内容来达到屏蔽xmlrpc.php文件的目的：

```
# Block WordPress xmlrpc.php requests
<Files xmlrpc.php>
order deny,allow
deny from all
allow from xxx.xxx.xxx.xxx
</Files>
```

可以禁用所有调用该文件的情况，但是可以设置例外，我们可以把允许的ip替换xxx.xxx.xxx.xxx来达到只允许特定的ip来访问该文件。

以上就是我因为xmlrpc.php文件遭遇到的问题以及针对这种情况进行的防御，希望可以帮到大家。
