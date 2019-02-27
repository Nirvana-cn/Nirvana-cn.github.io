---
title: Nightwatch入门指南
date: 2019-02-20 16:40:32
tags: 
- nightwatch
categories:
- 前端,每天学一点
---

随着`JavaScript`程序变得越来越复杂，前端测试也变得越来越重要了。`e2e`测试就是端对端测试，简而言之，就是利用一些工具库提供的API使用代码来模拟终端用户在`UI`界面上的操作，比如输入，点击等等。

`nightwatch.js`是一个用于`Web`应用程序和网站的自动化测试框架。它是一个完整的浏览器（端到端）测试解决方案，旨在简化设置持续集成和编写自动化测试的过程。

童鞋们，`nightwatch`了解一下🤣🤣🤣。
<!--more-->

## 1.前言

### 1.1 安装和初始化

大多数情况下，为了执行命令或断言，`nightwatch`需要向`WebDriver`服务发起至少两次请求，第一次请求通过`css`选择器或`Xpath`表达式确定一个元素的位置，第二次请求才会在元素上执行实际的命令或断言。

安装`WebDriver`服务可以通过直接下载二进制文件或使用`NPM`软件包来完成。

可以选择使用`Selenium Standalone Server`来管理各种浏览器驱动程序和服务，但从`Nightwatch 1.0`开始不再需要，也不推荐使用，除非您针对旧浏览器（如`Internet Explorer`）进行测试。

```json
{
"webdriver" : {
    "start_process": true,
    "server_path": "node_modules/.bin/chromedriver",
    "port": 9515
  }
}
```

官方教程实在是造成了严重的误导，关键是你安装完`chromedriver`这个包后，`node_modules/.bin/chromedriver`目录下面确实有一个`chromedriver`文件，但是我们需要的是`chromedriver.exe`文件。

我们可以通过`.path`属性获取包的安装路径（参考`demo-01`），也可以手动输入（参考`demo-05`）。

```javascript
const chromedriver = require('chromedriver');

module.exports = (function(settings) {
  settings.test_workers = false;
  settings.webdriver.server_path = chromedriver.path;
  return settings;
})(require('./nightwatch.json'));
```

`chrome`端口号`9515`。

`firefox`端口号`4444`，特别说明：`firefox`需要传入`debug`参数才能运行，这个坑真是坑死我了😂😂😂。

```json
{
"webdriver": {
    "start_process": true,
    "server_path": "./node_modules/geckodriver/geckodriver.exe",
    "log_path": "output",
    "port": 4444,
    "cli_args": [
      "--log", "debug"
    ]
  }
}
```

** `window`下，`firefox` 默认寻找路径为`C:\Program Files\Mozilla Firefox`，如果安装在其它路径，需要指定`moz:firefoxOptions`属性或者使用`--binary PATH`命令，告诉`geckodriver`启动路径。

当你想要关闭测试时，请记住始终调用`.end()`方法，以便正确关闭浏览器会话。

### 1.2 before[Each] and after[Each] hooks

在测试中，`nightwatch`提供标准的`before/after`和`beforeEach/afterEach`的钩子。

`before`和`after`将分别在执行测试套件之前和之后运行，而`beforeEach`和`afterEach`分别在每个测试用例之前和之后运行（参见demo4-hooks）。

### 1.3 Asynchronous hooks

所有`before[Each] and after[Each]`都可以执行异步操作，只需要将回调函数作为第二个参数传入。

```javascript
module.exports = {
  beforeEach: function(browser, done) {
    // 执行一个异步操作
    setTimeout(function() {
      // 完成异步任务
      done();
    }, 100);
  },

  afterEach: function(browser, done) {
    // 执行一个异步操作
    setTimeout(function() {
      // 完成异步任务
      done();
    }, 200);
  }
};
```

注意：当异步操作完成后，回调函数`done`必需作为最后一个步骤被调用，否则会导致超时错误。默认的超时时间是10s，可以通过指定`asyncHookTimeout`属性修改默认超时时间。

## 2. 运行测试

`nightwatch`提供命令行的方式来运行测试和生成测试报告，同时提供多种配置选项。

```bash
nightwatch [source] [options]
```

### 2.1 指定测试实例source

除了在`nightwatch.json`文件中配置`src_folders`来指定需要进行测试的实例，同样可以在命令行中直接传入测试实例。

```bash
// 测试单个文件
nightwatch tests/one/firstTest.js

// 独立的多个测试
nightwatch tests/one/firstTest.js tests/secondTest.js

// 独立的测试和整个文件夹
nightwatch tests/one/test.js tests/utils
```

### 2.2 命令行选项options

| 名称 | 简写  |  默认值 | 描述 |
| ------ | ------ | ------ | ------ |
|--config|	-c|	./nightwatch.json| 指定启动nightwatch的配置文件
|--output|	-o|		tests_output| 指定测试报告的输出路径
|--reporter|	-r|	junit| 预定义报告者的名称或要使用的自定义报告文件的路径。
|--env|	-e|	default	| 指定nightwatch的测试环境
|--verbose|	|	| 在会话期间显示扩展的selenium命令日志记录
| --version |	-v |   | 显示版本号
| --test |-t	 |   | 指定测试实例
| --testcase |	 |   | 只与--test一起使用
|--group  |	-g |   | 指定测试组
| --skipgroup |	-s |   | 指定需要跳过的测试组
| --filter |-f	 |   | 进行过滤操作
|--tag  |-a	 |   | 通过标签进行过滤
| --skiptags	 |	 |   | 跳过指定的标签
| --retries	 |	 |   | 重试失败或错误的测试用例达到指定的次数。
|--suiteRetries	  |	 |   | 重试失败或错误的测试套件（测试模块）达到指定的次数。


### 2.3 Test Environments

你可以定义测试设置的多个环境，以便覆盖每个环境的特定值。

默认`default`环境是必需的。所有其他环境都是从默认环境继承的，可以根据需要覆盖设置。

```json
{
  "test_settings" : {
    "default" : {
      "launch_url" : "http://localhost",
      "globals" : {
        "myGlobalVar" : "some value",
        "otherGlobal" : "some other value"
      }
    },

    "integration" : {
      "launch_url" : "http://staging.host",
      "globals" : {
        "myGlobalVar" : "other value"
      }
    }
  }
}
```

通过指定`--env`参数，我们可以选取特定的测试环（参见`demo5-env`）。

```bash
nightwatch --env integration
```

### 2.4 Test Tags

我们还可以选择性地将测试目标定位为基于标签运行，以便测试可以属于多个标签。

标签可以通过将`@tags`属性添加到测试模块来完成：

```javascript
module.exports = {
  '@tags': ['login', 'sanity'],
  'demo login test': function (browser) {
     // test code
  }
};
```

通过命令行对测试标签进行控制：

```bash
// 选择特定标签的测试模块
nightwatch --tag login

// 选择多个标签的测试模块
nightwatch --tag login --tag something_else

// 跳过指定标签
nightwatch --skiptags login
```

### 2.5 Disabling Tests

要阻止测试模块运行，只需将该模块中的`disabled`属性设置为`true`，如下所示：

```javascript
module.exports = {
  '@disabled': true, // 这会阻止测试模块运行

  'sample test': function (browser) {
    // test code
  }
};
```

## 3. Page Objects

待续... 

https://blog.csdn.net/qq_25324335/article/details/82052509

## 4. 更多

本文demo源码地址：[>>>点我进入](https://github.com/Nirvana-cn/Learning-nightwatch)




