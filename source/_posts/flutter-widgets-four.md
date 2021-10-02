---
title: Flutter Widget — Part 4
date: 2021-10-02 16:28:34
tags:
- flutter
categories:
- Flutter指南
---

`Flutter`中有数目繁多的`widget`，多了解一个`widget`就少造一个轮子😋 😋 😋 。

<!--more-->

## 31. ValueListenableBuilder

`ValueListenableBuilder`可以注册值变化的回调，这样当监听的值变化时会回调`ValueWidgetBuilder`方法同步值的变化。

```
class _MyHomePageState extends State<MyHomePage> {
  ValueNotifier<int> valueNotifier;


  @override
  void initState() {
    super.initState();
    valueNotifier = ValueNotifier(0);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: ValueListenableBuilder(
          valueListenable: valueNotifier,
          builder: (context, value, child) {
            return Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                Text(
                  'You have pushed the button this many times:',
                ),
                Text(
                  '${valueNotifier.value}',
                  style: Theme.of(context).textTheme.headline4,
                ),
              ],
            );
          },
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          valueNotifier.value++;
        },
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ),
    );
  }
}
```

![ValueListenableBuilder.gif](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/ValueListenableBuilder.gif)

以上代码在官方模板的基础上使用`ValueListenableBuilder`实现了一下，并没有感觉很好用😂😂😂。

不推荐使用该组件进行状态管理，因为`Flutter`本身就是响应式的，还不确定`ValueListenableBuilder`的最佳实践是什么。


## 32. Draggable

`Draggable`是`Flutter`中常用的拖拽组件，它的常用属性如下：

```
const Draggable({
  Key? key,
  required this.child,
  required this.feedback, // 设置拖拽时光标下的组件
  this.data, // 设置携带的数据
  this.axis, // 控制拖动方向
  this.childWhenDragging, // 设置拖拽发生时child的替代组件
  this.feedbackOffset = Offset.zero,
  this.dragAnchor = DragAnchor.child,
  this.affinity,
  this.maxSimultaneousDrags,
  this.onDragStarted,
  this.onDragUpdate,
  this.onDraggableCanceled,
  this.onDragEnd,
  this.onDragCompleted,
  this.ignoringFeedbackSemantics = true,
  this.rootOverlay = false,
})
```

`onDragxx`为拖拽过程的事件回调。`data`携带的数据通常用来进行一些标识，接收方会根据携带的数据判断是否响应。因此，`Draggable`需要与`DragTarget`组件配合使用。

```
const DragTarget({
  Key? key,
  required this.builder,
  this.onWillAccept,
  this.onAccept,
  this.onAcceptWithDetails,
  this.onLeave,
  this.onMove,
})
```
`DragTarget`有3个常用回调，说明如下：

- onWillAccept：拖到该控件上时调用，需要返回`true`或者`false`，返回`true`，松手后会回调`onAccept`，否则回调`onLeave`。
- onAccept：`onWillAccept`返回`true`时，用户松手后调用。
- onLeave：`onWillAccept`返回`false`时，用户松手后调用。

说完了常用回调，`DragTarget`的`builder`属性也是需要我们密切关注的。

```
DragTarget(
  builder: (BuildContext context, List<dynamic> candidateData, List<dynamic> rejectedData) {
      ...
  }
)
```

当`onWillAccept`返回`true`时，`candidateData`参数的数据是`Draggable`的`data`数据。当`onWillAccept`返回`false`时，`rejectedData`参数的数据是`Draggable`的`data`数据。

![Draggable.gif](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/Draggable.gif)

在上面的例子中，我们新建了两个可拖拽对象（square 1和2），但是接收区域根据`data`做了约束，只有`square 1`能被接收。具体来说就是在`onWillAccept`回调中，只有`data`为1时才会返回`true`，完整代码请访问附录中的`github`地址获取。

```
onWillAccept: (int value) {
  if (value == 1) {
  return true;
  }
  
  return false;
},
```

## 33. AnimatedList

`AnimatedList`可以为列表添加一个动画效果，使得对列表元素进行增删改查时过渡更为自然。

```
typedef AnimatedListRemovedItemBuilder = Widget Function(BuildContext context, Animation<double> animation);

```

由于`AnimatedList`的`itemBuilder`函数会回调一个`animation`对象，因此最优解是配合可以接收`Animation<double>`属性的组件使用，比如`SlideTransition`，`FadeTransition`。

