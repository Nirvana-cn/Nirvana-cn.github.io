---
title: Safari不同版本track接口行为差异总结
date: 2019-05-18 16:40:32
tags:
- javascript
- webrtc
categories:
- 前端,每天学一点
---

记录`Safari`浏览器不同版本那些奇奇怪怪的`BUG`🙄🙄🙄。

<!--more-->

## safari 11.0.2

1. `replaceTrack(null)`会将整个流杀掉，即流的状态变为`inactive`;

2. `removeTrack`方法只会将`track`的`muted`属性变为`true`，`readyState`属性变为`ended`，而不会将`track`移除，即流的`track`属性不为`null`;

3. 改变流的方向需要调用`RTCRtpTransceiver.prototype.setDirection`方法；

4. 调用`removeTrack`移除`track`后，不可以调用`addTrack`再次将原始`track`添加回去(例如改变分辨率的场景)，否则会报如下错误。推测是由于`MediaStream`中已经存在同源的`track`，造成添加失败。解决办法是先调用`track`的`clone`方法，再将克隆的`track`添加进去。

> InvalidAccessError: The object does not support the operation or argument.

```js
let track = stream.getTracks()[0].clone()
pc.addTrack(track, stream)
```
[Webkit bug 174327](https://bugs.webkit.org/show_bug.cgi?id=174327)也提到了该问题，并提供了一种解决方法：先调用`replaceTrack(null)`，再调用`removeTrack`，该方法可行，但是会报如下错误：

```js
sender.replaceTrack(null)
pc.removeTrack(sender)
```

> OperationError: CreateOffer called with invalid session options

## safari 12.1

1. 存在与11.0.2同样的问题4，而且`removeTrack`后第一次调用的`addTrack`无效，再次调用`addTrack`才可添加流成功；

2. `removeTrack`会将`RTCRtpSender`中的`track`置为`null`,虽然调用`addTrack`也能成功，`getSenders`中`track`不再为`null`，但是本地`sdp`中却是没有`ssrc`字段的，因此想要添加流必须先克隆再添加，而且`addTrack`需要调用两次才能生效，原因未知，造成的影响就是`getSenders`长度会加1。

3. `replaceTrack(null)`行为一致，但是使用`replaceTrack`移除流再添加流会失败，说出来你可能不信，`replaceTrack`也需要调用两次才能成功添加流，而且使用`replaceTrack`添加流的时候会使得`getSender`中产生两个相同的`sender`，造成`getSender`长度莫名+1。第一次会报如下错误：

> Unhandled Promise Rejection: OperationError: CreateOffer called with invalid session options

## tips

`safari`中是没有`addStream`和`removeStream`方法的，`adapter`中还是通过`addTrack`和`removeTrack`进行模拟。

demo地址：[>>>点我进入](https://github.com/Nirvana-cn/Live-platform/tree/master/demo/demo-track-api)
