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
好吧，查了资料，跟着我！

## 实例一号（自定义大小CheckBox）
[参考项目-Flutter实战2-10.6-customcheckbox](https://book.flutterchina.club/chapter10/custom_checkbox.html#_10-6-1-customcheckbox)

众所周知，Flutter的CheckBox不能自定义大小，今天我们解决这个痛点。
我们做一个可以自定义画笔大小，画笔颜色，填充颜色以及边角曲率的CheckBox;

像这样

![check](https://i.imgur.com/GmDNqdi.png)

首先创建一个类继承于*LeafRenderObjectWidget*,定义一些关于CheckBox的成员变量。


```
class SkyCastleCheckBox extends LeafRenderObjectWidget {
  //笔画宽度
  final double strokeWidth;
  //笔画颜色
  final Color strokeColor;
  //填充颜色
  final Color fillColor;
  //值
  final bool value;
  //圆角半径
  final double radius;
  //动画值变化
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
  //笔画宽度
  final double strokeWidth;
  //笔画颜色
  final Color strokeColor;
  //填充颜色
  final Color fillColor;
  //值
  final bool value;
  //圆角半径
  final double radius;
  //回调
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
  //笔画宽度
  final double strokeWidth;
  //笔画颜色
  final Color strokeColor;
  //填充颜色
  final Color fillColor;
  //值
  final bool value;
  //圆角半径
  final double radius;
  //回调
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

基本时是背景画一个DRRect,外圈不变，里圈跟据动画进程变化。Check则固定前两个点，点三个点根据动画进程做插值。

```
  
  class RenderSkyCastleCheckBox extends RenderBox {
  ///笔画宽度
  double strokeWidth;
  ///笔画颜色
  Color strokeColor;
  ///填充颜色
  Color fillColor;
  ///值
  bool value;
  ///圆角半径
  double radius;
  ///回调
  ValueChanged<bool>? onChanged;
  ///背景绘制时间占比
  double bgAnimationInterval = 0.4;
  ///动画进程
  double _progress = 0;
  ///最新时间戳
  int? _lastTimeStamp;
  ///动画时长
  Duration get duration => const Duration(milliseconds: 200);
  ///动画状态
  AnimationStatus _animationStatus = AnimationStatus.completed;
  ///触控点ID
  int pointerId = -1;

  set animationStatus(AnimationStatus status) {
    if (_animationStatus != status) {
      markNeedsPaint();
    }
    _animationStatus = status;
  }

  double get progress => _progress;

  set progress(double progress) {
    ///clamp->Progress 安全值
    _progress = progress.clamp(0, 1);
  }

  RenderSkyCastleCheckBox({
    required this.strokeWidth,
    required this.strokeColor,
    required this.fillColor,
    required this.value,
    required this.radius,
    this.onChanged});

  @override
  void performLayout() {
    ///测绘size，当有约束刚刚好满足时，使用这个约束。如果没有则指定为Size(25,25)。
    size = constraints.constrain(
        constraints.isTight ? Size.infinite : const Size(25, 25));
  }

  @override
  void paint(PaintingContext context, Offset offset) {
      Rect rect = offset & size;
      ///绘制背景
      _drawBackground(context, rect);
      ///绘制Check
      _drawCheckMark(context, rect);
      ///执行动画测绘
      _scheduleAnimation();
  }

  void _drawBackground(PaintingContext context, Rect rect) {
    ///创建画笔
    var paint = Paint()
      ..isAntiAlias = true
      ..style = PaintingStyle.fill
      ..strokeWidth = strokeWidth
      ..color = fillColor;
    ///外部矩形
    ///外部矩形是一个确定的矩形
    final outer = RRect.fromRectXY(rect, radius, radius);
    ///Rect? lerp(   Rect? a,   Rect? b,   double t, )
    ///通过两个矩形和一个0~1之间的值t,插值返回一个两个矩形之间的矩形
    ///Rect deflate(double delta)
    ///返回一个由原矩形向内延申一个delta距离的新矩形
    ///min(progress, bgAnimationInterval) / bgAnimationInterval
    ///计算当前时间进程t，bgAnimationInterval是固定的，意味着背景绘制时间占比
    ///当动画进程progress小于背景绘制时间占比，绘制背景
    var rectProgress = Rect.lerp(
        rect.deflate(strokeWidth),
        Rect.fromCenter(center: rect.center, width: 0, height: 0),
        // 背景动画的执行时长是前 40% 的时间
        min(progress, bgAnimationInterval) / bgAnimationInterval);
    final inner = RRect.fromRectXY(rectProgress!, radius, radius);
    ///绘制背景
    context.canvas.drawDRRect(outer, inner, paint);
  }

  ///绘制Check
  void _drawCheckMark(PaintingContext context, Rect rect) {
    ///当动画进程大于背景绘制占比进程是才开始绘制Check
    if (progress > bgAnimationInterval) {
      ///Check起点
      final firstOffset =
      Offset(rect.left + rect.width / 7, rect.top + rect.height / 2);
      ///第二个点
      final secondOffset =
      Offset(rect.left + rect.width / 2.5, rect.bottom - rect.height / 4);
      ///最终点
      final lastOffset =
      Offset(rect.right - rect.width / 6, rect.top + rect.height / 4);
      ///插值终点
      final progressingLastOffset = Offset.lerp(secondOffset, lastOffset,
          (progress - bgAnimationInterval) / (1 - bgAnimationInterval))!;
      ///创建路径
      final path = Path()
        ..moveTo(firstOffset.dx, firstOffset.dy)
        ..lineTo(secondOffset.dx, secondOffset.dy)
        ..lineTo(progressingLastOffset.dx, progressingLastOffset.dy);
      ///创建画笔
      final paint = Paint()
        ..isAntiAlias = true
        ..style = PaintingStyle.stroke
        ..color = strokeColor
        ..strokeWidth = strokeWidth;
      ///绘制check
      context.canvas.drawPath(path, paint..style = PaintingStyle.stroke);
    }
  }

  ///执行动画测绘
  void _scheduleAnimation() {
    ///仅在动画状态为未完成时运行
    if (_animationStatus != AnimationStatus.completed) {
      ///这个function只有在前一帧绘制完成后才有回调
      ///so现在是在安排下一帧的绘制任务并计算progress
      SchedulerBinding.instance.addPostFrameCallback((timeStamp) {

        ///!=null意味着动画正在进行中
        if (_lastTimeStamp != null) {

          ///计算时间间隔占比
          double delta = (timeStamp.inMilliseconds - _lastTimeStamp!) /
              duration.inMilliseconds;

          ///在特定情况下，可能在一帧中连续的往frameCallback中添加了多次，导致两次回调时间间隔为0，
          ///这种情况下应该继续请求重绘。
          if (delta == 0) {
            markNeedsPaint();
            return;
          }

          ///如果动画状态为反向，则反转时间间隔占比
          if (_animationStatus == AnimationStatus.reverse) {
            delta = -delta;
          }
          ///计算动画进程
          _progress = _progress + delta;

          ///当_progress = 1 或者 0 时，标记动画状态为完成
          ///赋予_progress为安全值
          if (_progress >= 1 || _progress <= 0) {
            _animationStatus = AnimationStatus.completed;
            _progress = _progress.clamp(0, 1);
          }
        }
        markNeedsPaint();
        _lastTimeStamp = timeStamp.inMilliseconds;
      });
    } else {
      _lastTimeStamp = null;
    }
  }

  @override
  bool hitTestSelf(Offset position) => true;

  @override
  void handleEvent(PointerEvent event, covariant BoxHitTestEntry entry) {
    super.handleEvent(event, entry);
    ///当指针事件为向下时，记录指针的标识符
    if (event.down) {
      pointerId = event.pointer;
    } else if (pointerId == event.pointer&&size.contains(event.localPosition)) {
      ///如果指针标识符相等且位置在size内，则更新回调
        onChanged?.call(!value);
    }
  }

  @override
  bool get isRepaintBoundary => true;
}
```



