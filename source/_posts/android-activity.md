---
title: Android指东：活动 Activity
date: 2019-08-20 10:59:08
tags:
- android
categories:
- Android指南
---

Android开发过程中的Activity必知必会。

<!-- more -->

## 1. Activity 的状态

在 `android` 中，`Activity` 拥有四种基本状态：

- `Active/Runing` 一个新 Activity 启动入栈后，它在屏幕最前端，处于栈的最顶端，此时它处于可见并可和用户交互的激活状态
- `Paused` 当 Activity 被另一个透明或者 Dialog 样式的 Activity 覆盖时的状态。此时它依然与窗口管理器保持连接，系统继续维护其内部状态，所以它仍然可见，但它已经失去了焦点故不可与用户交互
- `Stoped` 当 Activity 被另外一个 Activity 覆盖、失去焦点并不可见时处于 Stoped 状态
- `Killed` 当 Activity 被系统杀死回收或者没有被启动时处于 Killed 状态

`Android` 是通过一种 `Activity` 栈的方式来管理 `Activity` 的，一个 `Activity` 的实例的状态决定它在栈中的位置。处于前台的 `Activity` 总是在栈的顶端，当前台的 `Activity` 因为异常或其它原因被销毁时，处于栈第二层的 `Activity` 将被激活，上浮到栈顶。当新的 `Activity` 启动入栈时，原 `Activity` 会被压入到栈的第二层。一个 `Activity` 在栈中的位置变化反映了它在不同状态间的转换。

