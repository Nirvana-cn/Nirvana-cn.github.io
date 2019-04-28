---
title: 浏览器选择复制知多少
date: 2019-04-26 10:59:08
tags:
- html
- javascript
categories:
- 前端,每天学一点
---

浏览器里选择复制你还是只知道`ctrl+A`和`ctrl+C`，你就`out`啦。这里有一份很详细的浏览器文本选择复制操作介绍🥳🥳🥳。

<!-- more -->

## 1. 简单的练手需求

现在有一个`div`元素，点击复制按钮后我希望复制`div`元素里的内容。

```html
<div id="target">我想被复制</div>
<button onclick="copy()">复制div里的内容</button>
<script>
    var copyText = '';
    window.addEventListener('copy', function (e) {
        e.preventDefault();
        e.clipboardData.setData('text/plain', copyText);
    });

    function copy() {
        copyText = document.querySelector('#target').innerHTML;
        document.execCommand('copy')
    }
</script>
```

对于`input`，`textarea`这类本身就可编辑的元素而言，直接取元素的`value`值即可。

## 2. 进阶需求

现在我们希望使用`JavaScript`复制指定区域的内容，并且保留浏览器渲染引擎渲染的样式。

```html
<div id="target">
    <div>title is here</div>
    <ul>
        <li>aaa</li>
        <li>bbb</li>
        <li>ccc</li>
    </ul>
</div>
<div>
    <p>我不想被选中</p>
</div>
<div>
    <button onclick="copy()" id="copy">复制</button>
</div>
<script>
    var copyText = '';
    window.addEventListener('copy', function (e) {
        e.preventDefault();
        e.clipboardData.setData('text/plain', copyText);
    });

    function copy() {
        copyText = document.querySelector('#target').innerHTML;
        document.execCommand('copy')
    }
</script>
```
在这样的情况，我们粘贴出来的就是`html`文本，如果使用`innerText`，那么复制出来的就是纯文本，会丢失`ul`的样式。

我们希望看到的效果：

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p38.png)

## 3. createRange

这种情况下，我们就必须老老实实得进行复制的流程，首先选中需要复制的内容，然后进行复制操作，将选中的内容放到剪贴板上。选中文本的过程就必须使用`createRange`方法了。

```html
<div id="target" contenteditable>
    <div>title is here</div>
    <ul>
        <li>aaa</li>
        <li>bbb</li>
        <li>ccc</li>
    </ul>
</div>
<div>
    <p>我不想被选中</p>
</div>
<div>
    <button onclick="func1()" id="choose">选择</button>
    <button onclick="func2()" id="copy">复制</button>
</div>
</body>
<script>
    function func1() {
        selectText('target')
    }

    function func2() {
        document.execCommand('copy')
    }

    function selectText(containerid) {
        var range;
        if (document.selection) {
            range = document.body.createTextRange();
            range.moveToElementText(document.getElementById(containerid));
            range.select();
        } else if (window.getSelection) {
            range = document.createRange();
            range.selectNodeContents(document.getElementById(containerid));
            window.getSelection().removeAllRanges();
            window.getSelection().addRange(range);
        }
    }
</script>
```

`contenteditable`属性可以使得`div`元素处于可编辑状态，这种情况下，在`#target`元素中使用`ctrl+A`时也不会选中页面中的其它内容，同样也适合那些复制之前需要编辑更改的内容。

> IE9以下支持：document.selection

> IE9、Firefox、Safari、Chrome和Opera支持：window.getSelection()

## 4. 选择指定长度的文本内容

在上面的范例里，我们使用了`selectNodeContents()`的方法，让我们可以选取整个指定的`DOM`。

除了全选，我们也可以使用`setStart()`以及`setEnd()`的方法，指定要选取的文字，不过因为这两个方法是针对「节点`node`」的操作，所以放在里面的，就是我们要选择节点，以及从这个节点起算的第几个字元。

```html
<div id="target" contenteditable>
    <div id="start">title is here</div>
    <ul>
        <li>aaa</li>
        <li id="end">bbb</li>
        <li>ccc</li>
    </ul>
</div>
<div>
    <p>我不想被选中</p>
</div>
<div>
    <button onclick="func1()" id="choose">选择</button>
    <button onclick="func2()" id="copy">复制</button>
</div>
</body>
<script>
    function func1() {
        selectText()
    }

    function func2() {
        document.execCommand('copy')
    }

    function selectText() {
        var range;
        if (window.getSelection) {
            range = document.createRange();
            // 注意setStart和setEnd的参数
            range.setStart(document.querySelector('#start').firstChild, 0);
            range.setEnd(document.querySelector('#end').firstChild, 3);
            window.getSelection().removeAllRanges();
            window.getSelection().addRange(range);
        }
    }
</script>
```

除了纯粹的选取之外，我们也可以使用`cloneContents()`方法来同时复制并贴上，使用`deleteContents()`来删除，使用`extractContents()`同时剪下并贴上。

## 5. 更多

本文源码地址：[>>>点我进入](https://github.com/Nirvana-cn/WebTechnology/tree/master/JS/selectText)

有contenteditable属性的容器，单击后选中全部文本：[>>>点我进入](https://segmentfault.com/q/1010000007857595?_ea=1474484)

教你如何选中元素内的所有文本内容：[>>>点我进入](https://segmentfault.com/a/1190000012316525)

selectNode和selectNodeContents的区别：[>>>点我进入](https://blog.csdn.net/yana_loo/article/details/51487412)

自動選取某個區域的文字：[>>>点我进入](https://www.oxxostudio.tw/articles/201508/select-text.html)

MDN参考文档 - Range：[>>>点我进入](https://developer.mozilla.org/zh-CN/docs/Web/API/Range)