---
title: JavaScript中的in操作符
date: 2018-03-09 15:26:17
tags:
- javascript
categories:
- 前端,每天学一点
---
总结`JavaScript`中的`in`操作符
<!--more-->

本文转载自[淡水河桥](http://blog.csdn.net/xufeiayang/article/details/52727906)

`in` 操作检查对象中是否有名为 `property` 的属性。也可以检查对象的原型，以便知道该属性是否为原型链的一部分。

1. 对于一般的对象属性需要用字符串指定属性的名称
```javascript
var mycar = {make: "Honda", model: "Accord", year: 1998};
"make" in mycar  // returns true
"model" in mycar // returns true
```
2. 对于数组属性需要指定数字形式的索引值来表示数组的属性名称（固有属性除外，如`length`）
```javascript
var trees = new Array("redwood", "bay", "cedar", "oak", "maple");
0 in trees        // returns true
3 in trees        // returns true
6 in trees        // returns false
"bay" in trees    // returns false (you must specify the index number,
                  // not the value at that index)
"length" in trees // returns true (length is an Array property)
```
3. `in`的右边必须是一个对象，如：你可以指定一个用`String`构造器生成的，但是不能指定字符串直接量的形式
```javascript
var color1 = new String("green");
"length" in color1 // returns true
var color2 = "coral";
"length" in color2 // generates an error (color is not a String object)
```
4. 如果你使用`delete`操作符删除了一个属性，再次用`in`检查时，会返回`false`
```javascript
var mycar = {make: "Honda", model: "Accord", year: 1998};
delete mycar.make;
"make" in mycar;  // returns false
  
var trees = new Array("redwood", "bay", "cedar", "oak", "maple");
delete trees[3];
3 in trees; // returns false
```