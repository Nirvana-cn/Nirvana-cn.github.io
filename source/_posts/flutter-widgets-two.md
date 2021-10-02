---
title: Flutter Widget — Part 2
date: 2021-06-05 16:28:34
tags:
- flutter
categories:
- Flutter指南
---

`Flutter`中有数目繁多的`widget`，多了解一个`widget`就少造一个轮子😋 😋 😋 。

<!--more-->

## 11. Table
```
Table({
  Key key,
  this.children = const <TableRow>[],
  this.columnWidths,
  this.defaultColumnWidth = const FlexColumnWidth(1.0),
  this.textDirection,
  this.border,
  this.defaultVerticalAlignment = TableCellVerticalAlignment.top,
  this.textBaseline,
})
```

`Table`的构造函数如上，
1. `children`类型为`<TableRow>[]`；
2. `defaultColumnWidth`表示默认每一列的宽度，可以取固定值（`FixedColumnWidth`），也可以取弹性值（`FractionColumnWidth`），如果`columnWidths`不为`null`，则该值不生效；
3. `columnWidths`值类型为`Map<int, TableColumnWidth>` ，可以定制每一列的宽度；
4. `defaultVerticalAlignment`表示每一个单元格垂直方向上如何对齐，默认为顶部对齐。

```
Table(
  // defaultColumnWidth: FractionColumnWidth(0.25),
  defaultColumnWidth: FixedColumnWidth(130),
  columnWidths: {
    0: FractionColumnWidth(0.3),
    1: FractionColumnWidth(0.1),
    2: FractionColumnWidth(0.2),
  },
  defaultVerticalAlignment: TableCellVerticalAlignment.middle,
  border: TableBorder.all(
    color: Colors.blueGrey,
    width: 2,
  ),
  children: [
    TableRow(
      children: [
        WrapperWidget("1", height: 100, color: Colors.greenAccent,),
        WrapperWidget("2"),
        WrapperWidget("3"),
      ]
    ),
    TableRow(
        children: [
          WrapperWidget("4"),
          WrapperWidget("5", height: 70, color: Colors.green,),
          WrapperWidget("6"),
        ]
    ),
  ],
)
```

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/table.png)


## 12. SliverAppBar

在我们介绍`SliverAppBar`，我们先看一下牛津词典对`sliver`的解释：

```
a very small, thin piece of something, usually broken off something larger
碎片，薄片
```

理解`sliver`还是很重要，不然`AppBar`就是`AppBar`，加个`sliver`图啥，对不对。直译过来就是这是一个`AppBar`，但又只是一个碎片，这说明`SliverAppBar`不能单独使用，需要一些组件的配合。`CustomScrollView`是`Flutter`提供的可以用来自定义滚动效果的组件，它可以像胶水一样将多个`Sliver`粘合在一起，组成一个大的滑动区域。

`Sliver`家族还有常用的几个：`SliverList，SliverGrid，SliverPadding，SliverFixedExtentList，SliverPrototypeExtentList，SliverToBoxAdapter，SliverPersistentHeader`。

所以`SliverAppBar`组件通常需要结合`CustomScrollView`、`NestedScrollView`使用。

```
SliverAppBar({
  Key key,
  this.leading,
  this.automaticallyImplyLeading = true,
  this.title,
  this.actions,
  this.flexibleSpace,
  this.bottom,
  this.elevation,
  this.forceElevated = false,
  this.backgroundColor,
  this.brightness,
  this.iconTheme,
  this.actionsIconTheme,
  this.textTheme,
  this.primary = true,
  this.centerTitle,
  this.titleSpacing = NavigationToolbar.kMiddleSpacing,
  this.expandedHeight,
  this.floating = false,
  this.pinned = false,
  this.snap = false,
  this.stretch = false,
  this.stretchTriggerOffset = 100.0,
  this.onStretchTrigger,
  this.shape,
})
```

`SliverAppBar`的构造函数如上，属性比较多，我们就不一一说明了，一些属性可以参考`AppBar`：
1. `expandedHeight`用来设置导航栏完全展开时的高度；
2. `primary`代表导航栏是否显示在页面顶部，默认为`true`，此时会根据`MediaQuery`添加`padding`，使得工具栏不被设备状态栏遮挡；
3. `leading`和`actions`用来扩展工具栏，增加新功能；
4. `floating、pinned、snap`三个属性用来设置滚动的动画效果和行为，具体可以参考[SliverAppBar文档](https://api.flutter.dev/flutter/material/SliverAppBar-class.html)，文档提供了详细的说明和demo演示；
5. `brightness`用来控制状态栏主题，默认`Brightness.dark`，可选参数`light`；
5. `stretch`控制`AppBar`是否可以被拉伸；
6. 当`stretch`为`true`，`stretchTriggerOffset, onStretchTrigger`才会生效，当`AppBar`被拉伸到指定的`stretchTriggerOffset`值时，会触发`onStretchTrigger`回调；
7. `flexibleSpace`可以理解为`SliverAppBar`的背景内容区

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/SliverAppBar.gif)


