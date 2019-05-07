# 一、获取媒体流API

## 1. getUserMedia

60版本以上的chrome、firefox都已经支持新的基于Promise的接口与约束语法。

adapter.js主要为47-52版本的chrome和32-37版本的firefox提供Promise接口和新的约束语法。

特性|Chrome|	Edge|Firefox (Gecko)|Internet Explorer|	Opera|Safari (WebKit)|
---|:---:|:---:|:---:|:---:|:---:|:---:
基础支持	|53	|(Yes)|36|No|40|11
Promises|53|(Yes)|38 |No|?|11

## 2. get​Display​Media

72以上chrome可以从navigator.mediaDevices调用，70版本以上chrome已经支持get​Display​Media，但是属于实验性API。

66版本以上firefox中get​Display​Media成为标准API，adapter.js中通过getUserMedia模拟52版本以下get​Display​Media的行为。

edge中可以支持，但是该API挂载在navigator对象上，而不是navigator.mediaDevices对象。

get​Display​Media相对而言是比较新的API，基本上只有最新的浏览器才能支持原生API，adapter.js的模拟是需要借鉴的。

特性|Chrome|	Edge|Firefox (Gecko)|Internet Explorer|	Opera|Safari (WebKit)|
---|:---:|:---:|:---:|:---:|:---:|:---:
get​Display​Media	|72	|17|66|No|60|No

## 3. enumerate​Devices

特性|Chrome|	Edge|Firefox (Gecko)|Internet Explorer|	Opera|Safari (WebKit)|
---|:---:|:---:|:---:|:---:|:---:|:---:
enumerate​Devices|Yes|Yes|Yes|No|Yes|Yes

## 4. get​Supported​Constraints

特性|Chrome|	Edge|Firefox (Gecko)|Internet Explorer|	Opera|Safari (WebKit)|
---|:---:|:---:|:---:|:---:|:---:|:---:
get​Supported​Constraints|Yes|Yes|Yes|No|Yes|Yes

# 二、chrome源码分析

## 1. shimGetUserMedia

首先对老的接口进行约束完善

```javascript
navigator.getUserMedia = getUserMedia_.bind(navigator);
```

然后针对45版本以上的浏览器约束不完善的问题进行了优化，

## 2. shimMediaStream

只有一行语句，应该是针对版本较早的浏览器作的适配。

```javascript
window.MediaStream = window.MediaStream || window.webkitMediaStream;
```

## 3. shimOnTrack

兼容`ontrack`事件，如果`RTCPeerConnection`对象上没有`ontrack`属性，会调用`Object.defineProperty`进行添加，同时还会重新监听`onaddstream`事件，使用`track`机制处理。

```javascript
Object.defineProperty(window.RTCPeerConnection.prototype, 'ontrack', {
      get() {
        return this._ontrack;
      },
      set(f) {
        if (this._ontrack) {
          this.removeEventListener('track', this._ontrack);
        }
        this.addEventListener('track', this._ontrack = f);
      },
      enumerable: true,
      configurable: true
    })
```

## 4. shimPeerConnection

`shimPeerConnection`中对`RTCPeerConnection`对象上的三个方法进行了微调。

三个方法的参数封装。
```js
['setLocalDescription', 'setRemoteDescription', 'addIceCandidate'].forEach(function(method) {
        const nativeMethod = window.RTCPeerConnection.prototype[method];
        window.RTCPeerConnection.prototype[method] = function() {
          arguments[0] = new ((method === 'addIceCandidate') ? window.RTCIceCandidate : window.RTCSessionDescription)(arguments[0]);
          return nativeMethod.apply(this, arguments);
        };
      });
```

`addIceCandidate`方法提供了`Promise`的版本，但是在一些老旧的代码和文档中, 可能会出现一个回调函数(`callback`)的版本。这种函数是过期的，强烈建议不要使用。
```js
const nativeAddIceCandidate = window.RTCPeerConnection.prototype.addIceCandidate;
  window.RTCPeerConnection.prototype.addIceCandidate = function() {
    if (!arguments[0]) {
      if (arguments[1]) {
        arguments[1].apply(null);
      }
      return Promise.resolve();
    }
    return nativeAddIceCandidate.apply(this, arguments);
  };
```

## 5. shimAddTrackRemoveTrack

