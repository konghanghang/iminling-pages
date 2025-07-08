---
title: Mac更换源更快速的安装Homebrew
author: 要名俗气
type: post
date: 2023-10-02
url: /2023/mac-change-source-install-homebrew
description: Homebrew是一款Mac OS平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。简单的一条指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷。但是安装却是需要去github去拉仓库然后进行安装，github在国内的访问速度堪忧，所以会导致整个安装过程很长，甚至安装失败，这里我记录一下我替换源进行安装的过程，希望可以帮助大家。
image: https://images.iminling.com/app/hide.php?key=NmtEZ1ZGOE1SbXJlVENkbTFYQTBQcko2NVc4UHBtT044clNuYUZpQStJZHpieGVORmFLbFhTVnBRczhwNWpyRTlKL3N5TWM9
categories:
  - Mac
tags:
  - Homebrew
  - Mac
---
[Homebrew](https://brew.sh/)是一款Mac OS平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。简单的一条指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷。但是安装却是需要去github去拉仓库然后进行安装，github在国内的访问速度堪忧，所以会导致整个安装过程很长，甚至安装失败，这里我记录一下我替换源进行安装的过程，希望可以帮助大家。

## 安装准备

[Homebrew](https://brew.sh/)的官网提供了一键安装的脚本，脚本的实际指向是github仓库中的[install](https://github.com/Homebrew/install)仓库，查看安装脚本`install.sh`,可以发现安装的过程会优先读环境变量中的git仓库地址，如果没有则使用脚本中默认的github中的仓库地址，所以我们可以把要使用的源设置好环境变量，在install.sh脚本安装的时候直接使用我们设置好的源来加快安装速度：

```bash
HOMEBREW_BREW_DEFAULT_GIT_REMOTE="https://github.com/Homebrew/brew"
HOMEBREW_CORE_DEFAULT_GIT_REMOTE="https://github.com/Homebrew/homebrew-core"

# Use remote URLs of Homebrew repositories from environment if set.
HOMEBREW_BREW_GIT_REMOTE="${HOMEBREW_BREW_GIT_REMOTE:-"${HOMEBREW_BREW_DEFAULT_GIT_REMOTE}"}"
HOMEBREW_CORE_GIT_REMOTE="${HOMEBREW_CORE_GIT_REMOTE:-"${HOMEBREW_CORE_DEFAULT_GIT_REMOTE}"}"
# The URLs with and without the '.git' suffix are the same Git remote. Do not prompt.
if [[ "${HOMEBREW_BREW_GIT_REMOTE}" == "${HOMEBREW_BREW_DEFAULT_GIT_REMOTE}.git" ]]
then
  HOMEBREW_BREW_GIT_REMOTE="${HOMEBREW_BREW_DEFAULT_GIT_REMOTE}"
fi
if [[ "${HOMEBREW_CORE_GIT_REMOTE}" == "${HOMEBREW_CORE_DEFAULT_GIT_REMOTE}.git" ]]
then
  HOMEBREW_CORE_GIT_REMOTE="${HOMEBREW_CORE_DEFAULT_GIT_REMOTE}"
fi
export HOMEBREW_{BREW,CORE}_GIT_REMOTE
```

所以我们需要设置两个环境变量`HOMEBREW_BREW_GIT_REMOTE`和`HOMEBREW_CORE_GIT_REMOTE`

，我这里使用的是中科大的源来进行安装，具体可以参考中科的的介绍页面：[Homebrew 源使用帮助](https://mirrors.ustc.edu.cn/help/brew.git.html)。两个变量的设置脚本如下：

```bash
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.ustc.edu.cn/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.ustc.edu.cn/homebrew-core.git"
```

这样就完成了源的替换，不要关闭执行上边export命令的窗口，下一步在同一个窗口执行安装脚本。

## 安装

执行官网的一键安装脚本：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

由于替换了中科大的源，安装上不会很慢，等待程序执行完成就可以了。

安装完成后把brew添加到环境变量，编辑.bash_profile文件，添加下边一行代码：

```
export PATH=/opt/homebrew/bin:$PATH
```

保存后刷新环境，使配置的变量生效：

```
source .bash_profile
```

然后就可以正常使用brew来安装和管理应用了。

## 使用

下边列举一些常用的brew的使用命令：

```bash
brew search **  //查找某个软件包
brew list  //列出已经安装的软件的包
brew install ** //安装某个软件包,默认安装的是稳定版本
brew uninstall **//卸载某个软件的包
brew upgrade ** //更新某个软件包
brew info ** //查看指定软件包的说明
brew cache clean //清理缓存
```

以上就是homebrew简单快速的安装过程。
