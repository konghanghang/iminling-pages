---
title: Proxmox VE(PVE)8.0安装黑群晖NAS并直通硬盘
author: 要名俗气
type: post
date: 2024-09-16T10:00:29+00:00
url: /2024/pve-install-nas
description: 前段时间通过pve安装ikuai,openwrt]以及通过lxc安装了docker,之后就一直稳定使用，J4125上有一个2T的硬盘，于是就想着折腾一下在上边安装一下黑群晖，体验一下nas系统。于是就又开始了折腾之路，又学习了很多知识。下边就记录一下我折腾的过程。
image: https://images.iminling.com/app/hide.php?key=NFRXQWhja0FLK28xVDVKMjg3V1dLL0xOYi9sYmloRVIra2YveWJQNVNTY1NVempCcDZMUjNrKzI1NkZmL3o0WlJJTzI2TGc9
categories:
  - pve
tags:
  - nas
  - pve
  - Synology
  - 黑群晖
---
前段时间通过pve安装[ikuai]({{< ref "/post/linux/pve/Proxmox VE(PVE)8.0安装爱快ikuai虚拟机并直通网卡.md" >}}),[openwrt]({{< ref "/post/linux/pve/Proxmox VE(PVE)8.0安装Openwrt实现旁路由模式.md" >}})以及[使用lxc安装了docker]({{< ref "/post/linux/pve/Proxmox VE(PVE)8.0使用CT模板创建LXC版docker服务.md" >}}),之后就一直稳定使用，J4125上有一个2T的硬盘，于是就想着折腾一下在上边安装一下黑群晖，体验一下nas系统。于是就又开始了折腾之路，又学习了很多知识。下边就记录一下我折腾的过程。

## 准备

