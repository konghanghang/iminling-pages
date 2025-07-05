---
title: docker搭建aliyunpan-sync时实增量同步阿里云盘文件
author: 要名俗气
type: post
date: 2024-07-06T03:11:29+00:00
url: /2024/docker-deploy-aliyunpan-sync
description: 在前边的文章中介绍了如何快速的搭建[小雅alist](https://www.iminling.com/2024/04/20/586.html "搭建属于自己的影视小站：小雅Alist安装及使用")来满足大家的观影需求，但是有一些剧小雅更新没那么及时，这时候就需要大家八仙过海了。如何及时找到自己要观看的剧，并成功观看呢？下边就带着大家一起使用[aliyunpan-sync](https://github.com/tickstep/aliyunpan)工具来同步阿里云盘的数据。
featured_image: /wp-content/uploads/2024/07/aliyunpan.jpg
categories:
  - 影音
tags:
  - aliyunpan
  - docker
---
在前边的文章中介绍了如何快速的搭建[小雅alist](https://www.iminling.com/2024/04/20/586.html "搭建属于自己的影视小站：小雅Alist安装及使用")来满足大家的观影需求，但是有一些剧小雅更新没那么及时，这时候就需要大家八仙过海了。如何及时找到自己要观看的剧，并成功观看呢？下边就带着大家一起使用[aliyunpan-sync](https://github.com/tickstep/aliyunpan)工具来同步阿里云盘的数据。

## 网盘选择

这里为什么要选择阿里云盘呢？因为阿里网盘资源比较多，更新也比较快，然后网上也找了同步阿里云盘的同步工具：[aliyunpan-sync](https://github.com/tickstep/aliyunpan)，理论上115，百度云，夸克这些网盘也都是ok的，主要是把文件同步到本地，具体的网盘的同步工具就需要大家自己去找了。我这里只带着大家实现阿里云盘的同步。另外推荐给大家一个电报的阿里云盘的资源通知群，资源更新还是挺及时的：[https://t.me/yunpanpan](https://t.me/yunpanpan "阿里云盘telegram群组")。

## 安装aliyunpan-sync

我这里使用的是docker compose进行的安装，如果还没有安装docker的同学可以参考的我的文章把docker 给安装一下：[docker安装](https://www.iminling.com/2023/06/10/164.html "docker安装")。

### 获取阿里云盘token

要想从阿里云盘里下载文件，肯定是需要先登录阿里云盘才能进行数据同步，如何登录阿里云盘aliyunpan-sync工具里也有具体的说明：[登录阿里云盘](https://github.com/tickstep/aliyunpan/blob/main/docs/manual.md#%E7%99%BB%E5%BD%95%E9%98%BF%E9%87%8C%E4%BA%91%E7%9B%98%E5%B8%90%E5%8F%B7)，需要先在[release](https://github.com/tickstep/aliyunpan/releases)页面下载一个对应系统的release，我的操作系统是mac m1芯片我就下载arm那个

![aliyunpan sync release](https://www.iminling.com/wp-content/uploads/2024/07/FAEE60916F633EC24FFD16E75F5AAB73.png)

然后对文件进行解压，使用终端进入到对应的目录里，看到有一个`aliyunpan`的文件，执行这个文件获取登录链接，然后在网页端进行扫码登录就可以了。

```
# k @ Mac-mini in ~/Downloads/aliyunpan-v0.3.2-darwin-macos-arm64 [10:03:26]
$ ls
README.md             manual.md             sync.sh               webdav.sh
aliyunpan             plugin                sync_drive
aliyunpan_config.json sync.bat              webdav.bat

# k @ Mac-mini in ~/Downloads/aliyunpan-v0.3.2-darwin-macos-arm64 [10:03:28]
$ aliyunpan login
zsh: command not found: aliyunpan

# k @ Mac-mini in ~/Downloads/aliyunpan-v0.3.2-darwin-macos-arm64 [10:03:42] C:127
$ ./aliyunpan login
请在浏览器打开以下链接进行登录，链接有效时间为5分钟。
注意：你需要进行一次授权一次扫码的两次登录。
https://openapi.alipan.com/oauth/authorize?client_id=cf9f70e8fc6143031375&redirect_uri=https%3A%2F%2Fapi.tickstep.com%2Fauth%2Ftickstep%2Faliyunpan%2Ftoken%2Fopenapi%2F0bd327267c41f7b2fcadf927%2Fauth2&scope=user:base,file:all:read,file:all:write

请在浏览器里面完成扫码登录，然后再按Enter键继续...
```

打开url就需要使用手机端的阿里云盘进行扫码登录，提示进行授权：

![aliyunpan authority](https://www.iminling.com/wp-content/uploads/2024/07/2F4D63F1408D9F02A15CC4F6E7533A1B.png)

根据需要选择授权范围，我这里把`备份盘`和`资源库`都进行授权，其实就是整个阿里云盘这个工具都可以访问。然后就是回到终端看到具体的token。保存好这个token.

### docker镜像启动

我这里使用的是docker compose来进行安装的，我的文件目录如下：

```
$ tree
.
├── config
├── docker-compose.yaml
├── js
│   ├── download_handler.js.sample
│   ├── sync_handler.js
│   ├── sync_handler.js.sample
│   ├── token_handler.js
│   ├── token_handler.js.sample
│   └── upload_handler.js.sample
└── sync_drive
```

config目录是用来挂在配置文件的，等下token是要填在这个目录下的`aliyunpan_config.json`文件里的。

js目录是用来做一些脚本处理，用来控制什么文件可以上传，什么文件可以下载，以及进行一些通知操作的。

sync_drive是保存数据库的信息，用来记录同步的一些信息，即使容器销毁，同步数据库还可以用于以后使用。

具体的docker-compose.yaml文件内容如下：

```
version: '3'
services:
  sync:
    image: tickstep/aliyunpan-sync:v0.3.2
    container_name: aliyunpan-sync
    restart: unless-stopped
    volumes:
      # （必须）映射的本地目录
      - /data/webdav/data:/home/app/data:rw
      # （可选）可以指定JS插件sync_handler.js用于过滤文件，详见插件说明
      #- ./config/sync_handler.js:/home/app/config/plugin/js/sync_handler.js
      - ./js:/home/app/config/plugin/js/
      # （推荐）挂载sync_drive同步数据库到本地，这样即使容器销毁，同步数据库还可以用于以后使用
      - ./sync_drive:/home/app/config/sync_drive
      # （必须）映射token凭据文件
      - ./config/aliyunpan_config.json:/home/app/config/aliyunpan_config.json
    extra_hosts:
      - host.docker.internal:host-gateway
    environment:
      # 时区，东8区
      - TZ=Asia/Shanghai
      # 下载文件并发数
      - ALIYUNPAN_DOWNLOAD_PARALLEL=2
      # 上传文件并发数
      - ALIYUNPAN_UPLOAD_PARALLEL=2
      # 下载数据块大小，单位为KB，默认为10240KB，建议范围1024KB~10240KB
      - ALIYUNPAN_DOWNLOAD_BLOCK_SIZE=1024
      # 上传数据块大小，单位为KB，默认为10240KB，建议范围1024KB~10240KB
      - ALIYUNPAN_UPLOAD_BLOCK_SIZE=10240
      # 指定网盘文件夹作为备份目标目录，不要指定根目录
      - ALIYUNPAN_PAN_DIR=/来自分享
      # 备份模式：upload(备份本地文件到云盘), download(备份云盘文件到本地)
      - ALIYUNPAN_SYNC_MODE=download
      # 备份策略: exclusive(排他备份文件，目标目录多余的文件会被删除),increment(增量备份文件，目标目录多余的文件不会被删除)
      #- ALIYUNPAN_SYNC_POLICY=increment
      # 备份周期, 支持两种: infinity(永久循环备份),onetime(只运行一次备份)
      #- ALIYUNPAN_SYNC_CYCLE=infinity
      # 网盘：backup(备份盘), resource(资源盘)
      - ALIYUNPAN_SYNC_DRIVE=resource
      # 是否显示文件备份过程日志，true-显示，false-不显示
      - ALIYUNPAN_SYNC_LOG=true
      # 本地文件修改检测延迟间隔，单位秒。如果本地文件会被频繁修改，例如录制视频文件，配置好该时间可以避免上传未录制好的文件
      - ALIYUNPAN_LOCAL_DELAY_TIME=60
```

重点说一下上边的四个配置：

第一个是挂载配置下的`映射的本地目录`，这个是把阿里云盘的数据下载下来的目录，需要把他挂载在一个比较大的存储路径。

第二个是备份模式：`ALIYUNPAN_SYNC_MOD`，我这里是把云盘文件下载到本地，所以模式是`download`。

第三个是网盘：`ALIYUNPAN_SYNC_DRIVE`，阿里云盘分为备份盘和资源盘，我这里大部分转存的文件都是放在资源盘，所以这里配置`resource`。

第四个是备份目标目录：`ALIYUNPAN_PAN_DIR`，这个配置就是当前容器需要下载的目录，上边指定了需要下载资源盘的文件，但是肯定不能整个资源盘都下载，只需要指定特定的目录下载就行了，比如我转存的文件都是放在`来自分享`目录下，我这里就配置来自分享。

![aliyunpan share](https://www.iminling.com/wp-content/uploads/2024/07/9BB509B435FAFEFE2A9CE751D2D1E313.jpg)

js的通知后边在讲，这个目录里边暂时可以先不放任何js文件。

然后就可以启动docker compose文件了。

```
docker compose up -d
```

此时虽然启动了容器，但是因为没有配置token，容器会一直重启，但是此时目录config目录下就会多出一个aliyun_config.json的文件，编辑文件找到`accessToken`属性，把第一步生成好的token替换进去就可以了。保存文件后等待容器下一次重启或者手动重启容器就可以了。此时就可以了。

## 验证

接下来就是找一个文件保存到云盘的指定要下载的目录，我是`来自分享`，然后去看一下目录有没有进行下载。也可以通过查看容器的日志：

<pre class="corepress-code-pre">root@docker:~/images/aliyunpancli# docker compose logs -tf --tail 10
WARN[0000] /root/images/aliyunpancli/docker-compose.yaml: `version` is obsolete
aliyunpan-sync | 2024-07-06T02:52:51.522742265Z 扫描到云盘文件：/来自分享/Cover the sky.S01E61.2023.2160p.WEB.DL.H265.DDP2.0-BestWEB.mkv
aliyunpan-sync | 2024-07-06T02:52:56.282356345Z 完成全部文件的同步下载，等待下一次扫描
aliyunpan-sync | 2024-07-06T02:53:51.550419129Z 开始进行文件扫描...
aliyunpan-sync | 2024-07-06T02:53:51.756156867Z 扫描到云盘文件：/来自分享/Jade Dynasty.S02E43.2022.2160p.WEB-DL.H265.DDP2.0-BestWEB.mkv
aliyunpan-sync | 2024-07-06T02:53:51.756982180Z 扫描到云盘文件：/来自分享/Cover the sky.S01E61.2023.2160p.WEB.DL.H265.DDP2.0-BestWEB.mkv
aliyunpan-sync | 2024-07-06T02:53:56.554962070Z 下载文件：/来自分享/Jade Dynasty.S02E43.2022.2160p.WEB-DL.H265.DDP2.0-BestWEB.mkv
下载到本地:/home/app/data/Jade Dynasty.S02E43.2022.2160p.WEB-DL.H265.DDP2.0-BestWEB.mkv ↓ 773.00MB/779.46MB(99.17%) 9.64MB/s............下载完毕：/home/app/data/Jade Dynasty.S02E43.2022.2160p.WEB-DL.H265.DDP2.0-BestWEB.mkv</pre>

通过日志可以看出它在不停的扫描，检测到新文件就开始进行下载。

## JS通知

在docker compose文件中，指定了js文件的目录，那么要怎么使用呢？官方也有一些具体的例子：[JavaScript插件](https://github.com/tickstep/aliyunpan/blob/main/docs/manual.md#JavaScript%E6%8F%92%E4%BB%B6)，我这里需要提供一个下载完成通知的功能，保存文件后什么时候下载完成是大家比较关心的，总不能总是查看日志。

这里要使用到的js文件是`sync_handler.js`文件。文件具体的内容如下：

```
// ------------------------------------------------------------------------------------------
// 函数说明：同步备份-同步文件后的回调函数
//
// 参数说明
// context - 当前调用的上下文信息
// {
// 	"appName": "aliyunpan",
// 	"version": "v0.1.3",
// 	"userId": "11001d48564f43b3bc5662874f04bb11",
// 	"nickname": "tickstep",
// 	"fileDriveId": "19519111",
// 	"albumDriveId": "29519122"
// }
// appName - 应用名称，当前固定为aliyunpan
// version - 版本号
// userId - 当前登录用户的ID
// nickname - 用户昵称
// fileDriveId - 用户文件网盘ID
// albumDriveId - 用户相册网盘ID
//
// params - 同步文件后的参数
// {
// 	"action": "upload",
// 	"actionResult": "success",
// 	"driveId": "19519221",
// 	"fileName": "1.txt",
// 	"filePath": "D:/goprojects/dev/upload/1.txt",
// 	"fileSha1": "294C8F24C56042710813E95C55A0B018299BA9A7",
// 	"fileSize": "28077",
// 	"fileType": "file",
// 	"fileUpdatedAt": "2022-03-04 15:19:47"
// }
// action - 同步动作，
//          download-下载文件，
//          upload-上传文件，
//          delete_local-删除本地文件，
//          delete_pan-删除云盘文件，
//          create_local_folder-创建本地文件夹，
//          create_pan_folder-创建云盘文件夹
// actionResult - 同步结果，success-成功，fail-失败
// driveId - 网盘ID
// fileName - 文件名称
// filePath - 文件完整路径
// fileSha1 - 文件SHA1
// fileSize - 文件大小，单位B
// fileType - 文件类型，file-文件，folder-文件夹
// fileUpdatedAt - 文件修改时间
//
// 返回值说明
// （无）
// ------------------------------------------------------------------------------------------
function syncFileFinishCallback(context, params) {
	console.log(params)
    var header = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.88 Safari/537.36",
        "Content-Type": "application/json",
        "Accept": "application/json"
    };
    try {
        var text = "文件名：" + params["fileName"] + " 同步完成, 同步结果: " + params["actionResult"];
        if (params["actionResult"] === "fail") {
            var reqData = {
                "chat_id": "这里是自己的id",
                "text": text
            };
            var r = PluginUtil.Http.post(header, "https://api.telegram.org/{这里替换具体的电报机器人token}/sendMessage", JSON.stringify(reqData));
        }
    } catch (e) {
        if (e !== "Error") {
            throw e;
        }
    }
}
```

怎么创建电报机器人以及获取自己的id这里就不再说了，大家可以网上搜索一下，如果后续时间充裕，我也会详细教大家如何创建机器人和查看自己的id。

text就是通知的内容，来看一下最终的通知效果：

![aliyunpan sync notification](https://www.iminling.com/wp-content/uploads/2024/07/05B5E84F322C5B0AB5CFF6A562CEADFC.jpg)
