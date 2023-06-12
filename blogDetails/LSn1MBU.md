# Flutter Canvas 的使用
*使用Canvas自定义Widget*

总所周知，FlutterWidgets种类繁多，各种wigets达到三百多个，一般情况下用Flutter定义好的widget就可以实现我们想要的效果。但如果需求特殊，Widgets库里找不到合适组件可以使用，就可以用Canvas来画自己的组件了。

## Paint 如何使用？
我们先学习一下Canvas如何使用
首先定义一个Painter
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
然后建立一个Containter和CustomPaint来承载Painter，然后先把Painter注释掉。
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
得到了一个300x300的黄色小方块

![yellowContainer](https://i.imgur.com/bSOkO2o.png)

要开始画画，需要先实现并重写paint()。

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

可以看到，如果要在canvas上画东西，需要新建一个画笔，画笔有很多属性可以定义，接下来就先讲一下画笔的属性。
### 画笔的属性
*color*

画笔的颜色

*strokeWidth*

画笔的宽度

*style(PaintingStyle)*

这个属性有两个选择 PaintingStyle.stroke 和 PaintingStyle.fill 

让我们看看有什么区别

画线看不出来所以画一个矩形
```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 5..style = PaintingStyle.stroke;
    //定义一个矩形，包含偏移和size
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

是否抗锯齿

*shader*

覆盖在形状上的遮罩
```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    //定义一个矩形，包含偏移和size
    Rect rect = Offset(size.width/3, size.width/3)&const Size(100, 100);
    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 5
      ..style = PaintingStyle.fill
      ..isAntiAlias = true;
    ///定义一个包含两个颜色的线性渐变遮罩
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
    ///移动路径起始点
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

OK！画笔就讲到这里。这些就是常用的画笔属性。

## Canvas 如何使用
可以看到canvas提供了很多绘制方法，下面解释一些常用绘制方法的意思。

drawLine 绘制线条

drawPoint 绘制点

drawPath 绘制路径

drawRect 绘制矩形

drawCircle 绘制圆

drawOval 绘制椭圆

drawArc 绘制圆弧

drawDRRect 绘制两个矩形的差异部分

drawShadow 绘制阴影

rotate 旋转



上面已经演示过绘制线条，矩形和路径。剩下的我们逐一尝试

drawPoint 绘制点

```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {

    ///生成一个包含一些Offset的list,offset就是偏移，可以理解为点的位置
    List<Offset> points = List.generate(9, (index) => Offset((index+1)*30, size.height/2));

    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 15
      ..style = PaintingStyle.stroke
      ..isAntiAlias = true
      ..strokeCap = StrokeCap.round;
    ///画一些点
    canvas.drawPoints(PointMode.points, points,paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

![point](https://i.imgur.com/YsNPhKD.png)

drawCircle 绘制圆

```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {


    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 10
      ..style = PaintingStyle.stroke
      ..isAntiAlias = true
      ..strokeCap = StrokeCap.round;
    Offset circleCenter = Offset(size.width/2, size.height/2);
    ///c 中心点
    ///radius 半径
    ///-5是因为我们定义画笔宽度是10,我们需要半径减一半的宽度确保画笔内容全部都在画布内
    canvas.drawCircle(circleCenter, 150-5, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

![cicle](https://i.imgur.com/H0lkoFm.png)


drawOval 绘制椭圆

```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 10
      ..style = PaintingStyle.stroke
      ..isAntiAlias = true
      ..strokeCap = StrokeCap.round;
    ///定义一个矩形
    Rect rect = Offset(size.width/5, size.width/3)&const Size(200, 100);
    ///椭圆会根据形状绘制，如果是一个正方形，那绘制的就是一个正圆
    canvas.drawOval(rect, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

![oval](https://i.imgur.com/im1SXBA.png)


drawArc 绘制圆弧

```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 10
      ..style = PaintingStyle.stroke
      ..isAntiAlias = true
      ..strokeCap = StrokeCap.round;
    ///定义一个矩形
    Rect mount = Offset(size.width/3, size.width/3)&const Size(100, 100);
    canvas.drawArc(mount,0,pi,false,paint);
    ///干脆画一个笑脸吧
    Rect leftEye = Offset(size.width/4, size.width/4)&const Size(50 , 50);
    Rect rightEye = Offset(size.width/4+50*2, size.width/4)&const Size(50 , 50);
    canvas.drawArc(leftEye, pi, pi, false, paint);
    canvas.drawArc(rightEye, pi, pi, false, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

![arc](https://i.imgur.com/aqSEczl.png)

drawDRRect 绘制两个矩形的差异部分

```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 10
      ..style = PaintingStyle.fill
      ..isAntiAlias = true
      ..strokeCap = StrokeCap.round;
    RRect outer = RRect.fromRectAndRadius(Offset(size.width/3, size.width/3)&const Size(100, 100), Radius.circular(5));
    RRect inner = RRect.fromRectAndRadius(Offset(size.width/3+25, size.width/3+25)&const Size(50, 50), Radius.circular(5));
    canvas.drawDRRect(outer, inner, paint);
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

![drrect](https://i.imgur.com/2eQ1AUU.png)


drawShadow 绘制阴影

```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    Rect rect = Offset(size.width/3, size.width/3)&const Size(100, 100);
    Path shadowPath = Path()..moveTo(size.width/3, size.width/3);
    shadowPath.addRect(rect.translate(5, 5));
    canvas.drawShadow(shadowPath, Colors.grey, 10, true);

    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 10
      ..style = PaintingStyle.fill
      ..isAntiAlias = true
      ..strokeCap = StrokeCap.round;

    canvas.drawRect(rect, paint);

  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

![shadow](https://i.imgur.com/ticMMZk.png)

rotate 旋转

*需要注意的是旋转的是画布，旋转的中心点是左上角，这无法改变，也不需要疑惑为什么画笔可以画出画布，customPainter是允许越界绘制的。*

```
class SkyCastlePainter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    ///创建画笔
    Paint paint = Paint()
      ..color = Colors.lightGreen
      ..strokeWidth= 10
      ..style = PaintingStyle.fill
      ..isAntiAlias = true
      ..strokeCap = StrokeCap.round;

    ///画旋转中心
    canvas.drawPoints(PointMode.points, [Offset(0,0),Offset(size.width/3, size.width/3)], paint);


    Rect rect = Offset(size.width/3, size.width/3)&const Size(100, 100);
    Path shadowPath = Path()..moveTo(size.width/3, size.width/3);
    shadowPath.addRect(rect.translate(5, 5));
    canvas.drawShadow(shadowPath, Colors.grey, 10, true);
    canvas.drawRect(rect, paint);

    ///旋转300°
    canvas.save();
    canvas.rotate(300*pi/180);
    canvas.drawShadow(shadowPath, Colors.grey, 10, true);
    canvas.drawRect(rect, paint);
    canvas.restore();



    ///旋转45°
    canvas.save();
    canvas.rotate(pi/4);

    canvas.drawShadow(shadowPath, Colors.grey, 10, true);
    canvas.drawRect(rect, paint);
    canvas.restore();

  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => false;
}
```

![shadow](https://i.imgur.com/wGo4KgP.png)


## 实战（一）低质量模仿AppleWatch表盘

AppWatch表盘

![appleWatch](https://i.imgur.com/xhXVE54.png)

低质量模仿

![watch](https://i.imgur.com/5ODNy5D.png)

```
/*
Time:2023/6/12
Description:
Author:
*/

import 'dart:math';

import 'package:flutter/material.dart';

class Watch extends StatelessWidget {
  final double width;
  final Color? clothesColor;
  final Color? skinColor;
  final Color? hairColor;
  final Color? backgroundColor;
  final String? text;
  final TextStyle? textStyle;

  const Watch(
      {super.key,
      required this.width,
      this.clothesColor,
      this.skinColor,
      this.hairColor,
      this.text,
      this.backgroundColor,
      this.textStyle});

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width: width,
      height: width,
      child: ClipRRect(
        borderRadius: BorderRadius.circular(width/4),
        child: CustomPaint(
          painter: _WatchPainter(
              width: width,
              clothesColor: clothesColor,
              skinColor: skinColor,
              hairColor: hairColor,
              backgroundColor: backgroundColor,
              text: text,
              textStyle: textStyle),
        ),
      ),
    );
  }
}

class _WatchPainter extends CustomPainter {
  static const Color backgroundColorOrigin = Color.fromRGBO(82, 102, 101, 1);
  static const Color clothesColorOrigin = Color.fromRGBO(175, 104, 81, 1);
  static const Color skinColorOrigin = Color.fromRGBO(202, 139, 109, 1);
  static const Color hairColorOrigin = Color.fromRGBO(157, 130, 112, 1);
  static const Color mountColorOrigin = Color.fromRGBO(69, 64, 64, 1);
  final double width;
  final Color? clothesColor;
  final Color? skinColor;
  final Color? hairColor;
  final Color? backgroundColor;
  final String? text;
  final TextStyle? textStyle;

  _WatchPainter(
      {required this.width,
      this.clothesColor,
      this.skinColor,
      this.hairColor,
      this.backgroundColor,
      this.text,
      this.textStyle});

  @override
  void paint(Canvas canvas, Size size) {
    ///创建画笔
    Paint paint = Paint()
      ..color = backgroundColor??backgroundColorOrigin
      ..strokeWidth = 10
      ..style = PaintingStyle.fill
      ..strokeCap = StrokeCap.round
      ..isAntiAlias = true;

    ///定义背景形状
    Rect rect = Offset.zero & Size(width, width);
    RRect rRect = RRect.fromRectAndRadius(rect,Radius.circular(width/4));
    canvas.drawRRect(rRect, paint);

    ///画身体
    paint.color = clothesColor ?? clothesColorOrigin;
    Rect ovalRect = Offset(0, width * 4 / 5) & Size(width, width / 2);
    canvas.drawOval(ovalRect, paint);

    ///画脖子
    paint.color = skinColor??skinColorOrigin;
    Path neckPath = Path()..moveTo(width*2/3, width*3/5);
    neckPath.lineTo(width*2/3, width*5/6);
    neckPath.arcToPoint(Offset(width/3, width*5/6),radius: Radius.circular(width/5));
    neckPath.lineTo(width/3, width*3/5);
    neckPath.close();
    canvas.drawPath(neckPath, paint);

    ///画脸
    Path facePath = Path()..moveTo(width*5/6, width/5);
    facePath.quadraticBezierTo(width*19/20, width*10/11, width/2, width*3/4);
    facePath.quadraticBezierTo(width/3, width*11/16, width/4, width*3/5);
    facePath.lineTo(width/4,  width/5);
    facePath.close();
    canvas.drawShadow(facePath, Colors.black12.withOpacity(0.5), 5, true);
    canvas.drawPath(facePath, paint);

    ///画头发
    paint.color = hairColor??hairColorOrigin;
    Path hairPath = Path()..moveTo(width*6/7, width/5);
    hairPath.quadraticBezierTo(width*5/6, width/4, width*2/3, width/4);
    hairPath.quadraticBezierTo(width/3,  width/4, width/3, width/2);
    hairPath.quadraticBezierTo(width/3, width*2/3, width/4, width*2/3);
    hairPath.quadraticBezierTo(width/6, width*2/3, width/7, width/2);
    hairPath.quadraticBezierTo(width/10, width/8, width/3,  width/15);
    hairPath.quadraticBezierTo(width/2, width/40, width*2/3,  width/15);
    hairPath.quadraticBezierTo(width*8/9, width/9, width*6/7, width/5);
    canvas.drawShadow(hairPath,Colors.black12.withOpacity(0.5), 5, true);
    canvas.drawPath(hairPath, paint);

    ///嘴巴
    paint.color = mountColorOrigin;
    paint.style = PaintingStyle.stroke;
    paint.strokeWidth = width/60;
    Rect mountRect = Offset(size.width*5/9, size.width*4/7)&Size(width/5, width/10);
    canvas.drawArc(mountRect, pi/4, pi/2, false, paint);

    ///文字
    TextPainter textPainter = TextPainter(textDirection: TextDirection.ltr);
    textPainter.text = TextSpan(
        text: text ?? "${DateTime.now().hour}\\${DateTime.now().minute}",
        style: textStyle??TextStyle(color: mountColorOrigin,fontSize: width/6),
    );
    textPainter.layout();
    textPainter.paint(canvas, Offset((width-textPainter.width)*2/3, (width-textPainter.height)*2/5));
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) => true;
}

```


