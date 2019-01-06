---
title: MongoDB安装和使用
date: 2017-08-29 16:04:01
tags:
- mongodb
- webstorm
categories:
- 后端,不抛弃
---
windows下MongoDB安装和使用，以及Webstorm上MongoDB可视化插件的使用
<!--more-->
# 1.准备条件
从官网安装好相应的MongoDB版本，自选位置建立如下文件夹用来存放MongoDB数据和日志文件

# 2.初始化MongoDB

在MongoDB的安装目录下的bin文件夹下打开命令行窗口，执行命令，初始化数据库存储位置。命令执行成功之后，系统就会提示等待连接。

```shell
mongod --dbpath  D:\MongoData\db
```


如果使用win10的PowerShell

```shell
.\mongod --dbpath  D:\MongoData\db
```

![](http://img.blog.csdn.net/20170830202641333)

# 3.连接MongoDB数据库

初始化MongoDB存储位置之后，在当前目录下新开一个命令行窗口，输入mongo命令就可以连接上MongoDB数据库了，输入show dbs 进行验证（MongoDB默认存在三个数据库admin，local和test）

```shell
mongo

show dbs
```

![](http://img.blog.csdn.net/20170830204059660)

# 4.以Windows Service的方式启动MongoDB

在目前的情况下，第一个命令行窗口不能关闭，一旦关闭我们就无法使用MongoDB数据库，为了能更方便的使用MongoDB服务，我们可以以Windows Service的方式启动MongoDB服务。

依旧是在当前目录下，以管理员身份打开命令行窗口，输入以下命令。


```shell
mongod -dbpath "D:\MongoData\db" -logpath "D:\MongoData\log\MongoDB.log" -install -serviceName "mongodb"
```

此时服务已经安装成功，运行

```shell
net start mongodb   (开启服务)

net stop mongodb   (关闭服务)
```


# 5.使用webstorm可视化插件

（1）从Intellij插件库中安装Mongo插件（这款插件在JetBrains的产品中貌似是通用的）

![](http://img.blog.csdn.net/20170830205647763)

（2）配置

插件安装完成之后，设置中会多出一个Mongo Servers选项。如果你希望可以直接在webstorm中使用MongoDB的命令行工具，Path to Mongo Shell 选项一定要指向mongo.exe，而不是mongod.exe或其它。因为指向mongod.exe，点 Test 也可以验证通过，但是Mongo Shell是用不了的

![](http://img.blog.csdn.net/20170830210528087)

（3）添加Mongo Server

右上，绿色的加号，添加Mongo Server。MongoDB数据库初始化是没有权限验证，可以自己设定，初学者可以先忽略该部分。在添加服务的选项页中Label，顾名思义添加标签方便自己记忆，主要设置你要使用的数据库User Database选项即可。在Mongo服务开启的情况下，点击 Test Connection 进行测试。

![](http://img.blog.csdn.net/20170830210946650)

（4）最后

下图就是webstorm的可视化工具，点击红色箭头所指的图标，可以打开Mongo Shell

![](http://img.blog.csdn.net/20170830211726887)

在Mongo Shell中输入命令 show collections，使用 Ctrl+Enter 执行命令

![](http://img.blog.csdn.net/20170830211828701)

# 6.使用mongoose进行CRUD

（1）安装mongoose

```shell
npm install mongoose --save
```

（2）使用mongoose建立连接

SQL中的数据库表叫做table，对应MongoDB数据库中的collection。使用mongoose新建collection时，系统会默认collection为复数，所以当你新建一个名为"person"的表时，在数据库中的实际名称为"people"。解决的一种方法是指定collectionName，并作为参数传入，如下面代码第9-10行所示。

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
  
User.remove({name:'jiwei'},function () {        //删除操作  
    console.log("Remove success!")  
});  
User.create({       //增加记录  
    name:'jiwei',  
    age:20,  
    sex:'female'  
},function () {  
    console.log("Insert success!")  
});  
User.find({name: 'wuwei'}, function (err, data) {   //查询操作  
    if (err) {  
        console.log("find failure");  
    } else {  
        console.log("find success!");  
        data.forEach((item)=>console.log(item));  
    }  
});  
User.findOneAndUpdate({name:'wuwei'},{age:21},function () {     //查询并更新  
    console.log("Update success!")  
});  
User.where({name:'wuwei'}).update({$set:{age:22}},function () {     //更新操作  
    console.log("Update success!");  
    mongoose.disconnect(function () {       //断开数据库连接  
        console.log("Mongo is closed!")  
    });  
});  
```