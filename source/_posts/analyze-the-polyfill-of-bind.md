---
title: bind函数polyfill源码解析
date: 2018-05-24 16:28:34
tags:
- javascript
- bind polyfill
categories:
- 前端,每天学一点
---

分析`MDN`文档提供的`bind`函数`polyfill`实现细节。

<!--more-->

## 1. 准备知识

使用`new`来调用函数会自动执行下面的操作：
1. 创建一个全新的对象
2. 这个新对象会被执行原型连接
3. 这个新对象会绑定到函数调用的`this`
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象

注意`this`绑定规则，`new`操作具有最高的优先级

《你不知道的JavaScript(上卷)》提供了一个例子，`bar`被硬绑定到`obj`上，但是`new bar(3)` 并没有像我们预计的那样把`obj.a`修改为3。相反，`new`修改了硬绑定调用`bar()`中的`this`。
因为使用了`new`绑定，我们得到了一个名字为`baz`的新对象，并且`baz.a`的值为3。

```javascript
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

`instanceof`运算符的第一个变量是一个对象，暂时称为A；第二个变量一般是一个函数，暂时称为B。

`instanceof`判断准则：沿着A的`__proto__`这条线来找，同时沿着B的`prototype`这条线来找，如果两条线能找到同一个引用，即同一个对象，那么就返回`true`。

## 2. 源码分析

`MDN`上提供的`polyfill`如下，主要的疑惑点应该就是`this instanceof fNOP`作用是什么？

```javascript
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

`this instanceof fNOP`单独看是看不明白的，需要结合以下代码才能说明它的作用

```javascript
if (this.prototype) {
    fNOP.prototype = this.prototype; 
}
fBound.prototype = new fNOP()
```

首先我们要清楚：`bind(...)`会返回一个硬编码的新函数，它会把参数设置为`this`的上下文并调用原始函数。

重点就是`bind(...)`返回的是一个函数！函数！函数！这意味着可以对`bind(...)`返回的函数进行new操作，那么问题就来了。

`this`绑定中`new`操作具有最高的优先级，如果执行`new`操作，`bind(...)`应该不起作用，`this`应该指向`new`出来的新对象。

清楚了以上内容，我们开始阅读代码。

```javascript
if (typeof this !== 'function') {
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable')
    }
```

`bind(...)`必须由函数调用，所以以上代码对`this`的类型进行检查，如果不是函数类型则抛出错误。

```javascript
var aArgs   = Array.prototype.slice.call(arguments, 1)
var fToBind = this
var fBound  = function() {
              return fToBind.apply(this instanceof fNOP // 这段代码会判断硬绑定函数是否是被new调用，如果是的话就会使用新创建的this替换硬绑定的this
                     ? this
                     : oThis,
                     aArgs.concat(Array.prototype.slice.call(arguments)))
           }
```

`aArgs`获取传入的其它参数，`fToBind`获取需要硬绑定的函数，`fBound`为返回的绑定操作函数,我们先忽略`fBound`里面的内容，继续往下看。

```javascript
if (this.prototype) {
    fNOP.prototype = this.prototype; 
}
fBound.prototype = new fNOP()
```
我们假设对`bind(...)`返回的函数进行`new`操作(原型链如下),则`this instanceof fNOP`为`true`，此时我们就知道执行了`new`操作，
硬绑定的`this`不能生效，需要把`this`绑定到新生成的对象上。

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p15.png)

如果没有进行`new`操作的话，就用`apply`模拟`bind`绑定，一切按照原计划进行。

最后我们分析一下维护原型关系的重要性，例子如下：

```javascript
function Foo(){
    console.log(this.a);
    this.a=1;
}
Foo.prototype.show=function() {console.log(this.a)};
Foo();// undefined
var obj1=new Foo();
obj1.show();

var bar=Foo.bind({a:2});
bar();// 2
var obj2=new bar();
obj2.show();
```
因为`bind`函数内部保持了原型关系的继承，所以对象`obj2`才能访问到原型上的`show`方法。

** 注意：`Foo.show()`是错误的，因为`Foo`的原型指向的是`Function.prototype`，只有`Foo`的实例才能调用`show`方法。

## 3. 更多

`MDN`的`ployfill`地址：[>>>点我进入](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind#Polyfill)



