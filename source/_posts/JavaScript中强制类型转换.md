---
title: JavaScript中的强制类型转换
date: 2018-06-01 16:28:34
tags:
- JavaScript
categories:
- Web前端
---

总结和剖析JavaScript中的强制类型转换，主要参考《你不知道的JavaScript(中卷)》第四章。

<!--more-->

# 1.抽象值操作

## 1.1 ToString

抽象操作ToString负责处理非字符串到字符串的强制类型转换。

基本类型值的字符串化规则为：null转换为"null",undefined转换为"undefined",true转换为""true"。数字的字符串化遵循通用规则，那些极小和极大的数字使用指数形式：

```
var a = 1.07*1000*1000*1000*1000*1000*1000*1000
a.toString()    //"1.07e21"
```

对普通对象来说，除非自行定义，否则toString()返回内部属性[[Class]]的值，如"[object Object]"。

数组的默认toString()方法经过了重新定义，将所有单元字符串化以后再用","连接起来：

```
var a = [1,2,3]
a.toString()    //"1,2,3"
```

## 1.2 ToNumber

抽象操作ToNumber将非数字值转换为数字值。

其中true转换为1，false转换为0，undefined转换为NaN,null转换为0。

ToNumber对字符串的处理基本遵循数字常量的相关规则(字符串中含有非数字类型字符返回NaN)。

对象(包括数组)会首先被转换为相应的基本类型值，如果返回的是非数字的基本类型值，则再遵循以上规则将其强制转换为数字。

## 1.3 ToPrimitive

抽象方法ToPrimitive将对象值转换为相应的基本类型值。该方法会首先检查该值是否有valueOf()方法，如果有并且返回基本类型值，就使用该值进行强制类型转换；如果没有就使用toString()的返回值(如果存在)来进行强制类型转换；如果valueOf()和toString()均不返回基本类型值，会产生TypeError错误。

从ES5开始，使用Object.create(null)创建的对象原型属性为null，并且没有valueOf()和toString()方法，因此无法进行强制类型转换。

## 1.4 ToBoolean

抽象操作ToBoolean将非布尔值转换为布尔值。

### 1.4.1 假值

假值的布尔强制类型转换结果为false。

以下这些是假值：
- undefined
- null
- false
- +0、-0和NaN
- ""

从逻辑上说，假值列表以外的都应该是真值，但是JavaScript规范对此并没有明确定义，只是给出了一些实例，例如规定所有的对象都是真值。

### 1.4.2 假值对象

例如：

```
var a = new Boolean(false)
var b = new Boolean(0)
var c = new Boolean("")

var d = Boolean(a && b && c)    //true
```

a,b,c都是封装了假值的对象，但是d为true，说明a、b、c都为true。因此假值对象并非封装了假值的对象。

假值对象看起来和普通对象并无二致，但将它们强制类型转换为布尔值时结果为false。最常见的例子是document.all，它是一个类数组对象，包含了页面上所有元素，它以前曾是一个真正意义上的对象，布尔强制类型转换结果为true，不过现在它是一个假值对象。

### 1.4.3 真值

真值就是假值列表之外的值。真值列表可以无限长，无法一一列举，所以只能用假值列表作为参考。

# 2.显示强制类型转换

## 2.1 字符串和数字之间的显示转换

字符串和数字之间的转换是通过String()和Number()这两个内建函数来实现的，请注意它们前面没有new关键字，并不创建封装对象。

String()遵循前面讲过的ToString规则，将值转换为字符串基本类型。Number()遵循前面讲过的ToNumber规则，将值转换为数字基本类型。

除了String()和Number()以外，还有其他方法可以实现字符串和数字之间的显示转换：

```
var a = 42
var b = a.toSting() //"42"

var c = "3.14"
var d = +c  //3.14
```

a.toString()是显式的，不过其中涉及隐式转换。因为toString()对42这样的基本类型值并不适用，所以JavaScript引擎会自动为42创建一个封装对象，然后对该对象调用toString()。

上例中+c是+运算符的一元形式(即只有一个操作数)。+运算符显式地将c转换为数字，而非数字加法运算(也不是字符串拼接)。

