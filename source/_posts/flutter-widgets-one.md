---
title: Flutter Widget — Part 1
date: 2019-11-05 16:28:34
tags:
- flutter
categories:
- Flutter指南
---

`Flutter`中有数目繁多的`widget`，多了解一个`widget`就少造一个轮子😋 😋 😋 。

<!--more-->

## 1. 引言

Flutter中有许多的Widget，有些很常用，比如`Container、Row、Column`；而有些就非常生僻，比如`DraggableScrollableSheet 、ColorFiltered、ToggleButtons`。因此在这里我们对`Flutter`中的一些`Widget`进行整理，介绍相关的用法和效果。

本文是按照`Flutter`官方的视频教程[Flutter Widget of the Week](https://www.youtube.com/playlist?list=PLjxrf2q8roU23XGwz3Km7sQZFTdB996iG)来介绍`Flutter`中的`Widget`。同时每一个`Widget`都编写了简单的示例，可以在[Github](https://github.com/Nirvana-cn/Flutter-widget-weekly)上获取示例源码。最后文章部分内容摘自`wendux`的《Flutter 实战》，在此对作者表示感谢。

## 2. SafeArea

在当前手机屏幕不再是规整的矩形，比如`iPhone X`的刘海屏和圆角边框，这种不规则的造型可能会影响我们页面内容的显示。

`SafeArea`通过`MediaQuery`方法对我们的手机屏幕尺寸进行检查，避免需要显示的内容被边框遮挡。如下图所示，左图中部分背景色被刘海屏所遮挡，右图应用`SafeArea`后，顶部的刘海部分没有作为有效的渲染区域。

![safeArea](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/SafeArea.jpg)

```
SafeArea(
  child: SizedBox.expand(
    child: Container(
      color: Colors.red,
    ),
  ),
)      
```

我们甚至可以指定需要安全显示内容的方向。

```
SafeArea(
  child: SizedBox.expand(
    child: Container(
      color: Colors.red,
    ),
  ),
  top: true,
  bottom: true,
  left: false,
  right: false,
)
```
## 3. Expanded

`Expanded`组件必须作为`Column、Row、Flex`的子元素使用，当容易具有额外空间时，`Expanded`组件可以让子元素自动填充剩余的空间。

我们也可以指定`flex`属性，对剩余未使用的空间进行按比例分配。`flex`属性的值是一个整数。

实际上，`Expanded`继承自`Flexible`（详见第34小节），`Expanded`调用了`Flexible`的构造函数，将`fit`属性指定为`FlexFit.tight`。

** `Expanded`会"强制"子元素填充父元素的剩余空间，这会造成子元素的布局约束失效。与之相对的是`Align`（详见第26小节），`Align`会继续让它的子元素保持自己的属性（Thanks, Align 😘）。

## 4. Wrap

我们通常会使用`row`或`column`来进行布局，这非常方便，但是有些时候布局可能会超出尺寸。

`wrap`会像`row`和`column`一样排列`children`，但是当`children`超出容器时（如下图所示），`wrap`会进行自动换行来保证元素没有超出容器大小。

![flex-out](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/flex_out.png)

使用`wrap`后，超出的部分会自动进行换行，就没有溢出边框的风险了。

```
Wrap(
  direction: Axis.horizontal,
  spacing: 10,
  runSpacing: 0,
  children: <Widget>[
      OutlineButton(
        child: Text("A"),
      ),            
      OutlineButton(
        child: Text("B"),
      ),
      OutlineButton(
        child: Text("C"),
      ),
      OutlineButton(
        child: Text("D"),
      ),     
      OutlineButton(
        child: Text("E"),
      ),     
    ],
)
```

![wrap](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/wrap.png)

常用的`wrap`参数如下：

```
Wrap({
  ...
  this.direction = Axis.horizontal,
  this.alignment = WrapAlignment.start, // 子元素在主轴上的对齐方式
  this.runAlignment = WrapAlignment.start, // 各行（列）在交叉轴上的对齐方式
  this.spacing = 0.0, // 主轴上子元素间隔
  this.runSpacing = 0.0, // 交叉轴上各行（列）间隔
  this.crossAxisAlignment = WrapCrossAlignment.start, // 当同一行（列）的子元素宽高不一致时，子元素之间如何对齐
  this.textDirection,
  this.verticalDirection = VerticalDirection.down, // 子元素排列的方式，默认是自上而下，值为 up 时自下而上
  List<Widget> children = const <Widget>[],
})
```

可以认为`Wrap`和`Flex`（包括`Row`和`Column`）除了超出显示范围后`Wrap`会折行外，其它行为基本相同。

许多开发者可能会误把`crossAxisAlignment`作为交叉轴上的对齐方式，这是不正确的，定义`wrap`元素在交叉轴上的属性实际上是`runAlignment`，`crossAxisAlignment`被用来确定子元素的对齐方式。

当`runAlignment: WrapAlignment.start` 时

![竖直居中](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/runAlignment_center.png)

当`runAlignment: WrapAlignment.spaceEvenly` 时

![均匀分配](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/runAlignment_spaceEvenly.png)

## 5. AnimatedContainer

顾名思义，`AnimatedContainer`提供了隐含的动画效果。这些动画效果可用于尺寸、颜色、形状、阴影、变形等等...

`AnimatedContainer`的属性与`Container`（详见第55小节）基本一致，不同的地方在于可以通过`Duration duration`属性控制动画的过渡时长。

```
Color _color = Colors.red;
double _height = 200;

AnimatedContainer(
  width: 200,
  height: _height,
  color: _color,
  duration: Duration(seconds: 2),
  child: FlatButton(
    child: Text(
      "Click me",
      style: TextStyle(
        color: Colors.white,
      ),
    ),
    onPressed: () {
      setState(() {
        _height = 100;
        _color = Colors.green;
      });
    },
  ),
)          

```

## 6. Opacity

当你希望一个元素不可见而依然占据布局空间时，透明度`Opacity`就派上用场了。使用起来非常简单，只需要将元素用`Opacity`包裹，将`opacity`属性设置为0，就可以隐藏该元素了。

```
Opacity(
  opacity: 0.0,
  child: Container(
    margin: EdgeInsets.only(top: 100),
    width: 100,
    height: 100,
    color: Colors.red,
  ),
)  
```

与此同时，`Flutter`还提供了`AnimatedOpacity`（详见第51小节）组件，当透明度改变时会产生一个过渡的动画效果。用法与第5小节中的`AnimatedContainer`一致，通过`duration`属性设置过渡动画的时长。

** `Offstage`同样可以使得元素不可见，同时元素也不再占据布局空间。类似于前端布局中`display: none`。

## 7. FutureBuilder

很多时候我们会依赖一些异步数据来动态更新UI，比如在打开一个页面时我们需要先从互联网上获取数据，在获取数据的过程中我们显示一个加载框，等获取到数据时我们再渲染页面。因此`Flutter`专门提供了`FutureBuilder`和`StreamBuilder`（详见第15小节）两个组件来快速实现这种功能。

`FutureBuilder`的构造函数如下：

```
const FutureBuilder({
  Key key,
  this.future,
  this.initialData,
  @required this.builder,
}) : assert(builder != null),
super(key: key);
```

- future: 参数类型为`Future<T>`，泛型与`snapshot.data`相对应。这个参数建议在 `initState()`里初始化，切记不要在 `build` 方法里初始化，这样的话会一直`rebuild`；
- initialData: 初始化的数据。当`future`尚未完成时，通过该属性可以初始化`snapshot.data`的值。一旦`future`完成，`initialData`将不起作用;
- builder: `Widget`构建器；该构建器会在`future`执行的不同阶段被多次调用。

`builder`构建器签名如下：

> Function (BuildContext context, AsyncSnapshot snapshot)

`snapshot`会包含当前异步任务的状态信息及结果信息 ，比如我们可以通过`snapshot.connectionState`获取异步任务的状态信息（`FutureBuilder`没有`ConnectionState.active`状态）、通过`snapshot.hasError`判断异步任务是否有错误等等。

现在我们使用`FutureBuilder`模拟一个异步过程，间隔3秒后返回一个字符串。代码和实现效果如下：

```
Future<String> _future;

Future<String> mockNetworkData() async {
  return Future.delayed(Duration(seconds: 2), () => '我是异步获取的数据。');
}

@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: Text(widget.title),
    ),
    body: Center(
      child: FutureBuilder(
        future: _future,
        initialData: "Initial data",
        builder: (context, snapshot) {
          switch (snapshot.connectionState) {
            case ConnectionState.none:
              return Center(
                child: Text('None: ${snapshot.data}'),
              );
            case ConnectionState.active:
            case ConnectionState.waiting:
              return Center(
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: <Widget>[
                    CircularProgressIndicator(),
                    Text(" Waiting: ${snapshot.data}"),
                  ],
                ),
              );
            case ConnectionState.done:
              if (snapshot.hasError) {
                return Center(
                  child: Text('异步请求出错'),
                );
              }
              return Center(
                child: Text("Done: ${snapshot.data}"),
              );
          }
          return null;
        },
      ),
    ),
    floatingActionButton: FloatingActionButton(
      onPressed: () {
        setState(() {
          _future = mockNetworkData();
        });
      },
      child: Icon(Icons.add),
    ),
  );
}
```

---
Tips: `future`不要在 `build` 方法里初始化，这样的话会一直`rebuild`？

为什么呢，我们查看 didUpdateWidget 源码：

```
@override
void didUpdateWidget(FutureBuilder<T> oldWidget) {
  super.didUpdateWidget(oldWidget);
  if (oldWidget.future != widget.future) {
    if (_activeCallbackIdentity != null) {
      _unsubscribe();
      _snapshot = _snapshot.inState(ConnectionState.none);
    }
    _subscribe();
  }
}
```

可以看出来这里是判断了 `future` 这个字段，所以我们一定不要在 `build` 方法里初始化 `future` 参数！

---

## 8. FadeTransition

当组件透明度发生变化时，`FadeTransition`可以为该组件增加一个淡入淡出的动画效果。

`FadeTransition`只针对单个组件，如果你希望切换两个子组件时增加一些淡入淡出的效果，请参见`AnimatedCrossFade`（详见第60小节）。

`FadeTransition`的构造函数如下，我们需要为`opacity`提供一个`Animation<double>`类型的值，而不仅仅是`double`。

```
FadeTransition({
  Key key,
  @required Animation<double> opacity,
  bool alwaysIncludeSemantics,
  Widget child,
})
```

`Flutter`中的动画系统基于`Animation`对象的，`widget`可以在`build`函数中读取`Animation`对象的当前值， 并且可以监听动画的状态改变（更多内容需要自行阅读官方教程 [Flutter中的动画](https://flutterchina.club/tutorials/animation/)）。

`Tween`是一种补间动画。在补间动画中，定义了开始点和结束点、时间线以及定义转换时间和速度的曲线。然后由框架计算如何从开始点过渡到结束点。简单来说，就是向开始点和结束点之间插入一系列连续的值，使得转换效果更为顺畅。

`Flutter`中的`Animation`对象是一个在一段时间内依次生成一个区间之间值的类。`Animation`对象的输出可以是线性的、曲线的、一个步进函数或者任何其他可以设计的映射。根据`Animation`对象的控制方式，动画可以反向运行，甚至可以在中间切换方向。

`Animation`还可以生成除`double`之外的其他类型值，如：`Animation<Color>` 或 `Animation<Size>`。`Animation`对象有状态。可以通过访问其`value`属性获取动画的当前值。

一个`Animation`对象可以拥有`Listeners`和`StatusListeners`监听器，可以用`addListener()`和`addStatusListener()`来添加。 只要动画的值发生变化，就会调用监听器。一个`Listener`最常见的行为是调用`setState()`来触发UI重建。动画开始、结束、向前移动或向后移动（如`AnimationStatus`所定义）时会调用`StatusListener`。

```
AnimationController _controller;
Animation<double> _animation;

@override
void initState() {
  super.initState();
  _controller = AnimationController(
    vsync: this,
    duration: Duration(seconds: 2),
  );
  _animation = Tween(begin: 0.2, end: 1.0).animate(_controller)
    ..addListener(() {
      if (_animation.value == 1) {
        _controller.reverse(); // 结束时反向运行
      }
      if (_animation.value == 0.2) {
        _controller.forward(); // 回到开始状态时，重新开始
      }
    });
}
```

`CurvedAnimation`则可以将动画过程定义为一个非线性曲线。`Curves`类定义了许多常用的曲线，也可以创建自己的。

```
AnimationController _controller;
Animation<double> _animation;

@override
void initState() {
  super.initState();
  _controller = AnimationController(
    vsync: this,
    duration: Duration(seconds: 2),
  );
  final CurvedAnimation curve = CurvedAnimation(
    parent: _controller,
    curve: Curves.easeIn,
  );
  _animation = Tween(begin: 0.2, end: 1.0).animate(curve)
    ..addListener(() {
      if (_animation.value == 1) {
        _controller.reverse();
      }
      if (_animation.value == 0.2) {
        _controller.forward();
      }
    });
}
```

动画不会自己自动执行，需要主动调用`controller.forward()`。还有当页面被销毁时记得调用`controller.dispose()`进行释放。最终实现的效果如下：

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/FadeTransition.gif)

## 9. FloatingActionButton

`Flutter`中添加`FloatingActionButton（FAB）`特别简单，按照如下代码，就可以很方便快捷地在`Scaffold`下生成一个`FAB`。

```
Scaffold(
  floatingActionButton: FloatingActionButton(
    onPressed: (){},
    child: Icon(Icons.add),
  ),
)  
```

但是，当你有一个底部导航栏`bottomNavigationBar`，希望`FAB`嵌入时该怎么解决？在`Flutter`中这个问题很简单，只需要设置`floatingActionButtonLocation`属性即可。

```
Scaffold(
  floatingActionButton: FloatingActionButton(
    onPressed: () {},
    child: Icon(Icons.add),
  ),
  floatingActionButtonLocation: FloatingActionButtonLocation.endDocked,
  bottomNavigationBar: BottomAppBar(
    shape: CircularNotchedRectangle(),
    notchMargin: 4.0,
    color: Colors.yellow.shade800,
    child: Container(
      height: 50,
    ),
  ),
)
```

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/FloatingActionButton.png)

