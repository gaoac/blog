---
title: 为什么要使用闭包(closure)?
date: 2017-01-01 15:39:11
tags: [JavaScript]
categories: [JavaScript]
description: ""
---

## 前言

闭包(closure)这个词是在学习JavaScript过程中经常被提到的，很多人一开始都对此有些迷惑，然后在不断地参考学习中逐渐掌握，本文主要简述下自己学习过程中对闭包的理解。

## 什么是闭包

MDN官方解释：
> Closures are functions that refer to independent (free) variables (variables that are used locally, but defined in an enclosing scope). In other words, these functions 'remember' the environment in which they were created.

closure是拥有独立变量（在封闭空间中定义的可以在本地环境中使用的变量）的函数，换句话说，就是这种函数可以"记住"他们创建的环境。
简而言之，就是闭包是一个函数，能将创建的变量的值始终保持在内存中，以供本地环境使用。<!-- more -->

## 为什么要用闭包

说到这里就不得不提下JavaScript的变量作用域问题。变量作用域无非就两种：全局作用域和局部作用域。
在JavaScript（特指ECMAScript5前的版本）语言中具有作用域的仅有函数function。并且有个特点就是：函数内部可以直接访问外部变量，但在函数外部无法访问函数内部变量。这也就是Javascript语言特有的“链式作用域”结构（chain scope）。
那么我要是想在函数外部访问函数内部变量怎么办？所以闭包就出现了，简单说，我们使用闭包的主要作用就是间接访问函数的内部数据。

## 怎么创建闭包

理解了JavaScript的作用域局限以及为什么需要使用闭包之后，我们就可以动手解决这一问题。下面用一个简单例子来详细说明这一过程：

**1.首先我们来看下要解决的问题：**

 ```javascript
  var num = 12;
  function showNum() {
      console.log(num);
  }
showNum();//12
```

在上面代码中执行函数showNum()，因为函数内部可以访问外部变量num，所以执行结果为12.

```javascript
function showNum() {
     var num = 12;  //此处注意要使用var声明，否者等于直接声明全局变量num
  }
 console.log(num);//Uncaught ReferenceError: num is not defined
```

由于函数外部不能够访问函数内部变量，所以直接访问输出num时，由于此时num并未定义，故抛出错误信息。那么如何获取函数内部变量呢？

**2.在函数内部再定义一个内部函数，并将这个内部函数当做返回值，这样我们就可以获取函数的局部变量。**

```javascript
 function showNum() {
      var num = 12;
      function showNum2() {
           console.log(num);
      };
      return showNum2;
  }
var myNum = showNum();
myNum();//12
```

在上面的代码中，在函数showNum中嵌套一层函数showNum2，由于内层函数可以访问外层函数的变量，所以，执行showNum()函数时，showNum2函数可以直接读取外层函数showNum()的局部变量num的值，并输出12。
**以上实现了闭包的主要作用之一：读取函数内部变量**

**3.那么闭包还有什么作用呢？**
还记得文章开头引用的MDN的解释吗：
>Closures are functions that refer to independent (free) variables (variables that are used locally, but defined in an enclosing scope). In other words, **these functions 'remember' the environment in which they were created**.
//将创建的变量的值始终保持在内存中，以供本地环境使用

是的，这也是闭包的作用之一，如下例子：

```javascript
 function showNum() {
      var num = 12;
      function showNum2() {
           console.log(++num);
      };
      return showNum2;
  }
var myNum = showNum();
myNum();//13
```

一般情况下在函数被调用完后不再被引用时，该函数都会被垃圾回收机制（garbage collection），但是由于上述代码中函数showNum2被myNum引用，而函数showNum2又依赖于函数showNum，因此函数showNum不会被垃圾回收机制回收。

**以上只是目前学习中的一些总结整理，对其原理性认识还完全达不到全面，后续会继续了解完善。**
