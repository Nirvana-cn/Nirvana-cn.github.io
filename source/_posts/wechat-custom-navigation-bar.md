---
title: 微信小程序自定义导航栏
date: 2019-12-15 16:28:34
tags:
- 微信小程序
categories:
- 小程序
---

还在为微信小程序中Android、IOS上导航栏title的展示不一致，安卓title显示不居中等问题而烦恼吗？试试小程序自定义导航栏吧🦉 🦉 🦉。

<!--more-->

## 1. 准备工作

首先，我们需要取得小程序导航栏的控制权，禁用小程序默认的导航栏。小程序自定义导航栏只需要在app.json文件的window项中配置

```
"navigationStyle": "custom"
```

Tips: 现在小程序也支持每一个页面单独自定义导航栏，只需要在页面的json配置文件中进行如上设置即可。

获取自定义导航栏以后，页面布局将从整个屏幕的左上角开始，微信只保留右上角的胶囊，如下图所示：

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/wechat/no-navigation-bar.png)

一切准备就绪，接下来我们就要开始干正事了。

## 2. 布局结构

由于页面布局将从整个屏幕的左上角开始，所以需要我们占位填充的实际上包括两部分，一是状态栏，二是导航栏。如下图所示：

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/wechat/has-navigation-bar.png)

因此，我们导航栏组件的页面布局设计如下：

```
<view class="m-header">
  <!-- 状态栏 -->
  <view class="status-bar" style="height:{{statusBarHeight}}px"></view>
  <!-- 导航栏 -->
  <view class="nav" style="height:{{titleHeight}}px">
    <view class="nav-user" bindtap="openUserCenter">用户中心</view>
    <view class="nav-title">{{title}}</view>
    <view class="nav-user nav-user_invisible">用户中心</view>
  </view>
</view>
<!-- 占位元素 -->
<view style="width:100%;height:{{totalHeight}}px;text-align:center;background-color:#faa">下拉刷新</view>
```

其中，"nav-user nav-user_invisible"类是为了方便布局，而生成的对称元素。同时为了防止拖动时或者下拉刷新时导航栏移动，m-header类将被设计为`position: fixed;`。由于固定定位时元素会脱离文档流，所以需要一个与自定义导航栏高度相同的元素进行占位，使得其它元素不会与自定义导航栏重叠，这也就是占位元素存在的意义。

## 3. 高度计算

第2部分确定页面布局之后，就到了最重要的步骤：计算高度。由于每种机型状态栏和导航栏不一致，所以这里不能写死，而且出于美观的角度考虑，我们还必须使得微信预留的气泡在导航栏的竖直方向居中。

状态栏的高度可以通过微信小程序提供的接口`wx.getSystemInfoSync().statusBarHeight`拿到。而导航栏的高度则相对麻烦一些，需要根据不同机型进行适配，这里通常我们在Android上取48px，在IOS上取44px。当然，我们可以在组件初始化时为特定机型设置指定的高度（不同机型的高度取舍，这里不再详细说明，可以参考文末的参考链接）。

```
const app = getApp();
const DEFAULT_ANDROID_HEIGHT = '48';
const DEFAULT_APPLE_HEIGHT = '44';

Component({
  properties: {
    title: {
      type: String,
      value: '标题'
    }
  },

  data: {
    desc: '用户中心',
    statusBarHeight: '',
    titleHeight: '',
    totalHeight: ''
  },

  methods: {
    isApplePhone: function(model) {
      return model.includes('iPhone');
    },
  },

  lifetimes: {
    attached: function() {
      let systemInfo = wx.getSystemInfoSync();
      let statusBarHeight = systemInfo.statusBarHeight;
      let titleHeight = DEFAULT_ANDROID_HEIGHT;
      if (this.isApplePhone(systemInfo.model)) {
        titleHeight = DEFAULT_APPLE_HEIGHT;
      }
      this.setData({
        statusBarHeight,
        titleHeight,
        totalHeight: Number.parseInt(statusBarHeight) + Number.parseInt(titleHeight)
      });
    }
  }
});
```

## 4. 自定义导航内容

到此为止，小程序自定义导航栏基本完成了。至于自定义导航栏的功能完全是可配的。

我们可以通过`getCurrentPages`接口获取当前路由栈的层数来确定导航栏展示哪一项具体的功能。比如，如果当前只有一层路由栈，则导航栏显示个人中心的按钮；如果大于一层路由栈，则导航栏显示返回和主页两个按钮。

## 5. 更多

微信小程序自定义导航栏：[>>>点我进入](https://juejin.im/post/5ca563f8f265da307a16133b)

小程序自定义导航栏适配：[>>>点我进入](https://segmentfault.com/a/1190000018733860)

本文Demo代码仓库地址：[>>>点我进入](https://github.com/Nirvana-cn/wechat-navigation-bar)
