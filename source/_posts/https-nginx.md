---
title: Nginx实现https以及https二级域名转发
date: 2017-06-23 20:57:55
tags:
- nginx
categories:
- 后端,不抛弃
---

最近在开发微信小程序，但是小程序只能连接`https`服务器，所以就需要用`nginx`自己搭一个`https`服务器。

<!--more-->

## 0.前言

最近在倒腾微信小程序，研究小程序怎么和服务器通信，上一篇博客里通过`nginx`实现了`http`的二级域名转发，结果发现`https`实现二级域名转发又是另一会事，心累啊

目的：通过`https://abc.com`访问主域名，使用`nginx`将访问`https://shop.abc.com`二级域名的请求转发到`https://abc.com:3000`端口。

## 1.给域名搞个ssl证书

我偷懒直接从阿里云申请了一个免费的证书，阿里云的证书服务里面有介绍如何使用证书，唯一要注意就是阿里云的免费证书只能对一个域名有效，即不管你是主域名还是二级域名都需要单独申请一个`ssl`证书，否则不能使用`https`访问。

## 2.主域名配置nginx

找到`nginx`目录下的`nginx.conf`文件，进行如下配置，和`http`服务的配置类似，多添加一个证书服务。

```
server {  
      listen 443 ssl;  
      server_name abc.com;  
      server_name_in_redirect off;  
  
      ssl_certificate   cert/xxx.pem;       #添加证书服务  
      ssl_certificate_key  cert/xxx.key;  
        
      ssl_session_timeout 5m;  
      ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;  
      ssl_prefer_server_ciphers on;  
        
      location / {  
          tcp_nodelay     on;  
          proxy_set_header Host            $host;  
          proxy_set_header X-Real-IP       $remote_addr;  
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
              
          root html;  
          index index.html index.htm;  
      }  
  }  
```
## 3.二级域名配置nginx

`https`的转发要麻烦一点，因为`ssl`证书不一样，所以不能在主域名的`server`中进行二级域名转发。找到`nginx`目录下的`nginx.conf`文件，在主域名`server`下面增加一个`server`。

```
server {  
        listen 443 ssl;  
        server_name shop.abc.com;  
        server_name_in_redirect off;  
          
        #可以设置独立的ssl认证  
        ssl_certificate   cert/xxx.pem;  
        ssl_certificate_key  cert/xxx.key;  
          
        ssl_session_timeout 5m;  
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;  
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;  
        ssl_prefer_server_ciphers on;  
          
        location / {  
          tcp_nodelay     on;  
          proxy_set_header Host            $host;  
          proxy_set_header X-Real-IP       $remote_addr;  
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
              
          proxy_pass http://abc.com:3000;  
        }  
  }  
```
对于为什么二级域名里面转发的是`http`地址，我是这么理解的：如果`Nginx`作为前端代理的话，则后台服务器根本不需要自己处理 `https`，全是`Nginx`处理的。用户首先和`Nginx`建立连接，完成`SSL`握手，而后`Nginx` 作为代理以 `http` 协议将请求转给后台服务器处理，`Nginx`再把后台服务器的输出通过`SSL` 加密发回给用户，这中间是透明的，所以后台服务器只是在处理 `http` 请求而已。因此，这种情况下只需要配置 `Nginx` 的`SSL` 和 `Proxy`。可以参考[SSL证书与Https应用部署小结](http://blog.csdn.net/andy1219111/article/details/22716315)这篇文章

## 4.重启nginx

> /安装路径/sbin/nginx -t 

> /安装路径/sbin/nginx -s reload

tips:这是我目前的解决方法，应该还有其它更好的方法，欢迎大家指正。