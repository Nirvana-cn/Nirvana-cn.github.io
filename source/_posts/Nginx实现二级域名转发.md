---
title: Nginx实现二级域名转发
date: 2018-01-03 20:52:28
tags:
---
# 一.目的
解决只有一个服务器和域名，同时为几个应用提供服务的问题
# 二.举例
比如说你现在有 abc.com 的主域名，你又划分了 shop.abc.com 和 mail.abc.com 两个二级域名来实现不同的功能，并希望两个二级域名使用同一个IP地址和端口访问，但是提供不同的服务，nginx则可以监听指定的端口，根据域名的不同将请求转发给相应的端口。

# 三.实现

1.打开nginx的配置文件，打开  /安装路径/conf/nginx.conf，进行如下配置
```
server {  
        listen       80;  
        server_name  *.abc.com;  
  
        if ($http_host ~* "^(.*?)\.abc\.com$") {    #正则表达式  
                set $domain $1;                     #设置变量  
        }  
  
        location / {  
            if ($domain ~* "shop") {  
               proxy_pass http://abc.com:3001;      #域名中有shop，转发到3001端口  
            }  
            if ($domain ~* "mail") {  
               proxy_pass http://abc.com:3002;      #域名中有mail，转发到3002端口  
            }  
  
            tcp_nodelay     on;  
            proxy_set_header Host            $host;  
            proxy_set_header X-Real-IP       $remote_addr;  
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;  
            #以上三行，目的是将代理服务器收到的用户的信息传到真实服务器上  
              
            root   html;  
            index  index.html index.htm;            #默认情况  
        }  
}  
```
2.命令行输入 /安装路径/sbin/nginx -t 查看nginx配置是否正确
3.命令行输入 /安装路径/sbin/nginx -s reload 重新加载nginx
4.如果相应端口上有对应的服务，那么我们的目的就达到了