```
AnimatedList(
  key: _key,
  itemBuilder: (context, index, animation) {
    return FadeTransition(
      opacity: animation.drive(Tween(begin: 0, end: 1)),
      child: Container(
        margin: EdgeInsets.symmetric(
          vertical: 10,
          horizontal: 30,
        ),
        child: Text("水果 - ${fruit[index]}"),
      ),
    );
  },
  initialItemCount: fruit.length,
)
```

但需要注意的是，若想动画效果生效，除了修改原始数据外，还需要分别调用`AnimatedListState`的`insertItem`和`removeItem`方法。

```
Row(
  mainAxisAlignment: MainAxisAlignment.end,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    FloatingActionButton(
      onPressed: () {
        final int index = fruit.length;
        fruit.add("mango");

        AnimatedListState state = _key.currentState;
        state.insertItem(index);
      },
      child: Icon(Icons.add),
    ),
    FloatingActionButton(
      onPressed: () {
        final int index = fruit.length - 1;
        String name = fruit[index];

        AnimatedListState state = _key.currentState;
        state.removeItem(
          index,
              (context, animation) => FadeTransition(
            opacity: animation,
            child: Container(
              margin: EdgeInsets.symmetric(vertical: 10, horizontal: 30),
              child: Text("水果 - $name"),
            ),
          ),
        );

        fruit.removeAt(index);
      },
      child: Icon(Icons.remove),
    )
  ],
)
```

![AnimatedList.gif](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/AnimatedList.gif)

## 34. Flexible

使用`Flexible`灵活调整行和列中的小部件大小。可以使用它来调整不同子小部件相对于其父小部件占用的空间。`Flexible`是第三小节`Expanded`的父类，`Expanded`中把`fit`属性默认设置为`FlexFit.tight`，实际上`Flexible`还有一个`FlexFit.loose`配置可选。

```
Column(
  mainAxisAlignment: MainAxisAlignment.spaceAround,
  children: <Widget>[
    Flexible(
      flex: 1,
      fit: FlexFit.tight,
      child: Container(
        width: double.infinity,
        height: 100,
        color: Colors.redAccent,
      ),
    ),
    Flexible(
      flex: 1,
      fit: FlexFit.tight,
      child: Container(
        height: 100,
        color: Colors.green,
      ),
    ),
    Flexible(
      flex: 1,
      fit: FlexFit.tight,
      child: Container(
        height: 100,
        color: Colors.blueAccent,
      ),
    ),
  ],
)
```

当`fit`属性设置为`FlexFit.tight`时，子元素自己的尺寸约束会失效，子元素将尽可能地填充空白区域，使得元素之间更为紧凑。

![FlexFit.tight](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/Flexible_tight.png)

而当`fit`属性设置为`FlexFit.loose`时，即使有足够的空间，子元素也会优先按照自身尺寸约束进行布局。

![FlexFit.loose](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/Flexible_loose.png)

## 35. MediaQuery

首先，`MediaQuery`继承自`InheritedWidget`（参考第37小节），所以`MediaQuery`提供了`of`静态方法自下而上寻找最近的`MediaQuery`组件，并返回其`MediaQueryData`属性。

```
const MediaQueryData({
    this.size = Size.zero,
    this.devicePixelRatio = 1.0,
    this.textScaleFactor = 1.0,
    this.platformBrightness = Brightness.light,
    this.padding = EdgeInsets.zero,
    this.viewInsets = EdgeInsets.zero,
    this.systemGestureInsets = EdgeInsets.zero,
    this.viewPadding = EdgeInsets.zero,
    this.alwaysUse24HourFormat = false,
    this.accessibleNavigation = false,
    this.invertColors = false,
    this.highContrast = false,
    this.disableAnimations = false,
    this.boldText = false,
    this.navigationMode = NavigationMode.traditional,
})
```

相比于`LayoutBuilder`（参考第22小节），`MediaQueryData`则包含了更多设备的信息，包括屏幕尺寸（size），设备方向（orientation），默认字体大小（textScaleFactor），系统`UI`遮盖的屏幕大小(padding)等等。

## 36. Spacer

`Spacer`通常配合`Row`和`Column`使用，自定义子组件之间空白区域大小，实际底层还是使用`Expanded`（参考第3小节）实现，所以使用`Spacer`时会自动填充剩余空间。

```
class Spacer extends StatelessWidget {
  const Spacer({Key? key, this.flex = 1})
    : assert(flex != null),
      assert(flex > 0),
      super(key: key);

  final int flex;

  @override
  Widget build(BuildContext context) {
    return Expanded(
      flex: flex,
      child: const SizedBox.shrink(),
    );
  }
}
```

![Spacer.png](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/Spacer.png)


