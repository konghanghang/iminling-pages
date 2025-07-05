---
title: x-ui安装配置使用
author: 要名俗气
type: post
date: 2023-03-26T01:39:01+00:00
url: /2023/x-ui-configuration-and-use
description: 今天我们主要来介绍一下x-ui的安装使用。 x-ui在原地址是：[x-ui](https://github.com/vaxilu/x-ui) 但是由于原仓库太久没更新，我们今天使用的是改版的FranzKafkaYu/x-ui,支持单个端口多用户，还能分别统计流量，比原版较完善。
featured_image: /wp-content/uploads/2023/03/1680014847697-480x300.jpg
categories:
  - VPS
  - x-ui
  - Xray
tags:
  - x-ui
  - xray
---
今天我们主要来介绍一下x-ui的安装使用。

x-ui在原地址是：[x-ui](https://github.com/vaxilu/x-ui)

<del>但是由于原仓库太久没更新，我们今天使用的是改版的<a href="https://github.com/FranzKafkaYu/x-ui">FranzKafkaYu/x-ui</a>,支持单个端口多用户，还能分别统计流量，比原版较完善。</del>

本文使用改版[MHSanaei/3x-ui](https://github.com/MHSanaei/3x-ui "MHSanaei/3x-ui")进行配置。

## 安装

使用github仓库中的一键安装脚本很容易就可以安装完成安装：

<pre class="corepress-code-pre">bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)</pre>

安装完成如下：

```
Saving to: ‘/usr/local/x-ui-linux-amd64.tar.gz’

/usr/local/x-ui-linux- 100%[=========================>]  27.95M   124MB/s    in 0.2s

2023-03-26 08:10:51 (124 MB/s) - ‘/usr/local/x-ui-linux-amd64.tar.gz’ saved [29309974/29309974]

x-ui/
x-ui/x-ui.sh
x-ui/bin/
x-ui/bin/README.md
x-ui/bin/LICENSE
x-ui/bin/geosite.dat
x-ui/bin/geoip.dat
x-ui/bin/xray-linux-amd64
x-ui/x-ui
x-ui/x-ui.service
--2023-03-26 08:10:52--  https://raw.githubusercontent.com/FranzKafkaYu/x-ui/main/x-ui.sh
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.109.133, 185.199.110.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.109.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 25694 (25K) [text/plain]
Saving to: ‘/usr/bin/x-ui’

/usr/bin/x-ui          100%[=========================>]  25.09K  --.-KB/s    in 0.001s

2023-03-26 08:10:52 (34.8 MB/s) - ‘/usr/bin/x-ui’ saved [25694/25694]

出于安全考虑，安装/更新完成后需要强制修改端口与账户密码
确认是否继续,如选择n则跳过本次端口与账户密码设定[y/n]:y
请设置您的账户名:admin
您的账户名将设定为:admin
请设置您的账户密码:admin
您的账户密码将设定为:admin
请设置面板访问端口:8080
您的面板访问端口将设定为:8080
确认设定,设定中
set username and password success
账户密码设定完成
set port 11000 success面板端口设定完成
Created symlink /etc/systemd/system/multi-user.target.wants/x-ui.service → /etc/systemd/system/x-ui.service.
x-ui v0.3.4.1 安装完成，面板已启动
```

安装完成后我们就可以进行登录了，使用我们vps的ip地址加上安装的时候设置的端口就可以正常登录了。

## 配置域名访问

使用ip加端口的形式多少有点不怎么方便，我们可以配置nginx来进行反向代理，使用域名来访问我们的x-ui的界面。

### x-ui配置

第一次需要使用ip+端口的形式进行登录，登录x-ui后我们进入`面板设置`-`面板url根路径配置`，默认是`/`,我们将它改为我们想要使用的路径，我这里改为`/ui`.

<div id="attachment_389" style="width: 900px" class="wp-caption alignnone">
  ![xui配置](https://www.iminling.com/wp-content/uploads/2023/03/61B14FAC174D72C1E8A8E657FB1C7727.png)

  <p id="caption-attachment-389" class="wp-caption-text">
    xui配置
  </p>
</div>

### nginx配置 {.ant-list-item-meta-title}

配置nginx的配置文件，添加以下转发配置：

```
server {
    listen       80;
    server_name  _;

    #access_log  /var/log/nginx/host.access.log  main;

    location /ui {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_ignore_client_abort on;
        proxy_pass http://172.17.0.1:8080;
    }
}
```

如果不限制域名server_name配置为`_`就可以了，如果要限制域名，则配置为对应的域名。

就可以通过域名直接进行访问了。

## 节点添加

现在最常用的节点还是vless节点，下边就添加一个vless节点来使用：

<div id="attachment_390" style="width: 910px" class="wp-caption alignnone">
  ![vless](https://www.iminling.com/wp-content/uploads/2023/03/2D50578117F812968AE4D838CC57F379.png)

  <p id="caption-attachment-390" class="wp-caption-text">
    vless
  </p>
</div>

用户就使用xui自己生成的用户信息就可以了。

flow需要改成xtls-rprx-vision,

安全选项使用Reality，然后下边的都可以自动生成，

serverNames可以使用xui自己生成的，也可以使用我们自己找的。

然后就是保存，去相关的客户端里按照我们的配置进行填写，然后就可以正常上网了。

这里演示一下openwrt中的配置：

<div id="attachment_391" style="width: 910px" class="wp-caption alignnone">
  ![openwrt-vless](https://www.iminling.com/wp-content/uploads/2023/03/CBE7615DE20625625BA4CFE3F9C740E8.png)

  <p id="caption-attachment-391" class="wp-caption-text">
    openwrt-vless
  </p>
</div>

基本就是对着xui里的进行抄就行了，地址就是自己的vps地址，端口在xui的截图里没有截取到，按自己添加的端口进行填写就行了。然后就可以愉快的上网了。
