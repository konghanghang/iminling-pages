---
title: 升级Intellij IDEA 2023.3后找回Ant构建工具
author: 要名俗气
type: post
date: 2023-12-16T03:00:20+00:00
url: /2023/intellij-idea2023-find-ant-tool
description: 公司有一个项目使用的是Ant来进行打包构建的，没升级IDEA前还可以在工具栏找到Ant功能来进行构建，升级2023.3版本后，Ant就找不到了，经过一番折腾，终于找回了Ant。再次提醒各位，升级还是需要谨慎呀，旧的又不是不能用，哈哈。废话不多说，下边给出具体的解决方法。
image: https://images.iminling.com/app/hide.php?key=MklMSXFFczNLSWQ5Zkl4N1RrVGo4NkxlOXJXRTZ6L1B2cndwZEJ2ai9OMnRkV21kdVdzcTZsek1qK1ZKMFJINHVWenoyNjg9
categories:
  - IDEA
tags:
  - Ant
  - IDEA
---
公司有一个项目使用的是Ant来进行打包构建的，没升级IDEA前还可以在工具栏找到Ant功能来进行构建，升级2023.3版本后，Ant就找不到了，经过一番折腾，终于找回了Ant。再次提醒各位，升级还是需要谨慎呀，旧的又不是不能用，哈哈。废话不多说，下边给出具体的解决方法。

## 2023.3以前版本

在2023.3版本以前，如果在工具栏找不到Ant，可以在IDEA的菜单中把Ant给添加到工具栏，如下图：

![view菜单](https://images.iminling.com/app/hide.php?key=dzZLUEtFY05OQ1Bpc21TMlhaOG5RVEZ1d3VuS1JydGJQMmkrK2xzSXdGTDh4Ti90ODUrODRZMDlKTVg5VnUzOHFMU0tzSjQ9)
view菜单 tool windows - Ant就可以找到Ant工具。

## 2023.3版本

升级到2023.3版本后，在view菜单 tool windows中找不到Ant选项了，但是多了一个 AI Assistant 的功能，在IDEA中找Ant相关的配置也是无果。

![view无ant](https://images.iminling.com/app/hide.php?key=K2hxVHlXVmEvOFVNSnQydSt6d1dRS3gwNG16Z0JHeDdtZ3pWVHJZRW16OTlpYVlvMWdNcjJ1RkpkaEs5c1RsWHhpejBqcVU9)

最后在IDEA的官方日志中找到了原因，官方地址：[IntelliJ IDEA 2023.3 最新变化](https://www.jetbrains.com/zh-cn/idea/whatsnew/)。其实上图里也有说明，只是升级后并没有仔细阅读，而是后来网上找到了中文的升级说明：

![2023.3中文升级说明](https://images.iminling.com/app/hide.php?key=L255bWZmaFZCL1pDSnlDWXR4c1VDeXdySWdDZDk0dXlLcnhTcWFFeGM4aEJ1V3JhL1NOTEpwNElZQk8rWUtlRzVJOVJRWE09)

在新版本中Ant要去到市场下载了。

## 下载Ant插件

在plugin中搜索Ant，就可以看到一个jetbrains提供的Ant插件，进行下载，然后就可以在view - tool windows中找到Ant插件了。

![jetbrain Ant插件](https://images.iminling.com/app/hide.php?key=THpqL1FOb1dVbEVPRm9VK3JYTnhlUTc4eDIvU010QnB3YXRtN2JzNTRnTXdPL0YvTFNaV3ZOOGZ4RkxSTmFMSURYdHppMHc9)

安装后立马就可以找到，也不需要重启IDEA，经过上述过程就成功的找回了Ant插件，可以愉快的使用Ant来编译打包项目了。

IDEA如何使用开源项目申请可以参考我的另一篇文章：[开源项目申请JetBrains Open Source Development License]({{< ref "/post/java/开源项目申请JetBrains Open Source Development License.md" >}})。
