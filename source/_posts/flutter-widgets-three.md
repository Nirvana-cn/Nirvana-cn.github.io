
---
title: Flutter Widget — Part 3
date: 2021-07-03 16:28:34
tags:
- flutter
categories:
- Flutter指南
---

Flutter中有数目繁多的widget，多了解一个widget就少造一个轮子😋 😋 😋 。

<!--more-->

## 21. FittedBox

FittedBox有助于管理空间限制和盒子的布局。FittedBox的作用有点像图片容器，我也不知道图片多大，但是不管你提供什么图片，我都能放得下，还对能图片进行缩放、裁剪，使图片适应容器。

这就意味着：某些情况下，我们可以不再关心子元素的大小，借助FittedBox使得子元素在父元素中自适应，实现优雅的组件化。

```
const FittedBox({
    Key? key,
    // 填充效果
    this.fit = BoxFit.contain,
    // 对齐方式
    this.alignment = Alignment.center,
    this.clipBehavior = Clip.none,
    Widget? child,
  })
```

下面的例子中，借助FittedBox，尺寸大于200的Container都能正确显示，而不会溢出。

```
Container(
  width: 200,
  height: 200,
  color: Colors.lightBlueAccent,
  child: FittedBox(
    fit: BoxFit.contain,
    child: Container(
      width: 400,
      height: 300,
      color: Colors.green,
    ),
  ),
)

Container(
  width: 200,
  height: 200,
  color: Colors.lightBlueAccent,
  child: FittedBox(
    fit: BoxFit.fitHeight,
    child: Container(
      width: 300,
      height: 400,
      color: Colors.greenAccent,
    ),
  ),
)
```

![FittedBox](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/FittedBox.png)

## 22. LayoutBuilder

`LayoutBuilder`可以获取布局大小，允许我们根据布局大小调整我们的页面元素。

```
LayoutBuilder(
    builder: (context, constraints) {
      if (constraints.maxWidth >= 500) {
        //...
      }
    },
)
```

builder参数接收一个类型为LayoutWidgetBuilder的函数。

个人感觉比较适合横竖屏支持的移动端和界面可以任意伸缩的桌面端。


![LayoutBuilder](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/LayoutBuilder.gif)

但是这个变化还是比较突兀的，添加一个过渡动画就完美了。

## 23. AbsorbPointer

假如我们不想让某个子树响应PointerEvent的话，我们可以使用IgnorePointer（详见第77小节）和AbsorbPointer，这两个组件都能阻止子树接收指针事件，不同之处在于AbsorbPointer本身会参与命中测试，而IgnorePointer本身不会参与，这就意味着AbsorbPointer本身是可以接收指针事件的(但其子树不行)，而IgnorePointer不可以。

```
Listener(
  child: AbsorbPointer(
    child: Listener(
      child: Container(
        color: Colors.red,
        width: 200.0,
        height: 100.0,
      ),
      onPointerDown: (event)=>print("in"),
    ),
  ),
  onPointerDown: (event)=>print("up"),
)
```

点击Container时，由于它在AbsorbPointer的子树上，所以不会响应指针事件，所以日志不会输出"in"，但AbsorbPointer本身是可以接收指针事件的，所以会输出"up"。如果将AbsorbPointer换成IgnorePointer，那么两个都不会输出。

## 24. Transform

Transform可以在其子组件绘制时对其应用一些矩阵变换来实现一些特效。Matrix4是一个4D矩阵，通过它我们可以实现各种矩阵操作。关于矩阵变换的相关内容属于线性代数范畴，短短几句话是讲不清楚的，所以我们把焦点放在Flutter中一些常见的变换效果上。

下面是一个3D视角的例子：

```
Transform(
  transform: Matrix4.identity()
    ..setEntry(3, 2, 0.01)
    ..rotateX(0.6),
  alignment: FractionalOffset.center,
  child: getMyWidget("3D视角"),
)

Widget getMyWidget(String string) {
  return Container(
    alignment: Alignment.center,
    width: 80,
    height: 80,
    color: Colors.blue,
    child: Text(
      string,
      style: TextStyle(
        color: Colors.white,
      ),
    ),
  );
}    
```

