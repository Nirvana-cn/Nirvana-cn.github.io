---
title: 移动端1px边框的实现
date: 2017-10-21 16:32:57
tags:
- 移动开发
categories:
- Web前端
---
在移动端实现1px边框的方法
<!--more-->
# 1.为什么出不来1px？
简而言之：css的1px只是一个抽象的单位，并非实际设备中的1px。

关于retina屏： 
我们知道现在iphone大多数型号都用上了retina屏，而retina屏的分辨率相较于普通屏幕增加了一倍，也就是说原来1个像素宽度的区域内可以塞进2个像素了。我们css写的1px是一个概念像素，在retina屏的实际设备上占了2px的位置。

而对于手机屏幕整体来说，一个标注了750宽的手机（iPhone6）在css中只需要375px就能表示。
# 2.如何在手机上写出1px
网上其实有人列了非常多的方案，有用transform的、有用图片的、有用canvas的、还有用0.5px的……从操作简易性来看，用transform的方案是比较简单的，而且适配也比较容易（0.5px的方案安卓不支持）。

原理：写一条1px的线，然后 transform:scaleY(0.5)或scaleX(0.5)，就能够将retina上显示的2px缩小为实际屏幕中的1px。
# 3.微信WeUI上1px边框的实现代码
```css
.weui_grid::after {
    content: " ";
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    height: 1px;
    border-bottom: 1px solid #D9D9D9;
    color: #D9D9D9;
    -webkit-transform-origin: 0 100%;
    transform-origin: 0 100%;
    -webkit-transform: scaleY(0.5);
    transform: scaleY(0.5);
}
```
说明：在需要1px边框的元素上使用伪类选择器（：after，：before）生成一个内容为空，高度为1px的元素，然后进行绝对定位，最后通过transform：scaleY（0.5）对Y轴进行缩放。如果需要1px的左右边框，将width设置为1px，height设置为100%，边框选择border-left或border-right，对X轴进行缩放即可。
# 4.使用min-device-pixel-ratio媒体功能自主查询
```css
.border-1px::after{
  content: " ";
  position: absolute;
  top:50px;
  left:0;
  width: 100%;
  height: 1px;
  border-bottom: 1px solid #000;
}
@media (-webkit-min-device-pixel-ratio:2),(min-device-pixel-ratio:2){
    .border-1px::after{
        -webkit-transform: scaleY(0.5);
        transform: scaleY(0.5);
    }
}
```

