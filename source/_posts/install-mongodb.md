---
title: MongoDB安装和使用
date: 2017-08-29 16:04:01
tags:
- mongodb
- webstorm
categories:
- 后端,不抛弃
---
总结并记录了`windows`下`MongoDB`数据库的安装和使用，以及`Webstorm`上`MongoDB`可视化插件的使用。
<!--more-->
## 1.准备条件
从官网安装好相应的`MongoDB`版本，自选位置建立如下文件夹用来存放`MongoDB`数据和日志文件。

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p11.png)

## 2.初始化MongoDB

在`MongoDB`的安装目录下的`bin`文件夹下打开命令行窗口，执行命令，初始化数据库存储位置。命令执行成功之后，系统就会提示等待连接。

```shell
mongod --dbpath  D:\MongoData\db
```

如果使用`win10`的`PowerShell`

```shell
.\mongod --dbpath  D:\MongoData\db
```

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p23.png)

## 3.连接MongoDB数据库

初始化`MongoDB`存储位置之后，在当前目录下新开一个命令行窗口，输入`mongo`命令就可以连接上`MongoDB`数据库了，输入`show dbs` 进行验证（`MongoDB`默认存在三个数据库`admin，local和test`）

```shell
mongo

show dbs
```

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p24.png)

## 4.以Windows Service的方式启动MongoDB

在目前的情况下，第一个命令行窗口不能关闭，一旦关闭我们就无法使用`MongoDB`数据库，为了能更方便的使用`MongoDB`服务，我们可以以`Windows Service`的方式启动`MongoDB`服务。

依旧是在当前目录下，以管理员身份打开命令行窗口，输入以下命令。


```shell
mongod -dbpath "D:\MongoData\db" -logpath "D:\MongoData\log\MongoDB.log" -install -serviceName "mongodb"
```

此时服务已经安装成功，运行。

```shell
net start mongodb   (开启服务)

net stop mongodb   (关闭服务)
```


## 5.使用webstorm可视化插件

（1）从`Intellij`插件库中安装`Mongo`插件（这款插件在`JetBrains`的产品中貌似是通用的）

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p25.png)

（2）配置

插件安装完成之后，设置中会多出一个`Mongo Servers`选项。如果你希望可以直接在`webstorm`中使用`MongoDB`的命令行工具，`Path to Mongo Shell` 选项一定要指向`mongo.exe`，而不是`mongod.exe`或其它。因为指向`mongod.exe`，点 `Test` 也可以验证通过，但是`Mongo Shell`是用不了的

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p26.png)

（3）添加Mongo Server

右上，绿色的加号，添加`Mongo Server`。`MongoDB`数据库初始化是没有权限验证，可以自己设定，初学者可以先忽略该部分。在添加服务的选项页中`Label`，顾名思义添加标签方便自己记忆，主要设置你要使用的数据库`User Database`选项即可。在`Mongo`服务开启的情况下，点击 `Test Connection` 进行测试。

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p27.png)

（4）最后

下图就是`webstorm`的可视化工具，点击红色箭头所指的图标，可以打开`Mongo Shell`。

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p28.png)

在`Mongo Shell`中输入命令 `show collections`，使用 `Ctrl+Enter` 执行命令。

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/p29.png)

tip: 安装自带的`MongodbCompress`好像更方便哦~~

## 6.使用mongoose进行CRUD

（1）安装mongoose

```shell
npm install mongoose --D
```

（2）使用mongoose建立连接

`SQL`中的数据库表叫做`table`，对应`MongoDB`数据库中的`collection`。使用`mongoose`新建`collection`时，系统会默认`collection`为复数，所以当你新建一个名为`person`的表时，在数据库中的实际名称为`people`。解决的一种方法是指定`collectionName`，并作为参数传入，如下面代码第9-10行所示。

```javascript
let mongoose = require('mongoose');       //引用mongoose  
let UserSchema = new mongoose.Schema({       //创建数据模板  
    name: String,  
    age: {type: Number, default: 1},  
    sex: {type: String, default: "male"},    //指定数据类型，default选项指数据缺失时的默认值  
    birth: String  
});  
let db = mongoose.connection;    //当使用mongoose.connect()方法连接数据库时，数据库的实例依附在mongoose.connection上  
let collectionName='person';       
let User = mongoose.model('person', UserSchema,collectionName);     //将模板绑定到指定的collection上  
mongoose.connect('mongodb://localhost:27017/test');     //连接数据库  
```
（3）使用mongoose进行数据库操作

```javascript
db.once('open', function () {       //监测数据库实例的状态  
    console.log("Mongo is working");  
});       
db.once('close',function () {       //监测数据库实例的状态  
    console.log("Mongo is closed!")  
});  
  
User.remove({name:'mayun'},function () {        //删除操作
    console.log("Remove success!")  
});  
User.create({       //增加记录  
    name:'mayun',
    age:20,  
    sex:'female'  
},function () {  
    console.log("Insert success!")  
});  
User.find({name: 'zhaoyun'}, function (err, data) {   //查询操作
    if (err) {  
        console.log("find failure");  
    } else {  
        console.log("find success!");  
        data.forEach((item)=>console.log(item));  
    }  
});  
User.findOneAndUpdate({name:'caiyun'},{age:21},function () {     //查询并更新
    console.log("Update success!")  
});  
User.where({name:'caiyun'}).update({$set:{age:22}},function () {     //更新操作
    console.log("Update success!");  
    mongoose.disconnect(function () {       //断开数据库连接  
        console.log("Mongo is closed!")  
    });  
});  
```