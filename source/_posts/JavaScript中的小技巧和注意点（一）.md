---
title: JavaScript中的小技巧和注意点（一）
date: 2017-07-17 16:16:15
tags:
- JavaScript
categories:
- Web前端
---
记录平时学习过程中关于JavaScript容易遗忘的知识点和需要注意的地方，点点滴滴贵在积累
<!--more-->
# 1.检查变量是否存在
```javascript
var result = "";
if (typeof somevar !== "undefined"){
    result = "yes";
}
```
在这种情况下，typeof 返回的是一个字符串（先执行 typeof 再比较），这样就可以与字符串"undefined"进行直接比对，如果 somevar 这个变量存在，则执行if条件语句。实际上是在用 typeof 测试一个变量是否已经被初始化（或者说测试变量值是否为 undefined ）
# 2.parseInt()
parseInt（）会试图将其收到的任何输入值（通常是字符串）转换成整数类型输出。如果转换失败就返回NaN。
```javascript
parseInt('123');
// 123
parseInt('1abc123');
// 1
parseInt('abc123');
// NaN
```
除此之外，该函数还有个可选的第二参数：基数（radix），它负责设定函数所期望的数字类型——十进制、十六进制、二进制。
```javascript
parseInt('FF',10);
// NaN

parseInt('FF',16);
// 255
```
在调用parseInt时没有指定第二参数，函数会将其默认为十进制，但有两种情况例外：
（1）如果首参数字符串是0x开头，第二参数就会被默认指定为16；
（2）如果首参数以0开头，第二参数就会被默认指定为8。
```javascript
parseInt('377');
// 377

parseInt('0377');
// 255

parseInt('0x377');
// 887
```
当然，为了保证程序的正确运行，明确指定radix值总是最安全的。值得一提的是，从ES5开始，在严格模式下，八进制数值就不再允许使用前缀0表示；ES6进一步明确，要使用前缀0o表示，并且将全局方法 parseInt（）和 parseFloat（）移植到了Number对象上，行为完全保持不变。当然在非严格模式下，前缀 0o 和 0 均可以表示八进制。
# 3.能重写自己的函数
由于一个函数可以返回另一个函数，所以我们可以让函数从内部重写自己，比如：
```javascript
function a(){
    alert('A!');
    a=function(){
        alert('B!');
    };
}
```
这项技术对于某些浏览器相关的操作会相当有用。因为在不同浏览器中，实现相同任务的技术可能是不同的，我们都知道浏览器的特性不可能因为函数调用而发生任何改变，因此，最好的选择就是让函数根据其当前所在的浏览器来重新定义自己，即"浏览器兼容性探测"技术。
# 4.函数对象的长度
函数对象中也有一个length属性，用于记录该函数声明时所决定的参数数量（ES6中函数参数可以指定默认值，指定了默认值以后函数length属性将返回没有指定默认值的参数的个数）。

arguments.length用于表示函数实际接受到的参数的个数，可用于模拟多态。
```javascript
function mufun(a,b,c){
    return arguments.length;
}

myfun.length
// 3

myfun(1,2)
// 2
```
# 5.call()和apply()
在JavaScript中，每个函数都有call()和apply()两个方法，它们可以让一个对象去"借用"另一个对象的方法，并为己所用。这也是一种非常简单而实用的代码重用。
```javascript
var some_obj = {
    name:'Jack',
    say:function(who){
        return 'Hi ' + who + ', I am ' + this.name;
    }
};
var my_obj = {
    name:'Rose'
}

some_obj.say.call(my_obj,'Obama')
// "Hi Obama, I am Rose"
```
当调用say()函数的对象方法call()时，this自动指向传入的对象。另外，如果没有将对象传递给call()的首参数，或者传递给它的是null，它的调用对象将会被默认为全局对象，即this指向的是全局对象。

