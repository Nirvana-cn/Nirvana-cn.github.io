---
title: PWA登陆iOS了，但它还有这些缺陷
date: 2018-04-08 10:28:34
tags:
- PWA
categories:
- Web前端
---

Apple 在 iOS 11.3 中悄悄加入了对“渐进式 Web 应用”（PWA）这一系列新技术的基本支持。是时候看看这些技术是如何生效的？它有什么能力？会遇到哪些挑战？以及如果已经发布了 PWA，又需要了解哪些事情？本文概括介绍了最新发布的 iOS 11.3 对 PWA 的支持情况，以及 PWA 应用开发者需要注意的问题。

<!--more-->

本文转载自前端之巅

作者 Maximiliano Firtman

译者 大愚若智

原文地址 https://mp.weixin.qq.com/s?__biz=MzUxMzcxMzE5Ng==&mid=2247488633&idx=1&sn=b579b5345216aba71b66a7191e6bed0e&chksm=f951a13ace26282cbaf00be9c36a78136b5ffceb42998e4f0ff53aafbee2ffcc17170c8a399f&scene=0#rd

英文原文 https://medium.com/@firt/progressive-web-apps-on-ios-are-here-d00430dee3a7

PWA是指使用 Web 技术创建的应用，这样的应用无需打包或签名，可以脱机使用，并且可以安装到操作系统中，随后无论看起来或用起来都和其他应用完全一致。

在大部分平台上，PWA 的安装都无需“应用商店”介入，不过 Edge/Windows 10 平台目前依然强制要求通过商店安装 PWA。

因此你想得没错，现在你可以无需经历 App Store 的审批为 iOS 安装应用了。也许这也是 Apple 对这个新功能绝口不提的主要原因之一，他们可能不想让用户感到困惑。甚至 Safari 浏览器的发布说明里都没有提到这个技术。

### 1. Apple 到底是不是 PWA 的发明人？

老实说，虽然 Google 以及 Chrome 团队杜撰了 PWA 这个术语，但这个理念最早是初代 iPhone OS 发布时出现在 Safari 浏览器上的。2007 年，Steve Jobs 在 WWDC 上公布了“One more thing”：如何为初代 iPhone 开发应用，并提出了让人吃惊的方法：Web 应用！最初的产品路线图中并不包含 App Store，设备上市后的第一年内也没有原生 SDK。从 Apple 的角度来看，甚至今天的 PWA，实际上也属于“能从主屏直接启动的 Web 应用”，而应用的图标当时被称之为 WebClip。

11 年前，这个理念并未引起太大关注，而 Apple 自己也忘了更新这个平台，因此十多年来，这个平台充满了 Bug 和不一致的情况。几年后，其他平台借鉴了这个理念，例如 Nokia N9 上的 MeeGo Browser 以及 Android 上的 Chrome。

Chrome 进一步完善了这种技术，提供了更好的体验，而他们的改进主要围绕 Service Worker 和 Web App Manifest 的规范等方面。从目前（2018 年 3 月 30 日）的 iOS 11.3 开始，Apple 在这两个规范的支持方面赶上了 Chrome、Firefox、Samsung Internet、UC Browser 以及 Opera（这几家大部分仅面向 Android）。已支持 Service Worker 但尚不支持 Web App Manifest 的其他桌面端浏览器则是今年的主要完善目标。

是的，没错。然而这样的应用只能在浏览器或 Web 平台的安全和执行模型中运行。这意味着你可以“发布”未经商店审批的应用，例如为公司员工发布内部应用（当然，也可用来发布成人内容），但这样的应用无法访问纯原生功能，例如 iPhone X 上的面容 ID 或增强现实 ARKit。或者至少需要等待 Web 平台开始逐渐支持这些新功能。

PWA 可以作为网站或独立应用的模式运行在 Safari 浏览器内部，具体表现和系统中的其他应用完全相同。也许你会好奇 PWA 是否使用 Web View，对于 Safari 浏览器或已安装的应用图标来说，无需考虑这种情况；但如果使用其他浏览器，或使用诸如 Facebook 等应用的内嵌浏览器，就需要考虑了。

