---
title: JavaScript原型与闭包总结
date: 2017-06-12 15:26:17
tags:
- javascript
- prototype
categories:
- 前端,每天学一点
---
学习原型与闭包这两个知识点，会让你对`javascript`有更深层次的理解，至少理解了原型和作用域，就不能再算是`javascript`菜鸟了。另外，这两方面也是`javascript`与其他语言不同的地方，学习这样的设计，有助于你开阔眼界，帮助你了解编程语言的设计思路。

<!--more-->

## 知识要点

1.判断一个变量是不是对象

（1）值类型的类型判断用`typeof`

（2）引用类型的类型判断用`instanceof`

2.`JS`中一切（引用类型）都是对象，对象是属性的集合

3.为函数添加属性

在`jQuery`中，`"jQuery"`或`"$"`这个变量其实就是一个函数

```javascript
typeof $  
// function

$.trim()
// 为$函数加了一个trim属性
```

4.在`typeof`的输出类型中，`function`和`object`都是对象，为何要输出两种

函数和数组都是对象，数组好比对象的一个子集，而函数和对象之间却不仅仅是一种包含和被包含的关系，有一点鸡生蛋蛋生鸡的关系。

对象都是通过函数创建的！！！
```javascript
typeof Object
// function

typeof Array
// function
```

5.每个函数都有一个默认的属性`prototype`，这个`prototype`的属性值是一个对象（属性的集合），默认只有一个叫做`constructor`的属性，指向这个函数本身。

6.每个对象都有一个隐藏的属性——`__proto__`，这个属性引用了创建这个对象的函数的`prototype`！

能清楚地解释以下语句的输出原因，你就理解了。
```javascript
var a=[ ]; 
a.constructor
//  function Array()
```

7.自定义函数的`prototype`本质上就是和`var obj={ }`是一样的，都是被`Object`创建，所以它的`__proto__`指向就是`Object.prototype`（特例：`Object.prototype`指向`null`）

8.函数也是一种对象，同样有`__proto__`，谁创建了函数？--->`Function`

对象的`__proto__`指向的是创建它的函数的`prototype`，所以
```javascript
Object.__proto__ === Function.prototype
// true
```

9.`Function`也是一个函数，函数是一种对象，也有`__proto__`属性，既然是函数，那么它一定是被`Function`创建，所以`Function`是被自身创建的，所以它的`__proto__`指向自身的`prototype`。


10.因为`Function.prototype`指向的对象也是一个普通的被`Object()`创建的对象，所以它的`__proto__`也指向`Object.prototype`。


11.`instanceof`运算符的第一个变量是一个对象，暂时称为A；第二个变量一般是一个函数，暂时称为B。

`instanceof`判断准则：沿着A的`__proto__`这条线来找，同时沿着B的`prototype`这条线来找，如果两条线能找到同一个引用，即同一个对象，那么就返回`true`。


12.访问一个对象的属性时，先在基本属性中查找，如果没有，再沿着`__proto__`这条链向上找，这就是原型链。

由于所有的对象的原型链都会找到`Object.prototype`，因此所有的对象都会有`Object.prototype`的方法，这就是继承。

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p06.png)

13.执行上下文环境：在执行代码之前，把将要用到的所有变量都事先拿出来，有的直接赋值了，有的先用`undefined`占个位

14.执行上下文

- 变量、函数表达式——变量声明，默认赋值为`undefined`

- `this`——赋值

- 函数声明——赋值

* `JS`在执行一个代码段之前，都会进行这些准备工作来生成执行上下文

* 代码段分三种情况：全局代码、函数体、`eval`代码

15.函数每被调用一次，都会产生一个新的执行上下文环境，函数在定义的时候（不是调用的时候）就已经确定了函数体内部自由变量的作用域

函数体的执行上下文

- 参数——赋值

- `arguments`——赋值

- 自由变量的取值作用域——赋值

16.`this`取值（5种情况）——`this`指向调用该函数的对象

- 如果函数作为构造函数，其中`this`代表它即将`new`出来的对象

- 函数作为对象的一个属性，并且作为对象的一个属性被调用时，函数中的`this`指向该对象；如果函数被赋值到另一个变量中，并没有作为`obj`的一个属性被调用，那么`this`的值就是`Window`

```javascript
var obj={
        fn:function() {
        console.log(this);// this指向Window
    }
}
var fn1=obj.fn;
fn1();
```

- 当一个函数被`call`或`apply`调用时，`this`的值就取传入的对象

- 在全局环境下或调用普通函数，`this`永远是`Window`

- 不仅仅是构造函数的`prototype`，即便在整个原型链中，`this`代表当前对象

17.`JS`中没有块级作用域，`JS`中除了全局作用域之外，只有函数可以创建作用域。所以在声明变量时，全局代码要在代码前端声明，函数中要在函数体一开始就声明好，其它地方不用出现变量声明，而且建议使用单`var`形式。

18.在A作用域中使用变量`x`，却没有在A作用域中声明，那么对于A作用域来说，`x`就是一个自由变量。

19.作用域有上下级关系，上下级关系的确定就看函数是在哪个作用域下创建的。对于自由变量，要到创建这个函数的那个作用域中取值（创建，而不是调用）。

## 更多

深入理解javascript原型和闭包：[>>>点我进入](http://www.cnblogs.com/wangfupeng1988/p/3977924.html)