`shimAddTrackRemoveTrack`中区分了两种情况，一种情况是浏览器原生支持`addTrack`的`api`，另一种情况就是不支持`track`机制，需要`ployfill`。`IPVideoTalk`目前已经切换到`track`接口，所以应该不再需要第二种情况。

```js
if (window.RTCPeerConnection.prototype.addTrack &&
      browserDetails.version >= 65) {
    return shimAddTrackRemoveTrackWithNative(window);
  }
```

`shimAddTrackRemoveTrackWithNative`中对`getLocalStreams, addTrack, addStream, removeTrack, removeStream`添加和移除流的函数进行了优化，自定义了一个`_shimmedLocalStreams`对象，用来记录添加的流。当调用`addTrack, addStream`方法时向`_shimmedLocalStreams`增加对象属性值，当调用`removeTrack, removeStream`方法时移除`_shimmedLocalStreams`对象中的相应属性值。

由于`_shimmedLocalStreams`对象的存在，因此`getLocalStreams`可以取到所有激活的流。

```js
window.RTCPeerConnection.prototype.getLocalStreams = function() {
    this._shimmedLocalStreams = this._shimmedLocalStreams || {};
    return Object.keys(this._shimmedLocalStreams)
      .map(streamId => this._shimmedLocalStreams[streamId][0]);
  };
```

## 6. shimGetStats

`shimGetStats`对`getStats`方法没有重大的重写，主要提供了三个方面的优化：`getStats`返回结果的格式化，`getStats`返回`maplike`对象，提供`Promise`支持。新版本的`chrome`浏览器不存在这些问题，所以直接使用原生`api`更好。

## 7. shimSenderReceiverGetStats

为`RTCRtpSender`和`RTCRtpReceiver`不支持`getStats`的浏览器提供`ployfill`，使用`getStats`获取所有流的状态进行过滤。

## 8. shimGetSendersWithDtmf

新版本`chrome`中`getSender`和`dtmf`都是支持的，因此，`shimGetSendersWithDtmf`不是必须的。

```js
if (typeof window === 'object' && window.RTCPeerConnection &&
      !('getSenders' in window.RTCPeerConnection.prototype) &&
      'createDTMFSender' in window.RTCPeerConnection.prototype) {
    
  } else if (typeof window === 'object' && window.RTCPeerConnection &&
             'getSenders' in window.RTCPeerConnection.prototype &&
             'createDTMFSender' in window.RTCPeerConnection.prototype && window.RTCRtpSender &&
             !('dtmf' in window.RTCRtpSender.prototype)) {
    
  }
```

## 9. fixNegotiationNeeded

为`RTCPeerConnection.prototype`对象提供`onnegotiationneeded`事件监听。


# 三、firefox源码分析

## 1. shimOnTrack

`shimOnTrack`中只为`RTCTrackEvent`原型上添加了只读属性`transceiver`，但是根据`MDN`文档，当有流被添加时，`RTCTrackEvent`对象会被实例化，我们可以从`RTCPeerConnection`对象上得到相应属性，因此`MDN`文档不推荐实例化一个`RTCTrackEvent`对象。

```js
Object.defineProperty(window.RTCTrackEvent.prototype, 'transceiver', {
      get() {
        return {receiver: this.receiver};
      }
    });
```

## 2. shimPeerConnection

与`chrome`一样，对三个方法进行了参数封装

```js
['setLocalDescription', 'setRemoteDescription', 'addIceCandidate'].forEach(function(method) {
        const nativeMethod = window.RTCPeerConnection.prototype[method];
        window.RTCPeerConnection.prototype[method] = function() {
          arguments[0] = new ((method === 'addIceCandidate') ? window.RTCIceCandidate : window.RTCSessionDescription)(arguments[0]);
          return nativeMethod.apply(this, arguments);
        };
      });
```

重新封装`addIceCandidate`方法，使其支持`null`和`undefined`。
```js
window.RTCPeerConnection.prototype.addIceCandidate = function() {
    if (!arguments[0]) {
      if (arguments[1]) {
        arguments[1].apply(null);
      }
      return Promise.resolve();
    }
    return nativeAddIceCandidate.apply(this, arguments);
  };
```

优化了浏览器版本低于53的`getStats`方法，重构之后这部分代码不再需要。
```js
if (browserDetails.version < 53 && !onSucc) { 
    // ... 
}
```

## 3. shimSenderGetStats