为了更方便得使用Transfrom，Flutter也提供了平移、旋转、缩放等常用变换的语法糖。

### Transform.translate

Transform.translate接收一个offset参数，可以在绘制时沿x、y轴对子组件平移指定的距离，默认原点为左上角。

### Transform.rotate

Transform.rotate可以对子组件进行旋转变换。

> 注意：要使用math.pi需先进行如下导包。
> 
> import 'dart:math' as math;

Tips: RotatedBox和Transform.rotate功能相似，它们都可以对子组件进行旋转变换，但是有一点不同：RotatedBox的变换是在layout阶段，会影响在子组件的位置和大小。

### Transform.scale

Transform.scale可以对子组件进行缩小或放大。

具体示例如下：

```
Column(
  mainAxisAlignment: MainAxisAlignment.spaceEvenly,
  children: <Widget>[
    getMyWidget("No Transfrom"),
    Transform.translate(
      offset: Offset(50, 0),
      child: getMyWidget("向右平移50"),
    ),
    Transform.rotate(
      angle: math.pi / 4,
      child: getMyWidget("旋转45度"),
    ),
    Transform.scale(
      scale: 1.5,
      child: getMyWidget("放大1.5倍"),
    ),
  ],
)
```

![Transform](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/transform.png)

最后，需要注意的是：Transform的变换是应用在绘制阶段，而并不是应用在布局(layout)阶段，所以无论对子组件应用何种变化，其占用空间的大小和在屏幕上的位置都是固定不变的，因为这些是在布局阶段就确定的。因此Transform无需重新布局和构建等过程，所以性能很好。

** 上图中，我们使用了MainAxisAlignment.spaceEvenly来平均分配空白空间，但是元素之间的间距并不一致，也从侧面印证Transform变换的布局特性。


## 25. BackdropFilter

BackdropFilter用来对组件进行调整（tweak），比如旋转、倾斜、模糊。如果只是旋转、倾斜的话，使用第24小节的Transform也可以实现。BackdropFilter和Transform的区别在于：BackdropFilter是对于渲染在其下方的元素生效，这个下方指的是z轴方向上，而不是指子元素，比较好理解的一个例子就是Stack，靠前的元素总是渲染在靠后的元素下方。

比如，我们希望实现下面图片中倾斜的效果，我们只需要使用一次BackdropFilter，而不需要给每个元素都使用Transform包裹。

```
Stack(
  children: [
    Positioned(
      left: 0,
      child: Container(
        width: 100,
        height: 200,
        color: Colors.lightBlue,
      ),
    ),
    Positioned(
      right: 0,
      child: Container(
        width: 100,
        height: 200,
        color: Colors.lightGreen,
      ),
    ),
    BackdropFilter(
      filter: ImageFilter.matrix(Matrix4.skewX(0.1).storage),
      //高斯模糊
      //filter: ImageFilter.blur(sigmaX: 2, sigmaY: 2),
      child: Container(
        color: Colors.black.withOpacity(0.0),
      ),
    ),
  ],
)
```

![BackdropFilter_1](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/BackdropFilter_1.png)

BackdropFilter用来实现高斯模糊的场景比较多，对于平移、旋转、倾斜使用Transform更为方便。同时，BackdropFilter的child是不会受到自己影响的。

```
Stack(
  children: [
    Positioned.fill(
      child: Image.asset(
        "images/test.jpeg",
        fit: BoxFit.fill,
      ),
    ),
    Align(
      alignment: Alignment.center,
      child: ClipRRect(
        borderRadius: BorderRadius.all(Radius.circular(20)),
        child: BackdropFilter(
          filter: ImageFilter.blur(sigmaX: 2, sigmaY: 2),
          child: Container(
            width: 300,
            height: 200,
            alignment: Alignment.center,
            child: Text("Hello World!", style: TextStyle(color: Colors.white),),
          ),
        ),
      ),
    ),
  ],
)
```

