---
title: Mac终端美化之Oh My Zsh安装配置
author: 要名俗气
type: post
date: 2023-10-27T10:07:07+00:00
url: /2023/mac-install-oh-my-zsh
description: 使用Mac的同学应该对Mac自带的终端配置都没什么好感，默认白底黑字，文件夹也没有特殊标识，还是上一张图来感受一下吧： 黑呼呼的一团，看着实在是难受，下边就让我们使用Oh My Zsh来进行美化一下。
image: https://images.iminling.com/app/hide.php?key=U1V5NUZ5WlYrTGt1dElzaVc2VkgrOENSZkRib09zbVk0RjVKTFhkeXNyQkJvdmdOeVhYZWM1aHR1NjdDTWNQU2h3Y1ZHVTQ9
categories:
  - Mac
tags:
  - Mac
  - Oh My Zsh
---
使用Mac的同学应该对Mac自带的终端配置都没什么好感，默认白底黑字，文件夹也没有特殊标识，还是上一张图来感受一下吧：

![](https://images.iminling.com/app/hide.php?key=RjFDWXBkVGJlbVdYdkpqZTM0QmxzYzRIZWt3enBnaDltSEJyZUwvajIzUDJwRU14Z05hOTcraUNLMHViKzdMVEVhZTJhSUk9)

黑呼呼的一团，看着实在是难受，下边就让我们使用Oh My Zsh来进行美化一下。

## Oh My Zsh安装

oh my zsh的安装是非常简单的，大家可以参考官网来进行安装：[Oh My Zsh Install](https://ohmyz.sh/#install),通过curl或wget都可以安装，下边使用curl进行安装：

```bash
k@MacBook-Pro ~ % sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
Cloning Oh My Zsh...
remote: Enumerating objects: 1370, done.
remote: Counting objects: 100% (1370/1370), done.
remote: Compressing objects: 100% (1318/1318), done.
remote: Total 1370 (delta 32), reused 1155 (delta 28), pack-reused 0
Receiving objects: 100% (1370/1370), 2.01 MiB | 2.66 MiB/s, done.
Resolving deltas: 100% (32/32), done.
From https://github.com/ohmyzsh/ohmyzsh
 * [new branch]      master     -> origin/master
branch 'master' set up to track 'origin/master'.
Switched to a new branch 'master'
/Users/k

Looking for an existing zsh config...
Using the Oh My Zsh template file and adding it to /Users/k/.zshrc.

         __                                     __
  ____  / /_     ____ ___  __  __   ____  _____/ /_
 / __ \/ __ \   / __ `__ \/ / / /  /_  / / ___/ __ \
/ /_/ / / / /  / / / / / / /_/ /    / /_(__  ) / / /
\____/_/ /_/  /_/ /_/ /_/\__, /    /___/____/_/ /_/
                        /____/                       ....is now installed!

Before you scream Oh My Zsh! look over the `.zshrc` file to select plugins, themes, and options.

• Follow us on Twitter: https://twitter.com/ohmyzsh
• Join our Discord community: https://discord.gg/ohmyzsh
• Get stickers, t-shirts, coffee mugs and more: https://shop.planetargon.com/collections/oh-my-zsh
```

上边就安装成功了oh my zsh。

## 切换主题

安装成功后，默认的主题(<span class="s1">robbyrussell</span>)如下所示：

![](https://images.iminling.com/app/hide.php?key=NFZFUVpyNmtTaUY3TnhFd2o1MURwbUxWQTdJMVZwVFJuYUMxb3VyVlZHZjBGcWl5SWhHYnI1eXBTUXZxd1hYMHNIWXBaNEU9)

文件夹也可以高亮了，看着还是不错的。剩下的就是修改自己喜欢的主题了。oh my zsh自带了一些主题，存放在~/.oh-my-sh/themes下：

```bash
# k @ MacBook-Pro in ~/.oh-my-zsh/themes on git:master o [17:47:16]
$ ls
3den.zsh-theme                 kennethreitz.zsh-theme
Soliah.zsh-theme               kiwi.zsh-theme
adben.zsh-theme                kolo.zsh-theme
af-magic.zsh-theme             kphoen.zsh-theme
afowler.zsh-theme              lambda.zsh-theme
agnoster.zsh-theme             linuxonly.zsh-theme
alanpeabody.zsh-theme          lukerandall.zsh-theme
amuse.zsh-theme                macovsky-ruby.zsh-theme
apple.zsh-theme                macovsky.zsh-theme
arrow.zsh-theme                maran.zsh-theme
aussiegeek.zsh-theme           mgutz.zsh-theme
avit.zsh-theme                 mh.zsh-theme
awesomepanda.zsh-theme         michelebologna.zsh-theme
bira.zsh-theme                 mikeh.zsh-theme
blinks.zsh-theme               miloshadzic.zsh-theme
bureau.zsh-theme               minimal.zsh-theme
candy-kingdom.zsh-theme        mira.zsh-theme
candy.zsh-theme                mlh.zsh-theme
clean.zsh-theme                mortalscumbag.zsh-theme
cloud.zsh-theme                mrtazz.zsh-theme
crcandy.zsh-theme              murilasso.zsh-theme
crunch.zsh-theme               muse.zsh-theme
cypher.zsh-theme               nanotech.zsh-theme
dallas.zsh-theme               nebirhos.zsh-theme
darkblood.zsh-theme            nicoulaj.zsh-theme
daveverwer.zsh-theme           norm.zsh-theme
dieter.zsh-theme               obraun.zsh-theme
dogenpunk.zsh-theme            oldgallois.zsh-theme
dpoggi.zsh-theme               peepcode.zsh-theme
dst.zsh-theme                  philips.zsh-theme
dstufft.zsh-theme              pmcgee.zsh-theme
duellj.zsh-theme               pygmalion-virtualenv.zsh-theme
eastwood.zsh-theme             pygmalion.zsh-theme
edvardm.zsh-theme              random.zsh-theme
emotty.zsh-theme               re5et.zsh-theme
essembeh.zsh-theme             refined.zsh-theme
evan.zsh-theme                 rgm.zsh-theme
fino-time.zsh-theme            risto.zsh-theme
fino.zsh-theme                 rixius.zsh-theme
fishy.zsh-theme                rkj-repos.zsh-theme
flazz.zsh-theme                rkj.zsh-theme
fletcherm.zsh-theme            robbyrussell.zsh-theme
fox.zsh-theme                  sammy.zsh-theme
frisk.zsh-theme                simonoff.zsh-theme
frontcube.zsh-theme            simple.zsh-theme
funky.zsh-theme                skaro.zsh-theme
fwalch.zsh-theme               smt.zsh-theme
gallifrey.zsh-theme            sonicradish.zsh-theme
gallois.zsh-theme              sorin.zsh-theme
garyblessington.zsh-theme      sporty_256.zsh-theme
gentoo.zsh-theme               steeef.zsh-theme
geoffgarside.zsh-theme         strug.zsh-theme
gianu.zsh-theme                sunaku.zsh-theme
gnzh.zsh-theme                 sunrise.zsh-theme
gozilla.zsh-theme              superjarin.zsh-theme
half-life.zsh-theme            suvash.zsh-theme
humza.zsh-theme                takashiyoshida.zsh-theme
imajes.zsh-theme               terminalparty.zsh-theme
intheloop.zsh-theme            theunraveler.zsh-theme
itchy.zsh-theme                tjkirch.zsh-theme
jaischeema.zsh-theme           tjkirch_mod.zsh-theme
jbergantine.zsh-theme          tonotdo.zsh-theme
jispwoso.zsh-theme             trapd00r.zsh-theme
jnrowe.zsh-theme               wedisagree.zsh-theme
jonathan.zsh-theme             wezm+.zsh-theme
josh.zsh-theme                 wezm.zsh-theme
jreese.zsh-theme               wuffers.zsh-theme
jtriley.zsh-theme              xiong-chiamiov-plus.zsh-theme
juanghurtado.zsh-theme         xiong-chiamiov.zsh-theme
junkfood.zsh-theme             ys.zsh-theme
kafeitu.zsh-theme              zhann.zsh-theme
kardan.zsh-theme
```

比如默认的主题：`robbyrussell.zsh-theme`，根据自己的需求，在.zshrc文件中进行修改，

```bash
# Set name of the theme to load --- if set to "random", it will
# load a random theme each time oh-my-zsh is loaded, in which case,
# to know which specific one was loaded, run: echo $RANDOM_THEME
# See https://github.com/ohmyzsh/ohmyzsh/wiki/Themes
#ZSH_THEME="robbyrussell"
ZSH_THEME="ys"
```

## 终端配色

我这里将主题修改为了ys。下边展示一下我修改后的终端配色：

![](https://images.iminling.com/app/hide.php?key=U1V5NUZ5WlYrTGt1dElzaVc2VkgrOENSZkRib09zbVk0RjVKTFhkeXNyQkJvdmdOeVhYZWM1aHR1NjdDTWNQU2h3Y1ZHVTQ9)

加了一些透明在里边，下边把我自己使用的终端配色分享给大家，大家如果喜欢可以直接下载后进行导入：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>BackgroundAlphaInactive</key>
	<real>0.29155388779527558</real>
	<key>BackgroundBlur</key>
	<real>0.197831572862122</real>
	<key>BackgroundBlurInactive</key>
	<real>0.15034141538180704</real>
	<key>BackgroundColor</key>
	<data>
	YnBsaXN0MDDUAQIDBAUGKyxYJHZlcnNpb25YJG9iamVjdHNZJGFyY2hpdmVyVCR0b3AS
	AAGGoKcHCBMZHSQoVSRudWxs1QkKCwwNDg8QERJcTlNDb21wb25lbnRzVU5TUkdCXE5T
	Q29sb3JTcGFjZV8QEk5TQ3VzdG9tQ29sb3JTcGFjZVYkY2xhc3NPECowLjQwMjQ2MTQ0
	OTcgMC40MDExMDYzNjA2IDAuNDAzODE2NTM4OCAwLjVPECowLjMyODQwNDM2NyAwLjMy
	Njc0MDM4NDEgMC4zMjk2NjQyMzAzIDAuNQAQAYACgAbTFA0VFhcYVU5TSUNDWU5TU3Bh
	Y2VJRIADgAUQDNIaDRscV05TLmRhdGFPEQIkAAACJGFwcGwEAAAAbW50clJHQiBYWVog
	B+EABwAHAA0AFgAgYWNzcEFQUEwAAAAAQVBQTAAAAAAAAAAAAAAAAAAAAAAAAPbWAAEA
	AAAA0y1hcHBsyhqVgiV/EE04mRPV0eoVggAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAKZGVzYwAAAPwAAABlY3BydAAAAWQAAAAjd3RwdAAAAYgAAAAUclhZWgAAAZwA
	AAAUZ1hZWgAAAbAAAAAUYlhZWgAAAcQAAAAUclRSQwAAAdgAAAAgY2hhZAAAAfgAAAAs
	YlRSQwAAAdgAAAAgZ1RSQwAAAdgAAAAgZGVzYwAAAAAAAAALRGlzcGxheSBQMwAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAB0ZXh0AAAAAENvcHlyaWdodCBBcHBsZSBJ
	bmMuLCAyMDE3AABYWVogAAAAAAAA81EAAQAAAAEWzFhZWiAAAAAAAACD3wAAPb////+7
	WFlaIAAAAAAAAEq/AACxNwAACrlYWVogAAAAAAAAKDgAABELAADIuXBhcmEAAAAAAAMA
	AAACZmYAAPKnAAANWQAAE9AAAApbc2YzMgAAAAAAAQxCAAAF3v//8yYAAAeTAAD9kP//
	+6L///2jAAAD3AAAwG6ABNIeHyAhWiRjbGFzc25hbWVYJGNsYXNzZXNdTlNNdXRhYmxl
	RGF0YaMgIiNWTlNEYXRhWE5TT2JqZWN00h4fJSZcTlNDb2xvclNwYWNloicjXE5TQ29s
	b3JTcGFjZdIeHykqV05TQ29sb3KiKSNfEA9OU0tleWVkQXJjaGl2ZXLRLS5Ucm9vdIAB
	AAgAEQAaACMALQAyADcAPwBFAFAAXQBjAHAAhQCMALkA5gDoAOoA7ADzAPkBAwEFAQcB
	CQEOARYDPgNAA0UDUANZA2cDawNyA3sDgAONA5ADnQOiA6oDrQO/A8IDxwAAAAAAAAIB
	AAAAAAAAAC8AAAAAAAAAAAAAAAAAAAPJ
	</data>
	<key>BackgroundSettingsForInactiveWindows</key>
	<true/>
	<key>CursorColor</key>
	<data>
	YnBsaXN0MDDUAQIDBAUGKyxYJHZlcnNpb25YJG9iamVjdHNZJGFyY2hpdmVyVCR0b3AS
	AAGGoKcHCBMZHSQoVSRudWxs1QkKCwwNDg8QERJcTlNDb21wb25lbnRzVU5TUkdCXE5T
	Q29sb3JTcGFjZV8QEk5TQ3VzdG9tQ29sb3JTcGFjZVYkY2xhc3NPECswLjA5MDY1NzM4
	MzIxIDAuMDkwMzY5NTgxOTkgMC4wOTA5NDUxODQ0MyAxTxAqMC4wNzEzODQ3ODc1NiAw
	LjA3MTE0Nzg2NjU1IDAuMDcxNTY2Mzg4MDEAEAGAAoAG0xQNFRYXGFVOU0lDQ1lOU1Nw
	YWNlSUSAA4AFEAzSGg0bHFdOUy5kYXRhTxECJAAAAiRhcHBsBAAAAG1udHJSR0IgWFla
	IAfhAAcABwANABYAIGFjc3BBUFBMAAAAAEFQUEwAAAAAAAAAAAAAAAAAAAAAAAD21gAB
	AAAAANMtYXBwbMoalYIlfxBNOJkT1dHqFYIAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAACmRlc2MAAAD8AAAAZWNwcnQAAAFkAAAAI3d0cHQAAAGIAAAAFHJYWVoAAAGc
	AAAAFGdYWVoAAAGwAAAAFGJYWVoAAAHEAAAAFHJUUkMAAAHYAAAAIGNoYWQAAAH4AAAA
	LGJUUkMAAAHYAAAAIGdUUkMAAAHYAAAAIGRlc2MAAAAAAAAAC0Rpc3BsYXkgUDMAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAdGV4dAAAAABDb3B5cmlnaHQgQXBwbGUg
	SW5jLiwgMjAxNwAAWFlaIAAAAAAAAPNRAAEAAAABFsxYWVogAAAAAAAAg98AAD2/////
	u1hZWiAAAAAAAABKvwAAsTcAAAq5WFlaIAAAAAAAACg4AAARCwAAyLlwYXJhAAAAAAAD
	AAAAAmZmAADypwAADVkAABPQAAAKW3NmMzIAAAAAAAEMQgAABd7///MmAAAHkwAA/ZD/
	//ui///9owAAA9wAAMBugATSHh8gIVokY2xhc3NuYW1lWCRjbGFzc2VzXU5TTXV0YWJs
	ZURhdGGjICIjVk5TRGF0YVhOU09iamVjdNIeHyUmXE5TQ29sb3JTcGFjZaInI1xOU0Nv
	bG9yU3BhY2XSHh8pKldOU0NvbG9yoikjXxAPTlNLZXllZEFyY2hpdmVy0S0uVHJvb3SA
	AQAIABEAGgAjAC0AMgA3AD8ARQBQAF0AYwBwAIUAjAC6AOcA6QDrAO0A9AD6AQQBBgEI
	AQoBDwEXAz8DQQNGA1EDWgNoA2wDcwN8A4EDjgORA54DowOrA64DwAPDA8gAAAAAAAAC
	AQAAAAAAAAAvAAAAAAAAAAAAAAAAAAADyg==
	</data>
	<key>Font</key>
	<data>
	YnBsaXN0MDDUAQIDBAUGGBlYJHZlcnNpb25YJG9iamVjdHNZJGFyY2hpdmVyVCR0b3AS
	AAGGoKQHCBESVSRudWxs1AkKCwwNDg8QVk5TU2l6ZVhOU2ZGbGFnc1ZOU05hbWVWJGNs
	YXNzI0AqAAAAAAAAEBCAAoADXxAcTWVzbG9MR01Gb3JQb3dlcmxpbmUtUmVndWxhctIT
	FBUWWiRjbGFzc25hbWVYJGNsYXNzZXNWTlNGb250ohUXWE5TT2JqZWN0XxAPTlNLZXll
	ZEFyY2hpdmVy0RobVHJvb3SAAQgRGiMtMjc8QktSW2JpcnR2eJecp7C3usPV2N0AAAAA
	AAABAQAAAAAAAAAcAAAAAAAAAAAAAAAAAAAA3w==
	</data>
	<key>FontAntialias</key>
	<true/>
	<key>FontWidthSpacing</key>
	<integer>1</integer>
	<key>ProfileCurrentVersion</key>
	<real>2.0499999999999998</real>
	<key>SelectionColor</key>
	<data>
	YnBsaXN0MDDUAQIDBAUGKyxYJHZlcnNpb25YJG9iamVjdHNZJGFyY2hpdmVyVCR0b3AS
	AAGGoKcHCBMZHSQoVSRudWxs1QkKCwwNDg8QERJcTlNDb21wb25lbnRzVU5TUkdCXE5T
	Q29sb3JTcGFjZV8QEk5TQ3VzdG9tQ29sb3JTcGFjZVYkY2xhc3NPECcwLjYyOTkyNTMz
	OSAwLjY2NTY2MzQyMjEgMC42MzA0MjI5NTI5IDFPECcwLjU1NDMwNjk4MzkgMC42MDU2
	NzE1ODQ2IDAuNTU5NTI2NjIyMwAQAYACgAbTFA0VFhcYVU5TSUNDWU5TU3BhY2VJRIAD
	gAUQDNIaDRscV05TLmRhdGFPEQIkAAACJGFwcGwEAAAAbW50clJHQiBYWVogB+EABwAH
	AA0AFgAgYWNzcEFQUEwAAAAAQVBQTAAAAAAAAAAAAAAAAAAAAAAAAPbWAAEAAAAA0y1h
	cHBsyhqVgiV/EE04mRPV0eoVggAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAK
	ZGVzYwAAAPwAAABlY3BydAAAAWQAAAAjd3RwdAAAAYgAAAAUclhZWgAAAZwAAAAUZ1hZ
	WgAAAbAAAAAUYlhZWgAAAcQAAAAUclRSQwAAAdgAAAAgY2hhZAAAAfgAAAAsYlRSQwAA
	AdgAAAAgZ1RSQwAAAdgAAAAgZGVzYwAAAAAAAAALRGlzcGxheSBQMwAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAB0ZXh0AAAAAENvcHlyaWdodCBBcHBsZSBJbmMuLCAy
	MDE3AABYWVogAAAAAAAA81EAAQAAAAEWzFhZWiAAAAAAAACD3wAAPb////+7WFlaIAAA
	AAAAAEq/AACxNwAACrlYWVogAAAAAAAAKDgAABELAADIuXBhcmEAAAAAAAMAAAACZmYA
	APKnAAANWQAAE9AAAApbc2YzMgAAAAAAAQxCAAAF3v//8yYAAAeTAAD9kP//+6L///2j
	AAAD3AAAwG6ABNIeHyAhWiRjbGFzc25hbWVYJGNsYXNzZXNdTlNNdXRhYmxlRGF0YaMg
	IiNWTlNEYXRhWE5TT2JqZWN00h4fJSZcTlNDb2xvclNwYWNloicjXE5TQ29sb3JTcGFj
	ZdIeHykqV05TQ29sb3KiKSNfEA9OU0tleWVkQXJjaGl2ZXLRLS5Ucm9vdIABAAgAEQAa
	ACMALQAyADcAPwBFAFAAXQBjAHAAhQCMALYA4ADiAOQA5gDtAPMA/QD/AQEBAwEIARAD
	OAM6Az8DSgNTA2EDZQNsA3UDegOHA4oDlwOcA6QDpwO5A7wDwQAAAAAAAAIBAAAAAAAA
	AC8AAAAAAAAAAAAAAAAAAAPD
	</data>
	<key>TextColor</key>
	<data>
	YnBsaXN0MDDUAQIDBAUGKyxYJHZlcnNpb25YJG9iamVjdHNZJGFyY2hpdmVyVCR0b3AS
	AAGGoKcHCBMZHSQoVSRudWxs1QkKCwwNDg8QERJcTlNDb21wb25lbnRzVU5TUkdCXE5T
	Q29sb3JTcGFjZV8QEk5TQ3VzdG9tQ29sb3JTcGFjZVYkY2xhc3NHMSAxIDEgMU8QHDAu
	OTk5ODg2MDk1NSAxIDAuOTk5ODM5ODQyMwAQAYACgAbTFA0VFhcYVU5TSUNDWU5TU3Bh
	Y2VJRIADgAUQDNIaDRscV05TLmRhdGFPEQIkAAACJGFwcGwEAAAAbW50clJHQiBYWVog
	B+EABwAHAA0AFgAgYWNzcEFQUEwAAAAAQVBQTAAAAAAAAAAAAAAAAAAAAAAAAPbWAAEA
	AAAA0y1hcHBsyhqVgiV/EE04mRPV0eoVggAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAKZGVzYwAAAPwAAABlY3BydAAAAWQAAAAjd3RwdAAAAYgAAAAUclhZWgAAAZwA
	AAAUZ1hZWgAAAbAAAAAUYlhZWgAAAcQAAAAUclRSQwAAAdgAAAAgY2hhZAAAAfgAAAAs
	YlRSQwAAAdgAAAAgZ1RSQwAAAdgAAAAgZGVzYwAAAAAAAAALRGlzcGxheSBQMwAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
	AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAB0ZXh0AAAAAENvcHlyaWdodCBBcHBsZSBJ
	bmMuLCAyMDE3AABYWVogAAAAAAAA81EAAQAAAAEWzFhZWiAAAAAAAACD3wAAPb////+7
	WFlaIAAAAAAAAEq/AACxNwAACrlYWVogAAAAAAAAKDgAABELAADIuXBhcmEAAAAAAAMA
	AAACZmYAAPKnAAANWQAAE9AAAApbc2YzMgAAAAAAAQxCAAAF3v//8yYAAAeTAAD9kP//
	+6L///2jAAAD3AAAwG6ABNIeHyAhWiRjbGFzc25hbWVYJGNsYXNzZXNdTlNNdXRhYmxl
	RGF0YaMgIiNWTlNEYXRhWE5TT2JqZWN00h4fJSZcTlNDb2xvclNwYWNloicjXE5TQ29s
	b3JTcGFjZdIeHykqV05TQ29sb3KiKSNfEA9OU0tleWVkQXJjaGl2ZXLRLS5Ucm9vdIAB
	AAgAEQAaACMALQAyADcAPwBFAFAAXQBjAHAAhQCMAJQAswC1ALcAuQDAAMYA0ADSANQA
	1gDbAOMDCwMNAxIDHQMmAzQDOAM/A0gDTQNaA10DagNvA3cDegOMA48DlAAAAAAAAAIB
	AAAAAAAAAC8AAAAAAAAAAAAAAAAAAAOW
	</data>
	<key>UseBoldFonts</key>
	<false/>
	<key>columnCount</key>
	<integer>90</integer>
	<key>name</key>
	<string>me</string>
	<key>rowCount</key>
	<integer>25</integer>
	<key>type</key>
	<string>Window Settings</string>
</dict>
</plist>
```

新建一个文件，名字随意，文件格式为<span class="s1">.terminal,例如ohmyzsh.</span><span class="s1">terminal。然后在终端中进行导入并设置为默认：</span>

![](https://images.iminling.com/app/hide.php?key=TG1mVTNGNDVpMjF6TDVyc20zRkt5Yk5FWWErVFFzWW0rQUhHaXdsdjZHbzAzR3JSR1MzbkJQc2djSmZoZWdDbzJEdzU5cmM9)

选择import后，在上边的选项中就会有你刚才文件名称对应的一个配置文件，比如我这里叫me-old,然后设置为default就可以了，就和我图片中的终端配色一样了。
