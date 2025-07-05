---
title: 使用gradle kts脚本发布个人项目的jar包文件到maven中央仓库
author: 要名俗气
type: post
date: 2024-05-19T02:02:03+00:00
url: /2024/gradle-kts-push-jar-to-maven-central
description: 上篇文章记录了如何使用maven来发布自己的个人项目到中央仓库，后边项目改使用gradle来管理依赖了，为了能及时的把jar发布到maven仓库，所以就又折腾了一遍如何使用gradle来发布自己的jar到中央仓库，在这里记录一下折腾和踩坑经历，并分享给大家。
featured_image: /wp-content/uploads/2023/10/3F5D90BA94A32843C781364B437AB148.png
categories:
  - Gradle
tags:
  - Gradle
  - jar
  - java
  - maven
---
上篇文章记录了如何使用maven来发布自己的个人项目到中央仓库，后边项目改使用gradle来管理依赖了，为了能及时的把jar发布到maven仓库，所以就又折腾了一遍如何使用gradle来发布自己的jar到中央仓库，在这里记录一下折腾和踩坑经历，并分享给大家。

## 前提准备

如果你是第一次发布jar到仓库，则需要参考我的另一篇文章：[发布个人项目的jar包文件到maven中央仓库](https://www.iminling.com/2024/05/08/603.html "发布个人项目的jar包文件到maven中央仓库") 来创建好sonatype账号并生成gpg key。这些就不在另外说明了。

本篇文章就是在我已经使用了maven发布过jar到中央仓库，然后在此基础上把项目迁移到gradle，然后使用gradle来继续发布项目到中央仓库。

## gradle配置

### 设置发布plugins

想要把jar发布的maven仓库需要先将项目的代码打包成jar，然后对jar进行签名，最后发布到maven仓库，三步对应三个插件：

```
plugins {
    `java-library`
    `maven-publish`
    signing
}
```

上边就添加了三个插件，将用于后续的发布使用。

### 配置jar打包

下边设置对项目进行打包成jar的设置

```
tasks.jar {
    dependsOn(tasks.withType(GenerateMavenPom::class))
    manifest {
        attributes("Created-By" to "Gradle ${project.ext["gradleVersion"]}")
    }
    into("META-INF") {
        from("$buildDir/publications/mavenJava")
        exclude("*.asc")
        rename { it.replace("pom-default.xml", "pom.xml") }
    }
}

tasks.withType<JavaCompile> {
    options.encoding = "UTF-8"
}

java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
    withJavadocJar()
    withSourcesJar()
}
```

上边指定了jdk的版本，并且生成javadoc和java source文件，指定打包的编码以及对manifest文件属性添加等一系列的操作。

### 配置发布

主要是配置仓库，pom文件以及签名等信息，先贴一下整个配置，下面单独讲。

```
publishing {
    publications {
        create<MavenPublication>("mavenJava") {
            artifactId = project.name
            from(components["java"])
            /*versionMapping {
                usage("java-api") {
                    fromResolutionOf("runtimeClasspath")
                }
                usage("java-runtime") {
                    fromResolutionResult()
                }
            }*/
            pom {
                name.set(project.name)
                description.set("A concise description of my library")
                url.set("https://www.example.com/library")
                /*properties.set(mapOf(
                    "myProp" to "value",
                    "prop.with.dots" to "anotherValue"
                ))*/
                licenses {
                    license {
                        name.set("The Apache License, Version 2.0")
                        url.set("https://www.apache.org/licenses/LICENSE-2.0.txt")
                    }
                }
                developers {
                    developer {
                        id.set("yslao")
                        name.set("yslao")
                        email.set("yslao@outlook.com")
                    }
                }
                scm {
                    connection.set("scm:git:git://github.com/konghanghang/base-iminling-core.git")
                    developerConnection.set("scm:git:ssh://github.com/konghanghang/base-iminling-core.git")
                    url.set("https://github.com/konghanghang/base-iminling-core")
                }
            }
        }
    }

    repositories {
        maven {
            name = "mavenCentral"
            // change URLs to point to your repos, e.g. http://my.org/repo
            // val releasesRepoUrl = uri(layout.buildDirectory.dir("repos/releases"))
            val releasesRepoUrl = uri("https://oss.sonatype.org/service/local/staging/deploy/maven2/")
            val snapshotsRepoUrl = uri("https://oss.sonatype.org/content/repositories/snapshots")
            url = if (version.toString().endsWith("SNAPSHOT")) snapshotsRepoUrl else releasesRepoUrl
            credentials {
                // System.getenv放在.bash_profile中
                // System.getProperty可以放在命令行也可以放在gradle.properties中
                username = System.getProperty("SONATYPE_NEXUS_USERNAME")
                password = System.getProperty("SONATYPE_NEXUS_PASSWORD")
            }
        }
    }

    signing {
        sign(publishing.publications["mavenJava"])
    }

    tasks.javadoc {
        if (JavaVersion.current().isJava9Compatible) {
            (options as StandardJavadocDocletOptions).addBooleanOption("html5", true)
        }
        (options as StandardJavadocDocletOptions).encoding("UTF-8")
    }
}
```

整个publishing共分为四部分：

### publications

指定jar的version，groupId和artifactId。另外还有pom的信息，这个在使用maven发布的时候也设置过

### repositories

指定仓库地址，以及配置sonetype的账号和密码，可以放在用户目录的gradle.properties中：

```
# k @ MacBook-Pro in ~/.gradle [9:30:05]
$ pwd
/Users/k/.gradle

# k @ MacBook-Pro in ~/.gradle [9:30:06]
$ ls
caches            gradle.properties kotlin-profile    notifications     wrapper
daemon            jdks              native            workers
# k @ MacBook-Pro in ~/.gradle [9:30:09]
$ cat gradle.properties
systemProp.SONATYPE_NEXUS_USERNAME=aaa
systemProp.SONATYPE_NEXUS_PASSWORD=aaa
```

如上在gradle.properties中以key-value的形式进行配置。另外同时配置了快照版本的发布地址和release版本的发布地址，通过项目的版本好结尾是不是`SNAPSHOT`来确定到底是发布到哪里。

### signing

指定签名哪个publications，名字要和上边publications中配置的名称一致。另外有一点还需要特别进行配置，我maven发布的时候使用gpg生成了key，在使用gradle的时候还需要导出私钥，同样是导出到用户家目录下的.gnupg目录下，进入到此目录执行`gpg --export-secret-keys -o secring.gpg` ：

```
# k @ MacBook-Pro in ~ [9:39:08] C:1
$ cd .gnupg

# k @ MacBook-Pro in ~/.gnupg [9:39:13]
$ ls
common.conf       openpgp-revocs.d  public-keys.d     trustdb.gpg
crls.d            private-keys-v1.d secring.gpg

# k @ MacBook-Pro in ~/.gnupg [9:39:14]
$ gpg --export-secret-keys -o secring.gpg
```

会生成一个secring.gpg的文件，这时候需要在`/Users/k/.gradle/gradle.properties`进行配置

```
# k @ MacBook-Pro in ~/.gradle [9:41:37]
$ cat gradle.properties
signing.keyId=44405DA7
signing.password=12345
signing.secretKeyRingFile=/Users/k/.gnupg/secring.gpg
systemProp.SONATYPE_NEXUS_USERNAME=aa
systemProp.SONATYPE_NEXUS_PASSWORD=aa
```

上边有三个参数，keyId使用gpg --list-keys中展示出来的pub下`C87B0403E54CD05D431E5C1A7204BFB944405DA7` 中的后8位。

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

password则是生成gpg key的时候输入的密码，这个在maven发布的那篇文章中讲过，有不清楚的可以再去看一下那篇文章。

secretKeyRingFile则是刚才使用gpg导出的文件。

签名相关的配置就已经配置完成了。

### javadoc

这个配置相对简单，指定编码。

经过上边的配置基本就已经完成了，可以去发布了。

## 发布

publishing下有几个选项，主要用的还是publishToMavenLocal，这个是发布到本地的maven仓库，可以在本地测试使用。另外就是publish直接往maven仓库中发布。

![gradle](https://www.iminling.com/wp-content/uploads/2024/05/7CAE3057848173E2CCEB4307097347CF.png)

双击上边的任务进行运行，或者直接在终端使用gradle + 具体的参数来运行，比如：`gradle publishToMavenLocal` 。

## 遇到的问题

在经过上边配置后发现发布到maven仓库里的只有.moudle的文件，并没有我们的jar和doc相关的文件，经过网上查询添加以下配置就可以了：

```
// 解决打包后生成.module文件，导致发布到中央仓库时只上传了.module文件没有上传jar文件
tasks.withType<GenerateModuleMetadata> {
      enabled = false
}
```

以上就是使用gradle发布jar到中央仓库的折腾过程。

参考：

> [Publishing a project as module](https://docs.gradle.org/current/userguide/publishing_setup.html "Publishing a project as module")
