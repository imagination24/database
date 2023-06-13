# Flutter 如何用RenderObject自定义带动画的Widget
今天提炼一些关于RenderObject绘制自定义Widget的知识，需要注意的是RenderObject也是用CustomPainter来绘制，要先了解CustomPainter相关Api。

## RenderObjectWidget
众所周知，在Flutter里，一切都是Widget，拢共有大小三百多个Widget。Flutter有三棵树我们需要学习，其中一颗就是Widget树。在Flutter框架中，有一个叫Widget的超级抽象类，继承Widegt有四个子类，分别是大家熟悉的*StatelessWidget*和*StatefulWidget*,以及不是那么熟悉的*RenderObjectWidget*和*ProxyWidget*,而今天要用到的是*RenderObjectWidget*。（Flutter 3.7.3）

RenderObjectWidget实时上只有三个子类，分别是*LeafRenderObjectWidget*，*SingleChildRenderObjectWidget*，*MultiChildRenderObjectWidget*。

*LeafRenderObjectWidget*:LeafRenderObjectWidget顾名思义，叶子RenderObjectWidget,意味着这个Widget不能有child,一般是最底层的Widget，其中包括*ErrorWidget*/*RawImage*/*Texture*。

*SingleChildRenderObjectWidget*:SingleChildRenderObjectWidget只能有一个child的Widget,其中涉及到了大部分的容器类Widget。

*MultiChildRenderObjectWidget*：MultiChildRenderObjectWidget是可以拥有多个子组件的Widget，其中包括大部分布局类Widget,例如*Flex*/*Flow*/*Wrap*。

到此为之，你应该对RenderObjectWidget大致的脉络有了认识，然后再来看看RenderObject有哪些抽象方法。

```
abstract class RenderObjectWidget extends Widget {

  const RenderObjectWidget({ super.key });
  

  @override
  @factory
  RenderObjectElement createElement();


  @protected
  @factory
  RenderObject createRenderObject(BuildContext context);


  @protected
  void updateRenderObject(BuildContext context, covariant RenderObject renderObject) { }


  @protected
  void didUnmountRenderObject(covariant RenderObject renderObject) { }
  
}
```
可以看到有四个抽象方法等待被重写和实现。今天我们研究一下，RenderObjectWidget三个子类中的*LeafRenderObjectWidget*

先看看LeafRenderObjectWidget又有哪些抽象方法等待被实现。

```
abstract class LeafRenderObjectWidget extends RenderObjectWidget {
  
  const LeafRenderObjectWidget({ super.key });

  @override
  LeafRenderObjectElement createElement() => LeafRenderObjectElement(this);
}
```

这个类很简单，实现了*createElement*，那要使用LeafRenderObjectWidget绘制自定义Widget，就必须实现*createRenderObject*了，这个函数要求返回一个RenderObject...这是一个关于ElementTree的知识，不会呀...。
好吧，查了资料，跟着我！直接上手走一遍。

## 实例一号（自定义大小CheckBox）
众所周知，Flutter的CheckBox不能自定义大小，今天我们解决这个痛点。
首先创建一个类继承于*LeafRenderObjectWidget*,定义一些关于CheckBox的成员变量。

```
class SkyCastleCheckBox extends LeafRenderObjectWidget {
  final double strokeWidth;
  final Color strokeColor;
  final Color fillColor;
  final bool value;
  final double radius;
  final ValueChanged<bool>? onChanged;

  const SkyCastleCheckBox(
      {super.key,
      required this.strokeWidth,
      required this.strokeColor,
      required this.fillColor,
      required this.value,
      required this.radius,
      this.onChanged});

  @override
  RenderObject createRenderObject(BuildContext context) {
    // TODO: implement createRenderObject
    throw UnimplementedError();
  }
}
```
所以现在创建一个类继承于RenderBox
```
class RenderSkyCastleCheckBox extends RenderBox{
  @override
  void performLayout() {
    // TODO: implement performLayout
    super.performLayout();
  }
  
  @override
  void paint(PaintingContext context, Offset offset) {
    // TODO: implement paint
    super.paint(context, offset);
  }
  
  @override
  bool hitTestSelf(Offset position) {
    // TODO: implement hitTestSelf
    return super.hitTestSelf(position);
  }
  
  @override
  void handleEvent(PointerEvent event, covariant BoxHitTestEntry entry) {
    // TODO: implement handleEvent
    super.handleEvent(event, entry);
  }
  
  @override
  // TODO: implement isRepaintBoundary
  bool get isRepaintBoundary => true;
}
```
代码中的这几个类是我们必须实现的。

你会发现有个Paint很熟悉，是的，这和Canvas中的Paint是一样的。

ok! 定义和SkyCastleCheckBox一样的成员变量。并把RenderSkyCastleCheckBox应用于SkyCastleCheckBox

```
class SkyCastleCheckBox extends LeafRenderObjectWidget {
  final double strokeWidth;
  final Color strokeColor;
  final Color fillColor;
  final bool value;
  final double radius;
  final ValueChanged<bool>? onChanged;

  const SkyCastleCheckBox({super.key,
    required this.strokeWidth,
    required this.strokeColor,
    required this.fillColor,
    required this.value,
    required this.radius,
    this.onChanged});

  @override
  RenderObject createRenderObject(BuildContext context) =>
      RenderSkyCastleCheckBox(
          strokeWidth: strokeWidth,
          strokeColor: strokeColor,
          fillColor: fillColor,
          value: value,
          radius: radius);

}

class RenderSkyCastleCheckBox extends RenderBox {
  final double strokeWidth;
  final Color strokeColor;
  final Color fillColor;
  final bool value;
  final double radius;
  final ValueChanged<bool>? onChanged;

  RenderSkyCastleCheckBox({
    required this.strokeWidth,
    required this.strokeColor,
    required this.fillColor,
    required this.value,
    required this.radius,
    this.onChanged});

  @override
  void performLayout() {
    // TODO: implement performLayout
    super.performLayout();
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    // TODO: implement paint
    super.paint(context, offset);
  }

  @override
  bool hitTestSelf(Offset position) {
    // TODO: implement hitTestSelf
    return super.hitTestSelf(position);
  }

  @override
  void handleEvent(PointerEvent event, covariant BoxHitTestEntry entry) {
    // TODO: implement handleEvent
    super.handleEvent(event, entry);
  }

  @override
  // TODO: implement isRepaintBoundary
  bool get isRepaintBoundary => true;
}
```
接下来一一实现

*performLayout*：测绘布局

*paint*：实现Widget

*hitTestSelf*: 命中测试

*handleEvent*：处理命中事件