一元运算符 - 和 + 一样，并且还会反转数字的符号位。由于 -- 会被当作递减运算符来处理，所以我们不能使用 -- 来撤销反转，而应该像 - -"3.14"这样，在中间加一个空格。

尽量不要把一元运算符 + (还有 - )和其他运算符放在一起使用。此外d = +c也容易和d += c搞混，两者天壤之别。

### 2.1.1 日期显示转换为数字

一元运算符 + 的另一个常见用途是将日期(Date)对象强制类型转换为数字，返回结果为Unix时间戳。

```
var time = new Date()
+time
```

### 2.1.2 奇特的~运算符

~首先将值强制类型转换为32位数字，然后执行字位操作“非”(对每一个字位进行反转)。

字位反转是个很晦涩的主题，JavaScript开发人员一般很少需要关心到字位级别。

对~还可以有另外一种诠释：返回2的补码！

所以 ~x 大致等同于 -(x+1)。

```
~42 //-(42+1) ==> -43
```

在-(x+1)中唯一能够得到0(或者严格说是-0)的x值是-1，而在JavaScript中有些函数用-1来代表执行失败，用大于等于0的值来代表函数执行成功。

比如，indexOf()方法在字符串中搜索指定的字符串，如果找到就返回子字符串的位置，否则返回-1。

```
var a = "Hello World"
if(a.indexOf("lo") != -1){
    // 找到匹配
}

if(a.indexOf("ol") == -1){
    // 没有找到匹配
}
```

~和indexOf()一起可以将结果强制类型转换为真/假值，如果indexOf()返回-1，~将其转换为假值0，其他情况一律转换为真值。

```
var a = "Hello World"
~a.indexOf("lo")    // -4 ==>真值
if(~a.indexOf("lo")){
    // 找到匹配
}
~a.indexOf("ol")   // 0 ==>假值 
if(!~a.indexOf("ol")){
    // 没有找到匹配
}
```

### 2.1.3 字位截除

~~x能将值截除为一个32为整数，\~\~中的的第一个~执行ToInt32并反转字位，然后第二个~再进行一次字位反转，即将所有字位反转回原值，最后得到的仍然是ToInt32的结果。

首先~~只适用于32位数字，更重要的是它对负数的处理与Math.floor()不同。

```
Math.floor(-49.6)   // -50
~~-49.6 //-49
```

## 2.2 显式解析数字字符串

解析字符串中的数字和将字符串强制类型转换为数字的返回结果都是数字。但是解析和转换两者之间还是有明显的差别。比如：

```
var a = "42"
var b = "42px"

Number(a)   //42
parseInt(a) //42

Number(b)   //NaN
parseInt(b) //42
```

解析允许字符串中含有非数字字符，解析按从左到右的顺序，如果遇到非数字字符就停止。而转换不允许出现非数字字符，否则会失败返回NaN。

解析字符串中的浮点数可以使用parseFloat()函数。从ES5开始parseInt()默认转换为十进制数，除非指定第二个参数作为基数。

不要忘了parseInt()针对的是字符串，向parseInt()传递数字和其他类型的参数是没有用的。非字符串会首先被强制类型转换为字符串，应该避免向parseInt()传递非字符串参数。

```
parseInt(1/0,19)    //18
```

parseInt(1/0,19) 最后的结果是18，而非报错，因为parseInt(1/0,19)实际上是parseInt("Infinity",19)。基数19，它的有效数字字符范围是0-9和a-i(区分大小写)，以19为基数时，第一个字符"I"值为18，而第二个字符"n"不是一个有效的数字字符，解析到此为止，和"42px"中"p"一样。

## 2.3 显式转换为布尔值

和前面讲过的+类型，一元运算符!显式地将值强制类型转换为布尔值。但是它同时还将真值反转为假值。所以显式强制类型转换为布尔值最常用地方法是!!。

在if()这样的布尔值上下文中，建议使用Boolean()和!!来进行显式转换以便让代码更清晰易读。

# 3. 隐式强制类型转换

## 3.1 字符串和数字之间的隐式强制类型转换

