---
layout: post
title: JavaScript-let
date:   2018-03-10 10:14:00 +0800
categories: JavaScript
tag: JavaScript
author: Michael Wang
source_id: 180310101400
---

* content
{:toc}

##let
`let`语句声明一个块级作用域的本地变量，并且可选的将其初始化为一个值。<br/>
`let`允许你声明一个作用域被限制在块级中的变量、语句或者表达式。与`var`关键字不同的是，它声明的变量只能是全局或者整个函数块的

##作用域规则(let vs var)

`let`声明的变量只在其声明的块或子块中可用，这一点，与`var`相似。二者之间最主要的区别在于`var`声明的变量的作用域是整个封闭函数。


```javascript
function varTest() {
  var x = 1;
  if (true) {
    var x = 2;  // 同样的变量!
    console.log(x);  // 2
  }
  console.log(x);  // 2
}

function letTest() {
  let x = 1;
  if (true) {
    let x = 2;  // 不同的变量
    console.log(x);  // 2
  }
  console.log(x);  // 1
}
```
## let vs const

const声明的变量与let声明的变量类似，它们的不同之处在于，const声明的变量只可以在声明时赋值，不可随意修改，否则会导致SyntaxError（语法错误）。
```javascript
const MAX_CAT_SIZE_KG = 3000; // 正确
MAX_CAT_SIZE_KG = 5000; // 语法错误（SyntaxError）
MAX_CAT_SIZE_KG++; // 虽然换了一种方式，但仍然会导致语法错误
```
当然，规范设计的足够明智，用const声明变量后必须要赋值，否则也抛出语法错误。
```javascript
const theFairest;  // 依然是语法错误，你这个倒霉蛋
```

## 参考
[JavaScript参考文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/let)<br/>
[深入浅出ES6（十四）：let和const](http://www.infoq.com/cn/articles/es6-in-depth-let-and-const)<br/>
[JavaScript中var、let、const区别？](https://www.zhihu.com/question/52662013)<br/>