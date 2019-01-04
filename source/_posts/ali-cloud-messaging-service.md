---
title: 阿里云短信服务
date: 2018-05-22 16:28:34
tags:
- 短信服务
categories:
- 后端,不抛弃
---

通过`Node`使用阿里云提供的短信服务功能。

<!--more-->

我们直接从代码出发，了解如何使用阿里云短信服务

首先需要安装`sdk`工具包

```
npm install @alicloud/sms-sdk --save
```

其次是`Node.js`代码

```javascript
/**
 * 云通信基础能力业务短信发送、查询详情以及消费消息示例，供参考。
 */
const SMSClient = require('@alicloud/sms-sdk')
// ACCESS_KEY_ID/ACCESS_KEY_SECRET 根据实际申请的账号信息进行替换
const accessKeyId = 'yourAccessKeyId'
const secretAccessKey = 'yourAccessKeySecret'
//初始化sms_client
let smsClient = new SMSClient({accessKeyId, secretAccessKey})
//发送短信
smsClient.sendSMS({
    PhoneNumbers: '1500000000',
    SignName: '云通信产品',
    TemplateCode: 'SMS_000000',
    TemplateParam: '{"code":"123456"}'
}).then(function (res) {
    //ES6的解构赋值
    let {Code}=res
    if (Code === 'OK') {
        //处理返回参数
        console.log(res)
    }
}, function (err) {
    console.log(err)
})
```

`Node.js`代码很简单，也很清晰，它告诉我们还需要填写个人的`AccessKeyId`，`AccessKeySecret`，`SignName`以及`TemplateCode`。

进入阿里云主页，打开产品与服务中列表，在“云通信”服务中找到短信服务

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p13.png)

应用开发下可以找到我们需要的内容，依次注册AccessKey，添加签名和添加模板

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p14.png)

何为签名，何为模板？

请看下面的例子，【】中包裹的阿里云就是签名，告诉短信接收者该条短信的来源，我们在添加签名时不需要加【】，虽然`Node.js`代码中`SignName`是字符串，但是`sdk`会做验证，所以你提供的`SignName`必须在阿里云短信签名中注册过该`SignName`，否则会报错。
【】后面的内容就属于模板内容。

```
【阿里云】您的验证码${code}，该验证码5分钟内有效，请勿泄漏于他人！
```

** 我一直以为短信服务是阿里云随机生成验证码，然后发送给用户并在res中返回。好吧，我错了，其实是服务器生成，然后阿里云帮你发出去，所以`TemplateParam`字段也不能省略了。