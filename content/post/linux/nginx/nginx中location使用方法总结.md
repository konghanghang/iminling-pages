---
title: nginx中location使用方法总结
author: 要名俗气
type: post
date: 2023-08-30T14:57:18+00:00
url: /2023/nginx-location-use-summary
description: 在nignx中我们比较长使用的就是location，我们使用location来定位资源或者转发请求，使用不当经常就会遇到404错误，无法找到资源或者请求异常。下面我们来详细了解一下location的用法。 语法 location的具体语法如下： 语法规则很简单，一个location关键字，后面跟着可选的修饰符，后面是要匹配的字符，花括号中是要执行的操作。
featured_image: /wp-content/uploads/2023/08/d660acf4da171b8c91ba0bdfd7f73cd6.png
categories:
  - nginx
tags:
  - nginx
---
![](https://www.iminling.com/wp-content/uploads/2023/08/459feb46c0c414a826f2055d8d28cd27.png)

在nignx中我们比较长使用的就是location，我们使用location来定位资源或者转发请求，使用不当经常就会遇到404错误，无法找到资源或者请求异常。下面我们来详细了解一下location的用法。

## 语法

location的具体语法如下：

```
location [ = | ~ | ~* | ^~ ] uri { ... }
location @name { ... }
```

<span class="md-plain md-expand">语法规则很简单，一个</span><span class="md-pair-s" spellcheck="false"><code>location</code></span><span class="md-plain md-expand">关键字，后面跟着可选的修饰符，后面是要匹配的字符，花括号中是要执行的操作。</span>

## 修饰符

在语法中可以看到location有四中修饰符，其实还有一种就是修饰符就是什么也没有，location后边直接跟uri，下边来详细了解一下各种修饰符的作用：

<ol class="ul-list" data-mark="-">
  <li class="md-list-item md-focus-container">
    <p class="md-end-block md-p md-focus">
      <span class="md-pair-s md-expand" spellcheck="false"><code>=</code></span><span class="md-plain"> 表示精确匹配。只有请求的url路径与后面的字符串完全相等时，才会命中。</span>
    </p>
  </li>

  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-pair-s" spellcheck="false"><code>~</code></span><span class="md-plain"> 表示该规则是使用正则定义的，区分大小写。</span>
    </p>
  </li>

  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-pair-s" spellcheck="false"><code>~*</code></span><span class="md-plain"> 表示该规则是使用正则定义的，不区分大小写。</span>
    </p>
  </li>

  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-pair-s" spellcheck="false"><code>^~</code></span><span class="md-plain"> 表示如果该符号后面的字符是最佳匹配，采用该规则，不再进行后续的查找。</span>
    </p>
  </li>

  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-pair-s" spellcheck="false"><code>无修饰符</code></span><span class="md-plain"> 表示前缀匹配</span>
    </p>
  </li>
</ol>

<p class="md-end-block md-p md-focus">
  <span class="md-pair-s" spellcheck="false"><code>=</code></span><span class="md-plain">、</span><span class="md-pair-s" spellcheck="false"><code>~</code></span><span class="md-plain">和</span><span class="md-pair-s" spellcheck="false"><code>^~</code></span><span class="md-plain">是非正则匹配，</span><span class="md-pair-s" spellcheck="false"><code>~</code></span><span class="md-plain">和</span><span class="md-pair-s" spellcheck="false"><code>~*</code></span><span class="md-plain">是正则匹配。</span>
</p>

<p class="md-end-block md-p md-focus">
  <span class="md-plain">location匹配还有一个特点：</span><span class="md-pair-s ">**<span class="md-plain">非正则匹配优先！</span>**</span><span class="md-plain md-expand">非正则>正则</span>
</p>

## 匹配过程

总共5种修饰符，如果一个url请求过来可以匹配到多种，那nginx会如何选择呢？下边来看一下完整的匹配过程：

<ol class="ol-list" start="">
  <li class="md-list-item md-focus-container">
    <p class="md-end-block md-p md-focus">
      <span class="md-plain md-expand">用所有的前缀字符串去匹配URI(这里说的前缀字符串，是除</span><span class="md-pair-s" spellcheck="false"><code>~</code></span><span class="md-plain">和</span><span class="md-pair-s" spellcheck="false"><code>~*</code></span><span class="md-plain">修饰的字符串外的所有字符串，包括</span><span class="md-pair-s" spellcheck="false"><code>=</code></span><span class="md-plain">、</span><span class="md-pair-s" spellcheck="false"><code>^~</code></span><span class="md-plain">及</span><span class="md-pair-s" spellcheck="false"><code>无修饰符</code></span><span class="md-plain">三种都算是前缀字符串)；</span>
    </p>
  </li>

  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-plain">如果找到</span><span class="md-pair-s" spellcheck="false"><code>=</code></span><span class="md-plain">号修饰的字符串与URI匹配(无论它在哪个位置)，则终止匹配过程，并进入该location内部执行具体操作；</span>
    </p>
  </li>

  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-plain">如果不找到</span><span class="md-pair-s" spellcheck="false"><code>=</code></span><span class="md-plain">号修饰的字符串，则继续匹配普通前缀字符串(即用</span><span class="md-pair-s" spellcheck="false"><code>^~</code></span><span class="md-plain">修饰的或</span><span class="md-pair-s" spellcheck="false"><code>无修饰符</code></span><span class="md-plain">的字符串)；</span>
    </p>
  </li>

  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-plain">在</span><span class="md-pair-s" spellcheck="false"><code>^~</code></span><span class="md-plain">修饰的或</span><span class="md-pair-s" spellcheck="false"><code>无修饰符</code></span><span class="md-plain">的字符串中如果有多个规则能匹配上URI，则会查看最长的那一个是否用</span><span class="md-pair-s" spellcheck="false"><code>^~</code></span><span class="md-plain">修饰；</span>
    </p>
  </li>

  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-plain">如果最长的那个有用</span><span class="md-pair-s" spellcheck="false"><code>^~</code></span><span class="md-plain">修饰，则终止匹配，这个用</span><span class="md-pair-s" spellcheck="false"><code>^~</code></span><span class="md-plain">修饰的“最长”的匹配就是最终被选用的location，但是如果这个“最长”的匹配没有用</span><span class="md-pair-s" spellcheck="false"><code>^~</code></span><span class="md-plain">修饰(说明它是</span><span class="md-pair-s" spellcheck="false"><code>无修饰符</code></span><span class="md-plain">的)，则会暂时把它存储下来，但不会终止匹配，而是继续进行后面的正则匹配；</span>
    </p>
  </li>

  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-plain">开始正则匹配，匹配所有用</span><span class="md-pair-s" spellcheck="false"><code>~</code></span><span class="md-plain">或</span><span class="md-pair-s" spellcheck="false"><code>~*</code></span><span class="md-plain">修饰的字符串</span>
    </p>
  </li>

  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-plain">匹配到第一个正则表达式后终止匹配，使用对应的location；</span>
    </p>
  </li>

  <li class="md-list-item md-focus-container">
    <p class="md-end-block md-p md-focus">
      <span class="md-plain md-expand">如果没有匹配到正则表达式，则使用之前存储的前缀字符串对应的location，至此，整个匹配过程结束。</span>
    </p>
  </li>
</ol>

## 路径匹配

使用location的时候，代码块里一般会使用`root`或者`alias`或者`proxy_pass`下边详细说一下proxy_pass的用法。

### proxy_pass

<p class="md-end-block md-p md-focus">
  <span class="md-plain md-expand">Nginx的</span><span class="md-meta-i-c md-link"><a href="http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass"><span class="md-plain">官网</span></a></span><span class="md-plain">将proxy_pass分为两种类型：</span>
</p>

<ol class="ol-list" start="">
  <li class="md-list-item">
    <p class="md-end-block md-p">
      <span class="md-plain">一种是只包含IP和端口号的（连端口之后的/也没有，这里要特别注意），比如</span><span class="md-pair-s" spellcheck="false"><code>proxy_pass http://localhost:8080</code></span><span class="md-plain">，这种方式称为不带URI方式；</span>
    </p>
  </li>

  <li class="md-list-item md-focus-container">
    <p class="md-end-block md-p md-focus">
      <span class="md-plain">另一种是在端口号之后有其他路径的，包含了只有单个</span><span class="md-pair-s" spellcheck="false"><code>/</code></span><span class="md-plain">的，如</span><span class="md-pair-s" spellcheck="false"><code>proxy_pass http://localhost:8080/</code></span><span class="md-plain">，以及其他路径，比如</span><span class="md-pair-s" spellcheck="false"><code>proxy_pass http://localhost:8080/abc</code></span><span class="md-plain md-expand">。</span>
    </p>
  </li>
</ol>

####

#### 不带URI方式(相对路径)

<p class="md-end-block md-p md-focus">
  <span class="md-plain md-expand">对于不带URI方式，Nginx将会保留location中路径部分，比如：</span>
</p>

```
location /api1/ {
  proxy_pass http://localhost:8080;
}
```

<p class="md-end-block md-p md-focus">
  <span class="md-plain">在访问</span><span class="md-pair-s" spellcheck="false"><code>http://localhost/api1/xxx</code></span><span class="md-plain">时，会代理到</span><span class="md-pair-s md-expand" spellcheck="false"><code>http://localhost:8080/api1/xxx</code></span>
</p>

####

#### 带URI方式(绝对路径)

<p class="md-end-block md-p">
  <span class="md-plain">对于带URI方式，nginx将使用诸如alias的替换方式对URL进行替换，并且这种替换只是字面上的替换，比如：</span>
</p>

```
location /api2/ {
  proxy_pass http://localhost:8080/;
}
```

<p class="md-end-block md-p md-focus">
  <span class="md-plain">当访问</span><span class="md-pair-s" spellcheck="false"><code>http://localhost/api2/xxx</code></span><span class="md-plain">时，</span><span class="md-pair-s" spellcheck="false"><code>http://localhost/api2/</code></span><span class="md-plain">（注意最后的/）被替换成了</span><span class="md-pair-s" spellcheck="false"><code>http://localhost:8080/</code></span><span class="md-plain">，然后再加上剩下的</span><span class="md-pair-s" spellcheck="false"><code>xxx</code></span><span class="md-plain">，于是变成了</span><span class="md-pair-s" spellcheck="false"><code>http://localhost:8080/xxx</code></span><span class="md-plain">。</span>
</p>

下面再给一些例子来帮助大家理解相对路径和绝对路径：

```
server {
  listen       80;
  server_name  localhost;

  # proxy_pass中的域名是相对路径，所以会带上location中的路径
  location /api1/ {
    proxy_pass http://localhost:8080;
  }
  # http://localhost/api1/xxx -> http://localhost:8080/api1/xxx

  # proxy_pass中的域名是绝对路径，所以不会带上location中的路径
  location /api2/ {
    proxy_pass http://localhost:8080/;
  }
  # http://localhost/api2/xxx -> http://localhost:8080/xxx

  # proxy_pass中的域名是相对路径，所以会带上location中的路径
  location /api3 {
    proxy_pass http://localhost:8080;
  }
  # http://localhost/api3/xxx -> http://localhost:8080/api3/xxx

  # proxy_pass中的域名是绝对路径，所以不会带上location中的路径
  location /api4 {
    proxy_pass http://localhost:8080/;
  }
  # http://localhost/api4/xxx -> http://localhost:8080//xxx，匹配到/api4/xxx,location中是/api4需去掉剩下/xxx，加上proxy_pass则是//xxx

  # proxy_pass中的域名是绝对路径，所以不会带上location中的路径
  location /api5/ {
    proxy_pass http://localhost:8080/haha;
  }
  # http://localhost/api5/xxx -> http://localhost:8080/hahaxxx，匹配到/api5/xxx, 去掉location中的/api5/剩下xxx，加上proxy_pass则是hahaxxx

  # proxy_pass中的域名是绝对路径，所以不会带上location中的路径
  location /api6/ {
    proxy_pass http://localhost:8080/haha/;
  }
  # http://localhost/api6/xxx -> http://localhost:8080/haha/xxx

  # proxy_pass中的域名是绝对路径，所以不会带上location中的路径
  location /api7 {
    proxy_pass http://localhost:8080/haha;
  }
  # http://localhost/api7/xxx -> http://localhost:8080/haha/xxx

  # proxy_pass中的域名是绝对路径，所以不会带上location中的路径
  location /api8 {
    proxy_pass http://localhost:8080/haha/;
  }
  # http://localhost/api8/xxx -> http://localhost:8080/haha//xxx，匹配到/api8/xxx,去掉location中的/api8剩下/xxx,加上proxy_pass则是/haha//xxx
}
```

以上就是nginx中location的使用方法，匹配规则，以及代码块中proxy_pass的使用方法，希望可以帮到大家。

