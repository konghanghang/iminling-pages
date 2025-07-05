---
title: 升级Intellij IDEA 2023.3后找回Ant构建工具
author: 要名俗气
type: post
date: 2023-12-16T03:00:20+00:00
url: /2023/intellij-idea2023-find-ant-tool
description: 公司有一个项目使用的是Ant来进行打包构建的，没升级IDEA前还可以在工具栏找到Ant功能来进行构建，升级2023.3版本后，Ant就找不到了，经过一番折腾，终于找回了Ant。再次提醒各位，升级还是需要谨慎呀，旧的又不是不能用，哈哈。废话不多说，下边给出具体的解决方法。
featured_image: /wp-content/uploads/2023/12/F846E413CDDABDC244D0EE9F3917B2EE.png
categories:
  - IDEA
tags:
  - Ant
  - IDEA
---
公司有一个项目使用的是Ant来进行打包构建的，没升级IDEA前还可以在工具栏找到Ant功能来进行构建，升级2023.3版本后，Ant就找不到了，经过一番折腾，终于找回了Ant。再次提醒各位，升级还是需要谨慎呀，旧的又不是不能用，哈哈。废话不多说，下边给出具体的解决方法。

## 2023.3以前版本

在2023.3版本以前，如果在工具栏找不到Ant，可以在IDEA的菜单中把Ant给添加到工具栏，如下图：

<div id="attachment_342" style="width: 2004px" class="wp-caption alignnone">
  ![view菜单](https://www.iminling.com/wp-content/uploads/2023/12/B733553391FF97E5D326BEF1CD432110.png)

  <p id="caption-attachment-342" class="wp-caption-text">
    view菜单 tool windows - Ant就可以找到Ant工具。
  </p>
</div>

## 2023.3版本

升级到2023.3版本后，在view菜单 tool windows中找不到Ant选项了，但是多了一个 AI Assistant 的功能，在IDEA中找Ant相关的配置也是无果。

![view无ant](https://www.iminling.com/wp-content/uploads/2023/12/335EE8162A692A9022CCA6EDE5592C93.png)

最后在IDEA的官方日志中找到了原因，官方地址：[IntelliJ IDEA 2023.3 最新变化](https://www.jetbrains.com/zh-cn/idea/whatsnew/)。其实上图里也有说明，只是升级后并没有仔细阅读，而是后来网上找到了中文的升级说明：

![2023.3中文升级说明](https://www.iminling.com/wp-content/uploads/2023/12/0F21141A2DDF75BE050D5BC1DF27808C.png)

在新版本中Ant要去到市场下载了。

## 下载Ant插件

在plugin中搜索Ant，就可以看到一个jetbrains提供的Ant插件，进行下载，然后就可以在view - tool windows中找到Ant插件了。

![jetbrain Ant插件](https://www.iminling.com/wp-content/uploads/2023/12/9CEF649D6FD9F7D00EE85F158AC5EB59.png)

安装后立马就可以找到，也不需要重启IDEA，经过上述过程就成功的找回了Ant插件，可以愉快的使用Ant来编译打包项目了。

IDEA如何使用开源项目申请可以参考我的另一篇文章：[开源项目申请JetBrains Open Source Development License](https://www.iminling.com/2023/05/20/150.html "开源项目申请JetBrains Open Source Development License")。
