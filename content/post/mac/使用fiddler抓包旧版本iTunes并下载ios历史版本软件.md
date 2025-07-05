---
title: 使用fiddler抓包旧版本iTunes并下载ios历史版本软件
author: 要名俗气
type: post
date: 2024-02-20T15:17:33+00:00
url: /2024/fiddler-captures-the-old-version-of-itunes
description: 最近上车的office365到期了，搜索后发现在苹果土耳其商店使用onedrive可以以23.99里拉每个月的价格来开通office365，算下来一个月6块都不到(土耳其钱包FUPS注册可以参考另一篇文章：[土耳其虚拟银行FUPS注册教程](https://www.iminling.com/2024/01/29/396.html "土耳其虚拟银行FUPS注册教程"))，非常划算，所以就开始折腾。
featured_image: /wp-content/uploads/2024/02/App-Store.png
categories:
  - 软件
tags:
  - AppStore
  - fiddler
  - iTunes
  - OneDrive
  - 土耳其
---
最近上车的office365到期了，搜索后发现在苹果土耳其商店使用onedrive可以以23.99里拉每个月的价格来开通office365，算下来一个月6块都不到(土耳其钱包FUPS注册可以参考另一篇文章：[土耳其虚拟银行FUPS注册教程](https://www.iminling.com/2024/01/29/396.html "土耳其虚拟银行FUPS注册教程"))，非常划算，所以就开始折腾。手上的iphone se2的版本是15.5，在商店下载onedrive的时候提示当前版本只支持ios16以及更新的ios版本，我又不想升级手上这个手机的版本，于是就想着通过fiddler抓包来获取ios15支持的版本来进行安装。

## 旧版本iTunes安装

我这里下载的是12.6.5.3的版本，iTunes的历史版本可以去到wiki百科进行下载：[iTunes](https://www.theiphonewiki.com/wiki/ITunes)。这里边包含所有的版本的下载链接，如果打不开wiki百科，可以使用下边这个url进行下载，也是直链苹果官网：[iTunes 12.6.5.3](https://secure-appldnld.apple.com/itunes12/091-87819-20180912-69177170-B085-11E8-B6AB-C1D03409AD2A6/iTunes64Setup.exe)。下载正常安装，成功后登录自己的土耳其apple id就可以了。

## fiddler安装配置

fiddler直接去官网下载就可以，官网地址：[fiddler](https://www.telerik.com/fiddler/fiddler-classic)。安装成功后首先要开启解密https流量，位置：`Tools -> Options -> decrypt https traffic`，勾选后就会让添加证书到当前电脑，根据弹窗提示进行安装。

![fiddler Https](https://www.iminling.com/wp-content/uploads/2024/02/2A517C87D97945CF00A9B3980D5E71C3.png)

添加成功后，把`ignore server certificate errors(unsafe)`也勾选一下。到这里fiddler就设置好了。

## 获取oneDrive版本

直接在iTunes中进行搜索，找到oneDrive：

![onedrive appstore search](https://www.iminling.com/wp-content/uploads/2024/02/5CA29841710A3A72EA6FEC84C5D226C7.png)

点击查看详情，可以看到一个`show all versions`的链接，点击查看所有的版本说明：![onedrive all version](https://www.iminling.com/wp-content/uploads/2024/02/82A89F2D84A0F60B489B1DF60C3AADC5.png)

版本说明如下，可以看到官方的说明，14.21.7是最后一个支持ios15的版本，后续的版本只支持16和17.这个版本就是我需要的版本了。

![onedrive version desc](https://www.iminling.com/wp-content/uploads/2024/02/FEE478DAD829075554E45C087321FEDD.png)

现在只是知道了版本是哪个，在下载历史版本的时候需要获取一个9位数的id才能正确的下载到对应的版本。所以需要先获取历史版本的id。

### 获取历史版本id

此时需要使用到fiddler来进行抓包，如果已经启动了fiddler可以先清理一下已经抓取的历史记录,如下图，`remove all`或者使用快捷键：`ctrl+x`清除：

![fiddler remove history](https://www.iminling.com/wp-content/uploads/2024/02/51273F6202119CF0ED44B47DC5D63BC7.png)

清除后在iTunes里进行一次onedrive的下载，此时在fiddler里会有一个`pxx-buy`开头的url，这个url就是我们需要找的url，在这里边可以获取到onedrive所有的历史版本id，先双击这个url，在右侧就会有信息展示出来，如下：

![fiddler view decrypt](https://www.iminling.com/wp-content/uploads/2024/02/5C4BDB0CFB16E828F266C7F26D00E2BE.png)

上图我框起来的有一处是说响应体被加密，需要点击来解密(Response body is encoded. Click to decode)，鼠标点击一下，在textView下就可以找到对应的id了,这个id从上到下一次递增，越新的版本越靠后：

![onedrive id](https://www.iminling.com/wp-content/uploads/2024/02/BE87412FE81D2BE1B3F68EFF196D488C.png)

通过上边查看onedrive的版本说明看出是倒数第7个版本，从这个版本列表里倒序去找第7个再下载一下看看是不是需要的版本，我在尝试后发现并不是倒数第7个id，具体的规则也不清楚，但是大概位置不会错，下载后看版本：如果大了就继续倒序往上找，如果小了就正序再往下找。最终我是找到了id：861576168，下边说一下怎么去下载历史版本。

### 下载历史版本

通过上一步拿到了很多的版本id，倒序从后边先随便找一个id来下载一下看看版本具体是什么。因为在获取版本的时候已经下载过了一次，所以需要在iTunes中把原来的给删除了：

![itunes app](https://www.iminling.com/wp-content/uploads/2024/02/1FDC2097EB8FD8D1D31340FFFA78564D.png)

通过`文件->资料库->打开Genius`来打开已下载的程序,如下选择应用，然后把已经下载的给删除了。

![itunes app](https://www.iminling.com/wp-content/uploads/2024/02/712DF8E1A1185AE7EE7A2F7B7B6CEB41.png)

删除成功了后，需要在fiddler程序里输出一下命令: `bpu MZBuy.woa`。，然后fiddler就可以拦截请求，给咱们机会去修改请求的id，从而达到下载历史版本的需求，输入完需要敲回车，使之生效：

![fiddler bpu](https://www.iminling.com/wp-content/uploads/2024/02/B6D2A12BBF914CB1C5D8B9B49D558C98.png)

然后就是继续清除fiddler里已经抓取的url方便下边去找url，清除后再去itunes里点击一次下载：

![fiddler modify](https://www.iminling.com/wp-content/uploads/2024/02/94A5CF05F96BCD97117392A1F112966A.png)

点击下载后，在fiddler里可以看到图标是一个T字的url，它此时已经被拦截了，需要等待我们手动的处理是中断(Break on Response)还是继续运行完成(Run to Completion),此时需要我们在TextView下去修改`string`包裹的那一串id，替换成需要的历史id，替换完成后，点击`Run to Completion`就会按照我们替换后的id来继续下载了。下载完成后在文件夹里查看当前下载的文件，就会显示是什么版本：

![itunes app](https://www.iminling.com/wp-content/uploads/2024/02/712DF8E1A1185AE7EE7A2F7B7B6CEB41.png)

![onedriv](https://www.iminling.com/wp-content/uploads/2024/02/55F99B33CB9F199363E8DB191930B8EF.png)

如上下载到了最终的14.21.7版本。

## 安装

下载了之后就需要安装到手机上，这里也非常简单，电脑下载爱思助手，选择手动安装软件，选择上边下载好的ipa文件就可以了。

![i4](https://www.iminling.com/wp-content/uploads/2024/02/E8D97B6D45CD3EAE4776EA42EE05E1F0.png)

记录如上。愉快的使用oneDrive来购买office365
