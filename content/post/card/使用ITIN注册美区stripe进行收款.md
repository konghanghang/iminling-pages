---
title: 使用ITIN注册美区stripe进行收款
author: 要名俗气
type: post
date: 2024-03-25T05:35:52+00:00
url: /2024/itin-register-stripe
description: 很久以前就拿到了ITIN，在这期间使用ITIN也成功的申请到了[Capital One(C1)的信用卡](https://www.iminling.com/2023/04/03/61.html "使用ITIN申请capital one信用卡")和[American Express(AE)的信用卡](https://www.iminling.com/2023/12/09/313.html "使用ITIN申请American Express(美国运通)信用卡")，今天就来分享一下使用ITIN申请美区的Stripe的经历。 注册 stripe的注册很简单，只需要邮箱就可以注册成功，地址：[stripe官网](https://stripe.com/zh-us)。 验证完毕登录到stripe后就被要求需要激活支付账户。
featured_image: /wp-content/uploads/2024/03/Stripe-logo.png
categories:
  - 国外信用卡
tags:
  - ITIN
  - stripe
---
很久以前就拿到了ITIN，在这期间使用ITIN也成功的申请到了[Capital One(C1)的信用卡](https://www.iminling.com/2023/04/03/61.html "使用ITIN申请capital one信用卡")和[American Express(AE)的信用卡](https://www.iminling.com/2023/12/09/313.html "使用ITIN申请American Express(美国运通)信用卡")，今天就来分享一下使用ITIN申请美区的Stripe的经历。

## 注册

stripe的注册很简单，只需要邮箱就可以注册成功，地址：[stripe官网](https://stripe.com/zh-us)。

![stripe register](https://www.iminling.com/wp-content/uploads/2024/03/D45ED1C3A578E2CEC3126F360233BCD1.png)

验证完毕登录到stripe后就被要求需要激活支付账户。

## 账户激活

如下图，要求激活账户。

![stripe](https://www.iminling.com/wp-content/uploads/2024/03/4CBA59EB23A8B7E0FE2A77DC74434332.png)

点击激活支付功能

### 商家类型

地区美国，类型个人。

![stripe type](https://www.iminling.com/wp-content/uploads/2024/03/874A457B9334BEB3529853E5213E3094.png)

### 个人详情

法定名称就是自己的姓名，第一行名，第二行姓。

家庭地址我这里填写的申请ITIN时的地址。

电话号码填写的是GV号

社会保障码后四位：填写ITIN后四位

![stripe details](https://www.iminling.com/wp-content/uploads/2024/03/9B5937F9D9A8B85312C5B656C8DB8C51.png)

### 商家详情

纳税人识别码：填ITIN

商家地址：和上边个人详情的一样

网站：自己的网站

产品描述：介绍一下为何收款，我这里填的是英文的。

![stripe merchant info](https://www.iminling.com/wp-content/uploads/2024/03/CEFAFACE092C6DAC31CB971BF1A47D85.png)

### 公开详情

添加一些描述以及客服地址等信息

![stripe info](https://www.iminling.com/wp-content/uploads/2024/03/62D4FB59A8FA0FB2E4D413B16AA5BF24.png)

### 银行详情

我这里使用的是revolut的美元账户。

路径号码：ACH号码

账户：银行账号

![stripe card](https://www.iminling.com/wp-content/uploads/2024/03/C0EC1E2E9DCC8C2143F047BE97DEFEAA.png)

### 两步验证

出于安全，添加2FA

![stripe 2fa](https://www.iminling.com/wp-content/uploads/2024/03/D75D2A86735D93C12762FA890D2EAF83.png)

### 税款计算

默认

![stripe tax](https://www.iminling.com/wp-content/uploads/2024/03/B2CBD8A3E78E266925AD98EEEFAE7910.png)

### 捐款

拒绝白嫖

![stripe climate](https://www.iminling.com/wp-content/uploads/2024/03/A9079CF271069860CE08CF2C4D298ECC.png)

### 摘要

摘要信息

![stripe summary](https://www.iminling.com/wp-content/uploads/2024/03/69D9ED9C354E51E5DE2DD8BBC21A3FC9.png)

提交等代审核就可以了。

## 设置账户

进入stripe的首页，提示账户已激活。

![stripe index](https://www.iminling.com/wp-content/uploads/2024/03/94CB390C94812A5532978547F3B707BC.png)

但是当我们去创建收款的时候，发现还是不能使用。![stripe bill](https://www.iminling.com/wp-content/uploads/2024/03/40DD6EDA81CD677C4013750F38CBD570.png)

点击添加信息，发现还需要再次输入ITIN，提交。

![stripe itin](https://www.iminling.com/wp-content/uploads/2024/03/CEA3B0FDA1CF45F56B446A16AFC780C4.png)

### 手机号验证

账户详情处进行手机号验证

![stripe phoen verify](https://www.iminling.com/wp-content/uploads/2024/03/36E51A45881B78A2C34E817AAEC799E3.png)

经过这些处理后基本已经可以正常使用了。

可以在账户状态看是否还需要补充什么信息，如下图就说明完全不需要补充资料了。

![stripe](https://www.iminling.com/wp-content/uploads/2024/03/171D19D477A7CB0496AA8C5A7D389559.png)

账号注册完毕，可以开心收款了。
