---
title: Centos7命令行下安装和配置Apache服务器
date: 2017-05-07 20:48:49
tags:
- linux
- centos
- apache
categories:
- 后端,不抛弃
---

学习并记录在`centos`系统里如何在命令行下安装和配置`Apache`服务器。

<!--more-->

## 1. 安装centos系统

首先安装`centos`系统，这个比较简单，只是注意安装的时候不要最小化安装，否则系统内容比较少，需要自己重新配置。新手的话建议安装带有图形界面的`centos`系统，方便自己检查和验证，`KDE`界面做的还是不错的。

## 2. 安装Apache服务

安装`Apache`服务。`Apache`在`centos7`中是`Apache HTTP server`，所以想安装`Apache`其实是要安装`httpd`。命令如下
```shell
[root@localhost ~]# yum install httpd
```

启动和关闭`Apache`服务

```shell
[root@localhost ~]# systemctl start httpd.service

[root@localhost ~]# systemctl stop httpd.service
```

配置`Apache`服务。在开启`httpd`服务之前，需要手动配置`httpd`服务的一些参数。用`vi`打开`httpd`的配置文件。`httpd.conf`文件里各项参数的意义可以参考[Apache主配置文件详解](http://www.linuxidc.com/Linux/2015-02/113921.htm)。命令如下

```shell
[root@localhost ~]# vi /etc/httpd/conf/httpd.conf
```

首先，做好备份！！！方便随时修改随时还原！！！主配置文件里，需要修改的地方如下

找到` #Listen 12.34.56.78:80 `这一行，模仿注释在下面添加 Listen 你的IP地址或域名:你要监听的端口号，本地访问IP地址为127.0.0.1

## 3. 测试

访问你的服务器地址，如果显示`Apache`的测试页面，说明`httpd`服务已经成功启动了。

最后在运行`httpd`服务的时候，系统可能会报各种错误，你需要耐心的看系统给出的提示，一步一步解决，这一过程对个人的成长是非常有帮助的。如果系统提示`httpd`服务已经启动，但`httpd`服务没有运行，可以结束`httpd`的所有进程，然后重新启动`httpd`服务，命令如下
```shell
[root@localhost ~]# killall httpd

[root@localhost ~]# systemctl start httpd.service
```
说明：

`/etc/httpd`是`httpd`的根目录

`/var/www/html`是放置请求页面的目录

`vi`是`unix`操作系统和类`unix`操作系统中最通用的全屏幕纯文本编辑器，初次接触可能会不习惯，需要了解一下`vi`的基本操作

`httpd.conf`配置文件里，以#开头的都是注释，可以看到`httpd.conf`有非常多的注释并且给了非常多的例子，提示你如何修改配置，这是非常人性化的

