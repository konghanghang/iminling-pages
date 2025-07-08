---
title: 使用fiddler抓包旧版本iTunes并下载ios历史版本软件
author: 要名俗气
type: post
date: 2024-02-20T15:17:33+00:00
url: /2024/fiddler-captures-the-old-version-of-itunes
description: 手上的iphone se2的版本是15.5，在商店下载onedrive的时候提示当前版本只支持ios16以及更新的ios版本，我又不想升级手上这个手机的版本，于是就想着通过fiddler抓包来获取ios15支持的版本来进行安装。本文带你使用fiddler抓包旧版本iTunes并下载ios历史版本软件。
image: https://images.iminling.com/app/hide.php?key=U0c5REZ3L1p2bU1COUVtaVVxeWhJUndqNHNUYXYwQ0E5aGM1Zld1dEhyMUt1c0l5eGpxQjhUOGJOOS84RXhaN0FIcXo3SWs9
categories:
  - mac
tags:
  - AppStore
  - fiddler
  - iTunes
  - OneDrive
  - 土耳其
---
最近上车的office365到期了，搜索后发现在苹果土耳其商店使用onedrive可以以23.99里拉每个月的价格来开通office365，算下来一个月6块都不到(土耳其钱包FUPS注册可以参考另一篇文章：[土耳其虚拟银行FUPS注册教程]({{< ref "/post/card/土耳其虚拟银行FUPS注册教程.md" >}}))，非常划算，所以就开始折腾。手上的iphone se2的版本是15.5，在商店下载onedrive的时候提示当前版本只支持ios16以及更新的ios版本，我又不想升级手上这个手机的版本，于是就想着通过fiddler抓包来获取ios15支持的版本来进行安装。

## 旧版本iTunes安装

