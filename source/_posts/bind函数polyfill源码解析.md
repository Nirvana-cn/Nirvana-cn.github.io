---
title: bind函数polyfill源码解析
date: 2018-05-24 16:28:34
tags:
- JavaScript
categories:
- Web前端
---

分析MDN文档提供的bind函数polyfill实现细节

<!--more-->

MDN上提供的polyfill如下，主要的疑惑点应该就是 this instanceof fNOP 作用是什么？

```
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
    }
    var aArgs   = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          return fToBind.apply(this instanceof fNOP // 这段代码会判断硬绑定函数是否是被new调用，如果是的话就会使用新创建的this替换硬绑定的this
                 ? this
                 : oThis,
                 aArgs.concat(Array.prototype.slice.call(arguments)))
        }
    // 维护原型关系
    if (this.prototype) {
        // Function.prototype doesn't have a prototype property
        fNOP.prototype = this.prototype; 
    }
    fBound.prototype = new fNOP()
    return fBound
  }
}
```

this instanceof fNOP 单独看是看不明白的，需要结合以下代码才能说明它的作用

```
if (this.prototype) {
    fNOP.prototype = this.prototype; 
}
fBound.prototype = new fNOP()
```

首先我们要清楚：bind(...)会返回一个硬编码的新函数，它会把参数设置为this的上下文并调用原始函数。

重点就是bind(...)返回的是一个函数！函数！函数！这意味着可以对bind(...)返回的函数进行new操作，那么问题就来了。

this绑定中new操作具有最高的优先级，如果执行new操作，bind(...)应该不起作用。

清楚了以上内容，我们开始阅读代码。

```
if (typeof this !== 'function') {
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
    }
```

bind(...)必须由函数调用，所以以上代码对this的类型进行检查，如果不是函数类型则抛出错误。

```
var aArgs   = Array.prototype.slice.call(arguments, 1)
var fToBind = this
var fBound  = function() {
              return fToBind.apply(this instanceof fNOP // 这段代码会判断硬绑定函数是否是被new调用，如果是的话就会使用新创建的this替换硬绑定的this
                     ? this
                     : oThis,
                     aArgs.concat(Array.prototype.slice.call(arguments)))
           }
```

aArgs获取传入的其它参数，fToBind获取需要硬绑定的函数，fBound为返回的绑定操作函数,我们先忽略fBound里面的内容，继续往下看。

```
if (this.prototype) {
    fNOP.prototype = this.prototype; 
}
fBound.prototype = new fNOP()
```
我们假设对bind(...)返回的函数进行new操作(原型链如下),则this instanceof fNOP 为true，此时我们就知道执行了new操作，硬绑定的this不能生效，需要把this绑定到新生成的对象上。

![](http://112.74.18.120:3001/p15.png)

如果没有进行new操作的话，就用apply模拟bind绑定，一切按照原计划进行。

## 准备知识

使用new来调用函数会自动执行下面的操作：
1. 创建一个全新的对象
2. 这个新对象会被执行原型连接
3. 这个新对象会绑定到函数调用的this
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象

注意this绑定规则，new操作具有最高的优先级

《你不知道的JavaScript(上卷)》提供了一个例子，bar被硬绑定到obj上，但是new bar(3) 并没有像我们预计的那样把obj.a修改为3。相反，new修改了硬绑定调用bar()中的this。因为使用了new绑定，我们得到了一个名字为baz的新对象，并且baz.a的值为3。

```
function foo(something) {
    this.a = something
}
var obj = {}
var bar = foo.bind(obj)
bar(2)
console.log(obj.a)  //2
var baz = new bar(3)
console.log(obj.a)  //2
console.log(baz.a)  //3
```

instanceof运算符的第一个变量是一个对象，暂时称为A；第二个变量一般是一个函数，暂时称为B。

instanceof判断准则：沿着A的__proto__这条线来找，同时沿着B的prototype这条线来找，如果两条线能找到同一个引用，即同一个对象，那么就返回true。