## 13. SliverList＆SliverGrid

如果对`Sliver`感到陌生，需要回头复习一下第12节`SliverAppBar`。

如果你想要同时滚动一个列表和一个网格，或想要一个很酷和复杂的滚动效果，`SliverList`和`SliverGrid`正是您需要的！

通过在`CustomScrollView`中组合`SliverList`和`SliverGrid`，可以得到一个包含列表项和网格项的滚动列表。

```
CustomScrollView(
        slivers: [
          SliverList(
            delegate: SliverChildBuilderDelegate(
              (context, index) => Container(
                height: 50,
                color: randomColor(),
              ),
              childCount: 10,
            ),
          ),
          SliverPadding(
            padding: EdgeInsets.only(top: 5),
          ),
          SliverGrid(
            delegate: SliverChildBuilderDelegate(
              (context, index) => Container(
                color: randomColor(),
              ),
              childCount: 40,
            ),
            gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 4,
              mainAxisSpacing: 5.0,
              crossAxisSpacing: 5.0,
              childAspectRatio: 1.0,
            ),
          )
        ],
      )
```
![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/SliverList1.gif)

因为`CustomScrollView`只接受`Sliver`系列的组件，所以对于`非Sliver`系列的组件可以使用`SliverToBoxAdapter`包裹。通过`SliverToBoxAdapter`，我们在`CustomScrollView`中嵌套了一层`ListView`，实现了局部滚动和整体滚动。

```
CustomScrollView(
        slivers: [
          SliverToBoxAdapter(
            child: Container(
              height: 300,
              child: ListView.builder(
                itemCount: 50,
                itemExtent: 50.0,
                itemBuilder: (BuildContext context, int index) {
                  return Container(
                    height: 50,
                    color: randomColor(),
                  );
                },
              ),
            ),
          ),
          SliverPadding(
            padding: EdgeInsets.only(top: 5),
          ),
          SliverGrid(
            delegate: SliverChildBuilderDelegate(
              (context, index) => Container(
                color: randomColor(),
              ),
              childCount: 40,
            ),
            gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 4,
              mainAxisSpacing: 5.0,
              crossAxisSpacing: 5.0,
              childAspectRatio: 1.0,
            ),
          )
        ],
      )
```

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/SliverList2.gif)


## 14. FadeInImage

为避免图像在加载时弹出到屏幕上的刺眼效果，请尝试使用`FadeInImage`！您可以指定本地占位符图像并设置高度和宽度参数，以确定图像加载后应占用的空间大小。

```
FadeInImage({
  Key key,
  @required this.placeholder,
  @required this.image,
  this.excludeFromSemantics = false,
  this.imageSemanticLabel,
  this.fadeOutDuration = const Duration(milliseconds: 300),
  this.fadeOutCurve = Curves.easeOut,
  this.fadeInDuration = const Duration(milliseconds: 700),
  this.fadeInCurve = Curves.easeIn,
  this.width,
  this.height,
  this.fit,
  this.alignment = Alignment.center,
  this.repeat = ImageRepeat.noRepeat,
  this.matchTextDirection = false,
}
```

`FadeInImage`的构造函数如上：
1. `placeholder`为类型为`ImageProvider`的占位符图片，`fadeOutCurve`和`fadeOutDuration`用来设置占位图片隐藏时的动画效果和时长；
2. `image`为类型为`ImageProvider`的实际图片，`fadeInCurve`和`fadeInDuration`用来设置图片出现时的动画效果和时长。


```
FadeInImage(
  width: 300,
  height: 300,
  fit: BoxFit.cover,
  fadeInDuration: Duration(seconds: 1),
  placeholder: AssetImage("images/game.png"),
  image: NetworkImage("https://image"),
)
```
同时，`FadeInImage`还提供了从网络中加载图片的构造函数`FadeInImage.assetNetwork`和从内存中加载图片的构造函数`FadeInImage.memoryNetwork`。


## 15. StreamBuilder

