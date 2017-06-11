---
layout: post
title:  如何将GitHub Page项目与域名绑定
date:   2017-05-23 20:10:00 +0800
categories: 建站
tag: 教程
author: Michael Wang
source_id: 170523201000
---

* content
{:toc}


创建CNAME文件
------------------------------------

在项目的根目录下添加[CNAME]()文件，在文件中填写要绑定的域名，比如我的项目欲通过`blog.southeast-ideas.com`打开，那我就在CNAME文件中录入
{% highlight bash %}
blog.southeast-ideas.com
{% endhighlight %}
即可。

在域名控制台下添加解析（以阿里云万网为例）
------------------------------------

+ 在域名控制台，添加一条`记录类型`为`CNAME`的解析记录
+ 要将域名解析为`www.example.com`，在`主机记录(RR)`处填写`www`即可。（比如要以“blog.southeast-ideas.com”的网址访问项目，则主机记录(RR)填写为“blog”）
+ `记录值`为`username.github.io.`

<img src="{{ '/styles/images/blog_images/20170523_00.png' | prepend: site.baseurl }}" alt="域名解析记录示例" />
{% highlight bash %}
如果该域名下还绑定有其他记录，则可能存在冲突，此时需要修改其他记录的“主机记录”或“记录类型”以解决冲突。
{% endhighlight %}