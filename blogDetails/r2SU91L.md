# Flutter如何实现黑暗模式（Provider+ThemeData）

黑暗模式是一种对夜晚友好的模式，当用户处于暗光环境下，并不想要被过亮的屏幕刺伤眼镜时，黑暗模式往往有很好的效果。那么Flutter如何实现黑暗模式?事实上这很简单，Flutter作为一个友好的UI框架早就设计好了接口等待工程师去实现自己的黑暗模式。

## ThemeData
要实现黑暗模式，首先我们要使用ThemeData定义好黑暗模式和明亮模式的各种颜色和字体的配置，一般来说会创建一个AppThemes文件。

```
class AppThemes {
  static ThemeData light = ThemeData.light().copyWith(
    
  );
  
  static ThemeData dark = ThemeData.dark().copyWith(
    
  );
}
```
使用copyWith的好处是，light原有的颜色会被继承下来，如果没有被改动，则会使用原有的颜色。

里面有非常多的配置选项，具体可以看theme_data.dart文档，或者是文末的解释。

那么ThemeData如何应用在代码中呢？如下，需要注意的是有一些配置会自动应用，不需要自己应用。
```
          //textStyle
          Text(
              "SkyCastle",
              style: Theme.of(context).textTheme.displaySmall
          )
          //color
          Container(
            ···
            decoration: BoxDecoration(
              ···
              color: Theme.of(context).hoverColor
            ),
            ···
          )
```