## 37. InheritedWidget

`InheritedWidget`允许您的子组件访问父组件的数据。使用它可以省去在组件之间传递数据的麻烦。

`InheritedWidget`是抽象类，需要手动继承构造实例。

```
class SharedDataWidget extends InheritedWidget {
  final FavoriteFruit fruit;
  final Widget child;

  const SharedDataWidget({Key key, @required this.fruit, @required this.child})
      : super(child: child);

  @override
  bool updateShouldNotify(SharedDataWidget oldWidget) {
    return fruit != oldWidget.fruit;
  }

  static SharedDataWidget of(BuildContext context) {
    SharedDataWidget instance =
        context.dependOnInheritedWidgetOfExactType<SharedDataWidget>();
    assert(instance != null, 'No SharedDataWidget found in context');
    return instance;
  }
}
```

同时提供一个静态的`of`方法，方便其它组件访问 `InheritedWidget`。`BuildContext`提供了从子`Widget`向上寻找父`Widget/State/RenderObject`以及跨组件数据共享的方法集合。`InheritedWidget` 中调用 `dependOnInheritedWidgetOfExactType` 进行数据共享会进行注册绑定，以便当祖先元素发生变化时通知组件`rebuild`。而使用 `getElementForInheritedWidgetOfExactType` 则不会触发`rebuild`（涉及`didChangeDependencies` 生命周期的触发）。

需要特别注意的是：`InheritedWidget`通常是`immutable`（不可变的），内部的属性使用`final`标识，所以`InheritedWidget`在整个`app`的生命周期内都不会改变，如果想改变`InheritedWidget`，只有`rebuild`。即，当数据更新时，`SharedDataWidget`和其子组件都会发生重建（`rebuild`）。

像下面代码那样使用`InheritedWidget`是不行的哦，因为提供的`BuildContext`是父组件的，从父组件向上是找不到相应的`InheritedWidget`组件的（更详细的解释参考附录《深入理解BuildContext》）。

```
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: Text(widget.title),
    ),
    body: SharedDataWidget(
      fruit: _fruit,
      child: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
                "My favorite fruit —— ${SharedDataWidget.of(context).fruit.firstFavorite}"),
            Text(
                "My second favorite fruit —— ${SharedDataWidget.of(context).fruit.secondFavorite}"),
            OutlinedButton(
              onPressed: () {
                setState(() {
                  _fruit = FavoriteFruit("watermelon", "banana");
                });
              },
              child: Text("Update Data"),
            ),
          ],
        ),
      ),
    ),
  );
}
```

解决办法有两种：一种就是抽离出新的`Widget`，另一种就是使用`Builder`(详见第73小节)。

```
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(
      title: Text(widget.title),
    ),
    body: SharedDataWidget(
      fruit: _fruit,
      child: Builder( // 使用Builder拿到子组件的BuildContext
        builder: (BuildContext innerContext) {
          return Center(
            child: Column(
              mainAxisAlignment: MainAxisAlignment.center,
              children: <Widget>[
                Text(
                    "My favorite fruit —— ${SharedDataWidget.of(innerContext).fruit.firstFavorite}"),
                Text(
                    "My second favorite fruit —— ${SharedDataWidget.of(innerContext).fruit.secondFavorite}"),
                OutlinedButton(
                  onPressed: () {
                    setState(() {
                      _fruit = FavoriteFruit("watermelon", "banana");
                    });
                  },
                  child: Text("Update Data"),
                ),
              ],
            ),
          );
        },
      ),
    ),
  );
}
```


实际上，无论`SharedDataWidget`的`updateShouldNotify`方法返回`true`或`false`，子组件数据都会由于重建得到更新。`updateShouldNotify`的返回值只影响有状态组件（`StatefulWidget`）的`didChangeDependencies`生命周期，其返回`true`时会调用`didChangeDependencies`生命周期，而当其返回`false`时，则不会调用。

```
// Flutter团队对updateShouldNotify方法的说明
Whether the framework should notify widgets that inherit from this widget.

When this widget is rebuilt, sometimes we need to rebuild the widgets that
inherit from this widget but sometimes we do not. For example, if the data
held by this widget is the same as the data held by `oldWidget`, then we
do not need to rebuild the widgets that inherited the data held by
`oldWidget`.

The framework distinguishes these cases by calling this function with the
widget that previously occupied this location in the tree as an argument.
The given widget is guaranteed to have the same [runtimeType] as this
object.
```

运行如下代码，只有第一次点击`Update Data`按钮时会打印`WrapperWidget didChangeDependencies`，后续的点击都不会触发打印。