![BackdropFilter_2](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/BackdropFilter_2.png)

但是，有一点需要注意的是BackdropFilter会寻找祖先元素中的clip，否则BackdropFilter的效果会充满整个屏幕。这一点官方文档也有说明。

> The filter will be applied to all the area within its parent or ancestor widget's clip. If there's no clip, the filter will be applied to the full screen.
> 
> If the BackdropFilter needs to be applied to an area that exactly matches its child, wraps the BackdropFilter with a clip widget that clips exactly to that child.

如果把上面代码中的ClipRRect换成Container，那么整个屏幕都会充满模糊效果。

![BackdropFilter_3](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/BackdropFilter_3.png)

最后，Flutter还提供了ImageFilter组件用来调整图片（详见第93小节），而BackdropFilter应用范围则更广，BackdropFilter可以调整一切（Flutter中一切皆组件），而不仅仅是图片。当然与ImageFilter相比，BackdropFilter的性能要差一些，毕竟ImageFilter是专业的。所以当我们想对图片进行调整时，使用ImageFilter是更优的。

## 26. Align

Align 组件可以调整子组件的位置，并且可以根据子组件的宽高来确定自身的的宽高，定义如下：

```
Align({
  Key key,
  AlignmentGeometry alignment: Alignment.center,
  double widthFactor,
  double heightFactor,
  Widget child
})
```
- alignment : 需要一个AlignmentGeometry类型的值，表示子组件在父组件中的起始位置。AlignmentGeometry 是一个抽象类，它有两个常用的子类：Alignment和 FractionalOffset。

- widthFactor和heightFactor是用于确定Align 组件本身宽高的属性；它们是两个缩放因子，会分别乘以子元素的宽、高，最终的结果就是Align 组件的宽高。如果值为null，则组件的宽高将会占用尽可能多的空间。

### Alignment

Alignment继承自AlignmentGeometry，表示矩形内的一个点，他有两个属性x、y，分别表示在水平和垂直方向的偏移。

Widget会以矩形的中心点作为坐标原点，即Alignment(0.0, 0.0) 。x、y的值从-1到1分别代表矩形左边到右边的距离和顶部到底边的距离，因此2个水平（或垂直）单位则等于矩形的宽（或高），如Alignment(-1.0, -1.0) 代表矩形的左侧顶点，而Alignment(1.0, 1.0)代表右侧底部终点，而Alignment(1.0, -1.0) 则正是右侧顶点，即Alignment.topRight。为了使用方便，矩形的原点、四个顶点，以及四条边的终点在Alignment类中都已经定义为了静态常量。

Alignment可以通过其坐标转换公式将其坐标转为子元素的具体偏移坐标：

```
(Alignment.x*childWidth/2+childWidth/2, Alignment.y*childHeight/2+childHeight/2)
```

其中childWidth为子元素的宽度，childHeight为子元素高度。

### FractionalOffset

FractionalOffset 继承自 Alignment，它和 Alignment唯一的区别就是坐标原点不同！FractionalOffset 的坐标原点为矩形的左侧顶点，这和布局系统的一致，所以理解起来会比较容易。FractionalOffset的坐标转换公式为：

```
实际偏移 = (FractionalOffse.x * childWidth, FractionalOffse.y * childHeight)
```

```
Container(
  width: 500,
  height: 500,
  color: Colors.grey,
  child: Stack(
    children: [
      Align(
        alignment: Alignment.center,
        child: Container(
          width: 100,
          height: 100,
          color: Colors.lightBlueAccent,
        ),
      ),
      Align(
        alignment: Alignment(0.2, 0.2),
        child: Container(
          width: 100,
          height: 100,
          color: Colors.greenAccent,
        ),
      ),
      Align(
        alignment: FractionalOffset(0.2, 0.2),
        child: Container(
          width: 100,
          height: 100,
          color: Colors.lightGreenAccent,
        ),
      ),
      Align(
        alignment: FractionalOffset(0.2, -0.2),
        child: Container(
          width: 100,
          height: 100,
          color: Colors.orangeAccent,
        ),
      ),
    ],
  ),
)
```