ES5规范中定义：如果某个操作数是字符串或者能够通过以下步骤转换为字符串的话，+将进行拼接操作。如果其中一个操作数是对象(包括数组),则首先对其调用ToPrimitive抽象操作，该抽象操作再调用[[DefaultValue]]，以数字作为上下文。

简单来说就是，如果+的其中一个操作数是字符串(或者通过以上步骤可以得到字符串)，那么就执行字符串拼接，否则执行数字加法。

```
var a = [1,2]
var b = [3,4]
a + b   //"1,23,4"
```

因为数组的valueOf()操作无法得到简单基本类型值，于是调用toString()，因此两个数组变成了"1,2"和"3,4"，+将它们拼接后返回。

a + ""(隐式)和前面的String(a)(显式)之间有一个细微的差别需要注意。根据ToPrimitive抽象操作规则，a + ""会对a调用valueOf()方法，然后通过ToString抽象操作将返回值转换为字符串，而String(a)则是直接调用toString()方法。

## 3.2 布尔值到数字的隐式强制类型转换

如果其中有且仅有一个参数为true，则onlyOne()返回true。

```javascript
function onlyOne() {
    var sum = 0
    for (var i=0;i<arguments.length;i++){
        if(arguments[i]){
            sum += arguments[i]
        }
    }
    return sum == 1
}

var a = true
var b = false
onlyOne(b,a,b,b,b,b)    // true
```

无论使用隐式转换还是显式转换，我们都可以通过修改onlyTwo()或者onlyFive()来处理更加复杂的情况，只需要将最后的条件判断从改为2或5。这比加入一大堆&&和||表达式要简洁得多。

## 3.3 隐式强制类型转换为布尔值

相对布尔值，数字和字符串操作中的隐式强制类型转换还算比较明显。下面的情况会发生布尔值隐式强制类型转换。

- if()语句中的条件判断表达式
- for(..; ..; ..)语句中的条件判断表达式
- while()和do .. while()
- ? : 中的条件判断表达式
- 逻辑运算符||和&&左边的操作数

## 3.4 ||和&&

其实不太赞同将它们称为“逻辑运算符”，因为这不太准确。称它们为“选择器运算符”或者“操作数选择器运算符”更恰当一些。

ES5规范中说到：&&和||运算符的返回值并不一定是布尔类型，而是两个操作数其中一个的值。

对于||来说，如果条件判断结果为true就返回第一个操作数的值，如果为false就返回第二个操作数的值。

对于&&来说，如果条件判断结果为true就返回第二个操作数的值，如果为false就返回第一个操作数的值。

** 这里的条件判断结果指的是左边的操作数。

下面是一个十分常见的||的用法，也许你已经用过但却并未完全理解：

```javascript
function foo(a,b) {
    a = a||"hello"
    b = b||"world"
    console.log(a + '' + b)
}
foo()   // "hello world"
```

再来看看&&，有一种用法开发人员不常见，然而JavaScript代码压缩工具常用。就是如果第一个操作数为真值，则&&运算符选择第二个操作数作为返回值，这也叫做“守护运算符”，即前面的表达式为后面的表达式把关。

```javascript
function foo() {
    console.log(a)
}
var a = 42 
a && foo()
```

## 3.5 符合的强制类型转换

ES6中引入了符合类型，它的强制类型转换有一个坑。ES6允许从符合到字符串的显式强制类型转换，然而隐式强制类型转换会产生错误，例如：

```javascript
var s1 = Symbol("cool")
String(s1)  // "Symbol(cool)"
var s2 = Symbol("not cool")
s2 + '' // TypeError
```

符合不能够被强制类型转换为数字(显式和隐式都会产生错误),但可以被强制类型转换为布尔值(显式和隐式都是true)。

# 4. 宽松相等和严格相等

常见的误区是“==检查值是否相等，===检查值和类型是否相等”，正确的解释是：“==允许在相等比较中进行强制类型转换，而===不允许”。事实上，==和===都会检查操作数的类型，区别在于操作数类型不同时它们的处理方式不同。

## 4.1 抽象相等