### 2. iOS 平台上 PWA 的可用能力

借助 iOS 上的 Web 平台，你可以访问：

- 地理位置

- 传感器数据（磁力计、加速计、陀螺仪）

- 摄像头

- 音频输出

- 语音合成（仅限使用耳机的情况）

- Apple Pay

- WebAssembly、WebRTC、WebGL 以及很多带标签的实验性功能

### 3. 相比原生 iOS 应用存在的局限

- 应用可以存储脱机数据，文件大小上限为 50 Mb。

- 如果用户连着几周未使用某个应用，iOS 会清空应用的文件。应用图标依然会显示在主屏上，如果再次访问该应用，将会重新下载。

- 无法访问某些功能，如蓝牙、序列号、Beacon 信标、触控 ID、面容 ID、ARKit、高度计、电池信息。

- 处于后台运行状态时无法访问可执行代码。

- 无法访问私有信息（联系人、后台定位），也无法访问原生社交应用。

- 无法访问应用内付款，以及很多其他 Apple 服务。

- 在 iPad 上，无法与其他应用使用侧拉或分屏显示，PWA 将始终占据整个屏幕。

- 无推送通知，无图标标记，无法与 Siri 集成。



##### Android 上的 PWA 能做哪些 iOS 上无法实现的事？

- Android 上可以存储的数据量能够超过 50 Mb。

- 如果不使用某个应用，Android 并不删除其文件，但如果存储空间不足时依然可能删除。此外如果用户需要大量安装或使用，可以为 PWA 申请持久存储（Persistent Storage）。

- BLE 设备可访问蓝牙。

- 可通过 Web Share 访问原生共享对话框。

- 语音识别。

- 后台同步和 Web 推送通知。

- 可通过 Web App Banner 邀请用户安装应用。

- 可（略微）定制应用的启动界面以及要使用的屏幕方向。

- 对于 WebAPK 和 Chrome，用户无法安装超过一个的 PWA 实例。

- 对于 WebAPK 和 Chrome，PWA 会位于“设置”选项下，并且可以显示数据用量；iOS 上，一切均会显示在 Safari 浏览器选项下。

- 对于 WebAPK 和 Chrome，PWA 自行管理其 URL 意图，因此如果点击一个指向 PWA 的链接，可以直接用独立模式打开应用，无需在浏览器窗口内打开。

##### iOS 上的 PWA 能做哪些 Android 上无法实现的事？

- 用户可以在安装前更改图标的名称。

- 可通过 配置文件 进行配置，因此企业用户可以从公司接收 PWA 快捷方式（很棒的做法！）。Safari 浏览器将这个功能称作 WebClip，不过（根据文档介绍）似乎不是通过读取 Web App Manifest 实现的。

因为 Safari 浏览器不会显示任何提示或邀请（即 Android 上的 Web App Banner）。因此用户需要通过 Safari 浏览器访问 PWA URL，然后手工点按分享图标，并选择“添加到主屏”。但用户无法从视觉上区分所访问的网站是否是 PWA。

此外 App Store 中还有很多第三方浏览器，例如 Chrome、Firefox、Brave 或 Edge，它们都无法安装 PWA 或使用 Service Worker。

安装之后，这样的应用和主屏上的其他应用几乎没什么两样，不过 PWA 无法使用 3D 触控。此外如果再次安装同一个 PWA，会出现指向同一个 PWA 的新图标（好在安装后的文件是共用的）。

### 4. 目前不支持的特性

- 目前在 iOS 上不支持display: fullscreen和display: minimal-ui，fullscreen 将触发独立版本，而 minimal-ui 仅仅是 Safari 的快捷方式。不过你可以使用cover-fit viewport 扩展或已经废弃的 Meta tag 获得类似的全屏效果（状态栏依然存在，但会重叠到应用上层显示）。

- 如果必须使用后后台同步，则应准备好备用的实现方式。

- 无法锁定 PWA 的画面方向。

- 无法使用主题配色更改状态栏风格，只能使用已经废弃的 Meta tag 实现黑色或白色状态栏，或者使用 CSS/HTML 模拟主题配色。


