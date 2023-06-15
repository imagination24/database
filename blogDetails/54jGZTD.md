
# 关于Flutter与原生通讯的几种方式的使用

## 目录
- [关于Flutter与原生通讯的几种方式的使用](#--flutter-------------)
  * [BasicMessageChannel](#basicmessagechannel)
  * [MethodChannel](#methodchannel)
  * [EventChannel](#eventchannel)
- [如何实现Channel?两种方式](#----channel-----)
  * [1.实现Plugin（AndroidStudio演示）](#1--plugin-androidstudio---)
  * [2.手动注册（Android演示）](#2-----android---)



Flutter 与 原生通讯的方式有以下三种

BasicMessageChannel

MethodChannel

EventChannel


三种Channel的构造函数都有三个成员变量

*name : Channel的名字*

*codec : MessageCodec 类型或 MethodCodec 类型，代表消息的编解码器*

*binaryMessenger : BinaryMessenger 类型，代表消息信使，是消息的发送与接收的工具*

*需要注意的是原生端和Flutter端的Channel名一定要一致*
*热重载对涉及原生代码的更新无效*

## BasicMessageChannel
*BasicMessageChannel是可以双端通信的，Flutter 端可以给 iOS 发送消息，iOS 也可以给 Flutter 发送消息。这段代码实现了在 Flutter 中的 TextField 输入文字，在 iOS 端能及时输出。*

Flutter端
```
  basicMessageChannelTest(){
    ///创建Channel
    BasicMessageChannel basicMessageChannel = const BasicMessageChannel("skyCastle", StandardMessageCodec());
    ///发送消息
    basicMessageChannel.send("Mayday!Mayday!");
    ///监听消息并返回结果
    basicMessageChannel.setMessageHandler((message)async{
      print(message);
      return "Mayday!Mayday!";
    });
  }
```
Native端(Android)
```
class BasicMessageChannelDemo(messenger: BinaryMessenger) : BasicMessageChannel.MessageHandler<Any> {

    private var channel: BasicMessageChannel<Any>

    init {
        channel = BasicMessageChannel(messenger, "skyCastle", StandardMessageCodec())
        channel.setMessageHandler(this)
    }

    //接受消息
    override fun onMessage(message: Any?, reply: BasicMessageChannel.Reply<Any>) {
        val mssage = message as String
        reply.reply("wow")
    }
}

```

## MethodChannel
*MethodChannel一般用于原生代码的调用*
*MethodChannel的方法名也要保持一致*
*MethodChannel无需手动分离，通道返回后会自动调用onDetachedFromEngine*

Flutter端
```
  methodChannelTest()async{
    ///创建Channel
    MethodChannel methodChannel = const MethodChannel("SkyCastle");
    ///调用方法(携带参数)并获取结果
    var result = await methodChannel.invokeMethod("MethodPlus",{"a":1,"b":2});
    print(result);
  }
```
Native端（Android）
```
  public class SkyCastlePlugin implements FlutterPlugin, MethodCallHandler {

  private final String CHANNEL_TAG = "SkyCastle";


  private MethodChannel channel;
  private Context appContext;

  //如名字可知，注册插件到FlutterEngine
  @Override
  public void onAttachedToEngine(@NonNull FlutterPluginBinding flutterPluginBinding) {
    appContext = flutterPluginBinding.getApplicationContext();
    channel = new MethodChannel(flutterPluginBinding.getBinaryMessenger(),CHANNEL_TAG);
    channel.setMethodCallHandler(this);
  }



  //调用方法
  @Override
  public void onMethodCall(@NonNull MethodCall call, @NonNull Result result) {
    int a = call.argument("a");
    int b = call.argument("b");
    switch (call.method) {
      case "MethodPlus":
        result.success(a+b);
        break;
    }
  }
  //于FlutterEngine分离
  @Override
  public void onDetachedFromEngine(@NonNull FlutterPluginBinding binding) {
    channel.setMethodCallHandler(null);
  }
}
```

## EventChannel
*EventChannel只能是原生以Stream的方式发送消息给 Flutter 端。如果不需要再收听回调，可以在原生端或者Flutter端取消订阅*

Flutter端
```
  eventChannelTest()async{
    ///创建Channel
    EventChannel eventChannel = const EventChannel("SkyCastle");
    StreamSubscription ? subscription;
    ///订阅回调
    subscription = eventChannel.receiveBroadcastStream("arguments").listen((event) {
      print(event);
      ///如果不需要可以取消订阅
      subscription?.cancel();
    });
  }
```
Native端
```
public class MyEventChannel implements FlutterStreamHandler {
  private EventChannel.EventSink eventSink;
 
  @Override
  public void onListen(Object arguments, EventChannel.EventSink events) {
    // Flutter 应用打开了事件通道 
    eventSink = events;
  }
 
  @Override
  public void onCancel(Object arguments) {
    // Flutter 应用关闭了事件通道
    eventSink = null;
  }
 
  public void sendEvent(String message) {
    // 向 Flutter 应用程序发送消息
    if (eventSink != null) {
      eventSink.success(message);
    }
  }
}
```


# 如何实现Channel?两种方式

## 1.实现Plugin（AndroidStudio演示）

在AndroidStudio file卡 ，点击New ,点击Flutter Project


![AndroidStudio](https://i.imgur.com/jFpaJwM.png)

选择FlutterSDK路径，然后点击Next


![path](https://i.imgur.com/Pv7YMTt.png)

填写相应信息，并选择项目类型为Plugin


*名字不能大写*

*这里需要注意的是，Flutter可以使用项目地址来引用插件，如果需要用地址来引用，需要把地址写在主项目内*

![info](https://i.imgur.com/HAB0jlT.png)

初始结构

![stru](https://i.imgur.com/PpyPu9c.png)

如何引用

*纯演示*

  上传到github,使用github地址引用
  ```
  skycastle:
      git:
        url: https://github.com/imagination24/skycastle.git
  ```
  使用plugin地址，做为本机插件使用
  ```
  skycastle:
    path: *项目地址*
  ```
  上传到pub,使用项目名引用
  ```
    skycastle: ^0.0.1
  ```
## 2.手动注册（Android演示）
在主项目的原生代码库中新建类编写原生代码
```
public class SkyCastlePlugin implements FlutterPlugin, MethodCallHandler {

    private MethodChannel channel;
    private Context context;


    @Override
    public void onAttachedToEngine(@NonNull FlutterPluginBinding flutterPluginBinding) {
        channel = new MethodChannel(flutterPluginBinding.getBinaryMessenger(), "SkyCastle");
        channel.setMethodCallHandler(this);
    }


    @Override
    public void onDetachedFromEngine(@NonNull FlutterPluginBinding binding) {
        channel.setMethodCallHandler(null);
    }

    @Override
    public void onMethodCall(@NonNull MethodCall call, @NonNull MethodChannel.Result result) {
        
    }
}
```
在MainActivity中注册插件
```
public class MainActivity extends FlutterActivity {
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
    }

    @Override
    public void configureFlutterEngine(@NonNull FlutterEngine flutterEngine) {
        super.configureFlutterEngine(flutterEngine);
        GeneratedPluginRegistrant.registerWith(flutterEngine);

        try {
            flutterEngine.getPlugins().add(new SkyCastlePlugin());
        }catch (Exception e){
            Log.e("TAG", "Error registering plugin SkyCastlePlugin", e);
        }
    }
}
```