ES5规范11.9.3节的“抽象相等比较算法”定义了==运算符的行为。该算法简单而又全面，涵盖了所有可能出现的类型组合，以及它们进行强制类型转换的方式。

```
比较运算x==y, 其中x和 y是值，产生true或者false。这样的比较按如下方式进行：
	1. 若Type(x)与Type(y)相同， 则
		a. 若Type(x)为Undefined， 返回true。
		b. 若Type(x)为Null， 返回true。
		c. 若Type(x)为Number， 则
			i. 若x为NaN， 返回false。
			ii. 若y为NaN， 返回false。
			iii. 若x与y为相等数值， 返回true。
			iv. 若x 为 +0 且 y为−0， 返回true。
			v. 若x 为 −0 且 y为+0， 返回true。
			vi. 返回false。
		d. 若Type(x)为String, 则当x和y为完全相同的字符序列（长度相等且相同字符在相同位置）时返回true。 否则， 返回false。
		e. 若Type(x)为Boolean, 当x和y为同为true或者同为false时返回true。 否则， 返回false。
		f. 当x和y为引用同一对象时返回true。否则，返回false。
	2. 若x为null且y为undefined， 返回true。
	3. 若x为undefined且y为null， 返回true。
	4. 若Type(x) 为 Number 且 Type(y)为String， 返回comparison x == ToNumber(y)的结果。
	5. 若Type(x) 为 String 且 Type(y)为Number，返回比较ToNumber(x) == y的结果。
	6. 若Type(x)为Boolean， 返回比较ToNumber(x) == y的结果。
	7. 若Type(y)为Boolean， 返回比较x == ToNumber(y)的结果。
	8. 若Type(x)为String或Number，且Type(y)为Object，返回比较x == ToPrimitive(y)的结果。
	9. 若Type(x)为Object且Type(y)为String或Number， 返回比较ToPrimitive(x) == y的结果。
       10. 返回false。
```

### 4.1.1 字符串和数字之间的相等比较

```javascript
var a = 42
var b = '42'
a == b  //true
```

a==b是宽松相等，即如果两个值的类型不同，则对其中之一或两者都进行强制类型转换。具体怎么转换？这就需要匹配前文的“抽象相等比较算法”，寻找适应的转换规则。

根据第4条规则返回x == ToNumber(y)的结果。

### 4.1.2 其他类型和布尔类型之间的相等比较

==最容易出错的一个地方是true和false与其他类型之间的相等比较。

```javascript
var a = '42'
var b = true
a == b  //false
```

结果是false，这让人很容易掉坑里。如果严格按照“抽象相等比较算法”，这个结果也就是意料之中的。

根据第7条规则，若Type(y)为Boolean， 返回比较x == ToNumber(y)的结果，即返回'42' == 1，结果为false。

很奇怪吧？所以无论什么情况下都不要使用== true和== false。

### 4.1.3 null和undefined之间的相等比较

在==中null和undefined相等，这也就是说在==中null和undefined是一回事，可以相互进行隐式强制类型转换。

** 掌握“抽象相等比较算法”，读者可以自行推倒为什么[]==![]返回true。

## 4.2 比较少见的情况

```javascript
if(a == 2 && a == 3){
    //...    
}
```

你也许觉得这不可能，因为a不会同时等于2和3。但如果让a.valueOf()每次调用都产生副作用，比如第一次返回2，第二次返回3，就会出现这样的情况。

```javascript
var i = 2
Number.prototype.valueOf = function() {
  return i++
}
var a = new Number(42)
if(a == 2 && a == 3){
    console.log('Yeah, it happened!')
}
```

还有一个坑常常被提到：

```javascript
0 == '\n'   //true
```

""、"\n"(或者" "等其他空格组合)等空字符串被ToNumber强制类型转换为0。

## 4.3 完整性检查

再来看看那些“短”的地方：

```javascript
"0" == false    // true
false == 0      // true
false == ""     // true
false == []     // true
"" == 0          // true
"" == []         // true
0 == []          // true
```

其中有4种情况涉及== false，之前我们说过应该避免，所以还剩下后面3种。

