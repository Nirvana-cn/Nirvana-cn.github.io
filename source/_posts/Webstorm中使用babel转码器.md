---
title: Webstorm中使用babel转码器
date: 2018-01-06 16:09:45
tags:
---
首先我使用的是目前最新版的webstorm-2017.2.2，不确定老版本是否一致。

1. 安装babel-cli

```shell
npm install babel-cli --save
```

安装babel会报错，提醒你卸载babel安装babel-cli

2. 配置File Watcher

选择File -> setting -> Tools -> File Watchers,右上角添加Babel

![](http://img.blog.csdn.net/20170830194601801)

进入babel配置选项页，如果没有特殊要求，使用默认配置即可。Arguments：命令执行参数，参见[Babel CLI](https://babeljs.io/docs/usage/cli/)。其中无论你是全局安装babel-cli还是局部安装babel-cli，Program选项指向包安装目录中的babel.cmd即可

![](http://img.blog.csdn.net/20170830194644221)

3. 安装babel-preset-env

配置好之后babel还是无法运行的，因为在babel配置选项中Arguments有一个 --presets env参数会报错，提示找不到，所以还需要安装babel-preset-env

```shell
npm install babel-preset-env --save
```

注意，这里babel-preset-env一定要本地安装，不可以全局安装。

env在这里是可选项，如果你想使用es2015或其它的，只要把 --presets 后面换成es2015，即 --presets es2015，然后本地安装babel-preset-es2015即可。

测试效果如下

![](http://img.blog.csdn.net/20170830194721879)

![](http://img.blog.csdn.net/20170830194727879)

4. 疑问

在实际测试过程我发现ES6中的import语法经过babel转换后并不能直接使用，而必须使用webpack打包之后才有效，思考之后觉得应该是ES5没有模块的概念，babel转码之后使用的是Node中的CommonJS规范，在浏览器中并不适用，而经过webpack打包之后实际是一个JS文件，就不存在模块之间相互调用的关系了，所以可以直接在浏览器中运行。

补：在最新的node-v8.5.0版本中已经支持ES6的module语法