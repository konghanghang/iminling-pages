---
title: Mac使用Homebrew安装Gradle环境
author: 要名俗气
type: post
date: 2023-10-27T05:24:40+00:00
url: /2023/mac-use-homebrew-install-gradle
description: 开发中需要使用到gradle环境，在mac上我是通过Homebrew来安装的，本文主要讲述如何通过Homebrew来安装gradle并配置环境变量。
image: https://images.iminling.com/app/hide.php?key=b1RZQ08wVXRyaHZ6RWNrVmZJTU53b3VobUVYZG5PM1lPRjVqTTV3QSt5N0IxTzJidFh6VFlCRVhMSGY2cUZGcnR2VG1ob1k9
categories:
  - mac
tags:
  - Gradle
  - Homebrew
  - Mac
---
首先mac机器需要先安装Homebrew,具体的安装教程参考我的另一篇文章：[Mac更换源更快速的安装Homebrew]({{< ref "/post/mac/Mac更换源更快速的安装Homebrew.md" >}})，有了Homebrew后一切安装都变的很简单了。

## 安装

首先是来搜索一下gradle:

```bash
k@MacBook-Pro ~ % brew search gradle
==> Formulae
gradle              gradle-profiler     gradle@7            grails
gradle-completion   gradle@6            grace               glade

==> Casks
grads                                    qlgradle
```

如上有几个版本的gradle:6,7以及没有加版本号的代表最新的。我这里安装gradle@7的版本：

```bash
k@MacBook-Pro ~ % brew install gradle@7
==> Downloading https://ghcr.io/v2/homebrew/core/gradle/7/manifests/7.6.3
######################################################################### 100.0%
==> Fetching gradle@7
==> Downloading https://ghcr.io/v2/homebrew/core/gradle/7/blobs/sha256:cc0def9cb
######################################################################### 100.0%
==> Pouring gradle@7--7.6.3.all.bottle.tar.gz
==> Caveats
gradle@7 is keg-only, which means it was not symlinked into /usr/local,
because this is an alternate version of another formula.

If you need to have gradle@7 first in your PATH, run:
  echo 'export PATH="/usr/local/opt/gradle@7/bin:$PATH"' >> ~/.zshrc
==> Summary
🍺  /usr/local/Cellar/gradle@7/7.6.3: 11,535 files, 272.3MB
==> Running `brew cleanup gradle@7`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
```

经过上边就已经安装好了gradle,可以看到默认安装的是7.6.3版本的gradle,安装目录是/usr/local/Cellar/gradle@7/7.6.3,接下来我们就需要配置一下maven的环境变量。

## 添加环境变量

上边安装完输出的信息中有提示可以直接把上边提示的命令在终端执行就可以添加环境变量到.zshrc文件中：

```
echo 'export PATH="/usr/local/opt/gradle@7/bin:$PATH"' >> ~/.zshrc
```

或者直接编辑.zshrc文件，在文件末尾添加：

```
export PATH="/usr/local/opt/gradle@7/bin:$PATH"
```

然后就是刷新变量信息：

```
source .zshrc
```

然后就可以通过命令来查看gradle的版本信息：

```bash
k@MacBook-Pro ~ % gradle -v

Welcome to Gradle 7.6.3!

Here are the highlights of this release:
 - Added support for Java 19.
 - Introduced `--rerun` flag for individual task rerun.
 - Improved dependency block for test suites to be strongly typed.
 - Added a pluggable system for Java toolchains provisioning.

For more details see https://docs.gradle.org/7.6.3/release-notes.html

------------------------------------------------------------
Gradle 7.6.3
------------------------------------------------------------

Build time:   2023-10-04 15:59:47 UTC
Revision:     1694251d59e0d4752d547e1fd5b5020b798a7e71

Kotlin:       1.7.10
Groovy:       3.0.13
Ant:          Apache Ant(TM) version 1.10.11 compiled on July 10 2021
JVM:          17.0.9 (Homebrew 17.0.9+0)
OS:           Mac OS X 14.1 x86_64
```

以上，Gradle就安装成功。也可以参考官网的安装步骤进行手工下载安装：[Installing manually](https://gradle.org/install/)。
