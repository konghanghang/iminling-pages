---
title: 使用ITIN注册美区stripe进行收款
author: 要名俗气
type: post
date: 2024-03-25T05:35:52+00:00
url: /2024/itin-register-stripe
description: 很久以前就拿到了ITIN，在这期间使用ITIN也成功的申请到了Capital One和American Express的信用卡，今天就来分享一下使用ITIN申请美区的Stripe的经历。 
image: https://images.iminling.com/app/hide.php?key=UC9DejIvQjdPSGFkV2JMSGptSlBBOVhWS2dYR0Vpd0c5aHN1RSs4dE1BWko2bWNUcklXMDROejVFQXJCSkRQTklpcE0wVjA9
categories:
  - card
tags:
  - ITIN
  - stripe
---
很久以前就拿到了ITIN，在这期间使用ITIN也成功的申请到了[Capital One(C1)的信用卡]({{< ref "/post/card/使用ITIN申请capital one信用卡.md" >}}) 和 [American Express(AE)的信用卡]({{< ref "/post/card/使用ITIN申请American Express(美国运通)信用卡" >}})，今天就来分享一下使用ITIN申请美区的Stripe的经历。

## 注册

stripe的注册很简单，只需要邮箱就可以注册成功，地址：[stripe官网](https://stripe.com/zh-us)。

![stripe register](https://images.iminling.com/app/hide.php?key=Q0M5M1M0THNKQjJWaE9aUEljSFdwMlpuNkdqNWZ2c09KTG9GVjVXZmI2Wmxpb0ZUTEN4dm1EMlhOWFBVL0lvRjVDNUFNSEE9)

验证完毕登录到stripe后就被要求需要激活支付账户。

## 账户激活

如下图，要求激活账户。

![stripe](https://images.iminling.com/app/hide.php?key=VEh5NHBpdngzelgrNXFaWWFGSjR4VUp5ekxxVGxqMUgyVlZpQ2sxUGxQNTh5alNRdTBrMXRCLzcyaHZVM1JtZ1JQbitReWs9)

点击激活支付功能

### 商家类型

地区美国，类型个人。

![stripe type](https://images.iminling.com/app/hide.php?key=Wmw1WjFLZEs1ZVlybnFObDFodzFSdkdTTjVMbS9GNDRFTS9Zcm9uQVZoTlZpcUlBcXZ1SXZSUkJreEpLYlVEUytEdFhnR009)

### 个人详情

法定名称就是自己的姓名，第一行名，第二行姓。

家庭地址我这里填写的申请ITIN时的地址。

电话号码填写的是GV号

社会保障码后四位：填写ITIN后四位

![stripe details](https://images.iminling.com/app/hide.php?key=UGxmOWlJcFZ0cGJ6YzVZNmJ0ZTBOTS9CUDJTRlY2MFZHcjNTUGZTSGRKSEVLUzJHbWhJZDhPMFB3cmtTZVlFblpCYmpJV2c9)

### 商家详情

纳税人识别码：填ITIN

商家地址：和上边个人详情的一样

网站：自己的网站

产品描述：介绍一下为何收款，我这里填的是英文的。

![stripe merchant info](https://images.iminling.com/app/hide.php?key=M0lQWHJkY0dqZ2d6dVpIWHJkMzc1UXM5Zk5ESGVZdThLenAzaFNLQUtsdnhVakI0cnc2OVVoMlVEeUFuUVpWOU92ZVFHck09)

### 公开详情

添加一些描述以及客服地址等信息

![stripe info](https://images.iminling.com/app/hide.php?key=QWl6U0tHT3FwNnJ2TTBvWDRYT3pHV1ZudnVsUm92MURqOUVlOTgwQ1ZGQ3gwdVI2NnVjYlJVM1RGdzJ6OXorVXpDZ21JZE09)

### 银行详情

我这里使用的是revolut的美元账户。

路径号码：ACH号码

账户：银行账号

![stripe card](https://images.iminling.com/app/hide.php?key=UC9DejIvQjdPSGFkV2JMSGptSlBBenZaV0M3M01CL3lYWlpGcUl6WUNnSWJUM2huYjFncUVlbGY2azExdzRxdFc0NHQ1dTA9)

### 两步验证

出于安全，添加2FA

![stripe 2fa](https://images.iminling.com/app/hide.php?key=cVRSZXo4eTRHMTZoY1lTNVNRNCtwRjN3em41Zm54ZGVETEllTi82OXhWZnprUHhQZnpPdjc3Y0o1YXU3a1NaTVNNOVMxZTA9)

### 税款计算

默认

![stripe tax](https://images.iminling.com/app/hide.php?key=UkZtVXMvTUxMSzZCNURpVHdqUlozZkFQUStVMEtoYzN6bnZYZGRqeDZxTFg2SmZQYjJkVVhiVWs4Mkdyd0lldVNkUmJOZWs9)

### 捐款

拒绝白嫖

![stripe climate](https://images.iminling.com/app/hide.php?key=dEQwUjhJRGZCaUl6a041eDJ4bnc2SEp6Qm9qWVlTSjNSYmpoQzFtZk8zSDYwTFZhbWJGVmxDa1BhUDNWbExqK0FWbUxwWkU9)

### 摘要

摘要信息

![stripe summary](https://images.iminling.com/app/hide.php?key=bUl6UGdiL01WRmxOZ1FEcFh6SWlDTWs3RTBVM0dhOENlTnh4RXVRdjkzcmNUeHo2a0VjL0hJVUZsUHVwTmpIajU5dHllYVU9)

提交等代审核就可以了。

## 设置账户

进入stripe的首页，提示账户已激活。

![stripe index](https://images.iminling.com/app/hide.php?key=RTFFcVZPNVBMZU40Q1BlakF3Qno3S2orbU5HQ2NiQXZaL09ycjJDU0YxeUxDditwNlA2dHVHUWI0L3k0d3FyTU5ENkJlR1k9)

但是当我们去创建收款的时候，发现还是不能使用。![stripe bill](https://images.iminling.com/app/hide.php?key=ak5wbDdWWFYwSnhIMWNrUzAzS2xDa0o0UVpOM0lkYjJ3VjN1VExGelhEVUlGY0JWQnFaZVdwOWxCQ2pXWDljSjA5cFk4ZlE9)

点击添加信息，发现还需要再次输入ITIN，提交。

![stripe itin](https://images.iminling.com/app/hide.php?key=Mi95a2ExYkd6NnMwaWVjOTdVU3hyUVUvOG4wVklYQk03b3dUb2I2V09vZWVVRkVKOHVndDNnU0VvdjBHOFJEVHgwTWxQTzg9)

### 手机号验证

账户详情处进行手机号验证

![stripe phoen verify](https://images.iminling.com/app/hide.php?key=VTIzWE0vOGordnlaejFiWGN5cHJDNEE5dW96OXhLNmxQSnphZDJURm02T2Z3RVlXWCt0T2s0bjAwY2JLZ25iamwyRmk5TUE9)

经过这些处理后基本已经可以正常使用了。

可以在账户状态看是否还需要补充什么信息，如下图就说明完全不需要补充资料了。

![stripe](https://images.iminling.com/app/hide.php?key=WmpUNmFhUG5rZkdWdUltNXY2ZGNNTUhrT1duZHpWUEQ1WUZQdlJoS05obXhZdUZwNVJ1ZXhxZi9pMmhjSzVaSHJwUTY3Zmc9)

账号注册完毕，可以开心收款了。
