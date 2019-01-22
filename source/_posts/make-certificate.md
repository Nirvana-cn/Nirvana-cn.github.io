---
title: 借助mkcert签发本地证书
date: 2019-01-21 14:55:41
tags:
- https
categories:
- 少年,你渴望力量吗
---

使用`mkcert`签发本地证书，一步搭建`https`环境 😆😆😆。

<!-- more -->

[mkcert](https://github.com/FiloSottile/mkcert) 是由 [Filippo Valsorda](https://blog.filippo.io/hi/) 使用`go`语言开源的一款零配置搭建本地证书服务的工具，它可以兼容`Window, Linux, macOS`等多种开发平台，省去了我们自签本地证书的繁琐步骤，从而让我们专注于开发。

![](https://user-images.githubusercontent.com/1225294/51066373-96d4aa80-15be-11e9-91e2-f4e44a3a4458.png)

## 1. 安装mkcert

### 1.1 Windows

在`Windows`环境下，推荐使用`Chocolatey`包管理工具安装`mkcert`，你可以选择使用`cmd`或`power shell`安装`Chocolatey`：

以管理员权限打开`cmd`窗口，输入以下命令进行安装

```bash
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

以管理员权限打开`power shell`窗口，输入以下命令进行安装

```bash
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

`Chocolatey`安装完成之后，命令行窗口输入以下命令安装`mkcert`

```bash
choco install mkcert
```

### 1.2 Linux

** 以下假定使用的是全新的`Linux`系统！**

首先更新软件源，防止`E: Unable to locate package `的错误，

```bash
sudo apt update
```

首先安装证书数据库工具`certutil`:

```bash
sudo apt install libnss3-tools // ubuntu
sudo yum install nss-tools // centos
```

然后，安装包管理工具`Linuxbrew`:

```bash
sudo apt install linuxbrew-wrapper  // ubuntu
sudo yum install linuxbrew-wrapper // centos 待定
```

最后，使用`brew`安装`mkcert`

```bash
brew install mkcert
```

tips:可能需要执行两次命令，同时不要使用`sudo`命令，系统会提示你不要使用`root`权限来执行`brew`。

在`mkcert`安装目录下启动`mkcert`，推荐将`mkcert`加入全局命令，

```bash
PATH=$PATH:/home/linuxbrew/.linuxbrew/Cellar/mkcert/1.2.0/bin
```

### 1.3 macOS

使用[Homebrew](https://brew.sh/)安装`mkcert`：

```bash
brew install mkcert
brew install nss # if you use Firefox
```

## 2. 生成本地CA证书

### 2.1 生成根证书

`mkcert`安装完成之后就可以使用`mkcert`命令生成本地`CA`证书了，非常简单，傻瓜式安装。

第一步：

生成根证书，一般在`C:\Users\用户名\AppData\Local\mkcert`目录下会生成`rootCA.pem`和`rootCA-key.pem`两个文件。

根证书用来充当第三方证书签发机构，类似于`Symantec`这种机构，为网站签发CA证书。因为证书也可以伪造，所以浏览器需要验证证书的有效性，证书有效之后才可以进行`https`连接，而第三方签发机构提供证书的可信度验证。

`rootCA`文件就是告诉浏览器我们自签的证书是真实有效的，接下来我们签发的本地证书都离不开`rootCA`。

```bash
$ mkcert -install
Created a new local CA at "/Users/filippo/Library/Application Support/mkcert" 💥
The local CA is now installed in the system trust store! ⚡️
The local CA is now installed in the Firefox trust store (requires restart)! 🦊
```

### 2.2 签发本地证书

根证书生成之后就可以签发本地证书了，命令超级简单，唯一需要注意的就是生成的证书存放路径就是命令的执行路径。

```bash
$ mkcert example.com "*.example.org" myapp.dev localhost 127.0.0.1 ::1
Using the local CA at "/Users/filippo/Library/Application Support/mkcert" ✨

Created a new certificate valid for the following names 📜
 - "example.com"
 - "*.example.org"
 - "myapp.dev"
 - "localhost"
 - "127.0.0.1"
 - "::1"

The certificate is at "./example.com+5.pem" and the key at "./example.com+5-key.pem" ✅
```

### 2.3 root stores

`mkcert`支持以下根存储(mkcert supports the following root stores:)：

- macOS system store
- Windows system store
- Linux variants that provide either
    - update-ca-trust (Fedora, RHEL, CentOS) or
    - update-ca-certificates (Ubuntu, Debian) or
    - trust (Arch)
- Firefox (macOS and Linux only)
- Chrome and Chromium
- Java (when JAVA_HOME is set)

这句话的意思就是说，`mkcert`会自动把证书加入系统认证，操作系统和浏览器可以直接识别。比如，证书安装完成之后chrome的证书管理中会增加以下内容：

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p36.png)

### 2.4 node环境验证

`node`不会使用`root store`，因此需要特殊对待，命令行手动设置`NODE_EXTRA_CA_CERTS`这个环境变量。

```bash
set NODE_EXTRA_CA_CERTS="$(mkcert -CAROOT)/rootCA.pem"
```

** `mkcert -CAROOT`会输出本地根证书的存储路径，可以使用环境变量`$CAROOT`来指定寻找证书的默认路径。

使用`node`开启`https`服务：

```javascript
var https = require('https');
var fs = require('fs');

const hostname = '127.0.0.1';
const port = 3000;

var options = {
    key : fs.readFileSync('c:/证书路径'),
    cert : fs.readFileSync('c:/证书路径'),
}

const server = https.createServer(options ,(req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Hello World\n');
});

server.listen(port, hostname, () => {
    console.log(`Server running at https://${hostname}:${port}/`);
});
```

`https`进行访问：

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p37.png)

## 3. 分享

搞定了自己，接下来就要搞定别人，如何让其它用户也识别我们的本地证书呢？

`mkcert`同样也给我们提供了可移植的解决方案。

将自签证书(包括公钥和私钥)以及根证书的公钥拷贝给其它用户，修改`$CAROOT`环境变量，指定`mkcert`寻找根证书的路径，

`linux`系统下：

```bash
export CAROOT="/home/..."
```

`window`系统下：

```bash

```

最后执行`mkcert -install`即可，控制台会提示，此时我们的自签证书就可以在其它机器上运行了，根证书被导入到了浏览器的证书信任中心。

```bash
The local CA is now installed in the system trust store! ⚡️
The local CA is now installed in the Firefox and/or Chrome/Chromium trust store (requires browser restart)! 🦊
```