```
class _MyHomePageState extends State<MyHomePage> {
  FavoriteFruit _fruit;

  @override
  void initState() {
    super.initState();
    _fruit = FavoriteFruit("apple", "peach");
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: SharedDataWidget(
        fruit: _fruit,
        child: Column(
          children: [
            Expanded(
              flex: 1,
              child: WrapperWidget(),
            ),
            Expanded(
              flex: 1,
              child: Align(
                child: OutlinedButton(
                  onPressed: () {
                    setState(() {
                      _fruit = FavoriteFruit("watermelon", "banana");
                    });
                  },
                  child: Text("Update Data"),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
  }
}

class WrapperWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: DetailWidget(),
    );
  }
}

class DetailWidget extends StatefulWidget {
  @override
  State<StatefulWidget> createState() => _DetailState();
}

class _DetailState extends State<DetailWidget> {
  @override
  Widget build(BuildContext context) {
    String firstFavorite = SharedDataWidget.of(context).fruit.firstFavorite;
    String secondFavorite = SharedDataWidget.of(context).fruit.secondFavorite;

    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text("My favorite fruit —— $firstFavorite"),
        Text("My second favorite fruit —— $secondFavorite"),
      ],
    );
  }

  @override
  void didChangeDependencies() {
    super.didChangeDependencies();
    print("WrapperWidget didChangeDependencies");
  }
}
```

`InheritedWidget`组件提供了持有数据并跨组件分享的能力，我们可以结合`ValueNotifier`（详见第31小节）和`Notification`（详见第72小节）实现状态的管理。

最后，`InheritedWidget`更新粒度很粗，所有的观察者都会更新。上面的代码中我们在`FavoriteFruit`对象中保存了两个数据，有些场景可能某些组件只依赖其中一个数据，当希望细粒度得对数据进行更新时，使用`InheritedModel`（详见第16小节）是更高效的。


## 38. AnimatedIcon

使用`AnimatedIcon`可以将动画图标直接拖放到你的应用程序中。其中`progress`属性接收一个`Animation<double>`类型的值，进行动画控制；`icon`属性则接收一个`AnimatedIcons`类型的值描述`icon`的动画内容。

```
FloatingActionButton(
  onPressed: () {
    if (_progress.isCompleted) {
      _progress.reverse();
    } else {
      _progress.forward();
    }
  },
  child: AnimatedIcon(
    icon: AnimatedIcons.play_pause,
    progress: _progress,
  ),
)
```

![AnimatedIcon.gif](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/AnimatedIcon.gif)

`Flutter`中还有很多静态图标等待你的探索。

## 39. AspectRatio

有时候，我们并不关注具体的形状大小，而是要求约束元素的宽高比。比如，把照片变成16:9或者4:3。这个时候我们就需要使用`AspectRatio`了

`AspectRatio`会首先尝试布局约束所允许的最大宽度。通过给定的宽高比来确定小部件的高度，表示为宽度与高度的比率。

```
Container(
  width: 300,
  child: AspectRatio(
    aspectRatio: 16/9,
    child: Container(
      color: Colors.red,
    ),
  ),
)
```

** 不要在将`AspectRatio`作为`Expanded`的子元素，由于`Expanded`会强制子元素充满父元素，这会使得我们设置的`aspectRatio`属性失效。

## 40. LimitedBox

通常子组件会根据父组件的约束来设置本身的大小，但是像`ListView，Row，Column`作为父组件时，并不会向子组件施加尺寸约束。此时就可以使用`LimitedBox`了，`LimitedBox`会向子组件设置一个默认尺寸，当父组件存在尺寸约束时，`LimitedBox`不会生效；而当父组件不存在尺寸约束时，则会使用默认尺寸。

有些同学可能会把`LimitedBox`和`SizedBox`（详见第30小节）搞混淆，其实这两个组件没有太多可对比性，因为它们的用途是完全不一样。

## 附录

完整代码仓库地址: [>>>点我进入](https://github.com/Nirvana-cn/Flutter-widget-weekly)

深入理解BuildContext: [>>>点我进入](https://juejin.cn/post/6844903777565147150)

数据共享（InheritedWidget）: [>>>点我进入](https://book.flutterchina.club/chapter7/inherited_widget.html)

How to correctly use an Inherited Widget?: [>>>点我进入](https://stackoverflow.com/questions/49491860/flutter-how-to-correctly-use-an-inherited-widget)

源码分析系列之InheritedWidget: [>>>点我进入](https://juejin.cn/post/6919469982170480654)
