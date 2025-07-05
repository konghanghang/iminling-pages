---
title: 发布个人项目的jar包文件到maven中央仓库
author: 要名俗气
type: post
date: 2024-05-08T14:33:57+00:00
url: /2024/push-jar-to-maven-central
description: 自己有一个项目想要在任何机器上都能引用到，最简单的办法就是在所要使用的机器上把项目下载下来，然后install到本地仓库，但是如何直接通过引用坐标就能引用到呢？答案就是发布到maven的中央仓库，这样子无论你在什么机器上，只要能上网输入gav三个坐标就可以引入了。这篇文章就介绍一下自己项目打成jar包并发布到maven中央仓库的完整过程。
featured_image: /wp-content/uploads/2024/05/011ABDE59EE899C2F7CB8D9ECBEC066C.png
categories:
  - Maven
tags:
  - jar
  - java
  - maven
---
自己有一个项目想要在任何机器上都能引用到，最简单的办法就是在所要使用的机器上把项目下载下来，然后install到本地仓库，但是如何直接通过引用坐标就能引用到呢？答案就是发布到maven的中央仓库，这样子无论你在什么机器上，只要能上网输入gav三个坐标就可以引入了。这篇文章就介绍一下自己项目打成jar包并发布到maven中央仓库的完整过程。mac安装maven可以参考我的另一篇文章：[Mac使用Homebrew安装maven环境](https://www.iminling.com/2023/10/27/277.html "Mac使用Homebrew安装maven环境")。

## 账号注册

首先我们要先注册sonatype账号，访问地址<a href="https://issues.sonatype.org/secure/Signup!default.jspa" target="_blank" rel="nofollow noopener">sonatype</a>输入必须的内容就可以成功注册一个账号,不过对密码就有一些特殊的安全要求，正确注册就可以了。

## sonatype工单 {#item-2}

### 新建工单

点击新建按钮，项目选择`open`的那个，问题类型选择`new project`，概要，描述随便写就ok了

![sonatype工单](https://www.iminling.com/wp-content/uploads/2024/05/743AA473A26B0C2236A6C3B2EC8840BC.png)

摘要根据自己需要填写，我这里填写的是项目的名称。

group id就是maven坐标的名称，一般是域名反转过来。例如我的域名是iminling.com,则这里填写com.imining.

project URL则是github的仓库地址，或者其他代码仓库的地址。

SCM url就是在project URL后边加上`.git`

其他的默认就可以了，创建。新建完成后如下图：

![工单](https://www.iminling.com/wp-content/uploads/2024/05/1CBB45AE01C8EE964E17AC1E924592CC.png)

### 添加TXT记录

如上边的图所示，它为了验证你是域名的所有者，会让你去解析一条txt记录。两种方案选一种就可以了，我这里选择的是添加一条txt的记录，如下图所示，我这里是不清楚规则，提交了两个工单，所以添加了两条记录，最后其中一个工单被认为是重复提交，已关闭。其中记录值填写你的工单地址，下图中框住的部分，主机记录就是`jira tiket`.

![sonatype txt](https://www.iminling.com/wp-content/uploads/2024/05/E5A938D31A629B94379B42FA6DBAD231.png)

这里txt解析的值来源就是你的问题url，如下：

![txt record](https://www.iminling.com/wp-content/uploads/2024/05/ED9DE32D2805BFC9248F67E6FE8C774C.png)

解析完后就可以再等待审核了，我的大概是凌晨3点进行的审核，通过以后会有邮件通知，工单下边也有评论,此时我们就可以准备发布我们的jar包了。

```
com.iminling has been prepared, now user(s) yslaoo can:
Publish snapshot and release artifacts to https://oss.sonatype.org
Have a look at this section of our official guide for deployment instructions:
https://central.sonatype.org/pages/ossrh-guide.html#deployment

Please comment on this ticket when you've released your first component(s), so we can activate the sync to Maven Central.
Depending on your build configuration, this might happen automatically. If not, you can follow the steps in this section of our guide:
https://central.sonatype.org/pages/releasing-the-deployment.html
```

## 发布准备

### gpg安装

mac可以使用用brew进行安装 `brew install gpg`就可以了。

windows安装了git客户端就自带了这个功能。

检查自己的gpg的版本，有些是gpg，有些则需要输入gpg2.

```
# 或者使用gpg2 --version 就看自己的电脑上哪个命令可以运行
$ gpg --version
gpg (GnuPG) 2.2.13-unknown
libgcrypt 1.8.4
Copyright (C) 2019 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /c/Users
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
        CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
```

### 生成key

不论是mac还是windows都是使用命令`gpg --gen-key`来生成。

```
$ gpg --gen-key
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

注意：使用 “gpg --full-generate-key” 以获得一个功能完整的密钥产生对话框。

GnuPG 需要构建用户标识以辨认您的密钥。

真实姓名： yslaoo
电子邮件地址： ysl@test.com
您选定了此用户标识：
    “ysl <ysl@test.com>”

更改姓名（N）、注释（C）、电子邮件地址（E）或确定（O）/退出（Q）？ O
我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。
我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。
gpg: /Users/aaa/.gnupg/trustdb.gpg：建立了信任度数据库
gpg: 密钥 84040E735F931A32 被标记为绝对信任
gpg: 目录‘/Users/aaa/.gnupg/openpgp-revocs.d’已创建
gpg: 吊销证书已被存储为‘/Users/aaa/.gnupg/openpgp-revocs.d/DD1E1B8213D07DA46FC3F2B684040E735F931A32.rev’
公钥和私钥已经生成并被签名。

pub   rsa3072 2021-02-20 [SC] [有效至：2023-02-20]
      DD1E1B8213A07DA46FC3F2B684040E735F931A32
uid                      yslao <yslao@outlook.com>
sub   rsa3072 2021-02-20 [E] [有效至：2023-02-20]
```

期间会让输入密码，**牢记密码**，后边会使用。

### key操作

生成key后需要发布key到公有服务器。

先查看key：

```
$ gpg --list-keys
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2023-02-20
/c/Users/aaa/.gnupg/pubring.kbx
---------------------------------
pub   rsa2048 2021-02-20 [SC] [expires: 2023-02-20]
      C87B0403E54CD05D431E5C1A7204BFB944405DA7
uid           [ultimate] ysl <ysl@test.com>
sub   rsa2048 2021-02-20 [E] [expires: 2023-02-20]
```

然后发布：

```
# 命令格式：gpg --keyserver [key的服务器](这个有很多，随便找一个就行了) --send-keys [key] key就是查看key操作中pub对应的那串字符串
$ gpg --keyserver hkp://keyserver.ubuntu.com:11371 --send-keys C87B0403E54CD05D431E5C1A7204BFB944405DA7
gpg: sending key 7204BFB944405DA7 to hkp://keyserver.ubuntu.com:11371
```

### Distribution 管理

修改pom.xml, 添加以下代码

```
<!--父级是project-->
<distributionManagement>
    <snapshotRepository>
        <id>ossrh</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
    </snapshotRepository>
    <repository>
        <id>ossrh</id>
        <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
    </repository>
</distributionManagement>

<build>
  <plugins>
    <plugin>
      <groupId>org.sonatype.plugins</groupId>
      <artifactId>nexus-staging-maven-plugin</artifactId>
      <version>1.6.7</version>
      <extensions>true</extensions>
      <configuration>
        <serverId>ossrh</serverId>
        <nexusUrl>https://oss.sonatype.org/</nexusUrl>
        <autoReleaseAfterClose>true</autoReleaseAfterClose>
      </configuration>
    </plugin>
  </plugins>
</build>
```

### 认证配置

在`setting.xml`中添加认证信息,此处的id要和`pom`文件中的`distributionManagement`下`snapshotRepository`和`repository`的id保持一致.

```
<settings>
  <servers>
    <server>
      <id>ossrh</id>
      <!-- username就是注册sonatype时的username -->
      <username>your-jira-id</username>
      <!-- password就是注册sonatype时的password -->
      <password>your-jira-pwd</password>
    </server>
  </servers>
</settings>
```

### javadoc和源代码配置

在pom.xml中添加配置如下

```
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-source-plugin</artifactId>
      <version>2.2.1</version>
      <executions>
        <execution>
          <id>attach-sources</id>
          <goals>
            <goal>jar-no-fork</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-javadoc-plugin</artifactId>
      <version>2.9.1</version>
      <executions>
        <execution>
          <id>attach-javadocs</id>
          <goals>
            <goal>jar</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

### gpg签名配置

在pom中添加gpg插件

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-gpg-plugin</artifactId>
    <version>1.5</version>
    <executions>
        <execution>
            <id>sign-artifacts</id>
            <phase>verify</phase>
            <goals>
                <goal>sign</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

在setting.xml中添加gpg profile配置,`gpg.executable`属性要根据自己的电脑环境进行添加.

```
<settings>
  <profiles>
    <profile>
      <id>ossrh</id>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <properties>
        <!--这里根据实际情况填写gpg或gpg2,看自己的环境能使用哪个命令-->
        <gpg.executable>gpg2</gpg.executable>
        <!--passphrase就是我们在gpg安装生成key的时候设置的-->
        <gpg.passphrase>the_pass_phrase</gpg.passphrase>
      </properties>
    </profile>
  </profiles>
</settings>
```

### Nexus Staging Maven插件，用于部署和发布

在pom.xml中添加以下内容

```
<plugin>
  <groupId>org.sonatype.plugins</groupId>
  <artifactId>nexus-staging-maven-plugin</artifactId>
  <version>1.6.7</version>
  <extensions>true</extensions>
  <configuration>
     <serverId>ossrh</serverId>
     <nexusUrl>https://oss.sonatype.org/</nexusUrl>
     <autoReleaseAfterClose>true</autoReleaseAfterClose>
  </configuration>
</plugin>
```

经过上边操作，准备工作已完毕，下边就可以开始发布jar包了。

## 发布

所有的发布操作确保gpg命令是可以用的,在windows下进行发布一定要注意是在`git bash`客户端中进行,以确保gpg可以使用.以及发布过程中可能会让你再次输入gpg的密码，这里需要注意一下。

### 快照版本

项目的版本如果是以`-SNAPSHOT`结尾的,就会发布到快照仓库,如下:

```
D:\project\idea\base-iminling-parent>mvn clean deploy
INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.iminling:base-iminling-parent:pom:1.0.0-SNAPSHOT
[WARNING] 'build.pluginManagement.plugins.plugin.(groupId:artifactId)' must be unique but found duplicate declaration of plugin org.sonatype.plugins:nexus-staging-
maven-plugin @ line 326, column 25
[WARNING]
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING]
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING]
[INFO]
[INFO] -----------------< com.iminling:base-iminling-parent >------------------
[INFO] Building base-iminling-parent 1.0.0-SNAPSHOT
[INFO] --------------------------------[ pom ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ base-iminling-parent ---
[INFO]
[INFO] --- maven-install-plugin:2.4:install (default-install) @ base-iminling-parent ---
[INFO] Installing D:\project\idea\base-iminling-parent\pom.xml to D:\maven-repository\com\iminling\base-iminling-parent\1.0.0-SNAPSHOT\base-iminling-parent-1.0.0-S
NAPSHOT.pom
[INFO]
[INFO] --- maven-deploy-plugin:2.7:deploy (default-deploy) @ base-iminling-parent ---
Downloading from ossrh: https://oss.sonatype.org/content/repositories/snapshots/com/iminling/base-iminling-parent/1.0.0-SNAPSHOT/maven-metadata.xml
Uploading to ossrh: https://oss.sonatype.org/content/repositories/snapshots/com/iminling/base-iminling-parent/1.0.0-SNAPSHOT/base-iminling-parent-1.0.0-20210220.03
4207-1.pom
Uploaded to ossrh: https://oss.sonatype.org/content/repositories/snapshots/com/iminling/base-iminling-parent/1.0.0-SNAPSHOT/base-iminling-parent-1.0.0-20210220.034
207-1.pom (14 kB at 4.8 kB/s)
Downloading from ossrh: https://oss.sonatype.org/content/repositories/snapshots/com/iminling/base-iminling-parent/maven-metadata.xml
Uploading to ossrh: https://oss.sonatype.org/content/repositories/snapshots/com/iminling/base-iminling-parent/1.0.0-SNAPSHOT/maven-metadata.xml
Uploaded to ossrh: https://oss.sonatype.org/content/repositories/snapshots/com/iminling/base-iminling-parent/1.0.0-SNAPSHOT/maven-metadata.xml (609 B at 263 B/s)
Uploading to ossrh: https://oss.sonatype.org/content/repositories/snapshots/com/iminling/base-iminling-parent/maven-metadata.xml
Uploaded to ossrh: https://oss.sonatype.org/content/repositories/snapshots/com/iminling/base-iminling-parent/maven-metadata.xml (292 B at 54 B/s)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 15.105 s
[INFO] Finished at: 2021-05-20T11:42:18+08:00
[INFO] ------------------------------------------------------------------------
```

### release版本

项目的版本不是以`-SNAPSHOT`结尾的,就会发布到release仓库,如下:

```
D:\project\idea\base-iminling-parent>mvn clean deploy
[INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.iminling:base-iminling-parent:pom:1.0.0
[WARNING] 'build.pluginManagement.plugins.plugin.(groupId:artifactId)' must be unique but found duplicate declaration of plugin org.sonatype.plugins:nexus-staging-
maven-plugin @ line 326, column 25
[WARNING]
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING]
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING]
[INFO]
[INFO] -----------------< com.iminling:base-iminling-parent >------------------
[INFO] Building base-iminling-parent 1.0.0
[INFO] --------------------------------[ pom ]---------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.5:clean (default-clean) @ base-iminling-parent ---
[INFO]
[INFO] --- maven-install-plugin:2.4:install (default-install) @ base-iminling-parent ---
[INFO] Installing D:\project\idea\base-iminling-parent\pom.xml to D:\maven-repository\com\iminling\base-iminling-parent\1.0.0\base-iminling-parent-1.0.0.pom
[INFO]
[INFO] --- maven-deploy-plugin:2.7:deploy (default-deploy) @ base-iminling-parent ---
Uploading to ossrh: https://oss.sonatype.org/service/local/staging/deploy/maven2/com/iminling/base-iminling-parent/1.0.0/base-iminling-parent-1.0.0.pom
Uploaded to ossrh: https://oss.sonatype.org/service/local/staging/deploy/maven2/com/iminling/base-iminling-parent/1.0.0/base-iminling-parent-1.0.0.pom (14 kB at 59
7 B/s)
Downloading from ossrh: https://oss.sonatype.org/service/local/staging/deploy/maven2/com/iminling/base-iminling-parent/maven-metadata.xml
Uploading to ossrh: https://oss.sonatype.org/service/local/staging/deploy/maven2/com/iminling/base-iminling-parent/maven-metadata.xml
Uploaded to ossrh: https://oss.sonatype.org/service/local/staging/deploy/maven2/com/iminling/base-iminling-parent/maven-metadata.xml (312 B at 51 B/s)
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 32.049 s
[INFO] Finished at: 2021-02-20T14:11:23+08:00
[INFO] ------------------------------------------------------------------------
```

### 遇到的问题

在mac上进行发布的时候遇到下边问题：

```
[INFO] --- maven-gpg-plugin:1.5:sign (sign-artifacts) @ base-iminling-parent ---
gpg: 签名时失败： Inappropriate ioctl for device
gpg: signing failed: Inappropriate ioctl for device
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 17.069 s
[INFO] Finished at: 2021-02-21T09:36:03+08:00
[INFO] ------------------------------------------------------------------------
```

上网查询后，原因是 gpg 在当前终端无法弹出密码输入页面。

解决办法很简单在终端执行`export GPG_TTY=$(tty)` 。重新执行，发现会弹出一个密码输入界面。

### 同步中央仓库

发布后我们还需要在sonatype中问题下方进行评论,来激活同步到maven中心仓库.

![sync maven centrl](https://www.iminling.com/wp-content/uploads/2024/05/5A48C989D866A911702A5ED09278C106.png)

后续的release就可以同步到maven仓库了。

## 版本引用

### release

正常使用gav三个坐标来引入就可以了。可能刚发布的并没有同步到所有的仓库，需要等待同步，例如可以去阿里仓库里搜索几遍，让它去同步。

### 快照版本

快照版本需要引入仓库地址：

```
<!--定义snapshots库的地址-->
<repositories>
    <repository>
        <id>sonatype-snapshots</id>
        <name>sonatype-snapshots</name>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
<!--经测试,不要下边的应该也是可以的,留着做不时之需-->
<pluginRepositories>
    <pluginRepository>
        <id>sonatype-snapshots</id>
        <name>sonatype-snapshots</name>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </pluginRepository>
</pluginRepositories>
```

上边就是通过maven发布jar包到maven中央仓库的的过程，希望可以帮助到大家。
