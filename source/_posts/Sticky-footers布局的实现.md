---
title: Sticky footers布局的实现
date: 2018-01-05 16:35:02
tags:
---
# 1.什么是sticky footers布局
在网页设计中，Sticky footers设计是最古老和最常见的效果之一，大多数人都曾经经历过。它可以概括如下：如果页面内容不够长的时候，页脚块粘贴在视窗底部；如果内容足够长时，页脚块会被内容向下推送。
# 2.Flexbox解决方案
我们需要给外层.wrapper元素设置flex弹性布局，主轴排列方向为竖直方向，设置min-height值为100vh，让.wrapper内容不足视窗高度时也能占据整个视窗，再给内部的每个盒子设置flex值，使其自动伸缩的来适配剩余空间。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Sticky footers</title>
    <style>
        *{
            margin:0;
        }
        body {
            font-size: 40px;
        }
        .wrap {
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }
        .top {
            flex: 4;
        }
        .bottom {
            flex: 1;
        }
        .box {
            background-color: #afa;
            width: 64px;
            height: 64px;
            margin: 0 auto;
        }
    </style>
</head>
<body>
<div class="wrap">
    <div class="top">
        <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
        <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
        <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
        <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
        <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
        <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
        <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
        <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
    </div>
    <div class="bottom">
        <div class="box"></div>
    </div>
</div>
</body>
</html>
```
# 3.固定高度的解决方案
.detail元素绝对定位，高度和宽度铺满整个屏幕，.detail-wrapper元素最小高度为100%，关键是设置.detail-main元素padding-bottom值以及.detail-close元素负的margin-top值
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Sticky footers</title>
    <style>
        *{
            margin:0;
        }
        .detail {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            overflow: auto;
        }
        .detail-wrapper {
            min-height: 100%;
        }
        .detail-main {
            font-size:40px;
            padding-bottom: 100px;
        }
        .detail-close {
            background-color:#afa;
            width: 64px;
            height: 64px;
            margin: -100px auto 0;
        }
    </style>
</head>
<body>
<div class="detail">
    <div class="detail-wrapper">
        <div class="detail-main">
            <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
            <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
            <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
            <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
            <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
            <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
            <p>这是一篇我自己关于sticky footers的理解，今天因为做项目用到了，几经找资料，终于搞懂了，就赶紧记下来，免得忘记了！</p>
        </div>
    </div>
    <div class="detail-close"></div>
</div>
</body>
</html>
```