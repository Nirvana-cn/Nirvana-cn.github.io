---
title: 'font-size:0的妙用'
date: 2017-10-25 16:23:22
tags:
- CSS
categories:
- Web前端
---
使用font-size:0消除空白间距
<!--more-->
# 1.为什么要设置font-size：0

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
![](http://112.74.18.120:3001/p03.png)

.title类并没有设置任何margin，但是两个div并没有紧紧贴合在一起，这是出于排版的原因两个div之间留下了空白字符，才导致两个div元素之间留有间距

# 2.消除空白间距

虽然不换行可以暂时解决这一问题，但是肯定有更好的方法。在父类中设置font-size:0就可以完美地解决这个空白间距问题。 

![](http://112.74.18.120:3001/p04.png)
