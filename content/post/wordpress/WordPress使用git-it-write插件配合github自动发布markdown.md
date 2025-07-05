---
title: WordPress使用git-it-write插件配合github自动发布markdown
author: 要名俗气
type: post
date: 2025-04-06T09:20:36+00:00
excerpt: 以前发布文章都是直接在wordpress后台手动编辑，手动上传文件，确认无误后再发布，整个过程比较耗时，并且有点麻烦。最近这两天忽然有一个想法：我直接写markdown然后，然后在wordpress中找支持markdown的编辑器插件，把我的内容直接粘贴进去，这样子不就方便很多了吗？于是就找到了WP-Githuber-MD这个插件，再探索的过程中无意又发现了git-it-write这个可以自动化的插件，于是就开始了折腾之旅行。
url: /2025/wordpress-use-git-it-write-and-github-publish-markdown-file
description: 详细介绍WordPress使用git-it-write插件配合GitHub自动发布markdown文章的完整流程，包括插件安装配置、仓库设置、Webhook配置等实用技巧。
sha:
  - c83493c52aa405d3e38f1bc7f32699e8409133d4
github_url:
  - https://github.com/konghanghang/blog_pages/blob/master/wordpress/wordpress-use-git-it-write-and-github-publish-markdown-file.md
categories:
  - wordpress
tags:
  - git-it-write
  - github
  - markdown
  - wordpress
---
# WordPress使用git-it-write插件配合github自动发布markdown

