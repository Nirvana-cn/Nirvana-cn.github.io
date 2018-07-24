---
title: 我理解的JS事件轮询机制
date: 2018-03-26 16:40:32
tags: 
- JavaScript
- Event Loop

categories:
- Web前端
---
JS是单线程语言，深入理解JS里的Event Loop

<!--more-->
![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1522036338964&di=1eea5b73d1da870b6ef4f05cd08f521a&imgtype=0&src=http%3A%2F%2Fstatic.codeceo.com%2Fimages%2F2015%2F09%2F6a5656453ef29f5120550fa98b53eae1.png)

### 划重点！！！

#### JS的执行机制(一)：

1.首先判断JS是同步还是异步,同步就进入主进程,异步就进入event table

2.异步任务在event table中注册函数,当满足触发条件后,被推入event queue

3.同步任务进入主线程后一直执行,直到主线程空闲时,才会去event queue中查看是否有可执行的异步任务,如果有就推入主进程中

#### JS的执行机制(二)

1.执行一个宏任务,过程中如果遇到微任务,就将其放到微任务的【事件队列】里

2.当前宏任务执行完成后,会查看微任务的【事件队列】,并将里面全部的微任务依次执行完

#### 任务划分方式:

1.macro-task(宏任务)：script，setTimeout，setInterval

2.micro-task(微任务)：Promise，process.nextTick

我们从一道小题目出发

```javascript
for (var i=0;i<=5;i++){
    setTimeout(()=>{console.log(i)},1000)
}
```

输出结果应该是1s之后连续打印6个6，虽然这个题目的主要知识点是块级作用域，但非常适合用来引出事件轮询机制。因为setTimeout是异步任务，所以不会立即执行，它需要等到所有的同步任务执行完毕，即for循环结束，当i变为6时开始同时执行6个定时器，此时i指向值为6的全局变量，所以打印6，这就是JS执行机制(一)

加点难度，考虑JS执行机制(二)

```javascript
// promise里面的函数是立即执行的
// 分别输出 2 3 5 4 1
setTimeout(function () {
    console.log(1)
},0);
new Promise(function executor(resolve) {
    console.log(2);
    for(var i=0;i<10000;i++){
        i==9999 && resolve();
    }
    console.log(3);
}).then(function () {
    console.log(4);
});
console.log(5);
```
首先执行的第一个宏任务肯定是脚本(script)，所以定时器会被跳过(不论你延迟几秒执行)，紧接着执行Promise里面的内容，按顺序执行，先打印2，然后进行for循环，resolve()是异步回调函数，属于异步执行的内容，同时如我们在任务划分里面提到的，Promise属于微任务，所以会在宏任务结束之后清空微任务事件队列，所以接下来会打印3，5，4。
至此第一个宏任务便处理完毕，然后才会轮到定时器，打印1

Node.js略有不同，在node.js启动时，创建了一个类似while(true)的循环体，每次执行一次循环体称为一次tick，每个tick的过程就是查看是否有事件等待处理，如果有，则取出事件极其相关的回调函数并执行，然后执行下一次tick。node的Event Loop和浏览器有所不同。Event Loop每次轮询：先执行完主代码，期中遇到异步代码会交给对应的队列，然后先执行完所有nextTick()，然后在执行其它所有微任务。

#### process.nextTick

node方法process.nextTick可以把当前任务添加到执行栈的尾部，也就是在下一次Event Loop（主线程读取"任务队列"）之前执行。也就是说，它指定的任务一定会发生在所有异步任务之前。

#### setImmediate

Node.js0.8以前是没有setImmediate的，在当前"任务队列"的尾部添加事件，官方称setImmediate指定的回调函数，类似于setTimeout(callback,0)，会将事件放到下一个事件循环中，所以也会比nextTick慢执行，有一点——需要了解setImmediate和nextTick的区别。nextTick虽然异步执行，但是不会给其他io事件执行的任何机会，而setImmediate是执行于下一个event loop。总之process.nextTick()的优先级高于setImmediate