其实在开始装nas之前我看了很多教程，里边只是提供安装过程，并提供对应的安装包。说实在的，虽然很贴心，但是每个人的主机不尽相同，适合他的nas包不一定适合我，那么自己应该选择什么nas包呢？他们基本都没有说明，但是基本都是安装DS918+。于是在b站找到了一个[版本选择教程](https://www.bilibili.com/video/BV1KE421N7mN)，腾讯文档地址：[RR型号列表](https://docs.qq.com/sheet/DYXhtcVRPaXRCY0tv)。然后又看了rr安装，使用rr安装很方便：[RR地址](https://github.com/RROrg/rr)。

直接下载一个最新RR的镜像就可以了。至于nas的版本，两个选择，DS918+和SA6400，两个版本都是视频中推荐的版本，如果cpu是10代以后选择SA6400，如果10代以前则两个都可以。

## 安装

### 新建虚拟机

下载好镜像后就需要在pve中进行新建虚拟机了。步骤和openwrt差不多。

![new vm](https://images.iminling.com/app/hide.php?key=UW5tdlgzclNrZVdHd2FNU2hBSVlEOExGMG93WEdnbW5oczJjbXFNaGlRYlExTGpOZEhqNm9JazZhTFFtVDFFUk40UFNMVUE9)

常规选项：只需要填写一个名称就可以了。

操作系统：选择`不使用任何介质`。

系统：只需要将机型设置为`q35`，其他默认。

磁盘：直接把磁盘删除。

CPU：`核心数`根据自己的cpu设置，我这里给4.另外`类别`下拉到最后，选择`host`模式。

内存：内存给2-4GB都可以，我这里给4G.

网络：默认。模型为:VirtIo(半虚拟化)。

然后就是确认刚才填的信息，确认就可以了。

经过上边已经创建好了一个虚拟机，下边还需要上传刚才下载的rr镜像。

### 镜像上传

如图的位置进行上传。

![upload iso](https://images.iminling.com/app/hide.php?key=Mk9kNFJMNWJvTUhFRDZOL0dkVXFIVzlRQjdrV2kvNmtGTllBN3kwWEJTWVcxK1FpRmtEZjhYV2NFL3lBRjFFRTNNMnRHV009)

接下来在shell中使用命令将黑群晖的引导文件rr导入到创建好的DSM虚拟机，进入到`pve系统的shell`中执行。

命令如下：`qm importdisk 编号 /var/lib/vz/template/iso/rr.img local-lvm` , 其中编号为刚创建的虚拟机编号，rr.img的镜像上传路径基本不会有不同，基本都是在`/var/lib/vz/template/iso`目录下 再加上镜像名称，我这里就是`rr.img`。

我的完整命令如下：`qm importdisk 104 /var/lib/vz/template/iso/rr.img local-lvm`。执行成功后可以在nas虚拟机的硬件里看到一个未使用的磁盘，双击可添加磁盘：

总线设备选sata，磁盘映像默认就是刚看到的那个未使用的磁盘。其他的默认就可以了。

### 硬盘直通

经过上边的设置基本就可以了，此时还需要给nas添加一个磁盘。在pve的shell中通过命令找到我的固态盘ls -l /dev/disk/by-id/：

![disk](https://images.iminling.com/app/hide.php?key=aC9La1lEV1pSUHhsK1ZWcHZPN1hzL0JGQWExcjhnOEtPdUJWd3R0cnFyY2p3bEg2KzRMQ1lHSGNkRW9WWFIvRlkyNk9KMDA9)

可以看到ata开头的ZHITAI就是我的固态盘了。复制完整的名称：ata-ZHITAI\_SC001\_XT\_2000GB\_ZTB502TAB2404507RX。

然后通过命令直通给nas，set后边就是虚拟机的编号，我这里是103，后边--satax 根据自己的定义，刚添加的引导盘占了sata0,所以我这里就使用sata1 后边就是完整的磁盘路径 /dev/disk/by-id/拼上的判断完整名。

```
root@pve:~# qm set 103 --sata1  /dev/disk/by-id/ata-ZHITAI_SC001_XT_2000GB_ZTB502TAB2404507RX
update VM 103: -sata1 /dev/disk/by-id/ata-ZHITAI_SC001_XT_2000GB_ZTB502TAB2404507RX
```

执行完成后就可以看到新添加的磁盘了：
![nas vm](https://images.iminling.com/app/hide.php?key=dmlMc2RYVnRoTHNrNDV3UDlaNi80bDAzOUF3QzVoaVpiRXhzQ2w5VHFJTVdBRWErS2VDM1JtRHQ0ZWp6aExpaEZlWmZXNUU9)

然后修改启动顺序后就可以启动了。

把sata0设置为第一位：

![nas ](https://images.iminling.com/app/hide.php?key=THJaK0tLOGhrR1QxRjdXczNDcVIrZEhSY2FoeStlTmE5cUswZDVzKytvMlk5djFUN1ppRnNKVENtNWU3S2JoMU5SZ2xnNTg9)

启动成功后，在控制台中可以看到提示，会让访问一个ip:7681的网址去进行初次设置。

### 引导编译

访问7681端口的那个网址

先进行语言设置，选择Choose a language，找到zh-CN,切换语言为中文。

型号选择：我这里选择SA6400

版本选择：我这里选择DSM7.2

弹出的版本页面，点击URL会自动下载当前引导的系统文件（建议使用此方式下载，避免在群晖官网下载错系统），下载好的系统文件后缀为pat，然后点击确认。下边初始化会用到。

然后选择编译引导。

只需要等待。

编译好引导后会自动跳转到主菜单的启动选项，直接点击启动即可。

正常开机后，可以再次在`控制台`看到让访问`ip:5000`的端口，此时说明安装已经ok了。

## NAS初始化

接下来就是访问ip:5000来进行NAS的初始化,我这里的ip是192.1681.42:5000

![sa6400 1](https://images.iminling.com/app/hide.php?key=R0M1WXkrTnZTeGdUb1BPQzlENVhJNmZVK21td1A0T05rdG5PVGFUVm50cVVpV2hwUE05TERpUHg0WVE5YmZwZUZrdDIxRXc9)

安装。

![nas pat](https://images.iminling.com/app/hide.php?key=RGczYXZIWDlzZ0wrWmdETGFzWWZLVEVCaWZBaTIvNmVNOUlhdFplQ1F5bXFkTWw4S09INTBYekJjNUNNOURIVk5aTUhSdEE9)

选择`引导编译阶段`下载的pat文件。

![sa6400 clean disk](https://images.iminling.com/app/hide.php?key=VHduNWdkN2QyY2RocGEvbGNzOHBWelY1QXVESU5renFEWmU2TFA4N293dk14STQxQTVpNXZqWlFFQTkxcFNKZnZ5Mm5tb3c9)

提示会删除硬盘数据，继续进行确认。

![sa6400 install](https://images.iminling.com/app/hide.php?key=NmtEZ1ZGOE1SbXJlVENkbTFYQTBQaGFRVUJVVmdXWExPR3N3RmwvcVpITnJnNXJ6c1ZMSVRsUEd1RDNOVDlVM1FDYXNmZmc9)

等待安装。安装后就是重启，提示大概需要10分钟，实际上我等了3分钟就可以了。

![sa6400 reboot](https://images.iminling.com/app/hide.php?key=bjNUT2U1c1QvOHpVWDc5ZEJMQ3Jic1pCRU01VGhDZ1piR1FlNjEwR3BqTTd3eVZzTjlJMThWTTFjZ0lRY0xzcVJSV0dGMkE9)

重启完成后就会提示安装组件：

![sa6400 install](https://images.iminling.com/app/hide.php?key=VW54V3VOcDcvYVBFRnFQQjlSb1NIdVNKUDdhQmh3djlaZjR3bWZzMS95am1XMitiWlo3OWhYYW5nWU01L1FEaXMxbHU3elE9)

套件安装完成后进入安装引导：

![sa6400 install](https://images.iminling.com/app/hide.php?key=eGRka3lwRmVHcTFrZVl4QThxbXNELzFaSHJhcjNua2hUWUo4SjVwU0VtcmR3YTQ5aEY4ZmRrbEZFZ3kvSnZMSHJZVWJ1YVE9)

进入用户名和密码设置：

![sa6400 username setting](https://images.iminling.com/app/hide.php?key=UlI0WG8xcytRZm5KT1orMDYxcjBxbENyWjNiUEIvdHhSUjdKREd6RlUranBhOThtNjZVeENWclZqdmNValRWREx1ZHBCekk9)

接下来的更新就不需要了自动安装了：

![sa6400 udpate options](https://images.iminling.com/app/hide.php?key=aHVuLzFGaFFtTU1HKzE1VE8raitFNVErZjFkZGVXMmtzQnRCdTBVK1VucVRkZnNKYUZJVVIvaXJpNFd5Y2dkNVNodlUxblU9)

账号创建跳过：

![sa6400 account](https://images.iminling.com/app/hide.php?key=MU5XZFJXZ09VR2xVZkt3dS8vNEVzWitTQXIrOGJoVmxaZnVzb0tkUlBURk4xZVFXMDQzVHMwaU1EaXhrUVJPTU9nQlhieHc9)

用户体验不勾选，提交：

![sa6400 install](https://images.iminling.com/app/hide.php?key=QW5DMC95SVl5Q0pwSEpTSDV1aEw4b0ZZY1MwY2lWR1ZsTWo0U01UM2JPYmN2SlpqK3dOcHFkNzRkdW96Q3diQ1FNS213c1k9)

经过上边的一系列设置，终于来到了nas的桌面中。

### 桌面设置

群晖套件：不用了，谢谢。

![synology setting](https://images.iminling.com/app/hide.php?key=Q01pd2g3dFo0SmVOdWdvUUNjQjY2Q0h3TTJTZk9PSnl0OXJyTVJQUm9qeFRXUW0ydnNFZnJrSDBMNnVDUkNFWHNablFjbHc9)

双重2FA不开启：

![synology 2fa](https://images.iminling.com/app/hide.php?key=b2oyVytyblo2TnV2dUlzQzBReHE1RlVkTFJOYk5paXBDbmlRekR3dmcwYXFjc1cvZjZPZGJzRVQyeHQ2WmxmNXNEQnFIL3M9)

下一步账户保护不开启。

终于都设置完了。。。

### 磁盘设置

经过上边的设置，终于提示要设置磁盘了。

![sa6400 disk setting](https://images.iminling.com/app/hide.php?key=SjhTMHBELzNqUjFYZ01xbEtZSWY0R1BoOHBTUnFrdnl1WkJBMTBHdk0rRTZHWkFQb0x0blZUeXBDeGxGQUkreXBvVEI1ZDQ9)

点击开始进行设置：

![sa6400 raid](https://images.iminling.com/app/hide.php?key=dFNTWWc5QWthUEVZMTVObFEybnZGQk1JSEROMVZ4YnkvU2pRbHJTMkVBQUw2SHBkditpb3c0ZXJmVEtDTEdYUjdDTTFXakU9)

raid类别根据自己的需要选择，我这里 就一个硬盘，直接选择basic类型。继续下一步,选择硬盘，我这里需要一个硬盘：
![sa6400 disk select](https://images.iminling.com/app/hide.php?key=dmlMc2RYVnRoTHNrNDV3UDlaNi80akJEMnpsS0lhckd5MXd1VXllcHVoM0o2aDE2ZGcyQjZVeEl2bGNJYm1NWUdCTXVmWkk9)

继续下一步，跳过硬盘检查：

![sa6400 disk check](https://images.iminling.com/app/hide.php?key=Unk0QjN0amt5MmZUb1ByamhJMVJjVHljQWNCUXA2bjVyV2hCUURaMXpZcndqNnRZemhSRnRHdDFvUmh4aUwyOW9xWDhIdjA9)

继续下一步配置磁盘容量：

![sa6400 disk cap](https://images.iminling.com/app/hide.php?key=NGtJSkQrTXRhOVFBbXJySVNDbjZjSmJVUFdCMTU0aS9GTkl0cjR6NVpFK0pIcFJXQitKTVBHL1ZoQ1pqTjBPTnRPa2VTYlU9)

直接最大化，整个都分配给nas，继续下一步文件系统：

![sa6400 disk file type](https://images.iminling.com/app/hide.php?key=dExZaDkwMkc3ZitQdkFtdmFabVZYcUdPelFjRlZZRHM3OGdXeGRwc0lVRi9POTVMR2Jja2xZNFcvdUtkMFFhYVNGSFJlR2M9)

这里我就选择他推荐的，继续下一步配置加密：

![sa6400 file encrypt](https://images.iminling.com/app/hide.php?key=Rkx4TUllS2M5S3Z2RTFJY3BteUxsQ1BlSFZIVG5FOWJZL2orVzNBRTNmMFBEZjRMRVRTZmtLejREV3ZtRlJjSWFYSlZ6QUE9)

我这里不加密，默认，继续下一步，对刚才的配置在进行概览：

![sa6400 configuration check](https://images.iminling.com/app/hide.php?key=aWtSeWdQRnQxazhMN0RNdGFCdytHa2Q5V3ZjNW5LU2ZPdXNnOGg2UkVUWG1sK3dGZmdzbm5kZTRxb2ZmL3NrTllGdnV0ZTA9)

配置无问题应用就可以了。终于整个nas都配置后了。

最后再上一个桌面显示的图片：

![sa6400 desktop](https://images.iminling.com/app/hide.php?key=TlUwdWJRaGpRZ2llajJNcFpuYU1oZjByWGFCTml2Qjl4R0EvRk5FRkROMThwaldUTFJhODlrNk94RkZZbnZpWlkrS29ab1U9)

以上就是pve安装黑群晖的折腾记录。