![What Is Markdown, and How Do You Use It?](https://images.iminling.com/app/hide.php?key=SlBudmJhK1kyNkJ5NGtnVm53c1kwOUpBdFNJd1I2dHR5NXk5VjM1QkNsN1VtWm9DVWNJdi9jdEEyL0o0eTZDTXJPNUhlT3c9)
</figure>

以前发布文章都是直接在wordpress后台手动编辑，手动上传文件，确认无误后再发布，整个过程比较耗时，并且有点麻烦。最近这两天忽然有一个想法：我直接写markdown然后，然后在wordpress中找支持markdown的编辑器插件，把我的内容直接粘贴进去，这样子不就方便很多了吗？于是就找到了**WP-Githuber-MD**这个插件，再探索的过程中无意又发现了**git-it-write**这个可以自动化的插件，于是就开始了折腾之旅行。

## 插件安装和配置

Git-it-write插件在Wordpress的插件库是可以找到的，直接搜索安装就可以了。安装后可以在设置中找到对这个插件的配置。<figure class="wp-block-image size-full">

![image-20250406105054694](https://images.iminling.com/app/hide.php?key=MWp2VEQ0OWJhQ1ZvS3dTUmxGNHJ4VFhsc2JQcFFRY21MS0VORWFhSzRRekFOT2ZQYzBkcEl0WjlORk1pKy9ONHFiM3BRK1E9)
</figure>

## 仓库配置

想要配合github进行自动发布，首先要解决的就是从仓库里拉去到`.md`的文件，所以需要去配置一个git仓库，我们可以先创建好这个git仓库，然后再来进行配置。

点击左上的`add a new repository to publish posts form`按钮来添加一个新的仓库：<figure class="wp-block-image size-full">

![image-20250406105418250](https://images.iminling.com/app/hide.php?key=aHNJb2F0ZE5jMFU4azFuZ05GTVlTNS9UaHNnazV3RXhDT3pnY1F4Zk5iTkcvSzlhYy91R3JJVG5KTlBXQy96YWkzUENRajA9)
</figure>

有几个地方需要设置一下：

| 配置项                     | 说明                                |
| ----------------------- | --------------------------------- |
| Github username/owner   | 这个自己的github用户名                    |
| Repository name         | 仓库名称，新建或者已存在的一个仓库名称               |
| Branch to publish from  | 仓库的分支，根据自己的仓库来设置，master, main等    |
| Folder to publish from  | 我整个仓库都是需要发布的，而不是只发布其中一个目录，所以这里留空。 |
| Post type to publish to | 我要发布的是文章，自己根据自己的需求来选择。            |

其他的选项就默认了。

因为我这里建的是一个私有的仓库，想要拉取到仓库里的内容，我这里需要配置一下github access token,填在General settings的位置。如果是public类型的仓库，可以尝试一下不配置或者也配置一下，因为我不是这种情况，所以没有进行详细的研究。下边就来看一下如何获取github access token.

## 获取github access token

来到`github`的`settings`页面：登录`github`后，点击右上角的头像，就会看到有`settings`选项。进入`settings`后，拉到最下边找到`developer settings`选项，如下图：<figure class="wp-block-image size-full">

![image-20250406110451346](https://images.iminling.com/app/hide.php?key=dloxM1QyZDBSeHpkSHhuUXFnOXE3dE9KRjJ0N056ekYyb1pPMFNpak1ITmM0MWZjUHVkMDRVNVFIQjJOOW9LYkdjOXJPdHc9)
</figure>

点击在personal access tokens中进行添加和删除等操作：<figure class="wp-block-image size-full">

![image-20250406110617783](https://images.iminling.com/app/hide.php?key=UlVOOEFEekZDVTZkWkV1NElkd2U1NC9vWXYwVVZGRFdXZnNhMG1sZGlSZWx6NkpOT1dId201RmdaZkJVR3YvSEJQZkZvRkk9)
</figure>

点击右上角的Generate new token来新建一个token，设置页面如下：<figure class="wp-block-image size-full">

![image-20250406110820880](https://images.iminling.com/app/hide.php?key=NVpqaXRGRTUyZUQ1QytZdTA1WWthQzBLU2NZdjJqd0lrT2hEYkxTSjZuWEoyaG5kUHlQa2hlaDFYNC95anpwME5sWFNHYU09)
</figure>

相关的配置项说明：

| 配置项               | 说明                                                                 |
| ----------------- | ------------------------------------------------------------------ |
| Token name        | 自定义的一个说明名称，可以随意配置                                                  |
| Description       | 说明，可以留空                                                            |
| Resource owner    | 正常情况下，选择自己的github用户就可以了                                            |
| Expiration        | token的过期时间，根据自己的情况设置，我这里设置永不过期                                     |
| Repository access | 仓库访问权，我这里选择只给他访问特定的仓库                                              |
| Permissions       | 这里需要注意了，**Repository permission一定要点开，然后对每项进行设置**，我这里每项都只给access权限。 |

设置完成后就可以生成token了，把生成的tokent填写在`General settings`处了：<figure class="wp-block-image size-full">

![image-20250406111447759](https://images.iminling.com/app/hide.php?key=a3FndlRxSDZOd2xlWmVlS3pXT0FOTURqRjNWRzVMeXVsQ1AvYWpEUTc2SG1NNkhWdE9kU3U0WitWb0VSZWxIbXdsdTZnRTQ9)
</figure>

Github username就是自己的github用户名。

至此git it write插件就配置完成了。现在要做的就是在仓库里写一篇markdown格式的文档来进行发布了。

## 发布

在我们写完markdown文件后，push到上边配置的git仓库中，下边就可以来进行发布操作了。<figure class="wp-block-image size-full">

![image-20250406164203217](https://images.iminling.com/app/hide.php?key=eUtGZTQ1cTJqbWl2dHN0SVhtS1U4a0NuMEpIbTdwbE54VklTc1hldWVEMUphY2JYRTJJVGNXdlM4d2JFekxkMjNpWlNoUkU9)
</figure>

在wordpress插件git it write插件中添加仓库后，可以看到如上图所示的内容，我们可以点击pull posts中git仓库中拉取markdown文件进行发布。点击后有两个选项，一个是只pull更新，另一个是pull所有文件。如果是第一次拉取仓库内容，就使用pull all the files拉取左右的内容，后续就可以使用pull only changes只拉取变更的内容。<figure class="wp-block-image size-full">

![image-20250406164324796](https://images.iminling.com/app/hide.php?key=ZGlPT2cwSVE3VjJwS0p2Wk94ZWg2cWR4cFFpa3YrbVYzOFVjQ21rZktRREhyeFpEK3NlNkdCbnRYWXlneG9NRktaYmtJelk9)
</figure>

如果拉取有问题，也可以点击上边图片中的Logs来查看插件的日志来方便排查问题。截止到上边就已经差不多完成插件配置了。那么此时就需要我们每写完一篇文章就需要到wordpress的后台来进行手动pull一下，是不是也有点麻烦呢？接下来就讲一下如果来自动化发布。

## Webbook 配置

在git it write的配置页面可以看到webhook secret的配置选项，没错，这个就是和github的webhook进行联动的，需要我们在github中进行相关的配置。<figure class="wp-block-image size-full">

![image-20250406164919715](https://images.iminling.com/app/hide.php?key=RnVrcWhZd0RnMXVGdlA3Nm5mNHNXZDhPTkFtNUgvb3k0am1EaWdOZTdCYzdnRnNvbi81VlhUVDBuc3d0Y2FBOEJPQjVyZ0k9)
</figure>

在github仓库的设置页面，我们可以进行webbook的配置，可以添加一个webhook,具体的webhook地址可以看git it write插件页面给的提示，那里会详细说明完整的webhook地址是什么。如下图是我的完整webhook地址，当时只有地址还是不行的，我们需要配置一个密钥，这个就自己根据自己的情况来进行配置，防止被别人猜到。<figure class="wp-block-image size-full">

![image-20250406165045811](https://images.iminling.com/app/hide.php?key=V2FYb2J3Q05NL0NwTWppTmdTejN1NmNKZnZveDlXMjdzbEpObEVnVVEwbE1uckYxSEloWEplNkdrOWgyN1BYZ1JhamZPRHM9)
</figure>

github中添加webhook如下，需要填写url，content type以及secret，这里的secret和git it write中的是同一个。另外需要什么时候来触发这个webhook,我这里选择`Just the push event`.其他的大家根据需求来进行填写。<figure class="wp-block-image size-full">

![image-20250406165256944](https://images.iminling.com/app/hide.php?key=OWt4QU9weGhOSXJLL1NjTzVxQ3VRdGlPMG5WVmdFT21XdktOMzhWTFNqTGZSVnRsVWFUMklNNzhKdytQRVV6eks5M0lTQWM9)
</figure>

经过上边的配置，我们在写好markdown文件后，只需要push到git仓库中，那么github就会触发webhook通知到wordpress中的git it write插件，插件就会来仓库中拉取markdown文件，然后进行发布。做到自动化处理发布动作。

## Markdown 文件

经过上边的步骤基本已经可以正常的发布文章了，但是这里还有一个重要的问题就是发布后的文章的分类、url以及标签这些信息是怎么处理的？我们要怎么给指定呢？下边就来给大家介绍。

在markdown文件中，每个文件的顶部可以使用yaml来设置一些属性信息，这些属性信息就可以来设置我们发布文章的一些属性信息。设置的格式如下：

    ---
    
    title: "Wordpress使用git-it-write插件配合github自动发布markdown"
    date: 2025-04-06
    post_excerpt: 以前发布文章都是直接在wordpress后台手动编辑，手动上传文件，确认无误后再发布，整个过程比较耗时，并且有点麻烦。最近这两天忽然有一个想法：我直接写markdown然后，然后在wordpress中找支持markdown的编辑器插件，把我的内容直接粘贴进去，这样子不就方便很多了吗？于是就找到了WP-Githuber-MD这个插件，再探索的过程中无意又发现了git-it-write这个可以自动化的插件，于是就开始了折腾之旅行。
    post_status: publish
    comment_status: open
    taxonomy:
      category:
        - wordpress
      post_tag:
        - wordpress
        - github
        - git-it-write
        - markdown
    slug: "wordpress-use-git-it-write-and-github-publish-markdown-file"
    
    ---

是以`---`开始，然后设置一些特殊的属性。我这里介绍一下我上边使用到的一些信息。

| 属性             | 说明                                            |
| -------------- | --------------------------------------------- |
| Titile         | 发布文章的标题                                       |
| date           | 发布文章的日志                                       |
| post_excerpt   | 帖子摘录                                          |
| Post_status    | 发布文章的状态，可以选择：publish, draft, pending, future. |
| Comment_status | 评论的状态，这个默认值是closed.所以如果想要打开文章的评论功能，这个值要给open  |
| Taxonmy        | 这里设置文章的一些属性                                   |
| Category       | Taxonmy 的下级，文章的分类，这个是实际发布文章后的分类               |
| post_tag       | Taxonmy 的下级，文章的标签，也是实际发布后文章的标签                |
| slug           | 暂时也没发现有什么用处，本来以为是文章的自定义url，实际上不是。             |

我所使用到的就是上边这些属性，其他的属性暂时还没有使用到，如果有兴趣的小伙伴可以自己去看一下官方的文章，自行研究使用方法：[Getting started](https://www.aakashweb.com/docs/git-it-write/getting-started/).

**另外文章的自定义url是读取的md文件的文件名。**

另外，如果github仓库中的文章是有目录的，那么他会发布一个以文件夹命名的文章，暂时没有很好的解决这个问题，他只会发布一次，发布后改后私有的解决问题。

以上就是Wordpress使用git-it-write插件配合github自动发布markdown文章的折腾过程，希望可以帮助到大家。