我们知道，在`Dart`中`Stream`也是用于接收异步事件数据，和`Future`不同的是，它可以接收多个异步操作的结果，它常用于会多次读取数据的异步任务场景，如网络内容下载、文件读写等。`StreamBuilder`正是用于配合`Stream`来展示流上事件（数据）变化的`UI`组件。

下面是一个简单的例子，每点击一次`FloatingActionButton`，页面数字会加1，即`UI`会响应`stream`的变化,而不需要调用`setState`。最后当`_counter`增加至20时，`stream`被关闭。

```
class _MyHomePageState extends State<MyHomePage> {
  StreamController<int> _controller;
  int _counter = 0;

  @override
  void initState() {
    super.initState();
    _controller = new StreamController<int>();
  }

  void _incrementCounter() {
    _counter++;
    if(_counter == 21){
      _controller.close();
    }
    _controller.add(_counter);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: StreamBuilder<int>(
          stream: _controller.stream,
          builder: (BuildContext context, AsyncSnapshot<int> snapshot) {
            if (snapshot.hasError) return Text('Error: ${snapshot.error}');
            switch (snapshot.connectionState) {
              case ConnectionState.none:
                return Text('没有Stream');
              case ConnectionState.waiting:
                return Text('等待数据...');
              case ConnectionState.active:
                return Text('Active: ${snapshot.data}');
              case ConnectionState.done:
                return Text('Stream已关闭');
            }
            return null;
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          _incrementCounter();
        },
        child: Icon(Icons.add),
      ),
    );
  }
}
```

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/StreamBuilder.gif)

## 16. InheritedModel

在`Flutter`中，可以使用`InheritedWidget`（参考第37小节）跨组件更新数据，但是`InheritedWidget`更新的时候，会调用`updateShouldNotify`方法询问是否需要通知观察者更新。这个更新粒度很粗，所有观察者都会更新。

如果需要共享的数据有多个，比如有 `name` 和 `age`，有两个子组件，一个依赖 `name`，一个依赖 `age`，如果使用 `InheritedWidget` 组件，任何一个数据发生变化，这两个组件都会被通知更新。

当某个组件`rebuild`的开销很昂贵时，`InheritedModel`则是更好的选择。

具体的内容可以参考[Flutter中的数据传递](https://juju.one/inheritedwidget-inheritedmodel/)这篇文章：

```dart
class Inherited extends InheritedModel<String> {
  final int num1;
  final int num2;
  final DataContainerState parent;

  static Inherited of(BuildContext context, String aspect) {
    return InheritedModel.inheritFrom<Inherited>(context, aspect: aspect);
  }

  Inherited({
    Key key,
    @required Widget child,
    this.num1,
    this.num2,
    this.parent,
  }): super(key: key, child: child);

  @override
  bool updateShouldNotify(Inherited oldWidget) {
    return num1 != oldWidget.num1 || num2 != oldWidget.num2;
  }

  @override
  bool updateShouldNotifyDependent(Inherited oldWidget,
      Set<String> dependencies) {
    print(dependencies);
    var updateA =  num1 != oldWidget.num1 && dependencies.contains('count1');
    var updateB = num2 != oldWidget.num2 && dependencies.contains('count2');
    return updateA || updateB;
  }
}

class Counter1 extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    Inherited c = Inherited.of(context, 'count1');
    return RaisedButton(
      child: Text('${c.num1}'),
      onPressed: c.parent.incNum1,
    );
  }
}
```

`aspect`可以理解为注入一名观察者，当`InheritedModel`的属性修改的时候，`InheritedModel`会对观察者们一一调用`updateShouldNotifyDependent`方法，确定观察者是否需要更新。

当`aspect`为`null`时，`InheritedModel.inheritFrom`方法等同于`context.dependOnInheritedWidgetOfExactType<T>()`。

## 17. ClipRRect

`ClipRRect`中额外的`R`代表`Rounded`，所以`ClipRRect`的作用是裁剪圆角矩形。如果想尝试其它形状，可以使用`ClipPath`（路径裁剪，详见第74小节），`ClipOval`(圆形裁剪，详见第79小节)。

```
ClipRRect({
  Key key,
  this.borderRadius = BorderRadius.zero,
  this.clipper,
  this.clipBehavior = Clip.antiAlias,
  Widget child,
})
```

属性也比较简单，`borderRadius`用来控制裁剪的圆角矩形的曲率，默认为直角边。当`clipper`不为`null`时，`borderRadius`会被忽略。`clipBehavior`用来控制裁剪的内容，值为[Clip类型的枚举值](https://api.flutter.dev/flutter/dart-ui/Clip-class.html)。

对于`ClipRRect、ClipPath、ClipOval`组件，`clipper`这个属性可以让我们自己提供裁剪区域，`clipper`值是一个 `CustomClipper` 类型。

```
abstract class CustomClipper<T> {

  T getClip(Size size);

  Rect getApproximateClipRect(Size size) => Offset.zero & size;

  bool shouldReclip(covariant CustomClipper<T> oldClipper);
}
```

由于`CustomClipper`是抽象类，我们需要实现其中的`getClip`和`shouldReclip`方法。我们实现的`clipper`如下，最终的效果如下图中绿色圆角矩形所示：

```
class MyClipper extends CustomClipper<RRect> {
  @override
  RRect getClip(Size size) {
    return RRect.fromLTRBXY(0, 0, 80, 80, 20, 50);
  }

  @override
  bool shouldReclip(CustomClipper<RRect> oldClipper) {
    return true;
  }
}
```

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/ClipRRect.png)

实际生产中，我们设置`ClipRRect`的`clipper`的需求可能不多，通常情况下设置`borderRadius`基本就能满足需求，但是在`ClipPath`中`clipper`的作用特别重要。实际上，我们可以通过`ClipPath`实现`ClipRRect`和`ClipOval`的效果。（还有更厉害的Canvas😆）


## 18. Hero

`Hero`指的是可以在路由(页面)之间“飞行”的`widget`，简单来说`Hero`动画就是在路由切换时，有一个共享的`widget`可以在新旧路由间切换。由于共享的`widget`在新旧路由页面上的位置、外观可能有所差异，所以在路由切换时会从旧路逐渐过渡到新路由中的指定位置，这样就会产生一个`Hero`动画。

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/hero_1.gif)