![Activity 的状态与它在栈中的位置关系](https://www.ibm.com/developerworks/cn/opensource/os-cn-android-actvt/image002.jpg)

除了最顶层即处在 `Active` 状态的 `Activity` 外，其它的 `Activity` 都有可能在系统内存不足时被回收，一个 `Activity` 的实例越是处在栈的底层，它被系统回收的可能性越大。

## 2. Activity 生命周期

- `protected void onCreate(Bundle savedInstanceState)` 一个 Activity 的实例被启动时调用的第一个方法。一般情况下都要在这里从 xml 中加载设计好的用户界面。当然，也可从 savedInstanceState中读我们保存到存储设备中的数据，但是需要判断 savedInstanceState是否为 null，因为 Activity 第一次启动时并没有数据被存贮在设备中：
- `protected void onStart()` 在 Activity 从 Pause 状态转换到 Active 状态时被调用。
- `protected void onResume()` 在 Activity 从 Active 状态转换到 Pause 状态时被调用。
- `protected void onStop()` 在 Activity 从 Active 状态转换到 Stop 状态时被调用。一般我们在这里保存 Activity 的状态信息。
- `protected void onDestroy()` 在 Active 被结束时调用，它是被结束时调用的最后一个方法，在这里一般做些释放资源，清理内存等工作。

![activity_lifecycle](https://developer.android.com/guide/components/images/activity_lifecycle.png)

## 3. 启动另外一个 Activity

```
Intent intent = new Intent(CurrentActivity.this,OtherActivity.class); 
startActivity(intent);
```

当然，OtherActivity同样需要在 AndroidManifest.xml 中定义。

### 3.1 优雅地启动另一个 Activity

有时，另一个`Activity`可能不是你开发的，但是你想启动该`Activity`，却不知道应该传递哪些数据。如下的封装可以优雅地解决这个问题哦。

```java
public class BaseActivity extends AppCompatActivity {
    // ...
    public static void launch(Context context, String data){
        Intent intent = new Intent(context, BaseActivity.class);
        intent.putExtra("param",data);
        context.startActivity(intent);
    }
}
```

### 3.2 知晓当前Activity

新建一个`BaseActivity`，在`onCreate`方法中打印所需要的信息。

```java
public class BaseActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Log.d("BaseActivity", getClass().getSimpleName());
    }
}
```

让以后新建的`Activity`继承`BaseActivity`，而不是`AppCompatActivity`即可。

```java
public class MainActivity extends BaseActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

## 4. Activity 之间通信

### 4.1 使用 Intent 通信

在 `Android` 中，不同的 `Activity` 实例可能运行在一个进程中，也可能运行在不同的进程中。因此我们需要一种特别的机制帮助我们在 `Activity` 之间传递消息。`Android` 中通过 `Intent` 对象来表示一条消息，一个 `Intent` 对象不仅包含有这个消息的目的地，还可以包含消息的内容，这好比一封 `Email`，其中不仅应该包含收件地址，还可以包含具体的内容。

```
Intent intent = new Intent(CurrentActivity.this,OtherActivity.class);
intent.putExtra(key, value);
startActivity(intent);
```

在 `OtherActivity`类的 `onCreate()`或者其它任何地方使用下面的代码就可以打开这封`e-mail`阅读其中的信息：

```
Intent intent = getIntent();
Bundle bundle = intent.getBundleExtra("key");
// or
String string = intent.getStringExtra();
// or
Serializable str = intent.getSerializableExtra();
```

### 4.2 使用 SharedPreferences

`SharedPreferences` 使用 `xml` 格式为 `Android` 应用提供一种永久的数据存贮方式。对于一个 `Android` 应用，它存贮在文件系统的 `/data/ data/your_app_package_name/shared_prefs/`目录下，可以被处在同一个应用中的所有 `Activity` 访问。`Android` 提供了相关的 `API` 来处理这些数据而不需要程序员直接操作这些文件或者考虑数据同步问题。

```
// 写入 SharedPreferences 
SharedPreferences preferences = getSharedPreferences("name", MODE_PRIVATE); 
Editor editor = preferences.edit(); 
editor.putBoolean("boolean_key", true); 
editor.putString("string_key", "string_value"); 
editor.commit(); 
        
// 读取 SharedPreferences 
SharedPreferences preferences = getSharedPreferences("name", MODE_PRIVATE); 
preferences.getBoolean("boolean_key", false); 
preferences.getString("string_key", "default_value");
```


## 5. Activity 的 Intent Filter

`Action` 是一个用户定义的字符串，用于描述一个 `Android` 应用程序组件，一个 `Intent Filter` 可以包含多个 `Action`。在 `AndroidManifest.xml` 的 `Activity` 定义时可以在其 `<intent-filter>`节点指定一个 `Action` 列表用于标示 `Activity` 所能接受的“动作”，例如：

`Intent Filter` 描述了一个组件愿意接收什么样的 `Intent` 对象，`Android` 将其抽象为 `android.content.IntentFilter` 类。在 `Android` 的 `AndroidManifest.xml` 配置文件中可以通过 `<intent-filter>`节点为一个 `Activity` 指定其 `Intent Filter`，以便告诉系统该 `Activity` 可以响应什么类型的 `Intent`。


![Activity 种 Intent Filter 的匹配过程](https://www.ibm.com/developerworks/cn/opensource/os-cn-android-actvt/image004.jpg)

`Intent`的过滤器(`intent-filter`)按照以下优先关系查找：`action -> data -> category`

## 5.1 Category 匹配

```xml
<intent-filter>
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.TEST1" />
    <category android:name="android.intent.category.TEST2" />
</intent-filter>
```

一个`intent`对象也可以关联多个`category`。为了能让`intent`对象通过`intent-filter`的`category`测试，`intent`对象中的所有`category`都要在`intent-filter`中找到对应项。 

这意味着以下的`intent`对象能够通过`category`测试

```
Intent intent = new Intent();
intent.addCategory("android.intent.category.TEST1");
intent.addCategory("android.intent.category.TEST2");
```

此处需要特别说明的是，我们在上面的示例中，都给`Activity`的`intent-filter`添加了值为`android.intent.category.DEFAULT`的`category`，这是因为当我们把一个隐式的`intent`传递给`startActivity()`或`startActivityForResult()`方法时，`Android`会自动给该隐式`intent`添加值为`android.intent.category.DEFAULT`的`category`，所以为了能让`intent-filter`包含`intent`中全部的`category`，我们就需要在`Activity`的`intent-filter`中添加该`category`，在使用时需要特别注意。

** `Android` 预定义了一系列的 `Action` 分别表示特定的系统动作。这些 `Action` 通过常量的方式定义在 `android.content.Intent`中


### 5.2 Action 匹配

`Action` 是一个用户定义的字符串，用于描述一个 `Android` 应用程序组件，一个 `Intent Filter` 可以包含多个 `Action`。在 `AndroidManifest.xml` 的 `Activity` 定义时可以在其 `<intent-filter>`节点指定一个 `Action` 列表用于标示 `Activity` 所能接受的“动作”，例如：

```xml
<activity android:name=".SecondActivity">
        <intent-filter >
            <category android:name="android.intent.category.DEFAULT" />
            <action android:name="test" />
        </intent-filter>
</activity>
```

如果我们在启动一个 `Activity` 时使用这样的 `Intent` 对象：

```
Intent intent =new Intent(); 
intent.setAction("test");
```

那么所有的 `Action` 列表中包含了`"test"`的 `Activity` 都将会匹配成功。

### 5.3 Data 匹配

一个 `Intent` 可以通过 `URI` 携带外部数据给目标组件。在 `<intent-filter>`节点中，通过 `<data/>`节点匹配外部数据。

`mimeType` 属性指定携带外部数据的数据类型，`scheme` 指定协议，`host、port、path` 指定数据的位置、端口、和路径。如下：

```
<data android:mimeType="mimeType" android:scheme="scheme" android:host="host" android:port="port" android:path="path"/>
<data android:mimeType="video/mpeg" android:scheme="http://localhost:3000/path/"  />
```

一个`intent-filter`下可以有多个`<data/>`标签，`intent`对象无需通过所有的`<data/>`标签测试，一般情况下，`intent`对象只需通过了其中一个`<data/>`标签的测试，那么该`intent`对象就通过了该`intent-filter`的`data`测试。 


## 6. 一些关于 Activity 的技巧

### 6.1 锁定 Activity 运行时的屏幕方向

`Android` 内置了方向感应器的支持。Android 会根据所处的方向自动在竖屏和横屏间切换。但是有时我们的应用程序仅能在横屏 / 竖屏时运行，比如某些游戏，此时我们需要锁定该 `Activity` 运行时的屏幕方向，`<activity>`节点的 `android:screenOrientation`属性可以完成该项任务，示例代码如下：

```xml
<activity android:name=".EX01"
    android:label="@string/app_name"
    android:screenOrientation="portrait"> // 竖屏 , 值为 landscape 时为横屏
 </activity>
```


### 6.2 全屏的 Activity

要使一个 `Activity` 全屏运行，可以在其 `onCreate()`方法中添加如下代码实现：

```
// 设置全屏模式
getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN); 
// 去除标题栏
requestWindowFeature(Window.FEATURE_NO_TITLE);
```

### 6.3 在 Activity 的 Title 中加入进度条 (测试不通过)


为了更友好的用户体验，在处理一些需要花费较长时间的任务时可以使用一个进度条来提示用户“不要着急，我们正在努力的完成你交给的任务”。

在 `Activity` 的标题栏中显示进度条不失为一个好办法，下面是实现代码：

```
// 不明确进度条
requestWindowFeature(Window.FEATURE_INDETERMINATE_PROGRESS); 
setContentView(R.layout.main); 
setProgressBarIndeterminateVisibility(true); 
 
// 明确进度条
requestWindowFeature(Window.FEATURE_PROGRESS); 
setContentView(R.layout.main); 
setProgress(5000);
```

## 7. 更多

`Android`中`Intent`对象与`Intent Filter`过滤匹配过程详解：[>>>点我进入](https://blog.csdn.net/iispring/article/details/48481793)

详解 `Android` 的 `Activity` 组件：[>>>点我进入](https://www.ibm.com/developerworks/cn/opensource/os-cn-android-actvt/index.html)

























