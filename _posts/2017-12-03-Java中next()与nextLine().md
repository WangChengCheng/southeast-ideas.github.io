---
layout: post
title:  Java——next()与nextLine()
date:   2017-12-03 22:14:00 +0800
categories: Java
tag: Java Scanner next()
author: Michael Wang
source_id: 1712032214
---

* content
{:toc}

### Java中next()与nextLine()
```next()```这个函数会扫描从有效字符起到空格，Tab，回车等结束字符之间的内容并作为String返回。
```nextLine()```这个函数在你输入完一些东西之后按下回车则视为输入结束，输入的内容将被作为String返回。
```next()```这个函数与之不同在于,```next()```什么都不输入直接敲回车不会返回，而```nextLine()```即使不输入东西直接敲回车也会返回。