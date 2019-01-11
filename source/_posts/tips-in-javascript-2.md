---
title: JavaScript中的小技巧和注意点（二）
date: 2017-11-28 16:37:42
tags:
- javascript
categories:
- 前端,每天学一点
---
记录平时学习过程中关于JavaScript容易遗忘的知识点和需要注意的地方，点点滴滴贵在积累...
<!--more-->

## 1.函数优先
函数声明和变量声明都会被提升，但是一个值得注意的细节是函数会首先被提升，然后才是变量。
```javascript
foo();//1
var foo;
function foo(){
	console.log(1);
}
foo=function(){
	console.log(2);
};
```
会输出1而不是2，这个代码片会被引擎理解如下形式：

```javascript
function foo(){
	console.log(1);
}
foo();//1
foo=function(){
	console.log(2);
};
```
注意，`var foo`尽管出现在`function foo()...`的声明之前，但它是重复的声明（因此被忽略了），因为函数声明会被提升到普通变量之前。
## 2.循环和闭包
当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。

```javascript
for(var i=1;i<=5;i++){
    setTimeout(function timer() {
        console.log(i);
    },i*1000);
}
```
正常情况下，我们对这段代码行为的预期是分别输出数字1-5，每秒一个，每次一个，但实际上，这段代码在运行时会以每秒依次的频率输出五次6。
（1）通过立即执行函数IIFE进行修正。
```javascript
for(var i=1;i<=5;i++){
    (function (j) {
        setTimeout(function timer() {
            console.log(j);
        },j*1000);
    })(i);
}
```

（2）更简单的方法，使用`let`生成块级作用域。
```javascript
for(let i=1;i<=5;i++){
    setTimeout(function timer() {
        console.log(i);
    },i*1000);
}
```
## 3.被忽略的this
如果你把`null`或者`undefined`作为`this`的绑定对象传入`call`，`apply`或者`bind`，这些值在调用时会被忽略，实际应用的是默认绑定规则。

```javascript
function foo(){
	console.log(this.a);
}
var a=2;
foo.call(null);//2
```
一种非常常见的做法是使用`apply()`来展开一个数组，并当作参数传入一个函数；类似的，`bind()`可以对参数进行柯里化。然而，总是使用`null`来忽略`this`绑定可能产生一些副作用，导致一些难以分析和追踪的`bug`。更安全的做法是传入一个空对象。在`JavaScript`中创建一个空对象最简单的方法是`Object.create(null)`。`Object.create(null)`和 `{ }`很像，但是并不会创建`Object.prototype`这个委托，所以它比 `{ }` "更空"。
## 4.隐式丢失
一个最常见的`this`绑定问题就是被隐式绑定的函数会丢失绑定对象，从而把`this`绑定到全局对象或者`undefined`上，取决去是否是严格模式。参数传递就是一种隐式赋值，因此传入函数时会被隐式赋值。
```javascript
function foo(){
    console.log(this.a)
}
var obj={
    a:2,
    foo:foo
};
var a='oops,global';
setTimeout(obj.foo,1000);//oops,global
```
`JavaScript`环境中内置`setTimeout()`函数实现和下面的伪代码类似，可以看出`obj.foo`被赋值给了`fn`，所以`foo`中的`this`被绑定到了全局对象上。

```javascript
function setTimeout(fn,delay){
	//等待delay毫秒
	fn();
}
```
## 5.对象属性存在性

```javascript
var obj={
	a:undefined
}
```
对象属性访问返回值可能是`undefined`，但是这个值有可能是属性中存储的`undefined`，也可能是因为属性不存在而返回`undefined`

```javascript
"a" in obj;//true
obj.hasOwnProperty("a");//true
```
但是有的对象可能没有连接到`Object.prototype`(比如通过`Object.create(null)`来创建的)，这种情况下形如`obj.hasOwnProperty()`就会失败，这时可以使用一种更加强硬的方法来进行判断。

```javascript
Object.prototype.hasOwnProperty.call(obj,"a");
```
## 6.计时器
由于历史原因，`setTimeout()`和`setInterval()`的第一个参数可以作为字符串传入。如果这么做，那么这个字符串就会在指定的间隔之后进行求值(相当于执行`eval()`)。