嵌入在尾部不满意？没关系，`FloatingActionButtonLocation`还提供了其它可选项，嵌入或悬浮都可以满足。

```
floatingActionButtonLocation: FloatingActionButtonLocation.centerDocked,
floatingActionButtonLocation: FloatingActionButtonLocation.centerFloat,
......
```

`FloatingActionButton.extended` 还提供了一个较宽的`FAB`，它可以包含一个`icon`和一个`label`。

```
FloatingActionButton.extended(
  onPressed: () {},
  icon: Icon(Icons.save),
  label: Text("Save"),
),
```

有关`FloatingActionButton`更加详细的说明和使用示例可以参考`Medium`上的一篇博客 [A Deep Dive Into FloatingActionButton](https://proandroiddev.com/a-deep-dive-into-floatingactionbutton-in-flutter-bf95bee11627)

## 10. PageView

`PageView`创建一个可“逐页”滚动的列表。`PageView`的每个子元素会被强制保持与视口大小相同。

可以使用`PageController`来控制哪个页面在视图中可见。`PageController`也可以用于控制`PageController.initialPage`和`PageController.viewportFraction`，前者确定第一次构造`PageView`时显示哪个页面，后者确定页面大小（视口大小的一部分）。

```
class _MyHomePageState extends State<MyHomePage> {
  final _controller = PageController(initialPage: 0,  viewportFraction: 1.0);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: PageView(
        scrollDirection: Axis.horizontal,
        controller: _controller,
        pageSnapping: true, // 设置页面是否滚动到整数位置
        children: <Widget>[
          Container(
            color: Colors.blue,
          ),
          Container(
            color: Colors.green,
          ),
          Container(
            color: Colors.amber,
          ),
        ],
        onPageChanged: (int page) {
          print("Current Page: " + page.toString());
        },
      ),
    );
  }
}
```

![](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/PageView.gif)

`PageView`的默认构造函数适用于比较少的子`Widget`，`PageView.builder`则用于创建大量或无限的子控件。有关`PageView`更加详细的说明和使用示例可以参考`Medium`上的一篇博客 [A Deep Dive Into PageView](https://medium.com/flutter-community/a-deep-dive-into-pageview-in-flutter-with-custom-transitions-581d9ea6dded)。


## 附录

完整代码仓库地址: [>>>点我进入](https://github.com/Nirvana-cn/Flutter-widget-weekly)
