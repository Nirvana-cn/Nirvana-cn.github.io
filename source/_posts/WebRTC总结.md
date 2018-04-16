---
title: WebRTC学习总结
date: 2018-04-16 16:28:34
tags:
- WebRTC
- 直播
categories:
- Web前端
---

WebRTC (Web Real-Time Communications) 是一项实时通讯技术，它允许网络应用或者站点，在不借助中间媒介的情况下，建立浏览器之间点对点（Peer-to-Peer）的连接，实现视频流和（或）音频流或者其他任意数据的传输。WebRTC包含的这些标准使用户在无需安装任何插件或者第三方的软件的情况下，创建点对点（Peer-to-Peer）的数据分享和电话会议成为可能。

本篇文章从自身实践出发，结合相关代码，总结WebRTC实现的基本流程。

<!--more-->

### 1. 引言

首先我们先看《WebRTC权威指南》上给出的流程图，从这张图，我们要明确两件事：
- 第一，通信双方需要先通过服务器交换一些信息
- 第二，完成信息交换后，通信双方将直接进行连接以传输数据

![](http://112.74.18.120:3001/p12.jpg)

然后我们再介绍一下WebRTC中的专有名词，方便读者对下文的理解。

- RTCPeerConnection：核心对象，每一个连接对象都需要新建该对象
- SDP(Session Description Protocol，会话描述协议)：包含建立连接的一些必要信息，比如IP地址等，sdp由RTCPeerConnection对象方法创建，我们不需要知道该对象中的具体内容，使用黑盒传输即可
- ICE(Interactive Connectivity Establishment，交互式连接建立技术)：用户之间建立连接的方式，用来选取用户之间最佳的连接方式

### 2. WebRTC实现流程

以下代码不能直接运行，因为我这里并没有实现信令服务器，如何实现信令服务器可自由选择。

首先发起方获取视频流，如果成功，则新建RTCPeerConnection对象，然后创建offer，并发送给应答方。

- addStream方法将getUserMedia方法中获取的流(stream)添加到RTCPeerConnection对象中，以进行传输
- onaddStream事件用来监听通道中新加入的流，通过e.stream获取
- onicecandidate事件用来寻找合适的ICE
- createOffer()是RTCPeerConnection对象自带的方法，用来创建offer，创建成功后调用setLocalDescription方法将localDescription设置为offer，localDescription即为我们需要发送给应答方的sdp
- sendOffer和sendCandidate方法是自定义方法，用来将数据发送给服务器

```
// 引入<script src="https://webrtc.github.io/adapter/adapter-latest.js"></script>脚本
// 提升浏览器兼容性
var localConnection, 
var constraints={
    audio:false,
    video:true
}
navigator.mediaDevices.getUserMedia(constraints).then(handleSuccess).catch(handleError)
function handleSuccess(stream) {
  document.getElementById("video").srcObject = stream
  localConnection=new RTCPeerConnection()
  localConnection.addStream(stream)
  localConnection.onaddStream=function(e) {
    console.log('获得应答方的视频流' + e.stream)
  }
  localConnection.onicecandidate=function(event){
    if(event.candidate){
        sendCandidate(event.candidate)
    }
  }
  localConnection.createOffer().then((offer)=>{
    localConnection.setLocalDescription(offer).then(sendOffer)
  })
}
```
同样的，接收方也需要新建一个RTCPeerConnection对象
```
var remoteConnection
var constraints={
    audio:false,
    video:true
    }
}
navigator.mediaDevices.getUserMedia(constraints).then(handleSuccess).catch(handleError)
function handleSuccess(stream) {
  document.getElementById("video").srcObject = stream
  remoteConnection=new RTCPeerConnection()
  remoteConnection.addStream(stream)
  remoteConnection.onaddStream=function(e) {
    console.log('获得发起方的视频流' + e.stream)
  }
  remoteConnection.onicecandidate=function(event){
      if(event.candidate){
          sendCandidate(event.candidate)
      }
  }
}
```
当应答方收到发起方发送的offer之后，调用setRemoteDescription设置RTCPeerConnection对象的remoteDescription属性，设置成功之后调用createAnswer方法，创建answer成功之后将其设置为localDescription，然后把answer发送给服务器
```
let desc=new RTCSessionDescription(sdp)
remoteConnection.setRemoteDescription(desc).then(function() {
    remoteConnection.createAnswer().then((answer)=>{
        remoteConnection.setLocalDescription(answer).then(sendAnswer)
    })
})
```

当发起方收到应答方发送的answer之后，将其设置为remoteDescription，至此WebRTC连接完成。
```
let desc=new RTCSessionDescription(sdp)
localConnection.setRemoteDescription(desc).then(()=>{console.log('Peer Connection Success')})
```
此时虽然WebRTC连接已经完成，但是通信双方还不能直接通信，因为发送的ICE还没有处理，通信双方还没有确定最优的连接方式。

应答方收到发起方发送的ICE数据时，调用RTCPeerConnection对象的addIceCandidate方法。
```
remoteConnection.addIceCandidate(new RTCIceCandidate(ice))
```
发起方收到应答方发送的ICE数据时，同样调用RTCPeerConnection对象的addIceCandidate方法。
```
localConnection.addIceCandidate(new RTCIceCandidate(ice))
```
至此，一个最简单的WebRTC连接已经建立完成。

### 3. 数据通道

未完待续


### 参考文献

[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API)

博客[WebRTC学习资料大全](https://blog.csdn.net/foruok/article/details/53005728)

书籍《WebRTC权威指南》，《Learning WebRTC 中文版》

[Github](https://github.com/webrtc)地址