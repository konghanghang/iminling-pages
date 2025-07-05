---
title: Mac快速安装jdk17环境
author: 要名俗气
type: post
date: 2023-10-03T02:55:28+00:00
url: /2023/mac-install-jdk17
description: 对于java开发者来说jdk是再熟悉不过的了，在java圈还流行这样一句话：你发任你发，我用java8。jdk8是2013年发布的，是一个LTS版本，而在17年发布了jdk9之后，jdk的发布周期变化的更快了，很多开发者直呼学不动了。截止目前已经发布到jdk21了，关于jdk版本的发布可以参考wikipedia的[Java历史版本](https://zh.wikipedia.org/wiki/Java%E7%89%88%E6%9C%AC%E6%AD%B7%E5%8F%B2)来了解详细。我们今天要安装的是jdk17。
featured_image: /wp-content/uploads/2023/10/openjdk.jpg
categories:
  - Java
tags:
  - jdk
  - jdk17
---
![](https://www.iminling.com/wp-content/uploads/2023/10/openjdk.jpg)

对于java开发者来说jdk是再熟悉不过的了，在java圈还流行这样一句话：你发任你发，我用java8。jdk8是2013年发布的，是一个LTS版本，而在17年发布了jdk9之后，jdk的发布周期变化的更快了，很多开发者直呼学不动了。截止目前已经发布到jdk21了，关于jdk版本的发布可以参考wikipedia的[Java历史版本](https://zh.wikipedia.org/wiki/Java%E7%89%88%E6%9C%AC%E6%AD%B7%E5%8F%B2)来了解详细。我们今天要安装的是jdk17。

这么多jdk版本我们应该选择哪个来使用呢？我们先来看看jdk的版本收费情况：

  * JDK8以前版本，目前免费。
  * JDK8，免费版本至8u202，从8u211开始商用收费。
  * JDK9、JDK10 未收费。
  * JDK11，免费版本版本至11.0.2，从11.0.3开始商用收费。
  * JDK12、JDK13、JDK14、JDK15、JDK16，全版本商用收费。
  * JDK17开始，免费到2024 年 9 月，后续是否收费不明。

作为企业，很大一部分都是不想付费的，所以这也是为什么很多企业使用jdk8的8u202版本，并且不愿意升级的原因。而且jdk8是一个LTS版本，支持到2030年。而下一个LTS版本是jdk11，而jdk11又是收费的，很大一部分企业都不愿意使用，所以直到jdk17出来后，jdk又开始不收费了，而且jdk17是一个LTS版本，所以如果企业想要升级jdk，那么jdk17是一个很好的选择，oracel会支持jdk17到2029年。

为了体验jdk17以及为以后切换jdk17做准备，所以就需要我们来安装jdk17的开发环境。

mac可以选择安装Homebrew来快速安装jdk17，关于Homebrew的安装可以参考历史文章：[Mac更换源更快速的安装Homebrew](https://www.iminling.com/2023/10/02/265.html "Mac更换源更快速的安装Homebrew")，这里就不再多说了，下边我们使用Homebrew来安装jdk。

## 安装

先搜索jdk版本：

```
# ko @ MacMini in ~ [10:32:25]
$ brew search jdk
==> Formulae
openjdk                openjdk@17 ✔           jd                     cdk
openjdk@11             openjdk@8              mdk
```

可以看到有几个jdk的版本可以安装，我们需要的是17，接下来就是安装了：

```
# ko @ MacMini in ~ [11:28:24]
$ brew install openjdk@17
==> openjdk@17
For the system Java wrappers to find this JDK, symlink it with
  sudo ln -sfn /opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk

openjdk@17 is keg-only, which means it was not symlinked into /opt/homebrew,
because this is an alternate version of another formula.

If you need to have openjdk@17 first in your PATH, run:
  echo 'export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"' >> ~/.zshrc

For compilers to find openjdk@17 you may need to set:
  export CPPFLAGS="-I/opt/homebrew/opt/openjdk@17/include"
```

经过上边的命令后，jdk17就已经安装好了。

## 环境变量设置

根据安装最后的提示我们需要先新建一个软链接：

```
# konghang @ MacMini in ~ [10:37:47]
$ sudo ln -sfn /opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk
Password:
```

接下来是添加环境变量，我使用的是.base_profile，使用vim编辑后添加以下内容：

```
# JDK
#export JAVA_HOME="/Library/Java/JavaVirtualMachines/adoptopenjdk-8.jdk/Contents/Home"
export JAVA_HOME="/Library/Java/JavaVirtualMachines/openjdk-17.jdk/Contents/Home"
export PATH=${JAVA_HOME}/bin:$PATH
```

然后刷新变量，然后查看jdk版本看是否成功：

```
# ko @ MacMini in ~ [10:43:55]
$ source .bash_profile

# ko @ MacMini in ~ [10:43:59]
$ java -version
openjdk version "17.0.8.1" 2023-08-24
OpenJDK Runtime Environment Homebrew (build 17.0.8.1+0)
OpenJDK 64-Bit Server VM Homebrew (build 17.0.8.1+0, mixed mode, sharing)
```

看到当前的环境下使用的是jdk17了，截止到这里jdk17就已经安装成功了。接下来就可以使用jdk来进行开发和研究了。
