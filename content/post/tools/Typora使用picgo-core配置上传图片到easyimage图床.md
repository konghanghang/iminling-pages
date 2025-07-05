---
title: Typora使用picgo-core配置上传图片到easyimage图床
author: 要名俗气
type: post
date: 2025-04-05T02:38:24+00:00
excerpt: typora是一个非常方便的markdown编辑器，如果我们不做配置，图片仅仅只是存储在本地磁盘，如果本地磁盘出现问题，那么图片就全没了，所以今天就研究一下搭配picgo-core来把插入的图片上传到图床，我这里使用的是easyimage图床。
url: /2025/typora-how-to-use-picgo-upload-image-to-easyimage
description: Typora使用picgo-core配置上传图片到easyimage图床aaa @ Mac-mini-m4 in ~ [10:35:56] $ brew search nodejs ==Formulae node nodenv node@22 node@20 node@18 node@16 node@14 ==Casks nodeclipse aaa @ M...
sha:
  - 2024e94952fda72d9bab2e988e67c9632517e8e4
github_url:
  - https://github.com/konghanghang/blog_pages/blob/master/wordpress/typora-how-to-use-picgo-upload-image-to-easyimage.md
categories:
  - wordpress
tags:
  - easyimage
  - picgo
  - typora
---
# Typora使用picgo-core配置上传图片到easyimage图床<figure class="wp-block-image size-full">

