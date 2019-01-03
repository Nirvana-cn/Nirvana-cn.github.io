---
title: 'font-size:0的妙用'
date: 2017-10-25 16:23:22
tags:
- css
- font-size
categories:
- 前端,每天学一点
---

记录平时页面中令人困扰的元素空白间距问题出处，通过使用`font-size:0`消除空白间距。

<!--more-->

## 1. 为什么要设置 font-size：0

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>12</title>
    <style>
        *{
            margin:0;
            padding:0;
        }
        .content{
            background-color: #ccc;
            margin:20px;
            /*font-size: 0;*/
        }
        .title{
            display: inline-block;
            background-color: #afa;
            width:80px;
            height:80px;
        }
    </style>
</head>
<body>
<div class="content">
    <div class="title"></div>
    <div class="title"></div>
</div>
</body>
</html>
```
![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p03.png)

`.title`类并没有设置任何`margin`，但是两个`div`并没有紧紧贴合在一起，这是出于排版的原因两个`div`之间留下了空白字符，才导致两个`div`元素之间留有间距

## 2. 消除空白间距

虽然不换行可以暂时解决这一问题，但是肯定有更好的方法。在父类中设置`font-size:0`就可以完美地解决这个空白间距问题。

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p04.png)

## 3. 更多

本文的代码地址：[>>>点我进入](https://github.com/Nirvana-cn/WebTechnology/blob/master/HTML/font-size%E9%97%AE%E9%A2%98.html)