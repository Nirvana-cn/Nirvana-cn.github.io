---
title: git配置多个ssh key
date: 2019-01-18 08:57:32
tags:
- git
categories:
- 少年,你渴望力量吗
---

当有多个`git`账号的时候，比如一个`github`，用于自己进行一些开发活动，再来一个`gitlab`，一般是公司内部的`git`。这两者你的邮箱如果不同的话，就会涉及到一个问题，生成第二个`git`的`key`的时候会覆盖第一个的`key`，导致必然有一个用不了。此时，可以在`~/.ssh`目录下新建一个`config`文件对`ssh key`进行配置。

<!-- more -->

当有多个`git`账号时，比如：

> 一个github，用于自己进行一些开发活动

> 一个gitlab，用于公司内部的git

这两者如果邮箱不同的话，在生成第二个`key`的时候会覆盖第一个的`key`，会导致一个用不了。

解决办法就是：

> 生成两个（或多个）不同的公私密钥对，用config文件管理它们


## 1.步骤

我们假设原来在`~/.ssh`目录下已经生成了一个密钥对：

### 1.1 生成第二个key

接下来我们生成第二个`ssh key`：

```shell
ssh-keygen -t rsa -C "yourmail@gmail.com"
```

这里不要一路回车，我们需要自己手动填写保存路径和文件名（避免覆盖同名文件）：

```
Generating public/private rsa key pair.
Enter file in which to save the key (/c/Users/Gary/.ssh/id_rsa): /c/Users/Gary/.ssh/id_rsa_work
<剩下两个直接回车>
```

这里我们用`id_rsa_work`来区别原有密钥对，避免被覆盖。

完成之后，我们可以看到`~/.ssh`目录下多了两个文件，变成：

```
id_rsa
id_ras.pub
id_rsa_work
id_rsa_work.pub
known_hosts
```

### 1.2 打开ssh-agent

这里如果你用的`github`官方的`bash`，用：

```
ssh-agent -s
```

如果是其他的，比如`msysgit`，用：

```
eval $(ssh-agent -s)
```

略过这一步的话，下一步会提示这样的错误：`Could not open a connection to your authentication agent.`

### 1.3 添加私钥

```
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa_work
```

### 1.4 创建config文件

在`~/.ssh`目录下创建名为`config`的文件。

```
touch config
```

添加以下内容：

```
# gitlab
    Host git.iboxpay.com
    HostName git.iboxpay.com
    PreferredAuthentications publickey // 可以省略
    KexAlgorithms diffie-hellman-group1-sha1 // 可以省略
    IdentityFile ~/.ssh/id_rsa_work

# github
    Host github.com
    HostName github.com
    PreferredAuthentications publickey // 可以省略
    KexAlgorithms diffie-hellman-group-exchange-sha256 // 可以省略
    IdentityFile ~/.ssh/id_rsa
```

其中，`Host`和`HostName`填写`git`服务器的域名，`IdentityFile`指定私钥的路径，`KexAlgorithms`指定密钥加(解)密算法，注意需要和服务器提供的方法相匹配。

**  目前`github`提供以下几种加(解)密方法：`ecdh-sha2-nistp256,ecdh-sha2-nistp384`, `ecdh-sha2-nistp521`, `diffie-hellman-group-exchange-sha256`

### 1.5 测试

然后用`ssh`命令分别测试：

```
ssh -T git@github.com
```

## 2. 关于用户名

如果之前有设置全局用户名和邮箱的话，需要`unset`一下。

```
git config --global --unset user.name
git config --global --unset user.email
```

然后在不同的仓库下设置局部的用户名和邮箱。

比如在公司的`repository`下
```
git config user.name "yourname"
git config user.email "youremail"
```

在自己的`github`的仓库在执行刚刚的命令一遍即可。

这样就可以在不同的仓库，已不同的账号登录。

## 3. 说明

本文转载自歪麦博客，原文地址：[>>>点我进入](https://www.awaimai.com/2200.html)