代码中，我们分别使用Alignment和FractionalOffset为绿色方块和淡绿色方块施加了水平以及垂直方向上20%的偏移。根据最后两个元素的位置，我们可以很明确得理解二者定位坐标原点的区别。

同样，我们也可以施加负的偏移量，使得元素脱离父元素的显示区域，如橘色方块所示。

![Align](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/Align.png)

### Center组件

Center组件是很常用的用来居中子元素的组件，通过查找SDK源码，我们看到Center组件定义如下：

```
class Center extends Align {
  const Center({ Key key, double widthFactor, double heightFactor, Widget child })
    : super(key: key, widthFactor: widthFactor, heightFactor: heightFactor, child: child);
}
```

可以看到Center继承自Align，它比Align只少了一个alignment 参数；由于Align的构造函数中alignment 值为Alignment.center，所以，我们可以认为Center组件其实是对齐方式确定（Alignment.center）了的Align。

熟悉Web开发的同学可能会发现Align组件的特性和Web开发中相对定位（position: relative）非常像，是的！在大多数时候，我们可以直接使用Align组件来实现Web中相对定位的效果。

最后，在第3节中我们说过Expanded会造成子元素的布局约束失效，而Align则可以通过自身占满整个父元素，从而继续让它的子元素保持自己的属性。

## 27. Positioned

在Flutter中使用Stack进行层叠布局非常方便，Stack允许子组件堆叠，而Positioned用于根据Stack的四个角来确定子组件的位置。

```
const Positioned({
  Key key,
  this.left,
  this.top,
  this.right,
  this.bottom,
  this.width,
  this.height,
  @required Widget child,
})
```

left、top 、right、 bottom分别代表距离Stack容器左、上、右、底四边的距离。width和height用于指定需要定位元素的宽度和高度。注意，Positioned的width、height 和其它地方的意义稍微有点区别，此处用于配合left、top 、right、 bottom来定位组件，举个例子，在水平方向时，你只能指定left、right、width三个属性中的两个，如指定left和width后，right会自动算出(left+width)，如果同时指定三个属性则会报错，垂直方向同理。

```
Stack(
  children: <Widget>[
    Positioned(
      left: 20,
      width: 350,
      child: Container(
        width: 300,
        height: 50,
        color: Colors.amber,
      ),
    ),
  ],
)
```

在此种情况下，Container的宽度会被拉伸到350，即Positioned的width属性会覆盖子元素的width。

```
Stack(
  children: <Widget>[
    Positioned(
      left: 20,
      right: 20,
      child: Container(
        width: 300,
        height: 50,
        color: Colors.amber,
      ),
    ),
  ],
)
```

而在此种情况下，虽然我们并没有显示指定width，但是由于设置了水平方向上的left和right，Flutter会自动计算width值，并应用到子元素上。

如果希望子元素自动填充满Stack空间，可以使用Positioned.fill，此时Positioned的宽高会和Stack保持一致。

** tips: left、top 、right、 bottom以及width和height都是可以指定负数值的。

## 28. AnimatedBuilder

AnimatedBuilder是Flutter动画的一个语法糖，如果对动画不了解，推荐先看一下附录中的《Flutter实战—-动画》，熟悉各个类之间的关系，再继续学习AnimatedBuilder。

Flutter动画的原理就是生成一系列的值，然后不停地调用setState，使当前帧被标记为脏(dirty)，导致widget的build()方法再次被调用，最后渲染新的帧。

通过setState() 来更新UI这一步其实是通用的和繁琐的。而AnimatedBuilder可以将渲染逻辑分离出来，具有的优势如下：

1. 不用显式的去添加帧监听器，然后再调用setState() 了。

2. 动画构建的范围缩小了，如果没有builder，setState()将会在父组件上下文中调用，这将会导致父组件的build方法重新调用；而有了builder之后，只会导致动画widget自身的build重新调用，避免不必要的rebuild。