![Typora 0.10 beta - Typora Support](https://images.iminling.com/app/hide.php?key=ZXNZbUNpQ1NObTUvdVJuVFlEa0xZcURTZmgzM2FISnV5MHVGQjR4OVRGSXJSWHRORmFkSTFZdkRWaEd2UFI5bDh0VWttZTQ9)
</figure>

typora是一个非常方便的markdown编辑器，如果我们不做配置，图片仅仅只是存储在本地磁盘，如果本地磁盘出现问题，那么图片就全没了，所以今天就研究一下搭配picgo-core来把插入的图片上传到图床，我这里使用的是easyimage图床。

picgo可以直接安装官方app，也可以使用node命令去进行配置，我这里为了简单直接使用node的方式去配置，在此之前请确定在本机已正确安装了node的环境。我使用的是mac，且也已配置好brew环境，可以直接使用brew来进行安装。

## 安装node

搜索可使用的node版本并安装，这里安装18版本。

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~ [10:35:56]
$ brew search nodejs
==> Formulae
node       nodenv     node@22    node@20    node@18    node@16    node@14

==> Casks
nodeclipse

# aaa @ Mac-mini-m4 in ~ [10:36:04]
$ brew install node@18
==> node@18
node@18 is keg-only, which means it was not symlinked into /opt/homebrew,
because this is an alternate version of another formula.

If you need to have node@18 first in your PATH, run:
  echo 'export PATH="/opt/homebrew/opt/node@18/bin:$PATH"' >> ~/.zshrc

For compilers to find node@18 you may need to set:
  export LDFLAGS="-L/opt/homebrew/opt/node@18/lib"
  export CPPFLAGS="-I/opt/homebrew/opt/node@18/include"</code></pre>

安装后还需要添加环境变量，通过上面给的命令来执行`echo 'export PATH="/opt/homebrew/opt/node@18/bin:$PATH"' >> ~/.zshrc`，最后再刷新环境变量

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~ [10:38:36]
$ source .zshrc

# aaa @ Mac-mini-m4 in ~ [10:38:43]
$ node -v
v18.20.8</code></pre>

最后通过npm来安装picgo:

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~ [10:38:48]
$ npm install picgo -g</code></pre>

安装情况后可以使用picgo set uploader来设置上传配置，但是默认情况下不支持自定义图床，只能选定他给的几种：

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~ [10:39:06]
$ picgo set uploader
? Choose a(n) uploader
  aliyun
  tcyun
  smms
  github
  qiniu
  imgur
❯ upyun </code></pre>

## 安装web-uploader

通过easyimage官方的文档来看，应该是有一个插件可以安装来支持自定义图床上传的：[使用PicGo上传.md](https://github.com/icret/EasyImages2.0/blob/master/docs/%E4%BD%BF%E7%94%A8PicGo%E4%B8%8A%E4%BC%A0.md), 插件的名称是`web-uploader`，那怎么安装呢？通过picgo的命令来查看：

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~ [10:54:19]
$ picgo -h
Usage: picgo [options] [command]

Options:
  -v, --version                        output the version number
  -d, --debug                          debug mode
  -h, --help                           display help for command

Commands:
  install|add [options] <plugins...>   install picgo plugin
  uninstall|rm <plugins...>            uninstall picgo plugin</code></pre>

可以看到可以使用install或add命令来安装。

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~ [11:10:45]
$ picgo install web-uploader

added 1 package, and audited 2 packages in 2s

found 0 vulnerabilities
[PicGo SUCCESS]: 插件安装成功</code></pre>

picgo的配置文件所在位置如下：

>   * Linux / macOS → `~/.picgo/config.json`.
>   * Windows → `C:Users[your user name].picgoconfig.json`.

查看里边的目录情况：

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~ [11:17:34]
$ tree .picgo
.picgo
├── config.json
├── i18n-cli
├── node_modules
├── package-lock.json
├── package.json
└── picgo.log</code></pre>

在package里记录了所有的插件信息：

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~ [11:11:14] C:1
$ cat .picgo/package.json
{"name":"picgo-plugins","description":"picgo-plugins","repository":"https://github.com/PicGo/PicGo-Core","license":"MIT","dependencies":{"picgo-plugin-web-uploader":"^1.1.1"}}%   </code></pre>

## 配置自定义图床

安装web-uploader后，再次执行set uploader命令查看：

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~ [11:21:32]
$ picgo set uploader
? Choose a(n) uploader
  smms
  github
  qiniu
❯ imgur
  upyun
  web-uploader
  aliyun
(Move up and down to reveal more choices)</code></pre>

可以看到多了一个web-uploader选项，下边进行来设置：

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~ [11:21:32]
$ picgo set uploader
? Choose a(n) uploader web-uploader
? API地址 https://images.xxx.com/api/index.php
? POST参数名 image
? 图片URL JSON路径(eg: data.url) url
? 自定义请求头 标准JSON(eg: {"key":"value"})
? 自定义Body 标准JSON(eg: {"key":"value"}) {"token":"aaaaaa"}</code></pre>

每个参数是什么意义可以参考[使用PicGo上传.md](https://github.com/icret/EasyImages2.0/blob/master/docs/%E4%BD%BF%E7%94%A8PicGo%E4%B8%8A%E4%BC%A0.md),其中api地址是自己网站的地址，post参数和图片URL JSON路径是固定，自定义请求头标准JSON可以置空，自定义body标准JSON则是在easyimage后台获取到的token替换我上边的`aaaaaa`;<figure class="wp-block-image size-full">

![image-20250404135028240](https://images.iminling.com/app/hide.php?key=c0VrUS92K2Nka3VXbEpsc2xBaHc1Wkt0M3pBMjlsNnpSbjN6TDlQQnkwcTZRZTBlYkZBMVprZTNCZEVVamlDbGVYMVcrR2s9)
</figure>

另外需要自己开启支持API上传：<figure class="wp-block-image size-full">

![image-20250404135121045](https://images.iminling.com/app/hide.php?key=ZTJDMEJjSFZtRm5aWXNZR3dsYlhHeWJTRnBaN0taa3hrVG9YTE1YMVZVWmVZK0RNL3ZkL3VzTVR1SFRJdHh3SGZGb1ZXOU09)
</figure>

## Typora配置

在typora的设置中，选择插入图片时的动作，这里选择上传图片。

主要是上传服务设置，因为我这里没有安装picgo的客户段，而是使用的picgo-core来搭配node使用，所以上传服务选择自定义命令，命令的内容是：

`{node位置} {picgo位置} u`.<figure class="wp-block-image size-full">

![image-20250404135452149](https://images.iminling.com/app/hide.php?key=NFNUbFRLbXJwODZUKzVnTWp4QWpCQzYzUUd3SWtRenZRRmtEeGV4S3Vra240cWdCYUthbGNvWTNaNndGNnpwUFJseUFycm89)
</figure>

node的位置在安装node的时候就已经知道了，我们还把bin添加进了PATH中，具体位置：`/opt/homebrew/opt/node@18/bin/node`,那么安装的picgo在什么位置呢？通过以下命令来查看：

<pre><code class="language-shell"># aaa @ Mac-mini-m4 in ~/.npm [12:01:34]
$ npm list --depath=0
/Users/hangkong/.npm
└── (empty)

# aaa @ Mac-mini-m4 in ~/.npm [12:02:54]
$ npm list --depath=0 --global
/opt/homebrew/lib
└── picgo@1.5.8</code></pre>

可以看到位置在`/opt/homebrew/lib`目录下，完整的路径为：`/opt/homebrew/lib/node_modules/picgo/bin/picgo`。

那么设置的完整命令就是：`/opt/homebrew/opt/node@18/bin/node /opt/homebrew/lib/node_modules/picgo/bin/picgo u`。

经过上边的配置，就可以在使用typora插入图片的时候直接上传的easyimage图床了。