apply()的工作方式与call()基本相同，唯一的不同之处在于参数的传递形式，apply()所需要的参数都是通过一个数组来传递。所以下面两行代码的作用是等效的。
```javascript
some_obj.say.call(my_obj,'Obama');

some_obj.say.apply(my_obj,['Obama']);
```
arguments看上去像是一个数组，但它实际上是一个类数组对象，它和数组相似是因为其中也包含了索引元素和length属性。但相似之处也就到此为止了，因为arguments不提供一些像sort()、slice()这样的数组方法。但我们可以通过call()调用数组的各种方法，而不用重新编写对应的代码。
```javascript
function fun(){
    var args = [].slice.call(arguments);
    return args.reverse();
}

fun(1,2,3,4);
// [4,3,2,1]
```
这里的做法是新建一个空数组 [ ] ，再使用它的slice方法，当然，也可以通过Array.prototype.slice来调用同一个函数。
call()和apply()的另一个用法就是推断对象类型。

在JS中是无法使用 typeof 区分对象和数组的，要想区分数组和对象，方法之一就是使用Object对象的toString()方法，这个方法会返回所创建对象的内部类名。
```javascript
Object.prototype.toString.call({});
// "[Object Object]"

Object.prototype.toString,call([]);
// "[Object Array]"

(function (){
  return Object.prototype.toString.call(arguments);
})();
// "[Object Arguments]"
```
在这里，toString()方法必须要来自于Object构造器的prototype属性。直接调用Array的toString()方法是不行的，因为在Array对象中，这个方法已经出于其他目的被重写了。
# 6.枚举属性
如果想获得某个对象所有属性的列表，我们可以使用for-in循环，但并不是所有的属性都会在for-in循环中显示。例如数组的length属性和constructor属性就不会被显示。那些会显示的属性被称为可枚举的，我们可以通过各个对象所提供的propertyIsEnumerable()方法来判断对象的某个属性是否可枚举。

在for-in循环中，原型链中的各个原型属性也会被显示出来，当然前提是它们是可枚举的。我们可以通过对象的hasOwnProperty()方法来判断一个属性是对象自身属性还是原型属性。

对于所有的原型属性，propertyIsEnumerable()都会返回false，包括那些在for-in循环中可枚举的属性。如果propertyIsEnumerable()的调用是来自原型链上的某个对象，那么该对象的属性是可枚举的。
# 7.原型陷阱
在处理原型问题时，需要特别注意以下两种行为

（1）对原型对象执行完全替换时，可能会触发原型链中某种异常；

（2）prototype.constructor属性是不可靠的。
```javascript
function Dog() {
  this.tail=true;
}
var benji = new Dog();

Dog.prototype.say=function () {
   return 'Woof!';
};
 
benji.say()
// "Woof!"

benji.constructor === Dog
// true
//用一个自定义的新对象完全覆盖掉原有的原型对象
Dog.prototype = {
   paws:4,
   hair:true
};
typeof benji.paws
// "undefined"

benji.say()
// "Woof!"

typeof benji.__proto__.say
// "function"

typeof benji.__proto__.paws
// "undefined"

var lucy = new Dog();
lucy.say()
// TypeError:lucy.say is not a function

lucy.paws
// 4
typeof lucy.__proto__.say
// "undefined"

typeof lucy.__proto__.paws
// "number"

lucy.constructor
// function Object(){[native code]}

benji.constructor
// function Dog(){this.tail=true;}

```
事实证明，这会使原有对象不能访问原型的新增属性，它们依然可以通过__proto__与原有的原型对象保持联系。而之后创建的所有对象使用的都是被更新后的prototype对象，并且__proto__也指向了新的prototype对象。但这时候，新对象的constructor属性就不能再保持正确了，原本应该是Dog()的引用却指向了Object()。所以当我们重写某对象的prototype时，需要重置相应的constructor属性。
```javascript
Dog.prototype.constructor = Dog
// true
new Dog().constructor === Dog
// true

lucy.constructor
// function Dog() {
//     this.tail=true;
// }
```