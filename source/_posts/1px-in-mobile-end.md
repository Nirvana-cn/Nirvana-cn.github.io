---
title: 移动端1px边框的实现
date: 2017-10-21 16:32:57
tags:
- css
- 1px
categories:
- 前端,每天学一点
---
介绍移动端无法实现`1px`边框的原因以及模拟实现`1px`边框的方法。

<!--more-->

## 1.为什么出不来1px？

简而言之：`css`的`1px`只是一个抽象的单位，并非实际设备中的`1px`。

关于`retina`屏： 

我们知道现在`iphone`大多数型号都用上了`retina`屏，而`retina`屏的分辨率相较于普通屏幕增加了一倍，也就是说原来1个像素宽度的区域内可以塞进2个像素了。我们`css`写的`1px`是一个概念像素，在`retina`屏的实际设备上占了`2px`的位置。

而对于手机屏幕整体来说，一个标注了750宽的手机（iPhone6）在`css`中只需要`375px`就能表示。

## 2.如何在手机上写出1px

网上其实有人列了非常多的方案，有用`transform`的、有用图片的、有用`canvas`的、还有用`0.5px`的……从操作简易性来看，用`transform`的方案是比较简单的，而且适配也比较容易（`0.5px`的方案安卓不支持）。

原理：写一条`1px`的线，然后 `transform:scaleY(0.5)`或`scaleX(0.5)`，就能够将`retina上`显示的`2px`缩小为实际屏幕中的`1px`。

## 3.微信WeUI上1px边框的实现代码

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
说明：在需要`1px`边框的元素上使用伪类选择器（`：after，：before`）生成一个内容为空，高度为`1px`的元素，然后进行绝对定位，最后通过`transform：scaleY(0.5)`对Y轴进行缩放。如果需要`1px`的左右边框，将`width`设置为`1px`，`height`设置为`100%`，边框选择`border-left`或`border-right`，对X轴进行缩放即可。

## 4.使用min-device-pixel-ratio媒体功能自主查询

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

## 5. 更多

设备像素比devicePixelRatio简单介绍：[>>>点我进入](https://www.zhangxinxu.com/wordpress/2012/08/window-devicepixelratio/)