需要注意的是如果ThemeData应用在代码上不起效果，可能是Scffold重写了主题，需要在wigget上套一个Theme,具体代码如下
```
Widget _buildPage(BuildContext context) {
    return Theme(
      data: Theme.of(context),
      child: Scaffold(
        body: Container(
          padding: EdgeInsets.only(left: 40.w,right: 40.w),
          color: Theme.of(context).primaryColor,
        ),
      ),
    );
  }
```
<br/><br/><br/>
## MaterialApp配置
MaterialApp中有三个相关属性需要配置，如下，记住这个themeMode，等下要用。
```
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      ···
      title: "SkyCastle",
      theme: AppThemes.light,
      darkTheme: AppThemes.dark,
      themeMode: ThemeMode.system,
      ···
    );
  }
}
```
<br/><br/><br/>
## Provider配置
我们新建一个私有变量Mode，并重写get/set函数，暴露变量，并在改变变量时通知刷新。并写一个通过index获取ThemeMode的函数
```
class AppSetting extends ChangeNotifier{
  ThemeMode _mode = ThemeMode.system;


  ThemeMode get mode => _mode;

  set mode(ThemeMode mode){
    _mode = mode;
    notifyListeners();
    //保存数据
  }
  
    ThemeMode getThemeMode(int index){
    switch (index) {
      case 0:
        return ThemeMode.system;
      case 1:
        return ThemeMode.light;
      case 2:
        return ThemeMode.dark;
      default:
        return ThemeMode.system;
    }
  }
  
}
```
配置好之后，我们回到MaterialApp设置,把themeMode的值改成我们新建的变量，并用Provider的watch模式监听值的变化。同时我们要在MaterialApp之前创建AppSettingProvider.
```
void main() {
  runApp(ChangeNotifierProvider(
    create:(_) => AppSetting(),
    builder:(context,child)=>MyApp()，
  ));
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      ···
      title: "SkyCastle",
      theme: AppThemes.light,
      darkTheme: AppThemes.dark,
      themeMode: context.watch<AppSetting>().mode,
      ···
    );
  }
}
```
这样我们的功能编写就完成了，接下来是UI
<br/><br/><br/>
## DarkModeUI
关键代码如下
```
···
Selector<AppSetting, int>(
              selector: (context, appSetting) => appSetting.themeMode.index,
              builder: (context, mode, child) {
                return Center(
                  child: DropdownButton(
                    underline: Container(),
                    value: mode,
                    items: List.generate(
                        3,
                        (index) => DropdownMenuItem<int>(
                            key: UniqueKey(),
                            value: index,
                            child: SizedBox(
                                height: 30.h,
                                child: Center(
                                    child: Text(provider.modeString(index),
                                        style: Theme.of(context)
                                            .textTheme
                                            .labelLarge))))),
                    onChanged: (int? value) =>
                        appSetting.themeMode = appSetting.getThemeMode(value!),
                  ),
                );
              },
            )
            ···
```
至此功能就完成了，效果如下
![light](https://i.imgur.com/S3ufbDR.png)
![dark](https://i.imgur.com/e8JDzO5.png)

<br/><br/><br/>
## ThemeData属性解释
*bool? applyElevationOverlayColor*

是否开启Material 2 黑暗模式下叠加层的阴影颜色，如果开启了Material 3就无效了
<br/><br/><br/>
*NoDefaultCupertinoThemeData? cupertinoOverrideTheme*

默认情况下，cupertinoOverrideTheme为空，Material Theme 的后代 Cupertino 小部件将遵循从 Material ThemeData 派生的 CupertinoTheme。如果此项重写了， Cupertino 小部件将会遵循你重写的颜色.以下是可以重写的属性。
```
NoDefaultCupertinoThemeData NoDefaultCupertinoThemeData({
    Brightness? brightness,
    Color? primaryColor,
    Color? primaryContrastingColor,
    CupertinoTextThemeData? textTheme,
    Color? barBackgroundColor,
    Color? scaffoldBackgroundColor, })
```
<br/><br/><br/>
*Iterable<ThemeExtension<dynamic>>? extensions*
  
null
<br/><br/><br/>
*inputDecorationTheme*
  
  输入装饰主题，[InputDecorator]、[TextField] 和 [TextFormField] 的默认 [InputDecoration] 值基于此主题。可以定义的配置如下。
```
 InputDecorationTheme InputDecorationTheme({
  TextStyle? labelStyle,
  TextStyle? floatingLabelStyle,
  TextStyle? helperStyle,
  int? helperMaxLines,
  TextStyle? hintStyle,
  TextStyle? errorStyle,
  int? errorMaxLines,
  FloatingLabelBehavior floatingLabelBehavior = FloatingLabelBehavior.auto,
  FloatingLabelAlignment floatingLabelAlignment = FloatingLabelAlignment.start,
  bool isDense = false,
  EdgeInsetsGeometry? contentPadding,
  bool isCollapsed = false,
  Color? iconColor,
  TextStyle? prefixStyle,
  Color? prefixIconColor,
  TextStyle? suffixStyle,
  Color? suffixIconColor,
  TextStyle? counterStyle,
  bool filled = false,
  Color? fillColor,
  BorderSide? activeIndicatorBorder,
  BorderSide? outlineBorder,
  Color? focusColor,
  Color? hoverColor,
  InputBorder? errorBorder,
  InputBorder? focusedBorder,
  InputBorder? focusedErrorBorder,
  InputBorder? disabledBorder,
  InputBorder? enabledBorder,
  InputBorder? border,
  bool alignLabelWithHint = false,
  BoxConstraints? constraints, }) 
```
  <br/><br/><br/>
```
bool? applyElevationOverlayColor,
NoDefaultCupertinoThemeData? cupertinoOverrideTheme,
Iterable<ThemeExtension<dynamic>>? extensions,
InputDecorationTheme? inputDecorationTheme,
MaterialTapTargetSize? materialTapTargetSize,
PageTransitionsTheme? pageTransitionsTheme,
TargetPlatform? platform,
ScrollbarThemeData? scrollbarTheme,
InteractiveInkFeatureFactory? splashFactory,
bool? useMaterial3,
VisualDensity? visualDensity,
Brightness? brightness,
Color? canvasColor,
Color? cardColor,
ColorScheme? colorScheme,
Color? dialogBackgroundColor,
Color? disabledColor,
Color? dividerColor,
Color? focusColor,
Color? highlightColor,
Color? hintColor,
Color? hoverColor,
Color? indicatorColor,
Color? primaryColor,
Color? primaryColorDark,
Color? primaryColorLight,
Color? scaffoldBackgroundColor,
Color? secondaryHeaderColor,
Color? shadowColor,
Color? splashColor, 
Color? unselectedWidgetColor,
IconThemeData? iconTheme,
IconThemeData? primaryIconTheme,
TextTheme? primaryTextTheme,
TextTheme? textTheme,
Typography? typography,
AppBarTheme? appBarTheme,
BadgeThemeData? badgeTheme,
MaterialBannerThemeData? bannerTheme,
BottomAppBarTheme? bottomAppBarTheme,
BottomNavigationBarThemeData? bottomNavigationBarTheme,
BottomSheetThemeData? bottomSheetTheme,
ButtonBarThemeData? buttonBarTheme,
ButtonThemeData? buttonTheme,
CardTheme? cardTheme,
CheckboxThemeData? checkboxTheme,
ChipThemeData? chipTheme,
DataTableThemeData? dataTableTheme, 
DialogTheme? dialogTheme,
DividerThemeData? dividerTheme,
DrawerThemeData? drawerTheme,
DropdownMenuThemeData? dropdownMenuTheme,
ElevatedButtonThemeData? elevatedButtonTheme,
ExpansionTileThemeData? expansionTileTheme,
FilledButtonThemeData? filledButtonTheme,
FloatingActionButtonThemeData? floatingActionButtonTheme,
IconButtonThemeData? iconButtonTheme,
ListTileThemeData? listTileTheme,
MenuBarThemeData? menuBarTheme,
MenuButtonThemeData? menuButtonTheme,
MenuThemeData? menuTheme,
NavigationBarThemeData? navigationBarTheme,
NavigationDrawerThemeData? navigationDrawerTheme,
NavigationRailThemeData? navigationRailTheme,
OutlinedButtonThemeData? outlinedButtonTheme,
PopupMenuThemeData? popupMenuTheme,
ProgressIndicatorThemeData? progressIndicatorTheme,
RadioThemeData? radioTheme,
SegmentedButtonThemeData? segmentedButtonTheme,
SliderThemeData? sliderTheme,
SnackBarThemeData? snackBarTheme,
SwitchThemeData? switchTheme,
TabBarTheme? tabBarTheme,
TextButtonThemeData? textButtonTheme,
TextSelectionThemeData? textSelectionTheme,
TimePickerThemeData? timePickerTheme,
ToggleButtonsThemeData? toggleButtonsTheme,
TooltipThemeData? tooltipTheme,
Color? accentColor,
Brightness? accentColorBrightness,
TextTheme? accentTextTheme,
IconThemeData? accentIconTheme,
Color? buttonColor,
bool? fixTextFieldOutlineLabel,
Brightness? primaryColorBrightness,
AndroidOverscrollIndicator? androidOverscrollIndicator,
Color? toggleableActiveColor,
Color? selectedRowColor,
Color? errorColor,
Color? backgroundColor,
Color? bottomAppBarColor,
```