为了修复`RTCRtpSender`对象上没有`getStats`方法的问题，有如下判断。对此，重构之后这部分代码不再需要。

```js
if (window.RTCRtpSender && 'getStats' in window.RTCRtpSender.prototype) {
    return;
}
```
## 4. shimReceiverGetStats

与`shimSenderGetStats`中类似，重构之后不再需要。

## 5. shimRemoveStream

`firefox`本身是不支持`removeStream`方法的，因此需要使用`removeTrack`方法模拟。
```js
window.RTCPeerConnection.prototype.removeStream = function(stream) {
    utils.deprecated('removeStream', 'removeTrack');
    this.getSenders().forEach(sender => {
      if (sender.track && stream.getTracks().includes(sender.track)) {
        this.removeTrack(sender);
      }
    });
  };
```
## 6. shimRTCDataChannel

解决`firefox 60`以下版本`RTCDataChannel`命名不一致的问题。

```js
if (window.DataChannel && !window.RTCDataChannel) {
    window.RTCDataChannel = window.DataChannel;
}
```

# 四、safari源码分析

## 1. shimLocalStreamsAPI

首先`safari`是不支持一些与`stream`相关的`api`，因此`shimLocalStreamsAPI`中分别`ployfill`了`getLocalStreams, addStream, removeStream`。

`getLocalStreams`使用`_localStreams`保存流信息，这些流就可以在`addStream, removeStream`中获取，通过`addTrack, removeTrack`实现相应的操作。

```js
window.RTCPeerConnection.prototype.getLocalStreams = function() {
      if (!this._localStreams) {
        this._localStreams = [];
      }
      return this._localStreams;
    };
```

## 2. shimRemoteStreamsAPI

`shimRemoteStreamsAPI`结合支持的`ontrack`事件来模拟`onaddstream`事件，使用`_remoteStreams`变量来保存流信息，分别监听`ontrack`事件和`onaddstream`事件，`ontrack`事件的处理函数中自定义了一个`onaddstream`事件，然后使用`dispatchEvent`触发该事件，触发之后`onaddstream`事件的处理函数被调用，成功模拟了`onaddstream`事件流程。

```js

Object.defineProperty(window.RTCPeerConnection.prototype, 'onaddstream', {
      get() {
        return this._onaddstream;
      },
      set(f) {
        if (this._onaddstream) {
          this.removeEventListener('addstream', this._onaddstream);
          this.removeEventListener('track', this._onaddstreampoly);
        }
        this.addEventListener('addstream', this._onaddstream = f);
        this.addEventListener('track', this._onaddstreampoly = (e) => {
          e.streams.forEach(stream => {
            if (!this._remoteStreams) {
              this._remoteStreams = [];
            }
            if (this._remoteStreams.includes(stream)) {
              return;
            }
            this._remoteStreams.push(stream);
            const event = new Event('addstream');
            event.stream = stream;
            this.dispatchEvent(event);
          });
        });
      }
    });
```
不是很清楚为什么`setRemoteDescription`中又定义一次`track`事件？
```js
window.RTCPeerConnection.prototype.setRemoteDescription = function() {
      const pc = this;
      if (!this._onaddstreampoly) {
        this.addEventListener('track', this._onaddstreampoly = function(e) {
          e.streams.forEach(stream => {
            if (!pc._remoteStreams) {
              pc._remoteStreams = [];
            }
            if (pc._remoteStreams.indexOf(stream) >= 0) {
              return;
            }
            pc._remoteStreams.push(stream);
            const event = new Event('addstream');
            event.stream = stream;
            pc.dispatchEvent(event);
          });
        });
      }
      return origSetRemoteDescription.apply(pc, arguments);
    };
```

## 3. shimCallbacksAPI

`shimCallbacksAPI`中为`createOffer, createAnswer, setLocalDescription, setRemoteDescription, addIceCandidate`五个方法提供`Promise`的接口，但是经过实际测试，`safari 12`版本中已经原生支持`Promise`接口。

## 4. shimGetUserMedia

`shimGetUserMedia`中为`navigator`对象添加`getUserMedia`方法。

```js
navigator.getUserMedia = function(constraints, cb, errcb) {
    navigator.mediaDevices.getUserMedia(constraints).then(cb, errcb);
}.bind(navigator);
```

还为`safari 12.1`以下版本的约束(`constraints`)做了修正，`safari 12.1`以上版本是不需要的。