我这里下载的是12.6.5.3的版本，iTunes的历史版本可以去到wiki百科进行下载：[iTunes](https://www.theiphonewiki.com/wiki/ITunes)。这里边包含所有的版本的下载链接，如果打不开wiki百科，可以使用下边这个url进行下载，也是直链苹果官网：[iTunes 12.6.5.3](https://secure-appldnld.apple.com/itunes12/091-87819-20180912-69177170-B085-11E8-B6AB-C1D03409AD2A6/iTunes64Setup.exe)。下载正常安装，成功后登录自己的土耳其apple id就可以了。

## fiddler安装配置

fiddler直接去官网下载就可以，官网地址：[fiddler](https://www.telerik.com/fiddler/fiddler-classic)。安装成功后首先要开启解密https流量，位置：`Tools -> Options -> decrypt https traffic`，勾选后就会让添加证书到当前电脑，根据弹窗提示进行安装。

![fiddler Https](https://images.iminling.com/app/hide.php?key=Ni9teHZsaWUwOGliZUkyc05OdTFxTHJxMitiM1ltN3UxcmZSUnFEVncvZDhUR1ZSYkVZNVo0djZKcGp0Rm44LzBLNGlZb1k9)

添加成功后，把`ignore server certificate errors(unsafe)`也勾选一下。到这里fiddler就设置好了。

## 获取oneDrive版本

直接在iTunes中进行搜索，找到oneDrive：

![onedrive appstore search](https://images.iminling.com/app/hide.php?key=S3cwNHpESWhZUndqVGtCL04xUFAwcXZHbzR0eFJuZ3JRQnhXQk41YlRhNHVHS2JXR0dXckN6Q21Gczk4K3kwVm5zU2MvUU09)

点击查看详情，可以看到一个`show all versions`的链接，点击查看所有的版本说明：

![onedrive all version](https://images.iminling.com/app/hide.php?key=VnhkSm81T0cxajAydnFhZnRrbFRFbzF0d2VPWmNXeHdsdEtURi9abDJrOXhzQ2hXVUtLTzJsV293akFnRG1udHh4bE1yYnc9)

版本说明如下，可以看到官方的说明，14.21.7是最后一个支持ios15的版本，后续的版本只支持16和17.这个版本就是我需要的版本了。

![onedrive version desc](https://images.iminling.com/app/hide.php?key=T24wall1TmY4Q3p2dk5UckVXKzFXNXYzZ0VEMU93MWp6UFA3NVJTY2tXYkh6a21WSzV0NGV5OEYySUhueS9BbHRrWjVWRWs9)

现在只是知道了版本是哪个，在下载历史版本的时候需要获取一个9位数的id才能正确的下载到对应的版本。所以需要先获取历史版本的id。

### 获取历史版本id

此时需要使用到fiddler来进行抓包，如果已经启动了fiddler可以先清理一下已经抓取的历史记录,如下图，`remove all`或者使用快捷键：`ctrl+x`清除：

![fiddler remove history](https://images.iminling.com/app/hide.php?key=ejE5UmtyOWZkSlgwMlJDSGU4dENjUmZHYlJGbUJPU2NNaTJJdUlBaitLMlA1NzgzVGovVjcvVlBlVVdld0hyVTNld0orZjg9)

清除后在iTunes里进行一次onedrive的下载，此时在fiddler里会有一个`pxx-buy`开头的url，这个url就是我们需要找的url，在这里边可以获取到onedrive所有的历史版本id，先双击这个url，在右侧就会有信息展示出来，如下：

![fiddler view decrypt](https://images.iminling.com/app/hide.php?key=S0E0b2ZQVnJLWWFPL0lQRmZxd05LVnJmMUhiV3h0Z3hNV3NmbjhGS2x3N2xvNDJHYUQxSit5TEJhMktjSmlIcU1hUDVZU2M9)

上图我框起来的有一处是说响应体被加密，需要点击来解密(Response body is encoded. Click to decode)，鼠标点击一下，在textView下就可以找到对应的id了,这个id从上到下一次递增，越新的版本越靠后：

![onedrive id](https://images.iminling.com/app/hide.php?key=UEd3VThBMDRFdVR2QXdORWZWZ0Q4UkpJRnVjY0ZUTW5CamxEOHg4a1pFNE5nemN0bmdDTlA5aTFKaFZFQUdLNlA3UVdrYTA9)

通过上边查看onedrive的版本说明看出是倒数第7个版本，从这个版本列表里倒序去找第7个再下载一下看看是不是需要的版本，我在尝试后发现并不是倒数第7个id，具体的规则也不清楚，但是大概位置不会错，下载后看版本：如果大了就继续倒序往上找，如果小了就正序再往下找。最终我是找到了id：861576168，下边说一下怎么去下载历史版本。

### 下载历史版本

通过上一步拿到了很多的版本id，倒序从后边先随便找一个id来下载一下看看版本具体是什么。因为在获取版本的时候已经下载过了一次，所以需要在iTunes中把原来的给删除了：

![itunes app](https://images.iminling.com/app/hide.php?key=VlR1QmhlN1FvUkd2YWpZNGE5YVc2QXlPcjA3OFFDLzNuWGp1S21XMlNYYVVyUTlzQjZPenNxZWdCeVlTZFdUTDg1b2Q1RE09)

通过`文件->资料库->打开Genius`来打开已下载的程序,如下选择应用，然后把已经下载的给删除了。

![itunes app](https://images.iminling.com/app/hide.php?key=TWhRSDhwVlhXampacG5jMUpmZG1HalY4SmlDQ0tjRVRRdXcrSE5ETzh1Z0dPenlTZnpsS1lId1JKbDA2bkllZGpDS1lVMVk9)

删除成功了后，需要在fiddler程序里输出一下命令: `bpu MZBuy.woa`。，然后fiddler就可以拦截请求，给咱们机会去修改请求的id，从而达到下载历史版本的需求，输入完需要敲回车，使之生效：

![fiddler bpu](https://images.iminling.com/app/hide.php?key=L1ZsaG5KS0JGamJ6QngvTXRId0FEbjhoN0liQ0l3RWpSVTVoUDkwdmFTYndLUU4xZU1SY0xTM3c5WTVNSzd5WnhodHBEeTA9)

然后就是继续清除fiddler里已经抓取的url方便下边去找url，清除后再去itunes里点击一次下载：

![fiddler modify](https://images.iminling.com/app/hide.php?key=MnhCUGJXU1VoK3ZobWh4a25FdlhYT3JsNkpJYTAvMEk1TVJvUFVVOTh1KzdEN054UlAzdmlkSnkzUWxNaFd3czVkSm1JK2s9)

点击下载后，在fiddler里可以看到图标是一个T字的url，它此时已经被拦截了，需要等待我们手动的处理是中断(Break on Response)还是继续运行完成(Run to Completion),此时需要我们在TextView下去修改`string`包裹的那一串id，替换成需要的历史id，替换完成后，点击`Run to Completion`就会按照我们替换后的id来继续下载了。下载完成后在文件夹里查看当前下载的文件，就会显示是什么版本：

![itunes app](https://images.iminling.com/app/hide.php?key=aXJqeW9LcUVTSmRtV2N0TXhRc05EVHFkNzltZnNwN3dUd25ibXRHQ09RVEhoZDkzVXpqQ3dNczNYQmQyMmhOd1FoM3JWaDg9)

![onedriv](https://images.iminling.com/app/hide.php?key=b1RZQ08wVXRyaHZ6RWNrVmZJTU53ZzFXbHhPT3ZJa2hUc25qZVI2UCtUb2xuOVBvVUdhV3lnaGtFWUFIRzhFM1dJSU9uK2c9)

如上下载到了最终的14.21.7版本。

## 安装

下载了之后就需要安装到手机上，这里也非常简单，电脑下载爱思助手，选择手动安装软件，选择上边下载好的ipa文件就可以了。

![i4](https://images.iminling.com/app/hide.php?key=MzkrWk9hZHlmN2tseFZpek11am1oMUoyNnVrK1hQZkZ3UDBJSHBSMGMrdGVnTStEUUoybExRNUJWR1pmaWVUdmpVUFRCL1U9)

记录如上。愉快的使用oneDrive来购买office365
