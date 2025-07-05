---
title: Mac使用Homebrew安装maven环境
author: 要名俗气
type: post
date: 2023-10-27T04:01:20+00:00
url: /2023/mac-use-homebrew-install-maven
description: 首先mac机器需要先安装Homebrew,具体的安装教程参考我的另一篇文章：[Mac更换源更快速的安装Homebrew](https://www.iminling.com/2023/10/02/265.html "Mac更换源更快速的安装Homebrew")，有了Homebrew后一切安装都变的很简单了。
featured_image: /wp-content/uploads/2023/10/C57B82469E62154EDED97C0E6200B82F.png
categories:
  - Maven
tags:
  - Homebrew
  - Mac
  - maven
---
![](https://www.iminling.com/wp-content/uploads/2023/10/C57B82469E62154EDED97C0E6200B82F.png)

首先mac机器需要先安装Homebrew,具体的安装教程参考我的另一篇文章：[Mac更换源更快速的安装Homebrew](https://www.iminling.com/2023/10/02/265.html "Mac更换源更快速的安装Homebrew")，有了Homebrew后一切安装都变的很简单了。

## 安装

首先是来搜索一下maven:

```
k@MacBook-Pro ~ % brew search maven
==> Formulae
maven                      maven-completion           maven-shell

==> Casks
marvel              marvin              mauve               mavensmate
```

如上，第一个就是我们需要安装的maven,通过安装命令进行安装：

```
k@MacBook-Pro ~ % brew install maven
==> Downloading https://formulae.brew.sh/api/formula.jws.json
##O#- #
==> Downloading https://formulae.brew.sh/api/cask.jws.json
##O#- #
==> Downloading https://ghcr.io/v2/homebrew/core/maven/manifests/3.9.5
######################################################################### 100.0%
==> Fetching dependencies for maven: openjdk
==> Downloading https://ghcr.io/v2/homebrew/core/openjdk/manifests/21.0.1
######################################################################### 100.0%
==> Fetching openjdk
==> Downloading https://ghcr.io/v2/homebrew/core/openjdk/blobs/sha256:ab5d32ae29
######################################################################### 100.0%
==> Fetching maven
==> Downloading https://ghcr.io/v2/homebrew/core/maven/blobs/sha256:3c0cb2c2df11
######################################################################### 100.0%
==> Installing dependencies for maven: openjdk
==> Installing maven dependency: openjdk
==> Downloading https://ghcr.io/v2/homebrew/core/openjdk/manifests/21.0.1
Already downloaded: /Users/k/Library/Caches/Homebrew/downloads/bb8a78296beb0d39c64166622e2df1a553bdf78572bac0ce95765a7007a46b8a--openjdk-21.0.1.bottle_manifest.json
==> Pouring openjdk--21.0.1.sonoma.bottle.tar.gz
🍺  /usr/local/Cellar/openjdk/21.0.1: 600 files, 331.4MB
==> Installing maven
==> Pouring maven--3.9.5.sonoma.bottle.tar.gz
🍺  /usr/local/Cellar/maven/3.9.5: 92 files, 10.4MB
==> Running `brew cleanup maven`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
```

经过上边就已经安装好了maven,可以看到默认安装的是3.9.5版本的maven,安装目录是/usr/local/Cellar/maven/3.9.5,接下来我们就需要配置一下maven的环境变量。

## 添加环境变量

编辑~/.zshrc文件,在文件末尾添加环境变量信息：

```
# maven
export PATH=/usr/local/Cellar/maven/3.9.5/bin:$PATH
```

添加完环境变量信息后，我们还需要让环境变量生效，可以使用source：

```
source .zshrc
```

然后就可以通过命令来查看maven的版本信息：

```
k@MacBook-Pro ~ % mvn -v
Apache Maven 3.9.5 (57804ffe001d7215b5e7bcb531cf83df38f93546)
Maven home: /usr/local/Cellar/maven/3.9.5/libexec
Java version: 17.0.9, vendor: Homebrew, runtime: /usr/local/Cellar/openjdk@17/17.0.9/libexec/openjdk.jdk/Contents/Home
Default locale: en_CN, platform encoding: UTF-8
OS name: "mac os x", version: "14.1", arch: "x86_64", family: "mac"
```

至此maven就安装完成了，我这里使用的是brew来进行安装，除了这个方法，还可以直接去maven官网进行下载maven的压缩包，然后进行解压，再进行环境变量的配置就可以了。maven官网的下载地址：[Maven - Download](https://maven.apache.org/download.cgi)，这里默认下载的是最新版本，如果想下载历史的版本可以去到历史库下载，具体地址：[Maven Releases History](https://maven.apache.org/docs/history.html)。