```javascript
setTimeout('console.log(123)',1000)
```
## 7.值和类型
`JavaScript`中的变量是没有类型的，只有值才有，变量可以随时持有任何类型的值，即一个变量可以现在被赋值为字符串类型值，随后又被赋值为数字类型值。在对变量执行`typeof`操作时，得到的结果并不是该变量的类型，而是该变量持有的值的类型，因为`JavaScript中`的变量没有类型。

已在作用域中声明但还没赋值的变量是`undefined`。相反，还没有在作用域中声明过的变量是`undeclared`，让人抓狂的是`typeof`处理`undeclared`变量的方式同样返回`“undefined”`

```javascript
var a;
typeof a;//undefined
typeof b;//undefined
```

## 8.数组的键值

数组通过数字进行索引，但有趣的是它们也是对象，所以也可以包含字符串键值和属性(但是，这些并不计算在数组长度内！！！)

```javascript
var a=[]
a[0]=1
a['foo']=2
a.length    //1
```

这里有个问题需要特别注意，如果字符串键值能够被强制类型转换为十进制数字的话，它就会被当作数字索引来处理。

```javascript
var a=[]
a['3']=3
a.length    //4
```

## 9.页面拥有ID的元素会创建全局变量

在一张`HTML`页面中，所有设置了`ID`属性的元素会在`JavaScript`的执行环境中创建对应的全局变量，这意味着`document.getElementById`像人的阑尾一样显得多余了。但实际项目中最好老老实实该怎么写就怎么写，毕竟常规代码出乱子的机会要小得多。
```html
<div id="sample"></div>
<script type="text/javascript">
        console.log(sample);
</script>
```

## 10.if语句的变形

当你需要写一个`if`语句的时候，不妨尝试另一种更简便的方法，用`JavaScript`中的逻辑操作符来代替。

```javascript
var day=(new Date).getDay()===0;
//传统if语句
if (day) {
	alert('Today is Sunday!');
};
//运用逻辑与代替if
day&&alert('Today is Sunday!');
```

## 11.数字的语法

对于 `.` 运算需要给予特别注意，因为它是一个有效的数字字符，会被优先识别为数字常量的一部分，然后才是对象属性访问运算符。

```javascript
// 无效语法
42.toFixed(3)

// 有效语法
(42).toFixed(3) //"42.000"
0.42.toFixed(3) //"0.420"
42..toFixed(3)  //"42.000"
42 .toFixed(3)
```

`42..toFixed(3)`没有问题，因为第一个`.`被视为`number`的一部分，第二个`.`是属性访问运算符

## 12.获取当前时间

`ES5`引入了静态函数`Date.now()`来获取当前时间，`Date.now()`同样返回毫秒数。

```javascript
if(!Date.now){
    Date.now = function() {
        return (new Date()).getTime()
    }
}
```

## 13.清空和截短数组

最简单的清空和截短数组的方法就是改变`length`属性：

```javascript
// 截短
const arr=[11,22,33,44,55];
arr.length=3;
console.log(arr); //[11,22,33]
// 清空
arr.length=0;
console.log(arr); //[]
```
## 14.数组的对象解构

使用对象解构将数组项赋值给变量：

```javascript
const csvFileLine = '1997,John Doe,US,john@doe.com,New York';
const { 2: country, 4: state } = csvFileLine.split(',');
```

## 15.await 多个 async 函数

`await`多个`async`函数并等待他们执行完成，我们可以使用 `Promise.all`：

```javascript
await Promise.all([anAsyncCall(), thisIsAlsoAsync(), oneMore()])
```

## 16.格式化 JSON 代码

`JSON.stringify`不仅可以字符串化对象，它也可以格式化你的 `JSON` 输出：

```javascript
const obj = {
  foo: {
    bar: [11, 22, 33, 44],
    baz: {
      bing: true,
      boom: 'Hello'
    }
  }
};
// 第三个参数为格式化需要的空格数目
JSON.stringify(obj, null, 4);
// =>"{
  // => "foo": {
    // => "bar": [
      // => 11,
      // => 22,
      // => 33,
      // => 44
    // => ],
    // => "baz": {
      // => "bing": true,
      // => "boom": "Hello"
    // => }
  // => }
// =>}"
```

## 17.数组索引
`JavaScript`数组的索引是基于0的32位数值，第一个元素索引为0，数组最大能容纳4294967295（即2^32-1）个元素。
```javascript
new Array(2**32)//报错

new Array(2**32-1)
//(4294967295) [empty × 4294967295]
```