3. 通过AnimatedBuilder可以封装常见的过渡效果来复用动画。


接下来我们使用AnimatedBuilder封装一个简单的旋转动画，代码和效果如下：


```
class CustomRotateAnimation extends StatelessWidget {
  final AnimationController controller;
  final Widget child;

  CustomRotateAnimation({
    @required this.controller,
    @required this.child,
  });

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: controller,
      child: child,
      builder: (context, child) {
        return Transform.rotate(
          angle: controller.value,
          child: child,
        );
      },
    );
  }
}


class _MyHomePageState extends State<MyHomePage>
    with SingleTickerProviderStateMixin {
  AnimationController controller;

  @override
  void initState() {
    super.initState();
    controller = AnimationController(
      vsync: this,
      lowerBound: 0,
      upperBound: 2 * pi,
      duration: Duration(seconds: 3),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: CustomRotateAnimation(
          controller: controller,
          child: FlutterLogo(size: 200),
        ),
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () {
          controller.forward();
        },
      ),
    );
  }

  @override
  void dispose() {
    super.dispose();
    controller.dispose();
  }
}
```

![AnimatedBuilder](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/AnimatedBuilder.gif)

## 29. Dismissible

Dismissible小部件可通过向左或向右滑动来清除列表项。对于多方向滑动，它支持两种背景，如果您的UI需要用户垂直滑动，则有一个方向属性！

```
const Dismissible({
    required Key key,
    required this.child,
    // 默认背景，多方向时为左滑背景
    this.background,
    // 多方向时右滑背景
    this.secondaryBackground,
    this.confirmDismiss,
    this.onResize,
    // 清除回调事件
    this.onDismissed,
    // 滑动方向
    this.direction = DismissDirection.horizontal,
    this.resizeDuration = const Duration(milliseconds: 300),
    this.dismissThresholds = const <DismissDirection, double>{},
    this.movementDuration = const Duration(milliseconds: 200),
    this.crossAxisEndOffset = 0.0,
    this.dragStartBehavior = DragStartBehavior.start,
    this.behavior = HitTestBehavior.opaque,
  })
```

![Dismissible](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/Dismissible.gif)

## 30. SizeBox

SizedBox会强制设置它的孩子的宽度或者高度为指定值。如下代码，即使给Container设置了宽高为200\*200，由于SizedBox的存在，Container的大小依然只有100\*100。

```
SizedBox(
  width: 100,
  height: 100,
  child: Container(
    width: 200,
    height: 200,
    color: Colors.red,
  ),
)
```

实际上SizedBox只是ConstrainedBox（详见第49小节）的一个定制，上面代码等价于：

```
ConstrainedBox(
  constraints: BoxConstraints.tightFor(width: 100,height: 100),
  child: Container(
    width: 200,
    height: 200,
    color: Colors.red,
  ),
)
```

而BoxConstraints.tightFor(width: 80.0,height: 80.0)等价于：

```
BoxConstraints(minHeight: 80.0,maxHeight: 80.0,minWidth: 80.0,maxWidth: 80.0)

```

所以SizedBox可以看作是ConstrainedBox的语法糖。

当你希望子元素充满整个容器时，你可以使用SizedBox.expand。子元素会自动填充满 可用空间。

```
SizedBox.expand(
  child: Container(
    width: 200,
    height: 200,
    color: Colors.red,
    child: Text("When width is infinity in SizedBox"),
  ),
)

// 等价于

SizedBox(
  width: double.infinity,
  height: double.infinity,
  child: Container(
    width: 200,
    height: 200,
    color: Colors.red,
    child: Text("When width is infinity in SizedBox"),
  ),
)
```

![SizedBox](https://raw.githubusercontent.com/Nirvana-cn/Photograph-deposit/master/flutter-week/sized-box.png)


## 附录

Flutter实战—-动画9.1：[>>>点我进入](https://book.flutterchina.club/chapter9/intro.html)

Flutter实战—-动画9.2：[>>>点我进入](https://book.flutterchina.club/chapter9/animation_structure.html)