这些特殊情况会导致各种问题，使用中要多加小心。我们要对==两边的值认真推敲，以下两个原则可以让我们有效地避免出错。

- 如果两边的值中有true或者false，千万不要使用==
- 如果两边的值中有[]、""、或者0，尽量不要使用==

隐式强制转换在部分情况下确实很危险，为了安全起见就要使用===

# 5. 抽象关系比较

以 x 和 y 为值进行小于比较（x < y 的比较），会产生的结果可为="" true，false或 undefined（这说明 x、y 中最少有一个操作数是 NaN）。除了 x 和 y，这个算法另外需要一个名为 LeftFirst 的布尔值标记作为参数。这个标记用于解析顺序的控制，因为操作数 x 和 y 在执行的时候会有潜在可见的副作用。LeftFirst 标志是必须的，因为 ECMAScript 规定了表达式是从左到右顺序执行的。LeftFirst 的默认值是 true，这表明在相关的表达式中，参数 x 出现在参数 y 之前。如果 LeftFirst 值是 false，情况会相反，操作数的执行必须是先 y 后 x。这样的一个小于比较的执行步骤如下：

```
	1. 如果 LeftFirst 标志是 true，那么
		a. 让 px 为调用 ToPrimitive(x, hint Number) 的结果。
		b. 让 py 为调用 ToPrimitive(y, hint Number) 的结果。
	2. 否则解释执行的顺序需要反转，从而保证从左到右的执行顺序
		a. 让 py 为调用 ToPrimitive(y, hint Number) 的结果。
		b. 让 px 为调用 ToPrimitive(x, hint Number) 的结果。
	3. 如果 Type(px) 和 Type(py) 得到的结果不都是 String 类型，那么
		a. 让 nx 为调用 ToNumber(px) 的结果。因为 px 和 py 都已经是基本数据类型（primitive values 也作原始值），其执行顺序并不重要。
		b. 让 ny 为调用 ToNumber(py) 的结果。
		c. 如果 nx 是 NaN，返回 undefined
		d. 如果 ny 是 NaN，返回 undefined
		e. 如果 nx 和 ny 的数字值相同，返回 false
		f. 如果 nx 是 +0 且 ny 是 -0，返回 flase
		g. 如果 nx 是 -0 且 ny 是 +0，返回 false
		h. 如果 nx 是 +∞，返回 fasle
		i. 如果 ny 是 +∞，返回 true
		j. 如果 ny 是 -∞，返回 flase
		k. 如果 nx 是 -∞，返回 true
		l. 如果 nx 数学上的值小于 ny 数学上的值（注意这些数学值都不能是无限的且不能都为 0），返回 ture。否则返回 false。
	4. 否则，px 和 py 都是 Strings 类型
		a. 如果 py 是 px 的一个前缀，返回 false。（当字符串 q 的值可以是字符串 p 和一个其他的字符串 r 拼接而成时，字符串 p 就是 q 的前缀。注意：任何字符串都是自己的前缀，因为 r 可能是空字符串。）
		b. 如果 px 是 py 的前缀，返回 true。
		c. 让 k 成为最小的非负整数，能使得在 px 字符串中位置 k 的字符与字符串 py 字符串中位置 k 的字符不相同。（这里必须有一个 k，使得互相都不是对方的前缀）
		d. 让 m 成为字符串 px 中位置 k 的字符的编码单元值。
		e. 让 n 成为字符串 py 中位置 k 的字符的编码单元值。
                f.如果 n<m，返回 true。否则，返回 false。
```

 下面的例子就有些奇怪了：
 
 ```javascript
var a = {b:42}
var b = {b:43}

a < b   // false
a == b  // false
a > b   // false

a <= b  // true
a >= b  // true
```

如果a < b和a == b结果为false，为什么a <= b和a >= b的结果会是true呢？

因为根据规范a <= b被处理为b < a，然后将结果反转。因为b < a的结果为false，所以a <= b的结果为true。

这可能与我们设想的大相径庭，即<=应该是“小于或者等于”，实际上，JavaScript中<=是“不大于”的意思，即a <= b被处理为 !(b < a)。

** 规范设定NaN既不大于也不小于任何其他值。