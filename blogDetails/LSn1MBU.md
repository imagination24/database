# Flutter Canvas 的使用
*使用Canvas自定义Widget*

总所周知，FlutterWidgets种类繁多，各种wigets达到三百多个，一般情况下用Flutter定义好的widget就可以实现我们想要的效果。但如果需求特殊，Widgets库里找不到合适组件可以使用，我们就可以用Canvas来画自己的组件了。

## Paint 如何使用？
我们先学习一下Canvas如何使用
首先我们定义一个自己的Painter
```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    // TODO: implement paint
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;

}
```
然后建立一个Containter和CustomPaint来承载我们建立的Painter，先把Painter注释掉。
```
 ···
 Container(
   width: 300,
   height: 300,
   decoration: const BoxDecoration(color: Colors.amber),
      child: CustomPaint(
        // painter: SkyCastlePainter(),
      ),
  ),
 ···
```
我们得到了一个300x300的黄色小方块

![yellowContainer](https://i.imgur.com/bSOkO2o.png)

要开始画画我们需要先实现并重写paint()

以画一条线为例子。
```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 5;
    ///画一条线
    canvas.drawLine(Offset(0, size.height/2), Offset(size.width, size.height/2), paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```
![line](https://i.imgur.com/Su7aU8T.png)

可以看到，如果我们要在canvas上画东西我们需要新建一个画笔，画笔有很多属性可以给我们定义，接下来我们就先讲一下画笔的属性。
### 画笔的属性
*color*

画笔的颜色

*strokeWidth*

画笔的宽度

*style(PaintingStyle)*

这个属性有两个选择 PaintingStyle.stroke 和 PaintingStyle.fill 

让我们看看有什么区别

画线看不出来所以我们画一个矩形
```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 5..style = PaintingStyle.stroke;
    Rect rect = Offset(size.width/3, size.width/3)&const Size(100, 100);
    ///画一个矩形
    canvas.drawRect(rect, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```
PaintingStyle.stroke

![stroke](https://i.imgur.com/U41oX5t.png)

PaintingStyle.fill

![fill](https://i.imgur.com/kZbaXxQ.png)

这样就很明晰了，PaintingStyle.stroke 是绘制形状的边缘，PaintingStyle.fill 是填充整个形状

*isAntiAlias*

//是否抗锯齿

*shader*

覆盖在形状上的遮罩
```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    Rect rect = Offset(size.width/3, size.width/3)&const Size(100, 100);
    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 5
      ..style = PaintingStyle.fill
      ..isAntiAlias = true;
    paint.shader = const LinearGradient(colors: [
      Colors.indigoAccent,
      Colors.lightGreen
    ]).createShader(rect);
    ///画一个矩形
    canvas.drawRect(rect, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```
![shader](https://i.imgur.com/bakklq5.png)


*strokeCap*

线条末端的样式，有三个样式。StrokeCap.butt/StrokeCap.round/StrokeCap.square。

这个属性只有在style = PaintingStyle.stroke 时有效。

我们画路径来辨识这三种样式。
```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    Path path = Path()..moveTo(30, size.height/2);
    path.lineTo(size.width/2, size.height/2);
    path.lineTo(30, size.height/3);

    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 15
      ..style = PaintingStyle.stroke
      ..isAntiAlias = true
      ..strokeCap = StrokeCap.butt;
    ///画一条路径
    canvas.drawPath(path, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}

```

StrokeCap.butt

![butt](https://i.imgur.com/KstcfOt.png)

StrokeCap.square

![square](https://i.imgur.com/tgbjCkU.png)

StrokeCap.round

![round](https://i.imgur.com/7vFSvd7.png)

可以看到，StrokeCap.butt 样式路径开始和结束轮廓具有平坦的边缘且没有延伸，而StrokeCap.square
虽然同样有平坦的边缘但却有所延伸，StrokeCap.round则有圆润的边缘且同样有所延伸。

OK！画笔就讲到这里。这些就是我们常用的画笔属性。

## Canvas 如何使用
可以看到canvas给我们提供了很多方法，下面解释一些常用方法的意思。

drawLine	画线

drawPoint	画点

drawPath	画路径

drawImage	画图像

drawRect	画矩形

drawCircle	画圆

drawOval	画椭圆

drawArc	画圆弧

drawDRRect 画两个矩形的差异部分

rotate 旋转

transform 变换

translate 平移