注意`URL`的变化，我们可以看到当推入新的路由或路由被推出时，相机图标会自动施加一个动画效果，这一切都是由`Hero`组件完成的。实现`Hero`动画只需要用`Hero`组件将要共享的`widget`包装起来，并提供一个相同的`tag`即可，中间的过渡帧都是`Flutter Framework`自动完成的。必须要注意， 前后路由页的共享`Hero`的`tag`必须是相同的，`Flutter Framework`内部正是通过`tag`来确定新旧路由页`widget`的对应关系的。

`Hero`动画的原理比较简单，`Flutter Framework`知道新旧路由页中共享元素的位置和大小，所以根据这两个端点，在动画执行过程中求出过渡时的插值（中间态）即可，而感到幸运的是，这些事情不需要我们自己动手，`Flutter`已经帮我们做了！

同时，`Hero`还提供了`placeholderBuilder`属性，顾名思义就是一个占位`widget`。

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/hero_2.gif)

如上图所示，当`Hero`动画触发时，我们提供的黄色方块会替代原来的相机`icon`。

## 19. CustomPaint

`CustomPaint`十分有趣，它向我们提供了高效且快速调用`Canvas`实现自绘图形的能力。

后面会单独开一个章节详细说明。

待续...

## 20. Tooltip

`Dialog`、`Snackbar`以及`BottomSheet`的用法，这些操作提示都是比较重量级的，存在屏幕上的时间较长或者会直接打断用户的操作。而`Tooltip`是一种轻量级的操作提示。当用户长按或某种操作时，就会出现提示。

`Tooltip`的信息会被`Semantics`（详见第48小节）读取，因此屏幕读取工具可以语音化屏幕内容。

```
Tooltip({
  Key key,
  @required this.message,//提示的内容
  this.height = 32.0,//Tooltip的高度
  this.padding = const EdgeInsets.symmetric(horizontal: 16.0),//padding
  this.margin,
  this.verticalOffset = 24.0,//具体内部child Widget竖直方向的距离
  this.preferBelow = true,//是否显示在下面
  this.excludeFromSemantics = false,
  this.decoration,
  this.textStyle,
  this.waitDuration, // 延迟显示tooltip的时间，默认是0毫秒
  this.showDuration, // 当tooltip显示时，多久会被释放，默认是1.5秒
  this.child,
})
```

`Flutter`中一些`Widget`已经内置了`Tooltip`，例如`IconButton、FloatingActionButton`，因此我们只需要提供`message`即可。

## 附录

完整代码仓库地址: [>>>点我进入](https://github.com/Nirvana-cn/Flutter-widget-weekly)

在Flutter中创建有意思的滚动效果 - Sliver系列: [>>>点我进入](https://juejin.cn/post/6844903901720739848)

Flutter实战——CustomScrollView: [>>>点我进入](https://book.flutterchina.club/chapter6/custom_scrollview.html)
