---
title: 浏览器选择复制操作
date: 2019-04-26 10:59:08
tags:
- html
- javascript
categories:
- 前端,每天学一点
---

浏览器里选择复制你还是只知道`ctrl+A`和`ctrl+C`，你就`out`啦。这里有一份很详细的浏览器文本选择复制操作介绍。

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

对于`input`，`textarea`这类本身就可编辑的元素而言，直接去元素的`value`值即可。

## 2. 进阶需求
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

## 4. 选择指定长度的文本内容



## 5. 更多

本文源码地址：[>>>点我进入](https://github.com/Nirvana-cn/WebTechnology/tree/master/JS/selectText)

有contenteditable属性的容器，单击后选中全部文本：[>>>点我进入](https://segmentfault.com/q/1010000007857595?_ea=1474484)

教你如何选中元素内的所有文本内容：[>>>点我进入](https://segmentfault.com/a/1190000012316525)

selectNode和selectNodeContents的区别：[>>>点我进入](https://blog.csdn.net/yana_loo/article/details/51487412)