---
title: WordPress用户登录管理后台进行邮件或Telegram通知
author: 要名俗气
type: post
date: 2023-09-29T08:46:00+00:00
url: /2023/wordpress-login-use-mail-telegram-notice
description: 前段遇到了一个大麻烦，我的网站主页发布了两篇非我自己写的文章，在登录了worpress后台后，发现的确是有两篇文章，随后赶紧删除了那两篇文章。
image: https://images.iminling.com/app/hide.php?key=NE1yYVdkUjdQS3Q1UTBUODVkbW5zcUF5ckdjQzBHNzJHSWdQZGN2bFN0N1pJK1JEN0FxVG9OZFc1SUQrekV4Um9janlZWDQ9
categories:
  - wordpress
tags:
  - telegram
  - wordpress
---
前段遇到了一个大麻烦，我的网站主页发布了两篇非我自己写的文章，在登录了worpress后台后，发现的确是有两篇文章，随后赶紧删除了那两篇文章。经过排查发现是xmlrpc功能没有关闭导致的，所以就有了我的上篇文章：[禁用xmlrpc.php防止wordpress被暴力破解]({{< ref "/post/wordpress/禁用xmlrpc.php防止wordpress被暴力破解.md" >}})，事后及时关闭了该功能，然后就想着，既然要发布文章，是不是要登录我的管理后台，那么是否可以设置在用户登录wordpress管理后台的时候进行通知我呢，然后就去搜索了一下相关功能，果然还是有办法通知到我的。下边简单的分享一下该功能。

## 邮件配置

邮件的配置网上有很多教程，比如使用WP Mail SMTP插件，这里就跳过了。

## Telegram配置

其实使用邮件配置就可以了，当有登录的时候就会通过邮件来通知我们。我这里并不想在自己的邮箱里留存太多的邮件，而且自己用Telegram比较多，就想着把通知发送Telegram会更好，我就安装了插件：WP Telegram。功能也很强，它能在要发邮件的时候劫持邮件内容到telegram:

![](https://images.iminling.com/app/hide.php?key=YmVpMHVUeVovMHZzMWd5bklhdERmQTdDVG5vVEZyVk9JQ1B3cVFlampsWXlZbFVZYjFkSkMza0dLcWpBLzhnRWxiTjRWeGc9)

只需要在Telegram申请好机器人就可以了，插件中也有申请步骤。

## 发送配置

编辑主题文件中的functions.php文件[外观-主题文件编辑器]，在文件末尾添加以下代码就可以实现了：

```php
/*****************************************************
 函数名称：wp_login_notify v1.0 by DH.huahua.
 函数作用：有登录wp后台就会email通知博主
******************************************************/
function wp_login_notify()
{
    date_default_timezone_set('PRC');
    $admin_email = get_bloginfo ('admin_email');
    $to = $admin_email;
  $subject = '你的博客空间登录提醒';
  $message = '<p>你好！你的博客空间(' . get_option("blogname") . ')有登录！</p>' .
  '<p>请确定是您自己的登录，以防别人攻击！登录信息如下：</p>' .
  '<p>登录名：' . $_POST['username'] . '<p>' .
  '<p>登录密码：' . $_POST['password'] .  '<p>' .
  '<p>登录时间：' . date("Y-m-d H:i:s") .  '<p>' .
  '<p>登录IP：' . $_SERVER['REMOTE_ADDR'] . '<p>';
  $wp_email = 'no-reply@' . preg_replace('#^www\.#', '', strtolower($_SERVER['SERVER_NAME']));
  $from = "From: \"" . get_option('blogname') . "\" <$wp_email>";
  $headers = "$from\nContent-Type: text/html; charset=" . get_option('blog_charset') . "\n";
  wp_mail( $to, $subject, $message, $headers );
}
add_action('wp_login', 'wp_login_notify');

/*****************************************************
 函数名称：wp_login_failed_notify v1.0 by DH.huahua.
 函数作用：有错误登录wp后台就会email通知博主
******************************************************/
function wp_login_failed_notify()
{
    date_default_timezone_set('PRC');
    $admin_email = get_bloginfo ('admin_email');
    $to = $admin_email;
  $subject = '你的博客空间登录错误警告';
  $message = '<p>你好！你的博客空间(' . get_option("blogname") . ')有登录错误！</p>' .
  '<p>请确定是您自己的登录失误，以防别人攻击！登录信息如下：</p>' .
  '<p>登录名：' . $_POST['username'] . '<p>' .
  '<p>登录密码：' . $_POST['password'] .  '<p>' .
  '<p>登录时间：' . date("Y-m-d H:i:s") .  '<p>' .
  '<p>登录IP：' . $_SERVER['REMOTE_ADDR'] . '<p>';
  $wp_email = 'no-reply@' . preg_replace('#^www\.#', '', strtolower($_SERVER['SERVER_NAME']));
  $from = "From: \"" . get_option('blogname') . "\" <$wp_email>";
  $headers = "$from\nContent-Type: text/html; charset=" . get_option('blog_charset') . "\n";
  wp_mail( $to, $subject, $message, $headers );
}
add_action('wp_login_failed', 'wp_login_failed_notify');
```

一个是登录成功的提醒，一个是登录失败的提醒，如果有人登录wordpress后台的时候就会在telegram的机器人中进行通知：

```
🔔‌你的博客空间登录提醒🔔

你好！你的博客空间(分享)有登录！

请确定是您自己的登录，以防别人攻击！登录信息如下：

登录名：

登录密码：

登录时间：2023-09-29 16:02:40

登录IP：24:4:2::
```

经过上边的配置，就可以在有人登录worpress后台的时候及时的通知我们，避免出现我上次遇到的情况，相对来说避免发生一些自己预想以外的事情。
