---
title: JavaScript中判断一个对象是不是数组
date: 2017-07-16 16:06:05
tags:
- javascript
- array
categories:
- 前端,每天学一点
---
判断一个对象为数组的所有方法以及原理分析。
<!--more-->
## 一.typeof
首先`typeof`只能判断原始类型和对象，返回结果是一个说明运算数类型的字符串。如：`"number","string","boolean","object","function","undefined"`（可用于判断变量是否存在）。 `typeof` 的能力有限，其对于`Array、Date、RegExp`类型返回的都是`"object"`。

## 二.方法
### 1.Array.isArray（[ ]）
`Array.isArray()`用于确定传递的值是否是一个 `Array`，如果对象是 `Array`，则为`true`; 否则为`false`。鲜为人知的事实：其实`Array.prototype`也是一个数组。但是`Array.isArray()`是`ES5`新特性，许多老式浏览器可能并不支持，所以不是一个很保险的方法。
```javascript
Array.isArray(Array.prototype)//true
```
### 2.Object.prototype.toString.call（[ ]）
`Object`对象的`toString()`方法会返回所创建对象的内部类名。在这里，`toString()`方法必须要来自于`Object`构造器的`prototype`属性。直接调用`Array`的`toString()`方法是不行的，因为在`Array`对象中，这个方法已经出于其他目的被重写了。
```javascript
Object.prototype.toString.call({});
//"[Object Object]"

Object.prototype.toString,call([]);
//"[Object Array]"

(function (){
  return Object.prototype.toString.call(arguments);
})();
//"[Object Arguments]"
```
### 3.[ ] instanceof Array
`instanceof` 运算符用来测试一个对象在其原型链中是否存在一个构造函数的 `prototype` 属性。`instanceof` 运算符用来检测 `constructor.prototype` 是否存在于参数 `object` 的原型链上，即`object`是否是该构造函数的一个实例。该方法的本意和方法4类似。

语法： 

> object  instanceof  constuctor

### 4.[ ].__proto__ == Array.prototype
![数组原型链](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p01.png)
### 5.[ ].constructor  
调用该对象的构造器方法，该方法会返回该对象的构造器函数，由于 `[ ]` 实例中不存在`constructor`属性，`JS`会沿着原型链向上查找，会在`Array.prototype`中找到`constructor`属性，调用该方法，则返回`Array()`构造函数。
 ```javascript
 [ ].constructor
 //function Array() { [native code] }
```