```js
const mediaDevices = navigator.mediaDevices;
const _getUserMedia = mediaDevices.getUserMedia.bind(mediaDevices);
navigator.mediaDevices.getUserMedia = (constraints) => {
    return _getUserMedia(shimConstraints(constraints));
};
```

## 5. shimRTCIceServerUrls

将非标准的`RTCIceServer.url`迁移到`RTCIceServer.urls`上。这部分不是很了解。

在`RTCPeerConnection`上添加`generateCertificate`只读属性。
```js
Object.defineProperty(window.RTCPeerConnection, 'generateCertificate', {
      get() {
        return OrigPeerConnection.generateCertificate;
      }
    })
``` 

## 6. shimTrackEventTransceiver

为`RTCTrackEvent`对象的原型添加`transceiver`属性。与`firefox`源码中`shimOnTrack`功能类似。

```js
Object.defineProperty(window.RTCTrackEvent.prototype, 'transceiver', {
      get() {
        return {receiver: this.receiver};
      }
    });
```

## 7. shimCreateOfferLegacy

解决`createOffer`中的遗留问题，`createOffer`可以输入一个`offerOptions`参数，包含`offerToReceiveAudio `和`offerToReceiveVideo `两个废弃的选项来控制远端发送音视频流的能力，不建议使用该选择，推荐使用`RTCRtpTransceiver`来控制是否接受远端发来的流。


# 五、edge源码分析

老版本的`edge`就比较复杂了，除了`adapter`中的内容，`edge`还依赖`rtcpeerconnection-shim`和`sdp-shim`两个`npm`库，所以整体代码量比较大。

## 1. RTCPeerConnection

`RTCPeerConnection`中初始化了一些对象属性，用于保存连接参数和状态。

```js
var RTCPeerConnection = function(config) {
    var pc = this;

    var _eventTarget = document.createDocumentFragment();
    ['addEventListener', 'removeEventListener', 'dispatchEvent']
      .forEach(function(method) {
        pc[method] = _eventTarget[method].bind(_eventTarget);
      });

    this._canTrickleIceCandidates = null;
    this._localDescription = null;
    this._remoteDescription = null;
    this._signalingState = 'stable';
    this._iceConnectionState = 'new';
    this._connectionState = 'new';
    this._iceGatheringState = 'new';
    
    this._transceivers = [];

    this._sdpSessionId = SDPUtils.generateSessionId();
    this._sdpSessionVersion = 0;

    this._dtlsRole = undefined; // role for a=setup to use in answers.

    this._isClosed = false;
    this._needNegotiation = false;

    this._localStreams = [];
    this._remoteStreams = [];

    this._usingBundle = config ? config.bundlePolicy === 'max-bundle' : false;
    this._iceGatherers = [];

    // 处理 configuration.
    // ......
  };
```

使用`Object.defineProperty`方法定义了`RTCPeerConnection.prototype`上一些属性的`get`方法。

```js
['localDescription', 'remoteDescription', 'signalingState',
    'iceConnectionState', 'connectionState', 'iceGatheringState',
    'canTrickleIceCandidates'].forEach(function(propertyName) {
    Object.defineProperty(RTCPeerConnection.prototype, propertyName, {
      configurable: true,
      get: function() {
        return this['_' + propertyName];
      }
    });
  });
```

`RTCPeerConnection.prototype`对象上事件初始化。

```js
['icecandidate', 'addstream', 'removestream', 'track',
    'signalingstatechange', 'iceconnectionstatechange',
    'connectionstatechange', 'icegatheringstatechange',
    'negotiationneeded', 'datachannel'].forEach(function(eventName) {
    RTCPeerConnection.prototype['on' + eventName] = null;
  });
```

## 2. createOffer 和 createAnswer

`edge`的`RTCPeerConnection.prototype`对象上是有`createOffer`和`createAnswer`两个原生方法的，但是由于两个原生方法按照`ORTC`标准返回的`sdp`是`json`格式，因此`adapter`中重写了这两个方法，使用依赖中的`sdp`库对`sdp`字段进行了重新解析，最后返回一个`Promise`对象。

```js
RTCPeerConnection.prototype.createOffer = function() {
    // sdp操作
    // ...
    var desc = new window.RTCSessionDescription({
      type: 'offer',
      sdp: sdp
    });
    return Promise.resolve(desc);
  };
```

edge源码分析未完待续

