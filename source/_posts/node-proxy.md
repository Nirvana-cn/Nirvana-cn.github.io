---
title: 前端跨域快到碗里来
date: 2019-04-12 16:40:32
tags: 
- http-proxy
- 跨域
categories:
- 前端,每天学一点
---

使用`node`进行请求代理，摆脱各种限制，前端跨域不再怕🥰🥰🥰。

<!--more-->

## 1. 引言
关于跨域的相关问题我们不再赘述，详细内容可以参考文末的链接，我们的目的就是让跨域变得简单、简单、简单。

实习的时候碰到一个问题，本地开发需要向服务器请求数据，本地域名肯定是`localhost`了，明显的跨域问题，浏览器是不会帮你向服务器发送请求的。同时，服务器我们也没办法进行任何修改，请求的数据还比较复杂，这种情况下，前端的一些黑科技跨域方法（比如`jsonp`、`iframe+document.domain`）就没办法用了。

就在我苦恼之际，`node`给我带来了全新的曙光🥳。

PS: 以前没用过`node`做代理转发，用了之后发现真香，真香定律它`lei`了。

## 2. node代理跨域

### 2.1 安装http-proxy
使用`node`代理请求进行转发实现非常简单，而且适应各种请求类型。你只需正常向本地服务器发起请求，就可以得到远程服务器响应的数据，完全感觉不到代理的存在。

首先安装`http-proxy`包，
```bash
npm install http-proxy --save-dev
```

### 2.2 开启本地服务
搭建代理服务器，
```javascript
var httpProxy=require('http-proxy')
var proxy = httpProxy.createProxyServer()
```

如果遇到证书不安全的问题，开发环境下可以关闭证书验证。
```javascript
var proxy = httpProxy.createProxyServer({
    secure: false
})
```

配置本地服务器，同时进行代理转发。
```javascript
var PORT = 3000
var http = require('http')
var url=require('url')
var fs=require('fs')
var path=require('path')
var httpProxy=require('http-proxy')
var proxy = httpProxy.createProxyServer()

var server = http.createServer(function (request, response) {
    var pathname = url.parse(request.url).pathname
    console.log(pathname)
    if(pathname==='/'){
        fs.readFile('./index.html', function (error, data) {
            response.writeHead(200, {'Content-Type': 'text/html'})
            response.end(data, 'utf-8')
        })
    }

    if(pathname.includes('/proxy')){    // 只要请求路径中含有proxy字段，全部进行转发
        proxy.web(request, response, { target: 'http://127.0.0.1:3001' })   //目标服务器为3001端口
    }
})
server.listen(PORT)
console.log("Server runing at port: " + PORT + ".")

```

### 2.3 跨域请求服务器 

搭建远程服务器，进行请求响应。
```javascript
var PORT = 3001;
var http = require('http');
var url=require('url');

var server = http.createServer(function (request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log(pathname)
    if(pathname==='/proxy/test'){
        response.writeHead(200, {'Content-Type': 'text/plain'})
        response.end('ok', 'utf-8')
        console.log('response success')
    }

    if(pathname==='/proxy/doing'){
        response.writeHead(200, {'Content-Type': 'text/plain'})
        response.end('doing', 'utf-8')
        console.log('response success')
    }
});
server.listen(PORT);
console.log("Server runing at port: " + PORT + ".");

```

经过以上设置，我们只需要正常向3000端口进行请求，`http-proxy`模块会自动代理我们的请求，向3001端口发起请求，对我们的请求进行响应，这中间的代理转发过程我们不必关心，就好像直接从3000端口获取数据一样。
```javascript
axios.get('http://127.0.0.1:3000/proxy/test').then(response => {
      console.log(response.data)
}).catch(err => {
      console.log(err)
})
```
这里我们也可以对请求路径进行处理，截断后再转发给远程服务器，服务器处理略有不同。根据需求进行选择。
```javascript
if(pathname.includes('/proxy')){
    request.url=request.url.split('proxy')[1]
    proxy.web(request, response, { target: 'http://127.0.0.1:3001' })
}
```

## 3. 更多

正确面对跨域，别慌：[>>>点我进入](https://juejin.im/post/5a2f92c65188253e2470f16d#heading-19)

本文demo地址：[>>>点我进入](https://github.com/Nirvana-cn/Node-proxy)